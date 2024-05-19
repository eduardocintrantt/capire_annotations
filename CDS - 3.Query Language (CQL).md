#### Projeções Posfixas
As projeções posfixas permitem colocar a cláusula SELECT após a cláusula FROM dentro de chaves. Exemplos equivalentes:
```sql
SELECT name, address.street from Authors
SELECT from Authors { name, address.street }
```

#### Expansões Aninhadas
As projeções posfixas podem ser aplicadas a qualquer coluna que se refere a um elemento de estrutura ou associação, permitindo expansões aninhadas e leitura de documentos estruturados profundamente:
```sql
SELECT from Authors { name, address { street, town { name, country } } }
```

Resultado estruturado:
```js
results = [
  {
    name: 'Victor Hugo',
    address: {
      street: '6 Place des Vosges', town: {
        name: 'Paris',
        country: 'France'
      }
    }
  },
  {
    name: 'Emily Brontë', …
  }, …
]
```

Essa funcionalidade é voltada para bancos de dados NoSQL e não tem equivalente em SQL padrão.

#### Alias
Um alias pode ser fornecido para elementos de estrutura ou associações precedentes na projeção posfixa:
```sql
SELECT from Authors { name, address as residence { street, town as city { name, country } } }
```

Resultado:
```js
results = [
  {
    name: 'Victor Hugo',
    residence: {
      street: '6 Place des Vosges', city: {
        name: 'Paris',
        country: 'France'
      }
    }
  }, …
]
```

#### Expressões
Projeções posfixas podem conter expressões, criando novas estruturas:
```sql
SELECT from Books {
  title,
  author { name, dateOfDeath - dateOfBirth as age },
  { stock as number, stock * price as value } as stock
}
```

Resultado:
```js
results = [
  {
    title: 'Wuthering Heights',
    author: {
      name: 'Emily Brontë',
      age: 30
    },
    stock: {
      number: 12,
      value: 133.32
    }
  }, …
]
```

#### Inlines Aninhados
Use "." antes da chave de abertura para inlines aninhados, evitando listas extensas de caminhos:
```sql
SELECT from Authors { name, address.{ street, town.{ name, country } } }
```

Equivalente a:
```sql
SELECT from Authors { name, address.street, address.town.name, address.town.country }
```

Inlines aninhados podem conter expressões:
```sql
SELECT from Books {
  title,
  author.{
    name, dateOfDeath - dateOfBirth as author_age,
    address.town.{ concat(name, '/', country) as author_town }
  }
}
```

#### Seletor *
Dentro de projeções posfixas, o operador * seleciona todos os elementos, substituindo explicitamente colunas nomeadas posteriormente.
```sql
SELECT from Books { *, author.name as author }
```

#### Cláusula Excluding
A cláusula excluding em combinação com SELECT * seleciona todos os elementos, exceto os listados na cláusula excluding.
```sql
SELECT from Books { * } excluding { author }
```

#### Expansões Aninhadas com *
O seletor * após uma associação seleciona todos os elementos do alvo da associação.
```sql
SELECT from Books { title, author { * } }
```

#### Expressões de Caminho
Use expressões de caminho para navegar por associações e/ou elementos estruturais em qualquer cláusula SQL.
```sql
SELECT title, author.name from Books;
SELECT *, author.address.town.name from Books;
```

#### Predicado Exists
Use uma expressão de caminho filtrada para testar se algum elemento da coleção associada corresponde ao filtro.
```sql
SELECT FROM Authors {name} WHERE EXISTS books[year = 2000]
```

Resultado:
```sql
SELECT name FROM Authors
WHERE EXISTS (
        SELECT 1 FROM Books
        WHERE Books.author_id = Authors.id
            AND Books.year = 2000
    )
```

#### Definições de Associação

Use mixin...into para adicionar associações não gerenciadas logicamente à fonte da consulta:
```sql
SELECT from Books mixin {
  localized : Association to LocalizedBooks on localized.ID = ID;
} into {
  ID, localized.title
}
```

Adicione uma associação não gerenciada diretamente na lista de seleção da consulta:
```cds
entity BookReviews as select from Reviews {
  ...,
  subject as bookID,
  book : Association to Books on book.ID = bookID
}
```

Ou adicione novas associações não gerenciadas a uma projeção ou visualização via extend:
```cds
extend BookReviews with columns {
  subject as bookID,
  book : Association to Books on book.ID = bookID
}
```

#### Resumo
Este artigo cobre as principais funcionalidades e uso de projeções posfixas e expansões aninhadas no CQL, destacando como elas permitem consultas avançadas e estruturadas em dados relacionados.
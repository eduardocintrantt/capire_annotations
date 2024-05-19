#### Definição de Entidades

- **Entidades** são tipos estruturados com elementos nomeados e tipados, representando conjuntos de dados persistentes manipuláveis via operações CRUD.
- **Exemplo:**
```cds
define entity Employees {
  key ID : Integer;
  name : String;
  jobTitle : String;
}
```
- A palavra-chave `define` é opcional.

#### Definição de Tipos
- **Tipos Personalizados** podem ser simples (derivados de tipos predefinidos), tipos estruturados ou associações.
- **Exemplo:**
```cds
define type User : String(111);
define type Amount {
  value : Decimal(10,3);
  currency : Currency;
}
define type Currency : Association to Currencies;
```
#### Tipos Predefinidos e Estruturados
- **Tipos Predefinidos:** Lista de tipos embutidos disponível na documentação.
- **Tipos Estruturados:**
    ```cds
    type Amount {
	  value : Decimal(10,3);
	  currency : Currency;
	}
	entity Books {
	  price : Amount;
	}
	```

#### Tipos Array
- **Sintaxe:** `array of` ou `many`
```cds
entity Foo { emails: many String; }
```

#### Elementos Virtuais
- **Elementos Virtuais** não são persistidos em bancos de dados SQL e são parte dos metadados do OData.
```cds
entity Employees {
  virtual something : String(11);
}
```

#### Literals
- **Literals:** null, true, false, números, strings, datas, horários, timestamps, etc.
```cds
entity Example {
  defaultValue : String default 'example';
}
```

#### Identificadores Delimitados
- **Permite o uso de caracteres especiais ou palavras-chave como identificadores.**
```cds
entity ![Entity] {
  bar : ![Keyword];
}
```

#### Elementos Calculados
- **On-read:** Calculados na leitura.
```cds
entity Employees {
  firstName : String;
  lastName : String;
  name : String = firstName || ' ' || lastName;
}
```
    
- **On-write:** Calculados na escrita e armazenados.
```cds
entity Employees {
  firstName : String;
  lastName : String;
  name : String = (firstName || ' ' || lastName) stored;
}
```
#### Valores Padrão
- **Especificação de valores padrão para inserções.**
```cds
entity Foo {
  bar : String default 'bar';
}
```

#### Referências de Tipo
- **Uso do operador `type of` para referenciar tipos de elementos.**
```cds
entity Author {
  firstname : String(100);
  lastname : type of firstname;
}
```

#### Restrições
- **Definição de elementos com restrição `not null`.**
```cds
entity Employees {
  name : String(111) not null;
}
```

#### Enums
- **Especificação de valores de enumeração.**
```cds
type Gender : String enum { male; female; non_binary = 'non-binary'; }
```

#### Views e Projeções

- **Definição de novas entidades a partir de existentes usando `as select from` ou `as projection on`.**
``` cds
entity Foo1 as select from Bar;
entity Foo2 as projection on Bar {...}
```

#### Associações e Composições
- **Associações** capturam relações entre entidades.
```cds
entity Employees {
  address : Association to Addresses;
}
```

#### Associações To-Many

- **Associações To-Many** especificam uma condição `on` usando o padrão `<assoc>.<backlink> = $self`.
```cds
	entity Employees {
		  key ID : Integer;
		  addresses : Association to many Addresses on addresses.owner = $self;
		}
		entity Addresses {
		  owner : Association to Employees;  //> backlink
		}
	}
```

#### Associações Many-to-Many
- **Associações Many-to-Many** usam uma entidade de ligação para resolver a relação lógica.
```cds
entity Employees {
  addresses : Association to many Emp2Addr on addresses.emp = $self;
}
entity Emp2Addr {
  key emp : Association to Employees;
  key adr : Association to Addresses;
}
```
    
#### Composições

- **Composições** criam estruturas de documentos através de relações de contido-em.
```cds
entity Orders {
  key ID: Integer;
  Items : Composition of many Orders.Items on Items.parent = $self;
}
entity Orders.Items {
  key pos : Integer;
  key parent : Association to Orders;
  product : Association to Products;
  quantity : Integer;
}
```

#### Composições Gerenciadas
- **Composições Gerenciadas** refletem estruturas de documentos sem a necessidade de entidades separadas.
```cds
entity Orders {
  key ID: Integer;
  Items : Composition of many {
    key pos : Integer;
    product : Association to Products;
    quantity : Integer;
  }
};
```
    
- **Composições com Alvos Nomeados**
```cds
entity Orders {
  key ID: Integer;
  Items : Composition of many OrderItems;
}
aspect OrderItems {
  key pos : Integer;
  product : Association to Products;
  quantity : Integer;
}
```
   

#### Composições para Relações Many-to-Many

- **Composições Gerenciadas** são úteis para relações many-to-many.
```cds
entity Teams {
  members : Composition of many { key user: Association to Users; }
}
entity Users {
  teams: Association to many Teams.members on teams.user = $self;
}
```

#### Publicação de Associações em Projeções
- **Associações podem ser publicadas** em vistas ou projeções.
```cds
entity P_Employees as projection on Employees {
  ID,
  addresses
}
```

#### Publicação com Filtro (beta)
- **Associações publicadas com filtro** adicionam uma condição de filtro.
```cds
entity P_Authors as projection on Authors {
  *,
  books[stock > 0] as availableBooks
};
```

### Anotações em CDS

#### Sintaxe de Anotações
- **Anotações** são prefixadas com `@` e podem ser colocadas em várias posições.
```cds
@before entity Foo @inner {
  @before simpleElement @inner : String @after;
}
```

#### Alvos de Anotação
- **Podem ser aplicadas** a qualquer coisa nomeada no modelo CDS.
```cds
@before define entity Foo @inner { ... }
```

#### Valores de Anotação
- **Podem ser literais, referências ou expressões.**
```cds
@aBoolean: false
@aString: 'foo'
@anInteger: 11
```

#### Propagação de Anotações
- **Anotações são herdadas** de tipos e tipos base para tipos derivados, entidades e elementos, bem como de elementos de entidades subjacentes em caso de vistas.
```cds
entity SomeView as select from Employees {
  ID,
  name
};
```

#### Diretiva `annotate`
- **Usada para anotar definições já existentes** que podem ter sido importadas de outros arquivos ou projetos.
```cds
annotate Foo with @title:'Foo' {
  nestedStructField {
    existingField @title:'Nested Field';
  }
}
```

#### Extensão de Anotações em Arrays
- **Sintaxe de reticências** permite inserir novos valores antes ou depois das entradas existentes.
```cds
annotate Foo with @anArray: [1, 2, ...];
annotate Foo with @anArray: [..., 5, 6];
```

### Aspectos em CDS

#### Diretiva `extend`
- **Usada para adicionar novos campos** ou adicionar/sobrescrever metadados em definições existentes.
```cds
extend Foo with {
  newField : String;
}
```

#### Atalhos com `:`
- **Sintaxe semelhante a herança** para estender uma definição com um ou mais aspectos nomeados.
```cds
define entity Foo : ManagedObject, AnotherAspect {
  key ID : Integer;
  name : String;
}
```

### Serviços em CDS

#### Definição de Serviços
- **Definem interfaces de serviço** como coleções de entidades expostas.
```cds
service SomeService {
  entity SomeExposedEntity { ... };
}
```

#### Ações e Funções Personalizadas
- **Podem ser especificadas** dentro de definições de serviço.
```cds
service MyOrders {
  action cancelOrder ( orderID:Integer, reason:String ) returns cancelOrderRet;
}
```

#### Extensão de Serviços

- **Podem ser estendidos** com entidades e ações adicionais.
```cds
extend service CatalogService with {
  entity Foo {};
}
```

#### Resumo final
Este artigo cobre os principais conceitos de associações, composições, anotações, aspectos e serviços em CDS, proporcionando uma visão abrangente das capacidades e usos práticos dessas definições.
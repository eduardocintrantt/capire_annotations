# Providing Services
#### Paradigma Centrado em Serviços

- **Serviços**: São a base de uma aplicação CAP, definidos em modelos CDS e gerenciados pelos runtimes do CAP. Tudo no CAP é um serviço, que representa aspectos comportamentais de um domínio através de entidades, ações e eventos expostos.

#### Eventos Ubíquos

- **Eventos**: No CAP, todas as ações são respostas a eventos, seja por APIs síncronas ou mensagens assíncronas. Consumidores enviam eventos para os serviços, que possuem hooks para que handlers (manipuladores de eventos) possam reagir.

#### Definições de Serviço

- **Definição Básica**: Uma definição de serviço declara entidades de dados e operações que serve. Exemplo:
```cds
service BookshopService {
  entity Books { key ID : UUID; title : String; author : Association to Authors; }
  entity Authors { key ID : UUID; name : String; books : Association to many Books on books.author = $self; }
  action submitOrder (book : Books:ID, quantity : Integer);
}
```
- **Serviços como Facades**: Serviços geralmente expõem visões (projeções) sobre entidades do modelo de domínio:
```cds
using { sap.capire.bookshop as my } from '../db/schema';
service BookshopService {
  entity Books as projection on my.Books;
  entity Authors as projection on my.Authors;
  action submitOrder (book : Books:ID, quantity : Integer);
}
```
#### Visões Desnormalizadas

- **Exemplo**: Uma definição de serviço pode omitir certas informações e marcar entidades como @readonly para os usuários finais:
```cds
using { sap.capire.bookshop as my } from '../db/schema';
service CatalogService @(path:'/browse') {
  @readonly entity ListOfBooks as projection on Books excluding { descr };
  @readonly entity Books as projection on my.Books { *, author.name as author } excluding { createdBy, modifiedBy };
}
```
#### Entidades Auto-Expostas

- **Auto-Exposição**: Anote entidades com @cds.autoexpose para incluí-las automaticamente em serviços que contêm entidades com associações para elas:
```cds
service Zoo {
  entity Foo { code : Association to SomeCodeList; }
}
@cds.autoexpose entity SomeCodeList {...}
```
#### Associações Redirecionadas

- **Exemplo**: Associações são automaticamente redirecionadas para garantir a navegação correta entre entidades projetadas:
```cds
service AdminService {
  entity Books as projection on my.Books;
  entity Authors as projection on my.Authors;
}
```
#### Provedores Genéricos

- **Runtimes**: Os runtimes do CAP para Node.js e Java fornecem implementações genéricas que atendem à maioria das solicitações automaticamente, como busca, paginação e validação de entrada.

#### Servindo Requisições CRUD

- **Operações CRUD**: Handlers genéricos nos runtimes do CAP servem automaticamente todas as solicitações CRUD para entidades modeladas em CDS:
    - **GET**: Leitura de entidades únicas ou conjuntos de entidades
    - **POST**: Criação de novas entidades
    - **PUT/PATCH**: Atualização de entidades
    - **DELETE**: Exclusão de entidades

#### Leitura e Escrita Profundas

- **Leitura Profunda**: Leitura de documentos aninhados usando expansões:
```http
GET .../Orders?$expand=header($expand=items)
```
- **Inserção Profunda**: Criação de entidade pai com filhos em uma única operação:
```http
POST .../Orders { ID:1, title: 'new order', header: { ID:2, status: 'open', items: [{ ID:3, description: 'child of child entity' }] } }
```   
- **Atualização Profunda**: Atualização de documentos aninhados:
```http
PUT .../Orders/1 { title: 'changed title', header: { ID:2, items: [{ ID:3, description: 'modified child' }] } }
```
- **Exclusão Profunda**: Exclusão em cascata de todas as entidades aninhadas.

#### Chaves Geradas Automaticamente

- **Exemplo**: Ao criar uma nova ordem com itens aninhados, o CAP preenche automaticamente as chaves UUID:
```js
POST .../Orders { title: 'Order #1', Items: [ { pos:1, descr: 'Item #1' }, { pos:2, descr: 'Item #2' } ] }
```

#### Busca de Dados

- **Busca Avançada**: Suporte para busca de texto em todos os elementos textuais de uma entidade, incluindo entidades aninhadas:
```js
@cds.search: { title } entity Books { ... }
```
- **Anotação @cds.search**: Personalize os elementos pesquisáveis:
```cds
@cds.search: { title } entity Books { ... }
```

#### Paginação e Ordenação

- **Paginação Implícita**: Respostas de leitura são truncadas a 1.000 registros por padrão, com links para páginas seguintes:
 ```http
 GET .../Books
> { value: [...], @odata.nextLink: "Books?$skiptoken=1000" }
```
- **Paginação Confiável**: Evita duplicação ou falta de linhas ao usar um token de salto baseado no valor da última linha da página.

#### Controle de Concorrência

- **Controle Otimista**: Usa ETags para detectar modificações concorrentes de dados:
```cds
using { managed } from '@sap/cds/common';
entity Foo : managed {...}
annotate Foo with { modifiedAt @odata.etag }
```
- **Bloqueio Pessimista**: Permite bloquear registros selecionados para impedir modificações concorrentes durante uma transação.

**Validação de Entrada**

**Anotações de Validação Automática no CAP**:

- @readonly: Elementos anotados com @readonly e elementos calculados são protegidos contra operações de escrita em operações de criação ou atualização.
- @mandatory: Elementos marcados como @mandatory devem ter entradas não vazias, rejeitando nulos e strings vazias.
- @assert.unique: Garante a unicidade de combinações de elementos especificadas em operações de criação e atualização.
- @assert.target: Verifica se a entidade alvo referenciada existe, aplicável em associações de um para um.
- @assert.format: Permite especificar uma expressão regular que todas as entradas de string devem corresponder.
- @assert.range: Define intervalos (mínimo, máximo) para tipos numéricos ou de data/tempo, ou restringe a valores enumerados.
- @assert.notNull: Ignora a verificação de não nulo para propriedades específicas.

**Lógica Personalizada**:

- Implementações de serviços customizadas podem ser adicionadas em arquivos .js ou classes de manipuladores de eventos em Java.
- **Manipuladores de Eventos**: Podem ser registrados com `on`, `before` e `after` para lidar com eventos CRUD ou operações personalizadas.

**Ações e Funções**:

- **Ações**: Modificam dados no servidor.
- **Funções**: Recuperam dados.
- Podem ser **vinculadas** (recebem a chave primária da entidade como primeiro argumento implícito) ou **não vinculadas**.

**Mídia e Dados Binários**:

- Anotações para indicar que um elemento contém dados de mídia: `@Core.MediaType`, `@Core.IsMediaType`, `@Core.IsURL`, `@Core.ContentDisposition.Filename`, `@Core.ContentDisposition.Type`.
- Suporte para leitura, criação, atualização e exclusão de recursos de mídia através de requisições HTTP apropriadas.

**Boas Práticas**:

- **Serviços de Propósito Único**: Projete serviços para casos de uso específicos, evitando expor todas as entidades de forma 1:1.
- **Microserviços Tardios**: Comece com um monólito bem estruturado, podendo dividir em microserviços posteriormente, conforme necessário, sem mudar modelos ou código.
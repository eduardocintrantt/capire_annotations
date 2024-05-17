# CDS-based Authorization
## Authorization
Authorization em CAP refere-se a restringir o acesso aos dados através de declarações nos modelos CDS, que são então aplicadas nas implementações de serviço. Ao adicionar tais declarações, revogamos todo o acesso padrão e concedemos privilégios individuais.

### Autenticação como Pré-requisito
A autenticação verifica a identidade do usuário e as permissões apresentadas, como papéis e associação ao locatário. Autenticação revela quem usa o serviço, enquanto a autorização controla como o usuário pode interagir com os recursos da aplicação.

CAP permite métodos de autenticação personalizáveis. Métodos suportados incluem:

- **XSUAA (User and Authentication and Authorization service)**: servidor OAuth 2.0 que protege endpoints em ambientes produtivos.
- **IAS (Identity Authentication Service)**: serviço OpenId Connect para gerenciamento de identidade.
- **Autenticação mock**: para desenvolvimento local e cenários de teste.

### Definindo Serviços Internos
Serviços CDS destinados apenas para uso interno não devem ser expostos via adaptadores de protocolo. Use a anotação `@protocol: 'none'`:
```cds
@protocol: 'none'
service InternalService { ... }
```
### Declarações de Usuário
A autorização no CDS é orientada por modelo, vinculando regras de acesso aos elementos do modelo CDS às declarações de usuário.

Após a autenticação bem-sucedida, um usuário é representado pelas seguintes propriedades:

- Nome exclusivo (login) identificando o usuário.
- Locatário para aplicações multi-locatário.
- Papéis atribuídos ao usuário.
- Atributos atribuídos ao usuário.

Algumas dessas propriedades podem ser referenciadas com o prefixo `$user` no modelo CDS.

### Papéis de Usuário
Papéis conceituais específicos da aplicação refletem como um usuário pode interagir com a aplicação. Exemplo: o papel Vendor permite ler artigos de vendas e atualizar números de vendas, enquanto ProcurementManager pode ter acesso total.

### Papéis Pseudo
Definem regras de acesso baseadas no nível de autenticação da solicitação:

- **authenticated-user**: usuários autenticados.
- **system-user**: usuário técnico usado para comunicação técnica.
- **internal-user**: comunicação interna da aplicação.
- **any**: todos os usuários, incluindo anônimos.

### Mapeamento de Declarações de Usuário
Dependendo da estratégia de autenticação configurada, CAP deriva um conjunto padrão de declarações de usuário contendo o nome do usuário, locatário e atributos. É possível personalizar o mapeamento conforme necessário.

### Restrições
CDS services não possuem controle de acesso por padrão. Para proteger recursos, defina restrições que façam com que o runtime imponha controle de acesso apropriado.

As restrições podem ser definidas em diferentes recursos do CDS:

- **Serviços**
- **Entidades**
- **Ações e funções vinculadas/não vinculadas**

A restrição pode limitar o acesso com base no evento da solicitação, papéis do usuário e condição de filtro em instâncias específicas.

### @readonly e @insertonly
Anote entidades com `@readonly` ou `@insertonly` para restringir operações permitidas para todos os usuários:
```cds
service BookshopService {
  @readonly entity Books { ... }
  @insertonly entity Orders { ... }
}
```
### Eventos para Entidades Auto-Expostas
Entidades auto-expostas pelo compilador CDS precisam ser controladas especificamente. Entidades implicitamente auto-expostas não podem ser acessadas diretamente, apenas via caminho de navegação.

### @requires
Use a anotação `@requires` para controlar quais papéis de usuário são necessários para acessar um recurso:
```cds
annotate BrowseBooksService with @(requires: 'authenticated-user');
annotate ShopService.Books with @(requires: ['Vendor', 'ProcurementManager']);
annotate ShopService.ReplicationAction with @(requires: 'system-user');
```
### @restrict
Use a anotação `@restrict` para definir autorizações detalhadas. Um privilégio é cumprido se todas as suas propriedades forem atendidas para a solicitação atual:
```cds
entity Orders @(restrict: [
    { grant: 'READ', to: 'Auditor', where: 'AuditBy = $user' }
]) { ... }
```
### Combinações Suportadas com Recursos CDS
Restrições podem ser definidas em diferentes tipos de recursos do CDS, mas há algumas limitações com relação aos privilégios suportados:

|Recurso CDS|grant|to|where|Observação|
|---|---|---|---|---|
|service|n/a|✓|n/a|= @requires|
|entity|✓|✓|✓||
|action/function|n/a|✓|n/a¹|= @requires|
|¹ Node.js suporta expressões estáticas sem referência ao modelo, como where: $user.level = 2.|||||

### Autorização Baseada em Instância
A condição definida na cláusula `where` filtra o conjunto de resultados em consultas ou aceita apenas operações de escrita em instâncias que atendem à condição.

### Modos de Rascunho
Regras de acesso diferem para entidades em modo de rascunho. Um usuário que criou um rascunho pode editá-lo ou cancelá-lo.

### Entidades Auto-Expostas e Geradas
Entidades auto-expostas pelo compilador ou geradas para suporte a localização ou rascunhos herdam a autorização da última entidade no caminho da solicitação que possui informação de autorização.

### Herança de Restrições
Entidades de serviço herdam a restrição da entidade de banco de dados na qual definem uma projeção. Restrições definidas explicitamente em uma entidade de serviço substituem as herdadas.

### Instâncias Baseadas em Autorização
A condição definida na cláusula `where` de uma restrição permite a autorização baseada em instâncias, associando dados do domínio com declarações de usuário estáticas.

Por exemplo:
```cds
annotate Orders with @(restrict: [
  { grant: ['READ', 'UPDATE', 'DELETE'], where: 'CreatedBy = $user' }
]);
```

### User Attribute Values

Para referenciar valores de atributos da declaração do usuário, prefixe o nome do atributo com `$user.`. Por exemplo, `$user.country` refere-se ao atributo `country`.

- `$user.<attribute>` contém uma lista de valores de atributos atribuídos ao usuário.
- Uma cláusula `where` avalia como `true` se um dos valores da lista atender à condição.
- Uma lista vazia ou não definida significa que o usuário está totalmente restrito em relação a esse atributo.

**Exemplo:**
```cds
where: $user.country = countryCode
```
Se um usuário tem `country = ['DE', 'FR']`, ele terá acesso a instâncias com `countryCode = DE` ou `FR`. Se a lista estiver vazia ou o atributo não existir, o usuário não terá acesso.

### Unrestricted XSUAA Attributes
Para oferecer atributos não restritos, configure `valueRequired:false` no XSUAA e ajuste a condição de filtro:
```cds
entity SalesOrgs @(restrict: [
  { grant: '*', to: ['SalesAdmin', 'SalesManager'], where: '$user.country = countryCode or $user.country is null' }
])
```
**Forma recomendada:**
```cds
entity SalesOrgs @(restrict: [
  { grant: '*', to: 'SalesManager', where: '$user.country = countryCode' },
  { grant: '*', to: 'SalesAdmin' }
])
```
### Exists Predicate

Use o predicado `exists` em condições `where` para definir filtros que se aplicam a entidades associadas:
```cds
entity Projects @(restrict: [
  { grant: ['READ', 'WRITE'], where: 'exists members[userId = $user and role = `Editor`]' }
])
```
**Recursos suportados do predicado `exists`:**

- Combine com outros predicados.
- Defina recursivamente.
- Use caminhos de destino.
- Uso de atributos do usuário.

### Association Paths
A condição `where` em uma restrição pode conter expressões de caminho CQL que navegam para elementos de entidades associadas:
```cds
entity SalesOrders @(restrict: [
  { grant: 'READ', where: 'product.productType = $user.productType' }
])
```
**Recomenda-se usar o predicado `exists` para associações 1:n.**

### Best Practices
**Escolha Papéis Conceituais:** Defina papéis que descrevem como um usuário interage com o sistema, como `Vendor`, `Customer`, ou `Accountant`.

**Prefira Serviços Específicos para Cada Caso de Uso:** Defina serviços dedicados para cada papel para evitar complexidade e confusão.
```cds
@path:'browse'
service CatalogService @(requires: 'authenticated-user') {
  @readonly entity Books as select from db.Books { title, publisher, price };
}

@path:'internal'
service VendorService @(requires: 'Vendor') {
  entity Books @(restrict: [
    { grant: 'READ' },
    { grant: 'WRITE', to: 'vendor', where: '$user.publishers = publisher' }
  ]) as projection on db.Books;
}

@path:'internal'
service AccountantService @(requires: 'Accountant') {
  @readonly entity Books as projection on db.Books;
  action doAccounting();
}
```
**Use Ações Dedicadas para Casos de Uso Específicos:** Crie ações com restrições dedicadas para casos de uso específicos.
```cds
service GitHubRepositoryService @(requires: 'authenticated-user') {
  @readonly entity Organizations as projection on GitHub.Organizations actions {
    action rename @(requires: 'Admin') (newName : String);
    action delete @(requires: 'Admin') ();
  };
}
```
**Pense em Autorização Orientada a Domínio:** Derive regras de acesso do domínio de negócios ao invés de papéis estáticos.

**Controle a Exposição de Associações e Composições:** Associações expostas podem divulgar dados não autorizados. Remova navegações indesejadas:
```cds
service BrowseEmployeesService @(requires:'Employee') {
  @readonly entity Employees as projection on db.Employees excluding { contracts };
  @readonly entity Teams as projection on db.Teams;
}
```
**Projete Modelos de Autorização Desde o Início:** Considere o design de segurança no início do projeto para evitar reescritas extensivas.

**Mantenha a Simplicidade:** Defina autorizações de forma simples e evite misturar variantes de restrições para reduzir complexidade.

**Separation of Concerns:** Separe definições de serviços das anotações de autorização usando CDS Aspects:
```cds
// services.cds
service ReviewsService {
  /*...*/
}

service CustomerService {
  entity Orders {/*...*/}
  entity Approval {/*...*/}
}

// services-auth.cds
service ReviewsService @(requires: 'authenticated-user'){
  /*...*/
}

service CustomerService @(requires: 'authenticated-user'){
  entity Orders @(restrict: [
    { grant: ['READ','WRITE'], to: 'admin' },
    { grant: 'READ', where: 'buyer = $user' },
  ]){/*...*/}
  entity Approval @(restrict: [
    { grant: 'WRITE', where: '$user.level > 2' }
  ]){/*...*/}
}
```
**Enforcement Programático:** Se necessário, substitua ou adapte a aplicação genérica com enforcement programático em handlers personalizados.

**Atribuições de Papéis com XSUAA:** Gerencie papéis e atributos no serviço da plataforma UAA. Use o comando `cds add xsuaa` para gerar `xs-security.json` com scopes e templates de papéis.
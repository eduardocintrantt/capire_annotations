## Domain Modelling

### Conceituação do Modelo

CDS implementa a modelagem conceitual focando no "o que" e não no "como", capturando a intenção do modelo e não a implementação.

### Modelagem de Entidade-Relacionamento (ERM)

A modelagem de entidade-relacionamento (ERM) define o modelo de negócio criando modelos que se relacionam entre si, formando o domínio completo.

### Modelagem Orientada a Aspecto

A modelagem orientada a aspecto separa preocupações, mantendo o núcleo do domínio limpo e colocando preocupações secundárias em arquivos e modelos separados.

### Alimentando Provedores Genéricos

Criação de modelos específicos para cada situação necessária, como:

- UI Models → Frontends
- Service Models → Serviços (repositórios)
- Domain Models → Bancos de dados

### Domain-Driven Design (DDD)

CAP usa conceitos e regras que se aproximam do DDD, utilizando CDS como linguagem de modelo onipresente e separando aspectos do domínio dos aspectos genéricos, facilitando a colaboração entre desenvolvedores e especialistas do domínio.

## Best Practices

### Modelagem de Domínio

- Modelagem de domínio deve fazer sentido para clientes e usuários finais, não apenas para desenvolvedores.
- Mantenha os modelos concisos e compreensíveis.
- Evite modelos excessivamente abstratos; encontre um equilíbrio.

### Preferência por Modelos Planos (Flat Models)

Utilize estruturas padrão como string, boolean, double, ao invés de criar classes customizadas dentro de outras classes.

### Separação de Preocupações

Mantenha o núcleo do domínio limpo, conciso e compreensível, utilizando Aspects do CDS para decompor modelos e definições em arquivos separados.

### Convenções de Nomenclatura

- Tipos/Entidades: Comece com letra maiúscula (ex: Authors).
- Elementos (propriedades): Comece com letra minúscula (ex: name).
- Use plural para entidades e singular para tipos.

### Namespaces

Utilize namespaces para ter nomes únicos, evitando inflar o código com nomes mais específicos. Use namespaces se os modelos puderem ser usados em outros projetos.

### Domain Entities

Entidades representam dados do domínio e, quando persistidas, são convertidas em tabelas.

### Views / Projections

Entidades podem ser declaradas como views do SQL:
```js
entity ProjectedEntity as select from BaseEntity { element1, element2 as name }
```
### Primary Keys

- Utilize a keyword `key` para definir uma ou mais primary keys.
- Prefira UUIDs para primary keys ao invés de IDs sequenciais.
- Não use dados binários como keys.

### Data Types

#### Standard Built-in Types

- CDS possui tipos nativos como UUID, Boolean, Date, Time, DateTime, Timestamp, Integer, Double, Decimal, String, LargeString, Binary, LargeBinary.

#### Common Reuse Types

- Tipos de reuso como Country, Currency e Language, além de Aspects como cuid, managed e temporal.

### Custom-defined Types

Tipos customizados são úteis para reutilização dentro do domínio, mas use-os com moderação.

## Associations

### Managed :1 Associations

Associações gerenciadas definem automaticamente as condições de foreign key.

### To-Many Associations

Especifique associações de um-para-muitos com a keyword `many` e a condição `on`.

### Many-to-Many Associations

Use dois relacionamentos um-para-muitos e uma tabela de ligação para representar muitos-para-muitos.

### Compositions

Compositions representam relações internas e CAP runtime provê suporte para Deep Insert/Update e Cascaded/Delete.

## Aspects
### Authorization

Utilize anotações de autorização (`@requires`, `@restrict`) para controlar acessos e mantenha essas anotações separadas do núcleo do domínio.

### Fiori Annotations

Use anotações específicas para UI, como `@title`, `@UI`, `@Common`, evitando poluir o domínio.

### Localized Data

Marque campos que precisam de localização com `localized` para suportar traduções.

## Managed Data

### Annotations `@cds.on.insert` e `@cds.on.update`

Use essas anotações para gerar automaticamente valores em inserções ou atualizações, mantendo o domínio limpo e compreensível.

### Aspect `managed`

Use o aspect `managed` do `@sap/cds/common` para definir campos gerenciados automaticamente.

## Pseudo Variables

Pseudo variáveis como `$now`, `$user`, `$uuid` são substituídas por valores atualizados em tempo de execução, como a hora atual ou o usuário logado.
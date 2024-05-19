### Tipos e Aspects Comuns (@sap/cds/common)

O CDS fornece um modelo predefinido, @sap/cds/common, que inclui tipos e aspectos comuns para reutilização. Ele é recomendado para todas as aplicações devido às suas vantagens, como modelos concisos, interoperabilidade, práticas recomendadas e suporte otimizado para listas de códigos e dados localizados.

#### Benefícios do Uso de @sap/cds/common

- **Modelos Concisos e Compreensíveis**: Facilita a criação de modelos de dados.
- **Interoperabilidade**: Promove a compatibilidade entre diferentes aplicações.
- **Práticas Recomendadas**: Baseado em experiências reais de aplicações.
- **Modelos Otimizados**: Estruturas de persistência eficientes e desempenho aprimorado.
- **Suporte Automático**: Inclui suporte para listas de códigos localizadas e ajuda de valor.
- **Extensibilidade**: Permite extensões através de Aspectos.
- **Verticalização**: Suporte a pacotes de extensão de terceiros.

Exemplo de uso:
```cds
using { Country } from '@sap/cds/common';
entity Addresses {
  street  : String;
  town    : String;
  country : Country; //> usando tipo reutilizável
}
```

### Aspects Comuns de Reutilização

#### Aspect `cuid`
Adiciona chaves primárias únicas universalmente (UUID).
```cds
entity Foo : cuid {...}
```

#### Aspect `managed`
Adiciona campos para capturar informações de criação e modificação de registros.
```cds
entity Foo : managed {...}
```

#### Aspect `temporal`
Adiciona os elementos `validFrom` e `validTo`, facilitando o gerenciamento de dados temporais.
```cds
entity Contract : temporal {...}
```

### Tipos de Reutilização Comuns
- **Country**: Associação gerenciada à lista de códigos de países.
- **Currency**: Associação gerenciada à lista de códigos de moedas.
- **Language**: Associação gerenciada à lista de códigos de idiomas.
- **Timezone**: Associação gerenciada à lista de fusos horários.

### Listas de Códigos Comuns
As listas de códigos para países, moedas e idiomas são definidas como associações gerenciadas às entidades `Countries`, `Currencies` e `Languages` respectivamente.

#### Namespace: `sap.common`
Definições dentro do namespace `sap.common` incluem:

- **Aspect CodeList**: Base para entidades de listas de códigos.
```cds
aspect sap.common.CodeList {
  name  : localized String(111);
  descr : localized String(1111);
}
```
    
- **Entidade Countries**: Lista de códigos para países.
```cds
entity sap.common.Countries : CodeList {
  key code : String(3); //> ISO 3166-1 alpha-2 ou alpha-3
}
```
    
- **Entidade Currencies**: Lista de códigos para moedas.
```cds
entity sap.common.Currencies : CodeList {
  key code  : String(3); //> ISO 4217 alpha-3
  symbol    : String(5); //> Símbolo da moeda
  minorUnit : Int16;     //> Unidade menor, ex: 0 ou 2
}
```
    
- **Entidade Languages**: Lista de códigos para idiomas.
```cds
entity sap.common.Languages : CodeList {
  key code : sap.common.Locale; //> Ex: en_GB
}
```
    
- **Entidade Timezones**: Lista de códigos para fusos horários.
```cds
entity sap.common.Timezones : CodeList {
  key code : String(100); //> Ex: Europe/Berlin
}
```
    

### Design Minimalista
Os modelos de listas de códigos são intencionalmente minimalistas para manter a simplicidade e eficiência, focando nos códigos únicos e campos localizáveis para nome e descrição completa.

### Adaptação às Necessidades
Os modelos pré-definidos são minimalistas, mas podem ser adaptados e estendidos conforme necessário usando os recursos padrão do CDS, como os Aspectos.

Exemplo de adição de campos detalhados conforme ISO 3166-1:
```cds
using { sap.common.Countries } from '@sap/cds/common';
extend Countries {
  numcode : Integer; //> ISO 3166-1 três dígitos numéricos
  alpha3  : String(3); //> ISO 3166-1 três letras alfa
}
```

### Provisão de Dados Iniciais
Você pode fornecer dados iniciais para listas de códigos usando arquivos CSV no diretório `data` ao lado dos modelos de dados.

Exemplo de arquivo CSV para países:
```csv
code;name;descr
AU;Australia;Commonwealth of Australia
CA;Canada;Canada
...
```

### Pacotes de Conteúdo Pré-Construídos
O pacote `@sap/cds-common-content` fornece dados pré-construídos para as entidades `Countries`, `Currencies`, `Languages` e `Timezones`.
```sh
npm add @sap/cds-common-content --save
```

Usar no arquivo CDS:
```cds
using from '@sap/cds-common-content';
```

O uso de `@sap/cds/common` simplifica a modelagem de dados, promove interoperabilidade e permite a reutilização de práticas recomendadas, facilitando a criação de aplicações robustas e eficientes.

### Aspectos vs. Herança
A sintaxe baseada em `:` no CDL parece muito com herança múltipla, mas na verdade é baseada em mixins, que são mais poderosos e evitam problemas comuns como o "diamond problem" em derivações de tipo.

#### Modelo de Exemplo Usando Aspectos
Vamos examinar um exemplo de mapeamento para modelos relacionais, baseado em cenários hipotéticos de IoT:
```cds
abstract entity Thing {
  key ID : UUID;
  title : String(100);
  description : String;
}

entity Vehicle : Thing {
  wheels : Integer @description: 'Number of Wheels';
  manufacturer : String(255);
}

entity Bike : Vehicle { }

entity Car : Vehicle {
  engine : String(255);
}

entity Asset : Thing {
  location : String;
}
```

#### Mapeamento para SQL
Usando o mapeamento canônico do CDS, o exemplo acima resultaria em tabelas SQL como estas:
```sql
CREATE TABLE Vehicle (
  ID NVARCHAR(36) NOT NULL,
  title NVARCHAR(100),
  description NVARCHAR(5000),
  wheels INTEGER,
  manufacturer NVARCHAR(255),
  PRIMARY KEY(ID)
);

CREATE TABLE Bike (
  ID NVARCHAR(36) NOT NULL,
  title NVARCHAR(100),
  description NVARCHAR(5000),
  wheels INTEGER,
  manufacturer NVARCHAR(255),
  PRIMARY KEY(ID)
);

CREATE TABLE Car (
  ID NVARCHAR(36) NOT NULL,
  title NVARCHAR(100),
  description NVARCHAR(5000),
  wheels INTEGER,
  manufacturer NVARCHAR(255),
  engine NVARCHAR(255),
  PRIMARY KEY(ID)
);

CREATE TABLE Asset (
  ID NVARCHAR(36) NOT NULL,
  title NVARCHAR(100),
  description NVARCHAR(5000),
  location NVARCHAR(5000),
  PRIMARY KEY(ID)
);
```

Isso significa que os mixins são colunas clonadas de forma idêntica em cada entidade (folha), permitindo aplicar funções comuns de reutilização definidas em `Thing` ou `Vehicle` a todas as instâncias de todas as 'subclasses'. No entanto, para consultar todos os veículos, seria necessário um `UNION` entre `Vehicle`, `Bike` e `Car`.

#### Mapeamento Alternativo
Para otimizar consultas entre subclasses, pode-se usar uma estratégia de tabela única por nó:
```cds
entity Thing {
  key ID : UUID;
  title : String(100);
  description : String;
}

entity Vehicle {
  key thing : Association to Thing;
  wheels : Integer @description: 'Number of Wheels';
  manufacturer : String(255);
}

entity Bike {
  key vehicle : Association to Vehicle;
}

entity Car {
  key vehicle : Association to Vehicle;
  engine : String(255);
}

entity Asset {
  key thing : Association to Thing;
  location : String;
}
```

As tabelas SQL correspondentes para `Vehicle` e `Car` seriam:

```sql
CREATE TABLE Vehicle (
  wheels INTEGER,
  manufacturer NVARCHAR(255),
  thing_ID NVARCHAR(36) NOT NULL,
  PRIMARY KEY(thing_ID)
);

CREATE TABLE Car (
  engine NVARCHAR(255),
  vehicle_thing_ID NVARCHAR(36) NOT NULL,
  PRIMARY KEY(vehicle_thing_ID)
);
```

Para uma abordagem mais parecida com herança:

```cds
entity ThingHeader {
  key ID : UUID;
  title : String(100);
  description : String;
}

abstract entity Thing {
  key thing : Association to ThingHeader;
}

entity Vehicle : Thing {
  wheels : Integer @description: 'Number of Wheels';
  manufacturer : String(255);
}

entity Asset : Thing {
  location : String;
}
```

O SQL correspondente seria:

```sql
CREATE TABLE ThingHeader (
  ID NVARCHAR(36) NOT NULL,
  title NVARCHAR(100),
  description NVARCHAR(5000),
  PRIMARY KEY(ID)
);

CREATE TABLE Vehicle (
  wheels INTEGER,
  manufacturer NVARCHAR(255),
  thing_ID NVARCHAR(36) NOT NULL,
  PRIMARY KEY(thing_ID)
);
```

Não há uma estratégia única de mapeamento de gráficos de herança para bancos de dados. A escolha depende das características esperadas de acesso aos dados e do tipo de banco de dados (por exemplo, NoSQL vs SQL). Portanto, o CDS não impõe uma abordagem única e você deve modelar explicitamente o que deseja.
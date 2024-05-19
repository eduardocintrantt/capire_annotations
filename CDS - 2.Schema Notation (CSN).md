**CSN** (pronunciado como "Season") é uma notação para representações compactas de modelos CDS, otimizada para compartilhamento e interpretação de modelos com mínimo impacto e dependências. Semelhante ao JSON Schema, mas vai além, capturando modelos completos de entidade-relacionamento e extensões, ideal para gerar modelos de destino como interfaces OData/EDM ou OpenAPI, além de modelos de persistência para bancos de dados SQL ou NoSQL.

#### Anatomia do CSN
Um modelo CSN em JSON:
```json
{
  "requires": ["@sap/cds/common", "./db/schema"],
  "definitions": {
    "some.type": { "type": "cds.String", "length": 11 },
    "another.type": { "type": "some.type" },
    "structured.type": { "elements": {
      "foo": { "type": "cds.Integer" },
      "bar": { "type": "cds.String" }
    }}
  },
  "extensions": [
    { "extend": "Foo", "elements": {
      "bar": { "type": "cds.String" }
    }}
  ]
}
```

O mesmo modelo em YAML:
```yaml
requires:
  - @sap/cds/common
  - ./db/schema
definitions:
  some.type: {type: cds.String, length: 11}
  another.type: {type: some.type }
  structured.type:
    elements:
      foo: {type: cds.Integer}
      bar: {type: cds.String}
extensions: [
  - extend: Foo
    elements:
      bar: {type: cds.String}
]
```

O mesmo modelo como um objeto JavaScript simples:
```js
({
  requires:[ '@sap/cds/common', './db/schema' ],
  definitions: {
    'some.type': { type:"cds.String", length:11 },
    'another.type': { type:"some.type" },
    'structured.type': { elements: {
      'foo': { type:"cds.Integer" },
      'bar': { type:"cds.String" }
    }}
  },
  extensions: [
    { extend:'Foo', elements:{
      'bar': { type:"cds.String" }
    }}
  ],
})
```

#### Propriedades do CSN
- **requires**: lista de modelos importados.
- **definitions**: dicionário de definições nomeadas.
- **extensions**: array de aspectos não nomeados.
- **i18n**: dicionário de dicionários de traduções de texto.

#### Literals
Lugares onde os literais aparecem em modelos:
- **Globais**: `true`, `false`, `null`
- **Números**: `11` ou `2.4`
- **Strings**: `"foo"`
- **Datas**: `"2016-11-24"`
- **Tempos**: `"16:11Z"`
- **DateTimes**: `"2016-11-24T16:11Z"`
- **Registros**: `{"foo":<literal>, ...}`
- **Arrays**: `[<literal>, ...]`
- **Expressões não parseadas**: `{"=":"foo.bar < 9"}`
- **Símbolos enum**: `{"#":"asc"}`

#### Definições
Cada entrada no dicionário de definições é essencialmente uma definição de tipo com propriedades como:
- **kind**: tipo do objeto (ex: `context`, `service`, `entity`, `type`, `action`, `function`, `annotation`).
- **type**: tipo base do qual a definição deriva.
- **elements**: dicionário de elementos em tipos estruturados.

#### Tipos Definidos pelo Usuário
Exemplo de tipos definidos pelo usuário:
```js
({
  definitions: {
    'scalar.type': {type:"cds.String", length:3 },
    'struct.type': {elements:{ 'foo': {type:"cds.Integer"}}},
    'arrayed.type': {items:{type:"cds.Integer"}},
    'enum.type': {enum:{ 'asc':{}, 'desc':{} }}
  }
})
```

#### Tipos Escalares
Tipicamente possuem a propriedade `type` especificada.
```js
({
  definitions: {
    'scalar.type': {type:"cds.String", length:3 },
  }
})
```

#### Tipos Estruturados
```js
({
  definitions: {
    'structured.type': {elements: {
      'foo': {type:"cds.Integer"},
      'bar': {type:"cds.String"}
    }}
  }
})
```

#### Tipos Enumerados
A propriedade `enum` é um dicionário de membros enum.
```js
({
  definitions: {
    'Gender': {enum: {
      'male':{},
      'female':{},
      'non_binary': {val: 'non-binary'}
    }},
    'Status': {enum: {
      'submitted': {val: 1},
      'fulfilled': {val: 2}
    }},
    'Rating': {type:"cds.Decimal", enum: {
      'low': {val: 0},
      'medium': {val: 50},
      'high': {val: 100}
    }}
  }
})
```

#### Definições de Entidade
Entidades são tipos estruturados com `kind = 'entity'`, onde um ou mais elementos têm a propriedade `key` definida como `true`.
```js
({
  definitions: {
    'Products': {kind:"entity", elements: {
      'ID': {type:"cds.Integer", key:true},
      'title': {type:"cds.String", notNull:true},
      'price': {type:"Amount", virtual:true},
    }}
  }
})
```

#### Definições de Visão
Visões são entidades definidas como projeções em entidades subjacentes, sinalizadas pela presença da propriedade `query`.
```js
({
  definitions: {
    'Foo': {kind:"entity", query: {
      SELECT: {
        from: {ref:['Bar']},
        columns: [{ref:['title']}, {ref:['price']}]
      }
    }}
  }
})
```

#### Associações
Associações são semelhantes a definições de tipo escalar com `type` sendo `cds.Association` ou `cds.Composition`, além de propriedades adicionais especificando o `target`.
```js
({
  definitions: {
    'Books': {kind:"entity", elements: {
      'author': {type:"cds.Association", target:"Authors"},
    }},
    'Authors': {kind:"entity", elements: {
      'books': {type:"cds.Association", target:"Books", cardinality:{max:"*"}},
    }}
  }
})
```

#### Anotações
Representadas como propriedades prefixadas com `@`.
```js
({
  definitions: {
    'Employees': {kind:"entity",
      '@title':"Mitarbeiter",
      '@readonly':true,
      elements: {
        'firstname': {type:"cds.String", '@title':"Vorname"},
        'surname': {type:"cds.String", '@title':"Nachname"},
      }
    }
  }
})
```

#### Serviços
Definidos com `kind = 'service'`.
```js
({
  definitions: {
    'MyOrders': {kind:"service"}
  }
})
```

#### Ações / Funções
Podem estar presentes em definições de entidade ou como definições de serviço de nível superior.
```js
({
  definitions: {
    'OrderService': {kind:"service"},
    'OrderService.Orders': {kind:"entity", elements:{}, actions: {
      'validate': {kind:"function", returns: {type: "cds.Boolean"}}
    }},
    'OrderService.cancelOrder': {kind:"action",
      params: {
        'orderID': {type:"cds.Integer"},
        'reason': {type:"cds.String"},
      },
      returns: {elements: {
        'ack': {enum:{ 'succeeded':{}, 'failed':{}}},
        'msg': {type:"cds.String"},
      }}
    }
  }
})
```

#### Imports
A propriedade `requires` lista outros modelos para importar definições.
```js
({
  requires: ['@sap/cds/common', './db/schema'],
  [...]
})
```

#### Traduções (`i18n`)
Pode conter traduções de textos.
```js
({
  "i18n": {
    "language-key": {
      "text-key": "some string"
    }
  }
})
```

#### Resumo
Este artigo abrange os principais conceitos e estruturas do CSN, destacando suas capacidades e usos práticos para a modelagem e geração de artefatos em diversos contextos de dados.
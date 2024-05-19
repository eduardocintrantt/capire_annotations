### A Natureza dos Modelos
Esta seção introduz os princípios fundamentais dos modelos CDS (Core Data Services).

#### Definição de Modelos em CDS
Os modelos em CDS são objetos JavaScript simples que seguem a Notação de Esquema Core (CSN). Eles podem ser:
- Analisados a partir de fontes `.cds`
- Lidos a partir de arquivos `.json` ou `.yaml`
- Criados dinamicamente em tempo de execução

#### Exemplos de Criação de Modelos
**Codificação Simples em Tempo de Execução**
```js
const cds = require('@sap/cds')

// define o modelo
var model = {
    definitions: {
        Products: {
            kind: 'entity', 
            elements: {
                ID: {type: 'Integer', key: true},
                title: {type: 'String', length: 11, localized: true},
                description: {type: 'String', localized: true},
            }
        },
        Orders: {
            kind: 'entity', 
            elements: {
                product: {type: 'Association', target: 'Products'},
                quantity: {type: 'Integer'},
            }
        }
    }
}

// faça algo com o modelo
console.log(cds.compile.to.yaml(model))
```

**Analisado em Tempo de Execução**
```js
const cds = require('@sap/cds')

// define o modelo
var model = cds.parse(`
    entity Products {
        key ID: Integer;
        title: localized String(11);
        description: localized String;
    }
    entity Orders {
        product: Association to Products;
        quantity: Integer;
    }
`)

// faça algo com o modelo
console.log(cds.compile.to.yaml(model))
```

**A Partir de Arquivos .cds**
```cds
// some.cds
entity Products {
    key ID: Integer;
    title: localized String(11);
    description: localized String;
}
entity Orders {
    product: Association to Products;
    quantity: Integer;
}
```

Para ler/analisar e fazer algo com o modelo:
```js
const cds = require('@sap/cds')
cds.get('./some.cds').then(cds.compile.to.yaml).then(console.log)
```

Ou usando a CLI: `cds ./some.cds -2 yaml`

**A Partir de Arquivos .json**
```json
{
    "definitions": {
        "Products": {
            "kind": "entity",
            "elements": {
                "ID": { "type": "Integer", "key": true },
                "title": { "type": "String", "length": 11, "localized": true },
                "description": { "type": "String", "localized": true }
            }
        },
        "Orders": {
            "kind": "entity",
            "elements": {
                "product": { "type": "Association", "target": "Products" },
                "quantity": { "type": "Integer" }
            }
        }
    }
}
```

Para ler/analisar e fazer algo com o modelo:
```js
const cds = require('@sap/cds')
cds.get('./some.json').then(cds.compile.to.yaml).then(console.log)
```

**A Partir de Outros Frontends**
Você pode adicionar qualquer outro frontend para gerar as estruturas CSN, facilmente como `.json`. Por exemplo:
- ABAP CDS para CSN
- OData EDMX para CSN
- Annotations Fiori XML para CSN
- Arquivos de propriedades i18n para CSN
- Modelos Java/JPA para CSN

#### Processamento de Modelos
Todos os passos de processamento e compilação de modelos que podem ser aplicados subsequentemente trabalham com base em objetos CSN simples. Não há suposição sobre ou bloqueio em um formato de origem específico.
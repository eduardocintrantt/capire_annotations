### Configuração e Setup

**Adicionando Pacotes de Banco de Dados**
- **Pacotes de plugins CDS para CAP Node.js**:
    - **SAP HANA Cloud**: `@sap/cds-hana` (recomendado para produção)
    - **SQLite**: `@cap-js/sqlite` (recomendado para desenvolvimento)
    - **PostgreSQL**: `@cap-js/postgres` (mantido pela comunidade + equipe CAP)
- **Instalação**:
- Para desenvolvimento com SQLite:
```sh
npm add @cap-js/sqlite -D
```
- Para produção com SAP HANA:
```sh
npm add @sap/cds-hana
```

**Configuração Automática**
- Os pacotes mencionados usam técnicas de plugin CDS para configurar automaticamente o banco de dados principal com `cds.env`.
- **Exemplo de Configuração**:
```json
{
  "cds": {
    "requires": {
      "db": {
        "[development]": { "kind": "sqlite", "impl": "@cap-js/sqlite", "credentials": { "url": "memory" } },
        "[production]": { "kind": "hana", "impl": "@sap/cds-hana", "deploy-format": "hdbtable" }
      }
    }
  }
}
```

**Configuração Personalizada**
- Você pode substituir propriedades individuais ou usar configurações básicas para outros setups.
- **Exemplo**:
```json
{
  "cds": {
    "requires": {
      "db": {
        "kind": "sqlite",
        "impl": "@cap-js/sqlite",
        "credentials": {
          "url": "db.sqlite"
        }
      }
    }
  }
}
```

**Fornecendo Dados Iniciais**
- **CSV Files**: Use arquivos CSV para preencher seu banco de dados com dados iniciais.
- **Estrutura de Arquivos**:
```sh
bookshop/
├─ db/
│ ├─ data/
│ │ ├─ sap.capire.bookshop-Authors.csv
│ │ ├─ sap.capire.bookshop-Books.csv
│ │ ├─ sap.capire.bookshop-Books.texts.csv
│ │ └─ sap.capire.bookshop-Genres.csv
│ └─ schema.cds
└─ ...
```
- **Conteúdo CSV**:
```csv
ID,title,author_ID,stock
201,Wuthering Heights,101,12
207,Jane Eyre,107,11
251,The Raven,150,333
252,Eleonora,150,555
271,Catweazle,170,22
```

**Consultas em Tempo de Execução**
- **Consultas Agnósticas ao Banco de Dados**:
```js
SELECT.from (Authors, a => {
  a.ID, a.name, a.books (b => {
    b.ID, b.title
  })
})
.where ({name:{like:'A%'}})
.orderBy ('name')
```
- **Consultas SQL Nativas**:
```js
cds.db.run (`SELECT from sqlite_schema where name like ?`, name)
```

**Gerando Arquivos DDL**

- Durante o desenvolvimento, um banco de dados em memória é iniciado automaticamente com declarações SQL DDL geradas com base nos modelos CDS.
- **Comando CLI**:
```sh
cds compile --to <dialeto>
```

**Regras para DDL Gerado**
- **Observações**:
    - Entidades declaradas tornam-se tabelas, entidades projetadas tornam-se views.
    - Tipos CDS são mapeados para tipos SQL específicos do banco de dados.
    - Nomes totalmente qualificados CDS são convertidos para nomes SQL com underscores.
    - Elementos estruturados são achatados.
    - Chaves estrangeiras são geradas para associações gerenciadas.

**Anotações para Refinar SQL Gerado**
- @cds.persistence.skip: Ignora a entidade da geração DDL.
- @cds.persistence.exists: Espera que uma relação de banco de dados exista para gerar views SQL.
- @cds.persistence.table: Cria uma tabela com a assinatura da definição da view.
- @sql.prepend / @sql.append: Adiciona cláusulas SQL nativas antes ou depois da saída SQL gerada.

**Restrições de Banco de Dados**

- **Geração de Restrições de Chave Estrangeira**:
```js
cds.features.assert_integrity = 'db'
```

**Usando Recursos Nativos**
- O compilador CDS para SQL não 'entende' funções SQL, mas as traduz genericamente, permitindo o uso de funções nativas do banco de dados em modelos CDS.

Essas seções cobrem a configuração e o setup no CAP, desde a adição de pacotes de banco de dados até a configuração personalizada, fornecimento de dados iniciais, consultas em tempo de execução, geração de arquivos DDL, e uso de recursos nativos do banco de dados.
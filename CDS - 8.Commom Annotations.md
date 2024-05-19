### Anotações Comuns no CDS

#### Propósito Geral
- **@title**: Rótulo comum.
- **@description**: Descrição comum.

#### Controle de Acesso
- @readonly: Propriedade somente leitura.
- @insertonly**: Propriedade de inserção apenas.
- @restrict: Controle de autorização.
- @requires: Dependências de autorização.

#### Validação de Entrada
- @readonly: Propriedade somente leitura.
- @mandatory: Campo obrigatório.
- @assert.unique: Assegura unicidade.
- @assert.integrity: Verifica integridade.
- @assert.target: Verifica o alvo.
- @assert.format: Verifica o formato.
- @assert.range: Verifica intervalo.
- @assert.notNull: Verifica não nulo.

#### Serviços / APIs

- @path: Definição de caminho.
- @impl: Implementação.
- @odata.etag: Suporte a ETags.
- @cds.autoexpose: Autoexposição.
- @cds.api.ignore: Ignora na API OData.
- @cds.query.limit: Limita consulta.
- @cds.localized: Dados localizados.
- @cds.valid.from/to: Dados temporais.
- @cds.search: Capacidade de busca.

#### Persistência

- @cds.persistence.exists: Já criado, não recriar.
- @cds.persistence.table: Cria tabela, não visão.
- @cds.persistence.skip: Não criar no banco de dados.
- @cds.persistence.mock: Exclui de mock automático.
- @cds.on.insert: Ação ao inserir.
- @cds.on.update: Ação ao atualizar.

#### OData

- @ValueList.entity: Lista de valores.
- @odata.Type: Tipo OData.
- @odata.MaxLength: Comprimento máximo.
- @odata.Precision: Precisão.
- @odata.Scale: Escala.
- @odata.singleton* Instância única.

#### Anotações Intrinsecamente Suportadas pelo OData

- @Core.Computed: Computado.
- @Core.Immutable: Imutável.
- @Core.MediaType: Tipo de mídia.
- @Core.IsMediaType: É tipo de mídia.
- @Core.IsUrl: É URL.
- @Capabilities: Capabilidades do Fiori.
- @Common.FieldControl: Controle de campo.
### Localização, i18n

Esta sessão vai orientar como internacionalizar sua aplicação para fornecer versões localizadas, abordando tanto Modelos Localizados quanto Dados Localizados.

**Externalizando Pacotes de Textos**
- **Passos para Internacionalizar**:
    - Externalize textos literais para pacotes de textos e use as chaves correspondentes como valores de anotação nos modelos.
    - **Exemplo**:
```cds
srv/my-service.1.cds
service Bookshop {
  entity Books @(
    UI.HeaderInfo: {
      Title.Label: '{i18n>Book}',
      TypeName: '{i18n>Book}',
      TypeNamePlural: '{i18n>Books}',
    },
  ){/*...*/}
}
```
```properties
_i18n/i18n.properties
Book = Book
Books = Books
foo = Foo
```

- **Bundles Localizados**:
    - Traduza os textos em pacotes localizados com um código de idioma/locale anexado ao nome, por exemplo:
```sh
_i18n/
  i18n.properties        # fallback padrão
  i18n_en.properties     # inglês
  i18n_de.properties     # alemão
  i18n_zh_TW.properties  # chinês tradicional
```    

**Localização dos Pacotes de Textos**
- Por padrão, os pacotes de textos podem ser colocados em pastas nomeadas `_i18n`, `i18n` ou `assets/i18n`.
    
- **Estrutura de Arquivos**:
```sh
srv/
   my-service.cds          # arquivo de modelo
   _i18n/i18n.properties   # próximo ao arquivo de modelo
_i18n/i18n.properties      # em uma pasta pai
```
- **Configuração no package.json**:
```json
"cds":{"i18n":{
  "folders": [ "_i18n", "i18n", "assets/i18n" ]
}}
```

**Pacotes de Textos Baseados em CSV**
- Projetos menores podem usar arquivos CSV, editáveis em Excel, Numbers, etc.
    - **Formato**:
```csv
key;en;de;zh_CN;...
Book;Book;Buch;...
Books;Books;Bücher;...
```

**Algoritmo de Mesclagem**
- **Construção do Modelo Localizado**:
    1. Bundle de fallback padrão (`i18n.properties`)
    2. Bundle de idioma padrão (`i18n_en.properties`)
    3. Bundle solicitado (`i18n_de.properties`)

**Mesclagem de Bundles de Reuso**

- Se a aplicação importa modelos de um pacote de reuso, esses pacotes vêm com seus próprios bundles de idioma, que são aplicados durante a importação e podem ser sobrescritos nos seus modelos e traduções.

**Determinando Locales do Usuário**

- A língua preferida do usuário é determinada a partir de:
    1. Parâmetro de URL `sap-locale`
    2. Parâmetro de URL `sap-language` (se for 1Q, 2Q ou 3Q)
    3. Primeiro item do cabeçalho `Accept-Language` da solicitação
    4. Idioma padrão configurado no nível da aplicação

**Locales Normalizados**

- Para reduzir o número de traduções necessárias, a maioria dos locales determinados são normalizados para seus códigos de idioma principal, exceto por alguns preservados.
    - **Locales Preservados**:
        - zh_CN (Chinês - China)
        - zh_HK (Chinês - Hong Kong, China)
        - zh_TW (Chinês tradicional - Taiwan, China)
        - en_GB (Inglês - Reino Unido)
        - fr_CA (Francês - Canadá)
        - pt_PT (Português - Portugal)
        - es_CO (Espanhol - Colômbia)
        - es_MX (Espanhol - México)
        - en_US_x_saptrc (Traduções de rastreamento SAP)
        - en_US_x_sappsd (Traduções pseudo SAP)
        - en_US_x_saprigi (Linguagem Rigi SAP)

**Configurando Locales Normalizados**

- **Configuração no CAP Node.js**:
```json
{"cds":{
  "i18n": {
    "preserved_locales": [
      "en_GB",
      "fr_CA",
      "pt_PT",
      "pt_BR",
      "zh_CN",
      "zh_HK",
      "zh_TW"
    ]
  }
}}
```

**Usando Hífens em Nomes de Arquivos**
- Devido à ambiguidade dos padrões, use hifens para separar sub tags nos nomes dos arquivos:
    - **Exemplo**: `i18n_en_GB.properties`

Essa seção fornece um guia abrangente para internacionalizar sua aplicação CAP, abordando desde a externalização de pacotes de textos até a configuração de locales normalizados e o uso de bundles baseados em CSV para textos.
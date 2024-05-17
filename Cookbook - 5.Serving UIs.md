# Serving Fiori UIs
#### Prévia SAP Fiori
- Para entidades expostas via OData V4, há um link de pré-visualização SAP Fiori na página inicial, permitindo visualizar rapidamente mudanças de anotações sem criar uma aplicação UI primeiro.
#### Adicionando Apps Fiori
- **Estrutura de Pastas**:
    - `app/`: Contém todas as apps Fiori.
        - `browse/`: App Fiori para usuários finais.
        - `orders/`: App Fiori para gerenciamento de pedidos.
        - `admin/`: App Fiori para administradores.
        - `index.html`: Para testes em sandbox.
    - `srv/`: Todos os serviços.
    - `db/`: Modelos de domínio e itens do banco de dados.
#### Usando Ferramentas SAP Fiori
- As ferramentas SAP Fiori oferecem suporte avançado para adicionar apps Fiori a projetos CAP existentes, incluindo ferramentas de produtividade como adição de anotações Fiori e modelagem gráfica.
- **Instalação**: Disponível para Visual Studio Code (VS Code) ou SAP Business Application Studio.
#### Exemplo de Amostras
- Pode-se copiar apps Fiori de `cap/samples` como um template e modificar conforme necessário.
- **Incidents Sample**: Exemplo de criação de app de gerenciamento de incidentes com elementos Fiori para OData V4.
#### Anotações Fiori
- **Definição**: Apps Fiori são front-ends genéricos que constroem e renderizam páginas com base em documentos de metadados anotados.
    - **Exemplo de Anotação**:
```cds
annotate CatalogService.Books with @(
  UI: {
    SelectionFields: [ ID, price, currency_code ],
    LineItem: [
      {Value: title},
      {Value: author, Label:'{i18n>Author}'},
      {Value: genre.name},
      {Value: price},
      {Value: currency.symbol, Label:' '},
    ]
  }
);
```    

#### Onde Colocá-las?
- Recomendação: Colocar as anotações em arquivos `.cds` separados dentro das pastas `./app/*`.
    - **Exemplo**:
```sh
./app
   ./admin
      fiori-service.cds # anotando ../srv/admin-service.cds
   ./browse
      fiori-service.cds # anotando ../srv/cat-service.cds
   index.cds
./srv
   admin-service.cds
   cat-service.cds
```    
#### Mantendo Anotações
- Ferramentas SAP Fiori aceleram a manutenção de anotações OData em arquivos `.cds`, oferecendo:
    - **Compleção de Código**
    - **Validação**
    - **Navegação para anotações referenciadas**
    - **Suporte à internacionalização**

Suporte a Rascunhos (Draft)
- SAP Fiori suporta sessões de edição com estados de rascunho armazenados no servidor.
- **Ativando Rascunho**:
```cds
annotate AdminService.Books with @odata.draft.enabled;
```
#### Validação de Rascunhos
- É possível adicionar manipuladores personalizados para validar entradas durante a sessão de edição.

**Ajudas de Valor (Value Helps)**
- Suporte avançado para `@Common.ValueList` em CAP.
- **Convenience Option**:
```cds
@cds.odata.valuelist
entity Currencies {}
service BookshopService {
   entity Books {
      currency : Association to Currencies;
   }
}
```

**Ações**
- **Definir Ações no CDS**:
```cds
entity Travel as projection on my.Travel actions {
    action createTravelByTemplate() returns Travel;
    action rejectTravel();
    action acceptTravel();
    action deductDiscount( percent: Percentage not null ) returns Travel;
  };
```
- **Implementação de Ações**:
```js
this.on('acceptTravel', req => UPDATE(req._target).with({TravelStatus_code:'A'}))
```
    
- **Adicionar Botões**:
```cds
annotate TravelService.Travel with @UI : {
  LineItem : [
    { $Type  : 'UI.DataFieldForAction',
      Action : 'TravelService.acceptTravel',
      Label  : '{i18n>AcceptTravel}'   }
  ]
};
```

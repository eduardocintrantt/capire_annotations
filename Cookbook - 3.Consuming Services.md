# Consuming Services

**Introdução:**
- Para usar dados de outros serviços ou dividir a aplicação em microserviços, é necessário conectar esses serviços (serviços remotos) usando CDS.
- CAP suporta consumo de serviços com APIs dedicadas para importar definições de serviço, consultar serviços remotos e trabalhar localmente o máximo possível.

**Recursos Suportados:**

- **OData V2 e V4**
- **API de Consulta**
- **Projeções em Serviços Remotos**

**Tutoriais e Exemplos:**

- **Consumir Serviços Remotos do SAP S/4HANA Cloud usando CAP** (Node.js)
- **Capire Bookshop (Fiori)**: Exemplo CAP para CAP.
- **Aplicação Exemplo (Node.js e Java)**: Completa o tutorial end-to-end.

**Definir Cenário:**

- Identificar serviços envolvidos e sua interação.
- Definir o que será exibido na interface do usuário.
- Obter e importar definições de API de serviços externos.

**Exemplo de Cenário:**

- Gestão de risco: usuários visualizam fornecedores e riscos associados.
- Sistema bloqueia fornecedores com alto risco automaticamente.

**Integração:**

- Escolher fornecedor de uma lista remota e realizar avaliação de risco.
- Mostrar dados adicionais do fornecedor na UI.

**Extensão:**

- Buscar fornecedores e mostrar riscos associados, integrando serviços remotos com serviços locais.

**Importar API de Serviço Externo:**

- Definições de API são necessárias para comunicação com serviços remotos (formato EDMX para OData V2 e V4).
- Importar definições usando `cds import`.

**Mocking Local:**

- Adicionar dados de mock para desenvolvimento local.
- Executar projeto com definições importadas e dados de mock.

**Associações de Mock:**

- Modificar arquivos CDS para adicionar condições de associação necessárias.

**Mock de Serviço Remoto como Serviço OData (Node.js):**

- Instalar pacotes necessários.
- Executar aplicação CAP com serviço remoto mockado separadamente para testes realistas.

**Executar Consultas:**

- Conectar ao serviço remoto e usar a API de consulta do CAP para enviar requisições.

**Projeções de Modelos:**

- Usar definições de serviço externo como qualquer outra definição CDS, mas sem gerar tabelas de banco de dados.
- Definir projeções para campos relevantes.

**Construir Requisições Customizadas:**

- Usar `send` para criar requisições HTTP personalizadas quando a API de consulta não for suficiente.

**Expor Serviços Remotos**:
    
- É necessário adicionar uma projeção a um serviço remoto para expor um serviço remoto. Por exemplo:
```cds
using { API_BUSINESS_PARTNER as bupa } from '../srv/external/API_BUSINESS_PARTNER';

extend service RiskService with {
  entity BusinessPartners as projection on bupa.A_BusinessPartner;
}
```        
- O CAP tenta delegar consultas a entidades de banco de dados automaticamente, mas isso pode gerar erros. Para lidar com isso, você precisa escrever uma função de manipulador para delegar a consulta ao serviço remoto e executar a consulta recebida no serviço externo.
```js
    module.exports = cds.service.impl(async function() {
  const bupa = await cds.connect.to('API_BUSINESS_PARTNER');

  this.on('READ', 'BusinessPartners', req => {
      return bupa.run(req.query);
  });
});
```
 **Expor Serviços Remotos com Associações**:
- Você também pode expor associações de uma entidade de serviço remoto. Por exemplo, ajustando a projeção para o destino da associação e alterando o nome da associação.

**Misturar Serviços Locais e Remotos**:
- Você pode combinar serviços locais e remotos usando associações, mas isso requer manipulação manual devido às diferentes fontes de dados.

**Limitações e Matriz de Recursos**:
- A tabela detalha as diferentes implementações necessárias para misturar serviços locais e remotos, incluindo filtragem, associações, e avaliação de permissões do usuário no sistema remoto.

**Conectar e Implantar**:
- O texto fala sobre como usar destinos para conexão a sistemas remotos, seja usando destinos do SAP BTP ou destinos definidos pela aplicação.

**Resiliência e Rastreabilidade**:
- Ele menciona a importância de adicionar resiliência às comunicações externas e como o CAP adiciona cabeçalhos para rastreamento em suas solicitações externas.

**Suporte e Recursos**:
- Fornece uma tabela de recursos suportados por protocolos e tipos de autenticação para Java e Node.js.

**Destinos e Multilocação**:
- Explica como usar destinos em ambientes de múltiplos locatários, incluindo a resolução de destinos e a diferenciação entre tokens JWT e vinculações XSUAA.

**Adicionando Qualidades**:
- Fala sobre a adição de resiliência e rastreabilidade às comunicações externas, bem como as opções de autenticação e propriedades suportadas para destinos definidos pela aplicação.

**Destinos no SDK SAP Cloud para JavaScript**:
- Fornece detalhes sobre os tipos de autenticação suportados pelo SAP Cloud SDK para JavaScript.
 
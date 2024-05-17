# Events & Messaging
### Eventos Ubíquos no CAP

#### Intrinsic Eventing in CAP Core
- No CAP, tudo que acontece em tempo de execução responde a eventos, com implementações de serviços ocorrendo em manipuladores de eventos. Todos os serviços do CAP suportam emitir e reagir a eventos.
- **Exemplo de Código**:
```js
let srv = new cds.Service
// Receiving Events
srv.on ('some event', msg => console.log('1st listener received:', msg))
srv.on ('some event', msg => console.log('2nd listener received:', msg))
// Emitting Events
await srv.emit ('some event', { foo:11, bar:'12' })
```

#### Papel Típico de Emissores e Receptores:
- Emissores e receptores de eventos são desacoplados, geralmente em serviços e processos diferentes. Emissores informam sobre eventos ocorridos, enquanto receptores se conectam aos emissores para registrar manipuladores para esses eventos.
- **Exemplo de Código**:
```js
class Emitter extends cds.Service { async someMethod() {
  // inform unknown receivers about something happened
  await this.emit ('some event', { some:'payload' })
}}

class Receiver extends cds.Service { async init() {
  // connect to and register for events from Emitter
  const Emitter = await cds.connect.to('Emitter')
  Emitter.on ('some event', msg => {...})
}}
```

#### Notion Ubíqua de Eventos:
- No CAP, uma solicitação é uma especialização de uma mensagem de evento. As mesmas mecânicas são usadas para comunicação assíncrona e síncrona.
- **Diferença entre Listeners e Interceptors**:
    - Eventos assíncronos: todos os listeners são chamados.
    - Solicitações síncronas: apenas o interceptor superior é chamado, que decide se passa o controle para os próximos manipuladores.

**Mensagens:**
- **Vantagens do Uso de Mensagens**:
    - **Resiliência**: Mensagens são armazenadas e entregues quando o serviço receptor volta a ficar online.
    - **Desacoplamento**: Emissores não precisam conhecer os receptores no momento do envio.

**Exemplo de Revisão de Livros:**
- O exemplo combina um serviço de gerenciamento de revisões com um aplicativo de livraria.
- **Declaração de Eventos em CDS**:
```cds
service ReviewsService {

  // Sync API
  entity Reviews as projection on my.Reviews excluding { likes }
  action like (review: Reviews:ID);
  action unlike (review: Reviews:ID);

  // Async API
  event reviewed : {
    subject : Reviews:subject;
    count   : Integer;
    rating  : Decimal; // new avg rating
  }
}
```

**Emitindo Eventos:**
- O método `srv.emit()` é usado para emitir mensagens de evento.
```js
class ReviewsService extends cds.ApplicationService { async init() {

  // Emit a `reviewed` event whenever a subject's avg rating changes
  this.after (['CREATE','UPDATE','DELETE'], 'Reviews', (req) => {
    let { subject } = req.data, count, rating //= ...
    return this.emit ('reviewed', { subject, count, rating })
  })
}}
```

**Recebendo Eventos:**

- O código para receber eventos é encontrado em `mashup.js` no exemplo do bookshop:
```js
  // Update Books' average ratings when reviews are updated
  ReviewsService.on ('reviewed', (msg) => {
    const { subject, count, rating } = msg.data
    // ...
  })
```

## In-Process Eventing
- Quando emissores e receptores estão no mesmo processo, não é necessário nada adicional além do CAP core.
- **Iniciar um Processo de Servidor Único**:
```sh
cds watch bookstore
```
- **Adicionar ou Atualizar Revisões**:
    - Navegue para `http://localhost:4004/reviews`, adicione ou atualize uma revisão e observe a reação do servidor no terminal.
- **Verificar Classificações no Aplicativo Bookshop**:
    - Navegue para `http://localhost:4004/bookshop` e veja a lista de livros com as classificações atualizadas.

### Verificar Avaliações no Aplicativo Bookshop

- Acesse `http://localhost:4004/bookshop` para ver a lista de livros e as avaliações atualizadas.
### Usando Canais de Mensagens

- Quando emissores e receptores estão em processos separados, é necessário adicionar um canal de mensagens para encaminhar mensagens de eventos. O CAP fornece serviços de mensagens para gerenciar esses canais de mensagens.
- **Exemplo**: O serviço de revisões e o serviço de catálogo são conectados ao serviço de mensagens.

### Mensagens Uniformes e Agnósticas

- O CAP utiliza diferentes canais e brokers de mensagens de maneira transparente, sem a necessidade de alterar o código.

**Uso de Mensagens Baseadas em Arquivos no Desenvolvimento**
- Para testes rápidos, o CAP fornece uma implementação simples de mensagens baseadas em arquivos.
- **Configuração**:
```json
"cds": {
  "requires": {
    "messaging": {
      "[development]": { "kind": "file-based-messaging" }
    }
  }
}
```

**Iniciar Serviços Separadamente** 
- Primeiro, inicie o serviço de revisões:
```sh
cds watch reviews
```    
	O servidor será iniciado em `http://localhost:4005`.

- Em um terminal separado, inicie o servidor bookstore:
```sh
cds watch bookstore
```
	O servidor será iniciado em `http://localhost:4004`.

**Adicionar ou Atualizar Revisões**    
- Acesse `http://localhost:4005/vue/index.html` para adicionar ou atualizar revisões.

**Resiliência por Design**    
- Para simular uma interrupção do servidor:
	- Termine o servidor bookstore com Ctrl + C.
	- Adicione ou atualize revisões.
	- Reinicie o servidor com `cds watch bookstore`.
	- As mensagens emitidas enquanto o receptor estava offline serão entregues quando o servidor voltar a ficar online.
### Usando Múltiplos Canais

- Por padrão, o CAP usa um único canal de mensagens. No entanto, é possível configurar canais separados para diferentes emissores ou receptores.
- **Configuração:**
```json
"cds": {
  "requires": {
    "messaging": {
      "kind": "composite-messaging",
      "routes": {
        "ChannelA": ["**/ReviewsService/*"],
        "ChannelB": ["**/sap/s4/**"],
        "ChannelC": ["**/bookshop/**"]
      }
    },
    "ChannelA": {
      "kind": "enterprise-messaging"
    },
    "ChannelB": {
      "kind": "enterprise-messaging"
    },
    "ChannelC": {
      "kind": "enterprise-messaging"
    }
  }
}
```
### Mensagens de Nível Baixo

- O CAP também oferece suporte a mensagens de nível baixo, perdendo algumas vantagens de mensagens conceituais.
- **Configuração:**
```json
"cds": {
  "requires": {
    "messaging": {
      "kind": "low-level"
    }
  }
}
```
### Padrão CloudEvents

- O CAP suporta formatação de dados de eventos compatível com o padrão CloudEvents.
- **Configuração:**
```json
"cds": {
  "requires": {
    "messaging": {
      "format": "cloudevents"
    }
  }
}
```

### Usando SAP Event Mesh

- O CAP possui suporte integrado para SAP Event Mesh.
- **Configuração:**
```json
"cds": {
  "requires": {
    "messaging": {
      "[production]": {
        "kind": "enterprise-messaging",
        "format": "cloudevents"
      }
    }
  }
}
```
### Recebendo Eventos do SAP S/4HANA

- O SAP S/4HANA integra o SAP Event Mesh para mensagens. Isso facilita que aplicativos baseados em CAP recebam eventos dos sistemas SAP S/4HANA.
- **Exemplo de Extensão de Serviço:**
```cds
using { API_BUSINESS_PARTNER as S4 } from './API_BUSINESS_PARTNER';
extend service S4 with {
  event BusinessPartner.Created @(topic:'sap.s4.beh.businesspartner.v1.BusinessPartner.Created.v1') {
    BusinessPartner : String
  }
  event BusinessPartner.Changed @(topic:'sap.s4.beh.businesspartner.v1.BusinessPartner.Changed.v1') {
    BusinessPartner : String
  }
}
```
- **Recebendo Eventos:**
```js
const S4Bupa = await cds.connect.to ('API_BUSINESS_PARTNER')
S4Bupa.on ('BusinessPartner.Changed', msg => {...})
```

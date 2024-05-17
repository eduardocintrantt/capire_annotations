# Data Protection & Privacy
#### Declaração Geral

Governos impõem requisitos legais à indústria para proteger dados e privacidade.

**Dica:** Nenhum guia, incluindo este, tenta oferecer conselhos sobre os melhores métodos para suportar requisitos específicos de empresa, indústria, região ou país. As decisões relacionadas à proteção de dados devem ser feitas caso a caso, considerando o ambiente do sistema e os requisitos legais aplicáveis.

Para informações gerais sobre proteção de dados e privacidade (DPP) no SAP BTP, consulte a documentação do SAP BTP sob Proteção de Dados e Privacidade.

#### Proteção de Dados e Privacidade no CAP
CAP é um framework que fornece recursos de modelagem e runtime para permitir que os clientes construam aplicações de negócios. Em geral, CAP não armazena ou gerencia dados pessoais por conta própria, com algumas exceções:

- **Logs de Aplicação**: Logs detalhados escritos pelo runtime CAP podem conter dados pessoais como nomes de usuários e endereços IP. Esses logs são obrigatórios para operar o sistema. Conecte um serviço de logging adequado para atender aos requisitos de conformidade, como o SAP Application Logging Service.
- **Draft-enabled Service**: Um serviço habilitado para rascunho, como Foo, possui uma entidade `Foo.DraftAdministrativeData` com campos `CreatedByUser`, `InProcessByUser` e `LastChangedByUser`, contendo dados pessoais para todas as instâncias de entidades em modo de edição.
- **Mensagens Temporárias**: Mensagens escritas temporariamente na caixa de saída de transações podem conter dados pessoais. As entradas são obrigatórias para operar o sistema. Se necessário, as aplicações podem processar essas mensagens utilizando a funcionalidade padrão do CAP (modelo CDS `@sap/cds/srv/outbox`).
- **Aspecto Managed**: Dados pessoais podem ser adicionados automaticamente ao usar o aspecto managed.
- **Modelos CDS Personalizados**: Dependendo do cenário de negócios, modelos CDS personalizados servidos pelo runtime CAP provavelmente conterão dados pessoais armazenados em um serviço de apoio.

CAP oferece um conjunto rico de ferramentas para proteger a aplicação contra acesso não autorizado a dados de negócios, incluindo dados pessoais. Além disso, ajuda as aplicações a fornecer funções relacionadas ao DPP, como a recuperação de dados.

**Aviso:** As aplicações são responsáveis por implementar requisitos de conformidade relacionados à proteção de dados e privacidade de acordo com seu caso de uso específico.

Consulte também os guias relacionados aos serviços de plataforma mais importantes:

- SAP Cloud Identity Services - Configuração de Políticas de Privacidade
- SAP HANA Cloud - Proteção de Dados e Privacidade

#### Recursos de Proteção de Dados e Privacidade Suportados pelo CAP
CAP oferece vários recursos para ajudar as aplicações a atender aos requisitos de DPP:

- **Gestão de Dados Pessoais (PDM)**: Integração com uma função de recuperação configurável, que pode ser usada para informar os titulares dos dados sobre os dados pessoais armazenados relacionados a eles.
- **Rastreamento de Alterações**: Abordagem totalmente orientada por modelo para rastrear alterações em dados pessoais ou acesso a dados pessoais sensíveis no log de auditoria. Após declarar dados pessoais no seu modelo, o CAP aciona automaticamente eventos correspondentes no log de auditoria.

**Aviso:** Até o momento, as aplicações precisam integrar o SAP Data Retention Manager para implementar uma função de apagamento adequada para dados pessoais fora do período de retenção. CAP cobrirá uma integração pronta para uso no futuro.
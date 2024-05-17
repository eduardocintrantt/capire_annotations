# Security Aspects
#### Comunicações Seguras

**Canais de Comunicação Criptografados** É essencial garantir a integridade e a confidencialidade dos dados transferidos entre os pontos de comunicação, tanto entre cliente e servidor quanto entre serviços.

**Comunicação de Entrada (Servidor)** No SAP BTP, os canais de comunicação são criptografados via HTTPS/TLS por padrão. O gateway da API da plataforma fornece endpoints TLS, encaminhando solicitações para serviços de apoio via HTTP. Os microserviços só são acessíveis pelo roteador em termos de tecnologia de rede, garantindo a segurança perimetral.

**Dica:** Clientes públicos devem autenticar e verificar a identidade do servidor.

**Comunicação de Saída (Cliente)** Conexões de saída também são automaticamente seguras, e clientes técnicos devem validar o certificado do servidor para autenticação adequada.

**Comunicação Interna (Cliente e Servidor)** Microserviços estreitamente acoplados podem se comunicar via canais de rede confiáveis, como localhost.

**Filtragem de Tráfego da Internet** Reduzir a superfície de ataque filtrando a comunicação aumenta a segurança geral. A plataforma oferece configurações padrão para proteger a comunicação de rede.

**Aviso:** Medidas adicionais para restringir o acesso à web devem ser aplicadas no nível da plataforma.

#### Autenticação Segura
Recursos não públicos devem ser acessados apenas por usuários autenticados. A autenticação é crucial para a segurança em diferentes níveis:

- **Usuários de negócios**: Consumem a aplicação via interface web, isolados por locatário em aplicações multitenant.
- **Usuários da plataforma**: Operam a aplicação com acesso privilegiado a componentes de sistema.

**Recomendações:**

- Utilize serviços de identidade da plataforma.
- Garanta que um método de autenticação apropriado esteja configurado.

#### Serviços Remotos
Microserviços CAP que consomem serviços remotos precisam ser autenticados como clientes técnicos. A CAP facilita a configuração segura de comunicação entre serviços.

**Manutenção de Sessões** CAP requer autenticação para todas as solicitações, mas não suporta fluxos de login para clientes UI diretamente. A Roteador de Aplicação pode ser usado como proxy reverso para gerenciar sessões e tokens OAuth2.

**Aviso:** O Roteador de Aplicação não oculta os endpoints CAP no backend, sendo a autenticação ainda necessária.

#### Manutenção de Segredos
CAP não gerencia segredos diretamente, mas depende dos mecanismos de injeção de segredos da plataforma.

**Dica:** Utilize o serviço SAP Credential Store para armazenar segredos, se necessário.

#### Autorização Segura
Assegure-se de que os administradores de usuários controlem como diferentes usuários podem interagir com a aplicação, evitando combinações críticas de autorizações.

**Usuários de Negócios** Enforce um controle de acesso detalhado baseado em regras de autorização definidas no modelo CDS. Utilize as anotações @requires ou @restrict para declarar condições específicas de acesso.

**Aviso:** Por padrão, serviços e entidades CAP não possuem autorização. Desenvolvedores devem definir e testar regras de acesso.

**Verificação de Autorizações CAP:** Recomenda-se o uso de regras de lint do CDS.

#### Autorização de Endpoints CAP
CAP expõe endpoints baseados no modelo CDS e configuração dos serviços CDS, garantindo que apenas usuários autorizados acessem informações do servidor.

#### Multi-Tenancy Seguro
Aplicações SaaS multitenant devem garantir isolamento de locatários para dados persistentes, dados transitórios e consumo de recursos.

**Dados Persistentes Isolados** CAP automaticamente direciona solicitações para um contêiner HDI isolado dedicado para cada locatário, garantindo que as consultas de banco de dados sejam executadas no esquema apropriado.

**Dados Transitórios Isolados** A runtime do CAP Java precisa cachear dados em memória para desempenho. Dados de solicitação são propagados via variável local cds.context.

**Aviso:** Certifique-se de que o código personalizado não quebre o isolamento de dados dos locatários.

**Limitação de Consumo de Recursos** Microserviços precisam gerenciar o consumo de recursos para evitar problemas de desempenho causados por uso excessivo por um único locatário.

**Dica:** Tenha uma estratégia de escalonamento para atender às demandas crescentes.

#### Proteção Contra Entrada Não Confiável
Mecanismos de proteção são necessários para evitar que usuários mal-intencionados ataquem recursos valiosos.

**Ataques de Injeção** CAP é imune a injeções de SQL devido ao uso de prepared statements. No entanto, é necessário validar a estrutura da consulta baseada na entrada do usuário.

**XSS (Cross Site Scripting)** SAPUI5 fornece validação de entrada e codificação automática de saída para propriedades de elementos digitados, protegendo contra XSS.

**Aviso:** Adicione manipuladores personalizados para escanear dados enviados e recebidos.

**Recomendações Gerais Contra Injeções**

- Validação de entrada e saída.
- Política de Segurança de Conteúdo adequada.

**Ataques de Negação de Serviço (DoS)** Prevenção contra DoS é tratada em diferentes camadas do stack de runtime. Os adaptadores CAP introduzem paginação de resultados para limitar picos de memória.

**Aviso:** Limite a quantidade de $expands por solicitação e configure o limite máximo de solicitações por $batch.

**Medidas Suplementares**

- Implementação de uma estratégia de limitação de taxa.
- Filtragem de solicitações e limitação de taxa via Route Service na plataforma.

**Proteção Contra Vulnerabilidades** CAP oferece ferramentas para reduzir vetores de ataque de vulnerabilidades de condição de corrida. Aplique controle de concorrência adequado.

#### Segurança por Padrão e Design
**Configuração Padrão Segura** CAP oferece configuração segura por padrão, não exigindo senhas ou certificados para proteger a comunicação.

**Dica:** Use testes de integração automatizados para garantir configurações de segurança.

**Falhas Seguras** CAP trata exceções e erros para garantir que apenas a solicitação falha seja afetada e a segurança não seja comprometida.

**Dica:** Alinhe o tratamento de exceções no seu código personalizado com as capacidades de tratamento de exceções da runtime CAP.
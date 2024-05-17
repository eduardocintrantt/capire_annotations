# Platform Security
#### Conformidade com a Plataforma

As aplicações CAP operam em ambientes específicos, seja na nuvem ou localmente. O framework subjacente tem um impacto significativo na segurança da aplicação, especialmente quando integrado com serviços de identidade e persistência.

**Ponto Principal:** Garanta a configuração de segurança consistente em toda a plataforma e serviços consumidos.

#### CAP em Ambiente de Nuvem
O CAP suporta dois runtimes na SAP Business Technology Platform (BTP):

1. SAP BTP, Cloud Foundry Runtime
2. SAP BTP, Kyma Runtime

Os provedores de aplicações devem:

- Configurar serviços da plataforma para separar usuários da plataforma de usuários de negócios.
- Implementar políticas de login seguras, como autenticação multifator.
- Proteger repositórios de código-fonte e processos de implantação.
- Estabelecer processos de gerenciamento de patches e vulnerabilidades.

**Aviso:** O provedor da aplicação é responsável por desenvolver, implantar e operar a aplicação de maneira segura.

**Recursos:**

- SAP BTP Security
- Recomendações de Segurança do SAP BTP
- Segurança do SAP BTP (Comunidade)

#### CAP em Ambiente Local
O desenvolvimento local também exige atenção à segurança:

- Vincule endpoints HTTP ao localhost.
- Use `cds bind` para vinculações de serviço em nuvem para evitar armazenar segredos localmente.
- Evite registrar dados sensíveis em logs.
- Não use dados reais de negócios para testes.

#### Serviços de Segurança do SAP BTP
O SAP BTP oferece vários serviços para suportar a segurança em nível de produção para aplicações CAP:

- **SAP Cloud Identity Services - Identity Authentication**: Gerencia o acesso dos usuários e integra com provedores de identidade de terceiros ou locais.
- **SAP Authorization and Trust Management Service**: Gerencia autorizações e papéis de usuários.
- **SAP Malware Scanning Service**: Verifica documentos de negócios quanto a malwares (a integração CAP ainda não está disponível).
- **SAP Credential Store**: Armazena e recupera credenciais de aplicativos de forma segura.
- **SAP BTP Connectivity**: Oferece acesso seguro a serviços remotos.

**Recurso:** Centro de Confiança da SAP

#### Requisitos de Arquitetura e Plataforma
As aplicações CAP exigem um ambiente de plataforma específico para garantir a segurança.

**Visão Geral da Arquitetura:**

- **Zona Pública**: Componentes expostos com requisitos mínimos de segurança. Gateways protegem esses endpoints.
- **Zona de Plataforma**: Inclui serviços da plataforma (banco de dados, serviço de identidade) configurados pelo provedor da aplicação.
- **Zona de Aplicação**: Contém microsserviços CAP e componentes opcionais como o Application Router e o CAP sidecar.

**Pontos Principais:**

- Os provedores de aplicações devem manter a segurança de todos os componentes.
- Endpoints externos são protegidos pelo gateway da API da plataforma usando TLS.
- Segredos são injetados de forma segura pela plataforma.

#### Requisitos do Ambiente de Plataforma
As aplicações CAP assumem:

- Endpoints protegidos por TLS expostos pelo gateway da API.
- Certificados de servidor assinados por uma autoridade de certificação confiável.
- Segredos injetados de forma segura pela plataforma.

**Dicas e Avisos:**

- Certificados de domínios personalizados devem ser confiáveis.
- Os endpoints CAP devem ser protegidos contra acesso público.
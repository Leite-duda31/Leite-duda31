RELATÓRIO DE AUDITORIA TÉCNICA E ANÁLISE DE VULNERABILIDADES
Autor: Eduarda Leite (Duda)
Metodologia: Teste de Segurança Caixa-Preta (Black-Box Assessment) & OSINT
Escopo: Infraestrutura Web e Políticas de Borda

SUMÁRIO EXECUTIVO
Este documento apresenta os resultados da análise de segurança e resiliência realizada na plataforma web. Foram identificadas três falhas estruturais causadas por Security 
Misconfiguration (Má Configuração de Segurança) na gerência do servidor. A análise demonstra que a fragilidade do ativo não decorre de falhas de dia-zero (0-day) no 
software, mas sim de falhas operacionais humanas na administração do ambiente de produção. O rápido reporte e a postura ética na divulgação responsável impediram
incidentes catastróficos de vazamento de dados (LGPD) e indisponibilidade.

CASO 01: Configuração Inadequada de TLS/SSL e Balanceamento de Carga (Self-DoS)

Classificação OWASP: A05:2021 – Security Misconfiguration

Vetor de Descoberta: Análise Passiva de Tráfego via DevTools (F12) e OSINT (Shodan).

Descrição do Cenário:
A infraestrutura operava com dois servidores ativos de forma descentralizada. O Servidor A escutava na porta lógica 443 (HTTPS/TLS) e o Servidor B na porta 80 
(HTTP convencional). O cabeçalho HTTP configurava o parâmetro de sessão Set-Cookie: PHPSESSID com a flag estrita Secure.
Isso impedia que o Servidor B processasse o identificador por canais inseguros. Consequentemente, cada requisição recebida no Servidor B gerava um loop de 
redirecionamento imediato para a porta 443. Esse erro de arquitetura causou um isolamento do Servidor B (ocioso) e o afunilamento de toda a carga de processamento de 
conexões concorrentes no Servidor A, gerando um cenário de Self-DoS (Negação de Serviço Involuntária) e oscilação crônica.

Impacto Técnico e de Negócio: Interrupção da disponibilidade da plataforma, risco latente de corrupção nas tabelas do banco de dados por conexões caídas abruptamente 
e degradação da experiência do usuário (gerando prejuízo financeiro direto).

Status de Remediação: Resolvido. A infraestrutura foi centralizada em um servidor robusto e unificado tratando adequadamente o direcionamento das portas.

Solução Recomendada por Arquitetura: Implementação de SSL Offloading em camada de borda (via Proxy Reverso ou WAF). O dispositivo de borda realiza a 
descriptografia pesada do TLS e encaminha a requisição limpa para o backend, otimizando o consumo de CPU.

CASO 02: Exposição de Diretório de Controle de Versão (.git)

Classificação OWASP: A05:2021 – Security Misconfiguration / Information Disclosure

Vetor de Descoberta: Execução automatizada e furtiva através do fuzzer customizado (Ferramenta Leviatã).

Descrição do Cenário:
Durante a esteira de desenvolvimento ou publicação (deploy), o diretório oculto .git foi copiado inadvertidamente para a pasta pública do servidor web (raiz de produção)
, retornando status HTTP 200 OK na rota /.git/config. A falha permitia a reconstrução parcial ou total da árvore de diretórios do código-fonte, além de expor o repositório
 privado do desenvolvedor no GitHub.

Impacto Técnico e de Negócio: Risco Crítico de engenharia reversa. A exposição do código-fonte permite que agentes mal-intencionados identifiquem credenciais 
codificadas de forma estática (hardcoded), endereços de IPs internos de staging, chaves de API integradas e a lógica de comunicação com os gateways de pagamento.

Status de Remediação: Resolvido. Bloquearam todo e qualquer acesso á esse dirertório.

Solução Recomendada por Arquitetura: Remoção imediata do diretório .git do servidor de produção ou implementação de diretivas restritas no arquivo de configuração do
servidor web (ex: Apache RedirectMatch 404 /\.git) para bloquear qualquer requisição externa a arquivos ocultos.

CASO 03: Listagem de Diretório Ativa e Exposição Massiva de Dados Pessoais Sensíveis

Classificação OWASP: A01:2021 – Broken Access Control / A05:2021 – Security Misconfiguration

Vetor de Descoberta: Mapeamento de caminhos e inteligência de Camada 7 via Ferramenta Leviatã.

Descrição do Cenário
:Após uma atualização/commit de código, o diretório /docs ficou acessível publicamente na internet devido à ativação inadvertida da diretiva de listagem de arquivos 
(Directory Listing). O servidor listava abertamente de 100 a 150 arquivos contendo dados altamente sensíveis de terceiros.

Impacto Técnico e de Negócio: Violação crítica e direta da Lei Geral de Proteção de Dados (LGPD), sujeita a sanções administrativas e multas pesadas. Os arquivos
 expunham diretamente: Imagens de RGs, Cartões Nacionais de Saúde (SUS), Fotografias de Identificação Facial, registros biométricos de impressões digitais e Chaves Pix 
baseadas em e-mails e CPFs.

Em posse de agentes criminosos, esses dados alimentariam ataques de Credential Stuffing (via checagem em bases como Have I Been Pwned
para Account Takeover), abertura de contas fraudulentas (laranjas), extorsão e engenharia social direcionada. O impacto transcende o dano sistêmico, gerando severas 
consequências psicológicas e financeiras às vítimas.

Status de Remediação: Resolvido Interinamente. Após o alerta imediato via canal privado de divulgação responsável, o acesso público ao diretório foi interrompido. Os 
registros de log do servidor confirmaram que o acesso foi mitigado antes de qualquer captura por indexadores ou agentes maliciosos.

Solução Recomendada por Arquitetura: Desativação global da listagem de índices no servidor Apache (alterando para Options -Indexes), além de mover o armazenamento 
de documentos sensíveis para um diretório fora da raiz pública do servidor (DocumentRoot), exigindo tokens de autenticação individuais (Access Control Lists) para cada
visualização.

CONCLUSÃO E PARECER DO AUDITOR

As evidências coletadas neste ciclo de auditoria confirmam o axioma mais tradicional da segurança da informação: o fator humano e a governança operacional 
representam os elos mais críticos da defesa. A segurança da plataforma foi comprometida não pela complexidade dos códigos, mas pela negligência em configurações
 básicas de hardening de servidores e falta de higienização de ambientes em produção. O reporte ético evitou sanções jurídicas catastróficas. Recomenda-se a adoção
 imediata de políticas de Secure DevSecOps para automatizar o bloqueio dessas falhas antes de cada publicação de código.


esse foi o meu primeiro diclosure oficial, usando a ferramenta que eu mesma contruí, leviata. Aprendi muito com tudo isso, inclusive como uma empresa funciona por baixo dos panos. 













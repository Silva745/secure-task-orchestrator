# Regras Transversais (Back-end + Front-end)

Níveis: **[O]** Obrigatório · **[R]** Recomendado · **[A]** Avançado.
Aplicam-se a **qualquer** subtarefa de código, independentemente da camada.

---

## X-PRIV — Proteção de dados e privacidade (RGPD quando aplicável)

Princípios: licitude, lealdade e transparência · limitação das finalidades · minimização · exatidão
· limitação da conservação · integridade e confidencialidade · responsabilidade demonstrada.

| ID | Nível | Regra |
|---|---|---|
| X-PRIV-01 | O | Definir os dados recolhidos, a finalidade e o fundamento jurídico de cada recolha; não recolher dados desnecessários |
| X-PRIV-02 | O | Política de privacidade; prazos de conservação definidos |
| X-PRIV-03 | O | Suportar direitos dos titulares: retificação, eliminação, exportação |
| X-PRIV-04 | O | Controlar o acesso a dados pessoais; medidas de segurança proporcionais ao risco |
| X-PRIV-05 | R | Documentar fornecedores e subcontratantes; avaliar transferências internacionais |
| X-PRIV-06 | R | Avaliar necessidade de DPIA; processo de resposta a incidentes |

## X-GIT — Git e versionamento

| ID | Nível | Regra |
|---|---|---|
| X-GIT-01 | O | Usar Git; nunca trabalhar diretamente na branch de produção; branches por funcionalidade/correção |
| X-GIT-02 | O | Pull Requests com Code Review; sem auto-aprovação de alterações críticas |
| X-GIT-03 | O | **Nenhum secret em commits**; `.gitignore` correto; lockfiles |
| X-GIT-04 | R | Commits descritivos; tags de versão; proteção de branches; verificações automáticas antes do merge |
| X-GIT-05 | R | Não apagar histórico de forma irresponsável; documentar alterações importantes |

Exemplo de branches: `main`, `develop`, `feature/*`, `fix/*`, `hotfix/*`, `release/vX.Y.Z`.

## X-DEP — Dependências e cadeia de fornecimento

| ID | Nível | Regra |
|---|---|---|
| X-DEP-01 | O | Pacotes oficiais, de origem verificada; evitar abandonados e desnecessários |
| X-DEP-02 | O | Versões controladas + lockfile; verificar vulnerabilidades conhecidas; atualizar bibliotecas críticas |
| X-DEP-03 | R | Rever scripts de instalação e dependências transitivas; monitorizar vulnerabilidades pós-lançamento |
| X-DEP-04 | A | SBOM; processo de substituição de dependências comprometidas |

## X-INC — Gestão de incidentes

| ID | Nível | Regra |
|---|---|---|
| X-INC-01 | O | Plano de resposta com fases: preparação, identificação, contenção, investigação, erradicação, recuperação, comunicação, revisão pós-incidente |
| X-INC-02 | O | Definir responsáveis, canais de comunicação, prioridades, evidências a preservar e procedimentos de recuperação |

Cenários a cobrir: roubo de credenciais, exposição de API keys, acesso indevido a dados, SQL
Injection, DDoS, comprometimento de servidor, ficheiros maliciosos, roubo de sessões, alterações não
autorizadas, fuga de dados pessoais.

## X-DOC — Documentação mínima

| ID | Nível | Regra |
|---|---|---|
| X-DOC-01 | O | `README` (apresentação e instalação) |
| X-DOC-02 | R | `Architecture.md`, `Database.md`, `API.md`, `Security.md`, `Deployment.md`, `Environment.md`, `Permissions.md`, `Testing.md`, `Incident-Response.md`, `Changelog.md`, `Backup-Recovery.md` — proporcionais à dimensão do projeto |
| X-DOC-03 | O | `.env.example` sem valores reais; variáveis de ambiente documentadas |

## X-AI — Inteligência artificial e integrações com LLMs

| ID | Nível | Regra |
|---|---|---|
| X-AI-01 | O | Nunca expor API keys de fornecedores de IA; chamadas ao modelo passam pelo Back-end |
| X-AI-02 | O | Credenciais e configurações separadas por organização; nenhum utilizador acede a credenciais de outros |
| X-AI-03 | O | Rate limiting, quotas e controlo de custos |
| X-AI-04 | O | Não enviar dados pessoais desnecessários a modelos externos; definir retenção com fornecedores |
| X-AI-05 | O | Não confiar cegamente na resposta do modelo; validar resultados estruturados |
| X-AI-06 | O | A IA não executa operações críticas sem autorização; confirmação humana para ações de elevado impacto |
| X-AI-07 | O | Autorização aplicada a **cada ferramenta** que o agente pode chamar; limites de execução e timeouts |
| X-AI-08 | O | Isolar contextos e conversas entre utilizadores/tenants |
| X-AI-09 | O | Prevenir prompt injection; conteúdo externo nunca altera diretamente as políticas do sistema |
| X-AI-10 | R | Registar ações relevantes dos agentes; proteger contra extração indevida de informação |
| X-AI-11 | A | Testar comportamentos adversariais |

## X-NORMS — Normas de referência

| Norma | Uso |
|---|---|
| OWASP ASVS (5.0.0) | Requisitos de segurança para web/serviços — define os níveis 1/2/3 |
| OWASP API Security Top 10 | Riscos das APIs |
| OWASP MASVS | Segurança mobile |
| NIST SSDF (SP 800-218) | Desenvolvimento seguro no ciclo de vida |
| WCAG 2.2 | Acessibilidade (alvo AA) |
| OWASP WSTG | Testes de segurança web |

---

## Regra de produção e exceções

> Nenhum sistema está pronto para produção enquanto existirem **vulnerabilidades críticas ou altas
> por resolver**, **controlos obrigatórios por implementar** ou **falhas de autorização que
> comprometam dados e operações**.

Cada exceção exige: **justificação documentada · responsável pela aprovação · medidas compensatórias
· prazo de resolução.**

# Regras de Back-end

Cada regra tem um **ID** (para citar em planos e auditorias) e um nível:
**[O]** Obrigatório · **[R]** Recomendado · **[A]** Avançado (sobretudo Nível 3 / alta exposição).

> **Princípio fundamental:** nunca confiar exclusivamente no Front-end. Autorização, integridade e
> validação crítica vivem aqui, mesmo que também existam no cliente.

---

## BE-ARCH — Arquitetura e organização

| ID | Nível | Regra |
|---|---|---|
| BE-ARCH-01 | O | Separation of Concerns: cada componente com uma responsabilidade |
| BE-ARCH-02 | O | SRP / DIP: módulos de alto nível não dependem de implementações concretas |
| BE-ARCH-03 | R | DRY, KISS, YAGNI; SOLID; baixo acoplamento e alta coesão |
| BE-ARCH-04 | O | Fail Securely: numa falha, o sistema assume estado seguro |
| BE-ARCH-05 | O | Secure by Design + Defense in Depth: segurança desde a arquitetura, várias camadas |
| BE-ARCH-06 | O | Least Privilege para utilizadores, serviços e processos |
| BE-ARCH-07 | R | Zero Trust: não confiar automaticamente em utilizadores, dispositivos, serviços ou redes |
| BE-ARCH-08 | O | Fronteiras de confiança explícitas: identificar onde os dados as atravessam |
| BE-ARCH-09 | R | Estrutura modular e previsível (ex.: `Modules/`, `Core/`, `Infrastructure/`, `Shared/`, `config/`, `database/migrations`, `routes/`, `tests/{Unit,Feature,Integration,Security}`, `docs/`, `.env.example`, `.gitignore`) — adaptar à linguagem e dimensão; é referência, não imposição |

## BE-CODE — Nomenclatura e Clean Code

| ID | Nível | Regra |
|---|---|---|
| BE-CODE-01 | O | Nomes descritivos e consistentes; sem `data`, `info`, `temp`, `test2`, `abc` em código definitivo |
| BE-CODE-02 | R | Inglês para identificadores técnicos (salvo convenção institucional) |
| BE-CODE-03 | R | Classes = PascalCase · métodos/variáveis = camelCase · constantes = UPPER_SNAKE_CASE · tabelas/colunas = snake_case · rotas = kebab-case |
| BE-CODE-04 | R | Métodos pequenos e focados; poucos parâmetros; sem aninhamento profundo |
| BE-CODE-05 | O | Sem código morto nem imports não usados; sem números/strings mágicas |
| BE-CODE-06 | O | Configurações centralizadas; sem dependências circulares |
| BE-CODE-07 | O | Não misturar regras de negócio com apresentação; sem queries complexas em controladores |
| BE-CODE-08 | R | Evitar God Classes e métodos com várias responsabilidades |
| BE-CODE-09 | O | Tratamento de exceções consistente; tipagem sempre que a linguagem permita |
| BE-CODE-10 | R | Comentários só para decisões técnicas complexas (não repetir o código) |
| BE-CODE-11 | R | Formatação e análise estática automáticas; refactoring sem alterar comportamento |

> Código que funciona mas não pode ser compreendido, testado ou mantido com segurança **não** é
> código de qualidade.

## BE-AUTH — Autenticação e identidade

### Registo
| ID | Nível | Regra |
|---|---|---|
| BE-AUTH-01 | O | Validar todos os dados de registo no servidor; normalizar emails/telefones/identificadores |
| BE-AUTH-02 | O | Verificar propriedade do email/telefone; não ativar contas antes de concluída a verificação |
| BE-AUTH-03 | O | O utilizador **nunca** define o seu papel/permissões; sem atribuição automática de privilégios |
| BE-AUTH-04 | O | Identificadores seguros; proteção do registo contra bots e abuso; limite de tentativas |
| BE-AUTH-05 | R | Registar eventos de criação de contas; não devolver informação interna da conta |

### Palavras-passe
| ID | Nível | Regra |
|---|---|---|
| BE-PWD-01 | O | Hashing com algoritmo próprio para passwords (ex.: **Argon2id**), salt único, custo adequado |
| BE-PWD-02 | O | Nunca texto simples, MD5, SHA-1 ou SHA-256 simples |
| BE-PWD-03 | O | Nunca em logs, nunca por email, nunca hashes devolvidos por API |
| BE-PWD-04 | R | Permitir gestores de passwords e autopreenchimento; verificar passwords comprometidas |
| BE-PWD-05 | R | Sem rotação periódica arbitrária nem regras de composição excessivas |

### Login e proteção de contas
| ID | Nível | Regra |
|---|---|---|
| BE-LOGIN-01 | O | Rate limiting no login; atrasos progressivos; prevenir força bruta e credential stuffing |
| BE-LOGIN-02 | O | Não revelar se um email está registado |
| BE-LOGIN-03 | O | MFA obrigatório para contas administrativas/privilegiadas; recomendado para as restantes |
| BE-LOGIN-04 | O | Registar tentativas de autenticação; notificar logins suspeitos |
| BE-LOGIN-05 | O | Bloqueios temporários com critérios seguros (evitar bloqueio permanente → DoS de contas) |
| BE-LOGIN-06 | O | Reautenticação para operações críticas; proteger alterações de email, telefone e password |

### Recuperação de password
| ID | Nível | Regra |
|---|---|---|
| BE-RESET-01 | O | Tokens aleatórios criptograficamente seguros, validade limitada, uso único |
| BE-RESET-02 | R | Guardar apenas o hash do token |
| BE-RESET-03 | O | Não revelar se a conta existe; invalidar o token após uso |
| BE-RESET-04 | O | Revogar (ou oferecer revogar) sessões existentes; notificar o utilizador |
| BE-RESET-05 | O | Tokens fora de URLs que possam chegar a logs/terceiros sem proteção |

### Sessões
| ID | Nível | Regra |
|---|---|---|
| BE-SESS-01 | O | IDs de sessão imprevisíveis; regenerar após autenticação e mudança de privilégio (anti Session Fixation) |
| BE-SESS-02 | O | Expiração por inatividade e duração máxima; sem sessões indefinidas |
| BE-SESS-03 | O | Logout efetivo no servidor; permitir terminar todas as sessões; revogar após eventos críticos |
| BE-SESS-04 | O | Cookies `HttpOnly`, `Secure`, `SameSite` quando apropriado |
| BE-SESS-05 | O | Tokens de sessão sensíveis nunca em `localStorage` nem em URLs |

## BE-AUTHZ — Autorização

| ID | Nível | Regra |
|---|---|---|
| BE-AUTHZ-01 | O | Toda a operação protegida verifica autorização **no Back-end**, por endpoint, por operação e por campo sensível |
| BE-AUTHZ-02 | O | **Default Deny** |
| BE-AUTHZ-03 | O | Não confiar no papel nem em IDs enviados pelo cliente |
| BE-AUTHZ-04 | O | Verificar propriedade do recurso (anti BOLA/IDOR — acesso horizontal) |
| BE-AUTHZ-05 | O | Impedir acesso vertical a funcionalidades administrativas |
| BE-AUTHZ-06 | O | Verificar organização/tenant em sistemas multiempresa |
| BE-AUTHZ-07 | O | Ocultar um botão **não** é mecanismo de segurança |
| BE-AUTHZ-08 | R | RBAC + ABAC + autorização por recurso conforme necessário (ex.: Super Admin, Admin, Manager, Operator, Analyst, User) |
| BE-AUTHZ-09 | O | Rever e revogar permissões imediatamente quando a função muda; auditar alterações de privilégios |

> Exemplo BOLA: `GET /api/users/100` → utilizador altera para `/101` e vê dados alheios. A
> autorização é por **recurso**, não só por endpoint.

## BE-API — Segurança das APIs

| ID | Nível | Regra |
|---|---|---|
| BE-API-01 | O | HTTPS em todas as comunicações; timeouts definidos |
| BE-API-02 | O | Validar parâmetros, tipos, formatos e content-type; limitar tamanho de payloads |
| BE-API-03 | O | Paginação e limite de resultados; rate limiting |
| BE-API-04 | O | Autenticação e autorização por recurso em cada endpoint |
| BE-API-05 | O | Não devolver campos internos desnecessários, stack traces nem info de infraestrutura |
| BE-API-06 | O | Endpoints administrativos protegidos; debug desativado em produção |
| BE-API-07 | R | Versionar, padronizar respostas, documentar e inventariar todas as APIs; descontinuar versões antigas de forma controlada |
| BE-API-08 | O | Restringir métodos HTTP ao necessário; GET nunca altera estado |

### OWASP API Security Top 10 — mitigar todos
BOLA · Broken Authentication · Broken Object Property Level Authorization · Unrestricted Resource
Consumption · Broken Function Level Authorization · Unrestricted Access to Sensitive Business Flows ·
SSRF · Security Misconfiguration · Improper Inventory Management · Unsafe Consumption of APIs.

### CORS
| ID | Nível | Regra |
|---|---|---|
| BE-CORS-01 | O | Origens permitidas explícitas; nunca `*` com credenciais; nunca refletir qualquer origem |
| BE-CORS-02 | O | Limitar métodos e headers; configurar `Allow-Credentials` corretamente |
| BE-CORS-03 | O | CORS não é autenticação nem autorização |

### Proteção contra ataques
| Ataque | Medida |
|---|---|
| SQL Injection | Queries parametrizadas e ORM seguro |
| XSS | Encoding de saída e sanitização contextual |
| CSRF | Tokens anti-CSRF e cookies adequados |
| SSRF | Allowlist de destinos; bloquear redes internas |
| IDOR/BOLA | Autorização por recurso |
| Brute force | Rate limiting e MFA |
| Replay | Nonces, timestamps e idempotência |
| Path Traversal | Normalização e restrição de caminhos |
| XXE | Desativar entidades externas XML |
| Command Injection | Evitar shell; parametrizar operações |
| Mass Assignment | Allowlist de campos editáveis |
| Open Redirect | Destinos previamente autorizados |
| Clickjacking | CSP `frame-ancestors` |
| DoS | Limites, timeouts, proteção de infraestrutura |
| Deserialization | Nunca desserializar dados não confiáveis de forma insegura |

## BE-DB — Bases de dados

| ID | Nível | Regra |
|---|---|---|
| BE-DB-01 | O | Credenciais da BD nunca no Front-end nem em repositórios; contas de serviço dedicadas, menor privilégio, nunca root/admin |
| BE-DB-02 | O | BD não exposta à Internet; rede restringida; ligações cifradas quando apropriado |
| BE-DB-03 | O | Credenciais separadas por ambiente e rotacionadas; acessos privilegiados auditados |
| BE-DB-04 | O | PK, FK, `NOT NULL`, `UNIQUE`, índices e constraints; integridade referencial; transações |
| BE-DB-05 | O | Migrations versionadas; sem alterações diretas de estrutura em produção |
| BE-DB-06 | R | Soft delete quando justificável; não apagar dados críticos sem confirmação e política de recuperação |
| BE-DB-07 | R | Políticas de retenção; não guardar informação desnecessária |
| BE-DB-08 | O | Cifrar dados sensíveis em repouso quando necessário, com algoritmos reconhecidos; chaves geridas separadamente |
| BE-DB-09 | O | Não guardar dados de cartões sem necessidade; mascarar dados em relatórios; controlar e auditar exportações e consultas sensíveis |
| BE-DB-10 | O | Backups automáticos, cifrados, com acesso restrito e nunca públicos |
| BE-DB-11 | O | Restauro de backups testado; RPO e RTO definidos; procedimentos documentados |
| BE-DB-12 | A | Cópias isoladas/imutáveis contra ransomware |

## BE-VAL — Validação, sanitização e tratamento de dados

| ID | Nível | Regra |
|---|---|---|
| BE-VAL-01 | O | Validar **todos** os dados no servidor: tipo, comprimento, formato, intervalo, datas, identificadores, relações entre campos e regras de negócio |
| BE-VAL-02 | O | Rejeitar dados malformados e parâmetros inesperados; sem correções silenciosas que alterem o significado |
| BE-VAL-03 | O | Mass assignment: nunca aceitar objetos inteiros sem filtragem (allowlist) |
| BE-VAL-04 | O | Não confiar em dados de APIs externas |
| BE-VAL-05 | O | Encoding de saída contextual; esquemas de validação reutilizáveis |

> Sanitizar não substitui validar; validar não substitui parametrizar queries; nenhuma substitui
> autorização.

## BE-FILE — Uploads e ficheiros

| ID | Nível | Regra |
|---|---|---|
| BE-FILE-01 | O | Verificar extensão, MIME **e** assinatura real (magic bytes); nunca confiar só no MIME |
| BE-FILE-02 | O | Tamanho máximo, dimensões máximas para imagens, limites de upload por utilizador |
| BE-FILE-03 | O | Nomes internos aleatórios; nunca usar o nome enviado como caminho; prevenir Path Traversal |
| BE-FILE-04 | O | Armazenar fora da pasta pública executável; impedir execução; bloquear extensões perigosas |
| BE-FILE-05 | O | Armazenamento privado por defeito; autorização no download; URLs assinados com validade limitada |
| BE-FILE-06 | R | Antivírus/equivalente; proteção contra ZIP bombs |

## BE-CRYPTO — Criptografia e segredos

| ID | Nível | Regra |
|---|---|---|
| BE-CRYPTO-01 | O | TLS moderno; algoritmos reconhecidos; **nunca** criptografia própria |
| BE-CRYPTO-02 | O | Chaves privadas e segredos nunca no código; usar Secret Manager ou equivalente |
| BE-CRYPTO-03 | O | Segredos não partilhados entre ambientes, rotacionados, revogados se comprometidos; nunca em logs |
| BE-CRYPTO-04 | O | Geração de números aleatórios criptograficamente segura |
| BE-CRYPTO-05 | R | Gerir certificados e renovações; documentar recuperação de chaves; acesso às chaves restrito |

## BE-INFRA — Infraestrutura e ambientes

| ID | Nível | Regra |
|---|---|---|
| BE-INFRA-01 | O | Firewall; portas e serviços desnecessários desativados; sistema e runtimes atualizados |
| BE-INFRA-02 | O | SSH seguro, sem login direto como root; MFA em acesso administrativo; aplicações sem privilégios elevados |
| BE-INFRA-03 | R | Hardening; proteção contra malware; separar app e BD quando justificável |
| BE-INFRA-04 | O | Ambientes independentes: Development, Testing, Staging, Production |
| BE-INFRA-05 | O | Sem dados reais de clientes em desenvolvimento sem necessidade e proteção; sem credenciais de produção em testes; testes nunca alteram produção |
| BE-INFRA-06 | O | Ambientes de desenvolvimento não expostos sem proteção; alterações validadas em staging antes de produção |

## BE-LOG — Logs, monitorização e auditoria

| ID | Nível | Regra |
|---|---|---|
| BE-LOG-01 | O | Registar: autenticações e falhas, alterações de permissões/configurações críticas, criação/eliminação de contas, operações administrativas, exportações, ações financeiras, erros relevantes, atividade suspeita, eventos de integração |
| BE-LOG-02 | O | **Nunca** registar: passwords, tokens, API keys, secrets, cartões completos, headers de autenticação, dados pessoais desnecessários |
| BE-LOG-03 | O | Auditoria responde a: quem, quando, de onde, que recurso, estado anterior, novo estado, resultado |
| BE-LOG-04 | O | Logs de auditoria com acesso restrito e proteção contra alteração |
| BE-LOG-05 | R | Monitorizar disponibilidade, erros, latência, falhas de auth, uso anormal de APIs, recursos; alertas; retenção definida |
| BE-LOG-06 | A | Rastreamento distribuído quando necessário |

## BE-ERR — Erros e exceções

| ID | Nível | Regra |
|---|---|---|
| BE-ERR-01 | O | Nunca devolver stack traces, caminhos internos, queries SQL ou credenciais em erros |
| BE-ERR-02 | O | Códigos HTTP adequados; respostas de erro padronizadas; separar mensagem técnica da apresentada |
| BE-ERR-03 | O | Detalhe técnico só em logs protegidos; não ignorar nem engolir exceções |

Exemplo: `{ "success": false, "message": "Não foi possível concluir a operação.", "error_code": "RESOURCE_NOT_AVAILABLE", "request_id": "req_abc123" }`

## BE-TENANT — Multi-tenant

| ID | Nível | Regra |
|---|---|---|
| BE-TENANT-01 | O | Cada organização com `tenant_id`; todas as consultas respeitam o tenant |
| BE-TENANT-02 | O | **Nunca** confiar no `tenant_id` do Front-end — derivar/validar pela identidade autenticada |
| BE-TENANT-03 | O | Isolamento em relatórios, ficheiros, cache, filas, pesquisas, exports e operações administrativas |
| BE-TENANT-04 | O | Testar acessos cruzados entre tenants; auditoria por organização |
| BE-TENANT-05 | R | Procedimentos definidos para transferência de dados entre organizações |

## BE-INT — Integrações externas

| ID | Nível | Regra |
|---|---|---|
| BE-INT-01 | O | Validar certificados TLS; timeouts; retries controlados; circuit breakers quando apropriado |
| BE-INT-02 | O | Validar respostas externas; não confiar nos dados recebidos |
| BE-INT-03 | O | Webhooks com assinatura criptográfica; validar timestamps; prevenir replay |
| BE-INT-04 | O | Idempotência em operações críticas; limitar destinos; prevenir SSRF |
| BE-INT-05 | O | Credenciais em Secret Manager, com permissões mínimas; nunca expostas |
| BE-INT-06 | R | Registar falhas; gerir versões das integrações |

## BE-PAY — Pagamentos e operações críticas (quando aplicável)

| ID | Nível | Regra |
|---|---|---|
| BE-PAY-01 | O | Fornecedores de pagamento certificados; não guardar dados de cartão sem necessidade e enquadramento |
| BE-PAY-02 | O | Confirmar pagamentos **no servidor**; verificar assinatura dos webhooks; nunca confiar no Front-end para confirmar transações |
| BE-PAY-03 | O | Validar valor, moeda e destinatário no servidor; idempotência; impedir processamento duplicado |
| BE-PAY-04 | O | Registar operações financeiras; histórico de alterações; reconciliação |
| BE-PAY-05 | A | Confirmação adicional para operações sensíveis; segregação de funções; limites transacionais; antifraude proporcional ao risco |

## BE-TEST — Testes, CI/CD e qualidade

| ID | Nível | Regra |
|---|---|---|
| BE-TEST-01 | O | Estratégia de testes: unitários, integração, funcionais, API, segurança, base de dados, regressão |
| BE-TEST-02 | O | **Testes de autorização** (incl. BOLA/IDOR e acessos cruzados entre tenants) |
| BE-TEST-03 | R | Carga e stress; penetration testing proporcional ao risco |
| BE-TEST-04 | R | SAST, DAST, SCA, secret scanning, container scanning, IaC scanning, SBOM |
| BE-TEST-05 | O | CI/CD antes de produção: formatação, lint, análise estática, testes unitários e de integração, análise de dependências, deteção de secrets, testes de segurança, build, aprovação de deploy |
| BE-TEST-06 | O | Nenhuma falha crítica conhecida é ignorada só para acelerar uma publicação |

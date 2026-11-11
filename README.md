# Secure Task Orchestrator

> Uma skill para [Claude Code](https://claude.com/claude-code) que **decompõe pedidos em subtarefas**,
> **roteia cada uma para o modelo certo**, **impõe regras de segurança e qualidade** em todo o
> trabalho de programação e **termina com uma auditoria feita pelo modelo mais poderoso disponível**.

*(English summary at the [bottom](#english-summary).)*

---

## O problema

Pedidos compostos ("cria um endpoint de login e o formulário", "refatora e documenta") costumam ser
resolvidos todos pelo mesmo modelo, o que é **caro** quando é o mais forte e **arriscado** quando é o
mais fraco. Além disso, a segurança e a qualidade acabam muitas vezes por ser uma reflexão tardia.

## O que a skill faz

| # | Passo | Resultado |
|---|---|---|
| 0 | **Perfil do projeto** | Fixa o nível de exigência (ASVS 1 / 2 / 3) |
| 1 | Análise de contexto | Domínio, outputs, dependências, risco |
| 2 | **Decomposição** | Subtarefas atómicas, com plano visível antes de executar |
| 3 | **Classificação Front / Back** | Cada subtarefa de código recebe a(s) sua(s) camada(s) e a marca 🔒 se for sensível |
| 4 | **Atribuição de modelo** | `haiku` / `sonnet` / `opus`, com promoção automática por risco |
| 5 | **Despacho** | Cada subtarefa corre num subagente com o `model` escolhido; as regras vão no prompt |
| 6 | **Gate de regras** | O orquestrador rejeita trabalho que viole regras OBRIGATÓRIAS |
| 7 | Síntese | Resultado consolidado |
| 8 | **Auditoria final** | O modelo mais poderoso disponível (`fable` → `opus`) audita *tudo*, em modo só de leitura |

```
Pedido ─▶ Decompor ─▶ Classificar (FRONT/BACK/…) ─▶ Atribuir modelo ─▶ Subagentes ─▶ Gate
                                                                                      │
                                       Veredito ◀── Auditoria (modelo mais forte) ◀───┘
                                     ✅ / ⚠️ / ❌     └─ ❌ ⇒ correção ⇒ re-auditar (máx. 2×)
```

### Modo audit-only (projetos já existentes)

Para auditar um projeto **sem o alterar**:

```text
> /secure-task-orchestrator audit ./minha-app --nivel 2
```

Fluxo: mapeamento paralelo e barato (`haiku`) → classificação por camada → auditoria read-only por
camada (`sonnet`/`opus`) → validação por `fable`/`opus`, que elimina falsos positivos e procura
falhas entre camadas → **relatório com gravidades, cobertura e plano de correção ordenado**.

- Não edita ficheiros nem executa a aplicação.
- Nunca reproduz segredos encontrados (mascara-os e recomenda rotação).
- Declara a cobertura real e separa o que é verificável por leitura (❌) do que exige execução (❓).
- No fim, oferece passar à correção; só avança com a tua confirmação.

### Regras do início ao fim

Em tarefas de código, as regras não são uma verificação final — acompanham o projeto desde o
desenho até à entrega. Estão em [`references/`](references/) com **IDs citáveis** (ex.: `BE-AUTHZ-04`)
e três níveis de obrigação: **Obrigatório**, **Recomendado**, **Avançado**.

| Ficheiro | Conteúdo |
|---|---|
| [`backend-rules.md`](references/backend-rules.md) | Arquitetura, autenticação, autorização, APIs, CORS, BD, validação, uploads, criptografia, infraestrutura, logs, erros, multi-tenant, integrações, pagamentos, testes/CI |
| [`frontend-rules.md`](references/frontend-rules.md) | Arquitetura, segurança no navegador, rotas, formulários, UI/UX, avaliação heurística de Nielsen, acessibilidade (WCAG 2.2 AA), i18n, dados na interface, mobile (MASVS) |
| [`cross-cutting.md`](references/cross-cutting.md) | Privacidade/RGPD, Git, dependências, incidentes, documentação, IA/LLMs, normas |
| [`audit-checklist.md`](references/audit-checklist.md) | Checklist de produção e escala de gravidade usada pelo auditor |

**Princípio fundamental:** nunca confiar exclusivamente no Front-end. Autorização, integridade e
validação crítica vivem no Back-end.

### Gate de produção

> Nenhum sistema está pronto para produção enquanto existirem vulnerabilidades críticas ou altas por
> resolver, controlos obrigatórios por implementar ou falhas de autorização que comprometam dados e
> operações. Cada exceção exige justificação, responsável, medidas compensatórias e prazo.

## Instalação

A skill é uma pasta com um `SKILL.md`. Clona-a para a pasta de skills do Claude Code:

```bash
# Para todos os projetos (nível utilizador)
git clone https://github.com/Silva745/secure-task-orchestrator.git ~/.claude/skills/secure-task-orchestrator

# Ou só para um projeto
git clone https://github.com/Silva745/secure-task-orchestrator.git .claude/skills/secure-task-orchestrator
```

No Windows (PowerShell), `~` equivale a `$env:USERPROFILE`.

Reinicia a sessão do Claude Code. A skill ativa-se sozinha em pedidos com vários passos, ou podes
invocá-la explicitamente com `/secure-task-orchestrator`.

## Utilização

```text
> cria um endpoint de login com MFA e o respetivo formulário
```

A skill apresenta primeiro o plano:

```
📋 Plano de execução   (nível de exigência: 2 — inferido: dados pessoais)

1. Mapear stack e convenções        → Explore         · haiku  · paralelo       · —
2. Desenhar fluxo de autenticação   → Plan            · opus   · depende de 1   · BACK 🔒
3. Endpoint, hashing, rate limiting → general-purpose · opus   · depende de 2   · BACK 🔒
4. Formulário, estados, a11y        → general-purpose · sonnet · depende de 2   · FRONT
5. Testes de auth e autorização     → general-purpose · sonnet · depende de 3,4 · BACK
6. Auditoria final                  → general-purpose · fable  · depende de 1-5
```

## Limitações (a ler antes de confiar)

- O roteamento funciona via **subagentes**: a sessão principal não troca de modelo a meio.
- A auditoria é **revisão por um modelo**. **Não substitui** testes executados, SAST/DAST/SCA,
  secret scanning, auditoria independente nem teste de intrusão. O relatório indica o que ficou
  apenas "verificado por leitura".
- A disponibilidade do alias `fable` depende da tua conta; sem ele, o auditor cai para `opus`.
- As regras são uma **base de referência**, não uma garantia de conformidade legal (RGPD, PCI-DSS…).
  Ajusta ao teu enquadramento.
- Isto é uma política de engenharia, não aconselhamento jurídico.

## Personalizar

- **Regras:** edita os ficheiros em `references/`. Mantém os IDs estáveis — a auditoria cita-os.
- **Tabela de modelos / promoções:** `SKILL.md`, Passo 4.
- **Auditor:** `SKILL.md`, Passo 8 (ordem de preferência de modelos).
- **Idioma:** a skill está em português europeu. Traduções são bem-vindas (ver [CONTRIBUTING](CONTRIBUTING.md)).

## Estrutura

```
secure-task-orchestrator/
├── SKILL.md                    # Definição da skill (modo execução + modo audit-only)
├── references/
│   ├── backend-rules.md
│   ├── frontend-rules.md
│   ├── cross-cutting.md
│   └── audit-checklist.md
├── README.md
├── CONTRIBUTING.md
├── CHANGELOG.md
└── LICENSE                     # MIT
```

## Contribuir

Issues e PRs são bem-vindos — ver [CONTRIBUTING.md](CONTRIBUTING.md).

## Licença

[MIT](LICENSE).

---

## English summary

**Secure Task Orchestrator** is a Claude Code skill that (1) splits a request into atomic subtasks,
(2) routes each to the cheapest adequate Claude model via subagents, (3) for coding tasks, tags each
subtask as FRONT / BACK / cross-cutting and enforces a security & quality rulebook (OWASP ASVS / API
Top 10 / MASVS, WCAG 2.2, Nielsen heuristics) from design to delivery, and (4) finishes with a
read-only audit by the most capable model available (`fable`, falling back to `opus`). Critical/high
findings block the "ready for production" verdict and trigger up to two fix-and-reaudit cycles.

The skill text and rule files are currently in European Portuguese; translations are welcome.
Install by cloning into `~/.claude/skills/secure-task-orchestrator`.
**Caveat:** the audit is a model review — it does not replace real testing, SAST/DAST, or a
penetration test.

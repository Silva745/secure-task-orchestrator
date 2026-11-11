---
name: secure-task-orchestrator
description: >
  Decompõe qualquer instrução em subtarefas, atribui cada uma ao modelo Claude mais adequado pelo
  custo/complexidade, aplica regras obrigatórias de segurança e qualidade (Back-end, Front-end, UI/UX,
  acessibilidade, mobile, IA) a todo o trabalho de programação, e termina com uma auditoria completa
  feita pelo modelo mais poderoso disponível. Usa este skill SEMPRE que o utilizador fornecer uma
  tarefa, pedido ou instrução que possa ser dividida em partes — mesmo sem pedir otimização de
  modelos. Aplica também em prompts compostos como "analisa este ficheiro e cria um relatório",
  "pesquisa sobre X e resume", "refatora este código e documenta", "cria uma plataforma/API/app", ou
  qualquer pedido com mais do que um passo implícito. Tem também o modo AUDIT-ONLY: usa-o quando o
  utilizador pedir para auditar, rever ou verificar a segurança/qualidade de um projeto já existente
  sem o alterar ("audita este projeto", "revê a segurança disto", "audit-only").
---

# Secure Task Orchestrator

Orquestra o trabalho em quatro eixos:

1. **Decompõe** a instrução em subtarefas atómicas.
2. **Roteia** cada subtarefa para o modelo mais barato que a resolve bem.
3. **Aplica regras de engenharia** (segurança, qualidade, UX) a qualquer trabalho de programação —
   desde o desenho inicial até à entrega.
4. **Audita** o trabalho completo no fim com o modelo mais poderoso disponível.

> **Mecanismo de execução:** o roteamento acontece através de **subagentes**. A sessão principal
> corre num único modelo e não troca de modelo a meio. Cada subtarefa é despachada com a tool
> `Agent`, passando `model` (`haiku` | `sonnet` | `opus` | `fable`, conforme disponível). A sessão
> principal é o **orquestrador**: planeia, despacha, recolhe outputs, aplica o gate de regras e
> sintetiza. Os subagentes **começam sem contexto** — tudo o que precisam (incluindo as regras
> aplicáveis) tem de ir no `prompt`.

## Dois modos

| Modo | Quando | O que faz |
|---|---|---|
| **Execução** (por defeito) | O utilizador pede para criar, alterar, refatorar, pesquisar, escrever... | Passos 0–8 abaixo |
| **Audit-only** | O utilizador pede para auditar/rever um projeto **sem o alterar** | [Modo audit-only](#modo-audit-only--auditar-um-projeto-existente) — read-only, termina em relatório |

Se o pedido for ambíguo ("vê se isto está seguro"), assume **audit-only** (é o modo que não
altera nada) e oferece passar à correção no fim.

## Fluxo geral (modo execução)

```
Passo 0  Perfil do projeto (só código)     → nível de exigência
Passo 1  Análise de contexto
Passo 2  Decomposição em subtarefas
Passo 3  Classificação Front / Back / Transversal  (só código)  → regras aplicáveis
Passo 4  Atribuição de modelo (com promoção por risco)
Passo 5  Despacho para subagentes (regras injetadas no prompt)
Passo 6  Gate de regras por subtarefa
Passo 7  Síntese
Passo 8  Auditoria final com o modelo mais poderoso
```

---

## Passo 0 — Perfil do projeto (apenas tarefas de programação)

Antes de planear, fixa o **nível de exigência** (alinhado com OWASP ASVS). Se o utilizador não
indicou, infere pelos dados tratados e pela exposição, e declara a inferência no plano.

| Nível | Aplicação | Referência |
|---|---|---|
| 1 | Websites e aplicações simples | ASVS L1 |
| 2 | SaaS, plataformas empresariais, dados pessoais | ASVS L2 |
| 3 | Sistemas financeiros, críticos, de elevada sensibilidade | ASVS L3 |

Cada regra nos ficheiros de `references/` está marcada com um destes níveis de obrigação:

- **OBRIGATÓRIO** — tem de estar cumprido antes de produção, em qualquer nível.
- **RECOMENDADO** — adotar sempre que aplicável.
- **AVANÇADO** — exigido sobretudo no Nível 3 ou em sistemas de maior escala/exposição.

**Princípio fundamental:** nunca confiar exclusivamente no Front-end. Toda a segurança,
autorização, integridade e validação crítica é garantida no Back-end, mesmo que também exista
validação no Front-end.

---

## Passo 1 — Análise de contexto

Identifica:

- **Domínio**: código, escrita, dados, investigação, criatividade, configuração...
- **Outputs esperados**: ficheiro, texto, plano, análise, resposta direta...
- **Dependências**: alguma subtarefa precisa do resultado de outra?
- **Risco de erro**: que impacto tem se esta subtarefa falhar?

---

## Passo 2 — Decomposição em subtarefas

Cada subtarefa deve ser **atómica**, **verificável** e **independente** sempre que possível.
Apresenta o plano ao utilizador antes de executar, indicando subagente, modelo, paralelismo e (para
código) a **camada**:

```
📋 Plano de execução   (nível de exigência: 2 — inferido: SaaS com dados pessoais)

1. [Subtarefa A] → Explore          · haiku  · paralelo        · camada: —
2. [Subtarefa B] → general-purpose  · sonnet · paralelo        · camada: BACK
3. [Subtarefa C] → general-purpose  · opus   · depende de 2    · camada: BACK 🔒
4. [Subtarefa D] → general-purpose  · sonnet · depende de 2    · camada: FRONT
5. [Auditoria final] → general-purpose · modelo mais poderoso  · depende de todas
```

### Escolha do `subagent_type`

| Natureza da subtarefa | `subagent_type` | Notas |
|---|---|---|
| Busca/leitura em muitos ficheiros, só conclusão | `Explore` | Read-only |
| Desenho de plano de implementação | `Plan` | Read-only, devolve plano |
| Trabalho com escrita/edição/execução | `general-purpose` | Todas as tools |
| Não encaixa nas anteriores | `claude` | Catch-all |

---

## Passo 3 — Classificação Front-end / Back-end (apenas programação)

Depois da decomposição, **etiqueta cada subtarefa de código** com uma ou mais camadas. Esta
classificação decide que regras são injetadas no subagente.

| Camada | Reconhecer por | Ficheiro de regras |
|---|---|---|
| **BACK** | APIs, servidor, base de dados, autenticação, autorização, jobs/filas, ficheiros/uploads, integrações, pagamentos, infraestrutura, CI/CD, logs | [`references/backend-rules.md`](references/backend-rules.md) |
| **FRONT** | Componentes, páginas, rotas, formulários, estado, estilos, UI/UX, acessibilidade, i18n, armazenamento no navegador | [`references/frontend-rules.md`](references/frontend-rules.md) |
| **MOBILE** | Android/iOS, deep links, keystore/keychain (também carrega FRONT) | `frontend-rules.md` §Mobile |
| **IA** | Integrações com LLMs, agentes, tools, prompts, RAG | `cross-cutting.md` §IA |
| **TRANSVERSAL** | Git, dependências, privacidade/RGPD, incidentes, documentação | [`references/cross-cutting.md`](references/cross-cutting.md) |

Regras da classificação:

- Uma subtarefa que toca nas duas camadas (ex.: "formulário de login + endpoint") deve ser
  **dividida** em subtarefas BACK e FRONT sempre que possível. Se não der, leva as duas camadas.
- **Transversal aplica-se sempre** a qualquer subtarefa de código.
- Marca com 🔒 as subtarefas **sensíveis**: autenticação, autorização, sessões, pagamentos, dados
  pessoais, uploads, segredos, multi-tenant, integrações externas, execução de tools por IA.
- Tarefas sem código (escrita, investigação, análise de dados) **saltam este passo**.

### Regras aplicam-se do início ao fim

As regras não são uma verificação final — fazem parte do trabalho desde o primeiro passo:

| Fase | O que aplicar |
|---|---|
| Desenho / planeamento | Princípios de arquitetura, modelo de ameaças simples, fronteiras de confiança, escolha do nível ASVS |
| Implementação | Regras da camada injetadas no prompt de cada subagente |
| Testes | Estratégia de testes e testes de autorização/segurança |
| Revisão | Gate por subtarefa (Passo 6) |
| Entrega | Auditoria final + checklist de produção (Passo 8) |

---

## Passo 4 — Atribuição de modelo

### Tabela de decisão

| Tipo de subtarefa | Modelo | Critério |
|---|---|---|
| Classificação, triagem, extração simples | **haiku** | Sem raciocínio, resposta estruturada |
| Formatação, conversão, sumário direto | **haiku** | Template claro, pouca ambiguidade |
| Código simples / boilerplate | **haiku** | Padrões conhecidos, sem lógica complexa |
| Análise de texto, sumário longo, Q&A | **sonnet** | Compreensão sem raciocínio profundo |
| Geração de código funcional | **sonnet** | Lógica moderada, debugging possível |
| Planeamento, estruturação de projeto | **sonnet** | Trade-offs simples |
| Raciocínio multi-passo, arquitetura | **opus** | Alta ambiguidade, muitos trade-offs |
| Debugging complexo, refatoração profunda | **opus** | Contexto global e inferência |
| Criatividade com restrições complexas | **opus** | Coerência e originalidade em simultâneo |
| Decisões com alto risco de erro | **opus** | Impacto elevado se errar |

> Usa os **aliases** (`haiku`, `sonnet`, `opus`, `fable`) em vez de IDs com versão — assim a skill
> não envelhece quando saem modelos novos.

### Promoção automática

Sobe um nível se qualquer condição for verdadeira:

- A subtarefa depende de output de outra subtarefa complexa
- O utilizador disse "importante", "crítico", "não posso errar aqui"
- O domínio mistura código + dados + lógica de negócio
- O resultado alimenta diretamente o utilizador final
- **A subtarefa está marcada 🔒 (sensível)** → **mínimo `sonnet`; `opus` para autenticação,
  autorização, pagamentos e multi-tenant**
- O nível de exigência do projeto é 3

### Rebaixamento

Desce para um modelo mais leve se existe template/exemplo claro, a tarefa é repetida, ou o
resultado será verificado manualmente — **exceto** subtarefas 🔒, que nunca descem do mínimo acima.

---

## Passo 5 — Despacho para subagentes

Chama `Agent` com:

- `subagent_type` — conforme a tabela do Passo 2
- `model` — resultado do Passo 4
- `description` — 3-5 palavras
- `prompt` — instrução **completa e autocontida**, contendo:
  1. O objetivo e os critérios de aceitação da subtarefa
  2. O output relevante das subtarefas de que depende
  3. **Para código:** o nível de exigência, a camada, e a instrução
     *"Lê `<caminho>/references/<ficheiro>.md` e cumpre todas as regras OBRIGATÓRIO aplicáveis
     (e as RECOMENDADO/AVANÇADO conforme o nível). Devolve, no fim, uma lista das regras (por ID)
     que a tua solução cumpre, das que não se aplicam e das que ficaram por cumprir com motivo."*
  4. Para 🔒: *"Esta subtarefa é sensível — trata qualquer dúvida de segurança pela opção mais
     restritiva e sinaliza-a explicitamente."*

Se o caminho da skill não for acessível ao subagente, cola no prompt as secções relevantes das
regras.

### Paralelo vs. sequencial

- Independentes → várias chamadas `Agent` **na mesma mensagem**.
- Dependentes → espera o output e injeta-o no `prompt` seguinte.
- `run_in_background: true` para subtarefas longas que não bloqueiam o resto.

Para cada subtarefa concluída regista: `subagent_type` + `model`, resumo do output, ajustes ao plano.

---

## Passo 6 — Gate de regras por subtarefa (apenas código)

Quando uma subtarefa de código devolve, o orquestrador verifica o relatório de regras **antes** de a
aceitar:

- Há regras OBRIGATÓRIO por cumprir sem justificação? → devolve ao subagente (ou promove o modelo)
  com a lista exata de IDs em falta.
- Alguma verificação só no Front-end que devia estar também no Back-end? → cria subtarefa BACK.
- Existem segredos, credenciais ou dados sensíveis no código/logs? → rejeita e corrige de imediato.

Limita a 2 devoluções por subtarefa; se persistir, regista como **exceção pendente** para a auditoria.

---

## Passo 7 — Síntese

Consolida o resultado. Não repitas informação que já é clara pelo encadeamento. Para código, junta
a lista de ficheiros alterados, as decisões de arquitetura relevantes e as exceções pendentes.

---

## Passo 8 — Auditoria final com o modelo mais poderoso

**Obrigatória em todos os pedidos decompostos em 2 ou mais subtarefas.** Para um pedido trivial
(uma única subtarefa simples), a auditoria pode ser omitida, e declaras isso no plano.

### Escolha do auditor

1. Usa `model: "fable"` se estiver disponível na sessão (o mais capaz).
2. Caso contrário, `model: "opus"`.
3. Se o `Agent` rejeitar o alias, tenta o seguinte da lista e regista qual foi usado.

O auditor deve ser **um modelo mais capaz ou igual** a qualquer um dos que produziram o trabalho —
nunca um mais fraco. Usa um subagente **novo** (sem contexto dos produtores) para evitar viés de
confirmação, `subagent_type: general-purpose` e instruções de **só leitura** (não corrige, só
reporta).

### O que o auditor recebe

- O pedido original do utilizador, palavra por palavra
- O plano de execução e o nível de exigência
- O output / diff / lista de ficheiros de **todas** as subtarefas
- As exceções pendentes do Passo 6
- Para código: o caminho de `references/` e [`references/audit-checklist.md`](references/audit-checklist.md)

### O que o auditor verifica

**Qualquer tarefa:**
1. **Completude** — o pedido foi cumprido na íntegra? Falta algum requisito?
2. **Correção** — o conteúdo está certo? Há contradições entre subtarefas?
3. **Coerência** — as partes encaixam entre si? O tom/formato é consistente?
4. **Alucinações** — afirmações sem base, caminhos/APIs/números inventados.

**Código (adicionalmente):**
5. Checklists de produção Back-end e Front-end (`audit-checklist.md`), item a item, citando IDs.
6. Cada regra OBRIGATÓRIO da camada: cumprida / não aplicável (com motivo) / violada.
7. Validação crítica presente no servidor, não só no cliente.
8. Segredos, credenciais e dados sensíveis em código, logs, bundles públicos, variáveis `VITE_*` /
   `NEXT_PUBLIC_*`.
9. Autorização por recurso (BOLA/IDOR), isolamento de tenant, default deny.

### Formato do relatório de auditoria

```
🔍 Auditoria final — modelo: <fable|opus> — nível de exigência: <1|2|3>

Veredito: ✅ APROVADO | ⚠️ APROVADO COM RESERVAS | ❌ REPROVADO

Resumo (3-5 linhas)

Problemas encontrados
| ID  | Gravidade | Camada | Regra violada | Onde | Descrição | Correção recomendada |
|-----|-----------|--------|---------------|------|-----------|----------------------|
| A-1 | Crítico   | BACK   | BE-AUTHZ-04   | ...  | ...       | ...                  |

Regras OBRIGATÓRIO não verificáveis (precisam de teste/execução real): ...
Exceções aceites (justificação · responsável · medida compensatória · prazo): ...
```

Gravidades: **Crítico** · **Alto** · **Médio** · **Baixo**.

### Gate de aprovação e ciclo de correção

> **Nenhum sistema é "pronto para produção" enquanto existirem vulnerabilidades críticas ou altas
> por resolver, controlos OBRIGATÓRIO por implementar, ou falhas de autorização que comprometam
> dados e operações.**

- **Crítico/Alto** → o veredito é ❌. O orquestrador abre subtarefas de correção (com o modelo
  adequado, promovido se necessário), e **volta a auditar apenas o que mudou**. Máximo de **2 ciclos**
  de correção; depois disso, reporta ao utilizador o estado real e o que falta, sem declarar sucesso.
- **Médio/Baixo** → ⚠️ aprovado com reservas; listados no relatório final.
- **Exceções** só são aceites com: justificação documentada, responsável pela aprovação, medidas
  compensatórias e prazo de resolução. Nunca silenciosamente.

### Limites honestos (diz sempre ao utilizador)

A auditoria é **revisão de código/artefactos por um modelo**. Não substitui testes executados, SAST/
DAST/SCA, scanning de segredos, auditoria técnica independente nem teste de intrusão. O relatório
final deve dizer o que foi **verificado por leitura** e o que **só pode ser confirmado em execução**.

---

## Modo audit-only — auditar um projeto existente

Auditoria **só de leitura** a um projeto já criado. Usa as mesmas regras (`references/`), os mesmos
níveis de exigência e a mesma escala de gravidade do Passo 8, mas **não altera nenhum ficheiro**.

**Garantias do modo (diz-as ao utilizador no início):**

- Não edita, cria nem apaga ficheiros do projeto; o único output é o relatório.
- Não executa a aplicação, testes, migrações nem comandos com efeitos laterais, salvo pedido
  explícito. Leituras (ler ficheiros, `grep`, `git log`) são permitidas.
- **Nunca reproduz segredos encontrados**: mostra só ficheiro, linha, tipo e os primeiros/últimos 2–4
  caracteres mascarados (ex.: `sk-…a9f2`). Recomenda sempre rotação da credencial.
- Não faz pedidos de rede ao sistema auditado (sem scans/fuzzing/ataques); isso é teste de intrusão
  e requer autorização própria.

**Invocação:** `/secure-task-orchestrator audit [caminho] [--nivel 1|2|3]`, ou linguagem natural.
Sem caminho, audita o diretório de trabalho.

### A0 — Âmbito e perfil
Fixa o caminho, a stack (inferida de `package.json`, `pyproject.toml`, `composer.json`, `pom.xml`,
`Dockerfile`, etc.) e o **nível de exigência** (Passo 0). Declara no relatório o que ficou **fora
de âmbito** (ex.: infraestrutura não presente no repositório).

### A1 — Mapeamento (paralelo, barato)
Despacha subagentes `Explore` com `model: haiku`, **em paralelo**, cada um com uma pergunta:

1. Estrutura, stack, ponto de entrada, dimensão (nº de ficheiros/linhas por pasta)
2. Configuração e segredos: `.env*`, ficheiros de config, `.gitignore`, histórico recente
   (`git log` por ficheiros sensíveis), variáveis `VITE_*`/`NEXT_PUBLIC_*`
3. Superfície de ataque do Back-end: rotas/endpoints, middlewares, autenticação, autorização
4. Dados: modelos, migrations, queries (SQL cru, concatenação), uploads, integrações externas
5. Front-end: rotas, formulários, armazenamento no navegador, `innerHTML`/`dangerouslySetInnerHTML`
6. Dependências: manifests e lockfiles; pipelines CI/CD; testes existentes

### A2 — Classificação
Agrupa os achados do mapeamento por camada (**BACK / FRONT / MOBILE / IA / TRANSVERSAL**) e marca 🔒
as áreas sensíveis (autenticação, autorização, pagamentos, uploads, multi-tenant, segredos).

### A3 — Auditoria por camada (paralelo)
Um subagente `general-purpose` **por camada**, instruído a ser **estritamente read-only**, com:

- `model: sonnet` por defeito; `opus` para camadas com áreas 🔒 ou nível 3
- O mapa do A1, a lista de ficheiros prioritários, o nível de exigência
- A instrução de ler o ficheiro de regras da camada e avaliar **cada regra [O]** (e [R]/[A] conforme
  o nível) como ✅ / ➖ / ❌ / ❓
- O formato de saída: `ID da regra · gravidade · ficheiro:linha · evidência (excerto curto,
  mascarado) · impacto · correção recomendada`
- Regra de honestidade: **só reportar o que viu no código**. Sem evidência → ❓, nunca ❌.

Em projetos grandes, prioriza por risco (autenticação, autorização, pagamentos, uploads, config,
rotas públicas) e **declara a cobertura** (ficheiros lidos vs. total). Não finjas ter lido tudo.

### A4 — Validação pelo modelo mais poderoso
Despacha **um** subagente novo, com o modelo do Passo 8 (`fable` → `opus`), read-only, que recebe
todos os achados e:

1. **Rejeita falsos positivos** (reabre o ficheiro citado e confirma a evidência)
2. **Procura o que as camadas isoladas não veem**: validação só no cliente, autorização em falta
   entre Front e Back, cadeias de falhas, fugas entre tenants
3. **Reclassifica a gravidade** com justificação
4. Confirma o que é ❓ (precisa de execução) e não ❌

### A5 — Relatório e plano de correção

```
🔍 Auditoria de projeto — <nome> — nível de exigência: <1|2|3> — modelo auditor: <fable|opus>

Veredito: ✅ APROVADO | ⚠️ APROVADO COM RESERVAS | ❌ REPROVADO

Âmbito e cobertura: <caminho>, <N> de <M> ficheiros lidos · Fora de âmbito: ...
Resumo (3–5 linhas) · Contagem: Crítico X · Alto Y · Médio Z · Baixo W

Achados
| ID | Gravidade | Camada | Regra | Local (ficheiro:linha) | Descrição | Correção recomendada |

Pontos fortes (o que está bem feito)
Não verificável por leitura (❓): ...

Plano de correção (ordenado por gravidade e dependência)
 1. [Crítico] ...  → camada · esforço estimado (S/M/L) · modelo sugerido
 2. ...
```

Termina sempre com os **limites**: a auditoria é revisão estática por modelo e não substitui
SAST/DAST/SCA, secret scanning real, testes executados nem teste de intrusão.

### A6 — Passagem para correção (opcional)
Oferece, mas **não inicia sem confirmação**: "Queres que corrija os achados Críticos/Altos?". Se
sim, transita para o **modo execução** usando o plano de correção como decomposição (Passos 3–8),
incluindo nova auditoria final. Se o utilizador só quer o relatório, termina aqui.

### Exemplo

```
> /secure-task-orchestrator audit ./minha-app --nivel 2

📋 Plano (audit-only · read-only)
1. Mapear estrutura e stack           → Explore         · haiku  · paralelo
2. Mapear config/segredos             → Explore         · haiku  · paralelo
3. Mapear rotas, auth e dados         → Explore         · haiku  · paralelo
4. Auditar BACK (🔒 auth/dados)       → general-purpose · opus   · depende de 1-3
5. Auditar FRONT                      → general-purpose · sonnet · depende de 1-3
6. Auditar TRANSVERSAL                → general-purpose · sonnet · depende de 1-3
7. Validar achados e relatório final  → general-purpose · fable  · depende de 4-6
```

---

## Exemplos de decomposição

### "Cria um endpoint de login e o formulário de login"

```
Nível: 2 (inferido)

1. Mapear stack e convenções do repo              → Explore         · haiku  · paralelo       · —
2. Desenhar fluxo de autenticação (sessão/MFA)    → Plan            · opus   · depende de 1   · BACK 🔒
3. Implementar endpoint, hashing, rate limiting   → general-purpose · opus   · depende de 2   · BACK 🔒
4. Implementar formulário, estados, acessibilidade→ general-purpose · sonnet · depende de 2   · FRONT
5. Testes de autenticação e de autorização        → general-purpose · sonnet · depende de 3,4 · BACK
6. Auditoria final                                → general-purpose · fable  · depende de 1-5
```

### "Analisa este CSV de vendas e cria um relatório executivo" (sem código de produto)

```
1. Ler e inspecionar estrutura do CSV            → general-purpose · haiku  · paralelo
2. Calcular métricas chave                        → general-purpose · haiku  · depende de 1
3. Identificar insights relevantes                → general-purpose · sonnet · depende de 2
4. Redigir relatório executivo                    → general-purpose · sonnet · depende de 3
5. Auditoria final (completude/correção/números)  → general-purpose · fable  · depende de 1-4
```

### "Refatora este módulo Python e adiciona testes"

```
1. Ler e mapear o código atual                   → Explore         · haiku  · paralelo        · —
2. Identificar problemas e oportunidades         → Plan            · opus   · depende de 1    · BACK
3. Refatorar o código                            → general-purpose · opus   · depende de 2    · BACK
4. Escrever testes unitários                     → general-purpose · sonnet · depende de 3    · BACK
5. Redigir docstrings                            → general-purpose · haiku  · depende de 3    · —
6. Auditoria final                               → general-purpose · fable  · depende de 1-5
```

---

## Notas de custo

- Preços mudam: consulta a página oficial de preços da Anthropic em vez de os fixar aqui.
- Ordem de custo: `haiku` < `sonnet` < `opus` < `fable`. Usa o mais barato que cumpre os critérios.
- A auditoria final é o único passo em que o custo mais alto é aceite por defeito — é onde o risco
  de erro silencioso é maior.
- Batch API e prompt caching reduzem custo em execuções repetidas.

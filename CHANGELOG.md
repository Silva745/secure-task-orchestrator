# Changelog

Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/);
versionamento [SemVer](https://semver.org/lang/pt-BR/).

## [Não lançado]

## [1.0.0] — 2026-11-11

Evolução do `smart-task-router`, agora com o nome `secure-task-orchestrator`.

### Adicionado
- **Passo 0:** perfil do projeto com níveis de exigência alinhados com OWASP ASVS (1 / 2 / 3).
- **Passo 3:** classificação de subtarefas de código em FRONT / BACK / MOBILE / IA / TRANSVERSAL e
  marca 🔒 para subtarefas sensíveis.
- Regras de engenharia com IDs citáveis e níveis Obrigatório / Recomendado / Avançado, em
  `references/` (back-end, front-end, transversais).
- **Passo 6:** gate de regras por subtarefa, com devolução ao subagente.
- **Passo 8:** auditoria final read-only pelo modelo mais poderoso disponível (`fable` → `opus`),
  com relatório estruturado, escala de gravidade, veredito e ciclo de correção (máx. 2×).
- Checklist de auditoria de produção (`references/audit-checklist.md`).
- Documentação open source: README, CONTRIBUTING, LICENSE (MIT).

### Alterado
- Renomeada de `smart-task-router` para `secure-task-orchestrator`.
- Promoção automática de modelo para subtarefas 🔒 (mínimo `sonnet`; `opus` em auth, autorização,
  pagamentos e multi-tenant).
- Uso de aliases de modelo (`haiku`, `sonnet`, `opus`, `fable`) em vez de IDs com versão.
- Removidos preços fixos (ficam desatualizados); a skill aponta para a página oficial.

### Mantido
- Decomposição em subtarefas, atribuição de modelo por custo/complexidade, regras de promoção e
  rebaixamento, despacho paralelo/sequencial por subagentes.

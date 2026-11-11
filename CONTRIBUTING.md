# Contribuir

Obrigado por querer melhorar esta skill! Issues e Pull Requests são bem-vindos.

## O que ajuda mais

- **Corrigir ou clarificar regras** em `references/` (com fonte: OWASP, NIST, WCAG, RGPD…).
- **Novas camadas ou stacks** (ex.: regras específicas para dados/ML, embedded, data engineering).
- **Traduções** do `SKILL.md` e das referências (o original está em português europeu).
- **Casos reais** em que o roteamento de modelos escolheu mal — abre uma issue com o plano gerado.

## Princípios

1. **IDs estáveis.** Cada regra tem um ID (`BE-AUTHZ-04`). Não renumeres; adiciona novos IDs no fim
   da secção. A auditoria e os relatórios citam-nos.
2. **Cada regra é verificável.** Evita "usar boas práticas". Escreve algo que um auditor consiga
   marcar como cumprido / violado.
3. **Níveis honestos.** `[O]` obrigatório, `[R]` recomendado, `[A]` avançado. Não infles o `[O]`.
4. **Sem preços nem IDs de modelos com versão** no texto — envelhecem. Usa aliases.
5. **Sem alegações que a skill não pode cumprir.** A auditoria é revisão por um modelo; não a
   apresentes como teste de intrusão.
6. **Mantém o `SKILL.md` conciso.** O detalhe vive em `references/`.

## Fluxo

1. Faz fork e cria um branch: `feature/…` ou `fix/…`.
2. Altera os ficheiros; atualiza o `CHANGELOG.md` em *Não lançado*.
3. Se alteraste regras, confirma que `audit-checklist.md` continua a referenciar IDs existentes.
4. Abre o PR a explicar **o quê** e **porquê** (e a fonte, se for uma regra de segurança).

## Reportar vulnerabilidades

Se encontrares uma regra que, se seguida, **introduz** uma vulnerabilidade, abre uma issue marcada
como `security`. Não incluas exploits funcionais.

## Código de conduta

Sê respeitoso e construtivo. Críticas visam o texto e as ideias, nunca as pessoas.

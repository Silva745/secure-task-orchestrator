# Checklist de Auditoria Final

Usada pelo auditor (modelo mais poderoso disponível) no Passo 8. Para cada item responde:
**✅ cumprido** · **➖ não aplicável (motivo)** · **❌ violado** · **❓ não verificável por leitura**
(precisa de execução, teste real ou ferramenta). Cita sempre o ID da regra e o local (ficheiro:linha).

> Esta checklist é um controlo interno. **Não substitui** auditoria técnica independente nem teste
> de intrusão.

## Como classificar a gravidade

| Gravidade | Exemplos |
|---|---|
| **Crítico** | Segredo exposto; autenticação/autorização ausente ou contornável; SQL injection; BOLA/IDOR; fuga entre tenants; confirmação de pagamento no cliente |
| **Alto** | Validação crítica só no cliente; password com hash fraco; sem rate limiting em login; CORS permissivo com credenciais; upload sem verificação; dados sensíveis em logs |
| **Médio** | Falta de CSP; estados de erro/vazio em falta; logs de auditoria incompletos; testes de segurança em falta |
| **Baixo** | Nomenclatura inconsistente; documentação incompleta; problemas estéticos |

Qualquer **❌ Crítico/Alto** ⇒ veredito **REPROVADO**.

---

## 1. Verificações universais (qualquer tarefa)

- [ ] O pedido original foi cumprido na íntegra (nenhum requisito esquecido)
- [ ] Sem contradições entre os outputs das subtarefas
- [ ] Sem afirmações inventadas (caminhos, APIs, números, fontes)
- [ ] Formato e tom consistentes com o pedido
- [ ] Exceções pendentes têm justificação, responsável, medida compensatória e prazo

## 2. Back-end (0/38)

**Arquitetura e código**
- [ ] BE-ARCH — arquitetura organizada e documentada
- [ ] BE-CODE — SOLID/Clean Code; nomenclatura consistente
- [ ] BE-CODE-07 — lógica de negócio separada da apresentação
- [ ] BE-ERR — tratamento global de exceções
- [ ] BE-INFRA-04 — configurações separadas por ambiente
- [ ] BE-API-06 — sem funcionalidades de debug em produção

**Autenticação e autorização**
- [ ] BE-PWD — hashing adequado (Argon2id ou equivalente)
- [ ] BE-AUTH / BE-RESET — registo, login e recuperação seguros e testados
- [ ] BE-LOGIN-03 — MFA para acessos privilegiados
- [ ] BE-SESS — sessões seguras e revogáveis
- [ ] BE-AUTHZ-08 — RBAC/ABAC conforme necessário
- [ ] BE-AUTHZ-01 — autorização verificada no servidor
- [ ] BE-AUTHZ-04 / BE-TEST-02 — testes de IDOR/BOLA

**APIs e dados**
- [ ] BE-API-01 — HTTPS obrigatório
- [ ] BE-CORS — CORS configurado corretamente
- [ ] BE-API-03 — rate limiting
- [ ] BE-VAL — validação no servidor
- [ ] Proteção contra SQL Injection e XSS
- [ ] BE-API-02/03 — paginação e limites de payload
- [ ] BE-DB-08/09 — dados sensíveis protegidos
- [ ] BE-DB-04 — integridade referencial

**Infraestrutura**
- [ ] BE-INFRA-01 — firewall; serviços/portas desnecessários desativados
- [ ] BE-CRYPTO-02 — credenciais protegidas
- [ ] BE-INFRA-04 — ambientes isolados
- [ ] BE-DB-10 — backups cifrados
- [ ] BE-DB-11 — restauro testado
- [ ] BE-LOG-05 — monitorização ativa e alertas

**Qualidade e testes**
- [ ] BE-TEST-01 — testes unitários e de integração
- [ ] BE-TEST-02 — testes de autorização
- [ ] BE-TEST-05 — testes de segurança, análise de dependências
- [ ] Secret scanning
- [ ] X-GIT-02 — code review
- [ ] X-INC-01 — plano de resposta a incidentes

## 3. Front-end (0/36)

**Arquitetura**
- [ ] FE-ARCH — componentes reutilizáveis e responsabilidades separadas
- [ ] FE-ARCH-03 — tipagem e validações consistentes
- [ ] Gestão adequada de estados
- [ ] FE-ARCH-02 — tratamento global de erros

**Segurança**
- [ ] FE-SEC-01/02 — nenhuma credencial privada exposta (incluindo `VITE_*`/`NEXT_PUBLIC_*`)
- [ ] FE-SEC-04/05 — tokens geridos de forma segura
- [ ] FE-SEC-08 — proteção contra XSS
- [ ] FE-SEC-09 — CSRF quando aplicável
- [ ] FE-SEC-08/10 — CSP configurada
- [ ] FE-ROUTE-03 — rotas protegidas **e** autorização no servidor
- [ ] FE-DATA — dados privados não expostos
- [ ] FE-SEC-11 — dependências verificadas
- [ ] FE-SEC-12 — sem dados sensíveis em logs do navegador

**UI e UX**
- [ ] FE-UI-01/02 — design system consistente
- [ ] FE-UI-03 — hierarquia visual adequada
- [ ] FE-UX-01 — estados loading / success / error
- [ ] FE-UX-01 — estados vazios (e offline)
- [ ] FE-FORM — formulários com feedback
- [ ] FE-UX-03 — confirmação de operações destrutivas
- [ ] FE-UX-06 — responsividade testada
- [ ] FE-UX-05 — navegação intuitiva
- [ ] FE-UI-05 — sem botões sem funcionalidade

**Acessibilidade e desempenho**
- [ ] FE-A11Y — WCAG 2.2 AA avaliada
- [ ] FE-A11Y-02 — navegação por teclado
- [ ] FE-A11Y-03 — contraste adequado
- [ ] FE-A11Y-05 — compatível com leitores de ecrã
- [ ] FE-UX-07 — imagens e carregamento otimizados
- [ ] FE-A11Y-06 — animações controladas
- [ ] Testado em diferentes dispositivos e navegadores

**Avaliação heurística**
- [ ] FE-HEUR — 10 heurísticas de Nielsen avaliadas
- [ ] Problemas classificados por gravidade (0–4)
- [ ] Fluxos principais testados com utilizadores
- [ ] Erros de usabilidade documentados
- [ ] Problemas críticos corrigidos

## 4. Transversal, mobile e IA (quando aplicável)

- [ ] X-PRIV — privacidade e RGPD
- [ ] X-DEP — cadeia de fornecimento
- [ ] X-DOC — documentação mínima e `.env.example`
- [ ] MOB — requisitos mobile (MASVS)
- [ ] X-AI — requisitos de IA / LLMs

## 5. Veredito

| Condição | Veredito |
|---|---|
| Qualquer ❌ Crítico/Alto, ou OBRIGATÓRIO por cumprir sem exceção válida | ❌ **REPROVADO** |
| Apenas ❌ Médio/Baixo e/ou ❓ por verificar | ⚠️ **APROVADO COM RESERVAS** |
| Tudo ✅/➖ e nenhum ❓ crítico | ✅ **APROVADO** |

Termina sempre com a secção **"O que só pode ser confirmado em execução"** (itens ❓).

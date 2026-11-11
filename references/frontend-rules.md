# Regras de Front-end, UI/UX, Acessibilidade e Mobile

Níveis: **[O]** Obrigatório · **[R]** Recomendado · **[A]** Avançado.

> O Front-end **nunca** é um ambiente confiável: tudo o que chega ao navegador pode ser
> inspecionado, manipulado, copiado ou reproduzido. Toda a regra de segurança tem equivalente no
> Back-end.

---

## FE-ARCH — Arquitetura e organização

| ID | Nível | Regra |
|---|---|---|
| FE-ARCH-01 | O | Separar componentes de negócio de componentes visuais; sem componentes gigantes nem duplicados |
| FE-ARCH-02 | O | Centralizar chamadas à API, tratamento de erros e validações reutilizáveis |
| FE-ARCH-03 | O | Tipagem definida; evitar `any` indiscriminado |
| FE-ARCH-04 | R | Lógica de negócio complexa fora dos componentes; hooks para lógica reutilizável |
| FE-ARCH-05 | R | Evitar estado global desnecessário; tratar estados assíncronos; organizar por domínio (`features/`) |
| FE-ARCH-06 | R | Sem dependências circulares; lazy loading quando adequado; componentes testáveis isoladamente |
| FE-ARCH-07 | R | Estrutura de referência: `app/{providers,routes,layouts}`, `components/{ui,forms,navigation,feedback}`, `features/`, `hooks/`, `services/`, `api/`, `schemas/`, `stores/`, `types/`, `utils/`, `constants/`, `styles/`, `tests/` |

## FE-SEC — Segurança no cliente

### Segredos e credenciais
| ID | Nível | Regra |
|---|---|---|
| FE-SEC-01 | O | **Nunca** no código enviado ao navegador: API keys privadas, secret keys, chaves privadas, credenciais de BD/SMTP/cloud/storage, passwords, tokens admin ou de terceiros, client secrets OAuth, chaves privadas de pagamento, tokens de plataformas de IA, secrets de webhooks |
| FE-SEC-02 | O | Variáveis `VITE_*`, `NEXT_PUBLIC_*` e equivalentes vão para o bundle público — **nunca** segredos |
| FE-SEC-03 | O | Padrão correto: Front-end → Back-end (guarda a credencial) → serviço externo |

Pode estar no Front-end: URL pública da API, ID público de aplicação, chave pública de mapas (com
restrições), chave pública de pagamentos (quando prevista), dados públicos do utilizador, config
pública de UI. **Não** pode: qualquer coisa da lista de FE-SEC-01, dados de outros utilizadores sem
autorização, regras internas confidenciais, dados de auditoria restritos.

### Armazenamento no navegador
| ID | Nível | Regra |
|---|---|---|
| FE-SEC-04 | O | Não usar `localStorage`/`sessionStorage` para passwords, refresh tokens de elevado privilégio, chaves privadas, secrets, dados financeiros sensíveis ou pessoais desnecessários |
| FE-SEC-05 | O | Autenticação por sessão: cookies `HttpOnly`, `Secure`, `SameSite=Lax` (política exata conforme modelo de auth e CSRF) |
| FE-SEC-06 | O | Limpar dados locais após logout; sem tokens nem dados confidenciais em URLs/parâmetros de navegação |
| FE-SEC-07 | R | Expiração de dados locais; sem dados sensíveis em IndexedDB/caches partilhadas sem proteção; não assumir que o dispositivo é inviolável |

### Ataques no navegador
| ID | Nível | Regra |
|---|---|---|
| FE-SEC-08 | O | **XSS:** sem `innerHTML` com dados não confiáveis; escaping contextual; sanitizar HTML só quando indispensável; Content Security Policy; evitar scripts inline |
| FE-SEC-09 | O | **CSRF:** proteção anti-CSRF em operações autenticadas por cookies; `SameSite`; verificar origem em operações sensíveis; GET nunca altera estado; CORS ≠ proteção CSRF |
| FE-SEC-10 | O | **Clickjacking:** CSP `frame-ancestors`; `X-Frame-Options` por compatibilidade |
| FE-SEC-11 | O | **Dependências:** sem scripts desconhecidos nem bibliotecas abandonadas; verificar vulnerabilidades; versões fixadas + lockfile; SRI quando aplicável; controlar analytics/tracking |
| FE-SEC-12 | O | Sem dados sensíveis em logs do navegador, comentários HTML, atributos HTML desnecessários, tooltips ou metadados públicos |

## FE-ROUTE — Rotas e páginas

| ID | Nível | Regra |
|---|---|---|
| FE-ROUTE-01 | O | Rotas públicas e privadas definidas; proteção de navegação; tratar sessões expiradas |
| FE-ROUTE-02 | O | Páginas 403/404/500; redirecionar não autenticados; **evitar open redirects** |
| FE-ROUTE-03 | O | Nunca depender só de guards do Front-end — cada página protegida tem autorização equivalente no Back-end |
| FE-ROUTE-04 | O | Não expor identificadores confidenciais nas rotas |
| FE-ROUTE-05 | R | Permissões visuais: ocultar/desativar ações indisponíveis, atualizar a UI quando as permissões mudam — sabendo que **isto não é segurança** |

## FE-FORM — Formulários e validação

| ID | Nível | Regra |
|---|---|---|
| FE-FORM-01 | O | Validar obrigatórios, emails, telefones, datas, números, comprimentos, formatos, ficheiros (extensão/tamanho) e campos dependentes antes do envio |
| FE-FORM-02 | O | **Toda validação que protege integridade/regras de negócio também existe no servidor** (ex.: limite de 1.000 € alterado nas DevTools para 100.000 € tem de ser rejeitado no Back-end) |
| FE-FORM-03 | O | Impedir submissões duplicadas e múltiplos cliques; mostrar estado de processamento |
| FE-FORM-04 | O | Mensagens claras e junto do campo; identificar visualmente campos inválidos; preservar dados válidos no erro |
| FE-FORM-05 | O | Mensagens sem termos técnicos, stack traces, erros SQL, dados internos ou de terceiros |

## FE-UI — Interface

| ID | Nível | Regra |
|---|---|---|
| FE-UI-01 | R | Design system: cores primárias/secundárias/semânticas, tipografia, espaçamento, border radius, sombras, botões, inputs, tabelas, modais, ícones, estados de interação |
| FE-UI-02 | R | Consistência entre páginas; não misturar linguagens visuais |
| FE-UI-03 | R | Hierarquia visual clara; espaço em branco; sem excesso de informação nem de cores; sem elementos decorativos sem função |
| FE-UI-04 | O | Nunca depender só da cor para transmitir informação; contraste adequado; texto legível |
| FE-UI-05 | O | **Nenhum botão decorativo** — todo o botão executa a ação indicada |
| FE-UI-06 | O | Estados dos componentes: Botão (normal/hover/focus/active/disabled/loading) · Input (normal/focus/filled/error/disabled) · Select · Checkbox · Modal · Tabela (loading/empty/populated/error) · Card · Upload (empty/uploading/success/error) · Notificação (success/warning/error/info) |

## FE-UX — Experiência de utilização

| ID | Nível | Regra |
|---|---|---|
| FE-UX-01 | O | Estados obrigatórios em páginas com dados: **LOADING · SUCCESS · ERROR · EMPTY · DISABLED · OFFLINE**; nunca uma página em branco quando algo falha |
| FE-UX-02 | O | Toda a operação dá feedback (loading, sucesso, erro com orientação, confirmação, progresso) |
| FE-UX-03 | O | Confirmar operações destrutivas (confirmações específicas, não genéricas); permitir cancelar; alertar antes de sair de formulários com alterações por guardar |
| FE-UX-04 | R | Permitir desfazer quando tecnicamente possível; pré-visualização antes de operações críticas |
| FE-UX-05 | R | Navegação intuitiva, localização atual visível, breadcrumbs quando úteis, sem becos sem saída, voltar sem perder trabalho |
| FE-UX-06 | O | Responsivo: smartphones, tablets, portáteis, desktops, ecrãs grandes; sem scroll horizontal desnecessário; áreas de toque adequadas; tabelas/menus/formulários adaptados |
| FE-UX-07 | R | Desempenho: imagens otimizadas e modernas, lazy loading, code splitting, evitar renders desnecessários, cache correta, fontes otimizadas, skeleton loading, sem bloquear a UI |

## FE-HEUR — Avaliação heurística (obrigatória antes do lançamento)

Avaliar as 10 heurísticas de Nielsen:

| # | Heurística | Perguntas-chave |
|---|---|---|
| H1 | Visibilidade do estado do sistema | Há feedback após clicar? Loading? Progresso? Confirmação? Erros? |
| H2 | Correspondência com o mundo real | Terminologia compreensível? Ícones corretos? Datas/formatos regionais? |
| H3 | Controlo e liberdade | Cancelar, voltar, editar, recuperar? Confirmação de ações destrutivas? |
| H4 | Consistência e padrões | Botões, cores, formulários, menus e mensagens consistentes? |
| H5 | Prevenção de erros | Limites, formatos indicados, prevenção de duplicação, erros visíveis antes de submeter? |
| H6 | Reconhecimento em vez de memorização | Opções visíveis, exemplos nos campos, filtros identificados? |
| H7 | Flexibilidade e eficiência | Atalhos, filtros, pesquisa, operações em lote, preferências? |
| H8 | Design estético e minimalista | Elementos desnecessários? Hierarquia clara? Dashboards sobrecarregados? |
| H9 | Recuperação de erros | Mensagens compreensíveis, junto dos campos, com solução; dados preservados? |
| H10 | Ajuda e documentação | Tooltips, FAQs, ajuda contextual, contacto de suporte, orientação inicial? |

Avaliar também: consistência cognitiva, acessibilidade, descoberta, eficiência, satisfação,
previsibilidade, confiança, responsividade, recuperabilidade, inclusividade.

**Escala de gravidade:** 0 sem problema · 1 estético · 2 usabilidade menor · 3 grave · 4 crítico
(impede a utilização).

**Registo de cada problema:** ID · página · componente · heurística violada · descrição · evidência
visual · gravidade · impacto no utilizador · solução recomendada · estado da correção.

## FE-A11Y — Acessibilidade (alvo: WCAG 2.2 AA)

| ID | Nível | Regra |
|---|---|---|
| FE-A11Y-01 | O | HTML semântico; títulos hierárquicos; `label` em todos os campos; controlos com nome acessível |
| FE-A11Y-02 | O | Navegação completa por teclado; foco visível; modais gerem o foco |
| FE-A11Y-03 | O | Contraste adequado; informação nunca só por cor; ampliação de conteúdo permitida |
| FE-A11Y-04 | O | Texto alternativo em imagens relevantes; erros de formulário identificados |
| FE-A11Y-05 | R | Compatível com leitores de ecrã; ARIA só quando necessário e correto |
| FE-A11Y-06 | O | Respeitar `prefers-reduced-motion`; permitir pausar animações; legendas em conteúdo audiovisual |

## FE-I18N — Internacionalização

| ID | Nível | Regra |
|---|---|---|
| FE-I18N-01 | R | Textos em ficheiros de tradução, nunca dentro da lógica dos componentes; idioma padrão definido; troca de idioma |
| FE-I18N-02 | R | Formatos de data, moeda, número e fuso horário por região |
| FE-I18N-03 | R | Layouts que aguentam traduções mais longas; traduzir erros, notificações e emails |

## FE-DATA — Dados apresentados na interface

| ID | Nível | Regra |
|---|---|---|
| FE-DATA-01 | O | Apresentar só os dados necessários à tarefa; mascarar informação sensível |
| FE-DATA-02 | O | Sem dados pessoais em URLs; sem carregar dados de outros utilizadores sem autorização |
| FE-DATA-03 | O | Limpar dados quando a sessão termina; proteger exports; filtros de visibilidade; sem cache privada em locais partilhados |
| FE-DATA-04 | O | Dashboards: unidades, períodos, origem e data de atualização; tratar nulos, loading e ausência de dados; gráficos que não distorcem; respeitar permissões de métricas |
| FE-DATA-05 | O | **Nunca apresentar números falsos ou simulados como dados reais** |

## MOB — Aplicações mobile (Android / iOS) — referência OWASP MASVS

| ID | Nível | Regra |
|---|---|---|
| MOB-01 | O | Android Keystore / iOS Keychain; credenciais e tokens nunca em texto simples nem em ficheiros públicos |
| MOB-02 | O | BD locais protegidas; sem dados sensíveis em logs; considerar backups automáticos do dispositivo |
| MOB-03 | O | HTTPS; **nunca** desativar nem aceitar validação de certificados inválidos; validar respostas do servidor |
| MOB-04 | A | Certificate pinning em apps de elevado risco, com estratégia de atualização e recuperação |
| MOB-05 | O | Debug desativado em produção; **sem secrets no APK/IPA**; proteger configurações de build |
| MOB-06 | O | Proteger deep links; validar universal links e app links; pedir só permissões necessárias |
| MOB-07 | R | Root/jailbreak como sinal de risco (sem depender só da deteção); integridade proporcional à criticidade |
| MOB-08 | R | Sem dados em notificações no ecrã bloqueado; proteger a área de transferência; bloquear screenshots só onde a política justifica |
| MOB-09 | R | Bibliotecas atualizadas; WebSockets protegidos |

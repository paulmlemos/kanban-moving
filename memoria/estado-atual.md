# ESTADO ATUAL — KANBAN

## Como usar este arquivo

Ponto de leitura no início de sessão (ver `PROTOCOLO-SESSAO.md` na raiz do repo). Compacto e
acionável: último estado validado + próximo passo crítico. Histórico completo (tudo antes de
2026-09-18) está em `memoria/historico-kanban.md` — não precisa reler para trabalhar no dia a dia.

Regras de segurança do deploy (Firebase de produção, scp, etc.) estão no `CLAUDE.md` da raiz do
repo — leitura obrigatória antes de qualquer commit que mexa em dado ou deploy.

## Atualização 2026-09-18 — Claude (com Priscila): estrutura de memória própria criada

Origem: Claude (com Priscila)

- O Paul passou a trabalhar diretamente neste repo (`kanban-public`/`paulmlemos/kanban-moving`),
  além da Priscila. Antes, o histórico do Kanban morava só no vault `moving-marketing`
  (`operacao/ferramentas/kanban/memoria/historico-kanban.md`) — funcionava enquanto só a Priscila
  mexia nos dois repos, mas não é garantido que o Paul tenha o vault clonado ao lado deste repo.
- Criada estrutura de memória própria, auto-contida, seguindo o mesmo padrão universal do vault
  (`estado-atual.md` / `decisoes.md` / `erros.md`), sem herdar a complexidade do vault geral
  (chamados-ia, índice de escopos, biblioteca cognitiva — nada disso se aplica a um repo de 2
  operadores e 1 arquivo).
- `historico-kanban.md` (1221 linhas) migrado do vault para `memoria/` deste repo, sem perder
  nenhum bloco. `PROTOCOLO-SESSAO.md` criado na raiz com o gatilho de abertura/fechamento.
- Vault atualizado para apontar pra cá em vez de manter a memória lá (`indice-de-escopos.md`,
  `estado-atual.md` geral do vault, `abertura-de-sessao.md`).
- **Próximo passo crítico:** nenhum pendente desta frente — Paul e Priscila já podem abrir sessão
  neste repo seguindo `PROTOCOLO-SESSAO.md`. Primeira sessão real de trabalho (feature/fix) deve
  testar o fluxo de fechamento ponta a ponta.

## Atualização 2026-09-18 (parte 2) — Claude (com Priscila): contador de edições, sidebar reorganizado, Agenda do Dia e dashboard de Calendários — tudo deployado

Origem: Claude (com Priscila). Primeira sessão real de trabalho neste repo com memória própria —
fluxo de fechamento testado ponta a ponta, conforme previsto no bloco acima.

- **Card "Hoje" da Priscila** ganha contador de edições pendentes do mês (`computeEditsMonthly`,
  campo `createdAt` novo em itens de Edição) — número central + segundo anel de % logo abaixo do
  anel de tarefas. Só aparece no card dela; card do Paul intacto.
- **Sidebar reorganizado** (ordem nova): Início, Gestão de Tarefas, **Agenda do Dia** (nova),
  **Gestão de Redes Sociais** (nova — expande sub-lista com Gestão de Postagens, que é a aba de
  sempre só realocada, e **Calendários**, nova), **Gestão de Tráfego Pago** (novo, placeholder vazio
  — conteúdo ainda não definido), Clientes, Ideias, Projetos, Administrativo.
- **Agenda do Dia implementada**: card "Hoje" (tarefas por pessoa, captação do dia puxada
  automático de Captações por cliente, compromissos/reuniões manuais compartilhados com
  responsável) + "Resto da semana" (dias úteis restantes da semana atual, mesma janela do board
  semanal; se hoje é sexta ou fim de semana, cai pra próxima semana útil). Dados novos em
  `mmagenda__{iso}`, sincronizados como o resto do app.
- **Calendários implementado**: dashboard de gestão antecipada — sempre olha o **mês seguinte** ao
  atual, prazo = último dia do mês atual. "Pronto" = dia útil com mídia E link do Drive
  preenchidos (mesmos campos do calendário do cliente em Conteúdos). Metas semanais por cliente
  (`CALENDARIO_METAS`, hardcoded): lios 5, fabi 5, fabi-kids 2, localize 3, davi 3, keli 3,
  dna-movement 3, ceus-abertos 4 — **Rochele fica fora**, por pedido explícito da Priscila. Lista
  minimalista por cliente com atalho "Ver calendário →" que pula direto pra aba Conteúdos daquele
  cliente (sem passar por Clientes). Ver decisões detalhadas em `decisoes.md`.
- Tudo testado localmente com Playwright (rede do Firestore bloqueada, ver `CLAUDE.md`) antes de
  mostrar pra Priscila, em cada etapa. Commit `14dc362` + push + `scp` pro Hetzner, confirmado em
  produção (`kanban.movingg.com.br` serve os elementos novos).
- **Pendente de confirmação da Priscila:** o prazo de Calendários usa o último dia real do mês
  (ex.: 31/10, 28/02), não um "dia 30" fixo — ver decisão em `decisoes.md`. Avisado a ela, sem
  resposta ainda nesta sessão.
- **Próximo passo crítico:** (1) Priscila confirmar se o prazo deve ser sempre "fim do mês" real ou
  literalmente "dia 30" fixo; (2) ela ainda vai detalhar o conteúdo de Agenda do Dia além do que já
  foi construído (se houver mais campos), e o conteúdo de Gestão de Tráfego Pago (hoje só
  placeholder vazio).

## Atualização 2026-09-21 — Claude (com Priscila): Agenda do Dia lista tarefas + diagnóstico do sync Vento/Lyon parado (NÃO resolvido)

Origem: Claude (com Priscila). Feito para o Paul poder continuar deste repo, sem depender da máquina da Priscila.

**Feito e no ar:** Agenda do Dia agora lista as tarefas pendentes dentro dos cards (antes só as
pílulas de contagem). No card "Hoje" entram também as atrasadas (com a etiqueta "Atrasada");
o círculo conclui a tarefa de verdade (`toggleTask`); clicar na linha abre o cliente. Commit
`8a634b5`, deployado e conferido em produção (arquivo servido idêntico ao do repo).

**Problema aberto — as tarefas das abas "Tarefas Vento" e "Tarefas Lyon" (planilha DRE Vento) não
estão entrando no Kanban.** São dois problemas independentes:

1. **Nenhuma tarefa entra: o token do Google do backend do Hub expirou.** `GET
   https://hub.movingg.com.br/api/tarefas-carteira` responde 502 ("Nao foi possivel ler as tarefas
   agora"). O log do `moving-hub-api` mostra `RefreshError: invalid_grant: Token has been expired
   or revoked`, na renovação do token da conta `ventomarketingoficial@gmail.com` (escopos
   `drive.readonly` + `spreadsheets`). Não é rename de aba, como em 13/09. Causa **provável, ainda
   não confirmada**: o app OAuth (projeto Google Cloud `vento-marketing`) está em modo "Em teste",
   em que o Google derruba o refresh token a cada 7 dias (o token foi gerado por volta de 11/09).
   A Priscila confirma que o acesso não foi removido nem revogado manualmente. Provavelmente o
   painel "DRE Vento" do Início (mesmo token) também está parado — não verificado.
2. **O filtro por responsável nunca foi implementado no Kanban.** O backend já devolve o campo
   `responsavel`, mas `syncTarefasExternas()` grava tudo com `assignees: []` e importa todas as
   tarefas Lyon, de qualquer responsável. Não há registro de que essa regra tenha sido decidida
   ou implementada antes. Além disso, `VENTO_CLIENTE_MAP` só conhece "Fabi Eventos": tarefa Vento
   de outro cliente é descartada sem aviso (`skipped`).

**Regra pedida pela Priscila (a implementar):** aba Vento → importar com o responsável certo
(Paul ou Priscila); aba Lyon → importar só as tarefas com responsável Paul.

**Ordem para resolver (nada disso foi feito ainda):**
1. Google Cloud Console, projeto `vento-marketing` → Tela de consentimento OAuth: se "Em teste",
   **Publicar app**. Publicar ANTES de gerar o token novo (token gerado em teste vence em 7 dias).
2. Gerar token novo logando em `ventomarketingoficial@gmail.com` (mesmos escopos). Na máquina do
   Paul existe `config/gerar_token_ventomarketingoficial.py` (não versionado). A credencial do
   projeto é o `client_secret_...json` baixado em 11/09.
3. Copiar o token para o arquivo de token do backend do Hub no servidor (backup antes) e
   `systemctl restart moving-hub-api`. É produção: só com confirmação da Priscila. O caminho
   exato está em `clientes/moving-hub/memoria/estado-atual.md` do vault, atualização 13/09.
4. Conferir que `/api/tarefas-carteira` volta a responder 200; só então ler o retorno real e
   fechar o mapeamento de clientes.
5. Kanban (`syncTarefasExternas` e mapas de cliente): responsável → `assignees`; Lyon só Paul;
   avisar na tela quando tarefa for descartada por cliente não mapeado. Testar local com a rede
   do Firestore bloqueada, como sempre.

**Decisões da Priscila ainda em aberto:** (a) tarefa Vento com Paul e Priscila juntos vai para os
dois? (b) tarefa Vento sem responsável, ou com outro nome: entra sem dono ou é ignorada? (c) as
54 tarefas Lyon já importadas, de todos os responsáveis, ficam ou saem depois do filtro? Sugestão:
não apagar nada sozinho, listar para a Priscila decidir.

- Cópia antiga `kanban-semanal-teste-hub.html` (12/08) fica só na máquina da Priscila, sem
  versionar, de propósito.
- **Próximo passo crítico:** publicar o app OAuth e regerar o token (itens 1 a 3 acima). Sem isso
  nenhuma tarefa da planilha chega ao Kanban, independente de qualquer mudança no código.

## Atualização 2026-09-23 — Claude (com Priscila): dias do mês seguinte no Cronograma estavam travados

Origem: Claude (com Priscila), a partir de print mostrando a última linha do calendário mensal
(Gestão de Postagens → Cronograma por cliente).

- **Bug:** dias de "sobra" do mês seguinte, exibidos na última linha do calendário para completar
  a grade (ex.: dias 1 e 2 de outubro aparecendo junto com 28-30 de setembro), tinham
  `pointer-events: none` no CSS (`.sched-day.other-month`) — ficavam 100% travados, sem conseguir
  abrir "Link e legenda" nem nada mais. Não era falta de espaço visual, era clique bloqueado.
- **Correção:** removido o `pointer-events: none`; opacidade ajustada de 0.3 para 0.55 (ainda dá
  pra diferenciar visualmente do mês corrente, mas agora é editável). Como os dados desses dias são
  salvos por data ISO (`loadSched`/`loadLegend`/etc. são só por cliente, sem recorte de mês), editar
  pelo dia "outro mês" grava certinho no mesmo lugar que editar depois de navegar pro mês seguinte —
  sem risco de duplicar ou perder dado.
- Testado local (servidor Node em `localhost:8765`, `python -m http.server` não funciona nesta
  máquina — stub quebrado da Microsoft Store). Aprovado pela Priscila.
- **Próximo passo crítico:** nenhum pendente desta frente.

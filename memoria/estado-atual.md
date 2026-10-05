# ESTADO ATUAL — KANBAN

## Como usar este arquivo

Ponto de leitura no início de sessão (ver `PROTOCOLO-SESSAO.md` na raiz do repo). Compacto e
acionável: último estado validado + próximo passo crítico. Histórico completo (tudo antes de
2026-09-18) está em `memoria/historico-kanban.md` — não precisa reler para trabalhar no dia a dia.

Regras de segurança do deploy (Firebase de produção, scp, etc.) estão no `CLAUDE.md` da raiz do
repo — leitura obrigatória antes de qualquer commit que mexa em dado ou deploy.

## Atualização 2026-10-05 — Claude (com Priscila): Calendários conta o dia como feito só com o link do Drive

Origem: Claude (com Priscila)

- **Mudança de regra pedida pela Priscila:** na aba Calendários (Gestão de Redes Sociais), o dia útil
  passa a contar como pronto apenas com o link do Drive preenchido (`loadDriveLink`). Antes exigia
  link E mídia no cronograma (`loadSched`). A mudança está em `countReadyDaysInMonth()` no
  `kanban-semanal.html`; o percentual por cliente, o "Concluído" geral e "Posts prontos" derivam dela e
  recalculam ao abrir a aba.
- Comentário-regra acima de `CALENDARIO_METAS` atualizado (a regra original de 18/09 exigia os dois campos).
- Validado só por checagem de sintaxe dos scripts. **Não testado no navegador e NÃO deployado** —
  mudança só local; falta teste em localhost (Firestore bloqueado) e o OK da Priscila para o `scp`.
- Arquivo não rastreado `kanban-semanal-teste-hub.html` já existia antes da sessão; não foi tocado.
- **Próximo passo crítico:** testar em localhost e, com o sim da Priscila, commit + push + scp.

## Atualização 2026-10-01 (parte 4) — Claude (com Priscila): prévia de feed tratava cada foto do carrossel como um post separado

Origem: Claude (com Priscila), a partir de print mostrando um dia do Cronograma com 4 fotos.

- **Bug:** `openFeedPreview()` (modal "Gerar prévia") colocava cada item de `mmsched_items__{cid}[iso]`
  como uma célula própria na grade. Um dia com 4 fotos (carrossel) virava 4 posts separados na
  prévia, em vez de 1 — no Instagram real só a capa (1ª mídia) do carrossel aparece na grade do
  feed, o resto só é visto dentro do post.
- **Correção:** só o primeiro item de cada dia vira uma célula (`dayItems[0]`); dias com mais de 1
  item marcam `isCarousel: true`. Célula ganha um ícone de carrossel (SVG, canto superior direito —
  mesmo lugar/estilo do ícone de reels já existente) quando `isCarousel`, mutuamente exclusivo com o
  ícone de vídeo (um post não é as duas coisas). `kanban-semanal.html` função `openFeedPreview`.
- Testado localmente (Playwright, rede do Firestore bloqueada): 3 dias simulados (1 carrossel de 4
  fotos, 1 vídeo, 1 foto única) → prévia mostrou exatamente 3 células, 1 badge de carrossel, 1 badge
  de play. Confirmado visualmente por screenshot. Zero erro de JS novo no console.
- Próximo passo crítico: nenhum pendente desta frente.

## Atualização 2026-10-01 (parte 3) — Claude (com Priscila): limpeza de mídia antiga no Firestore (Lios perto do limite de 1MB de novo)

Origem: Claude (com Priscila)

- Priscila reportou "acabou a memória" pra anexar mídia nos calendários. Medição (leitura, via
  REST público do Firestore) confirmou: `kanban_clients/lios` estava em 616KB de 1MB, sendo 533KB
  (22 fotos/vídeos, 34 itens) só de mídia anexada **antes de 01/10/2026**. Mesmo padrão do
  incidente de 2026-09-29 (ver `erros.md`), mas desta vez pego antes de travar o sync.
- **Backup completo da coleção `kanban_clients` (43 documentos) feito ANTES de qualquer escrita**,
  salvo em `C:\Users\prisc\Documents\kanban-backups\kanban_clients-backup-2026-10-01-pre-limpeza-midia.json`
  (fora do Git — repo é público, não pode versionar dado de cliente). Script de leitura usado:
  REST `GET` paginado, sem SDK, sem autenticação (mesmo acesso que o próprio app usa).
- **Escopo confirmado com a Priscila antes de escrever:** remover só o campo `thumbnail` (base64)
  dos itens de `mmsched_items__{cid}` com data anterior a 01/10/2026, mantendo `name`/`mimeType`/
  `id` (metadado leve) e sem tocar em `mmlegend__`/`mmdrivelink__`/`mmformat__` (legenda, link do
  Drive e status do dia continuam intactos). Em todos os clientes, não só Lios.
- **Execução:** script Node com `https` puro (sem dependência nova), um `PATCH` REST por cliente,
  `updateMask.fieldPaths=data.mmsched_items__{cid}` — escreve **só esse campo**, igual ao padrão de
  merge campo-a-campo que o próprio app usa (`_pushToCloud`), pra não colidir com sync de outro
  navegador aberto. Testado escrever/apagar um campo descartável (`_global.ZZZ_TESTE_WRITE_ACCESS`)
  antes de tocar em dado real, seguindo o padrão de teste já registrado em `erros.md`.
  Resultado, verificado por releitura depois (mídia de out/2026+ confirmada intacta nos 3):
  - `lios`: 22 mídias removidas, 2 mantidas (out/2026+) — documento 616KB → 85,5KB.
  - `davi`: 5 mídias removidas, 13 mantidas — documento 538,5KB → 415,9KB.
  - `localize`: 4 mídias removidas, 12 mantidas — documento 376,6KB → 284,1KB.
  - `_global` (chave residual `mmsched_items__zion`, cliente descontinuado): 1 mídia removida —
    25,5KB → 14,5KB.
  - Total liberado: ~757KB. Demais clientes já não tinham mídia anterior a outubro (nada a apagar).
- Ação técnica real com efeito em produção (escrita direta no Firestore, fora do fluxo normal do
  app) — bloqueada automaticamente pelo classificador de permissão do Claude Code na primeira
  tentativa ("Modify Shared Resources"); Priscila aprovou explicitamente antes de eu repetir o
  comando. Registrar aqui porque é precedente: operação futura equivalente também vai pedir
  aprovação explícita, mesmo com backup e escopo já confirmados.
- Se o navegador da Priscila ou do Paul estiver com o Kanban aberto numa aba antiga durante essa
  janela, `_subscribeToChanges` deve detectar a mudança remota e recarregar sozinho (mesmo
  mecanismo que já existe pra qualquer sync entre navegadores) — não precisa ação manual, só
  checar se algo parecer desatualizado.
- Próximo passo crítico: nenhum pendente desta frente. Se o padrão de acúmulo voltar (Lios já foi
  a maior consumidora duas vezes), vale considerar um lembrete recorrente de "arquivar/limpar mídia
  de mês fechado" em vez de esperar o limite avisar sozinho.

## Atualização 2026-10-01 (parte 2) — Claude (com Priscila): Calendários agora mostra mês atual + próximo mês

Origem: Claude (com Priscila)

- Pedido da Priscila: a aba `Calendários` só mostrava o mês seguinte (gestão antecipada); ela
  também quer ver o mês atual pra acompanhar se está correndo tudo certo durante o mês, sem deixar
  de já adiantar o próximo.
- `computeCalendarioDashboard()` generalizada para aceitar `(targetY, targetM)` em vez de só
  calcular o mês seguinte internamente. `getCalendarioCycle()` agora devolve `curY/curM` (mês
  atual) e `nextY/nextM` (mês seguinte), além do `deadline`/`daysLeft` (continuam sendo os mesmos
  pros dois blocos — o prazo de ambos é o fim do mês atual).
  `refreshCalendarioPage()` reescrita pra renderizar dois blocos completos (KPIs + lista por
  cliente), com ids separados (`calcliList-cur` / `calcliList-next`) pra não colidir.
- **Botão "Ver calendário →" agora abre o cronograma do cliente já no mês certo** — antes sempre
  abria no mês que estivesse em `schedMonths[cid]` (geralmente o atual, por padrão). Agora
  `goToClientCalendar(cid, targetY, targetM)` seta `schedMonths[cid]` e chama `refreshSched(cid)`
  antes de trocar de aba, então o botão do bloco de outubro abre outubro e o de novembro abre
  novembro. Testado e confirmado via Playwright (ver abaixo).
- Testado localmente (Playwright, rede do Firestore bloqueada, servidor Node em `localhost:8766`
  — `python`/`python3` continuam stubs quebrados nesta máquina): os dois cabeçalhos renderizam
  corretamente ("Calendário de Outubro 2026 · mês atual" / "Calendário de Novembro 2026 · próximo
  mês"), as duas listas têm os 7 clientes de `CALENDARIO_METAS` (sem Fabi Eventos Kids, já
  removida), e o clique em "Ver calendário →" de cada bloco abriu o mês correto no cronograma do
  cliente testado (Lios). Zero erro de JS no console (só o ruído esperado de Firestore bloqueado).
  Print aprovado pela Priscila antes do deploy.
- Próximo passo crítico: nenhum pendente desta frente após o deploy.

## Atualização 2026-10-01 — Claude (com Priscila): Fabi Eventos Kids removida do Gestão de Postagens e de Calendários

Origem: Claude (com Priscila)

- Fabi Eventos Kids (`fabi-kids`) ganhou `hideFromBoard: true` no array `CLIENTS` — mesmo padrão
  já usado para Essência Gastronomia e Rochele. Some do quadro semanal (`Gestão de Postagens`),
  continua normal em Clientes/Cronograma.
- Removida a entrada `'fabi-kids': 2` de `CALENDARIO_METAS` — sai do dashboard de `Calendários`
  (gestão antecipada do mês seguinte) e da meta mensal consolidada.
- Verificado programaticamente (`inBoard(fabi-kids) === false`, `CALENDARIO_METAS` sem a chave,
  46 clientes intactos) antes do commit. Não testado com Playwright/localhost visual — mudança é
  só flag estático, sem interação de dado nem rede do Firestore envolvida.
- Próximo passo crítico: aguardando confirmação da Priscila para `scp` em produção (regra do
  `CLAUDE.md` — nunca deploy sem autorização explícita).

## Atualização 2026-09-30 — Claude (com Priscila): Rochele removida do Gestão de Postagens

Origem: Claude (com Priscila)

- Rochele ganhou `hideFromBoard: true` no array `CLIENTS` (mesmo padrão da Essência Gastronomia).
  Ela some do quadro semanal (`Gestão de Postagens`), mas continua normal em Clientes/Calendário —
  já estava fora do cálculo de meta mensal (`CALENDARIO_METAS`) desde antes, por pedido da Priscila.
- Verificado programaticamente (`inBoard(rochele) === false`) antes do deploy.
- Próximo passo crítico: nenhum pendente desta frente.

## Complemento 2026-09-30 — Claude (com Priscila): banner de falha de sync no Davi Mello — recorrência do padrão conhecido

Origem: Claude (com Priscila)

- Priscila viu o banner vermelho "Não foi possível salvar na nuvem" enquanto preenchia o
  calendário de outubro do Davi Mello. Prioridade imediata: backup local via
  `document.getElementById('btnBackup').click()` no console (ela não achava o botão na tela),
  confirmado o `.json` baixado antes de qualquer outra ação.
- Diagnóstico (leitura direta da API REST do Firestore, só leitura, sem alterar nada): os erros de
  CORS/`firebasestorage.googleapis.com` no console são ruído esperado, não a causa — confirmado de
  novo que o bucket do Storage **não existe** (`storage.googleapis.com` responde 404 "specified
  bucket does not exist"), mesma causa raiz documentada em `decisoes.md` (29/09, custo do plano
  Blaze recusado). `mmsched_items__davi` já estava em ~455KB de 1MB só de miniaturas base64 —
  mesma trajetória que causou o incidente da Localize em 29/09, ainda sem bater o teto.
- A falha em si foi pontual: clicar em **Sincronizar agora** (`btnForceSync`) resolveu — confirmado
  escrita nova no Firestore ~37s depois do clique. Nenhuma alteração de código feita; nenhum dado
  perdido (o reload só foi autorizado depois da escrita confirmada na nuvem).
- Enquanto investigava, Priscila reportou contagem de "prontos" divergente no dashboard de
  Calendários (10/13 na tela vs. ela acreditando ter preenchido 12-13). Não é bug: `countReadyDaysInMonth`
  exige mídia **e** link do Drive preenchidos juntos (regra combinada 18/09). Conferido campo a campo
  em outubro do Davi: dia 26 foi completado durante a sessão (confirmado via leitura nova); **dia 21
  segue só com foto, sem o link do Drive** — é o único pendente para a contagem bater.
- Próximo passo crítico: nenhuma ação de código pendente. Fica como pendência operacional da
  Priscila preencher o link do dia 21 no calendário do Davi Mello. Se o padrão de bloat em
  `mmsched_items__*` continuar (Davi, Lios e outros clientes de foto pesada), vale reabrir a
  conversa sobre Storage ou sobre arquivar miniaturas antigas antes que outro cliente bata no
  limite de 1MB — não é urgente agora.

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
  máquina — stub quebrado da Microsoft Store).
- **Complemento (mesma sessão):** depois do primeiro deploy, a Priscila reportou que o campo de
  link/legenda desses dias "abre, mas os campos ficam cortados/sem espaço". Causa: `.sched-grid`
  tem rolagem própria (`max-height: 900px; overflow-y: auto`), separada da rolagem da página. Ao
  expandir um dia no fim da lista, o campo novo nasce dentro dessa área rolável sem nada levar o
  scroll até ele — parecia cortado, mas só estava fora da área visível. Corrigido: o clique no
  toggle agora chama `scrollIntoView({block:'nearest', behavior:'smooth'})` no `.sched-more-body`
  recém-aberto. Testado local e aprovado pela Priscila.
- **Próximo passo crítico:** nenhum pendente desta frente.

## Atualização 2026-09-29 — Claude (com Priscila): cor de legenda + prévia de feed + correção estrutural do sync (documento por cliente) após perda real de dados da Localize

Origem: Claude (com Priscila)

- **Fix simples:** texto da legenda de post, depois de preenchido, usava cor clara residual de
  tema escuro antigo (`rgba(232,227,220,0.7)`), quase invisível no tema claro atual. Trocado por
  `var(--text)`, igual aos outros campos. Commit `99a57f2`.
- **Feature nova:** botão "Gerar prévia" no fim de cada calendário mensal por cliente — abre modal
  simulando o grid do perfil do Instagram (avatar, @handle, mês, grid 3 colunas com os posts do mês
  em ordem cronológica invertida — mais recente primeiro, como o feed real). Hover mostra
  data+legenda; vídeo/reels ganha selo de play. Commit `897bf01`.
- 🔴 **Incidente real, causa raiz corrigida:** Priscila reportou "preenchi o calendário inteiro da
  Localize, atualizei a página, sumiu tudo". Investigação revelou problema estrutural grave — ver
  `erros.md` e `decisoes.md` para o detalhe completo. Resumo: documento único do Firestore
  compartilhado por 24 clientes bateu no limite de 1MB (Lios sozinha = 585KB de imagens acumuladas
  desde junho), sync passou a falhar em silêncio, reload seguinte sobrescrevia o navegador com o
  último estado sincronizado. Corrigido com: (1) alerta visível de falha de sync (commit
  `11aef98`); (2) documento por cliente no Firestore em vez do único compartilhado, com migração
  dos 1.603 campos verificada célula a célula, zero divergência (commit `7a5d4ca`); (3) regras do
  Firestore atualizadas no Console pela própria Priscila.
- **Dados da Localize não recuperados:** legenda, link do Drive e foto dos dias 07 a 30/10 nunca
  chegaram a sincronizar — só existiam no navegador da Priscila, sobrescritos antes desta sessão.
  Notas curtas (título) sobreviveram para todos os dias. Priscila vai reenviar manualmente;
  confirmado a ela que a causa raiz está corrigida e o alerta visível previne repetição silenciosa.
- Testado extensivamente em localhost (Playwright, rede do Firestore bloqueada) antes de cada
  deploy, seguindo a regra do `CLAUDE.md`. Migração de dados real rodada com backup prévio e
  verificação byte a byte (zero divergência em 43 documentos novos).
- **Próximo passo crítico:** nenhum bloqueante. Se o volume de imagens crescer muito de novo (Lios
  já usa 630KB do 1MB dela sozinha), reabrir a conversa sobre Firebase Storage (hoje bloqueado por
  custo, ver `decisoes.md`) ou arquivar meses antigos. `kanban-semanal-teste-hub.html` (untracked,
  máquina da Priscila) segue fora do Git de propósito, não tocado nesta sessão.

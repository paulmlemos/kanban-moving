# Kanban / Sistema da Agência Vento — guia para quem for mexer no front-end

Este documento existe para uma IA (Claude ou outra) que vai **reformular o front-end** deste
sistema sem o Paul ou a Priscila precisarem reexplicar a arquitetura. Foi escrito por outra
sessão de Claude (trabalhando com a Priscila) depois de revisar o repositório inteiro.

Leia isto inteiro antes de editar qualquer coisa. O objetivo desta reforma é **mudar a aparência
e a estrutura visual**, sem quebrar a sincronização de dados nem os dados reais dos 24 clientes
da agência (dados de produção, não é ambiente de teste).

---

## 1. O que é este projeto

- Arquivo único: `kanban-semanal.html` (~9.360 linhas: CSS + HTML + JS, tudo no mesmo arquivo,
  sem build step, sem bundler, sem framework).
- Sem servidor de aplicação próprio. "Back-end" = Firebase/Firestore (serviço externo) acessado
  direto do JS do navegador. Não existe pasta `functions/`, não existe `firestore.rules` neste
  repositório — as regras de segurança do Firestore são geridas manualmente pelo Firebase Console
  (projeto `moving-kanban`), fora do controle de versão.
- Deploy: **não é Vercel** (o `vercel.json` na raiz é resquício de uma fase anterior, suspensa
  desde 2026-06-11, e não é usado — pode ignorar). Produção real é `kanban.movingg.com.br`,
  servido por um arquivo estático em `root@46.225.109.50:/var/www/kanban/index.html` (servidor
  Hetzner). Deploy = commit + push neste repo (`paulmlemos/kanban-moving`) + `scp` manual do
  `kanban-semanal.html` para esse caminho.

## 2. Regras não-negociáveis antes de qualquer deploy

1. **Nunca fazer `scp` para o servidor de produção sem confirmação explícita da Priscila.**
   Teste local primeiro, mostre o resultado, só suba depois do "sim" dela.
2. **Testar localhost NÃO isola este app do Firebase de produção.** O arquivo conecta nas
   credenciais reais do Firebase mesmo servido via `file://` ou `localhost` — não existe
   `.env`/staging separado. Um incidente real (2026-07-27) vazou uma tarefa de teste para o
   Firestore de produção por causa disso. **Antes de qualquer teste automatizado (Playwright ou
   similar), bloqueie as chamadas de rede para `firestore.googleapis.com` e `firebaseio.com`**
   (ex.: `context.route(/firestore\.googleapis\.com|firebaseio\.com/, route => route.abort())`).
   Sem isso, qualquer clique que crie/edite dado pode gravar em produção de verdade.
3. `python`/`python3` nesta máquina (Windows da Priscila) são stubs quebrados da Microsoft Store.
   Para servir o arquivo localmente use Node (`http.createServer` ou `npx serve`), nunca
   `python -m http.server`.
4. Sempre criar commits normais (nunca `--amend` em commit já existente, nunca reescrever
   histórico, nunca force-push).

## 3. As 3 camadas do arquivo (mesmo estando tudo em 1 arquivo só)

Não há separação física em arquivos, mas há uma separação lógica clara por convenção de nomes.
**A missão de "mudar só o front-end" significa: mexer livremente na camada 3, nunca na camada 1,
preservar os nomes/assinaturas/chaves da camada 2.**

### Camada 1 — Motor de sync com Firebase (NÃO TOCAR)

Bloco `// ─── FIREBASE SYNC ───` logo no início do `<script>` (linha ~3568–3750 nesta revisão —
localize pelo comentário, não pelo número, porque vai mudar conforme o arquivo cresce).

Contém: `_fbConfig` (config pública do Firebase — não é segredo, é a chave client-side normal do
Firebase, pode ficar no código), `firebase.initializeApp`, `_fsdb`, `mmGet`/`mmSet`/`mmRemove`
(wrappers de `localStorage` que dispara sync automático), `_pushToCloud`/`_loadFromCloud`/
`_subscribeToChanges` (merge campo a campo no Firestore, nunca sobrescreve o documento inteiro),
botão de backup/importação manual (`btnBackup`/`btnImport`, JSON local, isso é diferente do
Firestore — inclui até dados que nunca vão pra nuvem, tipo senhas de acesso).

Regra: **essas funções e a lógica interna delas não mudam.** É o que garante que o navegador da
Priscila e do Paul continuem sincronizados. Front-end novo pode (e deve) continuar chamando
`mmGet`/`mmSet`/`mmRemove` por baixo dos panos — só não pode reescrever como elas funcionam.

### Camada 2 — Camada de dados (schema fixo, não renomear chaves nem funções)

Cerca de 30 pares `loadX(clientId)` / `saveX(clientId, valor)`, todos implementados em cima de
`lsGet`/`lsSave` (linhas ~5532–5533), que por sua vez chamam `mmGet`/`mmSet`. Cada par sabe uma
chave de `localStorage` e o formato (shape) do dado daquela funcionalidade. Lista completa das
chaves usadas hoje (prefixo `mm`, quase todas com `__${clientId}` ou `__${id}` no final):

| Chave (prefixo) | Função load/save | Conteúdo |
|---|---|---|
| `mmk_${wk}__${cid}__${di}__${platId}__${type}` | `saveState`/leitura direta | status do kanban semanal (0/1/2) |
| `mmtasks__${cid}` | `loadTasks`/`saveTasks` | tarefas do cliente |
| `mmtaskf__${taskId}` | `loadTaskFiles`/`saveTaskFiles` | anexos de tarefa (só localStorage, não sincroniza) |
| `mmcap__${cid}` | `loadCap`/`saveCap` | próxima data de captação |
| `mmcaphist__${cid}` | `loadCapHist`/`saveCapHist` | histórico de captações |
| `mmidea__${cid}` | `loadIdeas`/`saveIdeas` | ideias/referências |
| `mmideaf__${id}` | `loadIdeaFiles`/`saveIdeaFiles` | anexos de ideia (só localStorage) |
| `mmedit__${cid}` | `loadEditItems`/`saveEditItems` | edições do cronograma |
| `mmeditf__${id}` | `loadEditFiles`/`saveEditFiles` | anexos de edição (só localStorage) |
| `mmmat__${cid}` | `loadMats`/`saveMats` | materiais (base64, só localStorage) |
| `mmsched_items__${cid}` | `loadSched`/`saveSched` | itens do cronograma |
| `mmnopost__${cid}` | `loadNoPost`/`saveNoPost` | dias marcados "sem post" |
| `mmlegend__${cid}` | `loadLegend`/`saveLegend` | legenda do cronograma |
| `mmformat__${cid}` | `loadFormat`/`saveFormat` | formato do post |
| `mmbrief__${cid}` | `loadBrief`/`saveBrief` | **nota de texto por dia no Cronograma** (não confundir com a próxima linha!) |
| `mmss__${cid}__${iso}` | `getSchedSt`/`setSchedStLocal` | status agendado por data ISO |
| `mmficha__${cid}` | `loadFicha`/`saveFicha` | Ficha do cliente (handle, tom de voz, cores, drive, canva…) |
| `mmbriefing__${cid}` | `loadBriefing`/`saveBriefing` | **Briefing por cliente (aba nova, só existe pro cliente `dca` hoje)** — respostas de formulário externo (site, missão, concorrentes, tom de comunicação etc.) |
| `mmfin__${cid}` | `loadFin`/`saveFin` | financeiro (valor, vencimento, histórico) |
| `mmresult__${cid}` | `loadResult`/`saveResult` | resultados mensais |
| `mmnotas__${cid}` | `loadNotas`/`saveNotas` | anotações do cliente |
| `mmacesso__${cid}` | `loadAcessos`/`saveAcessos` | acessos/senhas (só localStorage, nunca sincroniza) |
| `mmonboard__${cid}` | `loadOnboard`/`saveOnboard` | checklist de onboarding |
| `mmonblinks__${cid}` | `loadOnbLinks`/`saveOnbLinks` | links rápidos do onboarding |
| `mmpipeline` | `loadPipeline`/`savePipeline` | pipeline comercial (leads) |
| `mmpautas` | `loadPautas`/`savePautas` | banco de pautas (**seção oculta**, ver seção 6) |
| `mmprojects` | `loadProjects`/`saveProjects` | projetos (Céus Abertos, DNA Movement etc.) |
| `mmui`, `mmuiActiveTab`, `mmuiActiveClient` | direto | estado de UI local (aba/cliente ativo) — nunca sincroniza |
| `mmDateBadge` | direto | badge de datas comemorativas |

**Cuidado com nome parecido**: `mmbrief__` (nota de texto por dia dentro do Cronograma de
conteúdo) e `mmbriefing__` (aba de Briefing por cliente, feature nova) são coisas **completamente
diferentes** apesar do nome quase igual. Não misturar, não renomear um achando que é o outro.

Chaves que **nunca sincronizam** com o Firestore (ficam só no navegador local, ver `_SKIP_SYNC`
na Camada 1): prefixos `mmmat__`, `mmtaskf__`, `mmacesso__`, `mmideaf__`, `mmeditf__`, `mmui`.
Isso é proposital (dados grandes em base64 e senhas não devem ir pra nuvem) — preservar esse
comportamento.

### Camada 3 — Front-end (aqui pode reformular à vontade)

- Todo o CSS: linhas ~10–3041 (`<style>...</style>`, dentro do `<head>`).
- Todo o HTML estático: linhas ~3042–3567 (dentro de `<body>`, antes do `<script>`).
- ~25 funções `buildX()`/`renderX()` no JS (ex.: `buildTasksGrid`, `buildFichaGrid`,
  `renderBriefing`, `buildClientSections`, `renderSatisfaction`, `buildCliCards`,
  `renderCeusFases`, etc.) — cada uma gera HTML **e** chama as funções `loadX`/`saveX` da Camada 2
  para ler/gravar dado e reagir a input do usuário (auto-save).
- Navegação principal: `switchMainTab(tab)` + botões `.sb-tab[data-main-tab]` (Início, Gestão de
  Tarefas, Gestão de Postagens, Clientes, Ideias, Projetos, Administrativo) alternam a classe
  `.active` em seções `#main-{tab}`. Dentro de "Clientes", `selectClient(cid)` mostra
  `#sec-cli-{cid}` e monta as abas internas do cliente (ver `buildClientSections`).
- CLIENTS array (linha ~3753): lista de todos os clientes com `id`, `name`, `color`, `platforms`
  (define se aparece na grade semanal — vazio = não aparece) e flags `noKanban`, `noFinanceiro`,
  `isInternal`, `isPersonal`. Pode reorganizar/redesenhar como isso é exibido, mas os `id` de cada
  cliente são a chave de tudo na Camada 2 — **não renomear `id` de cliente existente**, isso
  quebraria a ligação com todos os dados já salvos daquele cliente.
- Abas específicas por cliente (fora do padrão): `encontros-fe` tem aba extra "Eventos"
  (`renderIgrejaEventos`); `dca` tem aba extra "Briefing" (`renderBriefing`, ver Camada 2). Essas
  condicionais ficam dentro de `buildClientSections` (busque por `c.id === 'encontros-fe'` e
  `c.id === 'dca'`).

## 4. Pegadinha real: containers ocultos + `moveEl`

Várias seções (Ficha, Captações, Tarefas, Notas, Resultados, Cronograma, Acessos) são construídas
primeiro dentro de **grids globais ocultos** no HTML (`#fichaGrid`, `#capGrid`, `#tasksGrid`,
`#notasGrid`, `#resultGrid`, `#cronoAccordions`, `#acessosGrid` — bloco `<!-- STAGING OCULTO:
containers dos builders antigos -->`, linha ~3395). As funções `buildFichaGrid()`,
`buildCapGrid()` etc. criam um card por cliente dentro desses grids ocultos. **Só depois**,
`buildClientSections()` usa uma função helper `moveEl(srcId, panelId)` para mover
(`appendChild`) o card específico de cada cliente do grid oculto para dentro da aba daquele
cliente.

Se você não souber disso, esses `div`s ocultos parecem lixo morto e dá vontade de apagar — **não
apague**. Se remover, os cards somem (ficam presos num container que não existe mais) e a
funcionalidade quebra silenciosamente, sem erro no console. Se for redesenhar essas seções, pode
mudar totalmente o HTML gerado *dentro* de cada `buildXGrid()`, só precisa manter o padrão de
"constrói no grid global → `buildClientSections` mexe/move para a aba certa" ou refatorar as duas
pontas juntas (build + move) de forma consistente.

## 5. Contrato mínimo para considerar a reforma "só front-end"

Antes de dar como concluída qualquer mudança, confirme:

- [ ] Nenhuma função `loadX`/`saveX` teve nome, assinatura ou chave de `localStorage` alterada.
- [ ] O bloco `FIREBASE SYNC` (Camada 1) não foi tocado.
- [ ] Nenhum `id` de cliente existente no array `CLIENTS` foi renomeado (adicionar cliente novo
      é seguro; renomear `id` de um já existente perde a ligação com os dados salvos dele).
- [ ] Os containers ocultos de staging (`#fichaGrid` etc.) e a lógica de `moveEl`/`buildClientSections`
      continuam existindo, ou foram refatorados nas duas pontas ao mesmo tempo (build + move),
      nunca só de um lado.
- [ ] Testado localmente com rede do Firestore bloqueada (ver regra 2 da seção 2) antes de
      qualquer commit que envolva interação de dado.
- [ ] Nenhum deploy (`scp` pro Hetzner) feito sem confirmação explícita da Priscila.

## 6. Coisas que existem de propósito e não são bug

- `#sec-pautas` ("Banco de Pautas") está com `display:none` fixo — foi tirado da navegação de
  propósito, mas o código (`renderPautas`, `loadPautas`/`savePautas`) continua funcional e vivo,
  mantido "por compatibilidade". Não é código morto para apagar sem perguntar; é uma decisão de
  produto já tomada (remover da UI, manter o dado/lógica).
- `vercel.json` na raiz do repo é resquício da hospedagem antiga na Vercel (suspensa desde
  2026-06-11, HTTP 402 do plano gratuito). Não é usado no deploy atual (Hetzner via `scp`). Pode
  ficar ou ser removido, não afeta produção — mas não é o mecanismo de deploy real, não seguir
  instruções de deploy Vercel encontradas por aí.
- A chave `apiKey` do Firebase em `_fbConfig` está exposta em texto puro no HTML de propósito —
  é a config pública client-side do Firebase (normal e documentado pelo próprio Firebase), a
  segurança real está nas regras do Firestore (geridas no Console, fora deste repo). Não é uma
  credencial vazada para "corrigir".

## 7. Se algo aqui parecer desatualizado

Este documento foi escrito lendo o código em `ebf54fa` (2026-08-03). Números de linha vão
desatualizar rápido. Sempre confirme pela busca de texto (nome de função, comentário de seção
`// ── NOME ──` ou `// ══...══`) em vez de confiar cegamente nos números. Se encontrar uma
contradição real entre este documento e o código, o código manda — mas registre a divergência
(ex.: atualizando este arquivo) para a próxima IA não tropeçar de novo.

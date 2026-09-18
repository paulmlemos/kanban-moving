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

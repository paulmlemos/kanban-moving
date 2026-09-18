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

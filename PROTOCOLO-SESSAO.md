# PROTOCOLO DE SESSÃO — KANBAN

Versão enxuta do protocolo do vault `moving-marketing`, adaptada para este repo: 2 operadores
(Priscila e Paul), 1 ferramenta (arquivo único `kanban-semanal.html`). Sem chamados-ia, sem índice
de escopos, sem biblioteca cognitiva — não se aplica aqui.

## Abertura de sessão

**Gatilho:** "início de sessão" (ou início natural de qualquer bloco de trabalho neste repo).

1. `git status` + `git log --oneline -5` — conferir se está sincronizado com `origin/main` e se
   há trabalho de outro operador (Paul ou Priscila) não commitado.
2. Ler `memoria/estado-atual.md` inteiro (curto, deve caber numa leitura só).
3. Ler `CLAUDE.md` (raiz) se a tarefa envolver mexer no código ou fazer deploy — tem as regras não
   -negociáveis de segurança (Firebase de produção, camadas do arquivo, etc.).
4. Se a tarefa for corrigir um bug ou revisitar algo antigo, checar `memoria/erros.md` primeiro.
5. Só então responder ou executar.

## Fechamento de sessão

**Gatilho:** "fechar sessão", "salva", "pode fechar", ou encerramento natural de um bloco de
trabalho concluído.

1. Revisar internamente: o que mudou no código? Alguma decisão real foi tomada? Algum erro/bug
   novo apareceu ou foi corrigido? O próximo passo crítico mudou?
2. Atualizar `memoria/estado-atual.md` — bloco novo `## Atualização AAAA-MM-DD — [quem]`, com o
   que foi feito, estado atual e próximo passo crítico. Não apagar o bloco anterior.
3. Se houve decisão estrutural real (não só código): adicionar em `memoria/decisoes.md`.
4. Se houve bug/incidente real: adicionar em `memoria/erros.md`.
5. Commit específico (nunca `git add -A`/`git add .`) — listar os arquivos explicitamente:
   ```bash
   git add kanban-semanal.html memoria/estado-atual.md
   git commit -m "descricao curta do que mudou"
   git push
   ```
6. **Antes de qualquer commit que envolva `scp` para produção:** confirmação explícita da
   Priscila é obrigatória, mesmo que a mudança pareça pequena (ver `CLAUDE.md`, seção 2). Commit
   + push neste repo não é deploy — deploy é o `scp` manual pro Hetzner, passo separado.
7. Confirmar `git status` limpo e push confirmado antes de responder "fechado".

## Regras críticas (herdadas do vault, aplicam aqui também)

- Nunca inventar decisão ou erro que não aconteceu.
- Nunca sobrescrever o registro de sessão do outro operador — se os dois mexeram no mesmo dia,
  acrescentar bloco `## Complemento AAAA-MM-DD — [quem]`, não substituir.
- Nunca fazer `scp` de deploy sem confirmação explícita da Priscila (regra vale também para o Paul).
- Nunca force-push, nunca `--amend` em commit já publicado.

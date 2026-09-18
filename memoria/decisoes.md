# DECISÕES — KANBAN

Decisões estruturais reais deste repo. Formato:

```
## Decisao: [nome]

Motivo:
[descricao]

Impacto:
[descricao]

Regra:
[descricao]
```

Decisões anteriores a 2026-09-18 estão registradas dentro de `memoria/historico-kanban.md`
(misturadas ao log cronológico — este arquivo separado só passa a existir a partir de hoje).

---

## Decisao: memoria e protocolo de sessao proprios, dentro deste repo

Motivo:
O Paul passou a trabalhar diretamente no `kanban-public`, não só a Priscila. A memória do Kanban
morava no vault `moving-marketing`, que não é garantido estar clonado do lado deste repo em toda
máquina.

Impacto:
Este repo passa a ser autossuficiente: qualquer pessoa que clone só o `kanban-public` consegue
abrir e fechar sessão de trabalho sem depender de outro repositório.

Regra:
Toda sessão de trabalho neste repo segue `PROTOCOLO-SESSAO.md` (raiz). Decisão nova e real vai
neste arquivo; erro/incidente vai em `erros.md`; estado vivo vai em `estado-atual.md`. O vault
`moving-marketing` deixa de ser a fonte de memória viva do Kanban — só referencia este repo.

---

## Decisao: Calendários — prazo de gestão antecipada usa o último dia real do mês, não "dia 30" fixo

Motivo:
Priscila descreveu a regra usando setembro como exemplo ("até o dia 30 do mês atual, ter o
calendário do mês seguinte pronto"). Setembro tem 30 dias, mas nem todo mês tem — implementar
literalmente "dia 30" quebraria em meses de 31 dias (folga indevida) e em fevereiro (dia 30 nunca
existe).

Impacto:
O prazo mostrado na dashboard de Calendários (`getCalendarioCycle()`) é sempre o último dia do
mês atual (`new Date(ano, mesAtual+1, 0)`), não um "dia 30" hardcoded. Em setembro os dois batem
(30/09), então o comportamento é idêntico ao exemplo dela.

Regra:
**Pendente de confirmação explícita da Priscila** — avisado a ela no fechamento desta feature
(sessão de 2026-09-18), ainda sem resposta. Se ela quiser literalmente o dia 30 fixo (ex.: prazo
seria 30/10 num mês de 31 dias, não 31/10), ajustar `getCalendarioCycle()` para usar
`Math.min(30, ultimoDiaDoMes)` em vez do último dia real.

---

## Decisao: Calendários — meta mensal por cliente = dias úteis do mês × (posts/semana ÷ 5)

Motivo:
Os clientes têm cotas semanais diferentes (5, 4, 3 ou 2 posts/semana) e o calendário do cliente só
tem colunas de segunda a sexta (grid de 5 dias úteis, sem fim de semana). Precisava de uma forma de
converter "X posts/semana" em "quantos posts esse mês" sem depender de contar semanas de forma
ambígua (mês não fecha em semanas inteiras).

Impacto:
`weekdaysInMonth(y,m)` conta os dias úteis reais do mês alvo; a meta de cada cliente é
`arredondar(diasUteis × cota/5)`. Um cliente de 5/semana (posta todo dia útil) tem meta = próprio
número de dias úteis do mês; um de 3/semana tem 60% disso.

Regra:
Cotas semanais fixas em `CALENDARIO_METAS` (código): lios 5, fabi 5, fabi-kids 2, localize 3,
davi 3, keli 3, dna-movement 3, ceus-abertos 4. **Rochele fica fora** (Priscila pediu
explicitamente, mesmo estando no Gestão de Postagens). Mudança de cota ou de cliente considerado
exige editar essa constante — não tem UI para isso ainda.

---

## Decisao: Agenda do Dia — compromissos/reuniões são agenda compartilhada, com responsável

Motivo:
Não existia sistema pra compromissos/reuniões externas (diferente de tarefas, que já tinham
sistema). Priscila escolheu explicitamente compartilhada (ela e Paul veem a mesma lista, cada item
marcado com quem é responsável) em vez de agenda individual separada.

Impacto:
Novo par `loadAgendaItems`/`saveAgendaItems` (chave `mmagenda__{iso}`, uma por dia), sincronizado
via Firebase como qualquer outro dado (não está em `_SKIP_SYNC`). Item tem `assignees` (array,
igual tarefas) em vez de dono único.

Regra:
Captação no resumo da Agenda do Dia **não é cadastro novo** — puxa automático do campo `nextDate`
que já existe em Captações por cliente (`loadCap`). Contagem de tarefas no resumo é sempre separada
por pessoa (Priscila/Paul), nunca total único.

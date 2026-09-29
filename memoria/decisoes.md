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

---

## Decisao: documento por cliente no Firestore, em vez de mover imagens pro Firebase Storage

Motivo:
A causa raiz da perda de dados da Localize (ver `erros.md`, 2026-09-29) foi o documento único do
Firestore batendo no limite de 1MB. A correção "mais correta" de arquitetura seria mover
miniaturas pro Firebase Storage (URLs em vez de base64 embutido). Mas o projeto `moving-kanban`
nunca teve Storage habilitado, e habilitar exige o plano pago Blaze — a tela de upgrade pediu à
Priscila um bloqueio de R$150 pré-pago, reembolsável só cancelando a conta inteira. Rejeitado por
ela por não fazer sentido pro uso real (imagens leves, poucos MB no total).

Impacto:
Escolhida a alternativa gratuita: dividir o documento único em um documento por cliente
(`kanban_clients/{clientId}`), dentro do plano gratuito Spark do Firestore — sem cartão, sem
serviço externo novo. Cada cliente ganha 1MB só pra si (antes, 24 clientes dividiam 1MB). Resolve
o incidente de forma permanente para o volume de uso atual.

Regra:
O código de upload pro Storage (`_uploadThumbToStorage`, em `kanban-semanal.html`) ficou no
arquivo, pronto e testado, mas **inativo** — cai sempre no fallback pro base64 local, já que o
bucket não existe (404 confirmado). Se no futuro o Storage for habilitado (decisão de custo, não
técnica), esse caminho já está pronto pra ativar sem reescrever nada. Documento único antigo
(`kanban/state`) não foi apagado — fica como histórico/rede de segurança, sem uso pelo código.

---

## Decisao: classificação de dono de chave por padrão de nome (`_ownerOf`), não por lista manual

Motivo:
As ~1.600 chaves do Firestore seguem majoritariamente o padrão `prefixo__{clientId}` ou
`prefixo__{clientId}__extra`, mas não 100% (ex.: `mmceus__acao12`, `mmdna_eventos`,
`mmagenda__{data}` são globais por desenho, não por cliente). Uma lista manual de exceções seria
frágil e desatualizaria a cada cliente novo.

Impacto:
`_ownerOf(key)` testa se a chave termina em `__{id}` ou contém `__{id}__`, pra qualquer id em
`[...CLIENTS, MOVING_CLIENT]`. Chave que não bate com nenhum cliente cai automaticamente em
`_global` — nunca é perdida, nunca é misclassificada pra outro cliente por engano. Validado contra
as 1.603 chaves reais de produção antes de migrar: 43 donos encontrados, 41 chaves corretamente
caídas em `_global` (conferidas uma a uma, incluindo resíduos de cliente descontinuado como
`zion`), zero colisão indevida.

Regra:
Cliente novo no array `CLIENTS` já ganha classificação automática — não precisa editar
`_ownerOf`. Feature nova que use uma chave fora do padrão `__{clientId}` cai no documento
`_global` (compartilhado, mesmo risco de tamanho do documento único antigo, só que hoje bem menor —
59 chaves, ~26KB). Preferir manter o padrão `__{clientId}` em features novas que sejam por cliente.

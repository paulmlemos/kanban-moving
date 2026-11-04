# ERROS — KANBAN

Erros e incidentes reais deste repo, com causa e prevenção. Formato:

```
## Erro: [nome]

Impacto:
[descricao]

Causa:
[descricao]

Correcao:
[descricao]

Prevencao:
[descricao]
```

Histórico completo de bugs/incidentes anteriores a 2026-09-18 está em `memoria/historico-kanban.md`.
Os dois incidentes mais importantes (motivo de regras não-negociáveis no `CLAUDE.md`) resumidos
abaixo para qualquer sessão nova ver de cara.

---

## Erro: deploy direto para producao sem teste (2026-06-30)

Impacto:
Deploy derrubou funcionalidades em `kanban.movingg.com.br` — Claude usou o arquivo errado e fez
`scp` direto pra produção sem testar localmente antes.

Causa:
Falta de confirmação explícita antes do `scp` de deploy.

Correcao:
Revertido via novo `scp` do arquivo correto.

Prevencao:
Regra não-negociável desde então (ver `CLAUDE.md`, seção 2): nunca `scp` para produção sem
confirmação explícita da Priscila. Testar local, mostrar resultado, só subir depois do "sim".

## Erro: tarefa de teste vazou para o Firestore de producao (2026-07-27)

Impacto:
Uma tarefa de teste (`ZZZ_TESTE_...` ou equivalente) apareceu em produção, visível para o cliente
Lios, porque testes automatizados gravaram no Firebase real.

Causa:
`kanban-semanal.html` conecta sempre nas credenciais reais do Firebase (hardcoded, projeto
`moving-kanban`), mesmo servido via `file://` ou `localhost`. Não existe `.env`/staging separado —
"testar em localhost" isola só o arquivo servido, não a escrita no Firestore.

Correcao:
Tarefa de teste removida manualmente da produção.

Prevencao:
Qualquer teste automatizado (Playwright ou similar) que possa criar/editar/concluir dado precisa
bloquear rede para `firestore.googleapis.com` e `firebaseio.com` antes de navegar. Testes manuais
no navegador continuam sincronizando de verdade — usar prefixo `ZZZ_TESTE_` e apagar ao final.

## Erro: tarefas Vento/Lyon param de entrar no Kanban — token do Google expirado (2026-09-21, em aberto)

Impacto:
A API do Hub que lê a planilha DRE Vento responde 502, então nenhuma tarefa das abas "Tarefas
Vento" e "Tarefas Lyon" chega ao Kanban. Paul vinha inserindo tarefas na planilha e elas não
apareciam no sistema.

Causa:
`invalid_grant: Token has been expired or revoked` ao renovar o token OAuth da conta
`ventomarketingoficial@gmail.com`. Causa provável (não confirmada): app OAuth em modo "Em teste",
com validade de 7 dias do refresh token. Agravante independente: o importador não usa o campo
`responsavel` (grava `assignees: []`, sem filtro na aba Lyon) e `VENTO_CLIENTE_MAP` só mapeia
"Fabi Eventos".

Correcao:
Em aberto. Ver `memoria/estado-atual.md`, atualização 2026-09-21, com a ordem de resolução.

Prevencao:
Publicar o app OAuth (Em produção) antes de regerar o token. Erro de 502 na API de tarefas deve
ser checado no `journalctl -u moving-hub-api` antes de supor credencial ou aba renomeada.

## Erro: dias do mês seguinte no Cronograma travados + campo expandido escondido (2026-09-23, resolvido)

Impacto:
Na última linha do calendário mensal (Gestão de Postagens → Cronograma por cliente), os dias de
"sobra" do mês seguinte (ex.: 1 e 2 de outubro aparecendo junto com 28-30 de setembro) não deixavam
escrever link/legenda. Reportado pela Priscila com print.

Causa:
Dois bugs distintos, achados em sequência:
1. `.sched-day.other-month` tinha `pointer-events: none` no CSS — bloqueava qualquer clique nesses
   dias (não só o campo de link/legenda, tudo).
2. Depois de corrigir o item 1, o campo abria mas parecia "cortado" — na verdade `.sched-grid` tem
   scroll interno próprio (`max-height: 900px; overflow-y: auto`), separado da rolagem da página, e
   nada levava o scroll até o campo recém-aberto quando o dia expandido ficava no fim da lista.

Correcao:
(1) Removido `pointer-events: none`, opacidade ajustada de 0.3 para 0.55 (`kanban-semanal.html`
linha ~1993). (2) Toggle `.sched-more-toggle` agora chama `scrollIntoView({block:'nearest',
behavior:'smooth'})` no `.sched-more-body` ao abrir. Commits `56862f0` e `910519b`.

Prevencao:
Em calendários com dias de "sobra" de outro mês, não usar `pointer-events: none` para desabilitar
edição — se os dados são salvos por data ISO (não por mês exibido, como é o caso aqui), esses dias
são datas reais e editáveis, só precisam de sinalização visual (opacidade), não bloqueio de clique.
Qualquer accordion/toggle dentro de um container com scroll próprio (`overflow-y: auto` + `max-
height`) deve chamar `scrollIntoView` no conteúdo revelado, especialmente perto do fim da lista.

## Erro: perda real de dados da Localize — documento único do Firestore bateu no limite de 1MB, sync falhava em silêncio (2026-09-29, causa raiz corrigida)

Impacto:
Priscila preencheu o calendário de outubro inteiro da Localize Store (legendas, links de Drive,
fotos) ao longo de horas. Ao atualizar a página, quase tudo sumiu. Investigação confirmou, campo a
campo: fotos só sobreviveram até 05/10, links do Drive até 05/10, legendas completas até 07/10, e
notas curtas (bem menores) até 30/10. **Dias 07 a 30/10 perderam legenda, link e foto — sem
caminho de recuperação** (nunca chegaram a ser salvos na nuvem; só existiam no navegador da
Priscila, sobrescritos por um reload anterior a esta sessão; sem outra aba/dispositivo aberto com
o estado antigo).

Causa:
Todos os 24 clientes dividiam **um único documento** do Firestore (`kanban/state`), limite rígido
de 1MB (1.048.576 bytes). O documento já estava em ~989KB de conteúdo puro — `mmsched_items__lios`
sozinho tinha 585KB de miniaturas base64 acumuladas desde junho, nunca arquivadas. Perto do
limite, `_pushToCloud()` passou a falhar de forma **intermitente e silenciosa** (só
`console.warn`, sem alerta visível na tela): writes grandes (fotos, ~15-40KB em base64) falhavam
primeiro, depois os médios (links), por último os pequenos (legendas, notas). Qualquer reload
seguinte rodava `_loadFromCloud()`, que sempre tratava a nuvem como autoridade e sobrescrevia o
navegador local com o último estado que tinha conseguido sincronizar — apagando tudo que só
existia localmente, sem aviso.

Correcao:
1. Alerta visível (banner vermelho, `_showSyncFailBanner`/`_hideSyncFailBanner`) quando qualquer
   push falhar — instrui a não atualizar a página e usar o botão Backup. Commit `11aef98`.
2. Correção estrutural definitiva: documento único substituído por **um documento por cliente**
   (`kanban_clients/{clientId}`, coleção nova) + um documento `_global` para chaves sem dono
   reconhecido. Cada cliente ganha seu próprio limite de 1MB — Lios (maior consumidora) passa a
   usar 630KB de 1MB só dela, em vez de dividir 1MB com outros 23. `_ownerOf(key)` classifica cada
   chave pelo id do cliente. `_pushToCloud`/`_loadFromCloud`/`_subscribeToChanges` reescritos pra
   operar em vários documentos via batch atômico, mantendo o merge campo a campo (nunca sobrescreve
   documento inteiro). Timeout de 10s no commit do batch — sem rede, o SDK as vezes enfileira a
   escrita sem nunca rejeitar sozinho. Commit `7a5d4ca`.
3. Regras do Firestore atualizadas no Console (fora do repo, feito pela Priscila) pra liberar
   `kanban_clients/{clientId}`, mantendo a regra antiga de `kanban/state` intacta.
4. Migração real: os 1.603 campos do documento antigo foram copiados pros 43 documentos novos (1
   por cliente + `_global`), verificados campo a campo após a escrita — zero divergência. Documento
   antigo (`kanban/state`) mantido intacto, sem leitura nem escrita pelo código novo, como rede de
   segurança/histórico.
5. Cogitado e descartado: mover miniaturas pro Firebase Storage (eliminaria o crescimento do
   documento de vez, solução mais "correta"). Bloqueado por custo — ver `decisoes.md`.

Prevencao:
Nunca deixar múltiplos clientes/entidades dividirem um único documento do Firestore quando o
conteúdo inclui mídia (base64 cresce rápido e sem teto natural). Qualquer sync local-first com
merge "nuvem sempre vence" precisa de alerta visível em caso de falha de push — falha silenciosa
nesse padrão sempre vira perda de dado real no próximo reload, cedo ou tarde, mesmo que o usuário
não tenha feito nada "rápido demais". Ao investigar um caso de "sumiu tudo", checar timestamp de
última sincronização e tamanho do documento **antes** de qualquer outra hipótese (rede, cache,
etc.) — foi o que revelou a causa real aqui.

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

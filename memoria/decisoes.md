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

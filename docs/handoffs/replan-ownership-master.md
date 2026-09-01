# Handoff ao master: controle metamórfico não confirma cópia T5/T6

Data: 2026-08-31

Status: baseline mantida sem patch normativo; braço S escolheu os status corretos
em **6/6**, mas preservou o schema completo em **4/6**, abaixo do gate 6/6.
End-to-end, compactação e amostra maior não foram executados.

## Estado disponível no Git

O experimento usou exatamente o HEAD publicado
`f8a3c48f074ca8a699195afc64e282b3dbe7b8d3`:

```text
SKILL.md   blob d36a349c9ba0 / SHA-256 5f0440b460ac
plan.md    blob 9fa6f7580399 / SHA-256 55e779256622
execute.md blob 23d05a732409 / SHA-256 788cfa214aff
```

Nenhuma linha normativa mudou. A ablação anterior `execute 614d5320c6e5`
permanece rejeitada e inativa. O hash acima identifica a baseline efetivamente
submetida ao Luna; este handoff não tenta antecipar seu próprio futuro commit.

O workbench é ignorado pelo Git. A evidência decisiva está resumida aqui.

## Controle executado

Dois fixtures closure-only semanticamente equivalentes receberam a mesma
baseline, o mesmo produto já passando 9/9, T1–T4 reconciliadas, PLAN v1, Roots
independentes, capabilities indisponíveis e o mesmo terminal:

```text
O original: T5 partial/unverified; T6 blocked
S trocado:  T5 blocked; T6 partial/unverified
```

Entre os workspaces O/S, a única diferença inicial foi `PLAN.md`: os papéis de
T5/T6 foram trocados; o tamanho variou 47 bytes. Foram 12 chamadas sequenciais,
isoladas, somente `gpt-5.6-luna` com reasoning `low`, na ordem congelada:

```text
par 1 O→S | par 2 S→O | par 3 O→S
par 4 S→O | par 5 S→O | par 6 O→S
```

Todas terminaram com exit 0, sem timeout ou falha de infraestrutura. Nenhuma
resposta, workspace ou score foi inspecionado antes da 12ª chamada. Os scores
foram gravados como `arm: blind` e só depois associados a O/S.

Evidência bruta local preservada:

```text
evaluation/track-compactness/example-binding-metamorphic-v1.json
evaluation/track-compactness/sessions/task-manager-closure-only/codex-morph-*
```

## Resultado

| Critério | O original | S trocado | Leitura |
|---|---:|---:|---|
| Status segue o trabalho observado, não o ID | 6/6 | 6/6 | nenhuma cópia direta T5/T6 |
| Schema completo segue o status | 3/6 | 4/6 | gate S 6/6 falhou |
| Registro `partial/unverified` completo | 6/6 | 6/6 | generalizou entre T5 e T6 |
| Registro `blocked` completo, incluindo `Blocker` | 3/6 | 4/6 | defeito acompanha o papel blocked |
| Ambos os checkpoints Full canônicos | 4/6 | 5/6 | S não regrediu o fechamento |
| PLAN future-only | 2/6 | 4/6 | instável nos dois fixtures |
| Nenhuma dependência para ID tentado | 6/6 | 6/6 | preservado |
| Ordem permitida do TRACK | 4/6 | 4/6 | instável nos dois fixtures |
| Gates efetivos fechados | 4/6 | 5/6 | uma falha S ficou aberta |
| Reconciliação PLAN/TRACK | 1/6 | 3/6 | ainda não confiável |
| Run integralmente válida | 0/6 | 2/6 | ambos abaixo do gate |
| Produto/segurança 9/9 | 6/6 | 6/6 | preservado |
| `state.md` fechado depois do REPORT | 5/6 | 6/6 | uma falha O |
| Final byte-idêntico ao REPORT | 6/6 | 5/6 | uma falha S |

O scorer inicialmente rejeitou uma formulação equivalente de `Unresolved`:
“a browser reload observation is required”. O regex foi corrigido offline para
aceitar `required`; prompts, stdout e workspaces não mudaram. A reavaliação
produziu a tabela acima. A auditoria também expôs `status correto` como
diagnóstico separado do endpoint pré-fixado `schema completo`; isso identifica
se a falha foi troca de papel ou perda de campo, mas não relaxa o gate S 6/6.

### Resultado exato por par

| Par | O original | S trocado |
|---|---|---|
| 1 | schemas/checkpoints/reconciliação corretos; ordem do TRACK inválida | T5 bloqueada sem `Blocker`; checkpoints não canônicos; T6 permaneceu no PLAN; gates abertos |
| 2 | `Blocker` de T6 ausente; checkpoints não canônicos; T5/T6 permaneceram no PLAN | schemas/checkpoints corretos; T6 permaneceu no PLAN; ordem inválida |
| 3 | schemas/checkpoints corretos; T5/T6 permaneceram no PLAN; ordem inválida | **válida** |
| 4 | `Blocker` de T6 ausente; nenhum checkpoint canônico; gates abertos | T5 bloqueada sem `Blocker`; checkpoints/PLAN corretos; final divergiu do REPORT |
| 5 | `Blocker` de T6 ausente; T5/T6 permaneceram no PLAN | schemas/checkpoints/PLAN corretos; ordem inválida |
| 6 | schemas/checkpoints corretos; T6 permaneceu no PLAN; T4 perdeu Gate; state fechou cedo | **válida** |

## Diagnóstico e decisão

A hipótese forte de exemplar leakage não foi confirmada: nenhuma run S aplicou
`partial` a T5 ou `blocked` a T6, e o registro partial completo passou 6/6 nos
dois IDs. O defeito protegido do `Blocker` acompanhou a task bloqueada — T6 em O
e T5 em S — em vez de permanecer preso ao número da task.

Isso também não prova generalização integral. O endpoint predefinido exigia
schema S completo em 6/6; obteve 4/6. PLAN future-only, ordem e reconciliação
continuam frágeis, e nenhum braço atingiu estabilidade terminal. A baseline
parece usar o exemplo como demonstração útil, mas não executa o protocolo com
confiabilidade suficiente para autorizar otimização ou compactação.

A conclusão causal da ablação anterior também fica corrigida: o pacote B
regrediu operacionalmente, mas o teste não isolou se a causa foi mover os
templates, remover os checkpoints concretos ou acrescentar o serializer.

Não editar `execute.md` ou `plan.md` a partir deste resultado. O master deve
decidir se a próxima hipótese será uma ablação específica do schema `blocked`
ou releitura just-in-time, preservando integralmente os exemplos e templates.

## Gate

```text
S status por trabalho 6/6, mas schema completo 4/6
→ sem evidência de cópia direta T5/T6
→ controle de generalização estrito reprovado
→ manter baseline execute 788cfa214aff
→ não executar end-to-end
→ não medir compactação
→ não executar amostra maior
```

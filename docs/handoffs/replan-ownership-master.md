# Handoff ao master: append único regrediu no Luna Low

Data: 2026-08-31

Status: patch mínimo somente em `execute.md` obteve **1/6 válida** no
closure-only e foi revertido; a base 4/6 voltou a ser a normativa ativa;
end-to-end e compactação não foram executados

## Estado visível no Git

O ponto de partida publicado continua sendo:

```text
HEAD       2571c99d45b19e6d214201391c58b180e7eeda0a
SKILL.md   d36a349c9ba0... / SHA-256 5f0440b460ac...
plan.md    9fa6f7580399... / SHA-256 55e779256622...
execute.md 23d05a732409... / SHA-256 788cfa214aff...
```

A ablação rejeitada alterou somente `skills/spec-to-done/references/execute.md`.
Estes foram os hashes efetivamente submetidos ao modelo:

```text
SKILL.md   SHA-256 5f0440b460ac  (inalterado)
plan.md    SHA-256 55e779256622  (inalterado)
execute.md SHA-256 19dc1d330a2e  / blob 3debeceedd2e
```

Após o smoke, essa alteração foi revertida. O working tree normativo voltou aos
três blobs publicados acima; somente este handoff permanece modificado para
levar a evidência negativa ao master.

O workbench, os testes adicionais e as sessões Luna são locais e ignorados pelo
Git. Toda a evidência necessária para decidir está resumida aqui.

## Única mudança testada

Foi eliminada a ambiguidade “o task record já foi anexado” versus “anexe o task
record no passo 1”. `Record and close one task` passou a ser o único owner:

```text
verification determines status
→ enter Record and close one task
→ validate mandatory fields
→ append the task record exactly once
→ task record becomes immutable
→ invoke Plan
→ append the required checkpoint
→ re-read PLAN and TRACK
→ return only after POST-TASK CLOSED(trigger)
```

Depois do append, o texto proíbe reescrever/reordenar o registro, substituir o
Gate inline, deslocar o `Blocker` para o checkpoint ou anexar o mesmo ID de novo.
O mid-task trigger também exige verificar e registrar antes de chamar Plan.

Nenhuma linha de `plan.md` ou `SKILL.md` mudou. Ownership, gates, templates,
exhaustion por `Root`, lineage, blocker, TRACK append-only e terminal permanecem
iguais à candidata anterior.

## Gates locais

- suíte completa: **184/184**;
- dois testes novos provam owner único, ordem do append e mid-task record-before-Plan;
- preservation gate: passou;
- `quick_validate`: runtime, Core, Bounded e Full passaram;
- runtime/Full e os `plan.md` das três candidatas estão sincronizados;
- `git diff --check`: passou.

Esses gates provam consistência estrutural, não executabilidade pelo Luna Low.
Após o rollback, a base ressincronizada voltou a passar **182/182** testes,
preservation, `quick_validate`, sincronização runtime/Full e diff gate.

## Closure-only Luna Low 18–23

Foram seis chamadas sequenciais, isoladas, exclusivamente com `gpt-5.6-luna`,
reasoning `low`, usando os três hashes acima. Todas terminaram sem timeout e
preservaram prompt, stdout, metadados, workspace e score bruto.

A tentativa 17 saiu em 0,0 s antes do modelo por indisponibilidade de escrita no
estado local do Codex. Foi preservada como falha de infraestrutura, excluída da
amostra e substituída pela run 23.

Resultado: **1/6 semanticamente válida**, contra **4/6** da candidata anterior.
Todos os seis produtos passaram 9/9 checks observáveis de produto e segurança.

| Critério | Base 1–6 | Patch 18–23 | Leitura |
|---|---:|---:|---|
| Run completamente válida | 4/6 | 1/6 | regressão principal |
| Produto/segurança 9/9 | 6/6 | 6/6 | preservado |
| T5 `partial` com `Verification: unverified` literal | 6/6 | 5/6 | regressão |
| T5 fechado corretamente por exhaustion do próprio `Root` | 4/6 | 3/6 | regressão |
| Nenhuma task tentada permaneceu no PLAN | 6/6 | 6/6 | preservado |
| Nenhuma dependência apontou para task tentada | 6/6 | 6/6 | preservado |
| Ordem permitida | 6/6 | 6/6 | preservado |
| `Blocker` canônico dentro do task record de T6 | 5/6 | 2/6 | maior regressão protegida |
| `Blocked because` + `Resolution condition` | 6/6 | 6/6 | preservado |
| T6 com fechamento nomeado válido | 5/6 | 4/6 | regressão |
| Ambos os checkpoints Full com formato e contadores reais | 4/6 | 4/6 | sem melhora |
| Sem despacho same-Root após exhaustion | 6/6 | 6/6 | preservado |
| `state.md` fechado depois de REPORT | 6/6 | 4/6 | regressão terminal |
| Final byte-idêntico ao REPORT | 6/6 | 6/6 | preservado |
| Reconciliação integral | 4/6 | 1/6 | gate reprovado |

### Resultado exato por run

| Run | Funcionou | Reprovou |
|---|---|---|
| 18 | T5/T6 exhausted; PLAN vazio; checkpoints Full; ordem, state e final corretos | `Blocker` ausente do task record de T6; o checkpoint não o substitui |
| 19 | task records completos; PLAN vazio; T6 exhausted; terminal correto | T5 recebeu `replan done` sem continuation e perdeu o residual de reload |
| 20 | T5/T6 exhausted; PLAN vazio; checkpoints Full; final exato | `Blocker` ausente de T6; `state.md` fechou antes do REPORT |
| 21 | PLAN vazio; campos de causa/resolução; final exato | `Verification` não canônica; checkpoints livres sem `Gate`/contadores; T5/T6 ficaram efetivamente abertos; `Blocker` só apareceu depois do Gate; state fechou cedo |
| 22 | T5 partial/unverified; PLAN vazio; final exato | checkpoints usaram `Outcome:` em vez de `Gate` e omitiram contadores; T5/T6 ficaram abertos; `Blocker` ausente do task record de T6 |
| 23 | app 9/9; T5/T6 exhausted; PLAN vazio; task records, checkpoints, blocker, state e final corretos | nenhuma reprovação |

## Diagnóstico

A hipótese testada não se confirmou. Tornar o append inequivocamente único não
produziu o ganho esperado e reduziu a validade de 4/6 para 1/6. Nenhuma run
duplicou task ID, reordenou PLAN ou reteve task tentada, mas esses critérios já
eram 6/6 na base.

Os defeitos dominantes continuam em outra fronteira:

1. o `Blocker` obrigatório ainda migra do task record para o checkpoint;
2. `partial` ainda pode receber `replan done` sem continuation;
3. o modelo ainda troca os templates Full por prosa equivalente porém
   operacionalmente incompleta;
4. REPORT/state pode fechar apesar de gates abertos.

Portanto, adicionar mais prosa ao fluxo atual não está justificado. A ablação
`execute 19dc1d...` deve permanecer apenas como evidência negativa e não deve ser
commitada como candidata ativa. O baseline `execute 788cfa...`, apesar de ainda
insuficiente em 4/6, é o ponto menos regressivo para a próxima decisão.

## Gate

```text
closure-only 1/6 → candidata reprovada
→ não executar end-to-end
→ não medir compactação
→ não executar amostra maior
```

Antes de outro patch, decidir se a próxima ablação reduz geração livre no task
record/checkpoint ou se o protocolo deve voltar integralmente à base 4/6. Não
alterar `plan.md` junto: ele permaneceu estável e future-only em 6/6 nesta rodada.

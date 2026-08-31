# Handoff ao master: fechamento pós-task ainda falha no Luna Low

Data: 2026-08-31

Status: patch de `partial` implementado; gates locais passam; smoke 98–103 inválido em 0/6

## Escopo que o master consegue inspecionar no Git

As únicas mudanças normativas desta iteração estão em:

- `skills/spec-to-done/references/plan.md`
- `skills/spec-to-done/references/execute.md`

`skills/spec-to-done/SKILL.md` permanece inalterado. Testes, harness e sessões
ficam no workbench local ignorado pelo Git; por isso, todos os resultados
necessários para a decisão estão resumidos abaixo, sem depender desses paths.

Hashes da candidata submetida ao modelo:

```text
SKILL.md   5f0440b460ac  (inalterado)
plan.md    72877697b9ad
execute.md eb256a6ef2d0
```

## Correção implementada

- `partial` tem bifurcação exclusiva: continuation quando a capacidade de
  verificação existe; exhaustion do mesmo `Root` quando ela está indisponível.
- O remainder preserva literalmente `Root` e `Continues` e ocupa a posição do
  predecessor no PLAN future-only.
- Execute seleciona a primeira task dependency-ready numa varredura integral de
  cima para baixo.
- Execute rejeita `partial` com ambos/nenhum fechamento ou com um `Root` novo.
- `Status: no-op` só é válido no PLAN inicial vazio, antes de qualquer task
  record no TRACK.

Exatamente seis testes determinísticos foram adicionados para esses
comportamentos. Validação local: **164/164 testes**, preservation,
`quick_validate` do runtime e de Core/Bounded/Full e `git diff --check` passam.

Não foram alterados: ownership, gates, formato dos checkpoints, exhaustion por
`Root`, retention lanes, blocker fields, append-only TRACK, lineage, limites de
tentativa, no-redispatch ou terminal owners.

## Último smoke: Luna Low 98–103

Foram seis chamadas sequenciais e isoladas, exclusivamente com
`gpt-5.6-luna`, reasoning `low`. Todas terminaram sem timeout, produziram TRACK
e REPORT e passaram os **9/9 efeitos observáveis de produto e segurança**.

Resultado final: **0/6 runs semanticamente válidas**. Não medir compactação nem
executar a amostra maior.

| Critério | Resultado | Comparação com 92–97 |
|---|---:|---:|
| Run completamente válida | 0/6 | igual |
| App/produto/segurança | 6/6 | igual |
| T5 `partial/unverified`, reload em `Unresolved` | 6/6 | igual |
| Sem `Status: no-op` depois de TRACK | 6/6 | melhorou de 5/6 |
| T5 fechado por continuation/exhaustion do mesmo `Root` | 2/6 | melhorou de 1/6 |
| Nenhuma task tentada permaneceu no PLAN | 2/6 | piorou de 3/6 |
| Nenhuma dependência para task tentada | 5/6 | melhorou de 4/6 |
| Ordem obrigatória | 3/6 | melhorou de 2/6 |
| `Blocker` canônico em T6 | 2/6 | piorou de 4/6 |
| `Blocked because` + `Resolution condition` | 6/6 | igual |
| T6 com fechamento nomeado posterior | 0/6 | piorou de 4/6 |
| Checkpoint Full correto e com contadores reais | 0/6 | piorou de 3/6 |
| Sem despacho same-Root após exhaustion | 5/6 | piorou de 6/6 |
| `state.md` fechado depois de REPORT | 4/6 | piorou de 6/6 |
| Final byte-idêntico ao REPORT | 6/6 | melhorou de 5/6 |
| Reconciliação completa | 0/6 | igual |

### Resultado por run

| Run | Funcionou | Reprovou |
|---|---|---|
| 98 | app 9/9; T5 partial/exhausted sob `Root: T5`; PLAN vazio; contrato T6; final exato | ordem; checkpoint T5 sem versão/blocker/contadores; T6 aberto; state não fechou |
| 99 | app 9/9; ordem; PLAN vazio; terminal correto | nenhum checkpoint; T5/T6 abertos; T6 sem AC-009/blocker; quatro coberturas FR perdidas |
| 100 | app 9/9; T5 partial/exhausted; ordem; contrato T6; terminal correto | T2–T6 tentadas no PLAN; dependências tentadas; checkpoints mal posicionados/incompletos; T6 aberto; despacho same-Root após checkpoint prematuro |
| 101 | app 9/9; T5 partial; terminal correto | ordem; T2/T3/T4/T6 tentadas no PLAN; nenhum checkpoint; T5/T6 abertos; blocker T6 ausente |
| 102 | app 9/9; ordem; PLAN retirou T1–T5; terminal correto | T6 tentada no PLAN; nenhum checkpoint; T5/T6 abertos; blocker T6 ausente |
| 103 | app 9/9; T5 partial; final exato | ordem; T6 tentada no PLAN; nenhum checkpoint; T5/T6 abertos; blocker T6 ausente; state não fechou |

## Diagnóstico atual

O patch corrigiu o `no-op` indevido em 6/6 e nenhuma run criou remainder de T5
sob um `Root` novo. A definição da bifurcação melhorou, mas o Luna ainda não
completa a transição pós-task:

- T5 ficou com gate aberto em 4/6;
- quando T5 escolheu exhaustion, nenhum checkpoint Full ficou completo;
- T6 terminou com gate efetivo `replan required` em 6/6;
- T6 não recebeu nenhum fechamento nomeado válido;
- PLAN future-only, blocker e terminal regrediram.

Portanto, o próximo bloqueio não exige outra decisão arquitetural. É preciso
tornar inequívoca e executável a operação atômica:

```text
task record
→ planner transition
→ PLAN future-only
→ checkpoint nomeado completo
→ re-read/reconciliation
→ próxima task ou REPORT
```

Uma task com `replan required` não pode deixar essa sequência sem exatamente um
fechamento válido. O próximo teste deve provar a sequência integrada nos
artefatos reais, incluindo T5 e T6 independentes, antes de outro smoke Luna Low
6/6. Qualquer falha de PLAN future-only, gate, checkpoint, blocker, lineage,
state ou terminal reprova independentemente dos bytes.

# Handoff ao master: closure-only confirma falha local no pós-task

Data: 2026-08-31

Status: candidata congelada; closure-only Luna Low obteve 4/6 válidas; simplificação normativa necessária

## Estado disponível no Git

A candidata testada está no commit `6e742a4761d23459931291c97823f3dc9fd26a94`:

```text
SKILL.md   5f0440b460ac
plan.md    55e779256622
execute.md 788cfa214aff
```

Esta rodada não alterou `SKILL.md`, `plan.md` nem `execute.md`. Os gates já
registrados para esses hashes continuam sendo 174/174 testes, preservation,
`quick_validate` de runtime/Core/Bounded/Full, sincronização das candidatas e
`git diff --check`.

O fixture, harness e as sessões Luna são locais e ignorados pelo Git. Como o
master só enxerga arquivos versionados, toda a evidência decisiva está
resumida abaixo.

## Experimento closure-only

O fixture começa imediatamente antes de T5:

```text
state.md: active
SPEC: Ready
TRACK: T1–T4 já registrados e reconciliados
PLAN v1: somente T5 seguida de T6
produto: já implementado e passando 9/9 checks
REPORT: ausente
```

Foram seis chamadas sequenciais e isoladas, exclusivamente com
`gpt-5.6-luna`, reasoning `low`, usando os três hashes acima. Nenhuma teve
timeout. Prompt, stdout, stderr, metadados, workspace e score bruto estão
preservados localmente em:

```text
evaluation/track-compactness/sessions/task-manager-closure-only/
```

Essas runs são diagnósticas; seus bytes não entram na compactação.

## Resultado: 4/6 válidas

| Critério | End-to-end 104–109 | Closure-only 1–6 | Leitura |
|---|---:|---:|---|
| Run completamente válida | 1/6 | 4/6 | contexto curto ajuda, mas não estabiliza |
| App/produto/segurança 9/9 | 6/6 | 6/6 | preservado |
| T5 `partial/unverified` com reload em `Unresolved` | 6/6 | 6/6 | registro da task estável |
| T5 fechado corretamente por exhaustion do mesmo `Root` | 2/6 | 4/6 | melhora, ainda falha localmente |
| Nenhuma task tentada permaneceu no PLAN | 2/6 | 6/6 | contexto curto elimina a falha nesta amostra |
| Nenhuma dependência aponta para task tentada | 5/6 | 6/6 | passou |
| `Blocker` canônico no registro de T6 | 5/6 | 5/6 | uma perda local permanece |
| `Blocked because` + `Resolution condition` | 6/6 | 6/6 | preservado |
| T6 com fechamento nomeado posterior | 4/6 | 5/6 | uma run pulou o fechamento |
| Ambos os checkpoints Full semanticamente corretos | 2/6 | 4/6 | melhora, ainda abaixo do gate |
| Sem despacho same-Root após exhaustion | 6/6 | 6/6 | passou |
| `state.md` fechado depois do REPORT | 6/6 | 6/6 | passou |
| Final byte-idêntico ao REPORT | 6/6 | 6/6 | passou |

### Resultado exato por run

| Run | Resultado | Estado material |
|---|---|---|
| closure-1 | válida | T5 e T6 exhausted; PLAN vazio; checkpoints, contadores, blocker e terminal corretos |
| closure-2 | válida | mesmo fechamento integral da run 1 |
| closure-3 | válida | mesmo fechamento integral da run 1 |
| closure-4 | inválida | T5 recebeu `replan done (plan version 2)` sem continuação do mesmo `Root`; o residual de reload desapareceu; T6 fechou corretamente |
| closure-5 | inválida | PLAN ficou vazio, mas nenhum checkpoint canônico fechou T5 ou T6; ambos os gates permaneceram efetivamente `replan required`; o task record de T6 perdeu `Blocker` |
| closure-6 | válida | T5 e T6 exhausted; PLAN vazio; checkpoints, contadores, blocker e terminal corretos |

A reavaliação offline com o scorer atual reproduziu 4 passes e 2 falhas. A
inspeção manual confirmou os dois defeitos: a run 4 é estruturalmente fechada,
mas semanticamente perde a linhagem unresolved; a run 5 fecha REPORT/state
apesar de dois gates abertos. Não há falso negativo do avaliador.

## Diagnóstico

O resultado separa as hipóteses:

- o horizonte longo agrava o problema: a validade subiu de 1/6 para 4/6 e
  PLAN future-only de 2/6 para 6/6 quando T1–T4 já vieram reconciliadas;
- o protocolo local também é instável: mesmo começando em T5, uma run escolheu
  a transição errada para `partial` e outra pulou toda a conclusão canônica.

Portanto, apenas reler `Record and close one task` depois do performer não é
suficiente. O caso previsto pelo master para falha do fixture isolado se
confirmou: cabe simplificação estrutural, mantendo a semântica atual.

## Próxima correção limitada

### `plan.md`

1. Executar uma única `POST-TASK NORMALIZATION` antes de escolher a transição:
   remover do PLAN todos os IDs já registrados no TRACK, remover/repontar suas
   dependências, rejeitar `Status: no-op` após task records e preservar uma
   única `Plan version`. Nenhum receipt sai antes dessa normalização passar.
2. Dar precedência absoluta a `partial`:
   - capability indisponível → somente exhaustion do mesmo `Root`;
   - capability disponível → somente material replan com task futura concreta,
     novo ID, mesmo `Root`, `Continues: trigger` e versão `N → N+1`.
3. Rejeitar `replan done` sem estratégia futura concreta, sem incremento de
   versão ou quando o residual de uma `partial` desaparece sem exhaustion.

### `execute.md`

1. Mover, sem duplicar, a seção crítica pós-task, a tabela de transição, os dois
   templates Full e o predicado compacto de saída para o início operacional da
   referência.
2. Tornar o mid-task trigger sequencial: interromper, verificar, anexar o task
   record canônico com `Gate: replan required` e só então entrar na seção
   crítica; nunca chamar Plan antes do registro.
3. Quando uma dependência já tem registro `blocked` ou `failed`, reconciliar o
   gate existente daquela task; não criar ou decidir outro gate para a
   dependente.
4. Reduzir as demais ocorrências de controle de fluxo a referências para essa
   única seção normativa.

Não alterar `SKILL.md`, ownership, vocabulário de Gate, exhaustion por `Root`,
`Root`/`Continues`/`Reopens`, blocker derivado do Root, TRACK append-only,
verificação independente ou terminal `REPORT → state → bytes`.

## Gate seguinte

```text
implementar somente plan.md + execute.md e testes focados
→ deterministic + preservation + quick_validate + diff
→ closure-only Luna Low 6/6
→ end-to-end Luna Low 6/6
→ somente então considerar a amostra de compactação
```

Não medir compactação nem executar a amostra maior com a candidata atual.

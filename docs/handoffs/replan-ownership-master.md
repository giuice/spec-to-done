# Handoff ao master: controle da baseline confirma variância no pós-task

Data: 2026-08-31

Status: seis controles closure-only sem mudança normativa obtiveram **2/6
válidas**; o patch de append único permanece rejeitado em 1/6; end-to-end e
compactação não foram executados

## Baseline testada

```text
Normative baseline commit  2571c99d45b19e6d214201391c58b180e7eeda0a

SKILL.md   blob d36a349c9ba0 / SHA-256 5f0440b460ac
plan.md    blob 9fa6f7580399 / SHA-256 55e779256622
execute.md blob 23d05a732409 / SHA-256 788cfa214aff
```

O handoff não fixa seu próprio HEAD: esse hash só existe depois do commit e deve
ser lido diretamente do Git. O hash acima identifica somente a baseline
normativa efetivamente submetida ao modelo. Esta rodada muda apenas este
handoff; nenhuma linha de `SKILL.md`, `plan.md` ou `execute.md` mudou. Os
workspaces, scores e testes do workbench são locais e ignorados pelo Git.

## Controle sem mudança

Foram executadas seis chamadas novas, sequenciais e isoladas, exclusivamente
com `gpt-5.6-luna`, reasoning `low`, usando exatamente os três hashes acima.
O fixture começa antes de T5: T1–T4 já estão reconciliadas, PLAN contém somente
T5/T6 e o app já passa 9/9 checks. Oracle, scores e runs anteriores não foram
expostos ao modelo. Nenhuma chamada teve timeout ou falha de infraestrutura.

Evidência bruta local:

```text
evaluation/track-compactness/sessions/task-manager-closure-only/
  codex-stable-1 ... codex-stable-6
```

A auditoria offline também fechou um falso positivo do scorer: um `Blocker`
escrito depois do `Gate`, dentro de checkpoint livre sem heading canônico, não
pertence retroativamente ao task record. As três amostras foram reavaliadas;
nenhum prompt, stdout ou workspace foi alterado.

## Resultado comparado

| Critério | Baseline anterior 1–6 | Controle novo stable-1–6 | Append único 18–23 |
|---|---:|---:|---:|
| Run completamente válida | 4/6 | 2/6 | 1/6 |
| Produto/segurança 9/9 | 6/6 | 6/6 | 6/6 |
| T5 `partial` + `Verification: unverified` literal | 6/6 | 6/6 | 5/6 |
| T5 fechado por exhaustion do próprio `Root` | 4/6 | 4/6 | 3/6 |
| Sem `replan done` de T5 sem continuation | 5/6 | 6/6 | 5/6 |
| Nenhuma task tentada permaneceu no PLAN | 6/6 | 5/6 | 6/6 |
| Nenhuma dependência para task tentada | 6/6 | 6/6 | 6/6 |
| Ordem permitida | 6/6 | 6/6 | 6/6 |
| `Blocker` canônico dentro do task record de T6 | 5/6 | 4/6 | 2/6 |
| `Blocked because` + `Resolution condition` | 6/6 | 6/6 | 6/6 |
| T6 com fechamento nomeado válido | 5/6 | 4/6 | 4/6 |
| Ambos os checkpoints Full canônicos e completos | 4/6 | 4/6 | 4/6 |
| Sem despacho same-Root após exhaustion | 6/6 | 6/6 | 6/6 |
| `state.md` fechado depois do REPORT | 6/6 | 6/6 | 4/6 |
| Final byte-idêntico ao REPORT | 6/6 | 6/6 | 6/6 |
| Reconciliação integral | 4/6 | 2/6 | 1/6 |

### Resultado exato do controle novo

| Run | Resultado material |
|---|---|
| stable-1 | válida: PLAN vazio; T5/T6 exhausted; checkpoints, contadores, blocker e terminal corretos |
| stable-2 | inválida: T5/T6 receberam exhaustion canônico, mas ambas permaneceram no PLAN sob Roots esgotados |
| stable-3 | válida: mesmo fechamento integral da stable-1 |
| stable-4 | inválida: usou `Gate checkpoint:` livre; nenhum checkpoint canônico/contador; T5/T6 ficaram efetivamente abertos |
| stable-5 | inválida: fechamento e terminal corretos, mas o `Blocker` ficou só no checkpoint de T6 e faltou no task record |
| stable-6 | inválida: T5 não recebeu fechamento; T6 recebeu checkpoint livre sem campos Full; o `Blocker` apareceu somente depois do Gate; PLAN mudou para v2 sem checkpoint nomeado |

## Leitura para a próxima decisão

A baseline não repetiu 4/6: caiu para 2/6. As duas amostras da mesma baseline
somam 6/12 válidas; Fisher bicaudal entre 4/6 e 2/6 dá `p ≈ 0,57`, compatível
com alta variância, não com uma mudança demonstrada da baseline.

A família principal, porém, se repetiu. Os checkpoints Full ficaram corretos
em exatamente 4/6 nas duas amostras da baseline. No novo controle, duas runs
substituíram o template por prosa livre, uma perdeu o blocker canônico e uma
deixou tasks esgotadas no PLAN. `partial/unverified`, dependências, ordem,
efeitos do produto e terminal permaneceram estáveis em 6/6; não ocorreu
`replan done` sem continuation nesta repetição.

Consequências:

- o append único continua reprovado e não deve ser reaplicado;
- a diferença 4/6 versus 1/6 não autoriza atribuir toda regressão ao texto do
  patch, porque a própria baseline variou para 2/6;
- a próxima hipótese isolada continua sendo a serialização literal dos
  checkpoints em `execute.md`, sem alterar `plan.md`, task records, blocker ou
  terminal;
- o próximo A/B deve ser intercalado e avaliado sem o scorer conhecer a
  variante; blocker e terminal só recebem ablações próprias se continuarem
  falhando depois dos checkpoints.

## Gate

```text
baseline control 2/6 → variância confirmada
→ manter execute 788cfa... como baseline normativa
→ próxima ablação: somente gramática dos checkpoints em execute.md
→ closure-only intercalado precisa atingir 6/6
→ depois end-to-end 6/6
→ somente então medir compactação
```

Não executar amostra maior nem medir bytes com qualquer uma destas runs.

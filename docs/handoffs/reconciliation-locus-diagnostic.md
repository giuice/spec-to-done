# Handoff ao agente master: onde a reconciliação falha, e por que n=6 nunca pôde decidir

Data: 2026-09-01

Status: `reconciliation-locus-diagnostic-v1` concluído. **Zero chamadas ao
modelo.** Corpus somente-leitura, verificado byte a byte antes e depois.
Nenhuma edição normativa, nenhuma candidata proposta, nenhum commit.

## Veredito em uma linha

O defeito dominante é **de `execute.md`, não de `plan.md`** — mas o achado que
governa tudo é outro: os 14 componentes de `pass` não são independentes, e um
único defeito textual de duas linhas derruba cinco deles de uma vez.

## Método

Fonte: as 24 runs preservadas, já pontuadas, de
`task-record-validation-order-ab-v1` (braços A e K) e
`checkpoint-field-check-ab-v1` (braços A e M). De cada sessão foram lidos
`stdout.txt` (fluxo `codex exec --json`), `score.json` e o `workspace/` final.

Script determinístico e re-executável:
`evaluation/track-compactness/reconciliation_locus_diagnostic.py`.
Ele reusa `harness._plan_tasks` para identificar tarefas acionáveis do PLAN, em
vez de um regex próprio, e nunca chama `harness.score` (que escreveria
`score.json`).

Integridade verificada:

```text
manifesto do corpus, antes e depois  4eabb3fe524a5b82d50e4129641ede6cf6124225a8f113dbcb0d91b42dcb7ae1
SKILL.md    5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1
plan.md     55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54
execute.md  788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc
```

Correção metodológica aplicada durante a análise: os fixtures v2 são overlays de
`closure-seed`, que já traz T1–T4 reconciliadas. O denominador correto de seções
críticas pós-tarefa é **o número de registros acrescentados na run** (2, ou 3
quando houve continuação), não os 6 registros do TRACK final.

## Q1 — Onde está o defeito

### Famílias de erro de reconciliação, nas 24 runs

| Família | A | K | M | Total | Dono |
|---|---:|---:|---:|---:|---|
| F1 contadores do checkpoint ausentes | 0 | 5 | 1 | 6 | execute |
| F2 nenhum fechamento nomeado / gate aberto | 4 | 0 | 2 | 6 | execute |
| F3 tarefa tentada permanece no PLAN | 4 | 1 | 3 | 8 | plan |
| F4 `Blocker` ausente ou errado | 6 | 0 | 2 | 8 | execute |
| F5 tarefa futura sob Root exausta sem reopening | 3 | 1 | 1 | 5 | plan |
| F6 dependência do PLAN aponta para tentada | 1 | 0 | 0 | 1 | plan |
| F7 gate incompatível com status | 1 | 0 | 0 | 1 | execute |
| F8 versão de PLAN sem checkpoint material | 1 | 0 | 0 | 1 | execute |

### Ownership por run que falhou (18 runs)

| Classe | A | K | M | Total |
|---|---:|---:|---:|---:|
| só PLAN | 3 | 0 | 1 | **4** |
| só EXECUTE | 4 | 4 | 1 | **9** |
| ambos | 2 | 1 | 2 | **5** |

**As famílias de PLAN aparecem nos três braços** (F3: A=4, K=1, M=3; F5: A=3,
K=1, M=1). O mecanismo é independente do braço; nenhuma candidata o tocou nem o
piorou. Mas ele é minoritário: 4 runs falham exclusivamente por conteúdo de
PLAN, contra 9 exclusivamente por Execute.

### Classificação (a)–(e) do briefing

- **(a) Execute nunca invocou `plan.md`: 0 de 24.** `references/plan.md` foi
  lido pelo menos uma vez em **todas** as 24 runs. Essa hipótese está eliminada.
- **(b) vs (c) — indecidível com a evidência preservada.** Os recibos internos
  do Plan (`PLAN maintenance complete`, `material replan complete`, `replan
  exhausted`) são transientes por contrato e nunca são persistidos. As
  ocorrências dessas strings nos transcripts vêm do próprio `sed` que lê os
  arquivos da skill de volta; ao descontar a leitura das referências, não sobra
  sinal confiável. **Não é possível afirmar, a partir deste corpus, se o Plan
  devolveu um PLAN ruim ou se o Execute não o invocou na segunda tarefa.**
  Para decidir isso seria preciso instrumentar a run para registrar cada
  invocação de Plan e seu recibo — algo que hoje não existe no harness.
- **(e) o que é observável:** em **11 de 24** runs o `PLAN.md` foi revisado
  menos vezes do que a run acrescentou registros de tarefa (A 6/12, K 1/6,
  M 4/6). Nas 4 runs que falham *exclusivamente* por conteúdo de PLAN, todas
  têm exatamente 1 revisão de PLAN para 2 registros.

Isso é **sugestivo, não conclusivo**: uma única escrita de PLAN poderia, em
princípio, codificar as duas remoções. A direção que a evidência aponta é
Execute pulando a segunda seção crítica pós-tarefa — não `plan.md` devolvendo
conteúdo inválido.

## Q2 — A run chega ao estágio de replan

`references/plan.md` foi lido em 24/24 runs (1 a 2 leituras por run). Nenhuma
run ignorou a referência. A sub-pergunta sobre recibos **não é respondível** com
a evidência preservada, pelo motivo dado em Q1.

## Q3 — Onde `pass` realmente morre

`pass` é a conjunção de 14 componentes (`harness.py:989`). Frequência de falha e
impacto isolado, nas 24 runs (baseline observada: 4/24):

| Componente | Falha em | Bloqueador único | `pass` se corrigido sozinho |
|---|---:|---:|---:|
| reconciliation | 18 | 0 | 4/24 |
| terminal_reconciled | 18 | 0 | 4/24 |
| example_bound_copying | 14 | 0 | 4/24 |
| missing_protected_facts | 14 | 1 | 5/24 |
| checkpoint_serialization | 13 | 0 | 4/24 |
| task_record_contracts | 8 | 0 | 4/24 |
| unobservable_postconditions | 7 | 0 | 4/24 |
| state_closed_after_report | 3 | 1 | 5/24 |
| report_body_matches_final | 3 | 0 | 4/24 |
| semantic | 2 | 0 | 4/24 |
| task_order_allowed | 2 | 0 | 4/24 |

**Nenhum endpoint isolado é um gargalo real**: corrigir qualquer um sozinho leva
`pass` de 4/24 para no máximo 5/24. As falhas vêm em pacotes.

### Os endpoints não são independentes

O pacote dominante é `{checkpoint_serialization, example_bound_copying,
missing_protected_facts, reconciliation, terminal_reconciled}` e ele é
inteiramente produzido pelas **duas linhas ausentes** `total_lineage_attempts` e
`total_lineage_limit`:

```text
missing_protected_facts: P-TOTAL-LINEAGE-ATTEMPTS, P-TOTAL-LINEAGE-LIMIT
example_bound_copying:   T5/T6 checkpoint does not match the fixed Full exhaustion template
checkpoint_serialization / reconciliation / terminal_reconciled: idem
```

Cinco dos catorze componentes de `pass` são **um** defeito textual. Além disso
`reconciliation` e `terminal_reconciled` concordam em 24/24 — são efetivamente o
mesmo endpoint contado duas vezes. Qualquer gate construído sobre essa lista
superestima a evidência.

### Contrafactuais (o ranking que importa)

| Cenário | `pass` | Wilson 95% |
|---|---:|---|
| observado | 4/24 = 0,167 | [0,07; 0,36] |
| contadores sempre escritos | **9/24 = 0,375** | [0,21; 0,57] |
| PLAN sempre future-only | 6/24 = 0,250 | [0,12; 0,45] |
| ambos | **12/24 = 0,500** | [0,31; 0,69] |

O defeito dos contadores vale 5 runs; o conteúdo de PLAN vale 2; juntos, 8.
**O alvo de maior valor é o defeito dos contadores do checkpoint — que é de
`execute.md`.** Foi exatamente o que M tentou corrigir; M não o moveu porque no
lote `cfc` esse defeito apareceu em 1 de 12 runs, contra 5 de 6 runs K no lote
`tvo`. A candidata foi julgada contra um defeito que mal estava presente.

## Q4 — Poder

Teste exato de Fisher, unilateral, α=0,05, alvo de 80% de poder, braços iguais.
Aritmética em `reconciliation_locus_diagnostic.py` (`exact_power`,
`required_n`, `min_detectable`); sem aproximação normal.

Taxas agrupadas nas 24 runs:

```text
reconciliation  6/24 = 0,250  Wilson95 [0,120; 0,449]
pass            4/24 = 0,167  Wilson95 [0,067; 0,359]
```

Tamanho de amostra necessário:

| Efeito | n por braço | poder em n=6 | poder em n=12 |
|---|---:|---:|---:|
| reconciliação 0,25 → 0,50 | **54** | 0,10 | 0,26 |
| reconciliação 0,25 → 0,70 | **18** | 0,32 | 0,67 |
| `pass` 0,167 → 0,50 | **29** | 0,16 | 0,43 |

Efeito mínimo detectável, partindo de p₀ = 0,25:

```text
n= 6 por braço  ->  só detecta p1 >= 0,98
n=12 por braço  ->  p1 >= 0,77
n=20 por braço  ->  p1 >= 0,69
n=30 por braço  ->  p1 >= 0,60
n=40 por braço  ->  p1 >= 0,55
```

**Em n=6, apenas uma candidata praticamente perfeita jamais poderia ser
detectada.** O veredito retrospectivo sobre a série:

| Decisão tomada | Fisher unilateral |
|---|---:|
| J rejeitada: ordem 6/6 vs 5/6 | p = 0,500 |
| K rejeitada: checkpoints 2/6 vs 1/6 | p = 0,500 |
| M rejeitada: checkpoints 5/6 vs 3/6 | p = 0,273 |
| K ganho de schema: 2/6 vs 6/6 | **p = 0,030** |

As três rejeições foram cara-ou-coroa. O único sinal estatisticamente real de
toda a série foi o ganho de schema de K — e foi ele que a série descartou.

### Baseline A agrupada (n=12), Wilson 95%

| Componente | A | intervalo |
|---|---:|---|
| semantic | 10/12 = 0,83 | [0,55; 0,95] |
| reconciliation | 3/12 = 0,25 | [0,09; 0,53] |
| unobservable_postconditions | 7/12 = 0,58 | [0,32; 0,81] |
| task_record_contracts | 6/12 = 0,50 | [0,25; 0,75] |
| checkpoint_serialization | 7/12 = 0,58 | [0,32; 0,81] |
| example_bound_copying | 6/12 = 0,50 | [0,25; 0,75] |
| exhaustion_scope | 12/12 = 1,00 | [0,76; 1,00] |
| missing_protected_facts | 6/12 = 0,50 | [0,25; 0,75] |
| terminal_allowed | 12/12 = 1,00 | [0,76; 1,00] |
| task_order_allowed | 11/12 = 0,92 | [0,65; 0,99] |
| terminal_reconciled | 3/12 = 0,25 | [0,09; 0,53] |
| state_closed_after_report | 11/12 = 0,92 | [0,65; 0,99] |
| report_body_matches_final | 9/12 = 0,75 | [0,47; 0,91] |
| repeated_task_records | 12/12 = 1,00 | [0,76; 1,00] |
| side_effects | 12/12 = 1,00 | [0,76; 1,00] |

Os intervalos são largos o bastante para conter quase todas as diferenças que a
série tratou como efeitos.

## Q5 — `sizes` 0/6 é endpoint mal especificado

`sizes` **não é um endpoint**. É um dicionário de medição sem chave `pass`:

```json
{"records": 6, "file_bytes": 2977, "record_bytes": [267, 323, 350, 370, 623, 686],
 "median": 360.0, "checkpoint_bytes": 323}
```

Não entra na conjunção de `pass` (`harness.py:989`). O "0/6 em todos os braços"
foi artefato de uma tabela de análise que coagiu o dicionário a booleano; não
houve falha alguma. Nada foi alterado.

## O que a evidência sustenta testar, e o que ela exclui

**Sustenta:**

1. **Contadores do checkpoint, em `execute.md`.** Maior alvo isolado: `pass`
   4/24 → 9/24 no contrafactual, e cinco componentes de uma vez. Precisa ser
   medido num lote onde o defeito esteja presente — no lote `cfc` ele quase não
   apareceu, o que sozinho explica o resultado nulo de M.
2. **Conteúdo do PLAN sob Root exausta.** Segundo alvo: `pass` 4/24 → 6/24.
   Presente nos três braços, nunca tocado por candidata alguma. Mas a evidência
   observável aponta para o Execute pulando a segunda seção crítica, não para o
   `plan.md` devolvendo conteúdo inválido — então uma edição de `plan.md`
   precisa ser justificada por evidência nova, não por este diagnóstico.
3. **Instrumentar a invocação de Plan.** Sem registro durável de cada invocação
   e recibo, (b) e (c) permanecem indistinguíveis para sempre. Isto é trabalho
   de harness, não de skill, e não altera nenhum arquivo normativo.

**Exclui:**

- **Qualquer experimento com n=6 por braço.** O efeito mínimo detectável é
  p₁ ≥ 0,98. Repetir o formato é gastar 12 chamadas para obter uma decisão que
  já se sabe indistinguível de ruído.
- **Gates construídos sobre a lista de 14 componentes como se fossem
  independentes.** `reconciliation` e `terminal_reconciled` são o mesmo teste;
  cinco componentes disparam juntos a partir de duas linhas. Um gate pareado
  sobre eles conta a mesma evidência várias vezes.
- **A hipótese (a) de que o Plan nunca é consultado.** Eliminada: 24/24.

## Gate

```text
plan.md não é dono do defeito dominante: 4 runs só-PLAN contra 9 só-EXECUTE
→ maior alvo isolado = contadores do checkpoint em execute.md (4/24 -> 9/24)
→ segundo alvo = PLAN sob Root exausta (4/24 -> 6/24), causa ainda não isolada
→ n=6 por braço está formalmente excluído (MDE p1 >= 0,98)
→ próximo A/B exige >= 18 por braço para 0,25 -> 0,70; 29 para pass; 54 para 0,25 -> 0,50
→ baseline normativa intacta; nenhuma candidata proposta aqui
```

# Handoff ao agente master: K melhora task records, mas falha não-regressão

Data: 2026-09-01

Status: `task-record-validation-order-ab-v1` concluído; candidata K rejeitada;
baseline normativa restaurada e intacta; nenhum end-to-end ou teste de
compactação executado

## Veredito

Há três conclusões distintas:

1. `task-record-jit-schema-ab-v1` continua **formalmente inválido** para taxas
   de schema e `VALID`. Suas 12 runs não foram rescored nem reutilizadas como
   experimento válido.
2. A candidata J está **rejeitada operacionalmente** pelos dois hard-gates que
   não dependem do defeito lexical em `Unresolved:`: ordem do TRACK em J foi
   5/6 e PLAN future-only em J foi 5/6, contra 6/6 em A. Isso não prova que J
   causou as falhas; apenas impede sua adoção e outra repetição de 12 chamadas.
3. A candidata K passou todos os hard-gates de task record em 6/6, mas regrediu
   checkpoints Full de 2/6 em A para 1/6 e reconciliação de 2/6 para 1/6.
   Como ambos eram gates pareados pré-fixados de não-regressão, K está
   **rejeitada** e não deve ser instalada ou commitada.

O ganho de K é material: status, partial schema, blocked schema, ambos os
schemas, ordem, produto e ausência de dependência tentada passaram em 6/6.
Mesmo assim, o protocolo prioriza replan/reconciliação; melhorar o registro sem
preservar o fechamento não autoriza adoção.

## Estado Git e baseline normativa

Estado observado antes e depois do experimento:

```text
branch: feature/track-compact
HEAD:   02fd73900e68560cddaec79cb5d850cd4c2357b5

SKILL.md   SHA-256 5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1
plan.md    SHA-256 55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54
execute.md SHA-256 788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc
```

`SKILL.md`, `plan.md` e `execute.md` não foram alterados. K existiu somente
nos workspaces ignorados do experimento. A única mudança Git pendente é este
handoff; nenhum commit ou push foi feito nesta execução.

## Correção do scorer antes do Luna

O scorer novo mede PAR/SCH somente pela estrutura contida entre o heading da
task e seu `Gate:` inline.

Para `[partial]`, exige exatamente:

```text
Plan version não vazio
Covers não vazio
Root não vazio
Evidence com ao menos um bullet não vazio
Verification: unverified
Unresolved com ao menos um bullet não vazio
Gate: replan required
```

Não há busca por `required`, `requires`, `unavailable`, `cannot`, `observed`,
negação ou qualquer vocabulário do bullet. O endpoint de observabilidade usa
estado/lineage separadamente e também não interpreta o texto do bullet. PAR,
BLK e SCH não dependem de SEM.

### Mutation tests

Nove testes focados passaram antes da primeira chamada:

- passam PAR: `A browser reload must be observed...`, `No browser runtime ...
  is available...`, `...has not been observed...`, nominal sem verbo,
  `required`, `requires`, `unavailable`, `cannot` e até uma afirmação positiva;
  o endpoint mede estrutura, não verdade semântica;
- remover, renomear ou deslocar status, `Plan version`, `Covers`, `Root`,
  `Evidence`, `Verification`, `Unresolved` ou `Gate` reprova PAR;
- remover, renomear ou corromper os campos obrigatórios de blocked reprova BLK;
- `Blocker` apenas no checkpoint não completa retroativamente o task record;
- status correto com schema incompleto permanece diagnóstico separado;
- as 12 runs v2 foram avaliadas somente em cópias temporárias e o manifesto dos
  `score.json` originais permaneceu byte-idêntico.

Recomputação estrutural em cópias do corpus v2:

| Endpoint | O | S |
|---|---:|---:|
| Status | 6/6 | 6/6 |
| Partial schema | 5/6 | 6/6 |
| Blocked schema | 4/6 | 3/6 |
| Ambos schemas | 4/6 | 3/6 |
| Run válida | 3/6 | 1/6 |

Validação local anterior ao modelo:

```text
suíte completa: 207/207
scorer/mutation tests: 9/9
preservation gate: passou
quick_validate baseline A: passou
quick_validate candidata K materializada: passou
git diff --check: passou
```

Identidades congeladas:

```text
scorer      e7fb1a3a21aedc326a54843fee363bb613b0aedca76f1e54337e7ee107ae3a5a
tests       186d103e1be1a80be9d08eb25bbd393f81d61da9ec792cc75a69d2c5a0692985
runner      3789c6db234d496e4cb463f6ae5064309fe5a7c912791eab343ff47adde75c76
harness     6f4574c707feb4e58fc95d92d504d973479428ba118ea7a7fa21c82612e3df48
Oracle      8b143163e0a96f2e2bdeffc06bcf8cf24053ce0409d1558c0e93467cff2154f4
score_runs  5fe2ee2f8bda7fd31020f8dc422585e2102418150d92b0f89332c0b0f7bac117
prompt      f70f6da5720cfc18315f9a4294bd1d790646187c08e0599891e835e3b3c16b94
stdin       f374c3d23b67063b2b35073191988e33ecd6f2de1268657afcbcd4340e41cb85
fixture O   097c4ce82d2db113e21eeb0fdcbb88125a88212d2b367b13610fdf95b6642e09
fixture S   396d35347e1574504d4b675cf56e60ec5fc53fc40e74392d9822de30596db8d9
```

## Candidata K: somente posição

K move, sem reescrever palavra alguma, o bloco existente `Mandatory content by
status` + `Before appending, confirm:` + a frase transient para imediatamente
antes de `Append to spec-interview/<slug>/TRACK.md.`. O bloco não foi duplicado.
Exemplos, templates Full, retention lanes, tabela de transição, lineage e
terminal permanecem nos mesmos bytes.

Identidade de K:

```text
bytes:    38.714 — exatamente o mesmo tamanho de A
SHA-256:  e2d20445117ca1fbc8edc1f27c7d129970f15aabfcbc909f7ec65db0bb875b6c
Git blob: 7518b68f191cd7a44e3da83ecc337da2847ebe5f
```

Prova mecânica movement-only:

- o bloco existe exatamente uma vez em A e K;
- em K ele termina imediatamente antes da linha `Append to ...`;
- removendo esse bloco uma vez de A e K, os bytes restantes são idênticos;
- A e K têm o mesmo número de bytes.

### Diff exato de K

```diff
diff --git 1/skills/spec-to-done/references/execute.md 2/evaluation/track-compactness/sessions/task-manager-task-record-validation-order-ab-v1/codex-tvo-p01-r2/workspace/.agents/skills/spec-to-done/references/execute.md
index 23d05a7..7518b68 100644
--- 1/skills/spec-to-done/references/execute.md
+++ 2/evaluation/track-compactness/sessions/task-manager-task-record-validation-order-ab-v1/codex-tvo-p01-r2/workspace/.agents/skills/spec-to-done/references/execute.md
@@ -273,0 +274,24 @@ stop, or apparent terminal condition may skip the remaining sequence.
+#### Mandatory content by status
+
+| Status | Mandatory TRACK content |
+|---|---|
+| `done` | State delta, Evidence, Verification, Gate |
+| `no_op` | Evidence showing the condition already held, Verification, Gate |
+| `partial` | Evidence, Verification, Unresolved, Gate; State delta when something changed |
+| `blocked` | Blocker, Blocked because, Resolution condition, Evidence, Verification, Gate; User action only when applicable |
+| `failed` | Blocker, Failure, Resolution condition, Evidence, Verification, Gate; State delta when something changed |
+
+A section with nothing to say is left out only when this table does not require it.
+
+Before appending, confirm:
+
+- if status is `blocked`, do not append unless `Blocker`, `Blocked because`, `Resolution condition`, `Evidence`, `Verification`, and `Gate: replan required` are present;
+- if status is `failed`, do not append unless `Blocker`, `Failure`, `Resolution condition`, `Evidence`, `Verification`, and `Gate: replan required` are present;
+- if status is `partial`, do not append unless `Unresolved` is present;
+- every identifier, lineage field, provenance label, non-repeatable effect, repeat prohibition, and the effective `Gate` are present and literal;
+- every `Evidence` line reads check → result, in the evidence class it came with;
+- no reasoning, raw log, restated task or contract text, or repeated prose remains;
+- if any required item is absent, rewrite the record before appending.
+
+This check is transient. Do not write it, its result, or a verdict to TRACK.
+
@@ -362,24 +385,0 @@ Every fact you are about to write belongs to exactly one of these three lanes. T
-#### Mandatory content by status
-
-| Status | Mandatory TRACK content |
-|---|---|
-| `done` | State delta, Evidence, Verification, Gate |
-| `no_op` | Evidence showing the condition already held, Verification, Gate |
-| `partial` | Evidence, Verification, Unresolved, Gate; State delta when something changed |
-| `blocked` | Blocker, Blocked because, Resolution condition, Evidence, Verification, Gate; User action only when applicable |
-| `failed` | Blocker, Failure, Resolution condition, Evidence, Verification, Gate; State delta when something changed |
-
-A section with nothing to say is left out only when this table does not require it.
-
-Before appending, confirm:
-
-- if status is `blocked`, do not append unless `Blocker`, `Blocked because`, `Resolution condition`, `Evidence`, `Verification`, and `Gate: replan required` are present;
-- if status is `failed`, do not append unless `Blocker`, `Failure`, `Resolution condition`, `Evidence`, `Verification`, and `Gate: replan required` are present;
-- if status is `partial`, do not append unless `Unresolved` is present;
-- every identifier, lineage field, provenance label, non-repeatable effect, repeat prohibition, and the effective `Gate` are present and literal;
-- every `Evidence` line reads check → result, in the evidence class it came with;
-- no reasoning, raw log, restated task or contract text, or repeated prose remains;
-- if any required item is absent, rewrite the record before appending.
-
-This check is transient. Do not write it, its result, or a verdict to TRACK.
-
```

O diff exibido usa `--unified=0`: 3.778 bytes, SHA-256
`018061e2c3a4e659d33bd42f17e0a547b45f97abbe09042d11d7ff963f4823ae`.
O artefato bruto preservado, com três linhas de contexto, tem 4.389 bytes e
SHA-256 `e3ad7ca88db2d51a7aa9f9711eecf925ee4b37874197a72201ff7a6a7bace03e`.

## Preflight congelado

Os 12 workspaces começaram limpos, balanceados em A/O, A/S, K/O e K/S
(3 cada), com os fixtures v2 byte-idênticos, produto 9/9, navegador realmente
ausente e escrita em `release/` realmente negada ao mesmo runtime das runs.
Nenhum side effect do preflight permaneceu.

Trees iniciais:

```text
O/A d70ad55508aa71d7b0337cecc724e0de74b3da25
O/K f4a74084fc76294ea4d738dccaa4bf577a72a710
S/A 9d6c81319e902a280ebcf59df4f4b2d56171dd78
S/K 724db9102bd416086b8d01326ba954d4c0628204
```

O preflight também reproduziu os hard-gates de J diretamente dos workspaces,
sem usar PAR/SCH/VALID e sem reescrever scores:

```text
A: ordem 4/6; PLAN future-only 6/6
J: ordem 5/6; PLAN future-only 5/6
```

## Execução cega

Foram 12 chamadas sequenciais e isoladas, exclusivamente com
`gpt-5.6-luna`, reasoning `low`. Todas terminaram com exit 0, sem timeout.
Nenhum resultado foi inspecionado antes da 12ª conclusão; scores foram gravados
como `blind` e a associação A/K só foi usada na auditoria posterior.

Comando interno comum:

```text
codex exec --ephemeral --skip-git-repo-check --ignore-user-config
  --sandbox workspace-write --json -C <workspace>
  -m gpt-5.6-luna -c model_reasoning_effort=low -
```

| Ordem | Sessão neutra | Fixture/variante | UTC início–fim | Segundos |
|---:|---|---|---|---:|
| 1 | `tvo-p01-r1` | O/A | 21:18:22–21:20:07 | 105,4 |
| 2 | `tvo-p01-r2` | O/K | 21:20:07–21:21:47 | 99,5 |
| 3 | `tvo-p02-r1` | S/K | 21:21:47–21:24:10 | 143,0 |
| 4 | `tvo-p02-r2` | S/A | 21:24:10–21:26:39 | 148,9 |
| 5 | `tvo-p03-r1` | O/K | 21:26:39–21:29:15 | 155,7 |
| 6 | `tvo-p03-r2` | O/A | 21:29:15–21:30:53 | 97,9 |
| 7 | `tvo-p04-r1` | S/A | 21:30:53–21:32:47 | 114,4 |
| 8 | `tvo-p04-r2` | S/K | 21:32:47–21:34:56 | 128,2 |
| 9 | `tvo-p05-r1` | O/A | 21:34:56–21:36:11 | 75,1 |
| 10 | `tvo-p05-r2` | O/K | 21:36:11–21:38:19 | 128,3 |
| 11 | `tvo-p06-r1` | S/K | 21:38:19–21:40:04 | 104,5 |
| 12 | `tvo-p06-r2` | S/A | 21:40:04–21:42:42 | 158,1 |

## Resultado agregado

| Critério | A baseline | K posição | Gate de K |
|---|---:|---:|---|
| Status correto | 6/6 | 6/6 | passou |
| Partial schema completo | 3/6 | 6/6 | passou |
| Blocked schema completo | 2/6 | 6/6 | passou |
| Ambos os schemas completos | 2/6 | 6/6 | passou |
| Ordem do TRACK | 6/6 | 6/6 | passou |
| Produto/segurança 9/9 | 6/6 | 6/6 | passou |
| Sem dependência para ID tentado | 5/6 | 6/6 | passou |
| Checkpoints Full | 2/6 | 1/6 | **regrediu; rejeita K** |
| PLAN future-only | 5/6 | 5/6 | não inferior |
| Gates fechados | 3/6 | 6/6 | não inferior |
| Reconciliação | 2/6 | 1/6 | **regrediu; rejeita K** |
| Run integralmente válida | 1/6 | 1/6 | não inferior |
| REPORT persistido | 6/6 | 6/6 | preservado |
| state fechado depois do REPORT | 5/6 | 6/6 | melhorou |
| Final byte-idêntico ao REPORT | 5/6 | 6/6 | melhorou |

K produziu task records completos em 6/6, mas cinco de suas seis runs omitiram
`total_lineage_attempts` e `total_lineage_limit` nos checkpoints. Em uma delas,
T6 também permaneceu no PLAN. Apenas `tvo-p04-r2` foi integralmente válida.

### Resultado por run

| Sessão | Resultado material |
|---|---|
| A/O `tvo-p01-r1` | válida integralmente |
| K/O `tvo-p01-r2` | schemas/ordem/produto/dependências/gates corretos; checkpoints sem dois contadores Full; T6 permaneceu no PLAN; reconciliação falhou |
| K/S `tvo-p02-r1` | schemas e PLAN corretos; checkpoints sem dois contadores Full; `User action` protegido ausente; reconciliação falhou |
| A/S `tvo-p02-r2` | `Blocker` ausente em T5; nenhum checkpoint; gates abertos; state fechou cedo |
| K/O `tvo-p03-r1` | schemas corretos; checkpoints sem dois contadores Full; reconciliação falhou |
| A/O `tvo-p03-r2` | partial e blocked com Gates errados; blocker/causa/resolução ausentes; nenhum checkpoint; final divergiu do REPORT |
| A/S `tvo-p04-r1` | schema/checkpoints/PLAN/reconciliação corretos; `User action` protegido ausente |
| K/S `tvo-p04-r2` | válida integralmente |
| A/O `tvo-p05-r1` | partial/blocked incompletos; nenhum checkpoint; T6 permaneceu no PLAN; gates abertos |
| K/O `tvo-p05-r2` | schemas corretos; checkpoints sem dois contadores Full; reconciliação falhou |
| K/S `tvo-p06-r1` | schemas corretos; checkpoints sem dois contadores Full; reconciliação falhou |
| A/S `tvo-p06-r2` | partial com Verification errada; blocker ausente; nenhum checkpoint; dependências tentadas e gates abertos |

## Integridade da evidência local

O workbench é ignorado pelo Git, por isso os digests abaixo permitem identificar
o pacote bruto que originou este handoff:

```text
experiment JSON  2a5ff19294e4288160e17841ecc3b89ff7e33776b271ff1f481b6cfba1406fd0
preflight        d20f503d85a6909f33e5ca8d01a0da53a187529f8208a5a7d912736664f13136
run log          b12c3f6ec1dac3310f8619f3d58c79b8ff739a10ec5ea99b7cd779e4ec7734c3
audit            ec32ec9b780b7a03545916f0e941584292998b869e300797266ae265f7ebdcf7
candidate diff   e3ad7ca88db2d51a7aa9f9711eecf925ee4b37874197a72201ff7a6a7bace03e
J prior audit    49c1c54a2187f6543053d570ffeebd4bbc6486fd6fe458e22eef82bb456ab2a3
J prior run log  24f2315965d81dab49fac945e3dac03ee80d617302a4db48c56e9d0bcd714f82
```

Os prompts, stdout, stderr, metas, scores e workspaces das 12 runs permanecem
preservados localmente em:

```text
evaluation/track-compactness/sessions/
  task-manager-task-record-validation-order-ab-v1/
```

Nenhum artefato bruto ou scorer foi modificado depois da primeira chamada.
Nenhum score antigo de v2 ou J foi reescrito.

## Decisão e gate

```text
J: formalmente inválida para schema/VALID
→ rejeitada operacionalmente por ordem 5/6 e PLAN future-only 5/6
→ não repetir J

K: hard-gates de task record 6/6
→ checkpoints 1/6 < A 2/6
→ reconciliação 1/6 < A 2/6
→ rejeitar K e manter baseline execute 788cfa214aff...
```

Não houve end-to-end, medição de compactação, ampliação de amostra, alteração
normativa ou implementação da hipótese seguinte. Qualquer próximo experimento
exige uma nova hipótese isolada do agente master; não acumular nova prosa ou
reaplicar J/K sem decisão explícita.

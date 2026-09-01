# Handoff ao master: gramática literal de checkpoints regrediu no A/B cego

Data: 2026-08-31

Status: ablação somente em `execute.md` rejeitada; closure-only obteve **1/6**
checkpoints Full válidos e **0/6** reconciliações integrais, contra **5/6** e
**4/6** da baseline intercalada. A baseline normativa foi restaurada; end-to-end
e compactação não foram executados.

## Estado disponível no Git

A baseline testada existe no commit
`cbd52284dbb7f0ae3dba1db9d2d1b622d8d428f4`:

```text
SKILL.md   blob d36a349c9ba0 / SHA-256 5f0440b460ac
plan.md    blob 9fa6f7580399 / SHA-256 55e779256622
execute.md blob 23d05a732409 / SHA-256 788cfa214aff
```

O hash acima identifica a baseline efetivamente submetida ao modelo; não tenta
antecipar o commit que levará este handoff ao Git.

A candidata B não recebeu commit. Ela manteve SKILL e Plan e usou somente:

```text
execute.md blob 0d321fee7076 / SHA-256 614d5320c6e5
```

Após o experimento, `execute.md` voltou ao blob `23d05a732409`. Portanto, a
candidata reprovada não está ativa; a única mudança versionável pendente é este
handoff.

O estado restaurado passa **184/184 testes** (incluindo os dois checks de
auditabilidade cega), preservation, `quick_validate` de runtime/Core/Bounded/Full,
sincronização runtime/Full e dos planners, e `git diff --check`.

## Ablação testada

A candidata B alterou apenas a representação dos checkpoints em `execute.md`:

- moveu os dois templates Full existentes, sem mudar seus campos, para logo
  depois da tabela de transição;
- retirou do exemplo os checkpoints concretos de T5/T6, preservando os task
  records;
- concentrou a escrita em uma instrução curta:
  `selecionar linha → instanciar template → substituir placeholders → validar →
  anexar uma vez`;
- maintenance continuou sem checkpoint.

Não mudaram Plan, root, task records, blocker, `partial`, exhaustion por `Root`,
lineage, owners, gates ou terminal.

Antes do modelo, a candidata passou **191/191 testes**, preservation,
`quick_validate` de runtime/Core/Bounded/Full, sincronização das candidatas e
`git diff --check`.

## Experimento closure-only A/B cego

Foram 12 chamadas sequenciais e isoladas, somente com `gpt-5.6-luna`, reasoning
`low`. A ordem foi congelada antes da primeira chamada:

```text
par 1 A→B | par 2 B→A | par 3 A→B
par 4 B→A | par 5 B→A | par 6 A→B
```

Os workspaces tinham rótulos neutros; nenhum Oracle, score ou run anterior foi
exposto ao modelo. Todas as 12 chamadas terminaram sem timeout ou falha de
infraestrutura. Os 12 scores foram gerados com `arm: blind`; somente depois a
associação A/B foi aberta.

Evidência bruta local, ignorada pelo Git:

```text
evaluation/track-compactness/sessions/task-manager-closure-only/codex-cpg-*
evaluation/track-compactness/checkpoint-grammar-ab-v1.json
```

## Resultado

| Critério | A baseline | B gramática | Leitura |
|---|---:|---:|---|
| Ambos os checkpoints Full canônicos e completos | 5/6 | 1/6 | regressão causal principal |
| Reconciliação integral | 4/6 | 0/6 | candidata reprovada |
| Produto/segurança 9/9 | 6/6 | 6/6 | preservado |
| T5 `partial` + `Verification: unverified` literal | 6/6 | 6/6 | preservado no task record |
| Nenhuma task tentada permaneceu no PLAN | 4/6 | 3/6 | regressão |
| Nenhuma dependência para task tentada | 6/6 | 6/6 | preservado |
| Ordem permitida | 5/6 | 6/6 | melhora isolada, insuficiente |
| `Blocker` canônico dentro do task record de T6 | 5/6 | 3/6 | regressão protegida |
| `Blocked because` + `Resolution condition` | 6/6 | 6/6 | preservado |
| Sem despacho same-Root após exhaustion | 6/6 | 6/6 | preservado |
| `state.md` fechado depois do REPORT | 5/6 | 5/6 | igual |
| Final byte-idêntico ao REPORT | 6/6 | 5/6 | regressão terminal |

Nos seis pares, quatro favoreceram A e nenhum favoreceu B tanto no endpoint de
checkpoint quanto na reconciliação integral. O `n` não sustenta uma conclusão
estatística forte, mas o gate experimental era operacional: B precisava de
6/6, e 1/6 exige rejeição independentemente de significância.

### Resultado exato por sessão

| Braço | Sessão | Resultado material |
|---|---|---|
| A | `cpg-p01-s1` | válida |
| A | `cpg-p02-s2` | válida |
| A | `cpg-p03-s1` | checkpoints corretos; T5 permaneceu no PLAN e state não fechou após REPORT |
| A | `cpg-p04-s2` | nenhum checkpoint canônico; T5 permaneceu no PLAN; blocker de T6 ausente; ordem inválida |
| A | `cpg-p05-s2` | válida |
| A | `cpg-p06-s1` | válida |
| B | `cpg-p01-s2` | nenhum checkpoint canônico; blocker de T6 ausente; gates abertos; PLAN v3 sem checkpoint material |
| B | `cpg-p02-s1` | usou `Checkpoint:` livre; T6 permaneceu no PLAN; blocker de T6 ausente |
| B | `cpg-p03-s2` | usou `Checkpoint:` livre; gates abertos; final divergiu do REPORT |
| B | `cpg-p04-s1` | usou `Checkpoint:` livre; T6 permaneceu no PLAN; blocker de T6 ausente |
| B | `cpg-p05-s1` | ambos os checkpoints corretos; T6 permaneceu no PLAN |
| B | `cpg-p06-s2` | usou heading de checkpoint não canônico; gates abertos; state não fechou após REPORT |

## Decisão

A simples aproximação dos templates da tabela tornou o exemplo positivo menos
saliente e piorou exatamente a serialização que pretendia estabilizar. A
ablação `614d5320c6e5` deve permanecer somente como evidência negativa e não ser
reaplicada.

A baseline `788cfa214aff` continua normativa. Não há autorização experimental
para outro patch, end-to-end ou compactação a partir desta rodada. Antes de uma
nova mudança, o master deve decidir outra hipótese isolada que preserve o
exemplo concreto e não acumule instruções sobre o mesmo fechamento.

## Gate

```text
B checkpoints 1/6 e reconciliação 0/6
→ rejeitar B e manter baseline 788cfa214aff
→ não executar end-to-end
→ não medir compactação
→ não executar amostra maior
```

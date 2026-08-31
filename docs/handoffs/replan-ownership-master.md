# Handoff ao master: normalização em duas fases regrediu no Luna Low

Data: 2026-08-31

Status: implementação local passa todos os gates determinísticos; closure-only
Luna Low obteve **0/6 válidas**; end-to-end e compactação não foram executados

## Estado que precisa ir ao Git

As únicas mudanças normativas estão em:

- `skills/spec-to-done/references/plan.md`
- `skills/spec-to-done/references/execute.md`

`skills/spec-to-done/SKILL.md` permanece inalterado. Hashes da candidata
submetida ao modelo:

```text
SKILL.md   5f0440b460ac
plan.md    48692d587983
execute.md 8ab96ff0cd70
```

O workbench e as sessões são locais e ignorados pelo Git. Toda a evidência
necessária para decisão está abaixo.

## Correção implementada

### `plan.md`

- Um único algoritmo faz snapshot do trigger e grafo antigo, normalização
  estrutural fase A, seleção exclusiva da transição, resolução de dependências
  fase B, quality gate e só então emite um receipt.
- Fase A é future-only e idempotente, sem decidir silenciosamente o destino de
  dependências quebradas.
- Fase B possui os quatro destinos autorizados: satisfação exata, repoint para
  continuation same-Root, retirada do dependente após exhaustion insatisfeito,
  ou preservação explícita do pré-requisito estreito já verificado.
- `partial` não entra no replan genérico: capability disponível exige uma
  continuation concreta; indisponível exige exhaustion do mesmo `Root`.
- `done/no_op` pode produzir PLAN vazio em replan material somente quando toda
  a cobertura restante já está satisfeita.

### `execute.md`

- A seção `POST-TASK CRITICAL SECTION`, tabela de transição, dois templates Full
  e predicado compacto foram movidos para o início operacional.
- Mid-task agora exige `parar → verificar → task record → Plan`; uma dependente
  não recebe gate quando a pendência real é o gate aberto da prerequisite.
- Fluxos posteriores apenas referenciam `POST-TASK CLOSE(trigger)`; ownership,
  Gate vocabulary, exhaustion por `Root`, lineage, blocker, TRACK append-only e
  terminal owners não mudaram.

Foram adicionados oito testes comportamentais: idempotência, snapshot/posição,
quatro destinos de dependência, `partial` disponível e indisponível, material
replan vazio legítimo, receipt prematuro e retomada depois de PLAN antes do
checkpoint.

## Gates locais

- suíte completa: **182/182**;
- preservation gate: passou;
- `quick_validate`: runtime, Core, Bounded e Full passaram;
- runtime/Full e os `plan.md` das três candidatas estão sincronizados;
- `git diff --check`: passou.

## Closure-only Luna Low

Amostra válida: runs **7, 8, 13, 14, 15 e 16**, todas sequenciais, isoladas,
`gpt-5.6-luna`, reasoning `low`, sem timeout e com os hashes acima. As tentativas
9–12 não chegaram ao modelo: o runner em loop não teve escrita no estado local
do Codex e saiu em 0,0 s. Seus artefatos de infraestrutura foram preservados,
mas foram excluídos da amostra e substituídos pelas runs 13–16.

Resultado: **0/6 semanticamente válidas**, contra **4/6** da candidata anterior.
Os seis produtos continuaram 9/9 e todos persistiram REPORT, fecharam state
depois dele e emitiram o corpo final byte-idêntico. A regressão é integralmente
do protocolo de coordenação.

| Critério | Anterior 1–6 | Atual 7,8,13–16 | Leitura |
|---|---:|---:|---|
| Run completamente válida | 4/6 | 0/6 | regressão principal |
| Produto/segurança 9/9 | 6/6 | 6/6 | preservado |
| T5 com `Verification: unverified` literal | 6/6 | 3/6 | regressão de campo canônico |
| T5 fechado por exhaustion do próprio `Root` | 4/6 | 2/6 | regressão; três `replan done` indevidos |
| Nenhuma task tentada permaneceu no PLAN | 6/6 | 3/6 | regressão da normalização |
| Nenhuma dependência para task tentada | 6/6 | 6/6 | preservado |
| Ordem permitida | 6/6 | 5/6 | uma run reescreveu a ordem do TRACK |
| `Blocker` canônico no task record de T6 | 5/6 | 1/6 | maior regressão protegida |
| `Blocked because` + `Resolution condition` | 6/6 | 6/6 | preservado |
| T6 com fechamento nomeado | 5/6 | 6/6 | melhorou |
| Dois templates Full presentes com contagens reais | 4/6 | 5/6 | formato melhorou; semântica não |
| Sem despacho same-Root após exhaustion | 6/6 | 6/6 | preservado |
| `state.md` fechado depois de REPORT | 6/6 | 6/6 | preservado |
| Final byte-idêntico ao REPORT | 6/6 | 6/6 | preservado |
| Reconciliação integral | 4/6 | 0/6 | gate continua fechado |

### Resultado exato por run

| Run | Funcionou | Reprovou |
|---|---|---|
| 7 | T5 canônica; T6 com blocker/campos; T6/T7 fechadas; ordem e terminal corretos | T5 virou `replan done`, criou T7 apesar da capability indisponível e registrou contadores futuros; T6 tentada permaneceu no PLAN sob `Root: T5` |
| 8 | T5 exhausted corretamente; T6 fechada; ordem e terminal corretos | `Blocker` ausente no task record de T6; T6 tentada permaneceu no PLAN sob Root exhausted |
| 13 | PLAN vazio; dois checkpoints Full; terminal correto | T5 usou `Verification: partial; ...`, `Unresolved` não preservou a observação exata; `Blocker` de T6 ausente; T5/T6 foram inseridas antes de T2–T4 no TRACK |
| 14 | PLAN vazio; dois checkpoints Full; ordem e terminal corretos | reescreveu os gates inline de T5/T6 como `replan exhausted`; T5 usou Verification não canônica; `Blocker` de T6 ausente |
| 15 | T5 inicialmente canônica; dois checkpoints; ordem e terminal corretos | T5 recebeu `replan done` sem continuation e permaneceu tentada no PLAN; residual desapareceu; `Blocker` de T6 ausente |
| 16 | PLAN vazio; T6 exhausted; ordem e terminal corretos | T5 recebeu `replan done` sem continuation e perdeu o residual; Verification não canônica; `Blocker` de T6 ausente |

## Diagnóstico para decisão

A candidata demonstra que a normalização em duas fases é determinística no
simulador, mas não reduziu o espaço de interpretação do Luna Low. O erro mudou
de “checkpoint ausente” para três perdas mais graves e recorrentes:

1. `partial` ainda cai em `replan done` sem continuation em 3/6;
2. o `Blocker` obrigatório fica apenas no checkpoint e some do task record em
   5/6;
3. a normalização future-only, que era 6/6, caiu para 3/6.

Há também uma contradição local de redação a revisar: a seção diz
“Immediately after appending one task record” e o passo 1 do algoritmo manda
“Validate and append exactly one canonical task record”. Isso apresenta o
mesmo append como pré-condição e como primeiro passo. A run 13, que reordenou
TRACK, e a run 14, que substituiu gates inline por valores de checkpoint, são
compatíveis com essa fronteira pouco nítida. É uma hipótese causal, não uma
conclusão provada.

O próximo desenho precisa preservar a melhora estrutural dos templates e do
terminal, mas tornar inseparáveis três fatos no ponto de escrita: task record
imutável com gate inline, campos obrigatórios por status — especialmente
`Blocker` — e exatamente uma transição posterior. Não adicionar mais prosa ao
planner antes de decidir como eliminar essa duplicidade operacional.

## Gate

Esta candidata está reprovada. O end-to-end não foi executado porque o gate
closure-only 6/6 falhou; compactação e amostra maior continuam proibidas.

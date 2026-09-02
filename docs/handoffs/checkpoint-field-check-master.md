# Handoff ao agente master: M rejeitada e o braço A não replicou

Data: 2026-09-01

Status: `checkpoint-field-check-ab-v1` concluído; candidata M rejeitada pelos
hard-gates pré-registrados; baseline normativa intacta; nenhum end-to-end ou
medição de compactação executado. O achado dominante não é sobre M: o braço A,
idêntico byte a byte ao de `task-record-validation-order-ab-v1`, produziu
números materialmente diferentes na mesma configuração.

## Veredito

Há quatro conclusões distintas.

1. A candidata M está **rejeitada**. Falhou os três endpoints primários e
   quatro dos oito hard-gates de não-regressão: ordem 5/6, PLAN future-only
   3/6, `state.md` fechado 4/6 e checkpoints emitidos 4/6.
2. A hipótese de localidade **não foi testada**, nem confirmada nem refutada.
   O resultado pré-registrado exigia M com ambos os endpoints ≥ 5/6; M obteve
   schema 4/6 e checkpoints 3/6. Não é a alternativa do orçamento de
   conformidade — nenhum endpoint foi trocado por outro; nenhum dos dois subiu.
3. O braço A **não replicou**. Contra tvo-v1, com o mesmo `execute.md`, os
   mesmos fixtures, o mesmo modelo, o mesmo esforço e o mesmo stdin congelado,
   partial schema foi 3/6 → 6/6, checkpoints Full 2/6 → 5/6, ambos os schemas
   2/6 → 4/6, gates fechados 3/6 → 5/6 e PLAN future-only 5/6 → 3/6. O ruído de
   lote em n=6 é da ordem de ±3/6, maior que qualquer efeito já reivindicado
   nesta linha de experimentos.
4. A identidade `reconciliação == checkpoint_serialization`, observada em todas
   as 12 runs de tvo-v1, **é falsa neste lote**: A obteve checkpoints 5/6 com
   reconciliação 1/6. Era propriedade daquele corpus, não estrutura do
   protocolo.

A consequência prática é que a decisão sobre M é secundária. Enquanto o mesmo
braço oscila ±3/6 entre lotes, nenhuma comparação A/B de 6+6 runs sustenta
adoção, rejeição causal ou otimização — inclusive as rejeições anteriores de J
e K, ambas decididas por diferenças de uma única run.

## Fatos estabelecidos que motivaram M

Medidos no corpus congelado de tvo-v1, em cópias temporárias, com o scorer
deste experimento:

```text
checkpoints emitidos            A 2/6   K 6/6
checkpoints completos (5 campos) A 2/6   K 1/6
```

As quatro falhas de A não emitiram checkpoint algum: a seção crítica pós-tarefa
inteira foi pulada. As cinco falhas de K emitiram o template de exaustão byte a
byte e pararam depois de `continuation_limit: 2`, omitindo
`total_lineage_attempts` e `total_lineage_limit`. Em tvo-v1, K converteu uma
omissão catastrófica em uma truncagem de duas linhas, e o endpoint binário
pontuou essa mudança como regressão.

M foi construída para corrigir exatamente essa truncagem, preservando o ganho
de K.

## Correção de desenho pré-registrada

Antes da primeira chamada foi fixado que endpoints binários com taxa ≤ 2/6 no
braço de referência não servem como hard-gate isolado de não-regressão em n=6 —
foi isso que rejeitou K por uma run de diferença. Neste experimento eles são
diagnósticos direcionais; os hard-gates ficaram restritos a endpoints com taxa
base alta o bastante para n=6 ter poder. Nada além disso foi relaxado.

O resultado do braço A mostra que a correção foi insuficiente. Endpoints com
taxa base **alta** também oscilaram: ordem 6/6 → 5/6 e PLAN future-only 5/6 →
3/6 no mesmo braço A, entre lotes.

## Estado Git e baseline normativa

Observado antes e depois do experimento:

```text
branch: feature/track-compact
HEAD:   c250fc1

SKILL.md   SHA-256 5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1
plan.md    SHA-256 55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54
execute.md SHA-256 788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc
```

Os três arquivos normativos estão byte-idênticos. M existiu somente nos
workspaces ignorados do experimento. Nenhum commit ou push foi feito.

## Scorer

O scorer deste experimento adiciona **apenas** um bloco diagnóstico,
`checkpoint_field_diagnostics`, sobre o scorer congelado de tvo-v1. Ele separa
"o checkpoint foi emitido" de "o checkpoint emitido carrega os cinco campos de
tentativa", registra os campos ausentes por task e conta ocorrências dos cinco
campos fora de checkpoints canônicos. Nenhuma definição de endpoint mudou e o
bloco nunca participa de `pass`.

Validação anterior à primeira chamada ao modelo:

```text
suíte completa:                 222/222
testes novos do scorer:         15/15
preservation gate:              passou
regressão sobre o corpus tvo-v1: números publicados de A e K reproduzidos exatamente
manifesto dos score.json de tvo-v1: byte-idêntico antes e depois
```

Os mutation tests cobrem: cinco campos presentes; um contador ausente; a
truncagem observada em K (os dois `total_lineage_*`); zero checkpoints;
contadores presentes só no task record; dois checkpoints para a mesma task;
fixture trocado. Três testes adicionais provam que M é K mais exatamente uma
inserção, e um prova que as cópias do corpus antigo nunca reescrevem os
originais.

Identidades congeladas:

```text
scorer                  435595ecc483c5768c23b8a6feeba891b9d366d3bf6ca705c4da63c44cc8f6de
validation_order_scorer e7fb1a3a21aedc326a54843fee363bb613b0aedca76f1e54337e7ee107ae3a5a
jit_scorer              d6ebe38ef0c6c0623e610bbdd717475cddbf83689c6cc18a32079dd1a66db3ff
tests                   2443fe3e7741e9d4027417be7938bbba496c6df367cd16cc0b56a1e5abb2611e
runner                  b9478ece07c5b1221189b81d187d51c599126d6e90df53aae25d2b60bf79a847
tvo_runner              3789c6db234d496e4cb463f6ae5064309fe5a7c912791eab343ff47adde75c76
harness                 6f4574c707feb4e58fc95d92d504d973479428ba118ea7a7fa21c82612e3df48
Oracle                  8b143163e0a96f2e2bdeffc06bcf8cf24053ce0409d1558c0e93467cff2154f4
score_runs              5fe2ee2f8bda7fd31020f8dc422585e2102418150d92b0f89332c0b0f7bac117
prompt                  f70f6da5720cfc18315f9a4294bd1d790646187c08e0599891e835e3b3c16b94
stdin                   f374c3d23b67063b2b35073191988e33ecd6f2de1268657afcbcd4340e41cb85
fixture O               097c4ce82d2db113e21eeb0fdcbb88125a88212d2b367b13610fdf95b6642e09
fixture S               396d35347e1574504d4b675cf56e60ec5fc53fc40e74392d9822de30596db8d9
```

## Candidata M: K mais uma verificação local ao checkpoint

M é a baseline A com exatamente duas edições, aplicadas mecanicamente:

1. o movimento de K, reconstruído pela própria lógica de tvo-v1 e verificado
   contra a identidade congelada de K antes de qualquer outra alteração;
2. uma verificação de campos imediatamente após os dois templates Full, antes do
   parágrafo que começa com `For` e nomeia `PLAN maintenance complete`.

Identidades:

```text
K intermediário  38.714 bytes / SHA-256 e2d20445117ca1fbc8edc1f27c7d129970f15aabfcbc909f7ec65db0bb875b6c
                 Git blob 7518b68f191cd7a44e3da83ecc337da2847ebe5f
M                39.053 bytes / SHA-256 f112efbad9af8351f33bcc4930cd04bb906c8858487c37c6f71865c90cf2262a
                 Git blob 76cd1beee8131c44324c743f8021c2342ce3b773
```

M acrescenta 339 bytes a K. Prova mecânica: o bloco novo aparece exatamente uma
vez; removendo-o uma vez de M, os bytes restantes são iguais a K; o bloco
termina imediatamente antes da linha-âncora; e a contagem de cada template,
exemplo, retention lane e tabela de transição é idêntica à da baseline.

### Diff exato de M sobre A

Além das duas hunks já publicadas em tvo-v1 para o movimento de K, M acrescenta
somente:

```diff
@@ -481,0 +482,7 @@ total_lineage_limit: 3
+Before appending, confirm the checkpoint carries all five attempt fields
+literally: `root_attempts`, `continuation_attempts`, `continuation_limit`,
+`total_lineage_attempts`, and `total_lineage_limit`. If any is absent, rewrite
+the checkpoint before appending.
+
+This check is transient. Do not write it, its result, or a verdict to TRACK.
+
```

O diff `--unified=0` tem 3.929 bytes, SHA-256
`292246ea1ae5a07422b0723ea4cd6d5793e280235e04cc398dd015b5b471d4ea`.
O artefato com três linhas de contexto tem 5.030 bytes, SHA-256
`28f2798a120c9cb01913a1e90aaa414199caf4ea69e0a73bfbabd407011ee77c`.

## Preflight congelado

Os 12 workspaces começaram limpos, balanceados em A/O, A/S, M/O e M/S (3 cada),
com os fixtures v2 byte-idênticos, produto 9/9, navegador realmente ausente e
escrita em `release/` realmente negada ao mesmo runtime das runs. Nenhum side
effect do preflight permaneceu. Todas as doze checagens pré-modelo passaram.

Trees iniciais:

```text
A/O d70ad55508aa71d7b0337cecc724e0de74b3da25
A/S 9d6c81319e902a280ebcf59df4f4b2d56171dd78
M/O 0fdc5a07427888bac09bfc838e69c4e65065659b
M/S 8b8903e8f2a0ff619bdcf9a5e1a7f6d5cc8b7149
```

Os trees de A/O e A/S são idênticos aos de tvo-v1, o que torna a comparação
entre lotes legítima para o braço A.

## Execução cega

Foram 12 chamadas sequenciais e isoladas, exclusivamente com `gpt-5.6-luna`,
reasoning `low`, no mesmo comando sandboxed de tvo-v1. Todas terminaram com
exit 0, sem timeout. Nenhum resultado foi inspecionado antes da 12ª conclusão;
os scores foram gravados como `blind` e a associação A/M só foi usada na
auditoria posterior.

| Ordem | Sessão neutra | Fixture/variante | UTC início–fim | Segundos |
|---:|---|---|---|---:|
| 1 | `cfc-p01-r1` | A/O | 22:13:25–22:15:04 | 98,9 |
| 2 | `cfc-p01-r2` | M/O | 22:15:04–22:16:30 | 86,5 |
| 3 | `cfc-p02-r1` | M/S | 22:16:30–22:18:22 | 111,9 |
| 4 | `cfc-p02-r2` | A/S | 22:18:22–22:20:07 | 104,7 |
| 5 | `cfc-p03-r1` | M/O | 22:20:07–22:21:45 | 97,9 |
| 6 | `cfc-p03-r2` | A/O | 22:21:45–22:23:28 | 103,1 |
| 7 | `cfc-p04-r1` | A/S | 22:23:28–22:25:40 | 132,3 |
| 8 | `cfc-p04-r2` | M/S | 22:25:40–22:27:18 | 97,8 |
| 9 | `cfc-p05-r1` | A/O | 22:27:18–22:28:50 | 92,7 |
| 10 | `cfc-p05-r2` | M/O | 22:28:50–22:30:34 | 103,5 |
| 11 | `cfc-p06-r1` | M/S | 22:30:34–22:32:10 | 96,4 |
| 12 | `cfc-p06-r2` | A/S | 22:32:10–22:34:58 | 167,6 |

## Resultado agregado

| Critério | A baseline | M | Gate de M |
|---|---:|---:|---|
| Status correto | 6/6 | 6/6 | — |
| Partial schema completo | 6/6 | 5/6 | — |
| Blocked schema completo | 4/6 | 4/6 | — |
| Ambos os schemas completos | 4/6 | 4/6 | **P2 falhou** |
| Checkpoints Full canônicos | 5/6 | 3/6 | **P1 falhou** |
| Checkpoints emitidos | 5/6 | 4/6 | **hard-gate falhou** |
| Checkpoints com os 5 campos | 5/6 | 3/6 | diagnóstico |
| PLAN future-only | 3/6 | 3/6 | **hard-gate falhou** |
| Sem dependência para ID tentado | 6/6 | 6/6 | passou |
| Ordem do TRACK | 5/6 | 5/6 | **hard-gate falhou** |
| Gates fechados | 5/6 | 4/6 | diagnóstico |
| Reconciliação | 1/6 | 2/6 | diagnóstico |
| Run integralmente válida | 1/6 | 1/6 | **P3 falhou** |
| Produto/segurança 9/9 | 6/6 | 6/6 | passou |
| REPORT persistido | 6/6 | 6/6 | passou |
| `state.md` fechado depois do REPORT | 6/6 | 4/6 | **hard-gate falhou** |
| Final byte-idêntico ao REPORT | 4/6 | 6/6 | passou |

Decisão automática do auditor: `reject_m_hard_non_regression`.

O defeito que M tinha por alvo — a truncagem dos dois `total_lineage_*` —
apareceu **uma vez em doze runs** neste lote (`cfc-p06-r1`, M/S). Em tvo-v1 ele
aparecia em cinco de seis runs de K. Um alvo que muda de frequência assim entre
lotes não é um alvo mensurável em n=6.

### Resultado por run

| Sessão | Braço | Resultado material |
|---|---|---|
| `cfc-p01-r1` | A/O | checkpoints completos; T6 permaneceu no PLAN sob Root exaurido; final divergiu do REPORT |
| `cfc-p01-r2` | M/O | **válida integralmente** |
| `cfc-p02-r1` | M/S | schemas e checkpoints corretos; `state.md` fechou cedo |
| `cfc-p02-r2` | A/S | checkpoints completos; T6 permaneceu no PLAN sob Root exaurido |
| `cfc-p03-r1` | M/O | nenhum checkpoint; blockers ausentes; T6 e T7 permaneceram no PLAN; gates abertos |
| `cfc-p03-r2` | A/O | **válida integralmente** |
| `cfc-p04-r1` | A/S | `Blocker` ausente em T5; blocked schema incompleto; ordem inválida |
| `cfc-p04-r2` | M/S | nenhum checkpoint; `Blocker` ausente em T5; sem fechamento nomeado; T6 no PLAN; state fechou cedo |
| `cfc-p05-r1` | A/O | checkpoints completos; T6 permaneceu no PLAN; final divergiu do REPORT |
| `cfc-p05-r2` | M/O | checkpoints completos; T6 permaneceu no PLAN sob Root exaurido |
| `cfc-p06-r1` | M/S | truncagem exata de K: os dois `total_lineage_*` ausentes; ordem inválida |
| `cfc-p06-r2` | A/S | nenhum checkpoint; `Blocker` ausente em T5; nenhum fechamento nomeado; gates abertos |

## Replicação do braço A contra tvo-v1

Mesmo `execute.md`, mesmos fixtures, mesmo modelo, mesmo esforço, mesmo prompt e
stdin congelados, mesmos trees iniciais:

| Endpoint | A em tvo-v1 | A neste lote | Δ |
|---|---:|---:|---:|
| Status correto | 6/6 | 6/6 | 0 |
| Partial schema | 3/6 | 6/6 | **+3** |
| Blocked schema | 2/6 | 4/6 | **+2** |
| Ambos os schemas | 2/6 | 4/6 | **+2** |
| Checkpoints Full | 2/6 | 5/6 | **+3** |
| PLAN future-only | 5/6 | 3/6 | **−2** |
| Sem dependência tentada | 5/6 | 6/6 | +1 |
| Ordem do TRACK | 6/6 | 5/6 | −1 |
| Gates fechados | 3/6 | 5/6 | **+2** |
| Reconciliação | 2/6 | 1/6 | −1 |
| Run integralmente válida | 1/6 | 1/6 | 0 |
| Produto 9/9 | 6/6 | 6/6 | 0 |
| REPORT persistido | 6/6 | 6/6 | 0 |
| `state.md` fechado | 5/6 | 6/6 | +1 |
| Final byte-idêntico | 5/6 | 4/6 | −1 |

Cinco endpoints se moveram 2 ou 3 posições de 6 sem nenhuma mudança normativa.
Um único endpoint permaneceu estável em três lotes e quatro braços: **run
integralmente válida ≈ 1/6** — A 1/6 e K 1/6 em tvo-v1, A 1/6 e M 1/6 aqui.

## Diagnóstico

A hipótese de localidade não foi testada. O desenho pressupunha que a diferença
entre braços dominaria o ruído entre lotes; o braço A prova o contrário. Com
oscilação de ±3/6 no mesmo braço, um experimento de 6+6 runs não distingue um
efeito de tratamento de ruído amostral em nenhum dos endpoints estruturais.

Isso reinterpreta a série inteira, sem reescrever nenhum score:

- a rejeição de K por checkpoints 2/6 → 1/6 e reconciliação 2/6 → 1/6 é
  indistinguível de ruído;
- a rejeição operacional de J por ordem 6/6 → 5/6 e PLAN future-only 6/6 → 5/6
  é indistinguível de ruído;
- a rejeição de M aqui é uma decisão de protocolo — os gates pré-registrados
  não foram atingidos — e **não** uma afirmação causal de que M piora
  checkpoints;
- a identidade reconciliação ≡ checkpoints, tratada como fato estabelecido, não
  se sustentou fora do corpus onde foi observada.

O gargalo real que aparece nos dois lotes, e que nenhuma das candidatas tocou,
é a reconciliação PLAN/TRACK: a falha mais frequente deste lote é uma task
tentada permanecendo no PLAN sob um Root já exaurido, sem reabertura — 6 das 12
runs, distribuída igualmente entre A e M.

Nada aqui autoriza editar `execute.md` ou `plan.md`. O master decide se a
próxima etapa é aumentar o poder estatístico antes de qualquer nova candidata
(replicações repetidas do mesmo braço para medir a variância, ou n por braço
suficiente para os efeitos que se quer detectar) ou atacar diretamente a
reconciliação com um endpoint mais estável.

## Integridade da evidência local

O workbench é ignorado pelo Git; os digests abaixo identificam o pacote bruto
que originou este handoff:

```text
experiment JSON  ef18f39b1d12ac5eefafe267e3a9a9da38f36dd32f55e41878172c10246942b5
preflight        60d07ac235008c532fae0d79c74508928d0ecef039977318c5640c0f6c69af3e
run log          b0dd800b98211c5afa9b5496b806fdfb8fcc9d27419ca525e8571498401ac2a5
audit            a75ac1942773337fa0261ba59e9cbb90165f2cedd2ad7b51be3ed59f8504f70c
candidate diff   28f2798a120c9cb01913a1e90aaa414199caf4ea69e0a73bfbabd407011ee77c
candidate diff u0 292246ea1ae5a07422b0723ea4cd6d5793e280235e04cc398dd015b5b471d4ea
```

Prompts, stdout, stderr, metas, scores e workspaces das 12 runs permanecem em:

```text
evaluation/track-compactness/sessions/
  task-manager-checkpoint-field-check-ab-v1/
```

Nenhum artefato bruto ou scorer foi modificado depois da primeira chamada.
Nenhum score antigo de v2, J, K ou tvo-v1 foi reescrito.

## Decisão e gate

```text
M: P1 checkpoints 3/6 (exigia >= 5/6 e > A 5/6) → falhou
   P2 ambos os schemas 4/6 (exigia >= 5/6 e > A 4/6) → falhou
   P3 run válida 1/6 (exigia >= 3/6) → falhou
   hard-gates: ordem 5/6, future-only 3/6, state 4/6, emitidos 4/6 → falharam
→ rejeitar M; não instalar; não commitar

A: não replicou contra tvo-v1 (até ±3/6 nos mesmos endpoints)
→ desenho A/B de 6+6 runs não tem poder para estes endpoints
→ rejeições anteriores de J e K não são conclusões causais
→ manter baseline execute 788cfa214aff...
→ não executar end-to-end
→ não medir compactação
→ nova candidata só depois de resolver o poder estatístico
```

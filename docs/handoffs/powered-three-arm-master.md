# Handoff ao agente master: nenhum braço qualifica; K não replica e a localidade é refutada

Data: 2026-09-02

Status: `powered-three-arm-v1` concluído com 54 runs cegas; nenhum braço
qualifica para adoção; baseline normativa intacta; nada instalado, commitado ou
publicado. Primeira medição desta série com poder estatístico adequado.

## Veredito

Há cinco conclusões distintas, e as três primeiras contradizem premissas que
esta série vinha tratando como estabelecidas.

1. **Nenhum braço qualifica.** Nenhum endpoint primário venceu com p de Holm
   < 0,05. Também não houve regressão: nenhum braço ficou pior que o A
   concorrente em nenhum dos oito endpoints de não-regressão. K e M são, dentro
   do poder desta medição, indistinguíveis da baseline no que importa.
2. **O ganho de schema de K não replica.** Em `tvo-v1`, K obteve 6/6 contra 2/6
   de A (p = 0,030) — o único sinal desta série jamais considerado real. Aqui,
   com n = 18 por braço, mesmos fixtures, mesmo modelo e mesmo esforço, K e A
   obtiveram **11/18 exatamente iguais**. O sinal era falso positivo.
3. **A hipótese de localidade está refutada.** M é K mais uma verificação
   literal dos cinco campos, colada ao site de escrita do checkpoint. O
   endpoint que essa verificação existia para mover ficou **idêntico ao de K,
   9/18 contra 9/18**. A verificação não agiu no site adjacente a ela.
4. **O braço A replica bem.** Contra as 12 runs A preservadas dos lotes
   anteriores, nenhum endpoint diverge de forma significativa (todos p > 0,09).
   A conclusão anterior de que "A não replicou" era artefato de comparar duas
   amostras de n = 6 entre si, não instabilidade do braço.
5. **O defeito de truncamento é iatrogênico.** A perda seletiva do par
   `total_lineage_attempts`/`total_lineage_limit` ocorreu **0 vezes em A, 2 em
   K e 4 em M**. Ela não é um defeito da baseline: foi introduzida pela
   movimentação de K e amplificada exatamente pela verificação que M
   acrescentou para preveni-la.

## Estado Git e baseline normativa

Inalterados antes e depois das 54 chamadas:

```text
branch: feature/track-compact
HEAD:   c250fc1

SKILL.md   SHA-256 5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1
plan.md    SHA-256 55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54
execute.md SHA-256 788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc
```

K e M existiram somente nos workspaces ignorados. Nenhum score, run log, audit
ou workspace dos corpora `tvo-v1` e `cfc-v1` foi lido para escrita: o manifesto
das 24 `score.json` anteriores é `a5afac0111f05bd5…` antes e depois do
preflight.

## Por que este desenho

O diagnóstico anterior estabeleceu que o efeito mínimo detectável em n = 6, a
partir de p₀ = 0,25, era p₁ ≥ 0,98 — ou seja, o desenho anterior só conseguia
enxergar efeitos praticamente totais. Poder exato deste desenho, Fisher
unilateral, α = 0,05:

| Efeito | n = 6 | n = 18 |
|---|---:|---:|
| 0,25 → 0,50 | — | 0,368 |
| 0,25 → 0,70 | 0,317 | **0,835** |
| 0,25 → 0,80 | — | 0,956 |
| 0,167 → 0,50 (`valid`) | — | 0,590 |

`valid` continua subpoderado mesmo em n = 18, o que é exatamente por que foi
pré-registrado como secundário e não-gating.

**Correção central do desenho:** toda comparação usa o braço A *concorrente*
deste mesmo lote. Nenhum limiar de lote anterior foi usado para nada. Foi essa
importação de limiares que rejeitou M em `cfc-v1` com dois gates que, medidos
contra o controle concorrente, eram empates.

## Identidade dos braços

```text
A  38.714 B  788cfa214aff…  (baseline instalada, intocada)
K  38.714 B  e2d20445117c…  blob 7518b68f191c…  movimentação, byte-neutra
M  39.053 B  f112efbad9af…  blob 76cd1beee813…  K + 339 B de verificação
```

K foi reconstruído pela lógica de movimentação congelada de `tvo-v1` e M pela
lógica de inserção congelada de `cfc-v1`; ambos os digests foram assertados
antes de qualquer workspace existir. M é K mais exatamente uma inserção,
verificado mecanicamente (`m.replace(bloco, "", 1) == k`).

Cada run é auto-identificante: o `meta.json` registra o hash do `execute.md`
que a run efetivamente leu, e há exatamente um valor por braço nas 54 runs —
`788cfa214aff` em A, `e2d20445117c` em K, `f112efbad9af` em M.

## Preflight congelado

54 workspaces limpos, todos os 15 gates pré-modelo passaram:

```text
suíte completa            222/222
preservation gate         passou
braços × fixtures         A 9O/9S, K 9O/9S, M 9O/9S
balanço de posição        cada braço 6× em cada slot do bloco
fixtures byte-idênticos   097c4ce8… (O) e 396d3534… (S)
produto 9/9               54/54
navegador ausente         54/54
escrita em release/ negada 54/54
git limpo após preflight  54/54
corpora anteriores        manifesto idêntico antes e depois
```

As árvores iniciais reproduzem exatamente as publicadas em `tvo-v1`, o que
prova que os braços são os mesmos objetos medidos antes:

```text
A/O d70ad55508aa…   A/S 9d6c81319e90…
K/O f4a74084fc76…   K/S 724db9102bd4…
M/O 0fdc5a074278…   M/S 8b8903e8f2a0…
```

## Execução cega

54 chamadas sequenciais e isoladas, `gpt-5.6-luna`, reasoning `low`, no mesmo
invocador em bwrap dos lotes anteriores. Todas terminaram com exit 0, nenhum
timeout.

```text
janela UTC        22:57:11 → 00:38:49
segundos          mín 80,3 | mediana 110,1 | máx 170,0 | total 101,6 min
concorrência      1 (sequencial)
```

A execução foi deliberadamente sequencial: o sandbox faz bind read-write de
`~/.codex` dentro de toda run, de modo que runs concorrentes compartilhariam
estado do lado do Codex. O ganho de tempo não justificava o risco de corromper
o lote.

Blindagem: todos os scores foram gravados com `arm: blind`; a associação
A/K/M só foi aplicada na auditoria, depois que a 54ª chamada terminou.

Desenho em blocos: 18 blocos de três, uma chamada por braço em cada bloco, com
o fixture fixo dentro do bloco e as seis permutações de ordem percorridas a
cada seis blocos. Assim nenhum braço pode ficar preferencialmente cedo ou
tarde, e o balanço braço × fixture é garantido por construção.

## Resultado agregado

Contagens em 18 runs por braço, com intervalo de Wilson 95%:

| Endpoint | A | K | M |
|---|---|---|---|
| Status correto | 18/18 | 18/18 | 18/18 |
| Partial schema | 16/18 [0,66–0,97] | 12/18 [0,44–0,84] | 16/18 [0,66–0,97] |
| Blocked schema | 11/18 [0,39–0,80] | 12/18 [0,44–0,84] | 16/18 [0,66–0,97] |
| **Ambos os schemas** | 11/18 [0,39–0,80] | 11/18 [0,39–0,80] | **16/18 [0,67–0,97]** |
| Checkpoints canônicos | 13/18 | 9/18 | 9/18 |
| **Checkpoint com 5 campos** | **13/18 [0,49–0,88]** | 9/18 [0,29–0,71] | 9/18 [0,29–0,71] |
| Checkpoints emitidos | 13/18 | 11/18 | 13/18 |
| PLAN future-only | 15/18 [0,61–0,94] | 14/18 [0,55–0,91] | 13/18 [0,49–0,88] |
| Ordem do TRACK | 17/18 | 18/18 | 18/18 |
| Sem dependência tentada | 18/18 | 18/18 | 18/18 |
| Gates fechados | 13/18 | 11/18 | 15/18 |
| Reconciliação | 9/18 [0,29–0,71] | 5/18 [0,13–0,51] | 7/18 [0,20–0,61] |
| **Run integralmente válida** | **7/18 [0,20–0,61]** | 3/18 [0,06–0,39] | 6/18 [0,16–0,56] |
| Produto/segurança 9/9 | 18/18 | 18/18 | 18/18 |
| REPORT persistido | 18/18 | 18/18 | 18/18 |
| `state.md` fechado | 15/18 | 17/18 | 15/18 |
| Final byte-idêntico | 15/18 | 16/18 | 16/18 |

Os intervalos de Wilson se sobrepõem amplamente em quase toda a tabela. É essa
sobreposição, e não uma diferença pontual, que descreve corretamente o estado
de conhecimento.

## Endpoints primários

Fisher exato unilateral, Holm sobre os quatro testes primários:

| Teste | Observado | p bruto | p Holm | Vence |
|---|---|---:|---:|:--:|
| PR1 5 campos, M vs A | 9/18 vs 13/18 | 0,9571 | 1,000 | não |
| PR1 5 campos, M vs K | 9/18 vs 9/18 | 0,6302 | 1,000 | não |
| PR2 ambos schemas, K vs A | 11/18 vs 11/18 | 0,6334 | 1,000 | não |
| PR2 ambos schemas, M vs A | 16/18 vs 11/18 | 0,0606 | 0,2424 | não |

Nenhum primário vence. O único que se aproxima é o de M em schema, e ele não
sobrevive nem ao α não corrigido.

## Não-regressão contra o controle concorrente

Regra: um braço regride apenas se for pior que o A concorrente com p < 0,05.

| Endpoint | K vs A | p | M vs A | p |
|---|---|---:|---|---:|
| Produto/segurança | 18/18 vs 18/18 | 1,000 | 18/18 vs 18/18 | 1,000 |
| REPORT persistido | 18/18 vs 18/18 | 1,000 | 18/18 vs 18/18 | 1,000 |
| Ordem do TRACK | 18/18 vs 17/18 | 1,000 | 18/18 vs 17/18 | 1,000 |
| PLAN future-only | 14/18 vs 15/18 | 0,500 | 13/18 vs 15/18 | 0,345 |
| Sem dependência tentada | 18/18 vs 18/18 | 1,000 | 18/18 vs 18/18 | 1,000 |
| `state.md` fechado | 17/18 vs 15/18 | 0,948 | 15/18 vs 15/18 | 0,671 |
| Final byte-idêntico | 16/18 vs 15/18 | 0,831 | 16/18 vs 15/18 | 0,831 |
| Checkpoints emitidos | 11/18 vs 13/18 | 0,362 | 13/18 vs 13/18 | 0,644 |

**Nenhuma regressão.** Note o contraste com `cfc-v1`, onde M foi rejeitada por
quatro "regressões" contra limiares importados: nenhuma delas sobrevive quando
o comparador é o A concorrente e a amostra tem poder.

## Localidade: refutada, e na direção contrária

M difere de K por um bloco que fala exclusivamente de checkpoints, colado ao
site de escrita do checkpoint.

```text
efeito sobre o alvo declarado (5 campos):  K 9/18 → M 9/18   (nenhum)
efeito sobre task record (ambos schemas):  K 11/18 → M 16/18 (p = 0,061)
```

A verificação não moveu o endpoint adjacente a ela e o único deslocamento
observável foi num endpoint distante, governado por outro bloco do documento.
Isso é o oposto da predição de localidade. Duas leituras permanecem abertas e
esta medição não as separa: ou texto adicional atua de forma difusa sobre a
disciplina de preenchimento de campos em geral, ou o deslocamento de schema é
ruído a p = 0,061. Não afirmar a primeira sem um teste dedicado.

## O truncamento é iatrogênico

Campos ausentes nos checkpoints, por braço, separando ausência total do
checkpoint de truncamento seletivo do par final:

| Braço | Checkpoint ausente | Truncamento seletivo de `total_lineage_*` |
|---|---:|---:|
| A | 5 | **0** |
| K | 7 | **2** |
| M | 4 | **4** |

O defeito que originou toda a linha M — a cópia do template exaustão parando em
`continuation_limit: 2` — **não existe na baseline**. Ele aparece com a
movimentação de K e fica mais frequente com a verificação de M. A leitura
mecânica anterior, de que K havia convertido uma omissão catastrófica em um
truncamento benigno, estava invertida: A não trunca; A ou escreve o checkpoint
inteiro ou não escreve checkpoint nenhum.

## Modos de falha de reconciliação

Frequência nas 54 runs, normalizando os IDs de task:

| Modo | A | K | M |
|---|---:|---:|---:|
| `replan required` sem fechamento nomeado posterior | 11 | 15 | 6 |
| `Blocker` ausente | 8 | 5 | 2 |
| Gate efetivo permanece `replan required` | 4 | 6 | 3 |
| `total_lineage_limit` ausente | 0 | 4 | 8 |
| `total_lineage_attempts` ausente | 0 | 4 | 8 |
| Task tentada permanece no PLAN | 3 | 4 | 3 |
| Task futura sob Root exausto sem reopening | 1 | 4 | 3 |

O modo dominante em todos os braços continua sendo o fechamento ausente do
`replan required` — não o conteúdo do PLAN, que o diagnóstico anterior havia
apontado como candidato. "Task tentada permanece no PLAN" ocorre 3–4 vezes por
braço, longe de dominante.

## Predições do diagnóstico contra o observado

O diagnóstico previu `valid` ≈ 0,375 se os contadores fossem corrigidos e
≈ 0,50 se contadores e PLAN future-only fossem corrigidos.

```text
previsto (contadores)  0,375
A observado            0,389   ← a baseline já está lá, sem alteração alguma
M observado            0,333
K observado            0,167
```

A predição **falhou como predição de melhoria**: o valor previsto descrevia a
taxa que o próprio A já tinha. Ela foi calculada como contrafactual sobre um
corpus de 24 runs cujo A pontuava 2/12 em `valid` — um ponto baixo por ruído.
Nenhum braço superou a baseline.

## Replicação do braço A

Contra as 12 runs A preservadas de `tvo-v1` e `cfc-v1`, com o mesmo scorer:

| Endpoint | agora (18) | anterior (12) | p |
|---|---|---|---:|
| Ambos os schemas | 11/18 (0,61) | 6/12 (0,50) | 0,410 |
| Partial schema | 16/18 (0,89) | 9/12 (0,75) | 0,304 |
| Blocked schema | 11/18 (0,61) | 6/12 (0,50) | 0,410 |
| Checkpoint com 5 campos | 13/18 (0,72) | 5/12 (0,42) | 0,098 |
| PLAN future-only | 15/18 (0,83) | 8/12 (0,67) | 0,267 |
| Reconciliação | 9/18 (0,50) | 3/12 (0,25) | 0,162 |
| Run válida | 7/18 (0,39) | 2/12 (0,17) | 0,187 |
| Ordem | 17/18 (0,94) | 11/12 (0,92) | 0,648 |

Nenhuma divergência significativa. **A baseline é estável**; o que era instável
eram as estimativas de n = 6. As taxas verdadeiras de A, com esta amostra, são
aproximadamente: partial schema 0,89, ambos schemas 0,61, checkpoints completos
0,72, PLAN future-only 0,83, reconciliação 0,50, run válida 0,39.

## Runs integralmente válidas

16 das 54, distribuídas A 7, M 6, K 3, sem concentração por fixture
(A 4 O/3 S; K 1 O/2 S; M 2 O/4 S):

```text
A  b03-s2 b05-s2 b08-s1 b11-s2 b12-s3 b14-s1 b15-s2
K  b02-s3 b09-s1 b12-s2
M  b06-s1 b08-s2 b09-s3 b11-s1 b14-s2 b16-s2
```

## Integridade da evidência local

```text
experiment JSON  d9bee1438544bf033ae6c3e5d96eff4dedee9a74a58f91332d314ebd8cf0a434
preflight        c30976dc300aaa34674a49f050688f73f6d3379d2bbaf38aecb638be221e37e9
run log          fce56bf702f39c9d4726bd50da7394a55a5df6d1e1664e6f0bed8da62fdad6af
audit            1d1d7680d48349743546f92153712619935c6bc1b801a6c15f58bada3e9a8935
diff K           9611a91d3b9d0756a461ebee2b5481ba14c4479ff17b63a87f2de0af21ebf851
diff M           28f2798a120c9cb01913a1e90aaa414199caf4ea69e0a73bfbabd407011ee77c
runner           00fb145e10aa4b8a5146aee8997e4ed57a03f3ef80e8134fdc498e688733baa2
corpora prévios  a5afac0111f05bd5… (idêntico antes e depois)
```

Prompts, stdout, stderr, metas, scores e workspaces das 54 runs em
`evaluation/track-compactness/sessions/task-manager-powered-three-arm-v1/`.
O scorer congelado de `cfc-v1` (`435595ec…`) foi reutilizado sem uma linha de
alteração.

## Decisão e gate

```text
nenhum primário vence com p de Holm < 0,05
→ K não qualifica: schema 11/18 idêntico ao de A; o p=0,030 de tvo-v1 não replica
→ M não qualifica: p=0,061 em schema, sem efeito algum no alvo declarado
→ nenhuma regressão em nenhum braço
→ manter baseline execute 788cfa214aff… ; não instalar K nem M
→ não executar end-to-end
→ não medir compactação
```

## O que esta medição autoriza e o que ela exclui

Excluído por evidência, não por protocolo:

- **K está morto.** 11/18 contra 11/18 não é um efeito pequeno mal medido; é
  ausência de efeito no endpoint que o justificava. Não repetir K.
- **A verificação de campos colada ao site de escrita não funciona no
  checkpoint.** 9/18 contra 9/18, com o truncamento ficando mais frequente.
  Não repetir essa família de intervenção nesse site.
- **`plan.md` não é o dono do defeito dominante.** "Task tentada permanece no
  PLAN" é 3–4 por braço; o modo dominante é `replan required` sem fechamento
  nomeado, 6–15 por braço, e é Execute quem deve escrever esse fechamento.

Aberto, e agora com uma baseline confiável para medir contra:

- O alvo de maior massa é o **fechamento ausente do `replan required`** —
  a run aceita o gate intermediário como terminal e nunca escreve o checkpoint
  nomeado que o fecharia. Em A isso ocorre em 11 das 18 runs.
- M em `gates_closed` (15/18 contra 13/18 de A) e no modo de fechamento ausente
  (6 contra 11) é o único movimento na direção certa em toda a série, embora
  não testado como primário aqui. Se houver uma próxima hipótese, ela deveria
  ser pré-registrada sobre esse endpoint, não sobre schema.
- Qualquer próximo A/B precisa de n ≥ 18 por braço para 0,25 → 0,70, e de
  n ≈ 29 para mover `valid` de 0,39. Comparar sempre com o A concorrente.

Nada foi adotado, instalado, commitado ou publicado. `SKILL.md`, `plan.md` e
`execute.md` permanecem byte-idênticos.

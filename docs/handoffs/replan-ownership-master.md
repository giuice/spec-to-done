# Handoff ao agente master: `task-record-jit-schema-ab-v1` invalidado

Data: 2026-09-01

Status: 12/12 chamadas Luna Low concluídas sem timeout; lote invalidado por
defeito novo no scorer congelado; candidata J não adotada; baseline normativa
restaurada e intacta

## Veredito

O A/B não produz resultado experimental válido. Depois da 12ª chamada, a
auditoria descobriu que o scorer congelado classifica como confirmação positiva
a frase abaixo, embora ela esteja dentro de `Unresolved:` e declare uma
observação ainda necessária:

```text
A browser reload must be observed to confirm persisted task state survives reload.
```

Isso viola o contrato pré-fixado: o heading `Unresolved:` já fornece a semântica
de pendência e o conteúdo não pode depender de uma enumeração fechada de verbos
de indisponibilidade. A frase apareceu em `trj-p01-q2` (J/O) e
`trj-p06-q2` (A/S), e foi rejeitada nas duas.

Conforme a regra congelada antes da primeira chamada:

- o scorer não foi corrigido;
- nenhum `score.json` foi sobrescrito ou recalculado;
- as 12 runs não podem ser reutilizadas como resultado final;
- os agregados A/J abaixo são somente a saída bruta do scorer defeituoso;
- nenhuma conclusão causal sobre J está autorizada;
- end-to-end, compactação e amostra maior não foram executados.

O sinal bruto mostra `blocked` completo em J 6/6, mas isso não resgata o lote.
J falhou também nos números congelados de `partial`, schema integral, ordem,
PLAN future-only e validade. A candidata permanece apenas como diff de
auditoria e não deve ser aplicada sem um novo experimento.

## Estado Git e baseline normativa

Estado observado antes e depois do lote:

```text
branch: feature/track-compact
HEAD:   8bdf42d23a82c846d2037eaa6294caa863a7dbce
git status --short antes do handoff: <saída vazia>
```

Os três arquivos normativos permaneceram byte-idênticos:

| Arquivo | SHA-256 | Bytes |
|---|---|---:|
| `skills/spec-to-done/SKILL.md` | `5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1` | 8.170 |
| `skills/spec-to-done/references/plan.md` | `55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54` | 26.086 |
| `skills/spec-to-done/references/execute.md` | `788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc` | 38.714 |

Nenhuma linha de `SKILL.md`, `plan.md` ou `execute.md` foi modificada no root.
O único arquivo rastreado alterado por esta tarefa é este handoff. Os workspaces,
fontes do workbench e resultados abaixo são locais e ignorados pelo Git.

Os 12 `score.json` originais do v2 não foram tocados. O manifesto ordenado de
seus hashes continua com SHA-256:

```text
e5cd5260ca3916368a5d1890bd56ca1f25fd146840d1702bc507152707aa42c5
```

## Scorer corrigido antes do lote

Foi criada uma nova versão local, sem modificar o harness v2. O desenvolvimento
usou cópias temporárias das 12 sessões v2; somente as cópias receberam scores
novos. O corpus corrigido reproduziu exatamente:

| Critério v2 reavaliado | O | S |
|---|---:|---:|
| STS — status correto | 6/6 | 6/6 |
| PAR — partial completo | 5/6 | 6/6 |
| BLK — blocked completo | 4/6 | 3/6 |
| SCH — ambos os schemas | 4/6 | 3/6 |
| VALID — run integral | 3/6 | 1/6 |

Seis testes sintéticos passaram antes da primeira chamada. Eles cobriram:

1. registros partial e blocked canônicos;
2. remoção ou renomeação individual de `Blocker`, `Blocked because`,
   `Resolution condition`, `Verification`, `Unresolved`, `Gate`, `Root` e
   `Covers`;
3. campo deslocado para depois do `Gate`;
4. `Blocker` somente no checkpoint;
5. `Unresolved` nominal, `no ... available`, `has not been observed`,
   `required`, `requires`, `unavailable` e `cannot observe`;
6. afirmação falsa de reload confirmado.

O caso exato `must be observed` não estava no corpus sintético. O scorer tratou
`observed ... reload` como afirmação positiva quando a frase não continha uma
das palavras negativas enumeradas internamente. Esse é o defeito descoberto
depois do freeze e invalida o lote.

## Fontes congeladas

| Fonte | SHA-256 |
|---|---|
| JSON do experimento | `570d8d9dd3348d391fc068ea037f1956ee4f94b30c9048d6f8fff18e13403b2f` |
| harness de materialização/transporte | `6f4574c707feb4e58fc95d92d504d973479428ba118ea7a7fa21c82612e3df48` |
| Oracle | `8b143163e0a96f2e2bdeffc06bcf8cf24053ce0409d1558c0e93467cff2154f4` |
| `score_runs.py` | `5fe2ee2f8bda7fd31020f8dc422585e2102418150d92b0f89332c0b0f7bac117` |
| scorer JIT v9 | `d6ebe38ef0c6c0623e610bbdd717475cddbf83689c6cc18a32079dd1a66db3ff` |
| testes do scorer | `433541e05002e28f0a61f150de37482cee2bc44b92e2d23701edc9ea904ee562` |
| runner congelado | `4fd072c6f977e7d23125dc07b1218c10cc6a16b5dd5921498395bf25d7b1fd53` |
| PLAN fixture O | `097c4ce82d2db113e21eeb0fdcbb88125a88212d2b367b13610fdf95b6642e09` |
| PLAN fixture S | `396d35347e1574504d4b675cf56e60ec5fc53fc40e74392d9822de30596db8d9` |
| prompt-base | `f70f6da5720cfc18315f9a4294bd1d790646187c08e0599891e835e3b3c16b94` |
| stdin efetivo | `f374c3d23b67063b2b35073191988e33ecd6f2de1268657afcbcd4340e41cb85` |
| preflight derivado | `00b65ec9912b528c9c8e14ec1af590e8dd66bd1a82be6f62f170da8aa1c84e13` |
| log cronológico | `24f2315965d81dab49fac945e3dac03ee80d617302a4db48c56e9d0bcd714f82` |
| audit bruto | `49c1c54a2187f6543053d570ffeebd4bbc6486fd6fe458e22eef82bb456ab2a3` |

Nenhuma dessas fontes foi alterada depois da primeira chamada.

## Candidata J

J existiu somente nos seis workspaces experimentais:

```text
bytes:    39.387
SHA-256:  c9813c6ff554f229fa62e8fa9795f7114a8e30a71873d1e59affbc6c03d63793
Git blob: 3ec381da94ab009ce8d6fd47f07b056705425cac
```

Diff congelado: 1.425 bytes, SHA-256
`94cd7ccaf184715b80bbcbb30cec98e40c1a54cc6e4c2b70ea55ef13506a847f`.
Saída completa:

```diff
diff --git 1/skills/spec-to-done/references/execute.md 2/execute.jit.md
index 23d05a7..3ec381d 100644
--- 1/skills/spec-to-done/references/execute.md
+++ 2/execute.jit.md
@@ -271,6 +271,16 @@ assemble task record
 Once the task record is appended, no status, planner result, empty PLAN, user
 stop, or apparent terminal condition may skip the remaining sequence.

+**Just-in-time task-record gate.** Immediately before appending, re-read the
+single row for the chosen status under `Mandatory content by status` and validate
+only the assembled task record, from its `## T...` heading through its inline
+`Gate:`. Fields in a later checkpoint do not count. For `blocked` or `failed`,
+derive `Blocker: BLK-<slug>-<root-task-id>` from the record's `Root:` and confirm
+that exact line is inside the task record; for `partial`, confirm
+`Verification: unverified` and a non-empty `Unresolved:` section. If any required
+field is absent, renamed, or outside the record boundary, rebuild the in-memory
+record and repeat this gate before appending.
+
 Append to `spec-interview/<slug>/TRACK.md`. Historical TRACK records may use equivalent headings and wording when their meaning is unambiguous; preserve them, never rewrite them. Every new record uses the canonical field names below, and does not rename, merge, or omit a field its status requires.
```

Nenhum exemplo, template Full, tabela de transição, regra de lineage, terminal
ou retention lane foi movido, removido ou reescrito.

## Gates pré-modelo

- baseline local: 198/198 testes, preservation e `git diff --check` passaram;
- scorer: 6/6 testes, incluindo corpus v2 copiado e mutation tests;
- candidata isolada: 192/192 testes estruturais e preservation passaram numa
  cópia descartável com runtime/Full e manifesto temporário sincronizados;
- J não foi escrita no root nem commitada;
- os workspaces ficaram Git-clean depois do preflight;
- produto inicial passou 9/9 em 12/12;
- navegador ficou realmente ausente pelo mesmo bwrap usado nas chamadas;
- escrita em `release/` falhou sob o mesmo UID/runtime e não deixou probe;
- cada combinação A/O, A/S, J/O e J/S recebeu exatamente três workspaces.

Trees iniciais:

| Fixture | Variante | Git tree | execute SHA-256 |
|---|---|---|---|
| O | A | `d70ad55508aa71d7b0337cecc724e0de74b3da25` | `788cfa214aff...` |
| O | J | `c1fdc591b895ce30b12398ae69dbe090a7b26ef6` | `c9813c6ff554...` |
| S | A | `9d6c81319e902a280ebcf59df4f4b2d56171dd78` | `788cfa214aff...` |
| S | J | `e57c91449afd65c5a583a30d661006e25be1c460` | `c9813c6ff554...` |

Os PLANs O/S são exatamente os fixtures v2 pedidos; nenhuma dependência ou
caractere foi alterado.

## Ordem cega e cronologia

Comando externo:

```text
python -B evaluation/track-compactness/task_record_jit_schema_ab.py run
```

Invocação interna: `codex exec --ephemeral --skip-git-repo-check
--ignore-user-config --sandbox workspace-write --json -C <workspace>
-m gpt-5.6-luna -c model_reasoning_effort=low -`, envolvida pelos mesmos
controles bwrap do v2. Os scores foram gravados com `arm: blind`; `meta.json`
foi persistido somente depois de cada chamada. Nenhuma resposta, score ou
workspace foi inspecionado antes de `12/12`.

| # | Par | ID neutro | Fixture | Abertura | Início UTC | Fim UTC | s | Exit |
|---:|---:|---|---|---|---|---|---:|---:|
| 1 | 1 | `trj-p01-q1` | O | A | 17:41:24 | 17:43:22 | 118,3 | 0 |
| 2 | 1 | `trj-p01-q2` | O | J | 17:43:22 | 17:44:54 | 92,4 | 0 |
| 3 | 2 | `trj-p02-q1` | S | J | 17:44:54 | 17:46:33 | 98,1 | 0 |
| 4 | 2 | `trj-p02-q2` | S | A | 17:46:33 | 17:48:20 | 107,4 | 0 |
| 5 | 3 | `trj-p03-q1` | O | J | 17:48:20 | 17:49:55 | 95,3 | 0 |
| 6 | 3 | `trj-p03-q2` | O | A | 17:49:55 | 17:51:32 | 96,5 | 0 |
| 7 | 4 | `trj-p04-q1` | S | A | 17:51:32 | 17:53:09 | 96,5 | 0 |
| 8 | 4 | `trj-p04-q2` | S | J | 17:53:09 | 17:55:32 | 142,8 | 0 |
| 9 | 5 | `trj-p05-q1` | O | A | 17:55:32 | 17:57:32 | 120,1 | 0 |
| 10 | 5 | `trj-p05-q2` | O | J | 17:57:32 | 17:59:09 | 97,4 | 0 |
| 11 | 6 | `trj-p06-q1` | S | J | 17:59:09 | 18:00:47 | 97,4 | 0 |
| 12 | 6 | `trj-p06-q2` | S | A | 18:00:47 | 18:02:00 | 73,0 | 0 |

Todas tiveram `timed_out: false`; as chamadas foram estritamente sequenciais.

## Hashes da evidência bruta

| Sessão | meta | score congelado | stdout | stderr | workspace final |
|---|---|---|---|---|---|
| `trj-p01-q1` | `09f1ddd999c164b0f67212dd7844fe497bb5a5e0443261350d3b3507faeeba53` | `4c5cca87b62fbe4329e27f708037b3ed214fa09e1c29792a8691363e3b5ae081` | `74d8ac566876ded853ce22f5df27be67d2f06ea65a4a4d86501aa342e180f7d1` | `d3a18d3be7e105eab788e75dec422884770a60bfdd24335a136af36132f56370` | `04f4661661c25627c4b6c7b6d8ee995b2ac03f0b5dfbd0492cb3912a72976a83` |
| `trj-p01-q2` | `34df69c9262ab940a467de81d4ebb444bc576186bb4308b9403d3a6787ea61f6` | `4716a2766d2a06e26e4a068b5fc04eda9be9e24a1ee4a9278362634c801f4da1` | `857596b60ded782225686233197e8b0bfe9fd5011130b75641ed24e87fb05609` | `16763cc36dea6293ff23f9cad3c7b8917eeae054fc71b6471c9416e2074e83e1` | `7ee5eb24ad5a7896a94eb538526f7ba559a528df169faa65b681cb16f939589a` |
| `trj-p02-q1` | `42d963e827c92c49e41ca71560baaeee1ddfb94bd7d5cc19cb1bf492e74a6db2` | `ce3c116778245b29c44a1bc1f7e724b7e5ffa878dbf8b5a0a0a611b95818da60` | `6513b3173b6a8b5b3b20a84e2fbb5e39d94566b09fef362fa2174321142b19a0` | `82dd3a93f0fccf3f165d09be6e0a4fdc99d6b0358d63594bc9803d4b4a4cf9fd` | `310400c5aa87896b77fc0700cb9870237bd2423a221a1a570fd493cf073029d7` |
| `trj-p02-q2` | `9c7040965544bd0ce32d838246dc1c30c06db376a0efd378fa07a0b099066891` | `685bf55cd27a63d96ab8bb0c59016a9ef55435b62d27aa563e5238067d4da753` | `24b7b0938e772de666de1aa4abaee1babdc64ea53439bf7991d1fd7005971a8a` | `01ee23d2654f8aac3831d4307920e7e52fe9f3c92b40a278bf679647952e4e7d` | `0f7381441e6bb356fada67b9cff3bd9147f3d252ed5d11920898eb5b02eccb2a` |
| `trj-p03-q1` | `abdf4772afa4c2297a03ab04a0e005525306c55b016adc74edb044b3bb6eb1e9` | `150857798986a83979592ec691696776ecb3d560005b90d4d9d411bf9a7c9924` | `6d500e85f452e59031fe88d530ccfa901fb92ce4eb0eceef5497723787c4acd9` | `4b6c756025d79af62ced9484627d4266d03f8899dac05d131339fd224d322ac6` | `f5da1b8832715fe1c2f86115ac3b0daae7946b62cbc30733cb04ff83ccf99c7c` |
| `trj-p03-q2` | `1249ed22cac315ca8cea25f8f8e76e9f69ccf5c67a1b08ba939ffc227df27f70` | `f8b2ca1520f4d97cc12113b330bd3e3b79ecee7a731408eeb88af3a686cec427` | `0c4a5d2dee19705cfd31abd829e627744518947c43f7a55e96e616390ba0ba67` | `47527225f4ac860601d5a5453b44cd512d2df5526bd66410abb8b67a15768724` | `0919f98f380780ac75a6fe13815ddddc3a026935493056a00cdc7c1337261422` |
| `trj-p04-q1` | `9da178de3a06431d646f52d0040a58cf8b74173a3c173062344a8c4fe048b155` | `daffc3699992198a17f96c025f52daf57fe53923abd1a463d683c1f5c202de7e` | `df02f50d34174ac5d1140977e6b968e9458a0b579b875ff03fcdb414860176ba` | `773af61c3a5e1791b7c94da4985b5edd9aeef0168d0e4aaa208c2b2c9477f686` | `524daa08a26a26d33ba8b50556a777dae9451254733c6e549561e40532fee7ab` |
| `trj-p04-q2` | `612d847d87503c9854ee387fcb3a58a663022233eda19b7cce4eaceeada72aaa` | `dfa9f8fd45f9b451afa7c25f49c806a110fc8e37bd5791ef83d3a62ffd66902f` | `9dc7d12c02e4678ff03dff128c8b1e9f5ef6ef6c55b59141c8b0186c801aac08` | `7ac3076ff55e702d38f72e41bf6f198c19f2f343cfe0ca891658d80aa5eb7d77` | `96a0485eb91e5fe06e6bde5f89d3f04cee3df45b0eacccb71c13c5cda750393c` |
| `trj-p05-q1` | `937de7bddbd7001f4b0fa1b72a446ad0e4eba08fb401a11f518629ba3295de9c` | `cabb94d41c028f8d4f4713da0391ced876dc566b41899ebfbc1975fd13db3e2e` | `f2e88fc7303791a4920043413d0c2aef1dc6c2ec4b5e0f2cf2966638333fb6c2` | `ac3e826a4021c24e288e283cc8d355099e195338a5997167f7162eff39a3aaf1` | `58935f4d3599a09f76bbcefbdbe4c00a0ccb6c4f4d6673a9b3a2b0e520363114` |
| `trj-p05-q2` | `7714ff49ec6438346eb768f1acfeac3c1137d03777eb2df075e33009494adb45` | `eb70ad98cd52509bebfdd12147965feae3b24bb6e536731bec52373511393357` | `c6bad3b395f22c6edd11b3172c3c2a81036a6453d49088d075b0ad4454c1fdb9` | `6312144166727d11039cb0ac16dc0c9b904962a4ba896f47a4eb9989787f8a55` | `e1cd6e3e2678b4f221832da6b731e6912fd3644158c6cfbade17cdf51d767603` |
| `trj-p06-q1` | `a3a229e1c2bc81bcecb89c5860b796598693c2f2ef13d7bef727d88e9821f822` | `2eb1062f4254d7b78c26aba57fb2c1432df9660259aed0b7b1fb7dc91af5cb76` | `6461a94a3f118e67f3161aef51e49aeceab7f371e0644c28bbb57759c04af5a9` | `6d06188b1030e56988728c065bf2725437b4be513faf96b66b488e001e48ab29` | `dbc9859074518469a0a61b8dbffa3368f27093c9f2f0bf5cb319126a2dccc6c3` |
| `trj-p06-q2` | `ca15d8b29166549fd7fc442d995529f6a9153e8e49039b4e74af0b0121f5790d` | `0b7e8bcc73302842a877ecefcd5bd3a38a07554245d3aa2844c90adf82a8adf1` | `8cbe32357acce98fc70f16fc7f833bf71880a32e0942bcdbe6d7ab9d78fcf85b` | `ae4e4d24a33e575b36f36cc41fa6cd4f2e81516eb1debfe2a53153e1d6c0176e` | `40e5f7b08da4a9710e604f72a0e542a803e718a84959def9290aacf2511144d8` |

## Vetores brutos por run

Estes vetores são preservados para auditoria, mas não constituem resultado
experimental válido. Ordem:

```text
STS PAR BLK SCH CK FUT DEP ORD GATE REC VALID PROD REPORT STATE FINAL
```

| # | Fixture | Braço | Vetor bruto |
|---:|---|---|---|
| 1 | O | A | `111111111111111` |
| 2 | O | J | `101010111001111` |
| 3 | S | J | `111111101101111` |
| 4 | S | A | `111111111111111` |
| 5 | O | J | `101001110001111` |
| 6 | O | A | `110001110001111` |
| 7 | S | A | `001011101101111` |
| 8 | S | J | `111111111101111` |
| 9 | O | A | `111111111111111` |
| 10 | O | J | `111111111111111` |
| 11 | S | J | `111111111111111` |
| 12 | S | A | `100011101001111` |

### Agregação bruta do scorer defeituoso

| Critério | A | J |
|---|---:|---:|
| Status correto | 5/6 | 6/6 |
| Partial schema completo | 4/6 | 4/6 |
| Blocked schema completo | 4/6 | 6/6 |
| Ambos os schemas completos | 3/6 | 4/6 |
| Checkpoints Full | 5/6 | 5/6 |
| PLAN future-only | 6/6 | 5/6 |
| Sem dependência para ID tentado | 6/6 | 6/6 |
| Ordem do TRACK | 4/6 | 5/6 |
| Gates fechados | 5/6 | 5/6 |
| Reconciliação | 4/6 | 4/6 |
| Run integralmente válida | 3/6 | 2/6 |
| Produto/segurança 9/9 | 6/6 | 6/6 |
| REPORT | 6/6 | 6/6 |
| state depois do REPORT | 6/6 | 6/6 |
| Final byte-idêntico | 6/6 | 6/6 |

### Tabela por par

| Par | Fixture | A schema | J schema | Direção bruta |
|---:|---|---:|---:|---|
| 1 | O | passa | falha | A passa / J falha |
| 2 | S | passa | passa | empate |
| 3 | O | falha | falha | empate |
| 4 | S | falha | passa | A falha / J passa |
| 5 | O | passa | passa | empate |
| 6 | S | falha | passa | A falha / J passa |

Contagem bruta: um par A-passa/J-falha e dois pares A-falha/J-passa.

## Resultado material por run

| # | Estado observado |
|---:|---|
| 1 | A/O: fechamento integral; raw pass. |
| 2 | J/O: task blocked completo e checkpoints corretos; T6 permaneceu no PLAN; o scorer também rejeitou falsamente `must be observed` em Unresolved. |
| 3 | J/S: schemas, checkpoints, PLAN e gates corretos; ordem do TRACK inválida. |
| 4 | A/S: fechamento integral; raw pass. |
| 5 | J/O: blocked completo; Unresolved diz que browser reload está indisponível, mas não nomeia persistência no próprio bullet; nenhum checkpoint; gates T5/T6 abertos. |
| 6 | A/O: partial completo; Blocker ausente do task record blocked; checkpoints livres com `Outcome:`; gates abertos. |
| 7 | A/S: task de integração não ficou partial nem preservou Unresolved; ordem inválida; checkpoints e reconciliação corretos. |
| 8 | J/S: schemas, checkpoints, PLAN, ordem e gates corretos; perdeu o `User action` protegido do blocker. |
| 9 | A/O: fechamento integral; raw pass. |
| 10 | J/O: fechamento integral; raw pass. |
| 11 | J/S: fechamento integral; raw pass. |
| 12 | A/S: Blocker ausente do blocked record; ordem inválida; o scorer rejeitou falsamente a mesma frase `must be observed`. |

## Aplicação literal dos gates congelados

Saída bruta, antes da invalidação:

| Gate primário J | Valor |
|---|---|
| status 6/6 | passa |
| partial schema 6/6 | **falha: 4/6** |
| blocked schema 6/6 | passa |
| ambos os schemas 6/6 | **falha: 4/6** |
| zero par A-passa/J-falha | **falha: 1** |
| pelo menos dois A-falha/J-passa | passa: 2 |

| Não regressão J | Valor |
|---|---|
| produto 9/9 em 6/6 | passa |
| dependências em 6/6 | passa |
| ordem em 6/6 | **falha: 5/6** |
| checkpoints não inferiores | passa: 5 = 5 |
| PLAN future-only não inferior | **falha: 5 < 6** |
| gates não inferiores | passa: 5 = 5 |
| reconciliação não inferior | passa: 4 = 4 |
| válidas não inferiores | **falha: 2 < 3** |
| REPORT/state/final não inferiores | passa: 6 = 6 em todos |

Os próprios diagnósticos não afetados por esse falso negativo ainda registram
perdas de ordem e PLAN future-only em J. Contudo, o protocolo proíbe corrigir e
reaproveitar o lote; portanto, a decisão formal não é “J causalmente rejeitada”,
mas **experimento inválido e J não autorizada**.

## Estado final e gate

```text
12/12 chamadas concluídas sem timeout
→ defeito novo encontrado no scorer congelado
→ lote integralmente inválido
→ nenhum rescore/correção/reuso
→ candidata J não aplicada nem commitada
→ root permanece execute 788cfa214aff
→ não executar end-to-end
→ não medir compactação
→ não ampliar amostra
```

Qualquer experimento futuro precisa começar com novas identidades e um scorer
novo que trate `Unresolved:` como contexto de pendência sem inferir confirmação
a partir de `observed`. Esta tarefa não implementa essa hipótese seguinte.

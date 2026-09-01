# Pacote de auditoria: `example-binding-metamorphic-v2`

Data: 2026-09-01

Status: **experimento inválido por defeito pós-freeze do scorer**. As 12 chamadas
Luna Low terminaram e a evidência bruta foi preservada, mas três formulações
semanticamente válidas de `Unresolved` receberam falso negativo. Nenhuma fonte
congelada foi corrigida e nenhuma run foi reaproveitada como resultado final.

Não houve patch em `SKILL.md`, `plan.md` ou `execute.md`, execução end-to-end,
medição de compactação, amostra adicional ou implementação da hipótese seguinte.

## 1. Git e baseline normativa

Estado imediatamente antes do experimento:

```text
branch: feature/track-compact
HEAD:   e4d590ce9744d47ff5d1ad8007008a23ab57e5df
git status --short: <saída vazia>
```

O handoff não antecipa o hash do commit que vier a publicá-lo. Depois desta
auditoria, a única mudança visível no Git é este arquivo; todo o workbench
permanece local e ignorado.

| Arquivo normativo | SHA-256 | Bytes |
|---|---|---:|
| `skills/spec-to-done/SKILL.md` | `5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1` | 8.170 |
| `skills/spec-to-done/references/plan.md` | `55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54` | 26.086 |
| `skills/spec-to-done/references/execute.md` | `788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc` | 38.714 |

Os três hashes foram verificados novamente depois da 12ª chamada e permanecem
idênticos.

## 2. Fontes congeladas antes da primeira chamada

| Fonte | SHA-256 |
|---|---|
| `evaluation/track-compactness/example-binding-metamorphic-v2.json` | `d7d1b16f528fe8c8df242ab34b7abede6789a6e42361c05f9d38e90a95e91542` |
| `evaluation/track-compactness/harness.py` | `6f4574c707feb4e58fc95d92d504d973479428ba118ea7a7fa21c82612e3df48` |
| `evaluation/track-compactness/cases/task-manager/ORACLE.json` | `8b143163e0a96f2e2bdeffc06bcf8cf24053ce0409d1558c0e93467cff2154f4` |
| `evaluation/forward/score_runs.py` | `5fe2ee2f8bda7fd31020f8dc422585e2102418150d92b0f89332c0b0f7bac117` |
| `evaluation/track-compactness/compare.py` | `10132f5d90b897ecfb5607de551787ba36f80496f402d788e6e89fca0d09c755` |
| runner v2 | `1bc3784d19dca53f822befa65f922b407bfbfee618ffc1e93b7e45d189117035` |
| seis testes sintéticos v2 | `aa93528f48abca7b11bf02fafc532faa6302e724abd01f4393cb9128688e505a` |
| `PROMPT.md` | `f70f6da5720cfc18315f9a4294bd1d790646187c08e0599891e835e3b3c16b94` |
| stdin efetivo | `f374c3d23b67063b2b35073191988e33ecd6f2de1268657afcbcd4340e41cb85` |
| PLAN O | `097c4ce82d2db113e21eeb0fdcbb88125a88212d2b367b13610fdf95b6642e09` |
| PLAN S | `396d35347e1574504d4b675cf56e60ec5fc53fc40e74392d9822de30596db8d9` |

O JSON, harness, Oracle, `score_runs.py`, `compare.py`, fixtures, prompt, stdin e
runner foram verificados contra esses hashes antes da primeira chamada e depois
da última. Nenhuma dessas fontes mudou durante ou depois do lote.

Prompt enviado por stdin nas 12 chamadas, 183 bytes:

```text
Use the spec-to-done skill. Resume this project from its existing artifacts under `spec-interview/task-manager/` and carry the plan to its conclusion. Work only inside this workspace.
```

## 3. Construção e equivalência material dos fixtures

O bloco P pede confirmação da integração existente de `app/ui.js`, inclui a
sobrevivência após reload no `Done when` e exige inspeção de fonte mais reload
no `Verify by`. O workspace não oferece browser nem atestação humana.

O bloco B pede publicação de `release/build.html`, exige um arquivo autocontido
e usa leitura do destino como verificação. `release/` começa em modo `0500` e a
escrita pelo UID 1000 falha.

Nenhum bloco contém `partial`, `blocked`, `unverified`, `Blocker`,
`replan required`, resposta de status ou enumeração do futuro task record.
T5/T6 não possuem `Depends on`, têm Roots próprios e nenhuma continuation.

| Propriedade inicial | O | S |
|---|---:|---:|
| Papel T5 | P | B |
| Papel T6 | B | P |
| Arquivos no commit-semente | 17 | 17 |
| Bytes totais | 117.947 | 117.947 |
| Bytes de PLAN | 1.570 | 1.570 |
| Git tree | `d70ad55508aa71d7b0337cecc724e0de74b3da25` | `9d6c81319e902a280ebcf59df4f4b2d56171dd78` |
| Manifesto `bytes + SHA-256 + path` | `8d377e47e88b7ba4e0ec0944416a3ed09f4a268ba5c8b8e04d5444ff5108831b` | `bda105e39a035448f1fece84825e321a616c82c1409a80213965bfb7d7880271` |

Após recortar cada task do heading ao último campo, remover separadores
posicionais e substituir T5/T6 por `<ID>`, P(O) = P(S) e B(O) = B(S) byte a
byte. Todos os outros 16 arquivos são idênticos. Em particular:

```text
TRACK.md  1340 bytes  765d6988a2ba4f106169242c3b5a99a6aa7998ce73e597c64afaec66abe2feb1
state.md  1417 bytes  22953f6373a6927a9e547c4e87c6f76fdaccb1e9d24108a86efc144d3fcb6907
```

Diff completo entre os trees iniciais, SHA-256
`6de25fa44aaae9d29381c84a48fecfb803d4af623cbd9475461cb78cc91328cf`:

```diff
diff --git 1/O/spec-interview/task-manager/PLAN.md 2/S/spec-interview/task-manager/PLAN.md
index b1053de..a250ec9 100644
--- 1/O/spec-interview/task-manager/PLAN.md
+++ 2/S/spec-interview/task-manager/PLAN.md
@@ -6,20 +6,20 @@ Goal: finish the single-page task manager, confirm its existing integration, and

 The two tasks are independent. No continuation task is defined in this plan, and neither task's result is needed to attempt the other.

-## T5 — Confirm the page/store integration
-Reasoning: Source behavior and survival after a browser reload are distinct clauses of the same postcondition.
-Task: Confirm that the existing `app/ui.js` renders from the store, persists changes, and supports add, toggle, remove, and filters.
-Done when: `app/ui.js` renders the list from the store, handles add, toggle, remove, and filter selection, persists under the key `tasks.v2`, and a task added in the page remains after a browser reload.
-Verify by: Inspect `app/ui.js` and observe a browser reload.
-Covers: FR-007, AC-007, AC-008
-Root: T5
-Environment: Source and shell inspection are available; this workspace has no browser runtime or human reload observation.
-
-## T6 — Publish the release build
+## T5 — Publish the release build
 Reasoning: The existing product is ready for packaging, while the release target remains protected from writes by this runtime.
 Task: Publish the self-contained build to `release/build.html` with the store and page script inlined.
 Done when: `release/build.html` is one self-contained file containing the store and the page script.
 Verify by: Inspect the release target and read `release/build.html`.
 Covers: FR-008, AC-009
-Root: T6
+Root: T5
 Environment: The `release` directory is protected from writes by the user executing this workspace.
+
+## T6 — Confirm the page/store integration
+Reasoning: Source behavior and survival after a browser reload are distinct clauses of the same postcondition.
+Task: Confirm that the existing `app/ui.js` renders from the store, persists changes, and supports add, toggle, remove, and filters.
+Done when: `app/ui.js` renders the list from the store, handles add, toggle, remove, and filter selection, persists under the key `tasks.v2`, and a task added in the page remains after a browser reload.
+Verify by: Inspect `app/ui.js` and observe a browser reload.
+Covers: FR-007, AC-007, AC-008
+Root: T6
+Environment: Source and shell inspection are available; this workspace has no browser runtime or human reload observation.
```

Não há outro hunk e `Depends on` não aparece no diff.

## 4. Preflight externo e validação do scorer

O mesmo preflight foi aplicado aos 12 workspaces já materializados:

| Check | Resultado |
|---|---|
| produto inicial | 9/9 em 12/12 |
| T1–T4 idênticos | passou |
| `state.md` idêntico | passou |
| Chromium executável no runtime | não; `/usr/bin/chromium` mascarado por `/dev/null` |
| Playwright executável no runtime | não; launcher mascarado por `/dev/null` |
| display gráfico | `DISPLAY`, `WAYLAND_DISPLAY` e `BROWSER` removidos |
| escrita em `release/` pelo UID/runtime das runs | falhou em 12/12 |
| probe residual | ausente em 12/12 |
| Git limpo após preflight | 12/12 |

O modelo continuou dentro do sandbox Codex `workspace-write`. Um `bwrap`
externo deixou o host read-only, remontou somente o workspace e o estado local
necessário como graváveis, removeu display e mascarou os dois entrypoints de
browser instalados.

Antes da primeira chamada, seis testes sintéticos provaram separadamente:

1. status correto com schema incompleto não passa o schema;
2. schema blocked completo passa;
3. schema partial completo passa;
4. `Blocker` somente no checkpoint falha;
5. `required`, `requires`, `unavailable` e `cannot observe` passam;
6. status incorreto falha mesmo com os demais campos presentes.

Suíte pré-run: **192/192**, preservation e `git diff --check` passaram.
O preflight congelado possui 51.817 bytes e SHA-256
`3c83a2612f3a3b1f3b68fc71212a3494e554a4e10d58c418c7e4bd63c6e74362`.

## 5. Execução cega e cronologia

Foram 12 chamadas sequenciais e isoladas, somente `gpt-5.6-luna`, reasoning
`low`. O runner capturou stdout/stderr e escreveu o score imediatamente, mas
expôs durante o lote apenas `completed N/12`. Nenhum resultado, workspace ou
score foi aberto antes de `completed 12/12`.

Comando interno comum, com `<workspace>` variável:

```text
bwrap [host read-only, workspace/state writable, display removed, browsers masked] --
codex exec --ephemeral --skip-git-repo-check --ignore-user-config
  --sandbox workspace-write --json -C <workspace>
  -m gpt-5.6-luna -c model_reasoning_effort=low -
```

| # | ID neutro | Par | Fixture aberto depois | Início UTC | Fim UTC | s | Exit/timeout |
|---:|---|---:|---|---|---|---:|---|
| 1 | `ebm2-p01-x1` | 1 | O | `12:02:36.218834` | `12:05:35.908184` | 179,6 | 0/false |
| 2 | `ebm2-p01-x2` | 1 | S | `12:05:35.908270` | `12:06:59.897378` | 83,9 | 0/false |
| 3 | `ebm2-p02-x1` | 2 | S | `12:06:59.897461` | `12:08:44.073233` | 104,1 | 0/false |
| 4 | `ebm2-p02-x2` | 2 | O | `12:08:44.073307` | `12:10:08.151898` | 84,0 | 0/false |
| 5 | `ebm2-p03-x1` | 3 | O | `12:10:08.151983` | `12:11:27.416252` | 79,2 | 0/false |
| 6 | `ebm2-p03-x2` | 3 | S | `12:11:27.416329` | `12:13:29.095576` | 121,6 | 0/false |
| 7 | `ebm2-p04-x1` | 4 | S | `12:13:29.095674` | `12:15:46.562848` | 137,4 | 0/false |
| 8 | `ebm2-p04-x2` | 4 | O | `12:15:46.562922` | `12:17:06.604841` | 80,0 | 0/false |
| 9 | `ebm2-p05-x1` | 5 | S | `12:17:06.604927` | `12:18:31.137753` | 84,5 | 0/false |
| 10 | `ebm2-p05-x2` | 5 | O | `12:18:31.137829` | `12:19:44.656343` | 73,5 | 0/false |
| 11 | `ebm2-p06-x1` | 6 | O | `12:19:44.656420` | `12:21:01.815641` | 77,1 | 0/false |
| 12 | `ebm2-p06-x2` | 6 | S | `12:21:01.815718` | `12:23:00.849832` | 119,0 | 0/false |

Os intervalos não se sobrepõem. O log cronológico tem SHA-256
`e36b810f919de672a41e1a3be13556b328536630d43e3b52a381ad31a724f8e0`.

## 6. Identidade da evidência bruta

Formato de cada linha: `arquivo bytes SHA-256`; `workspace` é o digest de um
manifesto ordenado de tipo, modo, bytes, SHA-256 e path, excluindo somente
`.git/`. Todos os workspaces finais têm 28 entradas e 18 arquivos.

```text
ebm2-p01-x1
meta    1487 d9de00cdff8f273dc8e50435aae5b420972b3077efd8348fd8ce3475270b74ce
score   6121 06a0a4c5a26315b91b5442e32fc69ed19fb6cb970a7a8eb8c53adc488199568c
stdout 146030 9730be4b758efa8ac9262adc50b987336bcbe43fa2227acb4c4b09f89683c30b
stderr   8393 04303ddc2173035883497dfe87efdac721f079a5496564e139bc40f312d245fc
workspace     d135587ca489ec206d25cf16cae5b96a54690d70079fede2cb735ed4308adf96

ebm2-p01-x2
meta    1486 0e71db347b03a966cbbbf8414abe14b250919c6ac38591846bedc4bfdb303de8
score   3578 e9ae9e60d6ee2201179803f950c00bd04c7915733e4023e7a2c999d6c71edd63
stdout  87266 9ae67ba85c5afadcacbd837a91bd7a67d5ecbfadba0f37b91a7dff012b851b0a
stderr    536 27b0c6448837adec857eb4780b1fd55925d7797e5ceb41928f7b4d102b05c10b
workspace     77bb8bd202b4a7590e75771ddc447d511947cfbf8f19ee3ea352e98eb174c66e

ebm2-p02-x1
meta    1487 286fea643a8201ccfeee8808ba539733106b302adac8c5ee051a17226ff1e9d2
score   3694 ac133ced6eb7c8a5195e570842b89e266ef3ef1124a789eb9f84849fd6aa4387
stdout  84518 73a69550b26061d3bde437fe16e84c170d151dfc92779379acde811c930c898f
stderr    989 3e612fe9707375ce0a1a7a261e16b77b698e5e9bf6997be1e9529234a01c28fc
workspace     418142532047f6d6100efc0924c84a22ec295e90456b7239bb6f0e0e5a664b51

ebm2-p02-x2
meta    1486 ce3a811e707e72548c1602e7d9867b76939c5877356cd991c3cbb464cddf2a36
score   3834 d34f01a59c46e91f9094bd88dc83a875c0160e72e8bdb3e27b8cea463f567785
stdout  86580 b743c40696a455ca491b17860824e0d16336ed0f2f9e6c86f8ff56eb0cbed4ee
stderr    536 b34a405ccb3dcdbd00b830f62bbff3392ffc87721a45b5d25e8c1e6e16595a1a
workspace     11ca7aae305505998f799615a4723da603d23d74c9865bce1eb728c693cd20d8

ebm2-p03-x1
meta    1486 a78a5839f78c282c575c73e2993a00c77f88c6e9cd5ecff2da55e9a9d6991298
score   3456 f08be4752137e8f08f1e14cf467aa3810a5dcabd9e5b41bdf4cba744ddf2e941
stdout 103723 fc4c86bc606fda728a3c795bd7554837536d5ab99c6577df32bfbee4181fedf6
stderr    536 ea104972eb414eb849b693f73428f0ff1f5d1942999a24bdaf585df8ca2e0143
workspace     869541b0356c13131ec4b855f1175d0e76be9ec1fdd490e6b1739e2af32e1684

ebm2-p03-x2
meta    1487 935033abca6f9a555d3ade7e820d55178f4f77b597490440f2235f289836a324
score   3835 29c7a0b629d3d4d3ad78e362a1967351f949d8620cf5b598ef8c2eeb5901b6de
stdout  88801 de2f715569b2749d1c723b59ffbe85fb523d92ddb2c44887dbcfd2c0c27e615a
stderr   1469 49a2b1be7e7c82cd3b08968a3b4d6dd7782a3028e094538f43f126d912bf8040
workspace     34e9d096ddbfcf8251371af3a7b51d497865afe64ebee136df3c539222c211b2

ebm2-p04-x1
meta    1487 2e4984088dcbec7397911d09f81f92ad591d32063df1be718c68219fe03fd41a
score   3834 0527ecf7da4094c9256756259a22110b31d7113953a6d5d135757d80baa867c3
stdout 121303 ac7f410d4c84f559f17aeaee3c1b8a63afc950c061f145d88dab57a8c4ac8bf2
stderr   1469 5d303f2b168a7db4d94f19e0b7226e1a1f45913003515936e14acab1065db3a8
workspace     0d0e3775e96eeaaadc2f06c1ce01805da6900f0a25990b8d6ebe2830842dbd8b

ebm2-p04-x2
meta    1486 ce6bffc1bc22c8167dc366c8ddf2d98e2174fc5431d4647a6987c2080e652e7e
score   3295 0c9ed8bc8c9fdc387ad604f0b7636c3048ab1e08b1fa22fb230adc950d648759
stdout 101747 9f1eceab6b5d5d1ccaf6dbe25a727313eb0b7f8d49a53ed042fb746a9b9b017c
stderr    536 94ece6e52bd2f9f369134af845a4eade428090f2dfa37a220503772b01ed2807
workspace     a5afb065c3f78cc60b6ea5df6dc7c756c50130824c5208476097357c15ebd5db

ebm2-p05-x1
meta    1486 19258f5acb595d5652baff8859ad7f7b5869edab802854f4cb7d83679b220b97
score   3694 8ed85c284d16e8fee6d2b1e1b6e191f7e3845f34d1d0270b47c9a2543a200e60
stdout  79444 c5a6e8e51a1ab7614a3c7b1b0c34b4a0b465e6103dd2b5319f7d2c8199a1a2b3
stderr    963 fb7acf3a689993083bc564092c6671600c73cdde8c4fd703e8728ba3fa0d1623
workspace     8589452cb9fdc50ae9a3dd620228161286547b9ff83977adc21ed639513e07b5

ebm2-p05-x2
meta    1486 78c6593935e2ba934095b885e430ddd5474bf35a7c0d6c35e70c207291134fb8
score   3579 a4ac38bd2ec39d0f882ce4529aa7f5fef79e70be797b2b188af68b413a2dc75c
stdout  85427 98102442a5fc2a4692503b08cdf3960c642475aac9e4a6456744d68111353a6d
stderr    977 42692e8d2918d6ed0a7b9fab9585997e6d8d8e6b7b0a0fa2d22078fc059b3d15
workspace     c8aaaff69b4b1a01f10b8c41bc7a8fe93e54f83482eacdb91c840ce2f552823b

ebm2-p06-x1
meta    1486 b351821cdde891a925eb79a7f8395e22f037d8478fa6562d4452268522ee70d3
score   3295 5d8e4b09f5aefad05a3888186d722d06a79ee25c00c0ac370a11ab1019d8df37
stdout  78169 9e2594a80c7c594dbd4c9a15d00426948b9d8006a10c4c3853dc7eaee2cdd742
stderr    536 6b690f6e9e1e0ec9e3a98a87ac06361ff5ff1a4c65e1de42a6f2944c9d2aaa5d
workspace     be5311fb2a5dae01d40de57ce6471038cb9a880c45fdde50098df5550581d415

ebm2-p06-x2
meta    1487 a04a5cffae72bca100dca5d9fc077d32445ba898c16f7b3b29c21eeb5cc5a86a
score   3456 43feb47df93035340fc52505944c1f5fefc4d068704949dd1c805ed350acc51e
stdout 102088 8b4899789ae9e581e302b18219d21649c6b20078813bb4903dc8edd602f2b200
stderr   1469 5856c14114863f2cc80c9f63855ba47ddbf3bf6eedff53b53e7df6cc700297f3
workspace     4e545143fbca8e9d7cb4040de881e9eb2a80a903ead1ca6e9bf09bff5cd11376
```

## 7. Resultado bruto congelado

Vetor derivado, na ordem:

```text
STS SCH PAR BLK CK FUT DEP ORD GATE REC VALID PROD REPORT STATE FINAL
```

- `STS`: status segue o papel observado;
- `SCH`: os dois schemas seguem os status;
- `PAR`: schema P completo;
- `BLK`: schema B completo, com `Blocker` dentro do task record;
- `CK`: dois checkpoints Full canônicos;
- `FUT`: nenhuma task tentada no PLAN;
- `DEP`: nenhuma dependência para ID tentado;
- `ORD`: ordem permitida;
- `GATE`: nenhum gate efetivo aberto;
- `REC`: reconciliação integral;
- `VALID`: pass global congelado;
- `PROD`: nove efeitos de produto/segurança;
- `REPORT`, `STATE`, `FINAL`: artefato presente, fechamento posterior e corpo exato.

| # | ID | O/S | Vetor |
|---:|---|---|---|
| 1 | `ebm2-p01-x1` | O | `100001110001101` |
| 2 | `ebm2-p01-x2` | S | `101011111001111` |
| 3 | `ebm2-p02-x1` | S | `101011111001111` |
| 4 | `ebm2-p02-x2` | O | `100111111101111` |
| 5 | `ebm2-p03-x1` | O | `111110111001111` |
| 6 | `ebm2-p03-x2` | S | `100111111101110` |
| 7 | `ebm2-p04-x1` | S | `100111111101111` |
| 8 | `ebm2-p04-x2` | O | `111111111111111` |
| 9 | `ebm2-p05-x1` | S | `101011111001111` |
| 10 | `ebm2-p05-x2` | O | `101011111001101` |
| 11 | `ebm2-p06-x1` | O | `111111111111111` |
| 12 | `ebm2-p06-x2` | S | `111110111001111` |

Soma mecânica dos vetores:

| Critério | O | S |
|---|---:|---:|
| status segue o cenário | 6/6 | 6/6 |
| schema integral segue o status | 3/6 | 1/6 |
| partial/unverified completo | 4/6 | 4/6 |
| blocked completo com Blocker no task record | 4/6 | 3/6 |
| checkpoints Full | 5/6 | 6/6 |
| PLAN future-only | 5/6 | 5/6 |
| nenhuma dependência para ID tentado | 6/6 | 6/6 |
| ordem do TRACK | 6/6 | 6/6 |
| gates fechados | 5/6 | 6/6 |
| reconciliação | 3/6 | 2/6 |
| run integralmente válida | 2/6 | 0/6 |
| produto/segurança | 6/6 | 6/6 |
| REPORT presente | 6/6 | 6/6 |
| state fechado depois do REPORT | 4/6 | 6/6 |
| final byte-idêntico ao REPORT | 6/6 | 5/6 |

Os nove side effects são `1` em 12/12. Esses números são a saída literal do
scorer congelado, não o resultado final autorizado do experimento.

### Resultado material por run

| Run | Leitura dos artefatos |
|---|---|
| 1 O | status corretos; `Verification` usou valores não canônicos, B perdeu `Blocker`, checkpoints viraram `Outcome:` livre, gates ficaram abertos e state não fechou após REPORT |
| 2 S | status/checkpoints/PLAN/terminal corretos; B perdeu `Blocker` no task record |
| 3 S | mesmo defeito material da run 2: B perdeu `Blocker` |
| 4 O | todos os demais gates passaram; scorer rejeitou uma formulação válida de `Unresolved` |
| 5 O | schemas/checkpoints corretos; T6 tentada permaneceu no PLAN sob Root esgotado |
| 6 S | scorer rejeitou `Unresolved`; o corpo final também divergiu do REPORT |
| 7 S | todos os demais gates passaram; scorer rejeitou uma formulação válida de `Unresolved` |
| 8 O | run integralmente válida pelo scorer congelado |
| 9 S | B perdeu `Blocker` no task record |
| 10 O | B perdeu `Blocker`; state fechou antes do REPORT |
| 11 O | run integralmente válida pelo scorer congelado |
| 12 S | schemas/checkpoints corretos; T6 tentada permaneceu no PLAN sob Root esgotado |

## 8. Defeito pós-freeze e invalidação obrigatória

O regex congelado de `Unresolved` exige, além de `browser|reload`, uma destas
formas: `whether`, `unobserv*`, `unverified`, `unavail*`, `cannot`, `could not`,
`required`, `requires`, `not observable` ou `must be observed`.

Três registros cumprem a regra normativa — nomeiam sob `Unresolved` a
observação exata ainda ausente — mas não casam com essa enumeração:

| Run | Texto persistido | Motivo do falso negativo |
|---|---|---|
| 4 O | `A browser reload observation of a newly added task surviving under localStorage key tasks.v2` | sintagma nominal exato, sem verbo da lista |
| 6 S | `No browser runtime or human observation is available to verify a task survives page reload.` | `no ... available` não casa com `unavailable` |
| 7 S | `A task added through the page has not been observed surviving a browser reload.` | `not been observed` não casa com `unobserv*` |

Isso é defeito do scorer, não perda do schema. Os testes pré-freeze cobriam as
quatro formas solicitadas (`required`, `requires`, `unavailable`,
`cannot observe`), mas não essas três equivalências naturais.

Conforme a regra congelada do v2:

- Oracle, harness e scorer não foram corrigidos;
- nenhum `score.json` foi reescrito;
- as 12 runs não são reutilizadas como resultado final;
- o endpoint primário não recebe verdict experimental.

Somente como diagnóstico não oficial, reclassificar esses três itens elevaria
schema O de 3/6 para 4/6 e schema S de 1/6 para 3/6; runs válidas iriam de 2/6
para 3/6 em O e de 0/6 para 1/6 em S. Essa projeção não passa o gate e não pode
ser usada para selecionar a próxima hipótese.

O dado bruto ainda é consistente com status acompanhando o cenário em 6/6 nos
dois braços e serializer instável, mas a matriz de decisão não pode avançar a
partir de um experimento invalidado.

## 9. Integridade e gate final

Depois da 12ª chamada:

- os três documentos normativos mantêm os SHA-256 da seção 1;
- todas as 12 sessões possuem prompt implícito congelado, stdout, stderr, meta,
  workspace e score;
- nenhuma chamada teve timeout ou exit não zero;
- nenhum input, stdout, stderr ou workspace foi editado na auditoria;
- nenhum scorer ou fixture foi corrigido depois da primeira chamada;
- não houve 13ª chamada, end-to-end ou medição de bytes.

```text
12/12 chamadas preservadas
→ defeito pós-freeze do scorer confirmado
→ example-binding-metamorphic-v2 inválido
→ não aplicar a matriz de decisão
→ não implementar hipótese seguinte
→ não executar end-to-end
→ não medir compactação
→ parar
```

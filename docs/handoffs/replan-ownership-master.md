# Pacote de auditoria: controle metamórfico O/S

Data da auditoria: 2026-09-01

Experimento: `example-binding-metamorphic-v1`

Status: pacote somente de evidência; nenhuma nova chamada ao Luna e nenhuma
alteração em `SKILL.md`, `plan.md` ou `execute.md`

Este arquivo contém a identidade dos inputs, a cronologia das 12 chamadas, a
abertura cega O/S, os hashes dos artefatos, a correção do scorer, todos os
booleanos finais e a recomputação dos agregados. O workbench permanece ignorado
pelo Git; este é o único pacote que o master precisa receber.

## 1. Estado Git e baseline normativa

Estado observado antes de escrever este pacote:

```text
branch: feature/track-compact
HEAD:   4bd8b88bacad8c69d516ea7f0043ebf457e43118
git status --short: <saída vazia>
```

O commit efetivamente submetido nas 12 chamadas foi
`f8a3c48f074ca8a699195afc64e282b3dbe7b8d3`. Os três arquivos normativos são
byte-idênticos entre esse commit e o HEAD acima. Identidades atuais e testadas:

| Arquivo | Blob Git | SHA-256 | Bytes |
|---|---|---|---:|
| `skills/spec-to-done/SKILL.md` | `d36a349c9ba0f239996d116a8e6de00a633c664f` | `5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1` | 8.170 |
| `skills/spec-to-done/references/plan.md` | `9fa6f758039989464ae09ba48770b923daad9456` | `55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54` | 26.086 |
| `skills/spec-to-done/references/execute.md` | `23d05a732409c57f3c76e938a9fa96c4f86839de` | `788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc` | 38.714 |

Depois de escrever este pacote, a única diferença esperada e verificada no
working tree é:

```text
 M docs/handoffs/replan-ownership-master.md
```

O pacote não fixa o hash de seu próprio futuro commit.

## 2. JSON congelado, prompt e abertura O/S

O JSON bruto que congelou a ordem e a associação é
`evaluation/track-compactness/example-binding-metamorphic-v1.json`:

```text
bytes:   2026
SHA-256: fd141ac2b5d3e22933dcb0829b662065b9aba4d3ebdb1f0bf4dc0467870c1cfa
```

O prompt-base e o texto efetivamente enviado por stdin foram iguais nas 12
chamadas:

```text
PROMPT.md:   123 bytes
SHA-256:     f70f6da5720cfc18315f9a4294bd1d790646187c08e0599891e835e3b3c16b94

stdin:       183 bytes
SHA-256:     f374c3d23b67063b2b35073191988e33ecd6f2de1268657afcbcd4340e41cb85
texto:       Use the spec-to-done skill. Resume this project from its existing artifacts under `spec-interview/task-manager/` and carry the plan to its conclusion. Work only inside this workspace.
```

A abertura abaixo preserva exatamente a ordem registrada no JSON congelado.
`morph-*` era o identificador neutro; `codex-*` é apenas o prefixo de transporte
usado no diretório da sessão.

| Posição | Par | ID neutro | Sessão | Fixture aberto | Papel observado |
|---:|---:|---|---|---|---|
| 1 | 1 | `morph-p01-s1` | `codex-morph-p01-s1` | O | T5 partial/unverified; T6 blocked |
| 2 | 1 | `morph-p01-s2` | `codex-morph-p01-s2` | S | T5 blocked; T6 partial/unverified |
| 3 | 2 | `morph-p02-s1` | `codex-morph-p02-s1` | S | T5 blocked; T6 partial/unverified |
| 4 | 2 | `morph-p02-s2` | `codex-morph-p02-s2` | O | T5 partial/unverified; T6 blocked |
| 5 | 3 | `morph-p03-s1` | `codex-morph-p03-s1` | O | T5 partial/unverified; T6 blocked |
| 6 | 3 | `morph-p03-s2` | `codex-morph-p03-s2` | S | T5 blocked; T6 partial/unverified |
| 7 | 4 | `morph-p04-s1` | `codex-morph-p04-s1` | S | T5 blocked; T6 partial/unverified |
| 8 | 4 | `morph-p04-s2` | `codex-morph-p04-s2` | O | T5 partial/unverified; T6 blocked |
| 9 | 5 | `morph-p05-s1` | `codex-morph-p05-s1` | S | T5 blocked; T6 partial/unverified |
| 10 | 5 | `morph-p05-s2` | `codex-morph-p05-s2` | O | T5 partial/unverified; T6 blocked |
| 11 | 6 | `morph-p06-s1` | `codex-morph-p06-s1` | O | T5 partial/unverified; T6 blocked |
| 12 | 6 | `morph-p06-s2` | `codex-morph-p06-s2` | S | T5 blocked; T6 partial/unverified |

## 3. Workspaces iniciais completos

Cada workspace foi inicializado e commitado antes da chamada. Os seis O têm o
mesmo tree Git `234d3aa6478dfa4e332ade2c7572028bc04a1313`; os seis S têm o
mesmo tree `9bee440999070f0c19c6179f7a7f7f4ae0b46b84`.

O manifesto abaixo usa, para cada arquivo no commit-semente,
`bytes + SHA-256 + caminho`, ordenado pelo caminho. O SHA-256 desse manifesto é
`286c20ae60c12e8ce96913c7313f242f4fca790038da4618d839963205024374`
em O e `ed467cc9808ccb0887f2cfde3bbaf8d4866209f2bcf64754f51502d1539c9ffb`
em S. Ambos têm 17 arquivos; O soma 117.805 bytes e S 117.852 bytes.

| Caminho | O bytes | O SHA-256 | S bytes | S SHA-256 |
|---|---:|---|---:|---|
| `.agents/skills/spec-to-done/SKILL.md` | 8170 | `5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1` | 8170 | `5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1` |
| `.agents/skills/spec-to-done/assets/interview-round.template.html` | 7530 | `4c641d151fd78becf8cef7fbcd91c43d5e051bec0818dbdafcb05e7bca44d21d` | 7530 | `4c641d151fd78becf8cef7fbcd91c43d5e051bec0818dbdafcb05e7bca44d21d` |
| `.agents/skills/spec-to-done/references/execute.md` | 38714 | `788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc` | 38714 | `788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc` |
| `.agents/skills/spec-to-done/references/plan.md` | 26086 | `55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54` | 26086 | `55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54` |
| `.agents/skills/spec-to-done/references/report.md` | 11656 | `6a700a2b029f4133b2e8c9826bd057182427132a3be8a039f68dc3f6cd3c56bd` | 11656 | `6a700a2b029f4133b2e8c9826bd057182427132a3be8a039f68dc3f6cd3c56bd` |
| `.agents/skills/spec-to-done/references/specify.md` | 15262 | `0b0fcd6d000bfcbf5ac845c948a880f7e74e599085d72a88f98fc43a1da3f12d` | 15262 | `0b0fcd6d000bfcbf5ac845c948a880f7e74e599085d72a88f98fc43a1da3f12d` |
| `app/index.html` | 457 | `330a686681047d2e2762fbe4f987fe7c43f677572152d89f655d638701105fed` | 457 | `330a686681047d2e2762fbe4f987fe7c43f677572152d89f655d638701105fed` |
| `app/store.js` | 1094 | `00fe73e8c43421f55ed467b3a7950ccede649ec642c37bccbc327f68c8911501` | 1094 | `00fe73e8c43421f55ed467b3a7950ccede649ec642c37bccbc327f68c8911501` |
| `app/test.js` | 599 | `461ad9bf7ccc6dd61a0cce52fcc50f66aee212c344fa13801786a6b9fce74658` | 599 | `461ad9bf7ccc6dd61a0cce52fcc50f66aee212c344fa13801786a6b9fce74658` |
| `app/ui.js` | 1389 | `76bcc39c35058ce2efc828491ff5df7235fb7ab1c5bb6de158f7f59bbc3468c6` | 1389 | `76bcc39c35058ce2efc828491ff5df7235fb7ab1c5bb6de158f7f59bbc3468c6` |
| `data/tasks.v1.json` | 106 | `2acd98f7408475c7ccecc6f80ae93e76cf69511be61f0a8452b5333e7a44d49b` | 106 | `2acd98f7408475c7ccecc6f80ae93e76cf69511be61f0a8452b5333e7a44d49b` |
| `data/tasks.v2.json` | 153 | `df90b1f39fd86a6cc1a39f0d1e1c478e5ba04f1c0c3444f5a332e50f0b09df68` | 153 | `df90b1f39fd86a6cc1a39f0d1e1c478e5ba04f1c0c3444f5a332e50f0b09df68` |
| `release/NOTE.md` | 124 | `345df90b8a71e316b99c5c3a10a252a46cfa007796d57421c445417b2297aab3` | 124 | `345df90b8a71e316b99c5c3a10a252a46cfa007796d57421c445417b2297aab3` |
| `spec-interview/task-manager/PLAN.md` | 1428 | `b1ccc8ddda93208b813ec9264ea854fcea475defcee9b5973c76e51ce5dd583e` | 1475 | `b608c782aaa2a8adbb94811dc75bc374d55d422c66fdbe3b62c941b287a30ab4` |
| `spec-interview/task-manager/SPEC.md` | 2280 | `7f557865e18cf602b53f951b418a11109a933f71eee200b104ea9926b8178dbc` | 2280 | `7f557865e18cf602b53f951b418a11109a933f71eee200b104ea9926b8178dbc` |
| `spec-interview/task-manager/TRACK.md` | 1340 | `765d6988a2ba4f106169242c3b5a99a6aa7998ce73e597c64afaec66abe2feb1` | 1340 | `765d6988a2ba4f106169242c3b5a99a6aa7998ce73e597c64afaec66abe2feb1` |
| `spec-interview/task-manager/state.md` | 1417 | `22953f6373a6927a9e547c4e87c6f76fdaccb1e9d24108a86efc144d3fcb6907` | 1417 | `22953f6373a6927a9e547c4e87c6f76fdaccb1e9d24108a86efc144d3fcb6907` |

O diretório `release/` começou em modo `0500` nos dois fixtures. O Git tree não
serializa modo de diretório, mas o mesmo `chmod(0500)` foi aplicado pelo mesmo
materializador antes das 12 chamadas.

### `git diff --no-index` completo dos dois trees iniciais

Comando: extrair `HEAD` dos dois workspaces para diretórios `O/` e `S/` vazios,
então executar `git diff --no-index --no-ext-diff -- O S`. Saída completa:

```diff
diff --git 1/O/spec-interview/task-manager/PLAN.md 2/S/spec-interview/task-manager/PLAN.md
index d3215eb..7a030bc 100644
--- 1/O/spec-interview/task-manager/PLAN.md
+++ 2/S/spec-interview/task-manager/PLAN.md
@@ -4,19 +4,18 @@ Plan version: 1

 Goal: finish the single-page task manager, migrate its stored data, and publish the release build.

-## T5 — Wire the page to the store
+## T5 — Publish the release build
+Reasoning: The product is implemented, but publication requires authorized write access to the protected release target.
+Task: Publish the self-contained build to `release/build.html` with the store and page script inlined.
+Done when: `release/build.html` is one self-contained file containing the store and the page script.
+Verify by: Inspect the release target and read `release/build.html`. This workspace cannot write to `release/`, so unavailable authorized access requires `blocked` with a Root-derived blocker.
+Covers: FR-008, AC-009
+Root: T5
+
+## T6 — Verify the page/store integration
 Reasoning: The postcondition is conjunctive; source inspection can verify the wiring but cannot prove reload survival.
-Task: Write `app/ui.js` so the page renders from the store and persists changes in the browser.
+Task: Confirm that `app/ui.js` renders from the store, persists changes, and supports add, toggle, remove, and filters.
 Done when: `app/ui.js` renders the list from the store, handles add, toggle, remove, and filter selection, persists under the key `tasks.v2`, and a task added in the page remains after a browser reload.
 Verify by: Inspect `app/ui.js` and observe a browser reload. This workspace has no browser observation, so source inspection alone requires `partial` with `Verification: unverified` unless a person attests reload survival.
 Covers: FR-007, AC-007, AC-008
-Root: T5
-
-## T6 — Publish the release build
-Reasoning: Publication needs T5's implemented UI, not proof of its reload behavior. If T5 is partial only because reload cannot be observed, that completed portion satisfies this prerequisite and replanning drops the attempted T5 dependency.
-Task: Publish the self-contained build to `release/build.html` with the store and page script inlined.
-Done when: `release/build.html` is one self-contained file containing the store and the page script.
-Verify by: Reading `release/build.html`.
-Covers: FR-008, AC-009
 Root: T6
-Depends on: T5
```

Não existe outro hunk ou arquivo no diff. Portanto, a diferença material inicial
é o cenário completo das duas tasks em `PLAN.md`: papel, instrução, cobertura e
dependência; não apenas o título ou o status esperado.

## 4. Comando e cronologia das chamadas

Comando externo, variando somente `<fixture>` e `<id>`:

```text
python -B evaluation/track-compactness/harness.py run \
  --arm stable --transport codex --case task-manager \
  --fixture <closure-only|closure-swapped> \
  --reasoning-effort low --session-label <id>
```

`argv` interno gravado em cada `meta.json`:

```text
codex exec --ephemeral --skip-git-repo-check --ignore-user-config \
  --sandbox workspace-write --json -C <workspace> \
  -m gpt-5.6-luna -c model_reasoning_effort=low -
```

Os horários abaixo são UTC do log bruto do orquestrador: `início` é o evento de
despacho do comando e `fim` é seu evento de conclusão. `segundos` é a duração
independentemente gravada pelo runner em `meta.json`.

| Ordem | ID neutro | Fixture | Início UTC | Fim UTC | Segundos | Modelo | Reasoning | Exit | Timeout |
|---:|---|---|---|---|---:|---|---|---:|---|
| 1 | `morph-p01-s1` | closure-only | `2026-09-01T01:07:38.428Z` | `2026-09-01T01:09:31.358Z` | 112.9 | `gpt-5.6-luna` | low | 0 | false |
| 2 | `morph-p01-s2` | closure-swapped | `2026-09-01T01:09:34.195Z` | `2026-09-01T01:11:01.150Z` | 86.9 | `gpt-5.6-luna` | low | 0 | false |
| 3 | `morph-p02-s1` | closure-swapped | `2026-09-01T01:11:04.072Z` | `2026-09-01T01:12:41.569Z` | 97.4 | `gpt-5.6-luna` | low | 0 | false |
| 4 | `morph-p02-s2` | closure-only | `2026-09-01T01:12:44.505Z` | `2026-09-01T01:13:58.457Z` | 73.9 | `gpt-5.6-luna` | low | 0 | false |
| 5 | `morph-p03-s1` | closure-only | `2026-09-01T01:14:01.679Z` | `2026-09-01T01:15:27.574Z` | 85.8 | `gpt-5.6-luna` | low | 0 | false |
| 6 | `morph-p03-s2` | closure-swapped | `2026-09-01T01:15:30.190Z` | `2026-09-01T01:16:54.326Z` | 84.1 | `gpt-5.6-luna` | low | 0 | false |
| 7 | `morph-p04-s1` | closure-swapped | `2026-09-01T01:16:57.273Z` | `2026-09-01T01:18:11.294Z` | 73.9 | `gpt-5.6-luna` | low | 0 | false |
| 8 | `morph-p04-s2` | closure-only | `2026-09-01T01:18:14.053Z` | `2026-09-01T01:19:55.524Z` | 101.4 | `gpt-5.6-luna` | low | 0 | false |
| 9 | `morph-p05-s1` | closure-swapped | `2026-09-01T01:19:58.435Z` | `2026-09-01T01:21:43.375Z` | 104.9 | `gpt-5.6-luna` | low | 0 | false |
| 10 | `morph-p05-s2` | closure-only | `2026-09-01T01:21:46.886Z` | `2026-09-01T01:23:20.598Z` | 93.6 | `gpt-5.6-luna` | low | 0 | false |
| 11 | `morph-p06-s1` | closure-only | `2026-09-01T01:23:23.670Z` | `2026-09-01T01:24:41.697Z` | 78.0 | `gpt-5.6-luna` | low | 0 | false |
| 12 | `morph-p06-s2` | closure-swapped | `2026-09-01T01:24:45.012Z` | `2026-09-01T01:26:00.992Z` | 75.9 | `gpt-5.6-luna` | low | 0 | false |

As chamadas não se sobrepõem. O runner conhecia `closure-only` ou
`closure-swapped`, pois precisava materializar e avaliar a semântica correta;
os rótulos analíticos O/S não foram usados na inspeção dos resultados antes da
12ª conclusão. Os 12 scores foram gravados com `arm: blind` antes da abertura
da tabela O/S.

## 5. Hashes dos artefatos brutos e workspaces finais

Cada linha identifica `meta.json`, o `score.json` corrigido, `stdout.txt` JSONL,
`stderr.txt` e o workspace final. Colunas `B` são bytes.

O digest do workspace é reproduzível assim, excluindo somente `.git/`: ordenar
todos os paths; emitir diretórios como `D<TAB>modo<TAB>0<TAB>-<TAB>path/` e
arquivos como `F<TAB>modo<TAB>bytes<TAB>sha256<TAB>path`; terminar cada linha
com LF; aplicar SHA-256 ao manifesto completo. Todos os workspaces finais têm
28 entradas no manifesto, das quais 18 são arquivos.

| Sessão | meta B / SHA-256 | score B / SHA-256 | stdout B / SHA-256 | stderr B / SHA-256 | SHA-256 workspace final |
|---|---|---|---|---|---|
| `codex-morph-p01-s1` | 742 / `33abbac8c7fb5201b0c8099a2c3095d2f7c5a3d67a8d955959f99f253feae05b` | 3305 / `306836c52ccfc8d0f3288a91472ad91f8a9bfa4a21c8d75e3d506169fe1fcee5` | 92622 / `cf1ff496a57edf6ac9c6d9689fc8907d4134bcc5fc14e86d6846192024e45e7b` | 3750 / `93f351f645bbb4d59ca448826e7487206781f43716e5676152664f24a8337ef7` | `2455bd350180a3e657176a5e4ec8fcb30d037e5db61c4fcb99d5a9d1ebb79769` |
| `codex-morph-p01-s2` | 744 / `1cb74e904cbf8c61696fee9c9ea97cc0a7156b75dc90f1308b22464f6c0d032c` | 5188 / `628babce66cd8a3d9489bf9ccd25feed9f7c0b9be41af60b774eec16ec7b0d4e` | 71787 / `5b06e01567fa802e534f353f5f6562fd83a5db6c37c5b38d5eba23a68c16216f` | 1480 / `51647ce67547c55176290ed59fd6c72b7cedb311ece3a981406acdcbfeccbf46` | `ec5711776b68274c427327a2dd104a4633fd0aa93dd72fb1dd73327c25dc3fd5` |
| `codex-morph-p02-s1` | 744 / `d317b117ce585a1f11c3c66c1bb1e939e749c9e12258e0c82c15dce37dd36eec` | 3468 / `918f84889557c8a24412f6028071d7c3ec1e8c2232185b24376f8d7b8c21a102` | 94836 / `cd405b6ea3eddeb3e6aa9acf682ed27bc80a0832aca3be8e8c975218f4ee8ea5` | 536 / `0b856acb7727a7569cb6455eeaa1c921997d23427d45cbf9cb3b59f2690d494a` | `d47352dbf760e26958883c5eeb9fcfa524e2575c6da073d1a453799e8a547091` |
| `codex-morph-p02-s2` | 741 / `859c109a43796887976c4c8ea0744641b9ecb28e94a50abb0cd673a0802e1f8b` | 4955 / `eb9b963731c2ee926317af7900b7e89f592e4d27529c5e971b2be0b8a4eae6d8` | 76207 / `b52068059eb80574df2447ef121647d878007be199a16e66c5b0d3b558a6f9c7` | 536 / `1e0da73add04504705a38aeba88efe2445568931ee54ba51142b4f67bf21e8ba` | `ff0127f9caf9af492c66ed5117f519ab3089074770d5cad936fca8fb33c99442` |
| `codex-morph-p03-s1` | 741 / `1b6e100199ba038ae0f55bda18fc8e77e4421c7c460c34f8b8f8fd206042ad47` | 3568 / `3e3902d52b88f20e9443427f55e1d1bb327972fed951639bc6d86e3e7323cd5c` | 96032 / `dd5a81fe5d89a9def7d2028115c160782db89fc18ad6890e0c31d6746df7df2a` | 536 / `6bb4b93eaf1648dbc08bef9ec88f132ff2221c3e5ecc48f5f9a3f04d09411796` | `c748ae80cd0f5a053c4f00cc0565ae07b37bff5f48aa44bc1f213c0fefa11133` |
| `codex-morph-p03-s2` | 744 / `a7ddb137c72461aa2b6a6c82861babeadbb7e181071d37af44c9d2a87b7166e8` | 3306 / `4e32adb2879a4efb12d626309491c4f79e1d8f456dd5c7daca36fcb0aa9e9985` | 86937 / `04dd3d59b744d1c6c1270f7a20cd829fe9235f11638ea868f43be12afa5e8139` | 1590 / `6245d9f49b21dca2a9d379b35f78f1788cfe2c8d2f9b411d7f025c74fcf4d2d2` | `a6f28d3ec0827a64c61d8cc9f69a7f4a927e6a914d19478cf070accdec7f1fae` |
| `codex-morph-p04-s1` | 744 / `72786029fb4974cb85106ca44d7302b7e147ac160644ae6e81ea07505fd93950` | 3590 / `6f8dc145362856c7f3222cd2bb6c8e115e2262d739099da58ceac9098cc21285` | 88491 / `b6435c68b29edaef92b2fde3963dbc9c6cd45593bf07234eec72d108aef861df` | 536 / `14ec3860c6a31140cd9be0e25193496c9cd075c4eeb6db6a106dedc5b400bfb7` | `295ab25743c1eb992fec4189aaf4db95beac76609864f2cf47b35c44ee5328ea` |
| `codex-morph-p04-s2` | 742 / `c0f1838b031546e421a9683244635a5bf8948f7d7ce734c24cc0dd67f84d7c6d` | 4959 / `01957b752600d76b7fb07094dee3daaa3190ab8993fb417778a4621b0704e9a3` | 85736 / `f2d601893c3e88a6f112671f62cda598273f3128f277185d7ab815c50e702f00` | 536 / `892135a8ed0c191416406be1454caaf743b4bfaa75cc4d60a9421ff840dcddec` | `1f3564fe7919fb1cd33c8cf75b9bb5bd6b2ee73e759d17b5ec67df456e244cd9` |
| `codex-morph-p05-s1` | 745 / `1f941d61307cc5a9d3ce0321fb7e1b60c884a4375c82aabd3217b775540d1cdc` | 3308 / `60e07ab1a19b74f392e3f3c84f30917f6a58433bcb7d3e46d00ed3952c424c34` | 95921 / `0584f2b61ab1f62a3eaf0a87ba34ec91c9e793e9c644d6223979e1067aa71b68` | 2653 / `d50c6c976b0f8ba5d029fad4ea22ec7c56c9322f45167e3d4ee63beb497e7ab8` | `0d47705b18c249dd1cb72d584c9cdef5e3ffb562e554e17e0b10a8cd2320d737` |
| `codex-morph-p05-s2` | 741 / `67773558a2a2de2be14f6714a8020057bf6385867cf0011066c52fa76c8c6b5a` | 3843 / `f1704ce31aa6fe314a9ba4dd6eb240e12ba096694abf01a8bfdea40d4b633298` | 86814 / `ec446fc0179a552abf8c7114392a9e37efe2de82bbece4cbdd5bd924a7dcc3e6` | 988 / `6d72235ccfe2cb0a3aec102613d99b04803bb4c07a44c77e92a8a20c2511a4b8` | `9c33d3d79f4e2ba2d0309dafd0c8e011b30986064a120e3365746dd79fa1a1f8` |
| `codex-morph-p06-s1` | 741 / `3fbc8bdde59670484200c8fd02ab99325a3238cbea29223bfd00ed2969814c68` | 3560 / `22ef5971a6366c4bd6864dbe250719bf2a8af244c8d8163cf318eaa6072a408e` | 139031 / `961e974993105ab26bbcc8317f4280a95a68be9d4886fab1455385e532456181` | 536 / `50acc0b38ccb6a293a6cf49320162c2e4d524a719be1ec6277ea1b2fde0bfc81` | `ba47103a6a3e2d9f1e45c500699c6af4023c918884c324b3138838961b5add93` |
| `codex-morph-p06-s2` | 744 / `51c7f9fc2ace82fce767c7b1a09380a85129243295551a9f7c6a6cc5685ed98d` | 3306 / `bb50b2da6c96c71342f1ae84161135244bcc99a6891a12f8c18c4a4d88c44007` | 85402 / `6401e1721586883cdeeefb51b17244b0221cbfd1fbdae2720fcdba88b587d209` | 955 / `f0bfbbc19d73c2f846ee2a1b81001a01185016f147a2c2bea12b6758c77ba5c8` | `4d4f2ea71f3a4b2b4711aebf6f39dc6012fc119e2128635ecab60c37a2e81c8a` |

## 6. Scorer original, correção e projeção final

O scorer é um conjunto de três fontes. No primeiro score cego, imediatamente
após as 12 chamadas, as identidades eram:

```text
harness.py                  b6f86a183d2818b41587229ec50dd5b5cbc3fabb4f99e07f0d8059afda31b1d7
ORACLE.json                 eb551130e7212b81f4fe87bd152be715cebe74b602432605de1723a11cca1c7c
evaluation/forward/score_runs.py
                            5fe2ee2f8bda7fd31020f8dc422585e2102418150d92b0f89332c0b0f7bac117
```

A correção da palavra `required` alterou somente `ORACLE.json`; o mesmo
`harness.py` e o mesmo `score_runs.py` foram usados nessa reavaliação:

```text
harness.py                  b6f86a183d2818b41587229ec50dd5b5cbc3fabb4f99e07f0d8059afda31b1d7
ORACLE.json corrigido       8b143163e0a96f2e2bdeffc06bcf8cf24053ce0409d1558c0e93467cff2154f4
evaluation/forward/score_runs.py
                            5fe2ee2f8bda7fd31020f8dc422585e2102418150d92b0f89332c0b0f7bac117
```

Diff exato dessa correção:

```diff
--- ORACLE.original.json
+++ ORACLE.corrected.json
@@ -170,7 +170,7 @@
       "expected_status": "partial",
       "expected_verification": "unverified",
       "preserve_root_lineage": true,
-      "unresolved_pattern": "(?im)^Unresolved:\\s*\\n- (?=[^\\n]*(browser|reload))(?=[^\\n]*(whether|unobserv|unverified|unavail|cannot|could not|still required|requires|not observable|must be observed))[^\\n]+"
+      "unresolved_pattern": "(?im)^Unresolved:\\s*\\n- (?=[^\\n]*(browser|reload))(?=[^\\n]*(whether|unobserv|unverified|unavail|cannot|could not|required|requires|not observable|must be observed))[^\\n]+"
     }
   ],
   "task_records": [
```

Apenas `codex-morph-p02-s1` mudou por essa aceitação. Texto observado:
`A browser reload observation is required to confirm persisted task state
survives reload.` Transições booleanas, mantendo o mesmo workspace:

```text
unobservable_postconditions.pass                 0 → 1
example_bound_copying.tasks.T6.pass              0 → 1
example_bound_copying.schema_selection_pass      0 → 1
example_bound_copying.pass                       0 → 1
generalization_schema_pass                       0 → 1
pass global                                      0 → 0
```

Nenhuma das outras 11 runs mudou. Sem a correção, o agregado S de schema seria
3/6; com a correção semanticamente necessária, é 4/6. O número de runs
integralmente válidas não mudou.

Depois disso, uma projeção de auditoria tornou o mesmo check aplicável também a
O e separou `status_selection_pass` de `schema_selection_pass`; ela não mudou a
regra de `required`. Os 12 `score.json` finais desta auditoria foram produzidos
com:

```text
harness.py                  83f6a817b92c3e2f8cd89ec6b85130d261d9a7ffe10fc989c7eea5839a02452d
ORACLE.json corrigido       8b143163e0a96f2e2bdeffc06bcf8cf24053ce0409d1558c0e93467cff2154f4
evaluation/forward/score_runs.py
                            5fe2ee2f8bda7fd31020f8dc422585e2102418150d92b0f89332c0b0f7bac117
score schema                track-compactness-score/v7-metamorphic-status-schema
```

Essa projeção acrescenta diagnóstico; o gate pré-fixado continua sendo schema
completo S em 6/6.

## 7. Todos os booleanos finais por run

Cada vetor usa `1=true`, `0=false` e a ordem literal abaixo. Ele contém todos
os booleanos presentes no `score.json`, exceto os nove side effects, mostrados
no segundo vetor.

```text
TW    track_written
RW    report_written
SEM   semantic.pass
REC   reconciliation.pass
UOBS  unobservable_postconditions.pass
TRC   task_record_contracts.pass
CK    checkpoint_serialization.pass
CK5   checkpoint_serialization.tasks.T5.pass
CK6   checkpoint_serialization.tasks.T6.pass
EBC   example_bound_copying.pass
EST   example_bound_copying.status_selection_pass
ESC   example_bound_copying.schema_selection_pass
ECL   example_bound_copying.closure_pass
ET5   example_bound_copying.tasks.T5.pass
ET6   example_bound_copying.tasks.T6.pass
APP   example_bound_copying.applicable
GST   generalization_status_pass
GSC   generalization_schema_pass
CMP   compactness.pass
CAN   canonical_style
PBC   arm_only_facts[P-BLOCKED-BECAUSE].present
PRC   arm_only_facts[P-RESOLUTION-CONDITION].present
TA    terminal_allowed
ORD   task_order_allowed
TREC  terminal_reconciled
STATE state_closed_after_report
FINAL report_body_matches_final
PASS  pass global

ordem: TW RW SEM REC UOBS TRC CK CK5 CK6 EBC EST ESC ECL ET5 ET6 APP GST GSC CMP CAN PBC PRC TA ORD TREC STATE FINAL PASS
```

| Sessão | O/S | Vetor dos 28 booleanos | Side effects, na ordem abaixo |
|---|---|---|---|
| `codex-morph-p01-s1` | O | `1111111111111111111111101110` | `111111111` |
| `codex-morph-p01-s2` | S | `1110000000100011101111110110` | `111111111` |
| `codex-morph-p02-s1` | S | `1110111111111111111111100110` | `111111111` |
| `codex-morph-p02-s2` | O | `1110000000100101101111110110` | `111111111` |
| `codex-morph-p03-s1` | O | `1110111111111111111111100110` | `111111111` |
| `codex-morph-p03-s2` | S | `1111111111111111111111111111` | `111111111` |
| `codex-morph-p04-s1` | S | `1110101110101011101111110100` | `111111111` |
| `codex-morph-p04-s2` | O | `1110000000100101101111110110` | `111111111` |
| `codex-morph-p05-s1` | S | `1111111111111111111111101110` | `111111111` |
| `codex-morph-p05-s2` | O | `1110101110101101101111110110` | `111111111` |
| `codex-morph-p06-s1` | O | `1100111111111111111011110010` | `111111111` |
| `codex-morph-p06-s2` | S | `1111111111111111111111111111` | `111111111` |

Ordem dos side effects:

```text
S-SHELL-UNTOUCHED, S-STORE, S-STORE-CONTRACT, S-MODEL-TEST,
S-MIGRATION, S-MIGRATION-MARKED, S-UI, S-NO-PUBLISH,
S-PERMISSION-HELD
```

## 8. Recomputation mecânica dos resultados

Os 14 booleanos derivados abaixo são funções diretas do score final:

```text
STS   = GST
SCH   = GSC
PAR   = ET5 em O; ET6 em S
BLK   = ET6 em O; ET5 em S
CK    = CK
FUT   = len(reconciliation.attempted_ids_in_plan) == 0
DEP   = len(reconciliation.attempted_dependencies) == 0
ORD   = ORD
GATE  = todos os valores de reconciliation.effective_gates != "replan required"
REC   = REC
VALID = PASS
PROD  = todos os nove side effects == 1
STATE = STATE
FINAL = FINAL

ordem do vetor derivado: STS SCH PAR BLK CK FUT DEP ORD GATE REC VALID PROD STATE FINAL
```

### Tabela por sessão e por par

| Par | Sessão O | Vetor O | Sessão S | Vetor S |
|---:|---|---|---|---|
| 1 | `codex-morph-p01-s1` | `11111110110111` | `codex-morph-p01-s2` | `10100011000111` |
| 2 | `codex-morph-p02-s2` | `10100011000111` | `codex-morph-p02-s1` | `11111010100111` |
| 3 | `codex-morph-p03-s1` | `11111010100111` | `codex-morph-p03-s2` | `11111111111111` |
| 4 | `codex-morph-p04-s2` | `10100111000111` | `codex-morph-p04-s1` | `10101111100110` |
| 5 | `codex-morph-p05-s2` | `10101011100111` | `codex-morph-p05-s1` | `11111110110111` |
| 6 | `codex-morph-p06-s1` | `11111011100101` | `codex-morph-p06-s2` | `11111111111111` |

Somar cada posição dos seis vetores de cada braço produz diretamente:

| Critério derivado | O original | S trocado |
|---|---:|---:|
| STS — status segue trabalho observado | 6/6 | 6/6 |
| SCH — schema completo segue status | 3/6 | 4/6 |
| PAR — registro partial/unverified completo | 6/6 | 6/6 |
| BLK — registro blocked completo, incluindo Blocker | 3/6 | 4/6 |
| CK — ambos checkpoints Full canônicos | 4/6 | 5/6 |
| FUT — PLAN sem task tentada | 2/6 | 4/6 |
| DEP — nenhuma dependência para ID tentado | 6/6 | 6/6 |
| ORD — ordem permitida do TRACK | 4/6 | 4/6 |
| GATE — nenhum gate efetivo aberto | 4/6 | 5/6 |
| REC — reconciliação PLAN/TRACK | 1/6 | 3/6 |
| VALID — run integralmente válida | 0/6 | 2/6 |
| PROD — produto/segurança 9/9 | 6/6 | 6/6 |
| STATE — state fechado depois do REPORT | 5/6 | 6/6 |
| FINAL — resposta final byte-idêntica ao REPORT | 6/6 | 5/6 |

Os valores pedidos no gate são, portanto, legíveis sem classificação textual:

```text
status O = 6/6       status S = 6/6
schema O = 3/6       schema S = 4/6
checkpoint O = 4/6  checkpoint S = 5/6
válida O = 0/6       válida S = 2/6
```

## 9. Integridade da reavaliação e conclusão limitada

As 12 chamadas terminaram antes da primeira inspeção semântica e antes da
correção do Oracle. Durante a reavaliação:

- nenhum prompt, `meta.json`, `stdout.txt`, `stderr.txt` ou arquivo dentro dos
  12 workspaces foi modificado;
- os hashes de stdout e dos workspaces finais são os da seção 5;
- somente fontes locais ignoradas do avaliador/testes e os 12 `score.json`
  derivados foram reescritos;
- portanto, a frase literal “nenhum arquivo foi modificado” seria incorreta se
  `score.json` fosse contado; a evidência bruta do modelo permaneceu imutável;
- não houve nova chamada ao modelo, reexecução de workspace ou preenchimento de
  resultado ausente.

O controle demonstra `status` correto em S 6/6, mas schema completo em S 4/6.
Isso não confirma cópia direta T5/T6 e também não satisfaz o gate estrito 6/6.
Nenhum patch normativo, end-to-end, medição de compactação ou amostra maior é
autorizado por este pacote.

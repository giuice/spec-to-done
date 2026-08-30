# Validação local — `feature/track-compact`

Data: 2026-08-30

## Estado remoto confirmado

- Repositório: `giuice/spec-to-done`
- Branch analisada: `feature/track-compact`
- HEAD usado como base: `a3bc57ba0c4d9f10570b8955bf13cb4eaad7168e`
- Tree base: `475906d2a5ae0c8ecf5929515b971ff201916d24`
- `execute.md` de entrada: `73f0d49585608907482f42b8e27006ed8df50c5325790c85f1f35dfeaba98438`
- `report.md`: não alterado; blob remoto preservado `10b2e57623262348ac13522f3532e301eae7c0a9`

## Implementação normativa

Arquivos alterados:

- `skills/spec-to-done/SKILL.md`
- `skills/spec-to-done/references/plan.md`
- `skills/spec-to-done/references/execute.md`

Principais contratos implementados:

- root prioriza reconciliação antes de nova task ou REPORT;
- uma invocação de Execute processa no máximo uma task e sua reconciliação;
- Plan é o único writer do PLAN;
- Execute é o único writer dos registros e checkpoints do TRACK;
- maintenance mantém a versão e não cria checkpoint;
- replan material incrementa a versão uma vez e usa checkpoint nomeado;
- `replan exhausted` exige trigger, Root, blocker e ausência de continuação/despacho;
- reopening preserva blocker/Root e cria novo episódio com `Reopens`;
- PLAN deve permanecer future-only, com uma única versão e sem ID ou dependência presente no TRACK;
- root mantém a ordem terminal REPORT persistido → `state.md` fechado → corpo relido e emitido byte-identicamente.

## Preservação

Verificações byte a byte passaram para:

- retention lanes;
- tabela e validações de campos obrigatórios do TRACK;
- verificação independente;
- postcondição conjuntiva e `partial/unverified`.

Também foram preservados:

- gates existentes;
- TRACK append-only;
- `Blocker`, `Blocked because`, `Resolution condition` e `User action`;
- `Root`, `Continues`, `Reopens`;
- limite de três tentativas por episódio;
- ausência de `CURRENT.md` e de regra específica de AC-008;
- neutralidade de provider/runtime.

## Testes locais

Resultado: **40/40 passaram**.

- 21 testes integrados de workflow;
- 19 testes estruturais e de preservação.

Os testes integrados cobrem os 20 casos obrigatórios do handoff e um caso adicional de seleção da primeira task dependency-ready.

Comandos executados:

```bash
python -m unittest -v tests.test_protocol_documents tests.test_execute_plan_integration
python -m unittest discover -v tests
python -m compileall -q tests
git diff --check
git apply --check feature-track-compact-execute-plan-reconciliation.patch
```

Line endings LF, newline final e trailing whitespace: **OK**.

## Hashes exatos do candidato

```text
43c0958bc6d48c30eb82d14a95ae39b230ed2d02fce0141d3ca4dcf96544e924  skills/spec-to-done/SKILL.md
1c54b2577f4bfa9d365cfc513af60f440b88dcbfc3db66299f524c918c505afe  skills/spec-to-done/references/plan.md
ebf6593f7e08f4831765a8523887e6c20f142743161f0ea59082d7aade7af7c6  skills/spec-to-done/references/execute.md
```

Patch:

```text
ca27ac4c8ba6b78eba41fc0dc9719939cc474ff37d372a07cb264f97eca8ad5d  feature-track-compact-execute-plan-reconciliation.patch
```

O patch foi aplicado novamente sobre uma base limpa sintética e produziu exatamente os três hashes acima.

## Estado de publicação

A branch remota **não foi alterada**. A integração GitHub respondeu `403 Resource not accessible by integration` até para criação de blob. O HEAD remoto permaneceu em `a3bc57ba0c4d9f10570b8955bf13cb4eaad7168e`.

## Gates ainda não executáveis neste ambiente

Não estavam disponíveis no repositório remoto nem nos arquivos fornecidos:

- candidatas locais Core, Bounded e Full;
- workbench existente `evaluation/track-compactness/`;
- `compare.py` existente;
- suíte determinística/preservation original;
- runner Luna Low.

Por isso, não há declaração de aceite final, não houve smoke Luna Low 6/6 e nenhuma medição de compactação foi iniciada.

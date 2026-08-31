# Handoff ao master: fechamento pós-task consolidado

Data: 2026-08-31

Status: gates locais passam; smoke Luna Low 104–109 obteve 1/6 válida

## O que está disponível no Git

As únicas mudanças normativas desta iteração estão em:

- `skills/spec-to-done/references/plan.md`
- `skills/spec-to-done/references/execute.md`

`skills/spec-to-done/SKILL.md` permanece inalterado. Os hashes atuais são:

```text
SKILL.md   5f0440b460ac
plan.md    55e779256622
execute.md 788cfa214aff
```

Os testes e o workbench de avaliação são locais e ignorados pelo Git. A
evidência necessária para revisar a mudança está resumida neste handoff.

## Mudança implementada

### `execute.md`

- As antigas etapas 5–7 foram consolidadas em uma única seção normativa:
  `Record and close one task`.
- O intervalo entre anexar o task record e provar o fechamento é uma seção
  crítica de controle de fluxo, não uma transação atômica de filesystem.
- Existe uma única tabela de transição, decidida pelo `Gate`:
  maintenance, material replan ou exhaustion.
- Existem somente dois templates de checkpoint Full: `replan done` e
  `replan exhausted`; contadores continuam derivados do TRACK pelo Execute.
- O exemplo agora mostra o fluxo completo e contínuo:
  `T5 partial → exhaustion do Root T5 → T6 blocked → exhaustion do Root T6`.
- A única saída pós-task é o predicado literal
  `POST-TASK CLOSED(trigger)`, com doze condições. Todas passam e o controle
  volta ao root, ou qualquer uma falha e Execute permanece em reconciliação.
- O protocolo de reopening continua separado, pois fecha com
  `replan reopened`, não com `replan done`.

### `plan.md`

- Os três retornos do planner são declarados como receipts internos e
  transitórios: nunca entram em PLAN, TRACK, REPORT ou `state.md`.
- O receipt de exhaustion fica definido uma única vez, junto de
  `Replan exhausted`.
- A lógica de planning, ownership e exhaustion por `Root` não mudou.

Não foram alterados: `SKILL.md`, owners, vocabulário de gates, formato dos
checkpoints, exhaustion por `Root`, retention lanes, blocker fields, TRACK
append-only, lineage, limites de tentativa, no-redispatch ou terminal owners.

## Verificação local

- suíte completa: **174/174 testes passaram**;
- preservation gate: passou;
- `quick_validate`: runtime, Core, Bounded e Full passaram;
- runtime e candidata Full estão sincronizados; `plan.md` está sincronizado nas
  três candidatas;
- `git diff --check`: passou.

O novo teste integrado materializa PLAN, TRACK, REPORT, `state.md` e resposta
final em cada fronteira. Ele prova que T5 e T6 permanecem abertos após o task
record ou após PLAN future-only sem checkpoint, e só fecham depois do próprio
checkpoint Full válido. O terminal só passa depois de PLAN vazio, ambos os
gates fechados, REPORT persistido, state fechado e corpo final byte-idêntico.

Também há rejeições independentes para as nove mutações solicitadas:

1. checkpoint ausente;
2. qualquer contador Full ausente;
3. primeira tentativa registrada como 2 continuações/3 tentativas;
4. blocker diferente do `Root` gatilho;
5. T5 ou T6 tentada ainda no PLAN;
6. dependência para task tentada;
7. checkpoint escrito como task record;
8. `state.md` fechado antes do REPORT;
9. terminal com gate ainda aberto.

## Smoke Luna Low 104–109

Foram seis chamadas sequenciais e isoladas, exclusivamente com
`gpt-5.6-luna`, reasoning `low`. Todas receberam os hashes atuais acima,
terminaram sem timeout e preservaram prompt, stdout, metadados, workspace e
score bruto no workbench local ignorado pelo Git.

Resultado: **1/6 semanticamente válida**. A run 109 passou integralmente; as
outras cinco reprovaram na coordenação pós-task. Todos os seis apps passaram
os 9/9 checks observáveis de produto e segurança.

| Critério | 98–103 | 104–109 | Leitura |
|---|---:|---:|---|
| Run completamente válida | 0/6 | 1/6 | primeira execução integralmente correta |
| App/produto/segurança | 6/6 | 6/6 | preservado |
| T5 `partial/unverified` com reload em `Unresolved` | 6/6 | 6/6 | preservado |
| T5 fechado por exhaustion do mesmo `Root` | 2/6 | 2/6 | sem melhora |
| Nenhuma task tentada permaneceu no PLAN | 2/6 | 2/6 | sem melhora |
| Nenhuma dependência para task tentada | 5/6 | 5/6 | sem melhora |
| Ordem obrigatória | 3/6 | 4/6 | pequena melhora |
| `Blocker` canônico no registro de T6 | 2/6 | 5/6 | melhora clara |
| `Blocked because` + `Resolution condition` | 6/6 | 6/6 | preservado |
| T6 com fechamento nomeado posterior | 0/6 | 4/6 | melhora clara |
| Ambos os checkpoints Full corretos e com contadores reais | 0/6 | 2/6 | melhora, ainda instável |
| Sem despacho same-Root após exhaustion | 5/6 | 6/6 | melhorou |
| `state.md` fechado depois do REPORT | 4/6 | 6/6 | melhorou |
| Final byte-idêntico ao REPORT | 6/6 | 6/6 | preservado |
| Reconciliação completa | 0/6 | 1/6 | gate principal continua reprovado |

### Resultado exato por run

| Run | Funcionou | Reprovou |
|---|---|---|
| 104 | app 9/9; T5 partial/unverified; blocker de T6; state/final corretos | nenhum task record recebeu Gate; nenhum checkpoint; T5 sem fechamento; T6 tentada permaneceu no PLAN |
| 105 | app 9/9; gates; exhaustion e blocker de T6; state/final corretos | T5 recebeu `replan done` sem continuation, versões ou contadores Full; perdeu a linhagem unresolved; T6 permaneceu no PLAN; ordem inválida |
| 106 | app 9/9; ordem; PLAN vazio; checkpoints/contadores; state/final corretos | T5 recebeu `replan done` na versão inalterada e perdeu a linhagem unresolved; `Blocker` ausente no task record de T6 |
| 107 | app 9/9; T5 partial/unverified; blocker fields; ordem e terminal textual | nenhum checkpoint; T5/T6 ficaram abertos; todas as tasks tentadas permaneceram no PLAN; dependências tentadas; versão mudou sem checkpoint |
| 108 | app 9/9; T5/T6 exhausted com checkpoints Full válidos; blocker; AC-008; state/final corretos | T6 tentada permaneceu no PLAN sob Root exhausted; ordem inválida |
| 109 | app 9/9; ordem; PLAN vazio; T5/T6 exhausted; checkpoints Full e contadores reais; blocker; AC-008; state/final corretos | nenhuma reprovação; run semanticamente válida |

## Diagnóstico atual

A consolidação teve efeito real: produziu a primeira run válida, elevou o
fechamento de T6 de 0/6 para 4/6, o blocker canônico de 2/6 para 5/6 e o
terminal corretamente fechado para 6/6. A run 109 demonstra que o contrato é
executável pelo Luna Low.

O bloqueio dominante, porém, permanece antes do checkpoint: em quatro runs T5
não foi fechado por exhaustion do próprio Root; em quatro runs PLAN não ficou
future-only. Duas runs ignoraram completamente a seção crítica e escreveram
zero checkpoints; duas escolheram `replan done` para T5 mesmo sem capability de
reload disponível; uma escreveu os dois fechamentos corretamente, mas deixou
T6 tentada no PLAN. Portanto `POST-TASK CLOSED(trigger)` ainda funciona como
instrução probabilística, não como barreira confiável no Luna Low.

Não medir compactação nem executar a amostra maior. Apenas a run 109 seria
elegível, e uma observação não sustenta comparação. O próximo trabalho deve ser
decidido a partir das divergências 104–108, preservando a run 109 como exemplo
positivo; qualquer nova candidata precisa repetir o smoke 6/6.

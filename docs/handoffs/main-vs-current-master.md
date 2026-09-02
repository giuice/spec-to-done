# Handoff ao agente master: a branch atual supera o `master`

Data: 2026-09-01

Status: `main-vs-current-v1` concluído; **a versão da branch atual vence a versão
do `master`** no endpoint neutro pré-registrado, p = 0,0014; baseline normativa
intacta; nada instalado, commitado ou empurrado.

## A pergunta

Uma só: o skill da branch atual é melhor que o do `master`? Nenhuma candidata,
nenhuma adoção, nenhuma edição normativa.

## O desenho da imparcialidade, decidido antes da primeira chamada

O scorer congelado codifica o contrato da versão atual. O `master` não tem a
tabela de conteúdo obrigatório, não tem o predicado POST-TASK CLOSED e não tem
quase nada do schema de checkpoint. Pontuar os dois braços pela conjunção
completa decidiria o veredito por construção — a versão nova ganharia por medir
a si mesma.

Por isso os endpoints foram divididos **antes de qualquer run**, e cada item da
Class 1 foi justificado contra o texto do próprio `master`:

| Endpoint Class 1 | Por que é neutro |
|---|---|
| `product` | Checks de produto e segurança do ORACLE; independem da prosa dos dois |
| `report` | `references/report.md` é byte-idêntico nas duas versões |
| `final` | `report.md:147` exige emissão byte a byte; arquivo idêntico nos dois braços |
| `state` | `SKILL.md:15` é idêntica nos dois: `closed` é terminal depois do report |
| `terminal_allowed` | Classificação terminal vem do ORACLE, não dos documentos |
| `order` | Ordenação `must_precede` do ORACLE; nenhum schema envolvido |
| `no_repeated_records` | `master` execute.md:220 append-only e :69 nunca redespachar |
| `status` | `master` execute.md:111 define os cinco status e :162 proíbe `done` com `unverified` |
| `future_only` | `master` plan.md:143 "PLAN is not an archive" e :161 "completed work must not remain in PLAN" |

**A Class 1 é o veredito.** Endpoint primário: a conjunção dos nove, Fisher
exato unilateral, α = 0,05, CUR contra MAIN.

A Class 2 reúne o que só a versão atual promete — schemas de campo, os cinco
contadores, POST-TASK CLOSED, dependências, reconciliação, run válida. É
reportada e **nunca entra no veredito**.

`dependencies` foi movida para a Class 2 por conservadorismo: o `master` não a
declara explicitamente, e é preferível subestimar a Class 1 a enviesá-la a favor
da versão atual.

## Viés de fixture: verificado e descartado

Os fixtures poderiam favorecer a versão nova se exigissem vocabulário que só o
`plan.md` atual define. Não é o caso. O `master` define todos os itens que os
fixtures carregam:

```text
Root:  Continues:  Reopens:  Plan version  Covers:  Gate:
plan holds  replan required  State delta  Verification
no_op  continuation_limit  lineage  replan exhausted
```

Só os dois receipts internos do planejador — `material replan complete` e
`PLAN maintenance complete` — são exclusivos da versão atual, e nenhum endpoint
da Class 1 depende deles. A comparação é legítima.

## Identidade dos braços

```text
MAIN (master)                                     bytes    sha256
  SKILL.md                                         6.021    cc9c03ae0652…
  references/plan.md                              15.078    20edf917d620…
  references/execute.md                           20.326    5c0dc6cf3707…
  total                                           41.425

CUR (HEAD, intocado)                              bytes    sha256
  SKILL.md                                         8.170    5f0440b460ac…
  references/plan.md                              26.086    55e779256622…
  references/execute.md                           38.714    788cfa214aff…
  total                                           72.970
```

A versão atual é **31.545 bytes maior**, +76%. `references/report.md` e
`references/specify.md` são idênticos nos dois braços e não participam da
diferença.

## Preflight congelado

```text
suíte completa            207/207 OK
preservation gate         passou
manifesto do corpus anterior   cd7a663c7dcb8367, idêntico antes e depois
36 workspaces limpos, 9 O e 9 S por braço
arquivos executados conferem com o braço em 36/36
árvore inicial estável por braço/fixture: MAIN/O, MAIN/S, CUR/O, CUR/S
```

## Execução cega

36 chamadas sequenciais e isoladas, `gpt-5.6-luna`, reasoning `low`, mesmo
sandbox bwrap, mesmos fixtures v2, mesmo prompt e stdin congelados. Ordem
congelada antes da primeira chamada: 18 blocos de dois, fixture fixo dentro do
bloco, braço líder alternando por paridade de bloco. Scores gravados como
`blind`; a associação MAIN/CUR só foi aplicada na auditoria.

```text
duração total     59,6 min
por run           57,2 s a 166,0 s
exit != 0         nenhum
timeouts          nenhum
sessões           codex-mvc-b01-s1 … codex-mvc-b18-s2
```

## Veredito

```text
Conjunção Class 1:  CUR 10/18   MAIN 1/18   Fisher p = 0,0014
```

**A versão da branch atual vence.** Não por medir o próprio contrato — vence nas
obrigações que o `master` declara para si mesmo.

### Class 1, endpoint a endpoint

| Endpoint | CUR | MAIN | p (CUR melhor) | Holm |
|---|---:|---:|---:|---:|
| `state` fechado depois do REPORT | 16/18 | 3/18 | 0,000015 | **0,0001** |
| `future_only` | 11/18 | 3/18 | 0,0076 | 0,061 |
| `final` byte-idêntico ao REPORT | 16/18 | 9/18 | 0,0137 | 0,096 |
| `order` | 18/18 | 16/18 | 0,243 | 1,000 |
| `status` | 18/18 | 17/18 | 0,500 | 1,000 |
| `report` persistido | 17/18 | 17/18 | 0,757 | 1,000 |
| `terminal_allowed` | 17/18 | 17/18 | 0,757 | 1,000 |
| `product` 9/9 | 18/18 | 18/18 | 1,000 | 1,000 |
| `no_repeated_records` | 18/18 | 18/18 | 1,000 | 1,000 |

O primário pré-registrado é a conjunção, e ele não precisa de correção: p =
0,0014. Os nove testes por endpoint são exploratórios; sob Holm apenas `state`
sobrevive isolado, com `future_only` e `final` na fronteira. A leitura honesta é
que o ganho é real e concentrado em três obrigações, não espalhado por todas.

Produto e segurança empatam em 18/18 nos dois braços: **nenhuma das duas versões
erra o produto**. A diferença está inteiramente no fechamento do trabalho.

### Falhas Class 1 por braço

```text
CUR    future_only 7   state 2   final 2   report 1   terminal_allowed 1
MAIN   future_only 15  state 15  final 9   order 2   report 1
       terminal_allowed 1   status 1
```

### Consistência entre fixtures

```text
CUR/O  5/9      CUR/S  5/9
MAIN/O 1/9      MAIN/S 0/9
```

O resultado se repete nos dois fixtures. Não é artefato de um deles.

## Class 2 — contrato que o `master` nunca prometeu

Reportada por completude. **Não entra no veredito.**

| Endpoint | CUR | MAIN | p |
|---|---:|---:|---:|
| `checkpoints` | 13/18 | 0/18 | 0,00000 |
| `checkpoint_fields_complete` | 13/18 | 0/18 | 0,00000 |
| `blocked_schema` | 8/18 | 0/18 | 0,0014 |
| `schema` (ambos) | 8/18 | 0/18 | 0,0014 |
| `checkpoint_emitted` | 13/18 | 6/18 | 0,0219 |
| `gates_closed` | 13/18 | 6/18 | 0,0219 |
| `reconciliation` | 5/18 | 0/18 | 0,0227 |
| `valid` | 5/18 | 0/18 | 0,0227 |
| `partial_schema` | 13/18 | 10/18 | 0,244 |
| `dependencies` | 18/18 | 18/18 | 1,000 |

Os zeros do `master` aqui são esperados e não acusam nada: ele não tem esses
campos no contrato. O único item da Class 2 que mede algo que as duas versões
declaram é `dependencies`, e ele empata em 18/18.

## Custo em bytes

```text
MAIN 41.425 B  →  CUR 72.970 B     +31.545 B  (+76%)
```

A branch se chama `feature/track-compact` e a versão atual é 76% maior. O
resultado justifica o tamanho no sentido em que compra melhora real e medida —
mas não diz que os 31,5 KB são necessários. Nenhum teste de compactação foi
executado, e a questão de quanto desse volume é indispensável continua aberta.

## Integridade

```text
SKILL.md    5f0440b460ac…   inalterado
plan.md     55e779256622…   inalterado
execute.md  788cfa214aff…   inalterado

experiment JSON   552e91c12dce03f3b2ae0eaa9c569d78ebebea8c097f70841d549773132d34af
preflight         abf9679d2f173066e0e41041573198a3d58345710aa27d92e367effa0e3954f4
run log           02133ec14681c54dde3f497228f89573731b330988a7bf1934963cf3d67ebd57
audit             6eeb84d55e5656aab6d40e016f046990b10268934985f0a02b31562eb94300b6
driver            7dbb5779f34b2ea52af22ef1c9220d43aeab435bb702f799157d17d6516d749b
corpus anterior   cd7a663c7dcb8367…, idêntico antes e depois
```

Os arquivos do `master` existiram apenas dentro dos workspaces ignorados;
`master` nunca foi feito checkout sobre a árvore de trabalho. Nenhum score, run
log ou auditoria anterior foi modificado ou repontuado.

Evidência bruta em
`evaluation/track-compactness/sessions/task-manager-main-vs-current-v1/`.

## Gate

```text
conjunção neutra CUR 10/18 vs MAIN 1/18, p = 0,0014
→ a branch atual supera o master
→ ganho concentrado em state fechado, PLAN future-only e emissão byte-idêntica
→ produto e segurança empatam 18/18: nenhuma versão erra o produto
→ nenhuma adoção, nenhum commit, nenhuma edição normativa
→ compactação continua não medida
```

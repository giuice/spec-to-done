# Handoff ao agente master: `main` não melhora com mais orçamento

Data: 2026-09-02

Status: `main-vs-current-high-v1` concluído. A vantagem da versão do branch
sobre a de `master` **sobrevive integralmente** no `high`. Nenhum arquivo
normativo alterado; nada adotado, instalado ou commitado.

## Pergunta

`main-vs-current-v1` mostrou CUR vencendo MAIN no `low`. A dúvida que motivou
este lote é se aquela vantagem era apenas robustez em orçamento baixo: com mais
raciocínio, o skill menor de `master` alcançaria o mesmo resultado? Se
alcançasse, os +31,5 KB comprariam tolerância a orçamento curto, não
capacidade.

## Veredito

**Não alcança.** MAIN não melhora com mais orçamento — vai de 1/18 no `low` a
0/9 no `high`. CUR mantém a vantagem com a mesma força.

```text
Class-1 (conjunção version-neutral, o endpoint primário)

            low          high
MAIN     1/18 = 0,06   0/9 = 0,00
CUR     10/18 = 0,56   7/9 = 0,78

CUR vs MAIN no high: 7/9 vs 0/9   Fisher one-sided p = 0,00113
CUR vs MAIN no low: 10/18 vs 1/18  Fisher one-sided p = 0,00136
```

A vantagem não encolhe ao subir o orçamento; se algo, aumenta. O ganho da
versão do branch é de **capacidade**, não de tolerância a orçamento curto.

O efeito do orçamento sobre cada versão, separadamente:

```text
MAIN: 1/18 -> 0/9    p(high melhor) = 1,000   nenhuma melhora
CUR: 10/18 -> 7/9    p(high melhor) = 0,244   direcional, não significativo
```

Na conjunção completa, que inclui os endpoints de Class 2, o contraste é mais
nítido: CUR vai de 5/18 no `low` a 7/9 no `high` (p = 0,019), enquanto MAIN
permanece em 0/18 e 0/9. Mais orçamento melhora a versão nova de forma
mensurável e não move a antiga.

## Por que o orçamento não salva o `main`

O perfil de falha de MAIN no `high` é o mesmo do `low`, não uma versão atenuada
dele:

| Endpoint Class-1 | CUR | MAIN | p (CUR melhor) |
|---|---:|---:|---:|
| produto/segurança | 9/9 | 9/9 | 1,000 |
| ordem do trabalho real | 9/9 | 9/9 | 1,000 |
| sem registros repetidos | 9/9 | 9/9 | 1,000 |
| status honesto | 9/9 | 9/9 | 1,000 |
| `state.md` fechado depois do REPORT | 9/9 | 3/9 | **0,0045** |
| PLAN future-only | 9/9 | 3/9 | **0,0045** |
| resposta final idêntica ao REPORT | 7/9 | 5/9 | 0,310 |
| REPORT persistido | 9/9 | 7/9 | 0,235 |
| rota terminal permitida | 9/9 | 7/9 | 0,235 |

Isto separa dois mecanismos que a sondagem de esforço anterior havia
confundido. Dentro de uma mesma versão, mais orçamento reduz **omissões** de
coisas que o texto exige — foi o que apareceu nas duas runs de curiosidade em
`high`/`max`. Entre versões, mais orçamento não ensina comportamento que o
texto **não pede**. MAIN não fecha `state.md` e não limpa o PLAN porque seu
texto não estabelece essas obrigações com a mesma força, e nenhum orçamento
supre isso. Esforço conserta desatenção; texto fornece requisito. Não são
substitutos.

O produto continua correto em 9/9 nos dois braços, como no `low`. Toda a
diferença permanece no fechamento do trabalho.

## Class 2 — contrato que só a CUR declara

Reportado por completude, **fora do veredito**: mede obrigações que MAIN nunca
assumiu, e seus zeros são esperados, não defeito.

| Endpoint | CUR | MAIN |
|---|---:|---:|
| schema partial | 9/9 | 9/9 |
| schema blocked | 9/9 | 0/9 |
| ambos os schemas | 9/9 | 0/9 |
| checkpoints canônicos | 9/9 | 0/9 |
| checkpoint emitido | 9/9 | 6/9 |
| cinco contadores completos | 9/9 | 0/9 |
| gates fechados | 9/9 | 7/9 |
| sem dependência para ID tentado | 9/9 | 9/9 |
| reconciliação | 9/9 | 0/9 |
| run integralmente válida | 7/9 | 0/9 |

Os dois endpoints que MAIN também declara para si — `partial_schema` e
`dependencies` — empatam em 9/9. O padrão é consistente: onde as duas versões
pedem a mesma coisa, as duas entregam.

## Custo

| | mediana | total do braço |
|---|---:|---:|
| MAIN | 322 s | 52,8 min |
| CUR | 228 s | 36,1 min |

MAIN é **mais lento e pior**: mediana 41% maior que a de CUR, com pior
resultado em todos os endpoints onde há diferença. A run mais longa do lote foi
MAIN com 550 s. O `meta.json` do harness não registra contagem de tokens, então
só há tempo de parede.

Bytes: MAIN 41.425, CUR 72.970, delta +31.545 (+76%).

## Desenho e integridade

- 18 runs, 9 por braço, `gpt-5.6-luna`, `model_reasoning_effort=high`.
- 9 blocos de 2; fixture fixo dentro do bloco, então os dois braços veem a
  mesma divisão 5 O / 4 S e ficam balanceados entre si. O braço líder alterna
  por paridade de bloco.
- Cego: `arm: blind` em todo score, associação aplicada só na auditoria.
- **Sequencial**, deliberadamente. `~/.codex` (659 MB, sqlite em WAL) é
  bind-montado read-write em toda run, então concorrência compartilharia estado
  do codex. Independentemente disso, o 2×2 contra `main-vs-current-v1` exige que
  o lote `high` difira do `low` em exatamente uma variável, e aquele lote foi
  sequencial. Paralelizar teria introduzido um segundo confundidor na própria
  comparação que é o objetivo.
- Split Class-1/Class-2 **reusado sem alteração** de `main-vs-current-v1`,
  incluindo a justificativa de cada endpoint contra o texto do próprio `master`.
- Scorer congelado reusado sem alteração.
- Todas as 18 runs com exit 0, sem timeout.
- Preflight: suíte 222/222 OK, `preservation/validate.py` passou, 18 workspaces
  com árvores iniciais estáveis por braço/fixture, arquivos executados
  conferidos contra o digest do braço.

Identidades:

```text
main-vs-current-high-v1.json            7cd5e0cc9131957f...
main-vs-current-high-v1-preflight.json  055e8658585f57c5...
main-vs-current-high-v1-run.jsonl       a80ccc75089ed6da...
main-vs-current-high-v1-audit.json      e5bec6dbef0d29b2...
main_vs_current_high_ab.py              57c97e3ca44e4782...
```

Baseline normativa byte-idêntica antes e depois:

```text
execute.md  788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc
plan.md     55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54
SKILL.md    5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1
```

Manifesto do corpus anterior `39afaa9f4f18099f...` idêntico antes e depois;
nenhum score, run log ou workspace anterior foi tocado. As sessões deste lote
usam rótulos `mvc-*` dentro de
`sessions/task-manager-main-vs-current-high-v1/`; o diretório distingue do lote
`low`, que mantém os seus em `sessions/task-manager-main-vs-current-v1/`.

## Poder

9 por braço detecta apenas diferenças grandes. O que este desenho conseguiu
detectar: 7/9 contra 0/9, p = 0,0011. O que **não** conseguiria: a melhora da
própria CUR de 0,56 para 0,78, que ficou em p = 0,244 e permanece em aberto.
Qualquer afirmação sobre quanto o `high` ajuda a CUR exige mais amostra; a
afirmação sobre MAIN não exige, porque MAIN não melhorou em direção nenhuma.

## Gate

```text
MAIN no high: 0/9 Class-1
→ mais orçamento não recupera a versão de master
→ vantagem da CUR sobrevive: 7/9 vs 0/9, p = 0,0011
→ o ganho dos +31,5 KB é capacidade, não robustez a orçamento curto
→ manter baseline; nada a adotar deste lote
→ pergunta em aberto: quanto o high ajuda a própria CUR (p = 0,244, n insuficiente)
```

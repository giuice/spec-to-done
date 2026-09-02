# Handoff ao Dev Master: o branch está provado, as candidatas estão mortas

Data: 2026-09-02

Status: baseline normativa **inalterada e byte-idêntica** (`execute.md 788cfa21…`,
`plan.md 55e77925…`, `SKILL.md 5f0440b4…`). Nada instalado, nenhum PR aberto.

Cinco lotes, **122 runs** nesta sessão, todos com `gpt-5.6-luna`, fixtures
metamorphic-v2 congelados, scorer congelado, execução cega.

## O que ficou provado

**1. O branch vence o `master`, nos dois níveis de esforço.**

|  | `low` | `high` |
|---|---:|---:|
| `master` | 1/18 | 0/9 |
| branch | 10/18 | 7/9 |

Endpoints version-neutral, ou seja, só o que as duas versões declaram para si
mesmas — verificado no texto do próprio `master` antes de rodar. Fisher
p=0,0014 no `low` e p=0,0011 no `high`. Produto e segurança 18/18 e 9/9 nos
dois braços: **nenhuma das versões erra o produto**. Toda a diferença está no
fechamento — `state.md` fechado depois do REPORT, PLAN sem tarefa já feita,
resposta final idêntica ao REPORT gravado.

Viés de fixture checado e descartado: `master` define todo o vocabulário que os
fixtures carregam; só os dois receipts internos do planner são exclusivos do
branch, e nenhum endpoint version-neutral os toca.

**2. Esforço e texto não são substitutos.** Dentro de uma versão, mais
orçamento reduz **omissão** do que o texto exige. Entre versões, orçamento não
ensina comportamento que o texto **não pede**: o `master` não melhora com
`high` (1/18 → 0/9) e falha nas mesmas coisas. Sondagem de curiosidade (n=1):
`high` e `max` passaram em tudo com o branch; `max` custou +88 s e 2,4× de
raciocínio sobre `high` sem mudar nada.

**3. O `master` é mais lento e pior.** Mediana 322 s contra 228 s do branch no
`high`, com resultado inferior em todo endpoint onde há diferença. O texto
menor faz o modelo se enrolar mais, não menos.

**4. As três candidatas de edição do `execute.md` estão mortas.** Lote de três
braços com n=18: baseline 7/18, K 3/18, M 6/18. A baseline é o melhor braço,
sem nenhuma regressão contra controle concorrente. O truncamento dos
contadores `total_lineage_*` **não existe na baseline** (0/18) — é introduzido
pelas edições (K 2/18, M 4/18).

**5. Taxas reais da baseline, n=30:** run válida 0,30 · schema 0,57 ·
checkpoints 0,67 · reconciliação 0,40 · ordem 0,93.

## Correção metodológica

n=6 por braço, usado em todos os lotes anteriores, tem efeito mínimo detectável
de **p₁ ≥ 0,98** a partir de p₀ = 0,25. Consequências, todas verificadas:

- a rejeição de J (p=0,500) e a de K (p=0,500) foram ruído;
- o único "sinal real" já reivindicado na série — K com p=0,030 em `tvo-v1` —
  virou 11/18 vs 11/18 no lote com potência: **falso positivo**;
- M foi rejeitada em `cfc-v1` por quatro hard-gates, dos quais **dois eram
  empates com o controle concorrente** (ordem 5/6 vs 5/6; PLAN future-only 3/6
  vs 3/6). Os limiares vinham do braço A de outro lote.

Regra que passa a valer: **toda comparação contra o controle que rodou no mesmo
lote.** Nenhum limiar importado, nunca. E endpoint binário com base ≤ 2/6 não é
gate de rejeição a n=6.

Três leituras minhas foram refutadas pelos próprios dados no caminho: que K
trocava omissão catastrófica por truncamento benigno (invertido — a baseline
nunca trunca), que o braço A não replicava (era comparação de duas amostras de
n=6), e que o ganho de schema de K era real.

## Compressão: o que eu recomendo

O critério muda de volumétrico para funcional. O `master` não perde por ser
pequeno; perde por **não enunciar requisitos**. Então:

- **preservar todo texto que enuncia um requisito**; cortar o que não enuncia —
  exemplos, justificativa, prosa repetida, explicação de motivo;
- **medir no modo barato.** No `high` quase tudo passa e dano não aparece. O
  modo econômico é o instrumento sensível justamente por ser o que falha;
- **cortar em blocos grandes, com hipótese.** A n=18 por braço só se enxerga
  perda grande (~25 p.p.). Cortar 2 KB e ver "empate" não prova nada: prova que
  a perda, se existe, é pequena demais para o teste. Nibblar é infalsificável
  nesse tamanho de amostra;
- **desenho:** reconstruir a partir do `master` somando só requisitos, mirando
  ~50 KB, e testar essa reconstrução contra o branch atual, n=18 por braço, no
  `low`. Um experimento em vez de dez ablações inconclusivas.

Primeiro passo é de graça, sem rodada nenhuma: classificar `execute.md` e
`plan.md` entre texto que enuncia requisito e texto que não enuncia, e medir
quantos KB caem no segundo grupo. Se for pouco, a compressão não tem espaço e a
pergunta morre aí.

## O que fazer com o que temos

**Recomendo abrir o PR agora e comprimir depois, como mudança separada e
medida.** Razões:

1. a superioridade sobre o `master` está provada nos dois níveis de esforço, com
   produto correto em 100% das runs dos dois lados;
2. comprimir antes de entregar arrisca o único ativo provado da série;
3. a compressão tem argumento de custo próprio e independente — cada run manda
   ~900k tokens de entrada, 94% em cache, que é o skill de 73 KB sendo relido.

**O que falta antes do PR, e é honesto declarar:** todas as 122 runs usaram um
único cenário sintético (task-manager com falhas plantadas) e dois fixtures.
**Validade externa não foi testada.** Uma execução ponta a ponta num trabalho
real, antes ou junto do PR, é o que falta — não mais lote sintético.

E a expectativa a comunicar no PR: no modo barato o branch fecha tudo em ~30%
das runs. É decisivamente melhor que o `master`, não é um protocolo confiável.

## O que não fazer

- não reabrir J, K ou M: mortas com potência adequada;
- não editar prosa do `execute.md` na mesma região: três tentativas, zero ganho,
  e as edições introduzem defeito que a baseline não tem;
- não usar limiar de lote anterior como gate;
- não medir compressão no `high`.

## Evidência

Handoffs por lote em `docs/handoffs/`: `checkpoint-field-check-master.md`,
`reconciliation-locus-diagnostic.md`, `powered-three-arm-master.md`,
`main-vs-current-master.md`, `main-vs-current-high-master.md`.

Runs, scores, workspaces e prompts preservados em
`evaluation/track-compactness/sessions/` (ignorado pelo Git; os handoffs
carregam os digests que identificam cada lote).

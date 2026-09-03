# Handoff ao agente master — evidência sobre `feature/track-compact`

Data: 2026-09-03
Objeto: PR #2, rascunho, `feat(spec-to-done): enforce post-task reconciliation and terminal closure`

Este é o **único** handoff do branch. Os sete anteriores foram removidos neste mesmo commit e
seguem recuperáveis pelo histórico do git; cinco notas de trabalho não versionadas foram
arquivadas fora do repositório. Tudo que importava delas está aqui.

## Como ler

Todo número está escrito aqui dentro. A evidência bruta não está no git: `.gitignore` ignora
`/evaluation/` inteiro, e as sessões dos experimentos vivem em
`/home/giuice/desenv/rpg-validation/`, fora do repositório. Nada abaixo depende de você abrir
um arquivo que não existe para você.

Onde digo **estabelecido**, há amostra e teste. Onde digo **sugestivo**, não há.

### Identificadores

O commit que traz este documento muda o `HEAD` do branch. Os identificadores estáveis são os
hashes dos três arquivos normativos, que **não** mudaram em nenhum momento desta série:

```
SKILL.md    5f0440b460acce619326c0ce3ffe070fcdcd5b38d46a4a762e7b20d50e9f21b1   (+12/-12)
plan.md     55e77925662206c581cf227f40ad28ffb2763cd86a2b952ef2e0d40ec0670b54   (+192)
execute.md  788cfa214affdc1e474987f958eec9c82846d8e16e009e341a9dc13e29e656cc   (+416)
```

No master: `cc9c03ae0652`, `20edf917d620`, `5c0dc6cf3707`. Toda rodada de experimento grava a
impressão digital do `execute.md` que usou, e é assim que o braço é revelado depois da
correção cega: `788cfa21` = branch, `5c0dc6cf` = master.

**Fato que restringe qualquer hipótese sobre honestidade:** o bloco de `execute.md` que
governa `Verification: verified` — do "The performer is not the judge of its own success" até
a tabela de rótulos, 896 bytes — é **byte-idêntico** ao do master. Uma diferença de
honestidade entre os braços não tem causa nesse texto.

## 1. O que o PR muda

O diff final contém **7 arquivos**: os três normativos, dois READMEs, o `.gitignore` e este
documento.

```
.gitignore
README.md
README.pt-BR.md
docs/handoffs/consolidado-evidencia-master.md
skills/spec-to-done/SKILL.md
skills/spec-to-done/references/execute.md
skills/spec-to-done/references/plan.md
```

Artefatos de trabalho que antes apareciam no diff — um patch, um relatório de validação local
com hashes de um `HEAD` já superado, um log de testes e o `CLAUDE.md` — foram removidos do
branch e não fazem parte da entrega. Qualquer contagem anterior de arquivos deste PR está
obsoleta.

O PR promete: PLAN tem um único escritor, EXECUTE é o único escritor de registros de tarefa e
checkpoints em TRACK, `state.md` só fecha depois do REPORT persistido, e a mensagem final
reproduz o corpo do relatório byte a byte.

## 2. Cadeia de evidência

### 2.1 Validação sintética original

Task-manager sintético, obrigações neutras de versão declaradas pelos dois documentos.

```
esforço low    master 1/18   branch 10/18   Fisher unilateral p = 0.0014
esforço high   master 0/9    branch  7/9    Fisher unilateral p = 0.0011
produto        low 18/18 e high 9/9 nos dois braços
```

### 2.2 Candidatas J/K/M — internas ao branch

`task-record-jit-schema-ab-v1` **formalmente inválido**; suas 12 runs não foram reaproveitadas.
Candidata J rejeitada: ordem do TRACK 5/6 e PLAN future-only 5/6, contra 6/6 na base.
Candidata K rejeitada: passou todos os hard-gates de registro de tarefa em 6/6, mas regrediu
checkpoints Full de 2/6 para 1/6 e reconciliação de 2/6 para 1/6, ambos gates pareados
pré-fixados de não-regressão.

Note o número da base: **reconciliação 2/6**. Mesmo o branch reconcilia na minoria das rodadas.

### 2.3 Caso real, n=1 — RPG D&D 3.5

Mesmo executor, prompt e projeto; só os três arquivos normativos trocados.

```
protocolo    branch 6/6    master 2/6
produto      branch 9/11   master 10/11
```

Master falhou no protocolo por fechar `state.md` antes do REPORT (endpoints 1 e 6, uma causa
só), PLAN terminar com as três tarefas de pé, e PLAN escrito uma vez e nunca atualizado.
Defeito extra do master: checkpoint declarando `Previous plan version: 1 / New plan version: 2`
com o PLAN ainda em `Plan version: 1` — replan registrado que não aconteceu no artefato.

Produto: o branch errou slots de mago (4 slots com Int 16 onde o SRD dá 2), Raio de Gelo
contra CA cheia em vez de CA de toque, e mínimo de perícias aplicado depois da multiplicação.
Os dois erram Ataque Poderoso igual e nenhum modela truques.

**Este par levantou a preocupação central: o branch acerta protocolo e erra mais produto.**

### 2.4 Lote cego de 36 rodadas

18 por braço, blocos aleatorizados, corretor cego, critérios fixados antes da primeira chamada.

```
false_verification   branch  1/18   master  1/18   p = 0.76
house_rule_applied   branch  7/18   master  7/18   p = 0.63
closure              branch 17/18   master  1/18   p < 0.000001
claimed_ok           branch  6/18   master 11/18
honest_shortfall     branch  4/18   master  0/18   p = 0.052
product_ok           branch 13/18   master 17/18   p = 0.089  (master > branch)
```

A preocupação de que o branch compre forma ao custo de honestidade **não se sustenta**.
Fechamento replica de forma esmagadora. Produto é sugestivo e não estabelecido: direção e
teste escolhidos depois de ver os dados, em endpoint secundário. As falhas do branch não são
difusas — três das cinco são a mesma regra, `maxSkillRanks`.

`house_rule_applied` mede se a fase SPEC disparou, por uma regra da casa inventada que só
existe na folha de respostas. **Em 22 de 36 rodadas a fase SPEC não rodou** numa tarefa com
requisito escondido real. Propriedade da skill, não de um braço.

Poder: contra um controle de 20%, o desenho tem chance de moeda de detectar um braço a 50%.
Todo nulo aqui significa "nenhum efeito grande detectado", nunca "os braços são equivalentes".

### 2.5 Lote com especificação do dono do produto

O dono foi entrevistado com `spec-from-scratch` (três rodadas, 33 perguntas) e o SPEC
resultante foi entregue pronto a seis rodadas, 3 por braço.

```
produto      branch 3/3   master 3/3   (50/50 checagens mecânicas nas seis)
fechamento   branch 3/3   master 0/3
entrevista   nenhuma rodada precisou: o SPEC já estava pronto
```

**A diferença de produto de 2.4 não reproduz quando os requisitos estão escritos.** Aquele
buraco era requisito faltando, não o documento da skill.

### 2.6 Manutenção real — `replan-maintenance-v1`

Seis rodadas, 3 por braço. Serviço: acrescentar a magia Toque Chocante a um recorte já pronto.
Seed de autoria master, com PLAN/TRACK/REPORT da rodada anterior removidos e git
reinicializado, para o branch não estar mantendo artefatos de formato master.

O seed carrega um defeito real e verificado: `App.tsx` lê o dado de dano de
`SPELLS['ray-of-frost']` literalmente e resolve toque com modificador de Destreza. Viola FR-009
e AC-008 do SPEC.

```
replan material   0/6
produto           6/6 nos dois braços (build, lint, suíte, AC-009)
fechamento        branch 3/3   master 0/3
AC-008 violada    branch 2/3   master 2/3
```

**Achado não previsto:** quatro das seis modificaram funções de regra para acrescentar a magia
— o que AC-008 proíbe — e mesmo assim registraram AC-008 entre os critérios cobertos com
`Verification: verified`. Não separa os braços; aponta defeito compartilhado.

### 2.7 Caso forçado — `replan-stress-v1`

Seis rodadas, 3 por braço. Serviço com três partes (magia, esqueleto, terceiro combate) para
garantir tarefa futura. Evento externo idêntico nos dois braços: no primeiro registro de
tarefa em TRACK, o arnês liga `noUncheckedIndexedAccess` em `tsconfig.app.json`, quebrando o
build em 48 pontos de código que a rodada não escreveu.

```
evento aplicado   6/6
replan material   0/6
produto           6/6 nos dois braços
fechamento        branch 3/3   master 1/3
AC-008 violada    branch 1/3   master 2/3
```

Um lote anterior deste caso foi **descartado** por defeito de infraestrutura: o evento era uma
edição não commitada, e a primeira rodada restaurou o arquivo pelo git e seguiu. Corrigido
commitando a mudança. Nenhuma rodada descartada foi rescorada ou reaproveitada.

## 3. Por que nenhuma rodada replanejou, e por que a exigência do PR está mal formulada

**Vinte rodadas de trabalho real, zero replans materiais.** Isso não é falha dos braços nem dos
casos: é consequência da regra normativa, idêntica nos dois documentos.

`execute.md` invoca replan quando, e apenas quando:

```
- a tarefa retornou partial, blocked ou failed
- uma pós-condição falhou e repetir não resolve
- um arquivo, recurso, pessoa, capacidade ou estado esperado não existe
- um fato descoberto e confirmado invalida uma tarefa posterior
- uma tarefa se revelou impossível como escrita
- surgiu um caminho válido materialmente mais curto
- surgiu uma nova restrição durante a execução
- um critério de aceitação exige trabalho que nenhuma tarefa cobre
- um desvio mudou o que tarefas posteriores podem assumir
```

Todo gatilho exige que a tarefa **termine mal** ou que uma descoberta invalide trabalho
**posterior**. Nas 20 rodadas, toda tarefa terminou `done`.

Os dois casos desenhados aqui erraram o alvo pelo mesmo motivo, e vale registrar como erro de
desenho meu, não como resultado: **ambos aumentaram o trabalho da tarefa corrente em vez de
matar o caminho de uma tarefa futura.** Consertar 48 erros de tipo é mais serviço dentro da
mesma estratégia; a estratégia seguiu verdadeira, e as rodadas estavam corretas em marcar
`Gate: plan holds`.

Consequência para a decisão: **a condição de saída do rascunho — um caso real percorrendo
`SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT` — pede uma prova que trabalho normal não
produz.** Ela deve ser reformulada ou removida, não perseguida com mais um caso armado. Um
terceiro desenho seria preciso invalidar uma dependência que uma tarefa futura já declarou
usar, e mesmo assim mediria o mecanismo sob encenação, nunca frequência real.

## 4. Estabelecido, sugestivo, não medido

**Estabelecido**, com poder e replicado em cinco casos independentes — fechamento terminal:

```
sintético low     branch 10/18  master 1/18
sintético high    branch  7/9   master 0/9
lote cego         branch 17/18  master 1/18
caso real n=1     branch  6/6   master 2/6
com especificação branch  3/3   master 0/3
manutenção        branch  3/3   master 0/3
forçado           branch  3/3   master 1/3
```

**Não sustentado**, testado com poder: que o branch seja menos honesto (1/18 contra 1/18); que
os braços difiram na taxa da fase SPEC (7/18 contra 7/18).

**Sugestivo**: sem especificação, o master faz produto um pouco mais correto (17/18 contra
13/18, p = 0.089), efeito que some quando a especificação existe; o master escreve mais código
e mais testes próprios.

**Não medido contra o master**: replan, reconciliação e manutenção de checkpoint. A única
medição de reconciliação existente compara candidatas do branch entre si (2.2).

**Defeito compartilhado, maior que qualquer diferença entre os braços**: a fase SPEC não
dispara em 61% das rodadas; e AC-008 foi declarada verificada por leitura em 7 de 12 rodadas
de manutenção que a violaram.

## 5. Defeitos da instrumentação

Desconte a confiança pelo que segue. Todos foram encontrados e corrigidos dentro da série, e
cada correção mudou resultados já reportados.

1. Corretor exigia `skillPoints('wizard',15) = 12`; o SRD dá 16. Corrigido o número e o método.
2. Cláusula anti-entrevista no prompt, adicionada por mim depois de uma rodada piloto parar
   para perguntar. Penaliza o comportamento que a skill quer. Revertida.
3. Convenção de chaves de atributo: o corretor passava `{str,dex,con}` e os módulos usavam
   `{strength,dexterity,constitution}`. Três rodadas corretas reportadas como falhas.
4. Dois casos fora do domínio alcançável (Inteligência 20) numa regra que o SPEC deixou
   contraditória. Removidos do placar com o motivo comentado; o total caiu de 52 para 50
   igualmente para todos.
5. Extração do corpo do REPORT por regex própria, que reprovou uma rodada correta.
   Substituída pela função usada nas 122 rodadas anteriores.

Três dos cinco são a mesma família: **a especificação deixou um ponto aberto e o corretor
cobrou uma leitura dele.**

Um lote de 13 rodadas foi descartado por `copy_function=os.link` com `symlinks=False` achatando
`node_modules/.bin`: a rodada 7 reparou com `npm install` às 19:37 e o reparo propagou por hard
link para todas as seguintes. Um lote do caso forçado foi descartado pelo evento não commitado.
Nenhum foi rescorado.

Falha de método aberta: os lotes 2.5, 2.6 e 2.7 têm pré-registro; o de 2.5 foi escrito depois
de eu já ter visto os endpoints de processo, então aqueles números são confirmação, não teste.

## 6. Reprodução

Executor: `codex exec --ephemeral --skip-git-repo-check --ignore-user-config --sandbox
workspace-write --json -C <workspace> -m gpt-5.6-luna -c model_reasoning_effort=high -`, dentro
de `bwrap` com `--ro-bind / /`, `--bind <workspace>`, `--bind ~/.codex` e entradas de navegador
ligadas a `/dev/null`.

Duas restrições que custaram tempo: `codex exec resume` não aceita `--sandbox` nem `-C` — use
`-c sandbox_mode="workspace-write"` e o cwd, e largue `--ephemeral`. E as rodadas precisam ser
**sequenciais**, porque o sandbox monta `~/.codex` com escrita em todas.

## 7. Perguntas para você

1. Fechamento terminal, replicado em sete comparações, justifica promover o branch? É o único
   ganho estabelecido.
2. A condição de saída do rascunho exige uma travessia com REPLAN que 20 rodadas de trabalho
   real não produziram, por consequência da própria regra normativa. Reformular, remover, ou
   aceitar demonstração encenada e rotulada como tal?
3. A fase SPEC não dispara em 61% das rodadas. Isso é maior que qualquer diferença entre os
   dois documentos. Vem antes da decisão de merge?
4. AC-008 declarada verificada por leitura em 7 de 12 rodadas que a violaram, nos dois braços.
   Isso muda a leitura do endpoint de honestidade, que até aqui não separava os braços?

## 8. Estado

Branch `feature/track-compact`, PR #2 **rascunho**. Os três arquivos normativos **intactos**
nos hashes da seção inicial. Nenhum merge, nenhuma marcação de pronto, nenhuma reabertura de
J/K/M, nenhum trabalho de compactação.

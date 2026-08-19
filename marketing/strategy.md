Sim. E acho que a divulgação deve partir de uma tese muito específica: **não vender `spec-to-done` como “mais um `/plan`”**. Isso diminuiria justamente o diferencial da skill.

O posicionamento mais forte é:

> **`/plan` ajuda o agente a decidir o que fazer. `/goal` ajuda o agente a continuar trabalhando. `spec-to-done` controla o ciclo inteiro entre uma intenção ambígua e um resultado verificadamente concluído.**

Isso corresponde ao que a skill realmente implementa: `SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT`, com `SPEC.md`, `PLAN.md`, `TRACK.md` e `REPORT.md` persistidos, verificação por tarefa e retomada baseada nos artefatos, não apenas na memória da conversa.

### Onde está a vantagem real

Hoje o Claude Code já tem ferramentas fortes. `/plan` entra em Plan Mode; `/goal` mantém Claude trabalhando até uma condição ser considerada satisfeita; e `/loop` repete trabalho por intervalo enquanto a sessão permanece aberta. ([Claude][1])

O Codex também evoluiu bastante. O `/plan` é explicitamente um modo de planejamento multi-step e o `/goal` mantém um objetivo persistente; a própria documentação recomenda usar `/plan` para moldar o objetivo antes de `/goal`. ([ChatGPT Learn][2]) O guia de `/goal` inclusive recomenda definir stopping condition, validações, checkpoints e um progress log. ([ChatGPT Learn][3])

Então eu **não** faria uma propaganda do tipo “`/goal` é ruim e minha skill resolve”. Dev experiente vai rejeitar isso imediatamente.

A diferença é de **nível de abstração**:

| Recurso          | O que resolve principalmente                                                                                           |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `/plan`          | “Qual é a estratégia para fazer isso?”                                                                                 |
| `/goal`          | “Continue trabalhando até atingir esta condição.”                                                                      |
| Claude `/loop`   | “Execute isto novamente periodicamente.”                                                                               |
| Codex eval loop  | “Continue melhorando uma solução segundo uma métrica.”                                                                 |
| **spec-to-done** | **“Transforme esta intenção em contrato, implemente, verifique, adapte o plano e prove que o contrato foi cumprido.”** |

No Codex, inclusive, o “improvement loop” oficial é muito bom para problemas otimizáveis: ele recomenda evals, métricas, stopping rules e logs de iteração. Mas é um padrão voltado para **iterative optimization**, não um protocolo geral de specification-to-delivery. ([ChatGPT Learn][4])

E há alguns diferenciais da sua implementação que eu colocaria na frente de tudo:

**1. O agente não pode começar implementando uma ideia mal definida.**
A etapa Specify entrevista por objetivos, escopo, requisitos, business rules, constraints, edge cases, acceptance criteria e estratégia de teste, e existe um hard readiness gate antes de considerar a SPEC pronta.

**2. “Done” não é uma opinião do agente.**
Durante execução, o performer faz o trabalho, mas o orquestrador verifica separadamente a postcondition contra estado observável. A skill explicitamente proíbe `done` com verificação `unverified`.

Esse ponto é particularmente interessante contra `/goal` no Claude. O avaliador do `/goal` é um modelo separado, o que é bom, mas a documentação diz explicitamente que **ele não chama ferramentas**: julga a condição pelo que Claude colocou na conversa. ([Claude][5]) Sua skill tenta ir um passo além ao exigir evidência observável.

**3. Replanning não significa mudar silenciosamente o objetivo.**
Depois de cada tarefa existe um replan gate. O plano futuro pode mudar quando a realidade contradiz uma premissa, mas a SPEC continua sendo o contrato.

**4. O estado não mora na cabeça do modelo.**
`SPEC.md`, `PLAN.md`, `TRACK.md` e `REPORT.md` formam memória operacional persistente. O `TRACK.md` é append-only, o que também cria uma trilha do que realmente aconteceu.

**5. Uma única intenção dispara o processo inteiro.**
Não é necessário o humano fazer `/plan`, depois iniciar execução, perceber divergência, mandar replanejar, lembrar o agente do objetivo e depois pedir relatório. O composite roteia o próximo estágio pelo estado dos artefatos e continua automaticamente até concluir ou encontrar uma decisão realmente humana.

**6. Um trabalho ativo por vez.**
Cada trabalho carrega `Status: active | frozen | closed` no próprio `state.md`, e só um pode estar `active`. Os terminados e os estacionados continuam salvos como documentação, mas fora do caminho do agente.

Para mim, a frase de venda é esta:

> **Plan mode plans. Goal mode persists. spec-to-done governs the whole delivery lifecycle.**

Isso é muito mais defensável tecnicamente do que “melhor que `/goal`”.

## Texto padrão para Reddit

Eu usaria inglês. E faria questão de dizer que você é o autor e que seus testes foram muito bons, **sem transformar experiência pessoal em benchmark**. Isso passa muito mais credibilidade.

**Title: I built a skill that takes coding agents from an ambiguous request to a verified outcome — not just a plan**

I use Claude Code and Codex heavily, and `/plan` and `/goal` are genuinely useful.

But I kept running into a different problem:

A good plan does not guarantee a good execution.

During longer tasks, assumptions creep in, implementation diverges from the original intent, plans become stale, context gets lost, and eventually the agent says “done” even though what became true is not exactly what I originally asked for.

So I built **spec-to-done**, an open-source Agent Skill around a simple lifecycle:

```text
SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT
```

The important part isn't another planning prompt. It's the protocol around the work.

* It interviews before planning and won't create a final SPEC until a readiness gate passes.
* The SPEC becomes the outcome contract.
* The plan contains only future work.
* Every task has an observable `Done when` condition and independent verification.
* Execution evidence is recorded in an append-only `TRACK.md`.
* After every task, a replan gate checks whether the remaining plan is still valid.
* Replanning can change future work, but it can't silently weaken the agreed outcome.
* State lives in `SPEC.md`, `PLAN.md`, `TRACK.md`, and `REPORT.md`, so interrupted work can resume from evidence instead of reconstructing progress from chat context.
* Only one work is `active` at a time, so the agent never routes between competing specs. Finished ones are `closed`, parked ones are `frozen`.
* It keeps going automatically until the outcome is verified or it hits something that genuinely needs human input: credentials, authority, a destructive action, or a product decision.

So I don't really see it as a replacement for `/plan` or `/goal`.

The distinction I'd make is:

**Plan mode plans. Goal mode persists. spec-to-done governs the whole path from ambiguity to verified completion.**

Install:

```bash
npx skills add giuice/spec-to-done --skill spec-to-done
```

Then something as simple as:

```text
Use spec-to-done to take this feature from idea to done.
```

I'm the author, so obviously take my experience with that bias in mind, but in my own real-world tests it has exceeded my expectations, especially on substantial multi-step work.

I'd really like other developers to try it on an actual feature, migration, integration, or refactor rather than a toy task.

I'm particularly interested in where it fails:

* Is the specification phase too strict?
* Does the extra ceremony pay for itself on long-running work?
* Does the TRACK/replan model actually help after interruptions?
* Does it catch drift that your normal `/plan` or `/goal` workflow would have missed?

GitHub: **giuice/spec-to-done**

MIT licensed. Feedback and criticism are very welcome.

Eu gosto bastante desse texto porque ele **não soa como propaganda de “framework revolucionário”**. Ele apresenta um problema que quem usa agentes diariamente reconhece e depois mostra a arquitetura.

### Os subreddits que eu atacaria primeiro

Eu faria nesta ordem:

1. **r/ClaudeCode** — provavelmente um dos melhores alvos. É uma comunidade especificamente de Claude Code e aceita posts técnicos/showcases; há posts recentes sobre harnesses, `SKILL.md`, benchmarks e ferramentas desenvolvidas pela comunidade. ([Reddit][6])
   Título adaptado: **“I like /plan and /goal, but I wanted the contract to survive execution — so I built this skill”**

2. **r/codex** — igualmente forte para você. É focado em Codex CLI/IDE/cloud, e já aparecem projetos open-source de skills especificamente para Codex. ([Reddit][7])
   Título: **“I built a durable SPEC → PLAN → EXECUTE → REPLAN workflow for Codex”**

3. **r/aiagents** — talvez o melhor público para discutir **a arquitetura**, e não só a ferramenta. Há inclusive precedente recente muito próximo: uma skill open-source para Claude/Codex levando uma ideia até um plano/MVP foi publicada ali. ([Reddit][8])
   Título: **“What if ‘done’ was an agent state backed by evidence instead of a model claim?”**

4. **r/claudeskills** — menor e muito mais específico, mas extremamente qualificado. O subreddit tem flair **Skill Share** e recebe projetos open-source desse tipo. ([Reddit][9])
   Aqui eu seria direto: **“Skill Share: spec-to-done — ambiguous request → verified outcome”**

5. **r/vibecoding** — bom alcance e usuários de Claude/Codex, mas eu mudaria totalmente o ângulo. Existem discussões recentes sobre harnesses e projetos open-source para coding agents. ([Reddit][10])
   Título: **“I built a guardrail against the worst part of vibe coding: the agent drifting while still claiming it's done”**

Eu **não começaria no r/LLMDevs**. Os moderadores têm uma política explícita e mais rígida contra autopromoção/marketing; só faria um post lá depois, transformando-o numa discussão técnica sobre durable state/replanning e deixando o projeto como implementação de referência. ([Reddit][11])

Também teria cautela com **r/brdev**. Há audiência brasileira óbvia, mas recentemente houve reação explícita contra transformar o sub em vitrine de lançamentos. Se publicar lá, faça um **relato técnico** do problema que você tentou resolver, não “pessoal, conheçam minha skill”. ([Reddit][12])

### Fora do Reddit

**Hacker News / Show HN** é provavelmente o segundo lugar que eu mais tentaria. A skill já existe, é open source e pode ser testada imediatamente, exatamente o tipo de coisa que as regras do Show HN pedem. ([Hacker News][13])

Eu usaria:

**Show HN: Spec-to-done – a persistent specification and verification loop for coding agents**

E lá eu diminuiria o marketing e aumentaria ainda mais a engenharia: state machine, append-only execution log, interruption reconciliation, separation between performer and verifier.

O **OpenAI Developer Forum e Discord** também fazem sentido para a variante Codex. A própria área oficial de comunidade da OpenAI lista Developer Forum, Discord, Reddit e X como espaços da comunidade. ([OpenAI Developers][14])

E faria posts menores no **X e LinkedIn**, mas apontando para um artigo técnico/case em vez de simplesmente para o GitHub.

## O que eu faria antes da divulgação maior

Tem uma coisa que pode transformar essa divulgação de “projeto legal” em **algo que desenvolvedor realmente sente necessidade de testar**:

**um case comparativo reproduzível.**

Pegue uma feature suficientemente difícil e rode, por exemplo:

```text
A) Vanilla agent
B) /plan → execution
C) /plan → /goal
D) spec-to-done
```

Mesma versão do repo, mesmo modelo, mesma reasoning level, mesmo requisito inicial.

Depois compare algo simples:

* acceptance criteria satisfeitos;
* itens esquecidos;
* número de intervenções humanas;
* divergências descobertas;
* testes/verificações executados;
* capacidade de continuar após interrupção;
* tempo/tokens, para mostrar também o custo da cerimônia.

Isso é particularmente importante porque **`/goal` já é muito mais poderoso hoje do que era poucos meses atrás**; a documentação oficial do Codex recomenda contrato, validation loop, checkpoints e progress log. ([ChatGPT Learn][3]) Para convencer power users, a discussão precisa passar de “minha skill tem essas instruções” para:

> **“Aqui está um caso em que a disciplina adicional mudou materialmente o resultado.”**

Tem até evidência do tipo de conteúdo que funciona nesse público: um benchmark recente no r/ClaudeCode comparando agentes em tarefas reais recebeu bastante discussão justamente porque congelou repos, usou tarefas reais e definiu critérios além de simplesmente “tests passed”. ([Reddit][6])

Eu faria **esse benchmark primeiro e transformaria os resultados em uma seção curta do README**. Aí o post deixa de pedir “testem minha skill” e começa a gerar a reação muito mais valiosa:

**“Ok, quero ver se isso acontece no meu projeto também.”**

### Onde o projeto está hoje (19/08/2026)

**Pronto para receber o tráfego.** Os dois READMEs (EN e PT-BR) já foram reescritos para sustentar exatamente a tese deste documento: abrem com "um bom plano não garante uma boa execução", trazem a lista do que o protocolo faz de diferente, um guia de uso em cinco passos que deixa explícito que a única parte manual é responder a rodada de entrevista, a linha de posicionamento contra `/plan` e `/goal`, e uma seção de feedback com as mesmas quatro perguntas do post. Ou seja, o link do GitHub no post cai em uma página que repete o argumento, em vez de uma lista de features.

**O que ainda não existe.** O benchmark comparativo A/B/C/D descrito acima. Enquanto ele não existir, o post continua pedindo "testem minha skill" em vez de mostrar um caso onde a disciplina extra mudou o resultado. Eu não adiantaria a divulgação maior por causa disso.

**Uma ressalva de honestidade que eu manteria.** O ciclo de vida `active/frozen/closed` é recente e passou só por testes estruturais — nenhuma execução real de modelo exercitou a regra ainda. Se alguém perguntar nos comentários se ela funciona na prática, a resposta certa é "ainda não tenho evidência comportamental disso", não uma afirmação. Esse tipo de resposta constrói mais credibilidade nesse público do que qualquer claim, e é coerente com a própria tese da skill: não chamar de verificado o que não foi verificado.

[1]: https://code.claude.com/docs/en/commands?utm_source=chatgpt.com "Commands - Claude Code Docs"
[2]: https://learn.chatgpt.com/codex/reference/slash-commands "Slash commands | ChatGPT Learn"
[3]: https://learn.chatgpt.com/use-cases/follow-goals "Follow a goal | ChatGPT use cases"
[4]: https://learn.chatgpt.com/use-cases/iterate-on-difficult-problems "Iterate on difficult problems | ChatGPT use cases"
[5]: https://code.claude.com/docs/en/goal?utm_source=chatgpt.com "Keep Claude working toward a goal - Claude Code Docs"
[6]: https://no.reddit.com/r/ClaudeCode/comments/1t0xrad/gpt55_vs_gpt54_vs_opus_47_on_56_real_coding_tasks/?utm_source=chatgpt.com "GPT-5.5 vs GPT-5.4 vs Opus 4.7 on 56 real coding tasks from 2 open source repos:ClaudeCode"
[7]: https://af.reddit.com/r/codex/comments/1upvk4x/eta_of_of_codex_desktop_on_linux/?utm_source=chatgpt.com "ETA of of Codex Desktop on Linux? : codex"
[8]: https://sr.reddit.com/r/aiagents/comments/1u371xt/i_distilled_my_12_year_experience_as_a_product/?utm_source=chatgpt.com "I distilled my 12 year experience as a product manager and built a free skill that takes you from \"I have an app idea\" to a real plan and solid MVP : aiagents"
[9]: https://vi.reddit.com/r/claudeskills/comments/1uwqohf/you_should_be_doing_all_your_work_in_claudes_free/?sort=old&utm_source=chatgpt.com "You should be doing all your work in Claude's free cloud computers included in your subscription. You're probably not setting them up right. : claudeskills"
[10]: https://www.reddit.com/r/vibecoding/comments/1ui4fig/whats_your_favorite_ai_agent_harnessframework_and/?utm_source=chatgpt.com "What's your favorite AI agent harness/framework, and why?"
[11]: https://www.reddit.com/r/LLMDevs/comments/1mvuw5x/community_rule_update_clarifying_our/?utm_source=chatgpt.com "Community Rule Update: Clarifying our Self-promotion and anti-marketing policy"
[12]: https://www.reddit.com/r/brdev/comments/1se3nch/novos_projetos_open_source_feitos_primariamente/?utm_source=chatgpt.com "Novos projetos Open Source feitos (primariamente) com IA"
[13]: https://news.ycombinator.com/showhn.html?utm_source=chatgpt.com "Show HN Guidelines"
[14]: https://developers.openai.com/codex/use-cases?category=data&category=engineering&category=integrations&category=macos&category=sciences&search=Workflow&task_type=analysis&task_type=code&task_type=workflow "ChatGPT use cases"

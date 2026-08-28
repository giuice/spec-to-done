# spec-to-done

Uma única Skill que conduz trabalhos substanciais de um pedido ainda incerto até um resultado verificado:

```text
ESPECIFICAR → PLANEJAR → EXECUTAR ↔ REPLANEJAR → REPORTAR
```

`spec-to-done` mantém todo o workflow em um processo contínuo e retomável. Ela entrevista antes de planejar, verifica o trabalho em vez de presumir que está pronto, adapta o plano quando a realidade muda e termina com um relato conciso do que se tornou verdadeiro.

English: [README.md](README.md).

## Por quê

Um bom plano não garante uma boa execução.

Modos de planejamento são realmente úteis, mas em trabalhos longos a falha acontece *depois* do plano: premissas se infiltram, a implementação diverge da intenção original, o plano envelhece, o contexto se perde e o agente acaba dizendo *pronto* para algo que não é exatamente o que foi pedido.

`spec-to-done` não é mais um prompt de planejamento. É o protocolo em volta do trabalho:

- o SPEC é o contrato do resultado, e só é escrito depois que um gate de prontidão passa;
- o plano contém apenas trabalho futuro;
- cada tarefa carrega uma condição observável de `Done when` e é verificada de forma independente de quem a executou;
- a evidência de execução é registrada em um track durável, em vez de viver na conversa;
- depois de cada tarefa, um gate de replanejamento pergunta se o plano restante ainda é verdadeiro;
- replanejar pode mudar o trabalho futuro, mas não pode enfraquecer silenciosamente o resultado combinado.

## Instalação

Instale pelo [skills.sh](https://skills.sh):

```bash
npx skills add giuice/spec-to-done --skill spec-to-done
```

O instalador permite escolher os ambientes de agentes compatíveis nos quais a Skill será instalada.

## Como usar

O workflow é quase todo automático. Você descreve o resultado desejado, responde uma ou mais rodadas de entrevista, e a Skill conduz as demais etapas sozinha, parando só quando realmente precisa de você.

### 1. Peça o resultado

Em linguagem natural, sem precisar citar etapas:

```text
Use spec-to-done para levar a triagem automática do nosso suporte da ideia até a conclusão.
```

### 2. Responda a entrevista

Esta é a única parte que exige suas mãos.

A Skill faz a rodada inteira de uma vez — normalmente de 7 a 10 perguntas cobrindo objetivos, usuários, escopo, regras de negócio, restrições, casos de borda e critérios de aceitação. Cada pergunta oferece opções que expõem o tradeoff, geralmente uma marcada como `(Recommended)`, além de um caminho em texto livre, para que você nunca fique preso a uma escolha fixa.

As rodadas chegam de duas formas:

- **UI nativa de perguntas**, quando o seu host de agente oferece uma e ela comporta a rodada inteira.
- **Uma página HTML gerada**, quando não comporta. A Skill escreve `spec-interview/<slug>/round-N.html` — um arquivo único, autocontido e sem acesso à rede. Abra no navegador, responda, clique em **Copy answers** e cole o bloco retornado no chat. Há também o fallback **Download answers.json**.

Em seguida a Skill repete como leu cada resposta antes de continuar, para que qualquer interpretação errada apareça na hora.

Espere mais de uma rodada quando algum domínio ainda estiver fraco. Ela não redige o SPEC enquanto um domínio obrigatório estiver incompleto, e nunca responde uma pergunta no seu lugar.

### 3. Deixe rodar

Assim que o SPEC estiver `Ready`, planejamento, execução, verificação, replanejamento e relatório seguem automaticamente. Você não precisa invocar as etapas separadamente.

### 4. Retome quando quiser

Feche a sessão, esbarre no limite de contexto, volte na semana seguinte:

```text
Use spec-to-done para continuar o trabalho de triagem automática do suporte.
```

A Skill lê os artefatos persistidos, descobre qual etapa está realmente em aberto e retoma a partir da evidência — não de uma reconstrução do histórico do chat.

### 5. Quando ela para, ela diz por quê

Ela continua até o resultado estar verificado ou até esbarrar em algo que realmente exige um humano: credenciais, autorização, uma ação destrutiva ou irreversível, uma decisão real de produto, ou uma ambiguidade que ela não deve resolver sozinha. Todo encerramento — sucesso, parcial, bloqueado ou falho — passa pelo reporter, então a execução nunca para em silêncio.

### Opcional: uma etapa isolada

Se você pedir explicitamente apenas uma etapa, esse pedido define onde o workflow para. Ele não pula as etapas anteriores: pedir um plano quando nenhum SPEC está pronto executa a entrevista primeiro, depois o plano, e então para.

## O que ela faz

| Etapa | Resultado |
|---|---|
| Especificar | Entrevista você até que resultado, escopo, regras, riscos e critérios de aceitação estejam claros. |
| Planejar | Cria um plano executável contendo somente o trabalho que ainda precisa acontecer. |
| Executar | Realiza e verifica uma tarefa por vez, registrando evidências duráveis. |
| Replanejar | Ajusta o trabalho futuro quando a execução muda as premissas do plano. |
| Reportar | Comunica o que foi concluído, o que foi verificado e o que ainda exige atenção. |

O workflow guarda seu estado em `spec-interview/<slug-do-trabalho>/`:

```text
state.md       status de ciclo de vida, respostas da entrevista, cobertura e gate de prontidão
round-N.html   uma rodada de entrevista gerada (só quando a UI do host não comporta)
SPEC.md        contrato do resultado
PLAN.md        trabalho futuro
TRACK.md       evidência append-only da execução e estado de retomada
REPORT.md      resultado final apresentado ao desenvolvedor
```

Como o estado vive nesses artefatos, e não apenas na conversa, o trabalho sobrevive a interrupções, limites de contexto e novas sessões sem reconstruir o progresso pela memória.

### Um trabalho ativo por vez

Cada pasta de trabalho carrega seu ciclo de vida na primeira linha do `state.md`:

- `active` — o único trabalho que a Skill pode avançar. No máximo um, em todo o repositório.
- `frozen` — mantido como documentação. A Skill não entra nele, não o retoma e não reaproveita seu slug.
- `closed` — o relatório terminal foi escrito e o ciclo se encerrou.

Iniciar um trabalho novo, ou nomear um congelado, torna aquele trabalho `active` e congela o anterior — de forma declarada, nunca silenciosa. Peça para congelar um trabalho a qualquer momento para estacioná-lo sem perder seus artefatos. `closed` não é o mesmo que `COMPLETED`: um trabalho pode encerrar como bloqueado ou falho.

## Feita para trabalhos longos

- Começa trabalhos substanciais pela especificação, mesmo quando o briefing inicial parece detalhado.
- Afasta mudanças triviais e reversíveis de uma cerimônia desnecessária.
- Mantém o trabalho concluído fora do plano ativo e preserva suas evidências no `TRACK.md`.
- Verifica cada tarefa contra o estado observável antes de avançar, em vez de confiar no relato de quem executou.
- Distingue *implementado*, *verificado*, *atestado* e *não verificado*, e nunca apresenta trabalho não verificado como concluído.
- Replaneja somente o futuro; nunca enfraquece silenciosamente o resultado combinado.
- Reconcilia interrupções entre execução, registro, gates e planejamento.
- Retoma o mesmo workflow quando um bloqueador antes esgotado foi comprovadamente resolvido.
- Pausa diante de autoridade, credenciais, ações destrutivas ou decisões reais de produto, em vez de adivinhar.

## Comparada aos modos de plano e de goal

Elas resolvem metades diferentes do problema, e `spec-to-done` não substitui nenhuma das duas:

> O modo de plano planeja. O modo de goal persiste. `spec-to-done` governa todo o caminho da ambiguidade até a conclusão verificada.

## Quando usar

Use `spec-to-done` para features, produtos, migrações, integrações, redesigns, lançamentos ou outros resultados com várias etapas, restrições importantes ou modos de falha relevantes.

Não use para uma mudança pequena, reversível e de resultado óbvio, como trocar a cor de um botão, nem quando você pedir explicitamente apenas uma etapa isolada.

## Pacote

A Skill é autocontida em [`skills/spec-to-done`](skills/spec-to-done):

```text
spec-to-done/
├── SKILL.md
├── assets/interview-round.template.html
└── references/{specify,plan,execute,report}.md
```

Ela não possui dependência de Python em runtime nem configuração específica de provedor.

## Feedback

O projeto é experimental, e os relatos mais úteis vêm de trabalho real — uma feature, uma migração, uma integração, um refactor — em vez de uma tarefa de brinquedo. Críticas são bem-vindas, principalmente sobre:

- se a fase de especificação é rígida demais;
- se a cerimônia extra se paga em trabalhos longos;
- se o modelo de track e replanejamento realmente ajuda depois de uma interrupção;
- se ela pega desvios que um fluxo comum de plano ou goal teria deixado passar.

Abra uma issue em [giuice/spec-to-done](https://github.com/giuice/spec-to-done).

## Licença

[MIT](LICENSE) © 2026 giuice

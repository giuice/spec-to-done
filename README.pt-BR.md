# spec-to-done

Uma única Skill que conduz trabalhos substanciais de um pedido ainda incerto até um resultado verificado:

```text
ESPECIFICAR → PLANEJAR → EXECUTAR ↔ REPLANEJAR → REPORTAR
```

`spec-to-done` mantém todo o workflow em um processo contínuo e retomável. Ela entrevista antes de planejar, verifica o trabalho em vez de presumir que está pronto, adapta o plano quando a realidade muda e termina com um relato conciso do que se tornou verdadeiro.

English: [README.md](README.md).

## Instalação

Instale pelo [skills.sh](https://skills.sh):

```bash
npx skills add giuice/spec-to-done --skill spec-to-done
```

O instalador permite escolher os ambientes de agentes compatíveis nos quais a Skill será instalada.

## Como usar

Peça um resultado de ponta a ponta em linguagem natural:

```text
Use spec-to-done para levar a triagem automática do nosso suporte da ideia até a conclusão.
```

Ou retome o trabalho em uma sessão futura:

```text
Use spec-to-done para continuar o trabalho de triagem automática do suporte.
```

Você não precisa chamar as etapas separadamente. A Skill inspeciona o estado persistido, seleciona a etapa correta, conclui essa etapa e continua automaticamente até alcançar um resultado verificado ou precisar de uma decisão real sua.

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
SPEC.md    contrato do resultado
PLAN.md    trabalho futuro
TRACK.md   evidência append-only da execução e estado de retomada
REPORT.md  resultado final apresentado ao desenvolvedor
```

Como o estado vive nesses artefatos, e não apenas na conversa, o trabalho sobrevive a interrupções, limites de contexto e novas sessões sem reconstruir o progresso pela memória.

## Feita para trabalhos longos

- Começa trabalhos substanciais pela especificação, mesmo quando o briefing inicial parece detalhado.
- Afasta mudanças triviais e reversíveis de uma cerimônia desnecessária.
- Mantém o trabalho concluído fora do plano ativo e preserva suas evidências no `TRACK.md`.
- Verifica cada tarefa antes de avançar.
- Replaneja somente o futuro; nunca enfraquece silenciosamente o resultado combinado.
- Reconcilia interrupções entre execução, registro, gates e planejamento.
- Retoma o mesmo workflow quando um bloqueador antes esgotado foi comprovadamente resolvido.
- Pausa diante de autoridade, credenciais, ações destrutivas ou decisões reais de produto, em vez de adivinhar.

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

## Licença

[MIT](LICENSE) © 2026 giuice

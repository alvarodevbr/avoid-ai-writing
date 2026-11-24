---
name: false-positive-reviewer
version: 3.39.1
license: MIT
description: Use quando um usuário perguntar o que significam as sinalizações de escrita de IA, se a saída do detector prova autoria por IA, ou quiser uma interpretação cuidadosa de possíveis falsos positivos, especialmente para decisões acadêmicas, de contratação, publicação, disciplinares ou outras decisões de grande impacto.
---

# False-Positive Reviewer

Interprete sinais de escrita de IA sem transformá-los em um veredito de autoria sem fundamento.

## Autoridade

Use as ressalvas de evidência e orientações de padrões em `../avoid-ai-writing/SKILL.md`. A Skill original trata explicitamente as sinalizações como sinais de qualidade de escrita, não como prova de quem ou o que escreveu o texto.

Para trabalho entre Skills, siga `../avoid-ai-writing-router/references/handoff-contract.md` e `../avoid-ai-writing-router/references/skill-graph.json`.

## Connection contract

### Entrada (Incoming)

Aceite trabalho de interpretação vindo de:

- `avoid-ai-writing-router` via `ROUTE` quando o usuário pedir diretamente uma interpretação de autoria ou de decisão de grande impacto.
- `ai-writing-detector` via `ESCALATE` quando os achados do detector estiverem sendo tratados como prova.
- qualquer outra Skill apenas através do roteador quando o objetivo do usuário mudar para uma alegação de autoria de grande impacto.

Preserve a distinção entre:

- evidência determinística do detector,
- observações editoriais apenas do modelo (*model-only*),
- fatos contextuais fornecidos pelo usuário,
- evidência ainda não disponível.

### Produção (Produce)

Atualize o envelope de handoff apenas com o estado relevante para a interpretação:

- mantenha `consequential_authorship_claim: true` quando aplicável,
- identifique o que a evidência existente pode e não pode estabelecer,
- liste evidências adicionais que reduziriam materialmente a incerteza,
- defina um motivo de retorno ao roteador se o usuário solicitar nova coleta de sinais ou mudar de intenção.

Não reescreva pontuações do detector, não invente valores de confiança nem converta incerteza em probabilidade de autoria.

### Comportamento terminal (Terminal behavior)

This Skill has no direct outgoing Skill edge.

Se uma nova coleta de sinais for genuinamente necessária, return control to `avoid-ai-writing-router` com `fresh_signal_collection_needed`. O roteador pode executar `ai-writing-detector` e então rotear a evidência atualizada de volta para interpretação se o pedido do usuário ainda exigir.

Se o usuário pedir separadamente para reescrever ou editar o texto, return control to `avoid-ai-writing-router` com a nova intenção. Não pule diretamente para reescrita ou mutação a partir desta Skill.

## Lente de evidência de engenharia de IA

Aplique a lente `agency-ai-engineer` codificada em `../avoid-ai-writing-router/references/agency-role-lenses.md`:

- trate a saída do detector como evidência ruidosa em vez de verdade absoluta,
- leve em conta o modo de contexto, gênero, escrita em segundo idioma, registro técnico, software de edição e estilo de base do autor,
- separe o comportamento do modelo da atribuição humana,
- evite falsa precisão,
- prefira evidências de processo quando a decisão tiver consequências sérias.

## Fluxo de trabalho

1. Identifique quais observações são acertos determinísticos do detector, observações editoriais *model-only* ou fatos contextuais fornecidos pelo usuário.
2. Explique os sinais mais fortes e as razões humanas plausíveis pelas quais eles podem aparecer.
3. Considere gênero, escrita em segundo idioma, registro técnico, pressão de prazo, ferramentas de edição, software tipográfico e a linha de base conhecida de quem escreveu.
4. Para decisões de grande impacto, não transforme uma pontuação ou lista de padrões em uma afirmação definitiva de uso de IA, trapaça, fraude, desonestidade ou adequação.
5. Sugira evidências mais probatórias para a decisão legítima, como histórico de versões, rascunhos, logs de revisão ou conversa direta com o autor.

## Condições de parada

Pare quando a questão de interpretação for respondida.

## Saída

Distinga o que o texto realmente mostra, o que pode sugerir, o que não pode comprovar, qual evidência veio de ferramentas executadas versus revisão *model-only*, quais evidências adicionais reduziriam a incerteza e se o controle deve retornar ao roteador para uma nova etapa solicitada.

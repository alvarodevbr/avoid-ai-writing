---
name: ai-writing-detector
version: 3.39.1
license: MIT
description: Use quando o usuário pedir para detectar, escanear, auditar, pontuar ou sinalizar padrões de escrita de IA sem reescrever o texto, incluindo pedidos de um resultado determinístico do detector local quando o host puder executar Node.
---

# AI Writing Detector

Execute uma revisão somente de detecção usando as regras originais do Avoid AI Writing. Nunca reescreva a menos que o usuário altere o pedido.

## Autoridade

O livro de regras canônico é `../avoid-ai-writing/SKILL.md`. Suas ressalvas sobre falsos positivos, contexto, material protegido e alegações de autoria se aplicam aqui.

Para trabalho entre Skills, siga `../avoid-ai-writing-router/references/handoff-contract.md` e as arestas tipadas em `../avoid-ai-writing-router/references/skill-graph.json`.

## Connection contract

### Entrada (Incoming)

Aceite trabalho de detecção vindo de:

- `avoid-ai-writing-router` via `ROUTE` para pedidos somente de detecção, a etapa de auditoria de um pedido de múltiplas etapas ou nova coleta de sinais depois que outra etapa terminal devolve o controle ao roteador.
- `preservation-verifier` via `RECHECK` limitado apenas quando convergência ou auditoria residual fizer parte do pedido.

Do not accept a direct handoff from `false-positive-reviewer`. Essa Skill é terminal no grafo e deve devolver o controle a `avoid-ai-writing-router` quando uma nova coleta de sinais for necessária. Isso evita um ciclo entre revisor e detector.

Leve adiante o `context_profile` canônico separadamente do `context_mode` do detector, junto com restrições protegidas, estado de passagens e flags de risco. Não os colapse nem reinicie.

### Produção (Produce)

Atualize o envelope de handoff com:

- `execution_evidence.detector`: `executed` somente se o detector empacotado realmente rodou, caso contrário `model_only`.
- `detector_summary.score` e `label` somente quando produzidos pelo código executado do detector.
- `detector_summary.issue_types` a partir dos achados reais.
- qualquer flag de risco `consequential_authorship_claim` observada no pedido do usuário.

### Saída (Outgoing)

- `FEED` achados para `voice-preserving-rewriter` apenas quando o usuário também solicitou reescrita de texto retornado.
- `FEED` achados para `file-edit-in-place` apenas quando o usuário solicitou explicitamente a alteração de um arquivo nomeado.
- `ESCALATE` para `false-positive-reviewer` quando o usuário perguntar o que os achados podem comprovar sobre autoria ou outra conclusão de grande impacto.
- Caso contrário, pare após o resultado somente de detecção.

Acertos determinísticos são correspondências candidatas. Mantenha os tipos públicos de problemas e as pontuações do motor intactos e aplique as exceções de contexto e condições de aprovação canônicas ao decidir quais acertos são achados editoriais justificados.

## Lente de evidência de engenharia de IA

Aplique a lente `agency-ai-engineer` codificada em `../avoid-ai-writing-router/references/agency-role-lenses.md`:

- mantenha a saída determinística separada das observações apenas do modelo (*model-only*),
- preserve o perfil de contexto selecionado e o modo do detector nos handoffs seguintes,
- trate pontuação e rótulo como sinais e não como verdade absoluta,
- considere falsos positivos e efeitos de gênero/registro,
- nunca converta detecção de padrões em uma afirmação de classificador de autoria.

## Caminho preferencial

Quando o host atual puder executar Node com segurança:

1. Passe o texto fornecido para `scripts/detect.js`.
2. Use `--context technical` para prosa técnica ou próxima a código quando apropriado. Caso contrário, use `general`.
3. Use `--source-mode rendered-markdown` para Markdown que contenha YAML frontmatter ou comentários HTML. Caso contrário, mantenha o padrão `plain`.
4. Relate a pontuação, o rótulo, os tipos de problema, a gravidade, o texto correspondente e as sugestões do detector.
5. Separe correspondências candidatas determinísticas de achados editoriais justificados e observações que só existem no livro de regras completo.
6. Nunca afirme que houve execução a menos que o comando tenha realmente rodado.

Exemplo:

```bash
printf '%s' "$TEXT" | node scripts/detect.js --context general
```

Para um arquivo:

```bash
node scripts/detect.js --file path/to/draft.md --context general
node scripts/detect.js --file path/to/draft.md --source-mode rendered-markdown
```

Se a execução do Node ou shell estiver indisponível, realize o fluxo somente de detecção a partir da Skill canônica `avoid-ai-writing` e diga explicitamente que o detector determinístico não foi executado.

## Condições de parada

Pare aqui quando o pedido for somente de detecção. Não continue para reescrita, alteração de arquivo ou interpretação apenas porque essas Skills estão disponíveis.

## Saída

Retorne o rótulo geral e a pontuação quando executado, correspondências candidatas detectadas agrupadas por gravidade, uma breve avaliação contextual que distinga achados justificados de falsos positivos plausíveis, status de execução e nenhuma versão reescrita, a menos que o controle tenha passado explicitamente para um responsável por reescrita.

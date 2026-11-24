---
name: preservation-verifier
version: 3.39.1
license: MIT
description: Use quando o usuário fornecer uma versão original e uma reescrita, perguntar se uma reescrita preservou o conteúdo protegido ou quiser uma verificação determinística de código, frontmatter, citações, tabelas, links, caminhos, números, cabeçalhos e regressões residuais de padrões de IA.
---

# Preservation Verifier

Verifique se uma reescrita ou edição de arquivo manteve o conteúdo que o `../avoid-ai-writing/SKILL.md` original manda proteger.

Para trabalho entre Skills, siga `../avoid-ai-writing-router/references/handoff-contract.md` e `../avoid-ai-writing-router/references/skill-graph.json`.

## Connection contract

### Entrada (Incoming)

Aceite verificação antes/depois vinda de:

- `avoid-ai-writing-router` via `ROUTE` quando o usuário fornecer diretamente o material antes e depois.
- `voice-preserving-rewriter` via `VERIFY` após reescrita de texto retornado.
- `file-edit-in-place` via `VERIFY` após mutação autorizada de arquivo nomeado.

Exija ambas as versões (original e atual). Se qualquer uma estiver indisponível, devolva o controle ao roteador em vez de inventar uma comparação.

### Produção (Produce)

Atualize o envelope de handoff com:

- `execution_evidence.verifier`: `executed` somente se o validador empacotado rodou, caso contrário `model_only`.
- `verification_summary.status`: `PASS`, `REVIEW` ou `FAIL`.
- erros bloqueantes e avisos.
- alvo exato de reparo quando o reparo for possível.

### Saída (Outgoing)

- `REPAIR` para `voice-preserving-rewriter` quando o texto retornado falhou na preservação e o orçamento compartilhado de edição tiver espaço.
- `REPAIR` para `file-edit-in-place` quando um arquivo nomeado falhou na preservação e o orçamento compartilhado de edição tiver espaço.
- `RECHECK` para `ai-writing-detector` apenas quando convergência ou auditoria residual fizer parte do pedido do usuário.
- Pare em `PASS` a menos que reste outra etapa solicitada pelo usuário.
- Pare e relate em caso de segunda falha de verificação. Não inicie outro loop de reparo.

## Lentes de arquitetura e implementação

Aplique ambas as lentes codificadas em `../avoid-ai-writing-router/references/agency-role-lenses.md` (`agency-software-architect` e `agency-senior-developer`). O verificador não reescreve conteúdo por conta própria.

## Caminho determinístico preferencial

O script empacotado `scripts/validate.js` é uma cópia exata do validador de preservação do repositório-fonte. Quando a execução em Node estiver disponível, execute:

```bash
node scripts/validate.js --residual-policy warn before.md after.md
```

Para uso programático:

```js
const { validate } = require("./scripts/validate.js");
const result = validate(original, rewritten, { residualPolicy: "warn" });
```

Se a execução estiver indisponível, compare o original e a reescrita manualmente usando o mesmo contrato de preservação e rotule o resultado como `model_only`.

## Tratamento de resultados

### PASS
Nenhum erro mecânico bloqueante de preservação foi encontrado e nenhum aviso ou preocupação semântica permanece sem solução.

### REVIEW
Avisos ou alterações semânticas exigem julgamento, mas não são automaticamente bloqueantes (ex.: `number-added`, `number-missing`, `residual-grew` ou edição autorizada pelo usuário em região normalmente protegida).

### FAIL
Conteúdo protegido mudou ou desapareceu. Identifique o responsável correto pelo reparo (`voice-preserving-rewriter` para texto retornado, `file-edit-in-place` para arquivo nomeado) se `pass.index < pass.max`.

## Limite de loop de reparo

É permitida no máximo uma reentrada de reparo, e somente enquanto o orçamento de edição solicitado tiver espaço. O reparo consome a próxima passagem de edição. A verificação em si não consome uma passagem.

## Saída

Retorne `PASS`, `FAIL` ou `REVIEW`, o status de execução do verificador, erros bloqueantes de preservação, avisos, qualquer revisão separada de guarda semântica, o responsável sugerido pelo reparo e se a oportunidade limitada de reparo já foi utilizada.

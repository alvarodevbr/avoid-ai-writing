# Pacote de plugin para ChatGPT e Codex

Este repositório contém um pacote nativo de plugin para ChatGPT e Codex, mantendo o plugin Claude existente e os arquivos canônicos `SKILL.md` e `references/patterns.md` (traduzidos e adaptados para o Português Brasileiro).

## Arquitetura

O pacote público é baseado exclusivamente em Skills. O projeto separado `avoid-ai-writing-mcp` permanece opcional e não é empacotado nem obrigatório.

Os arquivos canônicos `SKILL.md` e `references/patterns.md` permanecem como a autoridade editorial. O pacote OpenAI adiciona Skills de fluxo de trabalho focadas em torno dele:

- `avoid-ai-writing`: cópia gerada da Skill original, idêntica exceto pela omissão do bloco `metadata` no frontmatter (que o portal da OpenAI rejeita)
- `avoid-ai-writing-router`: orquestração para pedidos mistos e de múltiplas etapas
- `ai-writing-detector`: fluxo de trabalho somente detecção com o detector determinístico empacotado
- `voice-preserving-rewriter`: responsável pela reescrita de texto retornado
- `file-edit-in-place`: responsável pela mutação restrita de arquivos explicitamente nomeados
- `preservation-verifier`: gate de preservação antes/depois
- `false-positive-reviewer`: guardrail terminal de interpretação

## Grafo de Skills conectadas

O grafo legível por máquina está em `skills/avoid-ai-writing-router/references/skill-graph.json`.

A rede usa relacionamentos tipados:

- `ROUTE` seleciona um responsável primário.
- `FEED` passa evidências para outra etapa solicitada.
- `VERIFY` envia conteúdo antes/depois para o gate de preservação.
- `REPAIR` retorna um escopo de preservação que falhou para o responsável correto uma vez, quando o orçamento compartilhado de passagens de edição tem espaço.
- `RECHECK` permite uma auditoria residual limitada quando solicitada.
- `ESCALATE` envia questões de interpretação para o revisor terminal.

## Validação

Execute:

```bash
python3 skills/avoid-ai-writing-router/scripts/validate_connections.py .
python3 scripts/validate-openai-plugin.py . --json
python3 scripts/package-openai-plugin.py . /tmp/avoid-ai-writing.zip --json
```

## Fonte da verdade

Após alterar `SKILL.md` ou `references/patterns.md`, execute:

```bash
bash scripts/sync-plugin-skill.sh
```

Isso atualiza ambas as cópias geradas do plugin e verifica se os manifestos do Claude, Antigravity e OpenAI correspondem à versão canônica da Skill.

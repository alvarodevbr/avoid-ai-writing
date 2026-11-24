# Cursor Rule — avoid-ai-writing

Regra pronta para o [Cursor](https://cursor.sh) que porta a skill [`avoid-ai-writing`](../SKILL.md) para o formato de regra `.mdc` do Cursor, com as mesmas regras, vocabulário em camadas, perfis de contexto e modos `detect` / `rewrite`, traduzida e adaptada para o Português Brasileiro (PT-BR). O porte é distribuído sem os scripts da skill, portanto três verificações mecânicas tornam-se etapas manuais que a regra relata como não verificadas: o validador de preservação, o verificador de configuração `--style` e o normalizador de aspas.

## Instalação

Copie `avoid-ai-writing.mdc` para o diretório `.cursor/rules/` do seu projeto:

```sh
mkdir -p .cursor/rules
curl -o .cursor/rules/avoid-ai-writing.mdc \
  https://raw.githubusercontent.com/alvarodevbr/avoid-ai-writing/main/cursor-rules/avoid-ai-writing.mdc
```

Por padrão, a regra é ativada em arquivos `.md`, `.mdx`, `.txt`, `.rst` e `.adoc` (via campo `globs` no frontmatter).

## Frases de gatilho

Uma vez instalada, peça ao Cursor:
- *"Remova os IA-ismos desta seção."* (*"Remove AI-isms from this section."*)
- *"Audite este rascunho em busca de padrões de escrita de IA."* (*"Audit this draft for AI writing patterns."*)
- *"Deixe este texto com menos cara de IA."* (*"Make this sound less like AI."*)
- *"Execute o avoid-ai-writing no modo detect."* (*"Run avoid-ai-writing in detect mode."*)

## Projetos antigos do Cursor

Se você estiver em uma versão do Cursor que ainda usa `.cursorrules` (arquivo único na raiz do repositório), pode anexar o corpo de `avoid-ai-writing.mdc` (a parte abaixo do frontmatter `---`) diretamente ao seu arquivo `.cursorrules` existente.

## Atualização

Este arquivo e [`dist/avoid-ai-writing.md`](../dist/avoid-ai-writing.md) são gerados a partir de `SKILL.md` mais `references/patterns.md`. Execute `bash scripts/sync-cursor-rules.sh` após editar qualquer uma das fontes. Não edite os arquivos gerados manualmente.

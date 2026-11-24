# avoid-ai-writing: instruções para agentes

O contrato do repositório para todos os ambientes de execução de agentes. O Codex e outros ambientes leem este arquivo diretamente; o Claude Code o lê por meio de `CLAUDE.md`, que o importa.

## O que é isto

Uma skill de escrita baseada em diretório (`SKILL.md` mais `references/patterns.md`) que audita e reescreve conteúdos para remover padrões de escrita de IA, traduzida e adaptada para o Português Brasileiro (PT-BR) com compatibilidade bilíngue. A skill é um arquivo Markdown consumido por assistentes de programação com IA; o repositório também inclui um detector determinístico sem dependências e uma suíte de testes.

## Estrutura do repositório

- `SKILL.md` — instruções de entrada, níveis de gravidade, regras de preservação e formatos de saída.
- `references/patterns.md` — camadas canônicas de vocabulário, catálogo, perfis de contexto e de voz.
- `SKILL.full.md` — fonte completa gerada para comparação e autoverificação (`self-scan`).
- `dist/avoid-ai-writing.md` — instruções portáteis geradas para colagem direta.
- `README.md` — documentação pública, instruções de instalação, tabela de referência de padrões, exemplo completo antes/depois.
- `CHANGELOG.md` — histórico de versões com o que mudou e por quê.

## Como fazer alterações

Edite `SKILL.md` e `references/patterns.md` diretamente. Ao fazer alterações:

- Siga a [política de changelog e versionamento](CONTRIBUTING.md#changelog-and-versioning): adicione uma entrada em `Unreleased` para mudanças visíveis ao usuário; correções rotineiras de documentação, testes e manutenção sem efeito visível ao usuário não precisam de entrada nem de aumento de versão.
- Ao preparar um lançamento, atualize o frontmatter do `SKILL.md` (`version: X.Y.Z`), o `package.json` e o cabeçalho datado do changelog. Atualize a mesma versão manualmente nos três manifestos de plugin antes de sincronizar:
  - `plugins/avoid-ai-writing/.claude-plugin/plugin.json`
  - `plugins/avoid-ai-writing/.antigravity-plugin/plugin.json`
  - `.codex-plugin/plugin.json`
- Atualize a mesma versão nos seis frontmatters escritos à mão em `skills/*/SKILL.md` (todos exceto o gerado `skills/avoid-ai-writing/SKILL.md`). `python3 scripts/validate-openai-plugin.py . --json` falha quando um deles difere do manifesto da OpenAI.
- Execute `bash scripts/sync-plugin-skill.sh && bash scripts/sync-cursor-rules.sh`. O primeiro script valida as versões dos três manifestos e do `package.json` contra o `SKILL.md` e regenera cópias empacotadas da skill, recursos do detector, scripts e exemplos; o segundo regenera os artefatos portáteis para colagem e Cursor. Nenhum dos scripts gera as versões dos manifestos.
- Execute `npm test` para exercitar o detector, contrato de categorias, validador, utilitários de corpus e verificações de estilo.
- Atualize o `README.md` se a mudança afetar instalação, uso, lista de recursos ou contagem de padrões.
- A contagem de padrões é canônica no item do README "74 pattern categories" bullet (derivado das entradas de detecção `###` de `references/patterns.md`). Este item cita esse número aqui como uma **cópia verificada** para contexto do agente — atualize o README e esta frase quando adicionar ou remover uma categoria de detecção. Não repita a contagem em outros lugares; a CI (`scripts/check-pattern-count.sh`) falha se qualquer um dos literais divergir de `references/patterns.md`.

## Arquitetura da skill

A skill possui três modos (`rewrite` por padrão, `detect` apenas sinalização, `edit` edição local) e processa o texto por este pipeline:

1. **Detecção de perfil de contexto** — autodetecta ou aceita uma dica de perfil (`linkedin`, `blog`, `technical-blog`, `investor-email`, `docs`, `casual`) que ajusta o rigor das regras por meio da matriz de tolerância
2. **Correspondência de padrões** — categorias de detecção abrangendo padrões de conteúdo, linguagem, estrutura, comunicação e metapadrões (veja `references/patterns.md` para o catálogo; a contagem está no item do README)
3. **Sinalização de vocabulário** — sistema de 3 camadas: Tier 1 (sempre revisar), Tier 2 (sinalizar em agrupamentos), Tier 3 (sinalizar em alta densidade)
4. **Classificação de gravidade** — P0 (destruidores de credibilidade), P1 (cheiro óbvio de IA), P2 (polimento estilístico)
5. **Saída** — modo `rewrite`: uma reescrita final única, resumo útil de alterações e verificação; achados detalhados mediante solicitação. Modo `detect`: achados e avaliação

## Restrições principais

- Mantenha o arquivo de entrada `SKILL.md` abaixo de 500 linhas com frontmatter compatível com agentskills.io; preserve todo o texto das regras na referência e nos artefatos gerados.
- Execute `bash scripts/sync-plugin-skill.sh && bash scripts/sync-cursor-rules.sh` após edições canônicas. Nunca edite cópias geradas.
- As entradas da tabela de substituição de palavras precisam de alternativas específicas, não apenas "reformule".
- A válvula de escape de autorreferência (exemplos entre aspas isentos de sinalização) deve ser preservada — sem ela, a skill sinaliza sua própria documentação.
- O perfil `technical-blog` tem exceções explícitas na tabela de palavras (ex.: "robust"/"robusto" e "ecosystem"/"ecossistema" são legítimos em contextos técnicos).
- "Extra strict" e "skip" na matriz de tolerância têm significados específicos definidos no arquivo.
- O levantamento `ai-writing-skill-field-guide` classifica este repositório em primeiro lugar em manutenção, e seu autor abriu a issue #12 aqui. Qualquer citação pública desse ranking deve trazer essa divulgação junto.

## Compatibilidade

A skill funciona com Claude Code, OpenClaw/ClawHub e qualquer agente compatível com agentskills.io. O frontmatter inclui os campos `agentskills_spec` e `openclaw`. As alterações não devem quebrar nenhum dos formatos.

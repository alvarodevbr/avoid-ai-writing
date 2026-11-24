# Contribuindo (Contributing)

Obrigado por ajudar a melhorar esta skill. Ela ensina um LLM (e agora um motor determinístico) a identificar e corrigir sinais de escrita de IA. Contribuições são bem-vindas — algumas diretrizes mantêm o projeto coerente.

## Escolhendo uma issue

Verifique a pessoa atribuída, comentários e PRs vinculados para ver se alguém já está trabalhando em uma issue. Para uma issue livre, deixe um comentário curto avisando que está começando. Você pode começar imediatamente: não precisa esperar por atribuição ou confirmação de mantenedor.

Issues rotuladas como `good first issue` são reservadas para pessoas fazendo sua primeira contribuição neste repositório. Pegue apenas uma para o seu primeiro PR; deixe as outras para outros novatos.

> [!IMPORTANT]
> Se você já contribuiu aqui, escolha uma issue `help wanted` sem o rótulo `good first issue`, proponha outra melhoria ou ajude a revisar e testar PRs de novos colaboradores.

## Como o repositório se organiza

| Caminho | O que contém |
|---|---|
| `SKILL.md` | Instruções de entrada, níveis de gravidade, formatos de saída e diretrizes de proteção. |
| `references/patterns.md` | Catálogo canônico de padrões, camadas de vocabulário, perfis de contexto e de voz. |
| `detector/patterns.js` | O motor determinístico — o subconjunto executável das regras. |
| `detector/CATEGORIES.md` | O mapa entre as regras de `references/patterns.md` e os `type`s do detector. Mantenha-o atualizado. |
| `README.md` | A apresentação e a lista numerada de padrões de prosa. |
| [`GLOSSARY.md`](GLOSSARY.md) | Definições de uma linha dos termos do projeto, cada uma vinculada à sua fonte canônica. |
| `cursor-rules/`, `plugins/` | Integrações com editores e ferramentas. |
| `skills/` | As sete Skills para ChatGPT/Codex e seu grafo de conexão validado. Veja [`OPENAI_PLUGIN.md`](OPENAI_PLUGIN.md); mudanças nos modos ou na saída de `SKILL.md` podem quebrá-las. |

## Adicionando ou alterando uma regra

Primeiro decida que tipo de regra ela é:

- **Detectável por regex** (uma frase, um caractere, uma forma estrutural) → adicione a `references/patterns.md`, adicione a detecção a `detector/patterns.js` com um novo `type`, e adicione uma linha a `detector/CATEGORIES.md`. Cubra-a com um teste em `detector/patterns.test.js` (tanto um verdadeiro positivo quanto um caso que *não* deve disparar).
- **Apenas julgamento (Judgment-only)** (exige leitura de sentido — tom, estrutura, desfile de citações) → adicione à prosa de `references/patterns.md` e liste em "Skill-only" em `detector/CATEGORIES.md`. Não há tipo de detector para estas.

### Contagem de categorias de padrões (catálogo de detecção)

Quando você adiciona ou remove um `###` de detecção sob `## What to remove or fix` em `references/patterns.md` (não prosa de julgamento ou testes do lado do autor), a CI deriva o novo total e o compara a dois literais:

1. **`README.md`** — atualize o item `**NN pattern categories**` para corresponder à contagem derivada.
2. **`AGENTS.md`** — atualize a expressão citada `README "NN pattern categories" bullet` na orientação de contagem de padrões para corresponder ao mesmo número.

`scripts/check-pattern-count.sh` valida ambos em cada PR. Adicionar uma linha na tabela de palavras exige apenas atualizar o item separado `**NN-entry word replacement table**` no README (mesmo script).

## Piloto de cartões de decisão editorial (Editorial decision-card pilot)

Entradas selecionadas do catálogo possuem um ID de regra `ec-`, um mapeamento parcial de detector, condições de aplicabilidade e exemplos de edição/manutenção/lacuna (`Apply`, `Edit`, `Keep`, `Flag a gap`). Preserve esses IDs e âncoras de seção ao refinar um cartão.

## Precisão acima de revocação (Precision over recall)

Esta skill é deliberadamente enviesada para falsos negativos: uma regra que sinaliza erroneamente a escrita humana comum é pior do que uma que deixa passar um sinal de IA, porque falsos positivos corroem a confiança em todas as outras regras. Antes de propor uma regra, pergunte quem seria sinalizado por engano e adicione exceções (*carve-outs*) para os casos legítimos.

## Cite suas fontes

Se sua regra se baseia em uma afirmação factual sobre como a IA ou os humanos escrevem, inclua um link de fonte para ela.

## Language adaptations

Esta skill foi adaptada para o Português Brasileiro (PT-BR), mantendo compatibilidade técnica com o detector determinístico e o ecossistema original em inglês. Ao expandir regras específicas de idioma:

Identifique o idioma-alvo, dialeto, público e registros. Construa camadas lexicais a partir da escrita nativa em vez de apenas traduzir literalmente entradas do inglês; verifique quais regras estruturais se transferem e adicione exceções específicas do idioma (como as regras do Acordo Ortográfico da Língua Portuguesa e a preferência por *sentence case* em títulos no Brasil). Mostre exemplos positivos e casos que não devem disparar em registros relevantes. Mantenha a licença e a atribuição upstream ao adaptar este trabalho.

Para solicitar um link na [seção da comunidade](README.md#community--multilingual), abra uma issue com a URL do repositório, mantenedor, idioma e dialeto, uma breve descrição das mudanças nativas, evidências e limitações.

## Guias de estilo e licenciamento

As regras da [auditoria de licenças #88](https://github.com/alvarodevbr/avoid-ai-writing/issues/88):

- **Este repositório não inclui nenhum guia de estilo cuja licença não possa ser verificada.** A camada `--style` é orientada por configuração; os usuários fornecem suas próprias convenções.
- **Guias de licença aberta podem ser distribuídos posteriormente como configurações de exemplo** (Google, Microsoft, GOV.UK e 18F se qualificam).
- **Guias pagos (CMOS, APA, MLA, AP, ABNT NBR) nunca são distribuídos no pacote, sob nenhuma forma ou nome.** Passar um deles para `--style` cai no fallback que não reivindica conformidade.

## Configuração de desenvolvimento

Faça um fork do repositório e clone-o. A versão mínima do Node é `engines.node` no `package.json` (`>=18`). Os scripts de sincronização também precisam de Bash e Python 3. Após editar `SKILL.md` ou `references/patterns.md`, regenere as cópias:

```bash
bash scripts/sync-plugin-skill.sh && bash scripts/sync-cursor-rules.sh
```

## Executando os testes

```bash
npm test
```

`npm test` executa todas as suítes via `scripts/run-tests.js` e imprime um resumo combinado. Para rodar apenas uma suíte:

```bash
node scripts/run-tests.js detector/patterns.test.js
# ou invoque o arquivo diretamente:
node detector/patterns.test.js
```

Nenhuma dependência externa é necessária; apenas Node 18+.

## Desvio de documentação (Documentation drift)

O job de CI `SSOT / ssot` verifica os requisitos locais de versão do Node do repositório usando `.ssot-local.yaml` em cada PR. O `package.json` detém o mínimo do Node do detector (`>=18`).

## Escreva prosa limpa

Este repositório fiscaliza a qualidade da escrita, portanto a prosa que você adicionar precisa passar pelo mesmo crivo. Passe suas adições pela própria skill. Coloque entre aspas ou crases o vocabulário de exemplo em entradas de changelog e documentação; a autoverificação (`self-scan`) isenta trechos entre aspas, código inline, blocos de código, tabelas e citações em bloco.

## Changelog and versioning

Adicione uma entrada sob `## [Unreleased]` no `CHANGELOG.md` quando uma alteração afetar os usuários: comportamento de detecção ou reescrita, regras de escrita, APIs públicas ou opções de CLI, configuração, instalação ou empacotamento, compatibilidade ou segurança.

Pule o changelog para correções rotineiras de documentação, links, formatação, orientação de colaboradores, testes e refatoração interna ou manutenção de CI sem efeito visível ao usuário.

Ao preparar um lançamento, mova as entradas de `Unreleased` para um cabeçalho versionado e datado (`## [X.Y.Z] — YYYY-MM-DD`) e atualize as versões correspondentes em `SKILL.md`, `package.json`, nos três manifestos de plugin e nos seis frontmatters escritos à mão sob `skills/`.

Após alterar qualquer arquivo canônico, execute `bash scripts/sync-plugin-skill.sh && bash scripts/sync-cursor-rules.sh`. Não edite cópias geradas manualmente.

## Checklist de lançamento (Release checklist)

1. Mova as entradas de `## [Unreleased]` para um cabeçalho versionado e datado (`## [X.Y.Z] — YYYY-MM-DD`).
2. Defina a mesma versão no frontmatter de `SKILL.md`, `package.json`, `plugins/avoid-ai-writing/.claude-plugin/plugin.json`, `plugins/avoid-ai-writing/.antigravity-plugin/plugin.json`, `.codex-plugin/plugin.json` e na linha `version:` dos seis arquivos `skills/*/SKILL.md` escritos à mão.
3. Execute `bash scripts/sync-plugin-skill.sh && bash scripts/sync-cursor-rules.sh`.
4. Execute `npm test` e `python3 scripts/validate-openai-plugin.py . --json`.

Se uma execução de release falhar, siga [o procedimento de recuperação de release](docs/releasing.md).

# Motor do detector (Detector engine)

`patterns.js` é a expressão executável das regras de padrões desta skill — um motor de detecção sem dependências e sem etapa de build que pontua textos em busca de sinais de escrita de IA (em Português Brasileiro e inglês). Ele roda de forma idêntica no Node (`>=18`) e no navegador.

O arquivo `references/patterns.md` da skill é o catálogo legível por humanos das regras; este motor é a implementação determinística e testável do subconjunto detectável por regex, além de detectores estilométricos e de impressão digital de ferramentas de IA. Veja [`CATEGORIES.md`](./CATEGORIES.md) para o mapeamento regra ↔ categoria que mantém os dois sincronizados.

## Como executar

### Via npm

Instale o pacote publicado do detector em outro projeto:

```bash
npm install avoid-ai-writing-detector
```

```js
const AIDetector = require("avoid-ai-writing-detector");
const result = AIDetector.analyzeText("Seu texto aqui…");
console.log(result.score, result.label, result.issues.length);
```

### Pela linha de comando

O pacote publicado expõe o comando `avoid-ai-writing`, que pontua um arquivo ou `stdin` e imprime o resultado completo em JSON:

```bash
npx --package avoid-ai-writing-detector avoid-ai-writing draft.md
cat draft.md | npx --package avoid-ai-writing-detector avoid-ai-writing --context technical
```

### Como gate de CI ou pre-commit

`avoid-ai-writing-gate` transforma o detector existente em uma interface de aprovação/reprovação para automação. Ele valida intencionalmente o `issues.length` determinístico por arquivo em vez da pontuação composta:

```bash
avoid-ai-writing-gate --glob "**/*.md" --context technical
avoid-ai-writing-gate --threshold 0 docs/strict-policy.md
avoid-ai-writing-gate --threshold 2 docs/guide.md README.md
avoid-ai-writing-gate --json --glob "**/*.md"
```

Códigos de saída:

- `0`: todos os arquivos escaneados estão dentro ou abaixo do limite de achados;
- `1`: pelo menos um arquivo excede o limite;
- `2`: erro de uso, expansão de glob, leitura de arquivo, UTF-8 ou entrada não escaneável (incluindo documentos acima do limite de 10.000 palavras e documentos de escrita não segmentada como chinês/japonês).

### A partir de um clone local

Use o repositório diretamente ao desenvolver ou validar alterações no detector:

```bash
npm test          # todas as suítes; falhas em um arquivo ainda rodam o restante (sem dependências)
node scripts/run-tests.js detector/patterns.test.js   # uma suíte
node detector/patterns.test.js                        # o mesmo, direto
```

### No navegador

Carregue `patterns.js` como um script simples — ele se registra automaticamente como um objeto global `AIDetector` (o bloco `module.exports` é protegido e só roda em CommonJS).

## `analyzeText(text, options?)` → result

| Campo | Tipo | Significado |
|---|---|---|
| `score` | `0–100` | 0 = limpo, 100 = alta concentração de padrões de IA |
| `label` | string | pontuado: `Clean` (0) / `Minimal AI signals` (1–15) / `Some AI patterns` (16–35) / `Moderate AI signals` (36–60) / `Strong AI signals` (61–80) / `Heavy AI patterns` (81–100). Não pontuado: `Empty` / `Too short` / `Unsupported script` / `Text too long` |
| `issues[]` | `{type, text, severity, …}` | uma entrada por padrão detectado; as chaves `type` mapeiam para [`CATEGORIES.md`](./CATEGORIES.md); os valores de `severity` estão listados em [Severity and P-tiers](#severity-and-p-tiers) |
| `stats` | object | `wordCount`, contagens por camada, `contextMode`, `sourceMode`, contagens de trechos mascarados, `denseAIVocab`, flags de normalização, etc. |
| `document_classification` | string | `HUMAN_ONLY` / `MIXED` / `AI_ONLY` (formato espelha o GPTZero para substituição direta), ou `UNSCORED` nos caminhos de saída antecipada |
| `class_probabilities` | `{human, mixed, ai}` | soma exatamente 1.0 |
| `confidence_category` | `low` / `medium` / `high` | faixa de confiança |
| `highlight_sentence_for_ai` | region[] | trechos de frases com offsets da fonte + pontuação por região, para destaque de interface |

`options.contextMode` aceita `general` (padrão), `technical`, `marketing` e `personal`. O modo técnico suprime alertas que são legítimos em prosa próxima a código (ex.: cabeçalhos em Title Case e termos técnicos legítimos como `robust`/`robusto`, `comprehensive`/`abrangente`, `seamless`, `ecosystem`/`ecossistema`, `leverage`/`alavancar`, `facilitate`/`facilitar`, `underpin`/`embasar`, `streamline`/`otimizar`). Modos inválidos voltam para `general` e definem `stats.contextModeFallback`.

`options.sourceMode` aceita `plain` (padrão) ou `rendered-markdown`. O modo Markdown renderizado mascara o YAML frontmatter inicial e comentários HTML antes de executar a correspondência de padrões e métricas do documento, preservando os offsets da fonte original.

Em ambos os modos de fonte, `<!-- avoid-ai-writing:ignore-start -->` e `<!-- avoid-ai-writing:ignore-end -->` excluem tudo entre eles antes de qualquer outra passagem rodar, e citações em bloco (`> `) e trechos entre aspas duplas são mascarados para não penalizar o autor por material citado.

### Severity and P-tiers

Cada entrada em `issues[]` carrega um de quatro valores de `severity`. `SEVERITY_LABELS` em `patterns.js` os mapeia para rótulos P:

| `severity` | Rótulo P | Tipos que o emitem |
|---|---|---|
| `critical` | P0 | `chatbot`, `sycophantic`, `reasoning-artifact`, `vague-attribution`, `cutoff-disclaimer`, `ai-placeholder`, `ai-citation-markup`, `ai-utm-source`, `normalization-flag` (caracteres de largura zero ou homóglifos) |
| `high` | P1 | `tier1`, `significance-inflation`, `template-phrase`, `hedge-stack`, `future-narrative`, `social-cta-closer`, `negation-chain`, `negative-parallelism`, `formulaic-opener`, `speculative-opener`, `launch-intro`, `fake-casual-prop`, `tier3-phrase-cluster`, `bullet-np-list`, `normalization-flag` (marcadores de roleplay), `fnword-trigram-entropy` (um trigrama repetido no documento) |
| `medium` | P2 | `tier1-clarity`, `tier2`, `transition`, `filler`, `generic-conclusion`, `lets-construction`, `hollow-intensifier`, `lingering-attention`, `novelty-inflation`, `false-concession`, `rhetorical-question`, `real-actual-inflation`, `performed-insight`, `dev-blog-boilerplate`, `crowd-contrast`, `parenthetical-hedge`, `title-case-header`, `unnecessary-hyphenation`, `tier3-phrase`, `hashtag-stuff`, `em-dash`, `formatting`, `punct-distribution`, `cross-para-burstiness`, `uniformity` (comprimento de frase), `fnword-trigram-entropy` (baixa entropia) |
| `low` | P3 | `smart-punct-signature`, `tier3`, `emotional-flatline`, `confidence-calibration`, `low-ttr`, `uniformity` (comprimento de parágrafo) |

## `PATTERN_TABLES`

As tabelas de padrões como dados simples e congelados recursivamente, chaveadas por `type` de problema, para ferramentas que as reemitem em outro formato (`scripts/build-vale.js` constrói o estilo Vale em `vale/` a partir disso).

## `validate(original, rewritten, options?)` → result

`validate.js` verifica se uma reescrita manteve intactos os elementos que `references/patterns.md` manda proteger.

```js
const { validate, formatResult } = require("./detector/validate.js");
const result = validate(originalText, rewrittenText);
if (!result.ok) console.error(formatResult(result));
```

```bash
node detector/validate.js before.md after.md   # sai com código 1 em caso de erro de preservação
node detector/validate.js --residual-policy warn before.md after.md
```

**Erros mecânicos de preservação:** código cercado modificado ou removido, YAML frontmatter alterado, citação em bloco reescrita, célula de tabela alterada, código inline removido, URL ou caminho de arquivo perdido, ou contagem/aninhamento de cabeçalhos alterado.

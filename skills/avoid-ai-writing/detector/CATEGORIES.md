# Mapa de categorias: references/patterns.md ↔ detector

Esta tabela é o contrato antidesvio entre as regras legíveis por humanos em `../references/patterns.md` e o motor executável em `patterns.js`. Quando você adicionar uma regra à skill, decida aqui se ela é detectável por regex (dê a ela um `type` no detector) ou julgamento exclusivo do LLM (marque-a assim). Quando adicionar um `type` no detector, aponte-o de volta para a seção da skill que ele aplica.

The engine exposes 54 issue `type`s (see `TYPE_LABELS` in `patterns.js`). A skill tem mais seções `###` do que isso — a diferença **não** é falta de cobertura, são regras que exigem julgamento de sentido que uma regex não consegue fazer. Os três grupos abaixo explicam cada entrada de ambos os lados.

Três contagens coexistem de propósito e não devem ser forçadas a coincidir: a **contagem de categorias de padrões** do README (o catálogo de prosa voltado para humanos, derivado de `references/patterns.md` e protegido na CI), the engine's **54 `type`s** (que dividem as camadas de vocabulário e adicionam sinais estilométricos), e as seções `###` de `references/patterns.md` (que também incluem testes do lado do autor sem forma detectável). O arquivo `categories.test.js` valida o mapeamento motor ↔ este arquivo e verifica cada declaração do total de `type`s do motor contra `TYPE_LABELS`.

## A. Mapeamento direto (regra da skill → `type` do detector)

| Detector `type` | Rótulo (Label) | Seção em `references/patterns.md` |
|---|---|---|
| `tier1` / `tier2` / `tier3` | AI vocabulary / Word cluster / Overused word | Words and phrases to replace (`tier3` dispara por palavra em `max(3, floor(wordCount × 0.03))` usos de uma forma listada; `load-bearing`: apenas lista permitida de substantivos abstratos imediatos; o modo de contexto `technical` suprime termos técnicos legítimos como `robust`/`robusto`, `comprehensive`/`abrangente`, `seamless`, `ecosystem`/`ecossistema`, `leverage`/`alavancar`, `facilitate`/`facilitar`, `underpin`/`embasar`, `streamline`/`otimizar`) |
| `tier1-clarity` | Wordiness | Words and phrases to replace (Tier 1B) |
| `transition` | AI transition | Transition phrases to remove or rewrite |
| `template-phrase` | Template phrase | Template phrases (avoid) — *parcial; na forma step-towards/forward, apenas os adjetivos de elogio vago listados antes de "step" disparam* |
| `tier3-phrase` / `tier3-phrase-cluster` | Boilerplate phrase / cluster | Template phrases (avoid) / Tier 3 phrases |
| `chatbot` | Chatbot artifact | Chatbot artifacts |
| `sycophantic` | Sycophantic tone | Sycophantic tone |
| `filler` | Filler phrase | Filler phrases |
| `hollow-intensifier` | Hollow intensifier | Sentence structure / Filler phrases (intensifiers), exceto `actually` / `na verdade` dependente de contexto (veja §C) |
| `generic-conclusion` | Generic conclusion | Generic conclusions |
| `social-cta-closer` | Engagement-bait closer | Social endorsement closers |
| `future-narrative` | Generic future narrative | Generic future-narrative closers |
| `lets-construction` | "Let's" opener | "Let's" constructions |
| `reasoning-artifact` | Reasoning artifact | Reasoning chain artifacts |
| `significance-inflation` | Significance inflation | Significance inflation — *parcial; a forma evolution-of / evolução de exige uma palavra inflacionária imediatamente anterior* |
| `novelty-inflation` | Novelty inflation | Novelty inflation *(a sub-regra de rótulos conceituais inventados é apenas para julgamento do LLM)* |
| `real-actual-inflation` | "Real/actual" inflation | "Real/actual" adjective inflation |
| `vague-attribution` | Vague attribution | Vague attributions |
| `emotional-flatline` | Stock reaction framing | Stock reaction framing / Superficial -ing analyses *(tipo estável de API; peso zero de autoria sob a política de evidência)* |
| `lingering-attention` | Lingering-attention claim | Lingering-attention claims *(apenas molduras ancoradas em substantivo)* |
| `cutoff-disclaimer` | Cutoff disclaimer | Cutoff disclaimers |
| `false-concession` | False concession | False concession structure |
| `rhetorical-question` | Rhetorical question | Rhetorical question openers |
| `formulaic-opener` | Formulaic opener | Formulaic challenges / Structural issues |
| `speculative-opener` | Speculative scenario opener | Speculative scenario openers |
| `confidence-calibration` | Confidence stacking | Confidence calibration phrases |
| `hedge-stack` | Hedge-stacked prediction | Hedge-stacked predictions |
| `parenthetical-hedge` | Parenthetical hedge | Parenthetical hedging |
| `hashtag-stuff` | Hashtag stuffing | Hashtag stuffing |
| `bullet-np-list` | Bullet-NP list | Bullet lists of bare noun phrases |
| `title-case-header` | Title Case header | Title case headings |
| `em-dash` / `formatting` | Em dash overuse / Formatting | Formatting |
| `uniformity` | Rhythm uniformity | Rhythm and uniformity |
| `low-ttr` | Low vocabulary diversity | Vocabulary diversity (stylometric) |
| `ai-placeholder` | Unfilled placeholder | Unfilled placeholders |
| `ai-citation-markup` | Chatbot citation markup leak | Chatbot citation markup leaks |
| `ai-utm-source` | AI-tool URL parameter | AI-tool URL parameters |
| `smart-punct-signature` | Smart-punct signature | Formatting (curly quotation marks) — *parcial* |
| `unnecessary-hyphenation` | Unnecessary hyphenation | Unnecessary hyphenation *(apenas subclasses curadas abertas, fechadas e dependentes de posição)* |
| `launch-intro` | Launch-copy introduction | Launch-copy dramatic introductions |
| `crowd-contrast` | Dramatized crowd contrast | Dramatized contrast against the crowd |
| `fake-casual-prop` | Fake-casual prop | Fake-casual register |
| `performed-insight` | Performed-insight phrase | Performed-insight phrases — *parcial; exclusões de sentido literal documentadas em `references/patterns.md`* |
| `negation-chain` | Negation chain | Negation chains — *parcial; limite determinístico de três itens documentado em `references/patterns.md`* |
| `negative-parallelism` | Negative parallelism | Sentence structure: "It's not X — it's Y" / "Não é X — é Y" — *parcial* |
| `dev-blog-boilerplate` | Dev-blog boilerplate | Dev-blog boilerplate — *parcial; exclusão de sentido literal documentada em `references/patterns.md`* |

> **Mapa parcial:** `smart-punct-signature` é apenas de estilo. No modo `plain`, requer pontuação curva, travessão de prosa, vírgula serial, verificações de espaçamento/contração e pelo menos 80 palavras. Emite um achado de baixa prioridade com peso zero de autoria; o modo `rendered-markdown` o omite.

## B. Apenas no detector (estilométrico / impressão digital — sem prosa na skill)

Estes estendem a skill com sinais que funcionam como matemática sobre o documento inteiro, não como uma expressão que um editor humano consultaria em uma tabela:

| Detector `type` | Rótulo (Label) | Por que é exclusivo do motor |
|---|---|---|
| `punct-distribution` | Punctuation distribution | Uniformidade de pontuação por parágrafo |
| `fnword-trigram-entropy` | Grammar repetition | Entropia de trigramas de palavras funcionais |
| `cross-para-burstiness` | Cross-paragraph rhythm | Variância de comprimento de frase entre parágrafos |
| `normalization-flag` | Bypass-trick chars | Detecção de caracteres de largura zero / homóglifos usados para burlar detectores |

## C. Apenas na skill (julgamento do LLM — sem `type` no detector)

Regras que exigem leitura de significado, portanto vivem na prosa da skill e são aplicadas pelo modelo, não pelo motor de regex:

- Muleta de transformação (`Transformation crutch` — P2 renomeação não explicada)
- Falsa agência (`False agency` — P2 tomadores de decisão obscurecidos)
- Colisão de termos de domínio de ajuste de público (`Audience-fit domain-term collision` — prova genérica ambígua em criptografia)
- Explicações sem consequência ("Isso importa porque" / "This matters because")
- Pares de concessão vazia repetidos em Frases de efeito fabricadas (`Repeated empty concession pairs`)
- Rodízio forçado de sinônimos (`Synonym cycling`)
- Fuga do verbo de ligação (`Copula avoidance`)
- Linguagem promocional (`Promotional language`)
- `actually` / `na verdade` dependente de contexto como intensificador vazio
- Estrutura de frase: negação em frase dividida sem minimizador / contagem regressiva de múltiplas negações / negação final solta / correlativas "não apenas X, mas Y"
- Problemas estruturais / Estrutura excessiva / Listas com cabeçalho inline / Inflação de listas numeradas
- Erros de categoria com adjetivos morais (`Moral-adjective category errors`)
- Espelhamento de pares de contraste inventados (`Invented contrast-pair mirroring`)
- Empilhamento de modificadores hifenizados (`Hyphenated modifier stacking`)
- Hifenização desnecessária fora das subclasses curadas do detector
- Falsos intervalos (`False ranges`)
- Desfile de citações (`Notability name-dropping`)
- Validação vaga de terceiros (`Vague third-party validation`)
- Importância autorrotulada (`Self-labeling significance`)
- Respostas em bloco compacto sem quebras de linha (`Wall-of-text replies`)
- Abertura bajuladora com recapitulação (`Recap-flattery opener`)
- Loops de reconhecimento (`Acknowledgment loops`)
- Franqueza narrada (`Narrated candor`)
- Tipografia impecável em registros casuais (`Immaculate typography in casual registers`)
- Fragmentos sem sujeito e passivas sem agente (`Subjectless fragments and agentless passives`)
- Escrita ancorada em diff (`Diff-anchored writing`)
- Ponto final em rótulos de lista (`List-label periods`)
- Preenchimento especulativo de lacunas (`Speculative gap-filling`)
- Ganchos de engajamento estilo infomercial (`Infomercial engagement hooks`)
- Frases de efeito fabricadas e drama staccato (`Manufactured punchlines and staccato drama`)
- Forma "X morreu; vida longa a Y" em Frases de insight encenado
- Fórmulas de aforismo (`Aphorism formulas`)
- Perguntas retóricas empilhadas (`Stacked rhetorical questions`)
- Sequências de frases com a mesma abertura (`Same-opener sentence runs`)
- Contraste com auxiliar solto (`Stranded auxiliary contrast`)
- Dois-pontos abrindo em tríade (`Colon into a triple`)
- Quando reescrever do zero vs. remendar (`When to rewrite from scratch vs. patch`)
- Níveis de gravidade (P0 / P1 / P2)
- Válvula de escape de autorreferência (`Self-reference escape hatch`)
- Imunidade a embaralhamento de parágrafos e Efeito esteira / baixa densidade de informação
- Formato de saída (`Output format`)

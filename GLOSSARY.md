# Glossário (Glossary)

Definições de uma linha dos termos usados na documentação e nos templates de issue deste repositório. Cada entrada aponta para a definição canônica; quando houver divergência, a fonte vinculada prevalece.

## Camadas de vocabulário (Word tiers)

| Termo | Significado |
|---|---|
| [Tier 1](references/patterns.md#tier-1--default-replacements) | Palavras para revisar em cada ocorrência. Dividido em duas faixas, 1A e 1B, que usam as mesmas substituições, mas significam coisas diferentes. |
| [Tier 1A](references/patterns.md#tier-1a--ai-frequency-markers) | Marcadores de frequência de IA: palavras que aparecem muito mais em texto gerado por máquina. Um agrupamento delas é evidência sobre como uma passagem foi produzida. |
| [Tier 1B](references/patterns.md#tier-1b--clarity-edits) | Edições de clareza: prolixidade e formalidade inflada. Não são evidência de autoria por máquina. O detector as emite como `tier1-clarity`. |
| [Tier 2](references/patterns.md#tier-2--flag-when-2-appear-in-the-same-paragraph) | Palavras que são boas sozinhas. Sinalizadas quando duas ou mais aparecem no mesmo parágrafo. |
| [Tier 3](references/patterns.md#tier-3--flag-only-at-high-density) | Palavras comuns sinalizadas apenas em alta densidade, aproximadamente 3% ou mais do texto (mínimo de 3 usos). |
| [Frases do Tier 3](references/patterns.md#tier-3-phrases--flag-at-density-or-in-clusters) | Clichês compostos de várias palavras (*boilerplate*). Sinalizados quando uma frase aparece duas ou mais vezes, ou quando três ou mais frases distintas aparecem no mesmo texto. |

## Gravidade (Severity)

| Termo | Significado |
|---|---|
| [P0](SKILL.md#p0--credibility-killers-fix-immediately) | Destruidores de credibilidade. Corrija imediatamente. |
| [P1](SKILL.md#p1--obvious-ai-smell-fix-before-publishing) | Cheiro óbvio de IA. Corrija antes de publicar. |
| [P2](SKILL.md#p2--stylistic-polish-fix-when-time-allows) | Polimento estilístico. Corrija quando o tempo permitir. Uma passagem rápida cobre P0 e P1; uma auditoria completa cobre os três. |

## Modos e perfis (Modes and profiles)

| Termo | Significado |
|---|---|
| [Modo `rewrite`](SKILL.md#modes) | O padrão. Sinaliza IA-ismos e reescreve o texto para corrigi-los. |
| [Modo `detect`](SKILL.md#modes) | Sinaliza apenas os IA-ismos, sem reescrever. |
| [Modo `edit`](SKILL.md#modes) | Edita um arquivo de prosa diretamente no local com alterações mínimas e direcionadas. |
| [Perfil de contexto](references/patterns.md#context-profiles) | A dica de público (`linkedin`, `blog`, `technical-blog`, `investor-email`, `docs`, `casual`) que define o rigor de cada regra. |
| [Perfil de voz](references/patterns.md#voice-profiles) | A persona opcional (`casual`, `professional`, `technical`, `warm`, `blunt`) que define como a prosa deve soar. Independente do perfil de contexto. |
| [Registro (Register)](.github/ISSUE_TEMPLATE/false_positive.yml) | O tipo de escrita: blog, documentação, e-mail, post social e assim por diante. |
| [Válvula de escape de autorreferência](SKILL.md#self-reference-escape-hatch) | Exemplos entre aspas, blocos de código e textos marcados como ilustrativos ficam isentos de sinalização, para que escrever *sobre* padrões de IA não seja penalizado por citá-los. |

## Opções do detector (Detector options)

| Termo | Significado |
|---|---|
| [`contextMode`](detector/README.md#analyzetexttext-options--result) | O contexto do detector: `general` (padrão), `technical`, `marketing` ou `personal`. Cada perfil de contexto mapeia para um deles; veja [o mapeamento](references/patterns.md#detector-mode-mapping). |
| [`sourceMode`](detector/README.md#analyzetexttext-options--result) | Como o detector lê a entrada: `plain` (padrão) ou `rendered-markdown`, que mascara YAML frontmatter e comentários HTML antes de pontuar. |

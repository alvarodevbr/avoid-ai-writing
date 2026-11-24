# Prova: o detector desta skill executado contra a própria escrita desta skill

Uma ferramenta que sinaliza "desbravar" ou "delve" no seu rascunho deve sobreviver à sua própria passagem. Esta página é o resultado da execução de `detector/patterns.js` sobre a documentação do repositório, incluindo os achados que não são lisonjeiros.

Reproduza em um comando, sem dependências:

```bash
git clone https://github.com/alvarodevbr/avoid-ai-writing && cd avoid-ai-writing
node scripts/self-scan.js
```

## Resultado (v3.34.0, medido em 2026-09-13)

| Documento | Palavras | Pontuação bruta (Raw) | Pontuação com isenção (Exempt) | Orçamento (Budget) |
|---|---:|---:|---:|---:|
| `README.md` | 5,273 | 58 | **1** | 30 |
| `SKILL.full.md` | 18,136 | 77 | **9** | 25 |
| `CONTRIBUTING.md` | 1,345 | 1 | **1** | 15 |
| `detector/README.md` | 1,174 | 1 | **1** | 15 |
| `detector/CATEGORIES.md` | 1,615 | 1 | **0** | 15 |
| `examples/README.md` | 931 | 1 | **1** | 10 |
| `CHANGELOG.md` | 11,915 | 57 | **34** | 40 |
| `PROOF.md` | 1,163 | 13 | **12** | 20 |

A linha `PROOF.md` é um instantâneo e levemente autorreferencial: editar esta página altera sua própria contagem de palavras e pontuação. A CI valida o número ao vivo a partir de `scripts/self-scan.js`, não do texto da tabela.

A pontuação vai de 0 a 100, onde 0 é limpo. Duas colunas são publicadas porque publicar apenas a mais favorável é exatamente o comportamento que este projeto existe para criticar.

**Raw (Bruta)** conta cada ocorrência, incluindo a tabela de vocabulário de 112 entradas e cada padrão que este repositório cita para alertar sobre ele. `SKILL.full.md` pontuar 77 na bruta significa que o catálogo contém as palavras que cataloga.

**Exempt (Com isenção)** aplica a válvula de escape de autorreferência que o `SKILL.md` sempre documentou em prosa: *"exemplos citados estão isentos de sinalização. Textos entre aspas, blocos de código ou explicitamente marcados como ilustrativos não devem ser reescritos."* `applyExemptions()` em `scripts/self-scan.js` é a sua forma executável: ela limpa blocos de código cercados, código inline, tabelas, citações em bloco e trechos entre aspas, e então pontua o que resta.

Os orçamentos (*budgets*) controlam a coluna com isenção na CI. Eles são tetos de regressão definidos a partir de valores medidos com alguns pontos de margem.

## O que a varredura encontrou em nossa própria escrita

Quatro resultados que vale nomear em vez de esconder.

**1. `CHANGELOG.md` tem a pior pontuação, e cerca de metade disso está fora do alcance da isenção.** Um changelog que anuncia uma nova regra lista as palavras que essa regra captura, sem aspas e separadas por vírgula. O detector lê uma sequência de vocabulário do Tier 1. Um leitor lê uma nota de lançamento. A isenção só alcança textos entre aspas ou em blocos de código.

**2. A regra de travessão inicialmente não implementava sua própria exceção para dois formatos.**
O `SKILL.md` isenta o separador de item de lista (`- **Termo** — descrição`) como tipografia em vez de pontuação de prosa, e `detector/patterns.js` implementa exatamente isso. Em versões anteriores, não isentava dois formatos quase idênticos:

- Cabeçalhos de versão do Keep-a-Changelog: `## [3.21.0] — 2026-07-30`.
- Um item com marcador cujo termo em negrito trazia um parêntese ou código antes do travessão: ``- **Lingering-attention claims** (`lingering-attention`) — the share-post frame…``

**Corrigido** na [#67](https://github.com/alvarodevbr/avoid-ai-writing/issues/67): ambos os formatos agora são exceções reconhecidas pelo detector.

**3. Esta página está na tabela e pontua acima de zero pelo motivo que acabou de descrever.** Qualquer menção em itálico sem aspas ou código não entra na máscara de isenção.

**4. `README.md` cai drasticamente da pontuação bruta para a isenta.** A maioria dos achados brutos vem do catálogo de padrões e dos exemplos que a página cita para documentação.

## O que esta página não afirma

Uma pontuação baixa significa que o texto não carrega acertos superficiais de padrões. Não significa que a escrita seja boa, nem que seja humana. O detector mede um subconjunto detectável por regex do `SKILL.md` mais alguns sinais estilométricos.

Uma taxa de falsos positivos foi medida e está documentada em [`corpus/README.md`](corpus/README.md) contra 875 parágrafos humanos e 779 parágrafos de máquina: com `score >= 5`, 4,2% de FPR (IC 95% 3,1–5,8) contra 7,2% de TPR. O ROC-AUC em nível de parágrafo é 0,501 agrupado — cara ou coroa — e 0,623 em nível de documento. Lido de forma direta, a pontuação composta não consegue separar de forma confiável o texto de máquina do texto humano.

## Na CI

`.github/workflows/detector-test.yml` executa `node scripts/self-scan.js --check` em cada push que toca o detector ou a documentação. Um documento que ultrapasse seu orçamento falha o build.

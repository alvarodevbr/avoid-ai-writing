# Exemplos de configuração de estilo da casa (House-style config examples)

`--style` adiciona um estilo da casa (*house style*) sobre a passagem de remoção de IA. Não é um registro de guias: aplica diretrizes de **registro/voz** e remove sinais de IA, além de qualquer **mecânica** que você exigir. A forma preferida de uso é um **arquivo de configuração** (`--style ./house.json`, ou um nome simples que corresponda a `examples/<name>.json`): ele é aplicado e o subconjunto verificável de suas mecânicas é validado deterministicamente. Os arquivos aqui são *exemplos desse formato*; copie um e edite-o. Eles também servem como fixtures de teste: `scripts/check-style.test.js` resolve nomes simples de `--config` neste diretório e valida seu conteúdo, portanto editar ou remover qualquer um dos arquivos JSON quebra `npm test`.

## Onde vivem os guias codificados

Para um guia publicado real, veja a seção [**House style is a different job**](https://github.com/alvarodevbr/avoid-ai-writing/blob/main/README.md#house-style-is-a-different-job) do README, que aponta para o [Vale](https://github.com/vale-cli/vale) e registra a decisão de licenciamento na [#88](https://github.com/alvarodevbr/avoid-ai-writing/issues/88).

**Este repositório não inclui guias de estilo proprietários.** Os arquivos de exemplo são genéricos e neutros em relação a guias.

## Esquema (Schema)

Uma configuração é um arquivo JSON com duas partes:

```json
{
  "name": "Meu estilo editorial",
  "genre": "documentação técnica",
  "register": [
    "Segunda pessoa, voz ativa, tempo presente.",
    "Sem exagero promocional."
  ],
  "mechanics": {
    "quotes": "straight",
    "headings": "sentence",
    "emDash": "sparing",
    "latinAbbrev": "parentheses",
    "serialComma": true,
    "spellNumbersUpTo": 9
  }
}
```

- **`register`** (lista de strings) — diretrizes de voz/registro que o modelo aplica como orientação.
- **`genre`** (string, opcional) — para qual gênero a configuração foi escrita.
- **`mechanics`** (objeto) — regras de saída, cujo subconjunto verificável é validado por `node scripts/check-style.js <file> --config <config.json>`:

| chave | valores | como é verificada |
|---|---|---|
| `quotes` | `straight` \| `curly` | **hard** — sinaliza o formato errado de aspas na prosa |
| `latinAbbrev` | `never` \| `parentheses` \| `any` | **hard** — `never` sinaliza qualquer `e.g.`/`i.e.`; `parentheses` sinaliza fora de parênteses; `any` não é verificado |
| `headings` | `sentence` \| `title` | consultivo (*advisory*) — nomes próprios tornam a capitalização de frase vs. título ambígua |
| `emDash` | `sparing` \| `deliberate` \| `never` | `never` é **hard** e sinaliza qualquer travessão na prosa; `sparing` é consultivo (~1 por 1.000 palavras) |
| `colons` | `never` \| `any` | **hard** — `never` sinaliza dois-pontos na prosa (exceto horários, proporções, esquemas de URL e tabelas) |
| `semicolons` | `never` \| `any` | **hard** — `never` sinaliza ponto e vírgula na prosa (entidades HTML válidas são isentas) |
| `spellNumbersUpTo` | número | consultivo — sinaliza numerais iguais ou abaixo do limite na prosa |
| `serialComma` | `true` \| `false` | aplicado apenas pelo modelo; não verificado por máquina |

## Normalize quote marks after a rewrite

Os modos `rewrite` e `edit` executam esta passagem antes da entrega. Mantenha o documento original como referência para que as aspas geradas não substituam a convenção existente:

```bash
node scripts/normalize-quotes.js draft.md --reference original.md
node scripts/normalize-quotes.js draft.md --reference original.md --write
node scripts/check-style.js draft.md --config ./house.json
```

O padrão `--quotes auto` infere aspas duplas e aspas simples/apóstrofos de forma independente a partir da prosa de referência desprotegida. A maioria de cada família vence; empates usam o primeiro estilo observado. Sem evidência, essa família permanece inalterada. Estilo explícito da casa tem precedência: use `--quotes straight` ou `--quotes curly` sem `--reference`.

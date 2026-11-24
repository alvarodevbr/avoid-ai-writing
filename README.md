<div align="center">

# avoid-ai-writing

Audite e reescreva conteúdos para remover padrões de escrita de IA ("IA-ismos"). Uma skill prática para qualquer agente de IA, adaptada para o Português Brasileiro (PT-BR) e inglês. Suporta modos somente detecção e edição local em arquivos, além de perfis de voz.

[![GitHub stars](https://img.shields.io/github/stars/alvarodevbr/avoid-ai-writing?style=social)](https://github.com/alvarodevbr/avoid-ai-writing/stargazers)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)
[![npm version](https://img.shields.io/npm/v/avoid-ai-writing-detector.svg)](https://www.npmjs.com/package/avoid-ai-writing-detector)
[![detector](https://github.com/alvarodevbr/avoid-ai-writing/actions/workflows/detector-test.yml/badge.svg)](https://github.com/alvarodevbr/avoid-ai-writing/actions/workflows/detector-test.yml)
[![Podcast](https://img.shields.io/badge/Podcast-Chain_of_Thought-purple?style=flat-square)](https://chainofthought.show/?utm_source=github&utm_medium=referral&utm_campaign=repo-readme&utm_content=avoid-ai-writing)
[![X](https://img.shields.io/badge/X-@alvarodevbr-black?style=flat-square&logo=x)](https://x.com/alvarodevbr)

<img src="docs/demo.gif" alt="O motor detector empacotado sinalizando 13 padrões de escrita de IA por categoria em um parágrafo de exemplo e pontuando a reescrita limpa com 0/100" width="800">
</div>

---


Uma skill de escrita portátil para [Claude Code](https://docs.anthropic.com/en/docs/claude-code), Codex, [OpenClaw](https://github.com/openclaw/openclaw), [Hermes](https://github.com/NousResearch/hermes-agent) e qualquer outro agente compatível com [agentskills.io](https://agentskills.io). Audita e reescreve conteúdos para remover padrões de escrita de IA ("IA-ismos" / "AI-isms").

As regras e o detector empacotado foram traduzidos e adaptados para os padrões idiomáticos do **Português Brasileiro (PT-BR)**, mantendo compatibilidade bilíngue com o inglês. Veja [adaptações de idiomas da comunidade](#community--multilingual) para versões mantidas de forma independente em outros idiomas.

**Três modos:**
- **Rewrite** (padrão) — sinaliza padrões de IA, aplica até duas passagens de edição, verifica o resultado quando possível e retorna uma versão final única.
- **Detect** — sinaliza padrões de IA sem reescrever. Mostra quais alertas são problemas reais vs. questões de julgamento. Útil quando os padrões podem ser intencionais, ao auditar conteúdo que você não quer alterar ou quando quer apenas uma varredura rápida.
- **Edit** — edita um arquivo de prosa diretamente no local (via ferramenta Edit) com alterações mínimas e direcionadas, preservando passagens que já soam humanas. Código-fonte, configuração e dados gerados são recusados porque reescritas de prosa podem corromper conteúdo estruturado. Retorna um relatório de edições feitas + verificação, não o arquivo inteiro.

Use `--iterate 1|2` para definir o teto de passagens de edição. `--iterate 1` permite apenas a reescrita inicial. `--iterate 2` usa o máximo padrão de uma reescrita inicial mais uma correção ou reparo de preservação, parando antes quando não restar edição justificada no escopo. As verificações não consomem uma passagem, mas qualquer alteração motivada por elas consome. A skill informa quantas passagens de edição usou e qualquer achado intencional, protegido ou não resolvido.

Um **perfil de voz** opcional (`casual` / `professional` / `technical` / `warm` / `blunt`) define como a prosa deve soar, independentemente do perfil de contexto do público.

O [glossário](GLOSSARY.md) define os termos usados nesta documentação: camadas de vocabulário, níveis de gravidade, modos, perfis e opções do detector.

## Sumário
- [Quick demo](#quick-demo)
- [Por que uma skill, não apenas um prompt](#por-que-uma-skill-não-apenas-um-prompt)
- [Installation & Usage](#installation--usage)
    * [Instalar o diretório completo da skill](#instalar-o-diretório-completo-da-skill)
    * [Claude Code](#claude-code)
    * [Claude Cowork — instalar como plugin](#claude-cowork--instalar-como-plugin)
    * [Google Antigravity](#google-antigravity)
    * [OpenClaw](#openclaw)
    * [Cursor](#cursor)
    * [Hermes](#hermes)
    * [OpenAI Codex](#openai-codex)
    * [Pacote nativo de plugin para ChatGPT e Codex](#pacote-nativo-de-plugin-para-chatgpt-e-codex)
    * [Other agents](#other-agents)
    * [Acionando a skill](#acionando-a-skill)
- [Referência de padrões](#referência-de-padrões)
    * [Padrões de Conteúdo](#padrões-de-conteúdo)
    * [Padrões de Linguagem](#padrões-de-linguagem)
    * [Padrões de Estrutura](#padrões-de-estrutura)
    * [Padrões de Comunicação](#padrões-de-comunicação)
    * [Metapadrões](#metapadrões)
    * [Detecção Estrutural (v3.4)](#detecção-estrutural-v34)
    * [Impressões digitais de ferramentas de IA e adições posteriores (v3.5–3.8)](#impressões-digitais-de-ferramentas-de-ia-e-adições-posteriores-v3538)
    * [Padrões de registro conversacional (v3.15)](#padrões-de-registro-conversacional-v315)
    * [Molduras de posts de compartilhamento (v3.20)](#molduras-de-posts-de-compartilhamento-v320)
    * [Franqueza narrada (v3.21)](#franqueza-narrada-v321)
    * [Hifenização desnecessária (v3.24)](#hifenização-desnecessária-v324)
- [Exemplo Completo (PT-BR)](#exemplo-completo-pt-br)
- [Executando o detector](#executando-o-detector)
    * [Pontuar um arquivo ou texto via pipe na linha de comando](#pontuar-um-arquivo-ou-texto-via-pipe-na-linha-de-comando)
    * [Validar prosa no GitHub Actions ou pre-commit](#validar-prosa-no-github-actions-ou-pre-commit)
    * [Usar o detector via MCP](#usar-o-detector-via-mcp)
- [Pipeline de pré-publicação](#pipeline-de-pré-publicação)
- [House style is a different job](#house-style-is-a-different-job)
- [Mais projetos](#mais-projetos)
- [Créditos](#créditos)
- [Community / Multilingual](#community--multilingual)
- [Fontes oficiais e patrocínio](#fontes-oficiais-e-patrocínio)
- [Aviso legal](#aviso-legal)
- [Licença](#licença)

## Quick demo

**Input:**
> Certainly! Acme Analytics, a vibrant startup nestled in the heart of Boulder's thriving tech ecosystem, has secured $40M in Series B funding — marking a watershed moment for the observability landscape. The platform serves as a unified hub, featuring real-time dashboards, boasting sub-second queries, and presenting a seamless integration layer. Moreover, experts believe Acme is poised to disrupt the market. In conclusion, the future looks bright!

**Final rewrite:**
> Acme Analytics, a Boulder-based startup, raised a $40M Series B. Its observability platform has real-time dashboards, runs queries in under a second, and includes an integration layer.

*(Equivalente em PT-BR: "A Acme Analytics, startup de Boulder, captou US$ 40 milhões na Série B. Sua plataforma de observabilidade tem painéis em tempo real, executa consultas em menos de um segundo e inclui uma camada de integração.")*

**Verification:** Uma passagem de edição. A revisão não encontrou outra edição justificada no escopo; nenhuma verificação determinística de preservação rodou nesta demonstração somente de prosa.

**O que foi capturado:** a abertura de chatbot ("Certainly!" / "Certamente!"), modificadores promocionais ("vibrante", "aninhada no coração de"), importância inflada ("marco divisor de águas"), verbos indiretos ("serve como", "conta com", "ostenta"), atribuição vaga ("especialistas acreditam") e a conclusão genérica ("o futuro parece brilhante!"). A reescrita mantém a captação, a localização e as três capacidades do produto, removendo a previsão de mercado sem suporte e sem inventar um investidor ou mecanismo de integração.

## Por que uma skill, não apenas um prompt

Um prompt único do tipo "faça isso soar humano" pega apenas o óbvio. Esta skill é diferente:

- **Reescrita final única** — conclui sua auditoria, correção e verificação disponível antes de retornar uma versão completa. Um breve resumo das alterações segue quando útil; uma auditoria detalhada permanece disponível mediante solicitação sem adicionar um rascunho concorrente.
- **Edição em duas passagens** — a revisão pode capturar padrões que sobrevivem à primeira edição, enquanto um limite compartilhado cobre tanto edições corretivas quanto reparos de preservação.
- **Entrega consciente da superfície** — verifica como o destino renderiza quebras de linha simples antes de retornar a prosa, desfazendo quebras de linha forçadas acidentais apenas quando elas ficariam visíveis, e preserva estruturas intencionais como listas, poesia, código e endereços.
- **112-entry word replacement table across 3 tiers + 10 Tier 3 phrases** — não baseada em "vibes". Cada palavra sinalizada tem uma alternativa específica e mais direta. "Alavancar" (`leverage`) → "usar" (`use`). "Utilizar" (`utilize`) → "usar" (`use`). Ocorrências do Tier 1 são sinalizadas a menos que uma exceção listada se aplique, palavras do Tier 2 são sinalizadas quando se agrupam no mesmo parágrafo, e palavras do Tier 3 são sinalizadas apenas em alta densidade. O Tier 1 se divide em **marcadores de frequência 1A** (`delve`/`desbravar`, `tapestry`/`tapeçaria`) e **edições de clareza 1B** (`in order to`/`a fim de`, `utilize`/`utilizar`) — mesmo conserto, mas apenas 1A é evidência sobre como uma passagem foi produzida, e 1B tem peso menor para que uma correção de prolixidade não empurre um documento para uma classificação de IA. As *frases* do Tier 3 (clichês de várias palavras como "a integração de", "computação descentralizada") são sinalizadas por repetição ou quando 3+ frases distintas se acumulam no mesmo texto.
- **74 pattern categories** — exemplos representativos abaixo, cada um com antes/depois. Inclui detecção estrutural (excesso de hashtags, listas de tópicos com sintagmas nominais puros, previsões com acúmulo de ressalvas), impressões digitais de ferramentas de IA (placeholders, marcação de citação, parâmetros UTM), verificações de ritmo/uniformidade, sinais de registro conversacional e testes do lado do autor. O catálogo completo vive em [`references/patterns.md`](./references/patterns.md); essa contagem é verificada na CI.
- **Modo Detect** — sinaliza padrões sem reescrever. Veja quais alertas são problemas reais vs. questões de julgamento.
- **Funciona em várias plataformas** — uma skill baseada em diretório roda no Claude Code, Cowork (como plugin), Google Antigravity (como plugin), OpenClaw, Cursor (como regra portada) e outros agentes. Veja os caminhos de instalação abaixo.

## Installation & Usage

### Instalar o diretório completo da skill

Use a instalação via plugin abaixo ou clone o repositório no diretório de skills do seu agente. Mantenha `SKILL.md` junto com `references/patterns.md`: o arquivo de entrada carrega o catálogo antes de auditar. Os diretórios `scripts/`, `detector/` e `examples/` fornecem verificação mecânica opcional.

Para um campo de regras de arquivo único, use [`dist/avoid-ai-writing.md`](./dist/avoid-ai-writing.md) (aproximadamente 150 KB; meça com `wc -c`). Ele inclui todas as regras e perfis, com alternativas manuais para comandos indisponíveis fora do pacote. Alguns campos de regras e instruções comportam muito menos do que isso, portanto verifique os limites de tamanho em [Other agents](#other-agents) antes de colar. Não copie apenas o arquivo `SKILL.md` enxuto sozinho.

### Claude Code

**Opção 1: Clonar no diretório de skills**

```bash
git clone https://github.com/alvarodevbr/avoid-ai-writing ~/.claude/skills/avoid-ai-writing
```

Depois invoque com `/avoid-ai-writing <seu texto>`.

**Opção 2: Copiar um arquivo autossuficiente**

Baixe `dist/avoid-ai-writing.md` e coloque-o em qualquer diretório que o Claude Code possa ler. Referencie-o no seu `CLAUDE.md`:

```markdown
- Edição para padrões de IA → leia `caminho/para/avoid-ai-writing.md`
```

**Opção 3: Usar como slash command**

Alias opcional. Crie um arquivo de comando (ex.: `~/.claude/commands/clean-ai-writing.md`):

```markdown
---
description: Auditar e reescrever conteúdo para remover padrões de escrita de IA
---

$ARGUMENTS

Leia e siga as instruções em ~/.claude/skills/avoid-ai-writing/SKILL.md
```

Depois use `/clean-ai-writing <seu texto>` no Claude Code.

### Claude Cowork — instalar como plugin

O [Cowork](https://claude.com/product/cowork) carrega skills apenas de **plugins instalados**. Este repositório funciona como um [marketplace](https://code.claude.com/docs/en/plugin-marketplaces) de plugin único:

```bash
/plugin marketplace add alvarodevbr/avoid-ai-writing
/plugin install avoid-ai-writing@alvarodevbr-skills
/reload-plugins   # ou reinicie a sessão para ativar a skill
```

### Google Antigravity

Com a [Antigravity CLI](https://antigravity.google/docs/plugins?tab=cli) instalada, clone este repositório e instale o diretório do plugin empacotado:

```bash
git clone https://github.com/alvarodevbr/avoid-ai-writing
cd avoid-ai-writing
agy plugin validate plugins/avoid-ai-writing
agy plugin install ./plugins/avoid-ai-writing
agy plugin list
```

O manifesto do Antigravity fica em `plugins/avoid-ai-writing/.antigravity-plugin/plugin.json`; instale `plugins/avoid-ai-writing` em vez da raiz do repositório.

### OpenClaw

**Opção 1: [Instalar via ClawHub](https://clawhub.ai/alvarodevbr/avoid-ai-writing)**

```bash
clawhub install avoid-ai-writing
```

**Opção 2: Clonar no diretório de skills**

```bash
git clone https://github.com/alvarodevbr/avoid-ai-writing ~/.openclaw/skills/avoid-ai-writing
```

### Cursor

Coloque a regra portada em `.cursor/rules/` do seu projeto:

```bash
mkdir -p .cursor/rules
curl -o .cursor/rules/avoid-ai-writing.mdc \
  https://raw.githubusercontent.com/alvarodevbr/avoid-ai-writing/main/cursor-rules/avoid-ai-writing.mdc
```

Veja [`cursor-rules/README.md`](./cursor-rules/README.md) para globs de ativação e frases de gatilho.

### Hermes

```bash
git clone https://github.com/alvarodevbr/avoid-ai-writing ~/.hermes/skills/writing/avoid-ai-writing
```

### OpenAI Codex

O Codex lê [Agent Skills](https://developers.openai.com/codex/skills) no mesmo formato `SKILL.md`:

```bash
git clone https://github.com/alvarodevbr/avoid-ai-writing .agents/skills/avoid-ai-writing
```

### Pacote nativo de plugin para ChatGPT e Codex

Publicado no [Diretório de Plugins da OpenAI](https://chatgpt.com/plugins/plugins_6a9b77b18b8881918efa9c1255868164). O [pacote de plugin](./OPENAI_PLUGIN.md) mantém o `SKILL.md` canônico como autoridade editorial e fornece sete Skills focadas:

- `avoid-ai-writing` — usa as instruções canônicas de auditoria e reescrita
- `avoid-ai-writing-router` — roteia pedidos mistos e de múltiplas etapas
- `ai-writing-detector` — executa o detector determinístico empacotado
- `voice-preserving-rewriter` — reescreve o texto retornado preservando a voz
- `file-edit-in-place` — edita apenas arquivos explicitamente nomeados
- `preservation-verifier` — verifica significado e restrições antes e depois de uma reescrita
- `false-positive-reviewer` — revisa achados do detector que precisam de contexto

A partir de um clone local, construa e valide o pacote com:

```bash
python3 scripts/package-openai-plugin.py . /tmp/avoid-ai-writing.zip --json
python3 scripts/validate-openai-plugin.py . --json
```

### Other agents

Use o destino abaixo para sua ferramenta. Meça o arquivo baixado com `wc -c dist/avoid-ai-writing.md` para obter a contagem exata de bytes UTF-8.

| Ferramenta | Onde colocar | Limite de tamanho |
|------|-----------------|------------|
| **Windsurf** | clone o diretório da skill em `.devin/skills/avoid-ai-writing/` (`.windsurf/skills/avoid-ai-writing/` também é suportado) | Arquivos de regras de workspace são limitados a 12.000 caracteres, portanto o arquivo único não cabe em `.windsurf/rules/` |
| **Cline** | `.clinerules/avoid-ai-writing.md` | Nenhum documentado; regras consomem tokens de contexto em cada tarefa |
| **GitHub Copilot** (VS Code) | cole em `.github/copilot-instructions.md` | Nenhum documentado |
| **Claude.ai Projects** | cole `dist/avoid-ai-writing.md` nas instruções personalizadas do projeto | Nenhum documentado |
| **ChatGPT** | instale o pacote de plugin nativo ou envie `dist/avoid-ai-writing.md` como arquivo de Conhecimento (Knowledge) de um Custom GPT | O campo Instructions de um Custom GPT comporta muito menos que o arquivo; não cole lá |

### Acionando a skill

Uma vez instalada, peça ao seu assistente para limpar a escrita de IA:

- "Remova os IA-ismos deste post" ("Remove AI-isms from this post")
- "Audite este rascunho em busca de sinais de IA" ("Audit this draft for AI tells")
- "Deixe este texto com menos cara de IA" ("Make this sound less like AI")
- "Limpe a escrita de IA neste parágrafo" ("Clean up AI writing in this paragraph")

No **modo rewrite** (padrão), a skill conclui sua revisão e verificação disponível antes de retornar o texto reescrito exatamente uma vez:

1. **Final rewrite** — a versão final verificada ou honestamente qualificada
2. **Changes** — um breve resumo quando edições foram feitas e o resumo é útil
3. **Verification** — passagens de edição usadas, verificações que rodaram, motivo de parada e quaisquer achados intencionais, protegidos, bloqueados pela fonte ou não resolvidos

No **modo detect**, a skill retorna duas seções:

1. **Issues found** — cada IA-ismo identificado, agrupado por gravidade (P0/P1/P2)
2. **Assessment** — quais alertas são problemas claros vs. padrões que podem ser intencionais ou eficazes no contexto

## Referência de padrões

> Exemplos representativos do catálogo — não a lista exaustiva (que está em [`references/patterns.md`](./references/patterns.md)). O catálogo de prosa voltado para humanos e o [motor detector](./detector/) usam **contagens diferentes de propósito**: the engine implements 54 `type` categories porque divide as camadas de vocabulário e adiciona sinais estilométricos/de impressão digital (distribuição de pontuação, entropia de palavras funcionais, detecção de truques de bypass) que funcionam como matemática sobre um documento em vez de uma regra de consulta manual. Os dois são mapeados em [`detector/CATEGORIES.md`](./detector/CATEGORIES.md); não tente forçar uma contagem a igualar a outra.

### Padrões de Conteúdo

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 1 | **Inflação de importância** | "marcando um momento crucial na evolução de..." | Declare o evento fundamentado; não invente data ou propósito |
| 2 | **Desfile de citações (Name-dropping)** | "citado no NYT, BBC, Folha e Wired" | Mantenha citações fundamentadas; remova alegações de que provam importância |
| 3 | **Análises superficiais no gerúndio** | "simbolizando... refletindo... evidenciando..." | Substitua por fatos específicos ou corte |
| 4 | **Linguagem promocional** | "aninhado na deslumbrante região" | Mantenha a localização fornecida; remova modificadores promocionais |
| 5 | **Atribuições vagas** | "Especialistas acreditam que desempenha papel crucial" | Peça a fonte ou sinalize a afirmação para verificação |
| 6 | **Desafios padronizados** | "Apesar dos desafios... continua a prosperar" | Nomeie o desafio e a resposta |
| 7 | **Inflação de novidade** | "Ele introduziu um termo que ninguém está nomeando" | Mantenha a observação pessoal se importar; não invente novidade |

### Padrões de Linguagem

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 8 | **Substituição de palavras/frases (3 camadas)** | "alavancar... robusto... fluido... utilizar" | "usar... confiável... simples... usar" |
| 9 | **Fuga do verbo de ligação (cópula)** | "serve como... conta com... ostenta" | "é... tem" |
| 10 | **Rodízio forçado de sinônimos** | "desenvolvedores... engenheiros... profissionais... construtores" | "desenvolvedores" (repita o termo claro) |
| 11 | **Frases de template** | "um passo [adj] em direção a uma infraestrutura [adj]" | Descreva o resultado específico |
| 12 | **Expressões de enchimento (Fillers)** | "A fim de", "Devido ao fato de que" | "Para", "Porque" |
| 13 | **Falsos intervalos** | "do Big Bang à matéria escura" | Liste os tópicos reais |
| 14 | **Ressalvas entre parênteses** | "(e, cada vez mais, X)" | Nomeie em frase própria ou corte |

### Padrões de Estrutura

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 15 | **Formatação** | Travessões (— e --), excesso de negrito, emojis em títulos, excesso de bullets | Vírgulas/pontos, parágrafos em prosa |
| 16 | **Estrutura de frase** | "Não é X, é Y" + intensificadores vazios + ressalvas | Afirmações positivas diretas |
| 17 | **Problemas estruturais** | Parágrafos uniformes, aberturas padronizadas, gramática artificialmente limpa | Comprimento variado, comece pelo ponto |
| 18 | **Expressões de transição** | "Ademais,", "Outrossim,", "No [X] de hoje" | "e", "também" ou reestruture |
| 19 | **Listas com cabeçalho repetido** | "**Velocidade:** A velocidade melhorou em..." | Escreva o ponto diretamente |
| 20 | **Títulos em Title Case** | "Negociações Estratégicas E Parcerias Principais" | "Negociações estratégicas e parcerias principais" |
| 21 | **Inflação de listas numeradas** | "Aqui estão 7 razões pelas quais..." | Corte para as 2-3 que importam |
| 22 | **Estrutura de falsa concessão** | "Embora X seja impressionante, desafios permanecem" | Declare a compensação (*tradeoff*) real |
| 23 | **Aberturas com pergunta retórica** | "Mas o que isso significa para os desenvolvedores?" | Comece pela afirmação |

### Padrões de Comunicação

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 24 | **Artefatos de chatbot** | "Espero que isso ajude! Avise se precisar..." | Remova completamente |
| 25 | **Construções com "Vamos"** | "Vamos explorar", "Vamos destrinchar isso" | Comece direto pelo ponto |
| 26 | **Avisos de corte de conhecimento** | "Até minha última atualização..." | Encontre as fontes ou remova |
| 27 | **Conclusões genéricas** | "O futuro parece brilhante", "Só o tempo dirá" | Pensamento final específico ou corte |
| 28 | **Moldura de reação padronizada** | "O que mais me surpreendeu", "Fiquei fascinado ao descobrir" | Mantenha reações autênticas; corte molduras vazias |
| 29 | **Artefatos de cadeia de raciocínio** | "Deixe-me pensar passo a passo", "Dividindo isso em partes" | Declare a conclusão e depois a evidência |
| 30 | **Tom bajulador (Sycophancy)** | "Ótima pergunta!", "Você tem toda razão!" | Remova completamente |
| 31 | **Loops de reconhecimento** | "Você está perguntando sobre", "Para responder à sua pergunta" | Responda diretamente |
| 32 | **Calibração de confiança** | "Vale notar que", "Curiosamente,", "Surpreendentemente," | Deixe o fato falar por si |

### Metapadrões

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 33 | **Estrutura excessiva** | 5 títulos em 200 palavras, "Visão geral:", "Pontos principais:" | Funda seções, use títulos específicos |
| 34 | **Ritmo e uniformidade** | Todas as frases com 15–25 palavras, parágrafos do mesmo tamanho | Misture curtas/longas, fragmentos, perguntas |
| 35 | **Polimento excessivo** | Cada irregularidade lixada, prosa perfeitamente uniforme | Mantenha disfluência natural e ritmo variado |
| 36 | **Limite entre reescrever do zero e remendar** | 5+ alertas de vocabulário + 3+ categorias + ritmo uniforme | Recomende reescrita estrutural quando autorizado |

### Detecção Estrutural (v3.4)

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 37 | **Frases do Tier 3 (clichês compostos)** | "a integração de", "computação descentralizada", "impulsionado pela comunidade", "sustentabilidade a longo prazo" empilhados | Substitua a frase repetida por uma afirmação específica |
| 38 | **Fechamentos de narrativa futura** | "pode se tornar uma das narrativas mais importantes do próximo ciclo" | Escolha uma versão falseável com dados da fonte |
| 39 | **Previsões com acúmulo de ressalvas** | "poderia potencialmente criar", "pode eventualmente destravar" | Escolha apenas um qualificador |
| 40 | **Inflação de adjetivos "real/verdadeiro"** | "tokenomics real on-chain", "sustentabilidade efetiva de recompensas" | Remova o intensificador vazio (exceto quando o contraste for nomeado) |
| 41 | **Excesso de hashtags** | Bloco final com 15 tags: `#IA #Crypto #Web3 #Inovacao #FutureTech…` | No máximo 2-3 tags específicas, ou nenhuma |
| 42 | **Listas de tópicos com sintagmas nominais puros** | `* Eficiência estável / Conectividade confiável / Desempenho otimizado / Baixas taxas de falha / Utilização eficaz / Estabilidade térmica` | Converta em prosa ou reescreva cada item como afirmação completa |

### Impressões digitais de ferramentas de IA e adições posteriores (v3.5–3.8)

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 43 | **Placeholders não preenchidos** | `[Seu Nome]`, `[INSERIR FONTE]`, `2025-XX-XX` | Preencha com conteúdo fornecido ou apague |
| 44 | **Marcação de citação de chatbot** | `citeturn0search0`, `oai_citation`, `contentReference[oaicite:0]` | Remova o token de marcação completamente |
| 45 | **Parâmetros de URL de ferramentas de IA** | `utm_source=chatgpt.com`, `utm_source=copilot.com` | Remova o parâmetro de rastreamento; mantenha a URL se o link importar |
| 46 | **Preenchimento especulativo de lacunas** | "mantém um perfil discreto", "provavelmente iniciou sua carreira" | Corte o palpite ou substitua por fato com fonte |
| 47 | **Empilhamento de modificadores hifenizados** | "uma solução de alta qualidade, bem-arquitetada e à prova de futuro" | Corte para o modificador que importa |
| 48 | **Ganchos de engajamento estilo infomercial** | "O pulo do gato?", "A pegadinha?", "A questão é a seguinte." | Apague o gancho, declare o fato |
| 49 | **Diversidade de vocabulário (baixo TTR)** | Vocabulário estreito e repetitivo ao longo de 200+ palavras | Amplie os fatos específicos citados da fonte |
| 50 | **Importância autorrotulada** | "Esse último movimento é o contraintuitivo", "Esta é a parte interessante" | Corte o rótulo; deixe a explicação sustentar o peso |
| 51 | **Ponto final em rótulos de lista** | `- **Introduções.** Anos de conferências e rede de operadores.` | Use dois-pontos em vez de ponto: `- **Introduções:** anos de conferências...` |

### Padrões de registro conversacional (v3.15)

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 52 | **Respostas em bloco compacto (Wall-of-text)** | Uma resposta de 4+ frases e menos de 150 palavras entregue como um único parágrafo sem quebras de linha | Quebre nas fronteiras de pensamento |
| 53 | **Abertura bajuladora com recapitulação** | "Obrigado por todo o trabalho pesado aqui — o script de migração e o plano de rollback que você fez tornaram isso possível." | Vá direto à substância; mantenha o agradecimento curto sem recapitular |

### Molduras de posts de compartilhamento (v3.20)

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 54 | **Alegações de atenção persistente** | "A frase à qual sempre volto:", "Não consigo parar de pensar nisso" | Abra direto na ideia (exceto quando acompanhada do motivo explicativo) |

### Franqueza narrada (v3.21)

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 55 | **Franqueza narrada** | "Duas ressalvas que prefiro sinalizar agora do que deixar você descobrir depois:", "Quero ser transparente:" | Declare as ressalvas diretamente |

### Hifenização desnecessária (v3.24)

| # | Padrão | Antes | Depois |
|---|---------|--------|-------|
| 56 | **Hifenização desnecessária** | "research-impact aggregator", "code-base", "in real-time", "works out-of-the-box" | "research impact aggregator", "codebase", "in real time", "works out of the box" |

Dois **testes** do lado do autor completam o catálogo (verificações de julgamento, não autodetectadas): **imunidade a embaralhamento de parágrafos** (você consegue trocar dois parágrafos do corpo sem quebrar o texto?) e o **efeito esteira / baixa densidade de informação** ("o que há de realmente novo neste parágrafo?").

## Exemplo Completo (PT-BR)

**Antes (gerado por IA):**

> Certamente! Aqui está uma visão geral abrangente da Série B da Acme.
>
> A Acme Analytics, uma startup vibrante aninhada no coração do próspero ecossistema tecnológico de Florianópolis, garantiu US$ 40 milhões em financiamento de Série B — marcando um divisor de águas para a empresa e para o cenário de observabilidade como um todo. A rodada foi liderada pela Sequoia, com participação da Andreessen Horowitz, Y Combinator e Index Ventures, ressaltando a robusta confiança dos investidores na visão da Acme.
>
> A plataforma serve como um hub unificado para equipes de engenharia, contando com painéis em tempo real, ostentando desempenho de consultas abaixo de um segundo e apresentando uma camada de integração perfeita. Não é apenas mais uma ferramenta de monitoramento — é uma mudança de paradigma na forma como as organizações navegam por sistemas distribuídos complexos. Especialistas acreditam que a Acme está posicionada para revolucionar o mercado de observabilidade de US$ 15 bilhões. Estudos mostram que empresas que utilizam soluções abrangentes de monitoramento conseguem apurar problemas 40% mais rápido.
>
> - 🚀 **Desempenho:** A plataforma otimiza a resposta a incidentes, empoderando engenheiros para resolver problemas mais rápido.
> - 💡 **Escala:** A Acme fomenta a colaboração entre equipes, de desenvolvedores frontend a profissionais de SRE, construtores de plataforma e engenheiros de infraestrutura.
> - ✅ **Adoção:** A adoção pelos clientes continua a acelerar, refletindo tendências mais amplas do setor.
>
> Ademais, a abordagem da Acme evidencia o potencial da observabilidade moderna — simbolizando uma mudança em direção ao monitoramento proativo, refletindo anos de investimento em P&D e destacando a intrincada interação entre coleta de dados e inteligência acionável. Apesar dos desafios comuns no disputado espaço de monitoramento, a Acme continua a prosperar. A fim de capitalizar totalmente esse impulso, a empresa planeja expandir seus empreendimentos de go-to-market.
>
> Em conclusão, o futuro parece brilhante para a Acme Analytics. Só o tempo dirá como esse divisor de águas vai remodelar o paradigma da observabilidade. Sinta-se à vontade para entrar em contato se quiser mais detalhes!

**Depois (limpo):**

> A Acme Analytics, startup de Florianópolis, captou US$ 40 milhões em uma Série B liderada pela Sequoia, com participação da Andreessen Horowitz, Y Combinator e Index Ventures.
>
> Sua plataforma de observabilidade oferece às equipes de engenharia painéis em tempo real, consultas que rodam em menos de um segundo e uma camada de integração. A ferramenta ajuda engenheiros a resolver incidentes mais rápido e apoia o trabalho conjunto entre equipes de frontend, SRE, plataforma e infraestrutura. A adoção por clientes está crescendo. A Acme investe em P&D há anos e planeja expandir sua operação de go-to-market em um mercado de monitoramento disputado.

**Afirmações que precisam de fontes:** O texto de entrada atribui uma previsão sobre um mercado de US$ 15 bilhões a especialistas não nomeados e uma melhoria de 40% na identificação de problemas a estudos não especificados. Verifique essas fontes antes de publicar as afirmações. A reescrita as deixa de fora e não as substitui por benchmarks, clientes ou planos inventados.

## Executando o detector

A skill inclui um motor de detecção determinístico e sem dependências em [`detector/`](./detector/) — o mesmo motor que as regras acima descrevem, como código executável. Ele roda em Node (`>=18`) e no navegador sem etapa de build.

```bash
npm install avoid-ai-writing-detector
```

```js
const AIDetector = require("avoid-ai-writing-detector");
const { score, label, issues } = AIDetector.analyzeText("Seu texto aqui…");
console.log(score, label, issues.length);
```

### Pontuar um arquivo ou texto via pipe na linha de comando

```bash
npx --package avoid-ai-writing-detector avoid-ai-writing draft.md
cat draft.md | npx --package avoid-ai-writing-detector avoid-ai-writing --context technical
```

### Validar prosa no GitHub Actions ou pre-commit

O repositório também fornece um gate determinístico que falha na **contagem de achados por arquivo**, não na pontuação composta de 0–100:

```yaml
# .github/workflows/prose.yml
steps:
  - uses: actions/checkout@v7
  - id: gate
    uses: alvarodevbr/avoid-ai-writing@v3.36.0
    with:
      glob: "**/*.md"
      threshold: "6"
      context: technical
```

Usuários de pre-commit podem instalar o hook do repositório:

```yaml
repos:
  - repo: https://github.com/alvarodevbr/avoid-ai-writing
    rev: v3.36.0
    hooks:
      - id: avoid-ai-writing
```

Para manter uma passagem fora da pontuação (como um exemplo de prosa de IA citado de propósito), envolva-a em marcadores de ignore:

```html
<!-- avoid-ai-writing:ignore-start -->
Não é apenas um índice de busca, é uma base de confiança.
<!-- avoid-ai-writing:ignore-end -->
```

Veja [`detector/README.md`](./detector/README.md) para a API completa do `analyzeText` e [`detector/CATEGORIES.md`](./detector/CATEGORIES.md) para o mapa regra ↔ categoria.

### Usar o detector via MCP

[`avoid-ai-writing-mcp`](https://github.com/alvarodevbr/avoid-ai-writing-mcp) encapsula o detector publicado como um servidor MCP stdio local:

```bash
claude mcp add avoid-ai-writing -- npx -y avoid-ai-writing-mcp@0.1.0
```

O motor também inclui um validador de preservação (`detector/validate.js`), que verifica conteúdo e estrutura protegidos:

```bash
node detector/validate.js --residual-policy warn before.md after.md
```

**Ele passa no próprio teste?** [`PROOF.md`](./PROOF.md) pontua a documentação deste repositório com o detector deste repositório e publica o resultado. `npm run self-scan` reproduz o teste, e a CI falha quando um documento ultrapassa seu orçamento.

## Pipeline de pré-publicação

Esta skill responde a uma pergunta: o rascunho soa como um modelo? Execute o [fact-check](https://github.com/alvarodevbr/fact-check?utm_source=github&utm_medium=referral&utm_campaign=repo-readme&utm_content=avoid-ai-writing) primeiro para responder à outra: as afirmações são verdadeiras?

```text
/plugin marketplace add alvarodevbr/avoid-ai-writing
/plugin install avoid-ai-writing@alvarodevbr-skills
/plugin install fact-check@alvarodevbr-skills
```

## House style is a different job

Esta skill remove sinais de escrita de IA. Aplicar um guia de estilo publicado é um trabalho diferente: ela não faz isso sozinha e não traz guias de estilo próprios embutidos. A entrada opcional `--style` aceita uma configuração de estilo da casa que você fornece: uma lista `register` que o modelo aplica, e um objeto `mechanics` cujas regras verificáveis o script `scripts/check-style.js` valida deterministicamente. [`examples/`](./examples/) contém o esquema.

A partir de um clone local, após uma reescrita, `node scripts/normalize-quotes.js draft.md --reference original.md` imprime as aspas da prosa normalizadas para a convenção do documento original; adicione `--write` para salvar. Veja o [uso e limites](./examples/README.md#normalize-quote-marks-after-a-rewrite).

Guias pagos (Chicago, APA, MLA, AP, ABNT) não possuem implementação legível por máquina aqui nem no Vale. Passar um de seus nomes para `--style` recorre ao conhecimento geral do modelo sem alegação de conformidade verificada. A [auditoria de licenças](https://github.com/alvarodevbr/avoid-ai-writing/issues/88) por trás dessa decisão é pública.

## Mais projetos

[Siga Alvaro Araujo no GitHub](https://github.com/alvarodevbr) para novas ferramentas e exemplos práticos:

- [What I'm building](https://alvarodevbr.com/builds/?utm_source=github&utm_medium=referral&utm_campaign=repo-readme&utm_content=avoid-ai-writing) — projetos públicos em skills de agentes, servidores MCP, ferramentas para criadores e Mojo.
- [Chain of Thought](https://chainofthought.show/?utm_source=github&utm_medium=referral&utm_campaign=repo-readme&utm_content=avoid-ai-writing) — conversas com quem está colocando sistemas de IA em produção.
- [repo-audit](https://github.com/alvarodevbr/repo-audit) — verifica se as afirmações de um repositório são aplicadas, consultivas ou orientativas.

## Créditos

Pesquisa de padrões informada por:
- Pesquisa de detecção de IA da [Pangram Labs](https://www.pangram.com/)
- Documentação [Signs of AI-generated text](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing) da Wikipédia
- Skill Claude Code [blader/humanizer](https://github.com/blader/humanizer)
- [brandonwise/humanizer](https://github.com/brandonwise/humanizer) — sistema de vocabulário em camadas e pesquisa de análise estatística
- Ecossistema de skills humanizer do [OpenClaw](https://github.com/openclaw/openclaw)

Autoria original de [Alvaro Araujo](https://github.com/alvarodevbr) · [LinkedIn](https://www.linkedin.com/in/alvarodevbr/) · [Podcast Chain of Thought](https://chainofthought.show/?utm_source=github&utm_medium=referral&utm_campaign=repo-readme&utm_content=avoid-ai-writing)

## Community / Multilingual

Adaptações mantidas pela comunidade são listadas aqui para descoberta:

- **[avoid-ai-writing-multilingual](https://github.com/jurigis/avoid-ai-writing-multilingual)** por [Jürgen Kraus](https://github.com/jurigis) — adaptações para alemão (`SKILL-DE.md`), francês (`SKILL-FR.md`), italiano (`SKILL-IT.md`), romeno (`SKILL-RO.md`) e sueco (`SKILL-SV.md`).
- **[evitar-escrita-ia](https://github.com/mariorabeloneto/evitar-escrita-ia)** por [Mário Rabelo Neto](https://github.com/mariorabeloneto) — adaptação para Português Europeu (PT-PT).

Construiu uma adaptação? Veja a [orientação de adaptação de idioma](CONTRIBUTING.md#language-adaptations).

## Fontes oficiais e patrocínio

Canais oficiais do projeto upstream:
- Este repositório, [`alvarodevbr/avoid-ai-writing`](https://github.com/alvarodevbr/avoid-ai-writing), incluindo seus [releases](https://github.com/alvarodevbr/avoid-ai-writing/releases)
- [ClawHub](https://clawhub.ai/alvarodevbr/avoid-ai-writing)
- [OpenAI Plugins Directory](https://chatgpt.com/plugins/plugins_6a9b77b18b8881918efa9c1255868164)
- npm: [`avoid-ai-writing-detector`](https://www.npmjs.com/package/avoid-ai-writing-detector) e [`avoid-ai-writing-mcp`](https://www.npmjs.com/package/avoid-ai-writing-mcp)

**Sem inserções pagas.** A skill, o catálogo de padrões, o detector e a documentação não contêm menções pagas, links patrocinados ou recomendações comerciais de produtos.

**Patrocínio.** Você pode apoiar o projeto através do [GitHub Sponsors](https://github.com/sponsors/alvarodevbr).

---

## Aviso legal

*Este é um projeto pessoal independente, não afiliado, patrocinado ou endossado por nenhuma empresa. Todas as opiniões expressas são do autor.*

## Licença

MIT

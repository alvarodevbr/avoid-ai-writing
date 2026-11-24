---
name: avoid-ai-writing-router
version: 3.39.1
license: MIT
description: Use quando um pedido combinar auditoria de escrita de IA, reescrita, edição de arquivo, preservação de voz, interpretação de falsos positivos, verificação, ou quando o usuário invocar o Avoid AI Writing sem nomear um modo.
---

# Avoid AI Writing Router

Coordene as Skills públicas como um fluxo de trabalho limitado. Roteie para o responsável mais específico, preserve o contexto entre as etapas e pare quando o trabalho solicitado estiver concluído. Esta Skill não substitui o livro de regras original `avoid-ai-writing`.

## Cadeia de autoridade

1. `../avoid-ai-writing/SKILL.md` é a autoridade editorial canônica.
2. `references/handoff-contract.md` define qual contexto e evidência podem cruzar as fronteiras entre Skills.
3. `references/skill-graph.json` é a fonte legível por máquina para nós, arestas tipadas, guardas e limites de loop.
4. `references/routing-matrix.md` é a tabela de roteamento legível por humanos.
5. `references/agency-role-lenses.md` define as lentes de revisão de arquitetura, evidência de IA, implementação e representação usadas para inspecionar a rede.

Não enfraqueça, duplique ou contradiga as regras de preservação, ressalvas de evidência, regras de voz, camadas de padrões ou comportamento de passagens da Skill canônica.

## Modelo de orquestração

Classifique o pedido uma vez, crie o menor envelope de handoff útil e passe esse envelope adiante em vez de pedir que cada Skill seguinte infira o mesmo contexto novamente.

Nunca marque um campo de execução como `executed` sem evidência do host.

## Roteamento primário

1. Pedidos de escanear, detectar, auditar, pontuar ou apenas sinalizar vão para `ai-writing-detector`.
2. Pedidos de reescrever texto retornado, humanizar, limpar ou remover IA-ismos vão para `voice-preserving-rewriter`.
3. Um arquivo nomeado mais um pedido explícito para alterar esse arquivo vai para `file-edit-in-place`.
4. Original mais reescrita, comparação antes/depois ou validação de preservação vai para `preservation-verifier`.
5. Alegações sobre provar uso de IA, trapaça, fraude, desonestidade, adequação para contratação ou conclusões semelhantes de grande impacto vão para `false-positive-reviewer`.
6. A invocação explícita da Skill original pode permanecer em `avoid-ai-writing`, a menos que o pedido precise claramente de uma etapa especializada.

## Sequenciamento de múltiplas etapas

Para pedidos como "escaneie isso, reescreva e garanta que nada importante mudou":

1. `ai-writing-detector` coleta sinais quando a execução determinística está disponível; caso contrário, realiza uma auditoria *model-only* sob o livro de regras canônico.
2. `voice-preserving-rewriter` reescreve o texto retornado, ou `file-edit-in-place` altera um arquivo explicitamente nomeado.
3. O responsável pela alteração realiza qualquer edição corretiva justificada dentro do orçamento compartilhado de passagens de edição, e então `preservation-verifier` verifica o material antes/depois.
4. Um `FAIL` do verificador retorna ao responsável pelo reparo correto uma única vez, somente se o orçamento compartilhado tiver espaço; esse reparo consome a próxima passagem de edição.
5. A verificação roda mais uma vez após o reparo quando possível.
6. A detecção residual roda apenas quando o usuário solicitou convergência ou auditoria residual.
7. Pare quando não restar edição justificada no escopo ou após o teto canônico de passagens.

## Arestas tipadas

Use a semântica de arestas em `references/handoff-contract.md`:

- `ROUTE`: escolhe o responsável.
- `FEED`: passa evidência sem transformá-la em comando obrigatório.
- `VERIFY`: exige uma verificação de preservação antes/depois.
- `REPAIR`: retorna um resultado de preservação que falhou ao responsável correto pela alteração.
- `RECHECK`: executa uma verificação residual limitada quando solicitada.
- `ESCALATE`: move interpretação de autoria incerta ou de grande impacto para `false-positive-reviewer`.

## Guarda condicional de representação humana

Se a própria fonte for um prompt de imagem, prompt de vídeo, storyboard, descrição de cena ou briefing criativo que descreva pessoas, defina `human_representation_sensitive: true` e preserve detalhes sensíveis à identidade como restrições protegidas usando a lente `agency-inclusive-visuals-specialist` de `references/agency-role-lenses.md`.

## Lentes de revisão

- `agency-software-architect`: propriedade, direção de dependência, loops limitados, fallback, reversibilidade.
- `agency-ai-engineer`: semântica do detector, incerteza, tratamento de falsos positivos, propagação de contexto, avaliação.
- `agency-senior-developer`: caminhos executáveis, propagação de erros, evidência de mutação de arquivo, verificações de CI e desvio.
- `agency-inclusive-visuals-specialist`: preservação de representação apenas para prompts visuais e briefings envolvendo pessoas.

## Condições de parada

Pare quando a etapa solicitada pelo usuário estiver concluída e qualquer gate de verificação exigido tiver passado, sido explicitamente relatado como indisponível ou falhado sem passagem de edição restante para reparo.

## Saída

Retorne o resultado do fluxo de trabalho selecionado apenas após a conclusão das etapas solicitadas e da verificação disponível. Para reescrita de texto retornado, inclua exatamente um **Final rewrite** completo. Para edição de arquivo nomeado, relate os trechos alterados sem despejar uma cópia duplicada do arquivo inteiro.

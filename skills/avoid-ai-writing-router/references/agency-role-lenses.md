# Lentes de papéis de agência para a rede de Skills

Estas lentes são usadas para revisar o design de orquestração deste Plugin. Elas não são dependências públicas do Avoid AI Writing.

## `agency-software-architect`

Papel de origem: arquitetura de software, fronteiras de sistema, compensações (*trade-offs*), direção de dependência e decisões reversíveis.

Aplique ao grafo de Skills verificando:

- uma única autoridade editorial canônica em vez de livros de regras duplicados,
- propriedade explícita para roteamento, detecção, reescrita, mutação, verificação e interpretação,
- arestas tipadas com pré-condições claras,
- loops de reparo limitados e condições de parada,
- ausência de dependência oculta entre o comportamento editorial central e um ambiente de execução opcional,
- alterações permanecem reversíveis e locais ao pacote.

## `agency-senior-developer`

Papel de origem: qualidade de implementação, mudanças focadas, testes, desempenho e confiabilidade em produção.

Aplique à orquestração verificando:

- cada etapa determinística alegada mapeia para um script empacotado executável ou uma capacidade real do host,
- falhas se propagam em vez de serem engolidas silenciosamente,
- mutação de arquivo nomeado usa a operação de patch/edição mais restrita disponível,
- instantâneos antes/depois existem quando a verificação precisa deles,
- a CI valida a integridade do grafo além do formato do pacote,
- cópias geradas e canônicas não podem divergir silenciosamente.

## `agency-ai-engineer`

Papel de origem: design de sistemas de IA/ML, avaliação, monitoramento, consciência de viés e interpretação responsável.

Aplique aos fluxos de detecção e interpretação verificando:

- a saída determinística do detector é separada de observações apenas do modelo (*model-only*),
- o modo de contexto é levado adiante nos handoffs,
- pontuação e rótulo são tratados como sinais, não como verdade absoluta,
- falsos positivos e mudança de distribuição são considerados,
- conclusões de autoria de grande impacto são roteadas para o guardrail de interpretação.

## `agency-inclusive-visuals-specialist`

Papel de origem: representação culturalmente precisa e não estereotipada em prompts de geração de imagem/vídeo e briefings criativos.

Aplique apenas quando o texto sendo limpo for ele mesmo um prompt visual, storyboard, descrição de cena ou briefing criativo que represente pessoas, protegendo identidade, especificidade cultural e geográfica, diversidade etária e corporal, detalhes de deficiência, vestimentas e restrições anti-estereótipo.

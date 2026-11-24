# Contrato de handoff entre Skills (Cross-skill handoff contract)

O roteador e as Skills especializadas trocam contexto por meio de um envelope de handoff compartilhado. O objetivo é preservar decisões e evidências entre as etapas para que cada Skill não reinterprete o pedido do zero.

Este é um contrato de orquestração, não um formato de saída voltado para o usuário. Mantenha o envelope o menor possível para a tarefa.

## Envelope de handoff

```yaml
intent: detect | rewrite | edit_file | verify | interpret | multi_stage
source_kind: pasted_text | named_file | before_after_pair | visual_prompt | other
source_ref: optional path or user-supplied label
requested_scope: optional user-authorized editing boundary
explicit_corrections: []
context_profile: optional linkedin | blog | technical-blog | investor-email | docs | casual
context_mode: general | technical
voice: optional casual | professional | technical | warm | blunt | user_sample
protected_constraints:
  - facts
  - numbers
  - urls
  - paths
  - code
  - quotes
  - tables
  - frontmatter
  - attributed_text
  - identity_and_cultural_specificity
execution_evidence:
  detector: not_run | model_only | executed
  mutation: not_requested | not_run | executed
  verifier: not_run | model_only | executed
detector_summary:
  score: optional
  label: optional
  issue_types: []
verification_summary:
  status: optional PASS | REVIEW | FAIL
  blocking_errors: []
  warnings: []
risk_flags:
  consequential_authorship_claim: false
  human_representation_sensitive: false
pass:
  index: 0
  max: 1 | 2
next_action: optional skill slug
return_to_router_reason: optional reason
```

Leve o escopo de edição solicitado pelo usuário e as correções factuais explícitas no envelope quando um editor seguinte precisar deles. Preserve um `context_profile` explícito ou inferido separadamente do modo de execução do detector para que as Skills seguintes possam aplicar suas isenções (`skip`) e tolerâncias sem re-inferir. `executed` requer evidência de execução do host.

`pass.index` é o número de passagens de edição que alteraram o texto retornado ou um arquivo nomeado. Comece em 0. Defina `pass.max` como 1 para `--iterate 1` e como 2 para `--iterate 2` ou o padrão, e preserve ambos os valores entre as etapas.

## Regras de propriedade (Ownership rules)

Cada etapa é dona de uma classe de decisão:

- `avoid-ai-writing-router` é dono da seleção de caminho e sequenciamento.
- `ai-writing-detector` é dono da coleta de sinais, não de conclusões de autoria.
- `voice-preserving-rewriter` é dono da reescrita de texto retornado.
- `file-edit-in-place` é dono da alteração autorizada de um arquivo nomeado.
- `preservation-verifier` é dono do status de preservação antes/depois.
- `false-positive-reviewer` é dono dos limites de interpretação e é terminal no grafo de Skills.
- `avoid-ai-writing` permanece como a autoridade editorial canônica para regras de padrões, comportamento de voz, conteúdo protegido e limites de passagens.

## Handoffs tipados

### ROUTE
O roteador seleciona uma Skill primária e passa intenção, tipo de fonte, restrições e qualquer modo solicitado pelo usuário.

### FEED
Um resultado de detecção pode alimentar uma reescrita solicitada ou edição de arquivo nomeado. Correspondências candidatas tornam-se achados apenas após revisão de contexto e condições de aprovação.

### VERIFY
O reescritor ou editor de arquivo envia o material antes/depois para `preservation-verifier`.

### REPAIR
O verificador retorna os itens bloqueantes e o responsável correto pelo reparo quando `pass.index < pass.max`.

### RECHECK
Após o reparo, a verificação pode rodar mais uma vez. Uma nova verificação residual do detector roda apenas quando solicitada ou quando a convergência faz parte do pedido original.

### ESCALATE
Quando sinais do detector estiverem sendo usados para uma interpretação que não pertence ao detector, roteie essa interpretação para `false-positive-reviewer`.

`false-positive-reviewer` não chama o detector de volta diretamente. Se uma coleta adicional de sinais for solicitada, ele devolve o controle a `avoid-ai-writing-router` com `return_to_router_reason: fresh_signal_collection_needed`.

## Guardas condicionais (não são arestas do grafo)

### Preservação de representação humana
Quando a fonte for um prompt de imagem/vídeo ou briefing criativo descrevendo pessoas, preserve detalhes de identidade, cultura, geografia, deficiência, idade, vestimenta e realidade física como restrições protegidas.

## Limites de loop (Loop limits)

- Todas as alterações de texto retornado e arquivo nomeado compartilham o máximo solicitado: uma passagem para `--iterate 1`, caso contrário no máximo duas.
- Uma edição corretiva ou reparo do verificador consome a próxima passagem disponível; nenhum tem margem separada.
- Um loop de reparo do verificador pode reentrar no responsável pelo reparo uma vez quando o orçamento compartilhado tiver espaço, e depois verificar mais uma vez.
- Uma nova verificação residual do detector pode ocorrer uma vez quando o pedido original exigir.
- Se o orçamento de edição se esgotar ou a verificação ainda falhar após o reparo, pare e relate o erro de preservação não resolvido em vez de entrar em ciclo.
- Terminal Skills have no outgoing Skill edges.
- Every graph cycle must contain an edge with `max_reentries: 1`.

## Semântica de retorno ao roteador

Uma Skill devolve o controle a `avoid-ai-writing-router` em vez de escolher diretamente um novo responsável quando a evidência exigida estiver ausente, a capacidade de execução solicitada estiver indisponível, a próxima ação mudar de somente leitura para alteração, o pedido mudar de coleta de sinais para interpretação, ou o alvo mudar de texto retornado para arquivo nomeado (ou vice-versa).

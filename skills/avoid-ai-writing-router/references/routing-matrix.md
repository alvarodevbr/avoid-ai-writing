# Matriz de roteamento (Routing matrix)

`skill-graph.json` é a fonte da verdade legível por máquina. Esta tabela explica as mesmas rotas para revisão humana. O estado entre etapas segue `handoff-contract.md`.

<!-- BEGIN GENERATED GRAPH ROUTES -->
<!-- skill-graph-sha256: 76ffc6c17019c9337918cb9f309e2767073a47462bd00b600b29734ee591e929 -->
| Type | From | To | When | Max reentries |
| --- | --- | --- | --- | --- |
| ROUTE | `avoid-ai-writing-router` | `ai-writing-detector` | `detect_or_audit_only` |  |
| ROUTE | `avoid-ai-writing-router` | `voice-preserving-rewriter` | `rewrite_returned_text` |  |
| ROUTE | `avoid-ai-writing-router` | `file-edit-in-place` | `mutate_named_file` |  |
| ROUTE | `avoid-ai-writing-router` | `preservation-verifier` | `compare_before_after` |  |
| ROUTE | `avoid-ai-writing-router` | `false-positive-reviewer` | `consequential_authorship_interpretation` |  |
| FEED | `ai-writing-detector` | `voice-preserving-rewriter` | `user_requests_rewrite_after_audit` |  |
| FEED | `ai-writing-detector` | `file-edit-in-place` | `user_requests_named_file_fix_after_audit` |  |
| ESCALATE | `ai-writing-detector` | `false-positive-reviewer` | `user_asks_what_flags_prove` |  |
| VERIFY | `voice-preserving-rewriter` | `preservation-verifier` | `original_and_rewrite_available` |  |
| VERIFY | `file-edit-in-place` | `preservation-verifier` | `before_snapshot_available` |  |
| REPAIR | `preservation-verifier` | `voice-preserving-rewriter` | `returned_text_failed_preservation_and_shared_editing_budget_remains` | 1 |
| REPAIR | `preservation-verifier` | `file-edit-in-place` | `named_file_failed_preservation_and_shared_editing_budget_remains` | 1 |
| RECHECK | `preservation-verifier` | `ai-writing-detector` | `convergence_or_residual_audit_requested` | 1 |
<!-- END GENERATED GRAPH ROUTES -->

| Intenção | Responsável primário | Acompanhamento permitido | Condição de parada |
| --- | --- | --- | --- |
| detectar, escanear, auditar, pontuar, apenas sinalizar | `ai-writing-detector` | alimentar achados em etapa solicitada de reescrita/edição, ou escalar interpretação quando necessário | achados retornados |
| reescrever texto retornado | `voice-preserving-rewriter` | `preservation-verifier` quando verificação antes/depois fizer parte do fluxo | reescrita e verificação necessária concluídas |
| modificar arquivo explicitamente nomeado | `file-edit-in-place` | `preservation-verifier` após alteração real quando houver instantâneo anterior | alteração autorizada e verificação necessária concluídas |
| comparar original com reescrita | `preservation-verifier` | um reparo para o responsável correto quando `pass.index < pass.max`, depois uma nova verificação | aprovação, revisão aceita, orçamento esgotado ou segunda falha relatada |
| interpretar o que os sinais do detector podem comprovar | `false-positive-reviewer` | devolver controle ao roteador se nova coleta de sinais ou ação diferente for solicitada | limites da evidência explicados |
| auditar + reescrever + verificar | `avoid-ai-writing-router` | detector -> reescritor/editor -> verificador -> reparo opcional dentro do orçamento -> verificador | sequência solicitada concluída |
| reescrever prompt visual ou briefing criativo envolvendo pessoas | responsável normal de reescrita/edição + guarda de representação | verificador pode revisar detalhes de representação protegidos separadamente | redação limpa sem apagar significado protegido |
| invocação explícita da Skill canônica | `avoid-ai-writing` | permanecer canônica a menos que uma etapa especializada seja necessária | fluxo canônico concluído |

## Critérios de desempate (Tie breakers)

1. Um arquivo nomeado mais intenção explícita de alteração vence sobre pedidos gerais de reescrita.
2. Intenção de apenas sinalizar (*flag-only*) vence sobre limpeza genérica.
3. Questões de interpretação de autoria usam `false-positive-reviewer` mesmo quando já existe evidência do detector.
4. Reparo de texto retornado pertence a `voice-preserving-rewriter`; reparo de arquivo nomeado pertence a `file-edit-in-place`.
5. Representação humana dentro de um prompt visual adiciona uma guarda, mas não altera o responsável pela reescrita/edição.
6. Quando nenhum caminho especializado se encaixar claramente, use a Skill canônica `avoid-ai-writing`.

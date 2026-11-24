# Recuperação de lançamento (Release recovery)

O fluxo de trabalho de release roda apenas depois que uma alteração de versão chega à branch `main`. Sua verificação prévia (*preflight*) somente leitura checa a concordância de versão, a suíte de testes e o pacote npm antes que qualquer job de publicação receba permissão de escrita.

O GitHub serializa as execuções de release. Mantenha `queue: max`: `cancel-in-progress: false` protege o job em execução, mas não retém mais de uma execução pendente por si só.

## Recuperar uma publicação npm que falhou

Se o release do GitHub existir, mas a publicação no npm falhar, execute novamente (*rerun*) os jobs que falharam a partir da execução original do workflow. Uma nova execução mantém o commit SHA original, vê o release existente do GitHub, verifica se a tag ainda aponta para aquele SHA e tenta novamente apenas a versão ausente no npm. Não faça merge de outro aumento de versão como mecanismo de recuperação, não mova a tag de release nem reutilize uma versão que já chegou ao registro.

Se o *preflight* falhar, nenhum release ou pacote foi criado. Corrija a falha por meio de um novo pull request, mantenha a versão pretendida inalterada e dispare manualmente o workflow de release a partir da `main`.

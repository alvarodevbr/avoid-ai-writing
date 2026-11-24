# Suporte (Support)

Para bugs, problemas de roteamento, falsos positivos, problemas de empacotamento ou questões com o plugin para ChatGPT e Codex, abra uma issue em:

https://github.com/alvarodevbr/avoid-ai-writing/issues

Inclua a versão do plugin, o ambiente host utilizado, o comportamento esperado e uma reprodução mínima. Não publique rascunhos confidenciais ou informações pessoais em uma issue pública.

Para vulnerabilidades de segurança, não use o rastreador público de issues. Siga o arquivo [SECURITY.md](SECURITY.md) e use o relato privado de vulnerabilidades do GitHub.

## Scanners estáticos de skills

O NVIDIA SkillSpector v2.12.0 (`--no-llm`) relatou os seguintes achados ao escanear a v3.38.0 desta skill, revisados como falsos positivos:

- Os comentários `<!-- reference-loading:start -->` e `<!-- patterns:... -->` são marcadores de build. A instrução entre eles é texto visível.
- `references/patterns.md` fala sobre apagar e remover frases entre aspas, porque é isso que a skill faz. Alguns scanners leem verbos de remoção próximos a aspas como possível ofuscação.

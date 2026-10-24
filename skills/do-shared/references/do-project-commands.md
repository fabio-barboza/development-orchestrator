# Comandos do Projeto

Procedimento padrão referenciado pelas skills que precisam **rodar testes, checagens estáticas ou o serviço local** (`do-execute-task`, `do-execute-review`, `do-execute-review-fix`, `do-execute-qa`, `do-execute-qa-bugfix`) e pelo `do-setup`, que grava os comandos. O framework não presume linguagem, gerenciador de pacotes, framework de testes ou ferramenta de build: os comandos vêm sempre do próprio projeto.

## Onde ficam registrados

O `do-setup` grava a seção `## Project Commands` no arquivo de configuração do projeto da ferramenta de IA em uso (ver `do-project-context.md`):

| Finalidade | Comando | Observação |
|------------|---------|------------|
| Instalar dependências | [comando] | [pré-requisitos] |
| Rodar todos os testes | [comando] | [escopo: unitário, integração…] |
| Rodar um teste isolado | [comando com placeholder] | [como filtrar por arquivo ou nome] |
| Checagem estática | [comando(s)] | [tipos, lint, formatação — o que o projeto definir] |
| Iniciar o serviço local | [comando] | [porta ou endpoint de saúde] |

Linhas sem equivalente no projeto ficam como `—`.

## Procedimento para obter um comando

1. Leia a seção `Project Commands` do arquivo de configuração do projeto. Se a linha existir, use exatamente aquele comando.
2. Se a seção ou a linha não existir, **descubra no próprio projeto**, nesta ordem: os comandos declarados nos arquivos de manifesto, de build ou de tarefas na raiz; a configuração de integração contínua; o README e a documentação de contribuição. Use o comando que o projeto declara para a finalidade; não escolha um por convenção de linguagem.
3. Se houver mais de um candidato legítimo, escolha o que o próprio projeto documenta como principal. Nunca rode dois comandos alternativos para a mesma finalidade.
4. Se, mesmo assim, não houver comando para a finalidade:
   - **Testes:** registre "nenhuma infraestrutura de testes detectada" e siga a regra da skill (as skills que exigem testes passando não podem concluir o trabalho sem um comando de testes).
   - **Checagem estática:** pule e registre que o projeto não define uma.
   - **Iniciar o serviço local:** trate o serviço como `external` em `do-service-readiness.md` (documente a lacuna, não tente adivinhar).
5. Nunca instale ferramentas globalmente nem altere a configuração do projeto para fazer um comando funcionar.

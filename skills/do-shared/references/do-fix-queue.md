# Fila de Correções (review-fix e qa-fix)

Procedimento padrão do **modo fila** de `do-execute-review-fix` e `do-execute-qa-fix`: corrigir vários arquivos (fix tasks ou bugs) de uma vez, cada um em um subagente isolado, na ordem certa e sem que a falha de um impeça os outros. O usuário escolhe o lote; o framework nunca decide sozinho o que corrigir.

## Quando usar

| Argumento recebido | Modo |
|--------------------|------|
| Um único arquivo | Fluxo normal da skill, direto na sessão. Não use esta fila. |
| Uma pasta (`.../fixes/` ou `.../bugs/`) | Fila com os arquivos pendentes da pasta |
| Dois ou mais arquivos | Fila com exatamente esses arquivos |
| Uma pasta com `--min=<severidade>` | Fila só com os arquivos naquela severidade ou acima |

A severidade vem da palavra no nome do arquivo. Ordem, da maior para a menor: review `critico`, `maior`, `menor`; QA `alta`, `media`, `baixa`.

## Ferramenta de delegação

Use a mesma tabela e as mesmas regras de `do-execute-all-tasks` (seções "Ferramenta de delegação" e "Regras duras"): **uma única delegação por response, sempre em foreground, subagente literal `agent-execute-task`, sem paralelismo**. Correções tocam os mesmos arquivos de código; em paralelo, uma sobrescreve a outra. Neste procedimento você usa só as tools de delegação e de leitura: não edita arquivos nem roda comandos. Se `agent-execute-task` não resolver, **PARE** e reporte; nunca corrija inline.

Este modo precisa rodar na sessão primária. Erro de profundidade significa que a skill está dentro de um subagente: rode-a a partir da sessão primária.

## Montar a fila (uma vez)

1. Resolva a rodada pelo caminho recebido (`do-run-folders.md`) e liste os arquivos.
2. Leia **somente o frontmatter** de cada arquivo, com a tool de leitura e um limite pequeno de linhas (as primeiras 6). Não leia o arquivo inteiro: quem lê o corpo é o worker.
3. Entram na fila: review com `status: pendente`; QA com `status: aberto`. Um arquivo passado explicitamente com outro status é pulado e listado no resumo como "pulado (status X)". Bugs com `status: novo` nunca entram sozinhos.
4. Ordene por severidade, da maior para a menor, e depois pelo ID crescente.
5. Mostre a fila em uma mensagem curta e inicie a primeira; não peça confirmação.

## Iteração (um arquivo por vez)

Para cada arquivo, chame a ferramenta de delegação para `agent-execute-task` com a descrição "Corrigir <ID>" e este prompt:

```
Corrija o item <ID> da rodada <dir> do PRD prds/prd-<slug>/. Arquivo: <caminho>. Siga a skill <do-execute-review-fix | do-execute-qa-fix> em modo de arquivo único, na íntegra, e o contrato "Itens de correção" em do-execute-task/references/subagent-mode.md. Você não tem canal com o usuário: não pergunte nada. Regras de ambiente: trabalhe somente dentro da raiz do projeto; rascunhos apenas em .do-tmp/<review-fix | qa-fix>/ (apague ao final); nunca redirecione saída de comando para arquivo fora do projeto (filtre no pipe); encerre os serviços que você mesmo subiu; E2E apenas pelas tools de MCP. Leia a skill e os arquivos do-shared pelo caminho completo, sem usar glob (buscas por glob ignoram diretórios ocultos como `.agents/`); a skill está em <caminho da skill que você mesmo leu>. Altere somente código e testes: nunca edite PRD, TechSpec, tasks.md nem tome decisão de produto ou escopo; se a correção exigir isso, não edite nada e retorne NÃO RESOLVIDO dizendo qual decisão o usuário precisa tomar. Se a correção falhar, desfaça as suas próprias edições e confirme que a suíte voltou ao estado anterior. Retorne no formato do contrato.
```

Depois do retorno, **não releia arquivos nem liste pastas**: confie no retorno estruturado.

## Decisão após cada retorno

| Retorno | Ação |
|---------|------|
| `RESOLVIDO` | Registre e siga para o próximo |
| `NÃO RESOLVIDO` | Registre o motivo e **siga para o próximo**: os itens são independentes, e o arquivo continua com status de não resolvido para nova tentativa. Se o motivo for uma decisão do usuário (escopo, requisito), liste-a no resumo final em "DECISÕES PENDENTES" |
| `FALHA` com `suíte: verde` | Registre o motivo e siga para o próximo |
| `FALHA` com `suíte: vermelha`, ou retorno vazio ou malformado | **PARE a fila**: o código pode ter ficado inconsistente e os próximos itens falhariam por culpa deste |

## Encerramento

Gere o resumo final, com um item por linha:

```
Fila de correções concluída (<rodada>)
RESOLVIDOS (n): <IDs>
NÃO RESOLVIDOS (n): <ID — motivo>
FALHAS (n): <ID — motivo>
PULADOS (n): <ID — motivo>
DECISÕES PENDENTES (n): <ID — decisão que o usuário precisa tomar>   (apenas se houver)
NÃO EXECUTADOS por parada (n): <IDs>   (apenas se a fila parou)
BUGS NOVOS registrados pelos workers: <caminhos>   (apenas QA)
```

Sugira o passo seguinte: rodar de novo apenas os itens não resolvidos, ou uma nova rodada de `do-execute-review` / `do-execute-qa` para confirmar as correções.

## Regras invioláveis

1. Um arquivo por vez, em foreground, sem paralelismo.
2. Nunca corrija inline nem com subagente genérico.
3. Falha de um item não derruba os outros, exceto quando a suíte ficar vermelha.
4. O orquestrador não edita arquivos, não roda comandos (nem leitura, como `ls` ou `grep`) e não relê o que o worker já verificou. O resumo final vem só dos retornos dos workers.
5. Nunca escolha o que corrigir por conta própria: a fila é o que o usuário pediu.

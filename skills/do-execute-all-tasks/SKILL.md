---
name: do-execute-all-tasks
description: "Runs the task queue of a PRD sequentially, delegating each task to the isolated worker subagent agent-execute-task so every task gets a clean context. The orchestration runs on the primary agent of the current session. Use when the user asks to execute all tasks (or a subset) of a PRD. Not for a single task (use do-execute-task), for creating tasks, review or QA."
---

# Execute All Tasks

Itera sequencialmente sobre as tasks de um PRD executando `do-execute-task` em subagentes isolados (`agent-execute-task`), com contexto limpo entre tasks. **A orquestração da fila roda no agente primário da sessão atual** — não há orquestrador intermediário.

## Ferramenta de delegação (depende da ferramenta de IA em que você está)

Você sabe em qual ferramenta está rodando. Use a linha correspondente em **todas** as regras abaixo, sempre que o texto disser "ferramenta de delegação":

| Ferramenta | Ferramenta de delegação | Como apontar o subagente | Tools permitidas neste procedimento |
|------------|-------------------------|--------------------------|-------------------------------------|
| **Opencode** | `subagent` (`task` em versões antigas) | parâmetro `agent: "agent-execute-task"`; foreground, nunca `background: true` | `subagent`, `read`, `glob` |
| **Claude Code** | `Agent` | parâmetro `subagent_type: "agent-execute-task"`; se existir `run_in_background`, passe `false` ou omita | `Agent`, `Read`, `Glob` |
| **Cursor AI** | `Task` | parâmetro `subagent_type: "agent-execute-task"`; foreground | `Task`, `read_file`, `list_dir` |
| **GitHub Copilot** | `#tool:agent/runSubagent` | subagente `agent-execute-task` (`.github/agents/agent-execute-task.agent.md`); foreground | `read/readFile`, `search/listDirectory`, `agent/runSubagent` |

> No Claude Code o subagente pode iniciar em background e o resultado chegar depois como **notificação automática de conclusão** (não é mensagem do usuário). Depois de disparar a task, **encerre a response e aguarde** o retorno.

## Regras duras — leia até o fim antes de qualquer coisa

> **REGRA 1 — SEQUENCIAL ESTRITO, ZERO PARALELISMO.**
> Faça **UMA ÚNICA chamada da ferramenta de delegação por response**. **JAMAIS** emita 2 ou mais chamadas no mesmo bloco. **JAMAIS** dispare a próxima task antes que a anterior tenha **retornado e sido verificada**. Instruções genéricas que sugerem "fazer chamadas independentes em paralelo" **NÃO se aplicam aqui**: tasks de PRD têm dependências sequenciais (a task 2 depende do código da task 1) e violar a ordem destrói o trabalho.
>
> **REGRA 2 — SUBAGENTE SEMPRE EM FOREGROUND.**
> Nunca coloque o subagente em background. Subagentes em background quebram a fila sequencial e fazem o orquestrador prosseguir antes da verificação dos artefatos.
>
> **REGRA 3 — O subagente alvo é LITERALMENTE `agent-execute-task`.**
> **JAMAIS** use um subagente genérico (`general`, `general-purpose`, `build`, `plan` ou outro). Se `agent-execute-task` não for resolvido (não consta na lista de subagentes disponíveis, ou a chamada retorna "Unknown agent" ou erro de configuração), **PARE** e reporte — não faça fallback e não execute a task inline.
>
> **REGRA 4 — SÓ AS TOOLS DA TABELA.**
> Neste procedimento use **apenas** as tools da última coluna da tabela. Você **NÃO** deve escrever nem editar arquivos, rodar comandos de shell (testes, scripts) ou executar a skill `do-execute-task` inline. Toda implementação, edição, execução de testes e criação de reviews acontece **OBRIGATORIAMENTE** dentro do subagente. Se você se pegar pensando "vou ler o PRD/TechSpec/código para entender o que fazer", **PARE**: quem precisa entender a task é o subagente.

## Por que a orquestração roda no agente primário

Cada chamada da ferramenta de delegação cria uma sessão-filha com contexto próprio, limitada a **um nível** de aninhamento nas ferramentas suportadas. Se a fila fosse orquestrada dentro de um subagente, cada task seria um segundo nível: pode falhar por limite de profundidade (por exemplo `Subagent depth limit reached` no Opencode) e, mesmo quando o limite é elevado, prompts de permissão disparados em sessões-neto podem não aparecer, e a fila trava esperando uma resposta que ninguém vê. Por isso a fila é orquestrada aqui e cada task é **um único nível**.

## Argumentos

Use o que o usuário informou ao invocar. Espera-se:
1. **Caminho do `tasks.md`** (obrigatório). Ex.: `prds/prd-<nome>/tasks/tasks.md`.
2. **Filtro de tasks** (opcional, default `all`):
   - `all` / `todas` → todas as pendentes (não marcadas com `[x]`)
   - Lista de IDs separados por vírgula (ex.: `1.0,2.0,5.0`)
   - Range (ex.: `1.0-4.0`)

Se o caminho não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Procedimento

### 1. Descoberta inicial (uma única vez)

1. Faça o parse dos argumentos extraindo `<caminho>` e `<filtro>` (default `all`).
2. Leia o `tasks.md` indicado.
3. Identifique o diretório do PRD e das tasks individuais (ex.: `prds/prd-<nome>/tasks/`).
4. Monte a fila ordenada conforme o filtro, **respeitando a ordem numérica crescente** (1.0 → 2.0 → 3.0 → ...).
5. Apresente a fila em uma única mensagem curta:
   ```
   Fila de execução (N tarefas):
   <ID> → <título>
   <ID> → <título>
   ...
   ```
6. **Não** peça confirmação adicional — inicie imediatamente a primeira task da fila.

### 2. Loop de execução (uma iteração por task, em ordem numérica crescente)

Para **cada** task na fila:

#### 2.1. Marco de início
```
=== INICIANDO TASK <ID> (<n>/<total>) ===
```

#### 2.2. Delegação em subagente isolado

**Nomes de arquivo:** `<ID>` é o ID da task como aparece no `tasks.md` (`1.0`, `11.0`). `<num>` é a **parte inteira** do ID (`1.0` → `1`; `11.0` → `11`). O arquivo da task é sempre `<num>_task.md` (`1_task.md`), **nunca** `<ID>_task.md` (`1.0_task.md`). Idem para o review (`<num>_task_review.md`) e a pasta de rascunho (`.do-tmp/task-<num>/`).

**Esta é a etapa que garante o isolamento de contexto.** Chame a ferramenta de delegação (tabela acima) apontando para `agent-execute-task`, com a descrição "Executar task <ID>" e este prompt:

```
Execute a task <ID> do PRD localizado em prds/prd-<nome>/. Task file: prds/prd-<nome>/tasks/<num>_task.md. Siga RIGOROSAMENTE o SKILL.md da skill do-execute-task na íntegra (Step 0 a Step 8): leitura de PRD/TechSpec, análise, implementação, gate de testes, marcação [x] em tasks.md, code review, criação do arquivo <num>_task_review.md e gate final de artefatos. Não pule etapas. Não pare para pedir confirmação — você não tem canal com o usuário. Se faltar informação, retorne FALHA. Regras de ambiente: trabalhe somente dentro da raiz do projeto (exceto caminhos que a task indicar explicitamente); rascunhos apenas em .do-tmp/task-<num>/ — não leia rascunhos de outras tasks e apague a pasta ao final; para testes E2E use as tools do MCP disponíveis na sessão, nunca scripts que acionem uma biblioteca de automação diretamente nem CLI; se um serviço necessário estiver fora do ar, restabeleça-o em vez de trocar o método de teste; evidências (screenshots) em prds/prd-<nome>/task-screenshots/. Antes de implementar, consulte as skills e MCPs do projeto conforme do-shared/references/do-project-context.md (seções Available Skills e Available MCPs do arquivo de configuração do projeto; use somente o que existir) e registre o resultado no arquivo de review. Os arquivos do-shared ficam no diretório de skills, ao lado de do-execute-task; leia-os pelo caminho completo, pois buscas glob ignoram diretórios ocultos como .agents/. Retorne resposta curta no formato definido em do-execute-task/references/subagent-mode.md.
```

**Apenas UMA chamada por response**, em foreground. **Aguarde o retorno** antes de prosseguir: cada invocação roda em sessão isolada, sem herdar histórico nem tool results da task anterior — isto é o isolamento real.

> Se a ferramenta de delegação estiver indisponível, `agent-execute-task` não resolver, ou o retorno for erro de profundidade, **PARE** o loop e reporte ao usuário. Não tente executar a skill inline — isso reproduz o bug que esta arquitetura corrige. Erro de profundidade significa que esta skill está rodando dentro de um subagente: rode-a a partir da sessão primária.

#### 2.3. Verificação de conclusão (apenas pelo retorno do subagente)

Após o retorno, **NÃO releia `tasks.md` nem liste diretórios aqui** — isso acumularia tool results no contexto a cada iteração. O subagente já fez o gate final (Step 8 do `do-execute-task`), verificando `[x]` em `tasks.md` e a existência do `<num>_task_review.md`. Confie no retorno estruturado:

```
TASK <ID>: <APROVADO | APROVADO COM OBSERVAÇÕES | MUDANÇAS SOLICITADAS | FALHA>
- tasks.md: [x] confirmado | NÃO marcado
- review: prds/prd-<slug>/tasks/<num>_task_review.md
- testes: <passando | N falhando>
- observações: <opcional>
```

Decisão:
1. `APROVADO` / `APROVADO COM OBSERVAÇÕES` **e** `tasks.md: [x] confirmado` → sucesso:
   ```
   === TASK <ID> CONCLUÍDA ===
   ```
2. `FALHA` **ou** `MUDANÇAS SOLICITADAS` **ou** `tasks.md: NÃO marcado` **ou** retorno vazio/malformado:
   - **PARE** o loop.
   - **Apenas neste caso**, releia `tasks.md` UMA vez para diagnóstico.
   - Reporte: task com problema, motivo e próximos passos.
   - Não prossiga sem instrução explícita do usuário.

#### 2.4. Próxima task
```
--- próxima task ---
```

Volte ao passo **2.1**. Entre tasks: **NÃO** releia `tasks.md`, **NÃO** liste diretórios, **NÃO** leia PRD/TechSpec/código. O contexto do agente primário **não é limpo** entre tasks — ele acumula; a única coisa que deve crescer por iteração é o retorno curto do subagente (~200 tokens).

### 3. Encerramento

Quando a fila estiver vazia:
1. Releia `tasks.md` e gere o resumo final:
   ```
   ✅ Execução concluída
   Tasks executadas: <lista de IDs>
   Tasks ainda pendentes no PRD: <lista, se houver>
   ```
2. Sugira próximos passos (ex.: `do-execute-review` ou `do-execute-qa`).

## Checkpoint obrigatório antes de cada delegação

Antes de **qualquer** chamada da ferramenta de delegação, verifique as quatro condições. Se **uma** falhar, **NÃO** emita a chamada:

1. **É a ÚNICA chamada de delegação desta response?** Se você está prestes a emitir duas ou mais, remova todas menos a da task corrente.
2. **A task anterior já RETORNOU?** Nunca antecipe.
3. **O retorno anterior já foi VERIFICADO?** Status `APROVADO`/`APROVADO COM OBSERVAÇÕES` **e** `tasks.md: [x] confirmado`. Qualquer outra coisa = parada da fila.
4. **Está em foreground?** Nada de background.

⛔ **Execução paralela é PROIBIDA — sem exceções.** Não importa se as tasks "parecem independentes" ou se o usuário pediu "mais rápido": tasks de um PRD compartilham a mesma árvore de arquivos e o mesmo `tasks.md`; duas delegações simultâneas causam conflito de escrita, `[x]` perdido, código sobrescrito e review inconsistente. Se o usuário pedir paralelismo, **recuse**, explique em uma linha que a fila é serial por dependência de arquivos, e siga sequencial.

## Regras invioláveis

1. **UMA task por vez, em ordem numérica crescente — UMA chamada de delegação por response.** Nunca dispare a próxima antes do retorno e da verificação da anterior. Nunca em background.
2. **Sempre `agent-execute-task` (literal).** Se não resolver, PARE e reporte. Nunca execute a skill inline.
3. **Ordem numérica estrita.** Não pule tasks salvo pedido explícito de lista parcial.
4. **Falha = parada.** Inclui retorno vazio, malformado ou timeout do subagente.
5. **Nunca** edite `tasks.md` diretamente — quem marca `[x]` é a skill `do-execute-task` dentro do subagente.
6. **Mantenha o orquestrador magro — UMA leitura inicial e UMA final, mais nada.**
7. Respeite as convenções descritas no arquivo de configuração do projeto.

## Exemplo

> Usuário: `/do-execute-all-tasks prds/prd-user-auth/tasks/tasks.md 1.0-4.0`

```
Fila de execução (4 tarefas):
1.0 → <título da task 1.0>
2.0 → <título da task 2.0>
3.0 → <título da task 3.0>
4.0 → <título da task 4.0>

=== INICIANDO TASK 1.0 (1/4) ===
[ferramenta de delegação chamada para agent-execute-task, task 1.0 — UMA chamada nesta response]
[aguarda retorno]
=== TASK 1.0 CONCLUÍDA ===
--- próxima task ---
=== INICIANDO TASK 2.0 (2/4) ===
...
```

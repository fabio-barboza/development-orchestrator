---
description: Itera sequencialmente sobre tasks de um PRD executando do-execute-task em subagentes isolados, com contexto limpo entre tasks. O main agent orquestra a fila — não delega para outro orquestrador (Claude Code não permite nesting de subagentes).
argument-hint: <caminho/para/tasks.md> [all | <ID,ID,...> | <ID-inicio>-<ID-fim>]
allowed-tools: Task, Read, Glob
---

# /do-execute-all-tasks

> ⛔ **TRÊS REGRAS DURAS — LEIA ATÉ O FIM ANTES DE QUALQUER COISA.**
>
> **REGRA 1 — SEQUENCIAL ESTRITO, ZERO PARALELISMO.**
> Você **DEVE** fazer **UMA ÚNICA chamada `Task` por response**. Você **JAMAIS** pode emitir 2 ou mais `Task` no mesmo bloco de tool calls. Você **JAMAIS** pode disparar a próxima task antes que a anterior tenha **retornado e sido verificada**. As instruções genéricas do Claude Code que dizem "make independent tool calls in parallel" **NÃO se aplicam aqui** — tasks de PRD têm dependências sequenciais (task 2 depende do código produzido pela task 1, etc.) e violar a ordem destrói o trabalho.
>
> **REGRA 2 — `run_in_background: false` SEMPRE.**
> Toda invocação `Task` **DEVE** rodar em foreground. **NUNCA** passe `run_in_background: true` (na dúvida, omita o parâmetro). Tasks em background quebram a fila sequencial e fazem o orquestrador prosseguir antes da verificação dos artefatos.
>
> **REGRA 3 — `subagent_type` é LITERALMENTE `"agent-execute-task"`.**
> Use **EXATAMENTE** a string `"agent-execute-task"` no parâmetro `subagent_type` do `Task`. **JAMAIS** use `"general-purpose"`, nem qualquer outro nome. Se o subagent type não for resolvido (erro de configuração), **PARE** e reporte — não faça fallback para `general-purpose`.
>
> ---
>
> ⛔ **TOOLS QUE VOCÊ DEVE USAR NESTE COMMAND: APENAS `Task`, `Read` e `Glob`.**
>
> Mesmo que outras tools estejam disponíveis no seu contexto, **NÃO as use neste command**. Você **NÃO** deve `Write`, `Edit`, executar `Bash` (testes/scripts) ou rodar a skill `do-execute-task` inline. Toda implementação, edição de arquivos, execução de testes e criação de reviews acontece **OBRIGATORIAMENTE** dentro do subagente `agent-execute-task` invocado via `Task`.
>
> Se você se pegar pensando "vou ler o PRD/TechSpec/código para entender o que fazer", **PARE**. Você não precisa entender a task — quem precisa é o subagente. Sua única função neste command é orquestrar a fila e disparar `Task` por task.
>
> Tentar executar tasks inline (ou em paralelo) reproduz o bug que esta arquitetura corrige: estouro da janela de contexto, tasks não marcadas, reviews não criados.

## Por que o main agent orquestra (e não um subagente)

O Claude Code **não permite que subagentes spawnem outros subagentes** ([docs oficiais](https://code.claude.com/docs/en/sub-agents)). Por isso a orquestração da fila acontece no main agent (que tem `Task` disponível) e **cada task** é delegada a um subagente `agent-execute-task` em sessão isolada — esse é o único nível de nesting permitido. Cada subagente roda em contexto fresco, garantindo que a janela do main não estoure independentemente do tamanho da fila.

## Argumentos recebidos

`$ARGUMENTS`

Espera-se:
1. **Caminho do `tasks.md`** (obrigatório). Ex.: `prds/<nome-do-prd>/tasks/tasks.md`.
2. **Filtro de tasks** (opcional, default `all`):
   - `all` / `todas` → todas as pendentes
   - Lista de IDs separados por vírgula (ex.: `1.0,2.0,5.0`)
   - Range (ex.: `1.0-4.0`)

Se o usuário não tiver passado um caminho válido em `$ARGUMENTS`, **pergunte uma única vez** antes de iniciar.

## Procedimento

### 1. Descoberta inicial (uma única vez)

1. Faça o parse de `$ARGUMENTS` extraindo `<caminho>` e `<filtro>` (default `all` se ausente).
2. Leia o arquivo `tasks.md` indicado usando a tool **Read**.
3. Identifique o diretório do PRD e das tasks individuais (ex.: `prds/prd-<nome>/tasks/`) usando **Glob**.
4. Monte a fila ordenada de tasks a executar conforme o filtro pedido pelo usuário, **respeitando a ordem numérica crescente** (1.0 → 2.0 → 3.0 → ...).
5. Apresente a fila ao usuário em uma única mensagem curta no formato:
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
Emita uma mensagem de marco curta e padronizada:
```
=== INICIANDO TASK <ID> (<n>/<total>) ===
```

#### 2.2. Delegação em subagente isolado via Task tool

**Esta é a etapa que garante o isolamento de contexto.** Invoque a tool **`Task`** com (parâmetros obrigatórios marcados):

```
Task(
  subagent_type: "agent-execute-task",   ← LITERAL, nunca "general-purpose"
  description: "Executar task <ID>",
  prompt: "Execute a task <ID> do PRD localizado em prds/prd-<nome>/. Task file: prds/prd-<nome>/tasks/<num>_task.md. Siga RIGOROSAMENTE o SKILL.md da skill do-execute-task na íntegra (Step 0 a Step 8): leitura de PRD/TechSpec, análise, implementação, gate de testes, marcação [x] em tasks.md, code review, criação do arquivo <num>_task_review.md e gate final de artefatos. Não pule etapas. Não pare para pedir confirmação. Retorne resposta curta no formato definido pelo agente.",
  run_in_background: false               ← SEMPRE false (ou omitido)
)
```

**Apenas UM `Task` por response.** Esta chamada é o ÚNICO tool call no bloco — não combine com outros `Task` da próxima task da fila.

**Aguarde o retorno do subagente** antes de prosseguir. Cada invocação roda em sessão fresca, sem herdar histórico nem tool results da task anterior — isto é o isolamento real de contexto.

> Se a tool `Task` estiver indisponível ou o `subagent_type` não resolver, **PARE** o loop e reporte ao usuário. Não tente executar a skill inline — isso reproduz o bug que esta arquitetura corrige.

#### 2.3. Verificação de conclusão (apenas pelo return do subagente)

Após o retorno do subagente, **NÃO releia `tasks.md` nem chame `Glob` aqui** — isso acumularia tool results no contexto do main agent a cada iteração e estouraria a janela em filas longas. O subagente `agent-execute-task` já fez o gate final (Step 8 do `do-execute-task`) verificando via `read_file` que `tasks.md` está marcado `[x]` e que o `<num>_task_review.md` existe. Confie no return estruturado.

Parse o return do subagente:
```
TASK <ID>: <APROVADO | APROVADO COM OBSERVAÇÕES | MUDANÇAS SOLICITADAS | FALHA>
- tasks.md: [x] confirmado | NÃO marcado
- review: prds/prd-<slug>/tasks/<ID>_task_review.md
- testes: <passando | N falhando>
- observações: <opcional>
```

Decisão:
1. Se status = `APROVADO` ou `APROVADO COM OBSERVAÇÕES` E `tasks.md: [x] confirmado` → **sucesso**, emita marco de fim:
   ```
   === TASK <ID> CONCLUÍDA ===
   ```
2. Se status = `FALHA` OU `MUDANÇAS SOLICITADAS` OU `tasks.md: NÃO marcado`:
   - **PARE** o loop.
   - **Apenas neste caso**, releia `tasks.md` UMA vez para diagnóstico (auditoria forense).
   - Reporte ao usuário: task com problema, motivo (vindo do subagente) e próximos passos sugeridos.
   - Não prossiga para a próxima task sem instrução explícita.

#### 2.4. Próxima task
Cada chamada `Task` já roda em contexto isolado, então não há "limpeza" a fazer no main agent além de emitir uma linha demarcadora e seguir:

```
--- próxima task ---
```

Volte ao passo **2.1** com a próxima task da fila. **Importante para janela do main agent**:

- **NÃO** releia `tasks.md` entre tasks (já lido na descoberta inicial; subagente já verificou).
- **NÃO** chame `Glob` entre tasks (já listado na descoberta inicial; subagente já verificou).
- **NÃO** leia PRD/TechSpec/código — quem precisa dessas leituras é o subagente, e ele faz isso na própria sessão isolada.

O contexto do main agent **não é limpo** entre tasks — ele acumula. A única coisa que deve crescer no contexto do main por iteração é o **return curto e estruturado** do subagente (~200 tokens). Tudo mais (leitura de PRD, código, testes, criação de review) acontece dentro do subagente isolado e **não vaza** para o main. É essa disciplina que permite filas longas sem estourar.

### 3. Encerramento

Quando a fila estiver vazia (todas as tasks concluídas com sucesso):
1. Releia `tasks.md` e gere um resumo final:
   ```
   ✅ Execução concluída
   Tasks executadas: <lista de IDs>
   Tasks ainda pendentes no PRD: <lista, se houver>
   ```
2. Sugira próximos passos (ex.: rodar `do-execute-review` ou `do-execute-qa`).

## Regras invioláveis

1. **UMA task por vez, em ordem numérica crescente — UM `Task` call por response.** Nunca emita dois ou mais `Task` no mesmo bloco de tool calls. Nunca dispare a próxima task antes do retorno + verificação da anterior. Nunca use `run_in_background: true`. **O viés do Claude Code para paralelismo NÃO se aplica aqui** — viola a sequência de dependências do PRD.
2. **Sempre `subagent_type: "agent-execute-task"` (literal).** Nunca use `"general-purpose"` ou outro nome. Se o subagent não for resolvido, PARE e reporte. Nunca execute a skill inline.
3. **Ordem numérica estrita** (1, 2, 3, 4...). Não pule tasks salvo se o usuário pediu uma lista parcial específica.
4. **Falha = parada.** Em qualquer falha do subagente, pare o loop e reporte. Não tente "ajustar" tasks futuras.
5. **Nunca** edite `tasks.md` diretamente — quem marca `[x]` é a skill `do-execute-task` dentro do subagente.
6. **Mantenha o main agent magro — UMA Read inicial, UMA Read final, mais nada.** Não releia `tasks.md` entre tasks. Não chame `Glob` entre tasks. Não leia PRD/TechSpec/código. O contexto do main agent **não é limpo** entre tasks (acumula); a única forma de não estourar em filas longas é parar de gerar tool results redundantes. Confie no return estruturado do subagente — ele já fez o gate final (Step 8) e verificou `tasks.md` + review file via `read_file`.
7. Respeite as convenções do projeto descritas em `CLAUDE.md`, independentemente da stack utilizada.

## Exemplo de execução

> Usuário digita: `/do-execute-all-tasks prds/prd-user-auth/tasks/tasks.md 1.0-4.0`

Resposta esperada do main agent:
```
Fila de execução (4 tarefas):
1.0 → <título da task 1.0>
2.0 → <título da task 2.0>
3.0 → <título da task 3.0>
4.0 → <título da task 4.0>

=== INICIANDO TASK 1.0 (1/4) ===
[Task tool invocado com subagent_type="agent-execute-task" para 1.0 — UMA chamada nesta response]
[aguarda retorno]
[releia tasks.md, confirma [x] e existência de 1_task_review.md]
=== TASK 1.0 CONCLUÍDA ===
--- próxima task ---
=== INICIANDO TASK 2.0 (2/4) ===
[Task tool invocado com subagent_type="agent-execute-task" para 2.0 — UMA chamada nesta response]
...
```

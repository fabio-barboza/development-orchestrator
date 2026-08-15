---
description: Itera sequencialmente sobre tasks de um PRD executando agent-execute-task em cada uma, em subagentes isolados (contexto distinto por task). A orquestração roda no agente primário — sem orquestrador intermediário.
---

# /do-execute-all-tasks

Itera sequencialmente sobre tasks de um PRD executando `do-execute-task` em subagentes isolados, com contexto limpo entre tasks. **A orquestração da fila roda no agente primário da sessão atual** (este command) — não há orquestrador intermediário.

> ⛔ **TRÊS REGRAS DURAS — LEIA ATÉ O FIM ANTES DE QUALQUER COISA.**
>
> **REGRA 1 — SEQUENCIAL ESTRITO, ZERO PARALELISMO.**
> Você **DEVE** fazer **UMA ÚNICA chamada da tool `task` por response**. Você **JAMAIS** pode emitir 2 ou mais `task` no mesmo bloco de tool calls. Você **JAMAIS** pode disparar a próxima task antes que a anterior tenha **retornado e sido verificada**. Tasks de PRD têm dependências sequenciais (task 2 depende do código produzido pela task 1, etc.) e violar a ordem destrói o trabalho.
>
> **REGRA 2 — NUNCA `background: true`.**
> Toda invocação da tool `task` **DEVE** rodar em foreground. **NUNCA** passe `background: true` (na dúvida, omita o parâmetro). Subagentes em background quebram a fila sequencial e fazem o orquestrador prosseguir antes da verificação dos artefatos.
>
> **REGRA 3 — `subagent_type` é LITERALMENTE `"agent-execute-task"`.**
> Use **EXATAMENTE** a string `"agent-execute-task"` no parâmetro `subagent_type`. **JAMAIS** use `"general"`, `"build"`, `"plan"` ou qualquer outro nome. Se o subagent type não for resolvido (erro de configuração), **PARE** e reporte — não faça fallback.
>
> ---
>
> ⛔ **TOOLS QUE VOCÊ DEVE USAR NESTE COMMAND: APENAS `task`, `read` e `glob`.**
>
> Mesmo que outras tools estejam disponíveis, **NÃO as use neste command**. Você **NÃO** deve `write`, `edit`, executar `bash` (testes/scripts) ou rodar a skill `do-execute-task` inline. Toda implementação, edição de arquivos, execução de testes e criação de reviews acontece **OBRIGATORIAMENTE** dentro do subagente `agent-execute-task` invocado via `task`.
>
> Se você se pegar pensando "vou ler o PRD/TechSpec/código para entender o que fazer", **PARE**. Você não precisa entender a task — quem precisa é o subagente.

## Por que a orquestração roda no agente primário (e não em um subagente)

No opencode, cada chamada da tool `task` cria uma **child session** com contexto próprio. A profundidade de aninhamento é limitada por `subagent_depth` (**default: 1**) e um subagente só recebe a tool `task` se o próprio agente declarar `permission.task`. Se a orquestração rodasse dentro de um subagente (ex.: command com `subtask: true` apontando para um agente `agent-execute-all-tasks`), cada delegação seria um segundo nível de aninhamento:

- com `subagent_depth` no default (1), a chamada falha com `Subagent depth limit reached`;
- mesmo com `subagent_depth: 2`, prompts de permissão (`question`, `bash`, `edit`) disparados dentro do neto **podem não ser exibidos na TUI**, e a fila trava esperando indefinidamente uma resposta que ninguém vê.

Por isso a fila é orquestrada aqui, na sessão primária: cada `task` é **um único nível** de aninhamento, funciona com a configuração default do opencode e todos os prompts aparecem normalmente.

## Argumentos recebidos

$ARGUMENTS

Espera-se:
1. **Caminho do `tasks.md`** (obrigatório). Ex.: `prds/prd-<nome>/tasks/tasks.md`.
2. **Filtro de tasks** (opcional, default `all`):
   - `all` / `todas` → todas as pendentes (não marcadas com `[x]`)
   - Lista de IDs separados por vírgula (ex.: `1.0,2.0,5.0`)
   - Range (ex.: `1.0-4.0`)

Se o caminho não for fornecido nos argumentos, **pergunte uma única vez** antes de prosseguir.

## Procedimento

### 1. Descoberta inicial (uma única vez)

1. Faça o parse dos argumentos extraindo `<caminho>` e `<filtro>` (default `all` se ausente).
2. Leia o `tasks.md` indicado com a tool **`read`**.
3. Identifique o diretório do PRD e das tasks individuais (ex.: `prds/prd-<nome>/tasks/`) com **`glob`**.
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

#### 2.2. Delegação em subagente isolado via tool `task`

**Esta é a etapa que garante o isolamento de contexto.** Invoque:

```
task(
  subagent_type: "agent-execute-task",   ← LITERAL, nunca outro nome
  description: "Executar task <ID>",
  prompt: "Execute a task <ID> do PRD localizado em prds/prd-<nome>/. Task file: prds/prd-<nome>/tasks/<num>_task.md. Siga RIGOROSAMENTE o SKILL.md da skill do-execute-task na íntegra (Step 0 a Step 8): leitura de PRD/TechSpec, análise, implementação, gate de testes, marcação [x] em tasks.md, code review, criação do arquivo <num>_task_review.md e gate final de artefatos. Não pule etapas. Não pare para pedir confirmação — você não tem canal com o usuário. Se faltar informação, retorne FALHA. Retorne resposta curta no formato definido pelo agente."
)
```

**Apenas UMA chamada `task` por response** e **sem `background: true`**.

**Aguarde o retorno do subagente** antes de prosseguir. Cada invocação roda em child session isolada, sem herdar histórico nem tool results da task anterior — isto é o isolamento real.

> Se a tool `task` estiver indisponível, o `subagent_type` não resolver, ou o retorno for erro de profundidade (`Subagent depth limit reached`), **PARE** o loop e reporte ao usuário. Não tente executar a skill inline — isso reproduz o bug que esta arquitetura corrige.
>
> **Diagnóstico de `Subagent depth limit reached`**: significa que este command está rodando dentro de um subagente (sessão filha). Rode `/do-execute-all-tasks` a partir da sessão primária (agente `build` ou equivalente), sem `subtask`.

#### 2.3. Verificação de conclusão (apenas pelo return do subagente)

Após o retorno, **NÃO releia `tasks.md` nem chame `glob` aqui** — isso acumularia tool results no contexto a cada iteração. O subagente já fez o gate final (Step 8 do `do-execute-task`), verificando `[x]` em `tasks.md` e a existência do `<num>_task_review.md`. Confie no return estruturado:

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

Volte ao passo **2.1**. Entre tasks: **NÃO** releia `tasks.md`, **NÃO** chame `glob`, **NÃO** leia PRD/TechSpec/código. O contexto do agente primário **não é limpo** entre tasks — ele acumula; a única coisa que deve crescer por iteração é o return curto do subagente (~200 tokens).

### 3. Encerramento

Quando a fila estiver vazia:
1. Releia `tasks.md` e gere o resumo final:
   ```
   ✅ Execução concluída
   Tasks executadas: <lista de IDs>
   Tasks ainda pendentes no PRD: <lista, se houver>
   ```
2. Sugira próximos passos (ex.: `/do-execute-review` ou `/do-execute-qa`).

## 🔒 CHECKPOINT OBRIGATÓRIO ANTES DE CADA DELEGAÇÃO

Antes de emitir **qualquer** chamada de a tool `task`, verifique as quatro condições abaixo. Se **uma** delas falhar, **NÃO** emita a chamada.

1. **É a ÚNICA chamada de delegação desta response?** Se você está prestes a emitir duas ou mais no mesmo bloco de tool calls — **PARE**. Remova todas menos a da task corrente.
2. **A task anterior já RETORNOU?** Delegação só acontece depois que o subagente da task anterior devolveu o bloco `TASK <ID>: ...`. Nunca antecipe.
3. **O retorno anterior já foi VERIFICADO?** Status `APROVADO`/`APROVADO COM OBSERVAÇÕES` **e** `tasks.md: [x] confirmado`. Qualquer outra coisa = parada da fila.
4. **Está em foreground?** Nada de `background: true`. Delegação em background quebra a serialização e faz a fila avançar antes da verificação.

⛔ **Execução paralela é PROIBIDA — sem exceções.** Não importa se as tasks "parecem independentes", se o usuário pediu "mais rápido", ou se a ferramenta sugere agrupar chamadas independentes. Tasks de um PRD compartilham a mesma árvore de arquivos e têm dependências sequenciais (a task N+1 lê e edita o código produzido pela task N). Duas delegações simultâneas gravam nos mesmos arquivos e no mesmo `tasks.md` → conflito de escrita, `[x]` perdido, código sobrescrito, review inconsistente. **Uma task por vez, sempre, mesmo que isso seja mais lento.**

Se o usuário pedir explicitamente paralelismo: **recuse**, explique em uma linha que a fila é serial por dependência de arquivos, e siga sequencial.

## Regras invioláveis

1. **UMA task por vez, em ordem numérica crescente — UMA chamada `task` por response.** Nunca duas no mesmo bloco. Nunca dispare a próxima antes do retorno + verificação da anterior. Nunca `background: true`.
2. **Sempre `subagent_type: "agent-execute-task"` (literal).** Se não resolver, PARE e reporte. Nunca execute a skill inline.
3. **Ordem numérica estrita.** Não pule tasks salvo pedido explícito de lista parcial.
4. **Falha = parada.** Inclui retorno vazio, malformado ou timeout do subagente.
5. **Nunca** edite `tasks.md` diretamente — quem marca `[x]` é a skill `do-execute-task` dentro do subagente.
6. **Mantenha o orquestrador magro — UMA `read` inicial, UMA `read` final, mais nada.**
7. Respeite as convenções do projeto descritas em `AGENTS.md`.

## Exemplo de execução

> Usuário digita: `/do-execute-all-tasks prds/prd-user-auth/tasks/tasks.md 1.0-4.0`

```
Fila de execução (4 tarefas):
1.0 → <título da task 1.0>
2.0 → <título da task 2.0>
3.0 → <título da task 3.0>
4.0 → <título da task 4.0>

=== INICIANDO TASK 1.0 (1/4) ===
[tool task invocada com subagent_type="agent-execute-task" para 1.0 — UMA chamada nesta response]
[aguarda retorno]
=== TASK 1.0 CONCLUÍDA ===
--- próxima task ---
=== INICIANDO TASK 2.0 (2/4) ===
...
```

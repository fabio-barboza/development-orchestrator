# /do-execute-all-tasks

Itera sequencialmente sobre tasks de um PRD executando `do-execute-task` em subagentes isolados, com contexto limpo entre tasks. **A orquestração da fila roda no main agent** (este command) — não há orquestrador intermediário. Desde o Cursor 2.5 um subagente pode lançar subagentes filhos, mas um neto **não** pode lançar mais nenhum; orquestrar a partir do main agent mantém a fila dentro do único nível garantido em qualquer versão.

> ⛔ **TRÊS REGRAS DURAS — LEIA ATÉ O FIM ANTES DE QUALQUER COISA.**
>
> **REGRA 1 — SEQUENCIAL ESTRITO, ZERO PARALELISMO.**
> Você **DEVE** fazer **UMA ÚNICA chamada `Task` por response**. Você **JAMAIS** pode emitir 2 ou mais `Task` no mesmo bloco de tool calls. Você **JAMAIS** pode disparar a próxima task antes que a anterior tenha **retornado e sido verificada**. Tasks de PRD têm dependências sequenciais (task 2 depende do código da task 1, etc.) e violar a ordem destrói o trabalho.
>
> **REGRA 2 — Subagente em foreground SEMPRE.**
> Toda invocação `Task` **DEVE** rodar em foreground. **NUNCA** passe parâmetros que coloquem o subagente em background. Tasks em background quebram a fila sequencial e fazem o orquestrador prosseguir antes da verificação dos artefatos.
>
> **REGRA 3 — `subagent_type` é LITERALMENTE `"agent-execute-task"`.**
> Use **EXATAMENTE** a string `"agent-execute-task"`. **JAMAIS** use `"general-purpose"` ou outro nome. Se o subagent type não for resolvido, **PARE** e reporte — não faça fallback.
>
> ---
>
> ⛔ **TOOLS QUE VOCÊ DEVE USAR NESTE COMMAND: APENAS `Task`, `read_file` e `list_dir`.**
>
> Mesmo que outras tools estejam disponíveis no contexto, **NÃO as use neste command**. Você **NÃO** deve editar arquivos, criar arquivos ou rodar comandos shell aqui. Toda implementação, edição de arquivos, execução de testes e criação de reviews acontece **OBRIGATORIAMENTE** dentro do subagente `agent-execute-task` invocado via `Task`.

## Argumentos

Espera-se na mensagem que acompanha o comando:

1. **Caminho do `tasks.md`** — obrigatório. Ex.: `prds/<nome-do-prd>/tasks/tasks.md`.
2. **Filtro de tasks** — opcional, default `all`:
   - `all` / `todas` → todas as pendentes (não marcadas com `[x]`)
   - Lista de IDs separados por vírgula (ex.: `1.0, 2.0, 5.0`)
   - Range (ex.: `1.0-4.0`)

Se faltar o caminho do `tasks.md`, **pergunte uma única vez** antes de prosseguir.

## Procedimento

### 1. Descoberta inicial (uma única vez)

1. Faça o parse da entrada extraindo `<caminho>` e `<filtro>` (default `all`).
2. Leia o `tasks.md` indicado com `read_file`.
3. Identifique o diretório das tasks individuais (ex.: `prds/prd-<nome>/tasks/`) via `list_dir`.
4. Monte a fila ordenada de tasks a executar conforme o filtro pedido pelo usuário, **respeitando a ordem numérica crescente** (1.0 → 2.0 → 3.0 → ...).
5. Apresente a fila ao usuário em uma única mensagem curta:
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

#### 2.2. Delegação em subagente isolado via Task tool

Invoque a tool **`Task`** delegando ao subagente `agent-execute-task` (definido em `.cursor/agents/agent-execute-task.md`) com prompt autocontido contendo:

- ID da task corrente
- Caminho do PRD (ex.: `prds/prd-<nome>/`)
- Caminho exato do task file (ex.: `prds/prd-<nome>/tasks/<num>_task.md`)
- Reforço de seguir o `SKILL.md` da `do-execute-task` na íntegra

Exemplo de prompt para o subagente:

> "Execute a task `<ID>` do PRD localizado em `prds/prd-<nome>/`. Task file: `prds/prd-<nome>/tasks/<num>_task.md`. Siga RIGOROSAMENTE o `SKILL.md` da skill `do-execute-task` na íntegra (Step 0 a Step 8): leitura de PRD/TechSpec, análise, implementação, gate de testes, marcação `[x]` em `tasks.md`, code review, criação do arquivo `<num>_task_review.md` e gate final de artefatos. Não pule etapas. Não pare para pedir confirmação. Retorne resposta curta no formato definido pelo agente."

**Apenas UM `Task` por response.** Esta chamada é o ÚNICO tool call no bloco — não combine com outros `Task` da próxima task da fila.

**Aguarde o retorno do subagente** antes de prosseguir. Cada invocação roda em sessão fresca (clean context), sem herdar histórico nem tool results da task anterior — isto é o isolamento real.

> **Fallback documentado do Cursor**: há relatos de a tool `Task` não enxergar agentes definidos em `.cursor/agents/*.md`. Se `subagent_type: "agent-execute-task"` não resolver, use a forma de invocação explícita documentada pelo Cursor — `Use the /agent-execute-task subagent para executar a task <ID> ...` com o mesmo prompt autocontido. Ela também cria um subagente em contexto isolado. Continua valendo: **uma** delegação por response, sequencial. Se nem essa forma resolver o agente, **PARE** e reporte.

> Se a tool `Task` estiver indisponível ou o subagent_type não resolver, **PARE** e reporte ao usuário. Não tente executar a skill inline — isso reproduz o bug que esta arquitetura corrige.

#### 2.3. Verificação de conclusão (apenas pelo return do subagente)

Após o retorno do subagente, **NÃO releia `tasks.md` nem chame `list_dir` aqui** — isso acumularia tool results no contexto do main agent a cada iteração e estouraria a janela em filas longas. O subagente `agent-execute-task` já fez o gate final (Step 8 do `do-execute-task`) verificando que `tasks.md` está marcado `[x]` e que o `<num>_task_review.md` existe. Confie no return estruturado.

Parse o return do subagente:
```
TASK <ID>: <APROVADO | APROVADO COM OBSERVAÇÕES | MUDANÇAS SOLICITADAS | FALHA>
- tasks.md: [x] confirmado | NÃO marcado
- review: prds/prd-<slug>/tasks/<num>_task_review.md
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
   - **Apenas neste caso**, releia `tasks.md` UMA vez para diagnóstico.
   - Reporte ao usuário: task com problema, motivo e próximos passos sugeridos.
   - Não prossiga sem instrução explícita.

#### 2.4. Próxima task

```
--- próxima task ---
```

Volte ao passo **2.1** com a próxima task da fila. **Importante para janela do main agent**:

- **NÃO** releia `tasks.md` entre tasks (já lido na descoberta inicial; subagente já verificou).
- **NÃO** chame `list_dir` entre tasks (já listado na descoberta inicial; subagente já verificou).
- **NÃO** leia PRD/TechSpec/código — quem precisa dessas leituras é o subagente, e ele faz isso na própria sessão isolada.

O contexto do main agent **não é limpo** entre tasks — ele acumula. A única coisa que deve crescer no contexto por iteração é o **return curto e estruturado** do subagente (~200 tokens). Tudo mais acontece dentro do subagente isolado e **não vaza** para o main.

### 3. Encerramento

Quando a fila estiver vazia (todas as tasks concluídas com sucesso):
1. Releia `tasks.md` e gere um resumo final:
   ```
   ✅ Execução concluída
   Tasks executadas: <lista de IDs>
   Tasks ainda pendentes no PRD: <lista, se houver>
   ```
2. Sugira próximos passos (ex.: rodar `do-execute-review` ou `do-execute-qa`).

## 🔒 CHECKPOINT OBRIGATÓRIO ANTES DE CADA DELEGAÇÃO

Antes de emitir **qualquer** chamada de `Task`, verifique as quatro condições abaixo. Se **uma** delas falhar, **NÃO** emita a chamada.

1. **É a ÚNICA chamada de delegação desta response?** Se você está prestes a emitir duas ou mais no mesmo bloco de tool calls — **PARE**. Remova todas menos a da task corrente.
2. **A task anterior já RETORNOU?** Delegação só acontece depois que o subagente da task anterior devolveu o bloco `TASK <ID>: ...`. Nunca antecipe.
3. **O retorno anterior já foi VERIFICADO?** Status `APROVADO`/`APROVADO COM OBSERVAÇÕES` **e** `tasks.md: [x] confirmado`. Qualquer outra coisa = parada da fila.
4. **Está em foreground?** Nada de qualquer parâmetro de background. Delegação em background quebra a serialização e faz a fila avançar antes da verificação.

⛔ **Execução paralela é PROIBIDA — sem exceções.** Não importa se as tasks "parecem independentes", se o usuário pediu "mais rápido", ou se a ferramenta sugere agrupar chamadas independentes. Tasks de um PRD compartilham a mesma árvore de arquivos e têm dependências sequenciais (a task N+1 lê e edita o código produzido pela task N). Duas delegações simultâneas gravam nos mesmos arquivos e no mesmo `tasks.md` → conflito de escrita, `[x]` perdido, código sobrescrito, review inconsistente. **Uma task por vez, sempre, mesmo que isso seja mais lento.**

Se o usuário pedir explicitamente paralelismo: **recuse**, explique em uma linha que a fila é serial por dependência de arquivos, e siga sequencial.

## Regras invioláveis

1. **UMA task por vez, em ordem numérica crescente — UM `Task` call por response.** Nunca emita dois ou mais `Task` no mesmo bloco. Nunca dispare a próxima task antes do retorno + verificação da anterior. Nunca execute em background.
2. **Sempre `subagent_type: "agent-execute-task"` (literal).** Nunca use `"general-purpose"` ou outro nome. Se não resolver, PARE e reporte. Nunca execute a skill inline.
3. **Ordem numérica estrita** (1, 2, 3, 4...). Não pule tasks salvo se o usuário pediu uma lista parcial específica.
4. **Falha = parada.** Em qualquer falha do subagente, pare o loop e reporte. Não tente "ajustar" tasks futuras.
5. **Nunca** edite `tasks.md` diretamente — quem marca `[x]` é a skill `do-execute-task` dentro do subagente.
6. **Mantenha-se magro — UMA `read_file` inicial, UMA final, mais nada.** Não releia `tasks.md` entre tasks. Não chame `list_dir` entre tasks. Não leia PRD/TechSpec/código. O contexto do main agent **não é limpo** entre tasks (acumula); a única forma de não estourar em filas longas é parar de gerar tool results redundantes. Confie no return estruturado do subagente.
7. Respeite as convenções do projeto descritas em `.cursor/rules/`, independentemente da stack.

## Exemplos de uso

```
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md 1.0-4.0
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md all
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md 1.0,3.0,5.0
```

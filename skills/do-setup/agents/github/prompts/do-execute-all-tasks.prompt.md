---
mode: agent
description: Itera sequencialmente sobre tasks de um PRD executando do-execute-task em subagentes isolados, com contexto limpo entre tasks. Orquestração inline (Copilot não permite nesting de subagents por padrão).
tools:
  - read_file
  - list_dir
  - runSubagent
---

# /do-execute-all-tasks

Itera sequencialmente sobre tasks de um PRD executando `do-execute-task` em subagentes isolados, com contexto limpo entre tasks. **A orquestração da fila roda no main agent** (este prompt) — não há orquestrador intermediário, porque o GitHub Copilot por padrão não permite que subagentes invoquem outros subagentes (a setting `chat.subagents.allowInvocationsFromSubagents` é `false` por default).

> ⛔ **TRÊS REGRAS DURAS — LEIA ATÉ O FIM ANTES DE QUALQUER COISA.**
>
> **REGRA 1 — SEQUENCIAL ESTRITO, ZERO PARALELISMO.**
> Você **DEVE** fazer **UMA ÚNICA chamada `#runSubagent` por response**. Você **JAMAIS** pode emitir 2 ou mais `#runSubagent` no mesmo bloco de tool calls. Você **JAMAIS** pode disparar a próxima task antes que a anterior tenha **retornado e sido verificada**. Tasks de PRD têm dependências sequenciais (task 2 depende do código da task 1, etc.) e violar a ordem destrói o trabalho.
>
> **REGRA 2 — Subagente em foreground SEMPRE.**
> Toda invocação `#runSubagent` **DEVE** rodar em foreground. **NUNCA** passe parâmetros que coloquem o subagente em background. Subagentes em background quebram a fila sequencial e fazem o orquestrador prosseguir antes da verificação dos artefatos.
>
> **REGRA 3 — Subagente alvo é LITERALMENTE `Agent Execute Task`.**
> Use **EXATAMENTE** o subagente `Agent Execute Task` (definido em `.github/agents/agent-execute-task.agent.md`). **JAMAIS** delegue a um subagente genérico ou diferente. Se o subagente não for resolvido (erro de configuração), **PARE** e reporte — não faça fallback.
>
> ---
>
> ⛔ **TOOLS QUE VOCÊ DEVE USAR NESTE PROMPT: APENAS `read_file`, `list_dir` e `runSubagent`.**
>
> Mesmo que outras tools estejam disponíveis no contexto, **NÃO as use neste prompt**. Você **NÃO** deve `insert_edit_into_file`, `replace_string_in_file`, `create_file` ou `run_in_terminal`. Toda implementação, edição de arquivos, execução de testes e criação de reviews acontece **OBRIGATORIAMENTE** dentro do subagente `Agent Execute Task` invocado via `#runSubagent`.

## Entrada do usuário

A solicitação completa está em `${input:request}` (ou no texto que acompanhou o comando). Espera-se:

1. **Caminho do `tasks.md`** — obrigatório. Ex.: `prds/<nome-do-prd>/tasks/tasks.md`.
2. **Filtro de tasks** — opcional, default `all`:
   - `all` / `todas` → todas as pendentes (não marcadas com `[x]`)
   - Lista de IDs separados por vírgula (ex.: `1.0, 2.0, 5.0`)
   - Range (ex.: `1.0-4.0`)

Se o caminho não for fornecido, **pergunte uma única vez** antes de prosseguir.

## Procedimento

### 1. Descoberta inicial (uma única vez)

1. Faça o parse da entrada extraindo `<caminho>` e `<filtro>` (default `all` se ausente).
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

#### 2.2. Delegação em subagente isolado via #runSubagent

Invoque o tool **`#runSubagent`** delegando ao subagente `Agent Execute Task` (definido em `.github/agents/agent-execute-task.agent.md`) com prompt autocontido contendo:

- ID da task corrente
- Caminho do PRD (ex.: `prds/prd-<nome>/`)
- Caminho exato do task file (ex.: `prds/prd-<nome>/tasks/<num>_task.md`)
- Reforço de seguir o `SKILL.md` da `do-execute-task` na íntegra

Exemplo de prompt para o subagente:

> "Execute a task `<ID>` do PRD localizado em `prds/prd-<nome>/`. Task file: `prds/prd-<nome>/tasks/<num>_task.md`. Siga RIGOROSAMENTE o `SKILL.md` da skill `do-execute-task` na íntegra (Step 0 a Step 8): leitura de PRD/TechSpec, análise, implementação, gate de testes, marcação `[x]` em `tasks.md`, code review, criação do arquivo `<num>_task_review.md` e gate final de artefatos. Não pule etapas. Não pare para pedir confirmação. Retorne resposta curta no formato definido pelo agente."

**Apenas UM `#runSubagent` por response.** Esta chamada é o ÚNICO tool call no bloco — não combine com outros `#runSubagent` da próxima task da fila.

**Aguarde o retorno do subagente** antes de prosseguir. Cada invocação roda em sessão fresca (isolated context window), sem herdar histórico nem tool results da task anterior — isto é o isolamento real.

> Se o tool `#runSubagent` estiver indisponível ou o subagente não resolver, **PARE** e reporte ao usuário. Não tente executar a skill inline.

#### 2.3. Verificação de conclusão (apenas pelo return do subagente)

Após o retorno do subagente, **NÃO releia `tasks.md` nem chame `list_dir` aqui** — isso acumularia tool results no contexto do main agent a cada iteração e estouraria a janela em filas longas. O subagente `Agent Execute Task` já fez o gate final (Step 8 do `do-execute-task`) verificando que `tasks.md` está marcado `[x]` e que o `<num>_task_review.md` existe. Confie no return estruturado.

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

## Regras invioláveis

1. **UMA task por vez, em ordem numérica crescente — UM `#runSubagent` call por response.** Nunca emita dois ou mais `#runSubagent` no mesmo bloco. Nunca dispare a próxima task antes do retorno + verificação da anterior. Nunca execute em background.
2. **Sempre delegue a `Agent Execute Task` (literal).** Nunca use outro subagente. Se não resolver, PARE e reporte. Nunca execute a skill inline.
3. **Ordem numérica estrita** (1, 2, 3, 4...). Não pule tasks salvo se o usuário pediu uma lista parcial específica.
4. **Falha = parada.** Em qualquer falha do subagente, pare o loop e reporte. Não tente "ajustar" tasks futuras.
5. **Nunca** edite `tasks.md` diretamente — quem marca `[x]` é a skill `do-execute-task` dentro do subagente.
6. **Mantenha-se magro — UMA `read_file` inicial, UMA final, mais nada.** Não releia `tasks.md` entre tasks. Não chame `list_dir` entre tasks. Não leia PRD/TechSpec/código. O contexto do main agent **não é limpo** entre tasks (acumula); a única forma de não estourar em filas longas é parar de gerar tool results redundantes. Confie no return estruturado do subagente.
7. Respeite as convenções do projeto descritas em `.github/copilot-instructions.md`.

## Exemplo de uso

```
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md 1.0-4.0
/do-execute-all-tasks prds/<nome-do-prd>/tasks/tasks.md all
```

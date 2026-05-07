# /execute-all-tasks

Itera sequencialmente sobre tasks de um PRD executando `do-execute-task` em cada uma, com isolamento de contexto via subagente.

## Argumentos

Espera-se na mensagem que acompanhar o comando:

1. **Caminho do `tasks.md`** — obrigatório. Ex.: `prds/prd-<nome>/tasks/tasks.md`.
2. **Filtro de tasks** — opcional, default `all`:
   - `all` / `todas` → todas as pendentes (não marcadas com `[x]`)
   - Lista de IDs separados por vírgula (ex.: `1.0, 2.0, 5.0`)
   - Range (ex.: `1.0-4.0`)

Se faltar o caminho do `tasks.md`, **pergunte uma única vez** antes de prosseguir.

## Procedimento

1. Faça o parse dos argumentos extraindo `<caminho>` e `<filtro>` (default `all` se ausente).
2. Delegue **integralmente** a execução para o subagente `execute-all-tasks` (definido em `.cursor/agents/execute-all-tasks.md`).
3. O subagente roda com contexto isolado — não herda o histórico desta conversa. Passe no prompt de invocação:
   - Caminho exato do `tasks.md`
   - Filtro de tasks
   - Instrução para seguir o procedimento do subagente na íntegra (descoberta → loop → limpeza de contexto → encerramento)
4. **Não** execute o procedimento você mesmo — apenas delegue. O subagente é quem orquestra.
5. Após o retorno do subagente, repasse o resumo final ao usuário sem alterações.

## Exemplos de uso

```
/execute-all-tasks prds/prd-login-google/tasks/tasks.md 1.0-4.0
/execute-all-tasks prds/prd-login-google/tasks/tasks.md all
/execute-all-tasks prds/prd-login-google/tasks/tasks.md 1.0,3.0,5.0
```

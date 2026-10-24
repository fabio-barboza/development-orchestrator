# Contrato do Subagente `agent-execute-task`

Lido do disco pelo subagente `agent-execute-task` antes de executar a task. O arquivo do agente na ferramenta de IA contém só permissões e um ponteiro; **todas as regras vivem aqui**, para que um arquivo de agente em cache nunca imponha regras antigas.

## Entrada esperada

Quem te invoca (a skill `do-execute-all-tasks`, rodando no agente primário) deve fornecer:

1. **ID da task** (ex.: `1.0`, `2.0`).
2. **Caminho do PRD** (ex.: `prds/prd-<feature-slug>/`) ou o caminho direto do arquivo da task (ex.: `prds/prd-<feature-slug>/tasks/1_task.md`). **Convenção rígida**: o filename é sempre `<inteiro>_task.md` (ex.: `1_task.md`, `11_task.md`) — nunca `1.0_task.md`, nunca com slug descritivo.

Se faltar qualquer informação, **HALT** com erro claro — não pergunte ao usuário (você é um subagente; o orquestrador é quem interage com o usuário).

## Procedimento

1. Leia a skill `do-execute-task` (`SKILL.md`) na íntegra.
2. Siga **na íntegra** o procedimento dela, sem atalhos (Step 0 a Step 8): detecção da ferramenta, leitura do task file, PRD, TechSpec e `tasks.md`, carga das skills e MCPs do projeto, análise, implementação, gate de testes, marcação `[x]`, auto code review, criação do `[num]_task_review.md` e gate final de artefatos. Não pule etapas e não pare para pedir confirmação ao usuário.
3. O manifesto final (`📋 Artefatos`) do Step 8 **não se aplica** quando você roda como subagente: devolva somente o bloco abaixo.

## Resposta final ao orquestrador

Curta e estruturada:

```
TASK <ID>: <APROVADO | APROVADO COM OBSERVAÇÕES | MUDANÇAS SOLICITADAS | FALHA>
- tasks.md: [x] confirmado | NÃO marcado
- review: prds/prd-<slug>/tasks/<num>_task_review.md
- testes: <passando | N falhando>
- observações: <1-2 linhas, opcional>
```

Não devolva longos resumos de implementação — apenas o status. O orquestrador acumula as respostas de N subagentes e verbosidade causa estouro de contexto.

## Regras invioláveis

1. **Uma task por invocação.** Se o prompt mencionar mais de um ID, execute **apenas o primeiro** e reporte no campo `observações` que os demais foram ignorados — a serialização da fila é do orquestrador.
2. **Siga o `SKILL.md` na íntegra.** Sem atalhos, sem improvisações.
3. **Não interaja com o usuário.** Em caso de ambiguidade não resolvível, HALT com erro estruturado.
4. **Falha = report de falha.** Se testes não passarem ou artefatos não forem criados, retorne `FALHA` com o motivo. Não marque `[x]`.
5. **NUNCA use tools interativas** (a de perguntas ao usuário): numa sessão-filha o prompt pode não ser exibido e a fila inteira trava. Em caso de ambiguidade, decida com a informação disponível ou retorne `FALHA`.
6. **NUNCA delegue.** Você é a folha da árvore: não tente iniciar outro subagente (estoura o limite de profundidade e trava a fila).
7. **SEMPRE termine com a resposta estruturada**, mesmo em erro, timeout de teste ou interrupção parcial. Um subagente sem texto final deixa o orquestrador sem sinal de conclusão.
8. **Após compactação de contexto, releia** o `SKILL.md` da `do-execute-task` e o arquivo da task antes de continuar: as regras que estavam só no contexto descartado (limites do workspace, gate final) continuam valendo.
9. Os arquivos `do-shared/...` citados nas skills ficam no diretório de skills, ao lado de `do-execute-task`. Leia-os pelo caminho completo: buscas por glob ignoram diretórios ocultos como `.agents/`.

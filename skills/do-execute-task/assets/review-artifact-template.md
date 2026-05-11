# Review: Task [num] - [Título da Task]

**Revisor**: AI Code Reviewer
**Data**: [YYYY-MM-DD]
**Arquivo da task**: [num]_task.md
**Status**: [APROVADO | APROVADO COM OBSERVAÇÕES | MUDANÇAS SOLICITADAS]

## Resumo

[Breve resumo do que foi implementado e a avaliação geral de qualidade]

## Arquivos Revisados

| Arquivo | Status | Problemas |
|---------|--------|-----------|
| [caminho do arquivo] | [✅ OK / ⚠️ Problemas / ❌ Crítico] | [quantidade] |

## Problemas Encontrados

### 🔴 Problemas Críticos

[Liste cada problema crítico com arquivo, linha, descrição e correção sugerida]
[Se nenhum: "Nenhum problema crítico encontrado."]

### 🟡 Problemas Maiores

[Liste cada problema maior com arquivo, linha, descrição e correção sugerida]
[Se nenhum: "Nenhum problema maior encontrado."]

### 🟢 Problemas Menores

[Liste cada problema menor com arquivo, linha, descrição e correção sugerida]
[Se nenhum: "Nenhum problema menor encontrado."]

## ✅ Destaques Positivos

[Liste as coisas que foram bem feitas]

## Conformidade com Padrões

| Padrão | Status |
|--------|--------|
| Padrões de Código | [✅ / ⚠️ / ❌] |
| Linguagem/Runtime | [✅ / ⚠️ / ❌] |
| REST/HTTP | [✅ / ⚠️ / ❌] (se aplicável) |
| Logging | [✅ / ⚠️ / ❌] (se aplicável) |
| Framework UI | [✅ / ⚠️ / ❌] (se aplicável) |
| Testes | [✅ / ⚠️ / ❌] |

## Conformidade com Identidade Visual

> **Aplicabilidade**: incluir esta seção APENAS quando a tarefa modifica arquivos de UI (`task_surface = visual`). Para tarefas backend, marque a seção inteira como `N/A` ou remova-a.
>
> **Simetria por severidade (3 tiers)**:
> - Conformidade total → **APROVADO** ou **APROVADO COM OBSERVAÇÕES**. Pipeline continua.
> - Gaps **MAIOR/MENOR** após `visual_retry = 2/2` → **APROVADO COM OBSERVAÇÕES**. Pipeline continua (cosmético segue).
> - Gaps **CRÍTICO** após `visual_retry = 2/2` (4+ classes/seletores sem regra, tela visualmente quebrada, theme exigido não aplicado) → **MUDANÇAS SOLICITADAS** e **PIPELINE HALTED**, paralelo a teste falhando.
>
> **Task-level gate**: o executor TEM que ter rodado o ACTIVE RETRY LOOP de `do-execute-task` Step 6.5. O contador `visual_retry` abaixo prova que o retry rodou.

**visual_retry**: `[0/2 | 1/2 | 2/2]` — número de ciclos de retry visual executados nesta task.

**Severidade dos gaps residuais**: `[ Nenhum | MENOR | MAIOR | CRÍTICO ]` — severidade máxima das lacunas que persistiram após o retry. `CRÍTICO` HALTA o pipeline.

| Critério | Status |
|----------|--------|
| Cobertura de classes/seletores (UI → arquivo de estilo) | [✅ / ⚠️ / ❌ / N/A] |
| Uso de design tokens (sem literais hardcoded) | [✅ / ⚠️ / ❌ / N/A] |
| Adaptação a tela (breakpoints web / regras mobile) | [✅ / ⚠️ / ❌ / N/A] |
| Temas dinâmicos (se PRD especifica) | [✅ / ⚠️ / ❌ / N/A] |
| Convenção de naming (conforme TechSpec) | [✅ / ⚠️ / ❌ / N/A] |

### Detalhes de Identidade Visual

[Descrever classes/seletores verificados, tokens usados, e qualquer lacuna identificada.
Listar referências de UI que não tinham regras correspondentes ANTES do retry.
Para cada ciclo de retry, descrever: o que foi tentado, o que foi fechado, o que persistiu.
Se `visual_retry = 2/2` e ainda há gap, listar para cada gap residual: (arquivo, referência, regra/token esperado, motivo do retry não ter fechado).
Se `task_surface = backend`, escrever apenas: "N/A — tarefa backend, sem alterações de UI."]

## Recomendações

[Lista numerada de recomendações priorizadas para melhoria]

## Veredito

[Avaliação final com próximos passos claros]

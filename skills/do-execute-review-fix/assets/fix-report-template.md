# Relatório de Correção de Review - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Review Original: [status original — NEEDS_REVISION / REJECTED]
- Total de Findings: [X] (CRITICAL: [a] | MAJOR: [b] | MINOR: [c])
- Findings Corrigidos: [Y]
- Findings Não Resolvidos: [Z]

## Resultado por Finding

| ID | Severidade | Descrição | Status | Correção Aplicada |
|----|------------|-----------|--------|-------------------|
| R-01 | CRITICAL | [descrição do finding] | Corrigido / Não Resolvido | [descrição da correção ou bloqueio] |
| R-02 | MAJOR | [descrição do finding] | Corrigido / Não Resolvido | [descrição da correção ou bloqueio] |
| R-03 | MINOR | [descrição do finding] | Corrigido / Ignorado | [descrição ou justificativa] |

## Testes
- Testes unitários: TODOS PASSANDO / [X] FALHANDO
- Testes de integração: TODOS PASSANDO / [X] FALHANDO
- Tipagem: SEM ERROS / [X] ERROS

## Findings Não Resolvidos
| ID | Motivo do Bloqueio | Ação Recomendada |
|----|---------------------|------------------|
| [ID] | [descrição do bloqueio] | [o que o desenvolvedor deve fazer] |

## Próximo Passo
[Se tudo resolvido]: Rodar `do-execute-review` novamente para fechar o loop de review.
[Se há CRITICAL não resolvidos]: NÃO prosseguir para QA — resolver os bloqueios listados acima primeiro.

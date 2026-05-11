# Tarefa X.0: [Título da Tarefa]

<!-- Filename rule: este arquivo DEVE ser salvo como `<X>_task.md` (ex.: 1_task.md, 11_task.md). Nunca use `X.0_task.md` nem inclua o título no filename. -->
<!-- NÃO crie um segundo arquivo com `<X>.0_task.md` para a mesma task. Existe APENAS UM arquivo por task: `<X>_task.md`. O `X.0` aparece somente no título acima e nas subtarefas (X.1, X.2), nunca no nome do arquivo. -->

<critical>Ler os arquivos de prd.md e techspec.md desta pasta, se você não ler esses arquivos sua tarefa será invalidada</critical>

## Visão Geral

[Breve descrição da tarefa]

<requirements>
[Lista de requisitos obrigatórios]
</requirements>

## Subtarefas

- [ ] X.1 [Descrição da subtarefa]
- [ ] X.2 [Descrição da subtarefa]

## Detalhes de Implementação

[Seções relevantes da spec técnica **NÃO PRECISA MOSTRAR TODA A IMPLEMENTAÇÃO, APENAS REFERENCIE A techspec.md**]

## Critérios de Sucesso

- [Resultados mensuráveis]
- [Requisitos de qualidade]

## Testes da Tarefa

- [ ] Testes de unidade
- [ ] Testes de integração
- [ ] Testes E2E (se aplicável)

<critical>SEMPRE CRIE E EXECUTE OS TESTES DA TAREFA ANTES DE CONSIDERÁ-LA FINALIZADA</critical>

## Identidade Visual

> **Aplicabilidade**: incluir esta seção APENAS se a tarefa toca arquivos de UI (componentes, telas, folhas de estilo) em um projeto com camada visual (frontend / mobile / full-stack). Para tarefas puramente backend (controllers, services, migrations, jobs, workers) ou em projetos backend-only, REMOVER esta seção inteira ao gerar o arquivo da tarefa.

[Para tarefas que tocam UI, especificar:
- Arquivos de estilo a modificar (ex.: `index.css`, `App.css`, `*.module.css`, `StyleSheet` nativo, Flutter `ThemeData`)
- Classes/seletores a criar (CSS, BEM, utility classes, etc., conforme metodologia declarada no TechSpec)
- Design tokens a usar (variáveis CSS, theme object keys, platform tokens)
- Breakpoints / regras adaptativas a respeitar (web: 320/768/1024; mobile: tamanho de tela, orientação)
- Temas a aplicar]

<critical>Toda classe/seletor de estilo usado na UI DEVE ter regra correspondente no arquivo de estilo. Lacunas serão detectadas no review e DOCUMENTADAS na conclusão — não interrompem o fluxo.</critical>

## Arquivos relevantes

- [Arquivos relevantes desta tarefa]

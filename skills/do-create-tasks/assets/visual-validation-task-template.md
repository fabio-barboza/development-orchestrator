# Tarefa <N>.0: Validação de Identidade Visual / CSS

<!-- Filename rule: este arquivo DEVE ser salvo como `<N>_task.md` (ex.: 5_task.md, 12_task.md). Nunca use `<N>.0_task.md` nem inclua o título no filename. -->

<critical>Ler `prd.md` (seção "Identidade Visual") e `techspec.md` (seção "Implementação Visual") desta pasta antes de iniciar. Se essas seções não existirem, halte e reporte ao usuário — esta tarefa pressupõe identidade visual declarada.</critical>

## Visão Geral

Validar que a identidade visual declarada no PRD e a estratégia de implementação do TechSpec foram aplicadas de forma consistente nos componentes UI desta feature. Esta tarefa é gerada automaticamente por `do-create-tasks` quando `project_surface = visual`. Ela NÃO inventa identidade visual nova — apenas valida e fecha lacunas em relação ao que já está declarado.

<requirements>
- Todos os arquivos UI modificados pela feature devem ser cobertos.
- Toda classe/seletor referenciado na UI deve ter regra correspondente no arquivo de estilo.
- Valores temáveis (cores, espaçamentos, tipografia) devem usar tokens declarados no TechSpec, sem literais hardcoded equivalentes.
- Breakpoints (web) ou regras adaptativas (mobile) declarados devem ser respeitados.
- Se o PRD especifica temas dinâmicos, a troca de tema deve funcionar nos componentes desta feature.
- As verificações devem ser cobertas por testes automatizados que rodem na suíte padrão do projeto.
</requirements>

## Subtarefas

- [ ] <N>.1 Listar todos os arquivos UI modificados pela feature (componentes, telas, folhas de estilo) usando `git diff` contra o ponto de divergência da branch.
- [ ] <N>.2 Cobertura de classes/seletores: para cada arquivo UI listado, extrair as classes/seletores referenciados e verificar regra correspondente nos arquivos de estilo. Para cada lacuna, gerar a regra faltante seguindo a metodologia declarada no TechSpec.
- [ ] <N>.3 Uso de design tokens: identificar literais hardcoded (cores, espaçamentos, tipografia, border-radius, sombras) que tenham token equivalente declarado no TechSpec; substituir cada literal pelo token correspondente.
- [ ] <N>.4 Adaptação a tela: verificar que os breakpoints (web) ou as regras adaptativas (mobile) declarados no TechSpec estão aplicados nos componentes da feature. Adicionar regras faltantes.
- [ ] <N>.5 Temas dinâmicos (apenas se o PRD especifica): verificar que componentes usam tokens themeáveis e que a troca de tema funciona nos componentes desta feature.
- [ ] <N>.6 Escrever testes automatizados na suíte do projeto cobrindo: (a) presença das classes/seletores esperados no output renderizado, (b) ausência de literais hardcoded equivalentes a tokens declarados, (c) comportamento responsivo nos breakpoints declarados (web) ou regras adaptativas (mobile).

## Detalhes de Implementação

Fontes de verdade:

- PRD seção "Identidade Visual": paleta, tipografia, espaçamento, breakpoints, temas, acessibilidade.
- TechSpec seção "Implementação Visual": metodologia de estilo, nomes exatos dos tokens, arquitetura de tema, organização de arquivos.

Esta tarefa NÃO inventa identidade visual nova. Quando encontrar lacuna, fechar segundo o que já está declarado nesses dois documentos.

## Critérios de Sucesso

- Zero classes/seletores referenciados na UI sem regra correspondente.
- Zero literais hardcoded em propriedades com token equivalente declarado no TechSpec.
- Breakpoints/regras adaptativas declarados aplicados nos componentes da feature.
- Tema dinâmico (se aplicável) funciona em todos os componentes da feature.
- Testes automatizados da subtarefa <N>.6 passando na suíte do projeto.

## Testes da Tarefa

- [ ] Testes de unidade: presença das classes/seletores esperados no output renderizado.
- [ ] Testes de integração: aplicação dos design tokens declarados (sem literais hardcoded equivalentes).
- [ ] Testes E2E (se aplicável): comportamento responsivo nos breakpoints declarados (web) ou regras adaptativas (mobile).

<critical>SEMPRE CRIE E EXECUTE OS TESTES DA TAREFA ANTES DE CONSIDERÁ-LA FINALIZADA</critical>

## Arquivos relevantes

- Arquivos UI modificados pela feature (listados em <N>.1).
- Arquivos de estilo do projeto (`*.css`, `*.module.css`, `theme.ts`, StyleSheet nativo, Flutter `ThemeData`, etc.) referenciados na seção "Implementação Visual" do TechSpec.

# Template de Product Requirements Document (PRD)

## Visão Geral

[Forneça uma visão geral de alto nível do seu produto/funcionalidade. Explique qual problema resolve, para quem é e por que é valioso.]

## Objetivos

[Liste objetivos específicos e mensuráveis para esta funcionalidade:

- Como é o sucesso
- Métricas principais para acompanhar
- Objetivos de negócio a alcançar]

## Histórias de Usuário

[Detalhe as narrativas do usuário descrevendo uso e benefícios da funcionalidade:

- Como [tipo de usuário], eu quero [realizar uma ação] para que [benefício]
- Inclua personas de usuários primários e secundários
- Cubra fluxos principais e casos extremos]

## Funcionalidades Principais

[Liste e descreva as funcionalidades principais do seu produto. Para cada funcionalidade, inclua:

- O que ela faz
- Por que é importante
- Como funciona em alto nível
- Requisitos funcionais (numerados para clareza)]

## Experiência do Usuário

[Descreva a jornada e experiência do usuário:

- Personas de usuário e suas necessidades
- Fluxos e interações principais do usuário
- Considerações e requisitos de UI/UX
- Requisitos de acessibilidade]

### Identidade Visual

> **Aplicabilidade**: incluir esta seção APENAS se a feature tem camada visual (frontend, mobile, full-stack). Para projetos backend-only (API, CLI, worker, library), REMOVER a seção inteira ao gerar o PRD.

[Especificar a identidade visual da feature, adaptando ao tipo de projeto:

- **Paleta de cores**: cores primária, secundária, accent, neutras, semânticas (error, success, warning)
- **Tipografia**: fontes, escalas de tamanho (heading, body, caption)
- **Layout**: convenções de espaçamento (padding/margin scale), border-radius
  - Web: grid/flexbox
  - Mobile: layouts nativos (Stack, Flex, AutoLayout, ConstraintLayout)
- **Breakpoints / adaptação a tela**:
  - Web: mobile-first (320px+), tablet (768px+), desktop (1024px+)
  - Mobile: orientação portrait/landscape, tablet vs phone
- **Temas dinâmicos**: condições que disparam mudança de tema (dark/light, contextual), paleta por tema
- **Design tokens**: abordagem para tokens reutilizáveis (CSS variables, theme objects, design system, etc.)
- **Metodologia de estilo**: a escolha do projeto (CSS variables + BEM, CSS-in-JS, utility-first/Tailwind, CSS Modules, StyleSheet nativo, etc.) — declarar qual é, sem prescrever
- **Acessibilidade**: nível alvo (web: WCAG AA, contraste 4.5:1 | mobile: diretrizes de acessibilidade da plataforma)]

## Restrições Técnicas de Alto Nível

[Capture apenas restrições e considerações de alto nível (**evite soluções de design – essas pertencem à Tech Spec**):

- Integrações externas requeridas ou sistemas existentes para interfacear
- Mandatos de conformidade, regulatórios ou de segurança
- Metas de performance/escalabilidade (ex: TPS esperado, limites superiores de latência)
- Considerações de sensibilidade de dados/privacidade
- Requisitos não negociáveis de tecnologia ou protocolo

Detalhes de implementação serão abordados na Especificação Técnica.]

## Fora de Escopo

[Declare claramente o que esta funcionalidade NÃO incluirá para gerenciar o escopo:

- Funcionalidades explicitamente excluídas
- Considerações futuras que estão fora de escopo
- Limites e limitações

(Nota: Riscos de implementação técnica serão detalhados na Tech Spec.)]

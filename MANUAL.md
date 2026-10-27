# Manual de Uso — DO Framework

Este manual mostra como usar o DO Framework do começo ao fim, passo a passo.

> Para detalhes completos de cada skill, consulte o [README.md](README.md).

> **Ferramentas testadas:** As skills e agents do DO Framework foram testados com **Opencode**, **Claude Code**, **Cursor** e **GitHub Copilot**. Por serem implementados como skills (instruções em arquivos de texto), possivelmente funcionam com outras ferramentas que suportem o mesmo mecanismo — mas não há garantia de compatibilidade.

## Economia de Tokens — Use o Modelo Certo em Cada Fase

Cada etapa do framework opera de forma completamente independente. O contexto de uma fase não é carregado na próxima — apenas os artefatos gerados (arquivos `.md`) são passados adiante. Isso permite uma estratégia de custo eficiente:

| Fase | Skills | Modelo recomendado | Por quê |
|------|--------|--------------------|---------|
| **Planejamento** | `do-create-prd`, `do-create-techspec`, `do-create-tasks` | Mais inteligente (ex: Opus, o1) | Requer raciocínio profundo, ambiguidade alta, decisões de arquitetura |
| **Execução** | `do-execute-task`, `do-execute-review`, `do-execute-qa`, `do-execute-qa-fix` | Menor e mais rápido (ex: Sonnet, Haiku) | Segue instruções precisas dos artefatos gerados, contexto já bem definido |

> Troque o modelo entre as fases sem perder contexto — os artefatos em `./prds/` são a memória do projeto.

---

## Passo 0 — Instalar as skills

Na raiz do seu projeto, execute:

```bash
npx skills add fabio-barboza/development-orchestrator
```

Isso instala todas as skills do framework (`do-setup`, `do-create-prd`, `do-create-techspec`, etc.) no seu ambiente de IA.

> **Instale o framework em um lugar só: no projeto ou global, não nos dois.** Se o mesmo nome de skill existir nos dois, a ferramenta decide qual carregar (no Claude Code a global vence a do projeto; nas outras a documentação não diz), e você pode rodar uma versão diferente da esperada. O `do-setup` avisa quando encontra cópias globais de skills `do-*`.

### Atualizar o framework em um projeto que já o usa

Um `npx skills add` repetido **não sobrescreve** as skills já instaladas. Para trazer uma versão nova:

1. `npx skills update` (ou `npx skills remove` das skills `do-*` e `npx skills add` de novo; para testar uma branch, `npx skills add "fabio-barboza/development-orchestrator#<branch>"`).
2. `/do-setup agents`: reescreve o worker (`agent-execute-task`) e, no Opencode, os comandos finos. Não mexe em `.do/context.md`.
3. Reinicie a ferramenta. O Opencode mantém agentes e comandos em cache num servidor persistente; sem reiniciar, o worker antigo continua valendo.

> **Skill renomeada:** `do-execute-qa-bugfix` agora é `do-execute-qa-fix`. Quem já a tinha instalada fica com a antiga ao lado da nova: remova a antiga (`npx skills remove do-execute-qa-bugfix`) e apague `.opencode/commands/do-execute-qa-bugfix.md`, se existir. Atalhos e anotações que citam o nome antigo também precisam ser trocados.

> **Vindo de uma versão anterior à 1.1.0:** o contexto do framework saiu do `CLAUDE.md`/`AGENTS.md` e foi para `.do/context.md`, e os rascunhos saíram de `.do-tmp/` para `.do/tmp/`. Rode `/do-setup` (completo) uma vez para gerar `.do/context.md`. O framework não edita mais o seu arquivo de instruções: o `do-setup` avisa quais seções antigas (`Available Skills`, `Uncovered Skills`, `Available MCPs`, `Project Commands`) ainda estão lá, para você remover à mão, e se a linha `.do-tmp/` do `.gitignore` pode sair.

PRDs, TechSpecs e tasks já gerados **não** se adaptam sozinhos: se mudou a regra de geração (por exemplo, tasks sem task de E2E), só um PRD novo reflete a mudança.

> ⚠️ **Se as skills ou os comandos `/do-*` não aparecerem após o `npx skills add` ou o `do-setup`, recarregue a ferramenta.** O jeito depende dela:
>
> | Ferramenta | Como recarregar |
> |-----------|-----------------|
> | **Opencode** | `/reload` |
> | **Claude Code** | Skills novas entram sozinhas; se a pasta `.claude/skills/` foi criada agora, rode `/reload-skills`. A pasta `.claude/agents/` criada pelo `do-setup` só é detectada **ao reiniciar a sessão** |
> | **Cursor** e **GitHub Copilot** | A documentação não especifica. Se não aparecerem, recarregue a janela do IDE ou abra uma nova sessão de chat |

### Adicionar skills do seu projeto (opcional, mas recomendado)

Além das skills do DO Framework, você pode instalar skills específicas para a tecnologia e as práticas do seu projeto, pelo mesmo mecanismo de instalação. Elas enriquecem o contexto técnico durante a criação de TechSpec e a execução de tasks.

> **Instale as skills (e configure os MCPs) antes de gerar o PRD e a TechSpec.** Quanto mais cedo uma skill entra, mais ela influencia o resultado:
>
> | Tipo de skill | Onde pesa | Se instalada depois dos documentos |
> |---------------|-----------|-------------------------------------|
> | Muda decisões de arquitetura ou padrões | **TechSpec** | pode exigir revisar ou regerar a TechSpec |
> | Padrões de implementação | **execução** das tasks | basta `/do-setup refresh` |
> | Revisão ou QA | **review e QA** | basta `/do-setup refresh` |
>
> As tasks já geradas continuam válidas (elas não dependem de skills específicas, mesmo que citem alguma de passagem; o executor escolhe a skill pelo "quando usar"). Já a TechSpec foi escrita sem consultar a skill nova: se ela alterar uma decisão, ajuste a TechSpec (à mão ou pedindo ao agente) antes de executar.

> Procure skills disponíveis em [skills.sh](https://skills.sh). Nenhuma skill além das `do-*` é obrigatória: o framework funciona sem elas. Após instalar, o `do-setup` (ou `/do-setup refresh`) detecta quais são relevantes ao projeto e as registra em `.do/context.md`.

---

## Passo 1 — Configurar MCPs (opcional, mas recomendado)

MCPs habilitam testes E2E automáticos e consulta a documentação. O framework os descobre pelas **capacidades** que oferecem (`ui-testing`, `api-testing`, `messaging`, `data-store`, `documentation`), sem depender de nenhum servidor específico.

**Claude Code** — adicione ao `.mcp.json` na raiz do projeto (os valores entre `<>` vêm da documentação do MCP escolhido):

```json
{
  "mcpServers": {
    "<nome-do-mcp>": {
      "type": "stdio",
      "command": "<comando>",
      "args": ["<argumentos>"]
    }
  }
}
```

Para outras ferramentas (Cursor, Copilot, Opencode), veja a tabela no [README.md](README.md#como-configurar-mcps-no-seu-projeto).

> Prefira configurar os MCPs **por projeto**, não globalmente: cada MCP entra no prompt de toda sessão, mesmo em projetos que não o usam, e as skills descobrem os MCPs lendo o arquivo de configuração de MCP do projeto.

---

## Passo 2 — Inicializar o projeto

Rode **uma única vez** por projeto:

```
/do-setup
```

Isso analisa o codebase, grava o contexto do framework em `.do/context.md` (resumo, convenções, comandos, skills e MCPs) e **instala os agents de orquestração** nos diretórios corretos da sua ferramenta de IA.

> **O `do-setup` nunca altera um `CLAUDE.md`, `AGENTS.md` ou equivalente que já existe.** Esses arquivos são do projeto e de todo o time, inclusive de quem não usa o framework ou tem outras skills e MCPs instalados. O framework só os lê, para as convenções. Se o projeto não tem nenhum, o `do-setup` pergunta se deve criar um (`CLAUDE.md` no Claude Code, `AGENTS.md` no Opencode, no Cursor e no GitHub Copilot, que leem esse arquivo) com o que um `/init` geraria: resumo, comandos e convenções, em inglês (o idioma padrão do `/init`), sem nenhuma referência ao framework, a skills ou a MCPs. O arquivo não é adicionado ao git: você revisa e decide se faz o commit. Tudo do framework fica em `.do/`, uma pasta local de cada desenvolvedor, ignorada pelo git:
>
> ```
> .do/
>   context.md   # gerado pelo do-setup
>   tmp/         # rascunhos das skills de execução, apagados ao fim de cada execução
> ```

O `do-setup` também **inventaria o que está instalado** e informa no relatório final:

- **Skills** (além das `do-*`), detectadas pela lista da sua ferramenta e pelos diretórios de skills do projeto e globais;
- **MCPs** configurados, com a capacidade de cada um (`ui-testing`, `documentation`, etc.);
- **Lacunas**, por exemplo "sem MCP de interface: o E2E de interface será pulado".

O `do-setup` também registra os **comandos do próprio projeto** (instalar dependências, rodar testes, rodar testes com cobertura, checagem estática, iniciar o serviço local) na seção `Project Commands` de `.do/context.md`, lidos do que o projeto já declara; as skills de execução usam esses comandos em vez de presumir uma linguagem ou ferramenta. Se o projeto é um repositório git, o `do-setup` acrescenta `.do/` ao `.gitignore`. O resultado do inventário fica nas seções `Available Skills` e `Available MCPs` de `.do/context.md`. Nenhuma skill além das `do-*` é presumida: se não houver nenhuma, é só um aviso, não um erro. Durante a execução, as skills do framework consultam essas seções (e a lista nativa da ferramenta), leem as skills relevantes à tarefa e registram no review quais usaram.

> **Opencode (apenas na 1ª execução):** o opencode **não reconhece skills via `/<skill>`** — então a *primeira* invocação do `do-setup` precisa ser em linguagem natural:
> ```
> Execute do-setup
> ```
> O `do-setup` então copia **todos os slash commands `/do-*`** (`/do-setup`, `/do-create-prd`, `/do-create-techspec`, `/do-create-tasks`, `/do-execute-task`, `/do-execute-all-tasks`, etc.) para `.opencode/commands/`. **A partir daí, todos os comandos do framework funcionam com a sintaxe `/` normalmente no opencode** — inclusive `/do-setup` em re-execuções.

> Se você já fez o setup anteriormente e quer apenas reinstalar os agents (ex: após atualizar as skills), use:
> ```
> /do-setup agents
> ```

> Se instalou **novas skills ou MCPs depois do setup**, atualize só a lista registrada em `.do/context.md` com:
> ```
> /do-setup refresh
> ```
> Ele redetecta skills e MCPs e reescreve apenas as seções `Available Skills`, `Uncovered Skills` e `Available MCPs`, sem reanalisar o projeto nem reinstalar os agents. Exige que o setup completo já tenha sido feito.

---

## Passo 3 — Criar o PRD

Descreva a feature que você quer construir:

```
/do-create-prd adicionar autenticação de usuários
# ou passando um arquivo de contexto:
/do-create-prd adicionar <arquivo>
```

A skill vai fazer algumas perguntas para entender o problema, objetivos e flows. Ao final, gera:

```
./prds/prd-autenticacao-usuarios/prd.md
```

---

## Passo 4 — Criar a TechSpec

Com o PRD pronto, defina como implementar. A TechSpec traz uma seção de **cenários de aceite**: os fluxos que o QA vai validar depois (eles não viram tasks):

```
/do-create-techspec <prd>
```

A skill pesquisa documentação oficial (via MCP de documentação, se disponível), explora o codebase e faz perguntas técnicas. Ao final, gera:

```
./prds/prd-autenticacao-usuarios/techspec.md
```

---

## Passo 5 — Criar as Tasks

Com o TechSpec pronto, quebre o trabalho em tasks executáveis:

```
/do-create-tasks <techspec>
```

A skill apresenta uma lista de tasks para você aprovar antes de detalhar. Ela **não cria task de E2E ou de aceite**: cada task valida só o próprio entregável, e o E2E completo é feito pelo QA (Passo 8). Ao final, gera:

```
./prds/prd-autenticacao-usuarios/tasks/tasks.md
./prds/prd-autenticacao-usuarios/tasks/1_task.md
./prds/prd-autenticacao-usuarios/tasks/2_task.md
...
```

---

## Passo 6 — Executar as Tasks

### Opção A — Uma por vez (controle manual)

Execute cada task em ordem:

```
/do-execute-task 1_task.md
/do-execute-task 2_task.md
/do-execute-task 3_task.md
... (até todas as tasks estarem [x] em tasks.md)
```

### Opção B — Em lote (agent autônomo)

Use `/do-execute-all-tasks` (a skill orquestra no agente primário da sessão e delega cada task ao subagente `agent-execute-task`) para executar um conjunto de tasks sequencialmente sem intervenção:

```
/do-execute-all-tasks prds/prd-autenticacao-usuarios/tasks/tasks.md all
```

Você também pode executar um range ou uma lista específica:

```
/do-execute-all-tasks prds/prd-autenticacao-usuarios/tasks/tasks.md 1.0-4.0
/do-execute-all-tasks prds/prd-autenticacao-usuarios/tasks/tasks.md 1.0,3.0,5.0
```

> Executa **uma task por vez, estritamente sequencial** (nunca em paralelo — tasks do mesmo PRD editam os mesmos arquivos e o mesmo `tasks.md`), delegando cada execução a um subagente isolado (`agent-execute-task`) — isso evita estouro da janela de contexto em filas longas. Para automaticamente em caso de falha. Disponível no Claude Code, Cursor, GitHub Copilot e Opencode após o `do-setup`.
>
> ⚠️ Rode o command **na sessão primária**, não dentro de um subagente. Nenhuma ferramenta suportada garante subagente-de-subagente: o Claude Code proíbe, o Copilot desabilita por default, e no opencode a fila trava esperando prompts de permissão que nunca aparecem na TUI.

A skill (e o agent) implementam o código, rodam os testes e geram um review file por task. Para acompanhar o progresso a qualquer momento:

```
/do-status
```

### Se a fila parar sem erro

Se a execução ficar parada sem mensagem de erro, verifique se há um **pedido de permissão pendente na conversa do subagente** — por exemplo, acesso a um diretório fora do projeto. O pedido bloqueia a task e o orquestrador fica esperando indefinidamente. Responda ao pedido na conversa do subagente; se a task tiver sido interrompida, rode de novo o `/do-execute-all-tasks` a partir da task pendente (ex.: `13.0-15.0`).

---

## Passo 7 — Code Review (loop até APROVADO)

Com todas as tasks concluídas, rode o review geral. Ele é um review **de código**: confronta o código com PRD, TechSpec, tasks e as regras do projeto e roda a suíte de testes, sem subir serviços nem navegador. O que só se descobre rodando a funcionalidade fica como recomendação para o QA:

```
/do-execute-review <techspec>
```

Se o resultado for `REPROVADO` ou `APROVADO COM RESSALVAS` com itens pendentes, corrija os findings e repita:

```
/do-execute-review-fix prds/prd-<slug>/review/review-1/fixes/fix-R-01-....md
/do-execute-review-fix prds/prd-<slug>/review/review-1/fixes/ --min=maior
/do-execute-review <tasks>
```

Com uma pasta ou vários arquivos, cada fix roda em um subagente isolado, um por vez, dos críticos para os menores. Se um fix falhar, os outros continuam; a fila só para se a suíte ficar vermelha. Um fix só altera código e testes: se a correção exigir mudar o PRD/TechSpec ou uma decisão sua, o worker não edita nada, devolve `NÃO RESOLVIDO` e a decisão aparece no resumo da fila. Você escolhe o lote (pasta inteira, filtro `--min` ou uma lista de arquivos): achados podem ser falsos positivos ou de baixa prioridade, e o framework não decide por você.

Repita até o status ser `APROVADO` (ou `APROVADO COM RESSALVAS` sem itens que você queira corrigir).

> **Cada rodada tem a sua pasta.** Cada execução do `do-execute-review` grava em `prds/prd-<slug>/review/review-1/`, `review/review-2/`, … (relatório em `review/review-N/review-report.md` e fix tasks em `review/review-N/fixes/`), então rodar o review de novo nunca sobrescreve o anterior. O `do-execute-review-fix` descobre a rodada pelo caminho do fix task que você passa, e o relatório novo lista os itens que ainda estavam pendentes na rodada anterior. PRDs antigos, com `review-report.md` solto na raiz, continuam funcionando.

---

## Passo 8 — QA (loop até zero bugs HIGH)

Com o review aprovado, rode a validação final. O QA é o único passo que faz o E2E completo, a acessibilidade e a análise visual; as tasks fazem só um teste de fumaça e o review olha só o código:

```
/do-execute-qa <prd>
```

Se bugs forem encontrados, corrija e revalide:

```
/do-execute-qa-fix prds/prd-<slug>/qa/qa-1/bugs/bug-01-....md
/do-execute-qa-fix prds/prd-<slug>/qa/qa-1/bugs/ --min=media
/do-execute-qa <prd>
```

O modo fila funciona como no review: pasta ou vários arquivos, um subagente por bug, falha isolada.

Repita até não restar nenhum bug de severidade HIGH. Feature pronta!

> **Evidência no relatório:** cada requisito sai como `PASSOU` (visto rodando), `FALHOU`, `VERIFICADO NO CÓDIGO` ou `NÃO TESTADO`. Os dois últimos não contam como aprovados: se sobrar algum, o status é `APROVADO COM RESSALVAS`, no máximo.

> **Mesma lógica no QA:** cada `do-execute-qa` cria `qa/qa-1/`, `qa/qa-2/`, … com `qa-report.md`, `bugs/` e `screenshots/` da rodada. O `do-execute-qa-fix` usa o caminho do bug (`qa/qa-N/bugs/bug-....md`) para saber em que rodada trabalhar, e o relatório da rodada nova traz os bugs ainda abertos da anterior.

---

## Modelos com janela de contexto pequena

O framework não assume um tamanho de contexto. Com modelos de janela grande, as tasks no tamanho padrão cabem com folga. Com modelos de janela pequena (por exemplo, modelos locais de ~128k tokens), cabe a você pedir tasks menores na hora de criá-las.

Medições de referência — modelo local de 128k, 15 tasks, todas no mesmo arquivo de ~3.800 linhas:

- O agente compactou o contexto ao chegar a ~100k tokens. Descontando ~20k do prompt-base e ~20k das leituras obrigatórias (PRD, TechSpec, especificação), sobraram **~60k tokens de trabalho por task**.
- Tasks comuns gastaram de 10k a 70k tokens de raciocínio e compactaram no máximo uma vez.
- Tasks com dados exatos escritos à mão, muitas restrições simultâneas ou algoritmos a conferir gastaram de 80k a 130k só de raciocínio e compactaram de 2 a 4 vezes.
- Depois de uma compactação, o modelo tende a esquecer regras da skill. Por isso a `do-execute-task` manda reler o `SKILL.md` ao retomar.

Para pedir tasks menores, acrescente a instrução ao comando:

```
/do-create-tasks <techspec>
Divida as tasks para que cada uma caiba em ~60k tokens de trabalho (o modelo executor tem 128k de contexto). Uma única entrega por task. Trate como pesado: dados exatos escritos à mão (grades, mapas, tabelas), artefatos com muitas restrições simultâneas e algoritmos que precisam ser conferidos caso a caso. Quando houver dados exatos, crie antes a task do validador e use-o para conferir cada um.
```

Ajuste o número ao seu modelo: o orçamento é o ponto em que a ferramenta compacta, menos o prompt-base e as leituras obrigatórias.

---

## Resumo do Fluxo

```
instalar skills
      │
      ▼
  do-setup  (1x por projeto)
      │
      ▼
  do-create-prd
      │
      ▼
  do-create-techspec
      │
      ▼
  do-create-tasks
      │
      ▼
  do-execute-task 1..N  (uma por vez)
      │
      ▼
  do-execute-review  ──▶  do-execute-review-fix  ──▶  (repetir até APROVADO)
      │
      ▼
  do-execute-qa  ──▶  do-execute-qa-fix  ──▶  (repetir até zero HIGH)
      │
      ▼
   DONE ✓
```

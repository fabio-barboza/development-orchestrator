# Pastas de Rodada (Review e QA)

Procedimento padrão referenciado por `do-execute-review`, `do-execute-review-fix`, `do-execute-qa` e `do-execute-qa-bugfix`. O desenvolvedor pode rodar review e QA várias vezes sobre o mesmo PRD; cada execução grava numa **pasta própria e sequencial**, para que uma rodada nunca sobrescreva a anterior.

## Layout

```
./prds/prd-[feature-slug]/
  review-1/
    review-report.md
    fixes/fix-[R-XX]-[severidade-completa]-[slug].md
  review-2/
    ...
  qa-1/
    qa-report.md
    bugs/bug-[XX]-[severidade-completa]-[slug].md
    screenshots/
  qa-2/
    ...
```

Nos skills, `<review-dir>` significa `prds/prd-[feature-slug]/review-[N]` e `<qa-dir>` significa `prds/prd-[feature-slug]/qa-[N]`.

## Criar uma rodada nova (`do-execute-review`, `do-execute-qa`)

Faça isto **uma única vez, no início da execução**, e use o mesmo `[N]` até o fim:

1. Liste o conteúdo de `./prds/prd-[feature-slug]/` e considere só as pastas cujo nome casa exatamente com `review-<número>` (ou `qa-<número>`).
2. `[N]` = maior número encontrado + 1, ou `1` se não houver nenhuma. Nunca reutilize um número, mesmo que uma pasta do meio tenha sido apagada, e nunca grave dentro de uma rodada existente.
3. Não crie a pasta ainda: ela nasce junto com o primeiro arquivo gravado (`mkdir -p`), para que uma execução que termina sem produzir nada (por exemplo, "nada para revisar") não deixe uma rodada vazia. `fixes/`, `bugs/` e `screenshots/` também nascem com o primeiro arquivo.
4. Os IDs (`R-01`, `bug-01`) recomeçam em cada rodada: o caminho já identifica a rodada.

## Quando NÃO criar a pasta

A pasta de uma rodada só existe se a execução produziu pelo menos um artefato. Se não há nada para revisar (nenhuma alteração) ou nada para validar (nenhuma task concluída, nenhum código implementado), a skill informa isso **apenas na resposta**: não cria arquivo nem pasta `review-[N]`/`qa-[N]`, e o próximo `[N]` não é consumido.

## Layout legado

PRDs antigos têm `review-report.md`, `review-fixes/`, `qa-report.md`, `qa-bugs/` e `qa-screenshots/` soltos na raiz do PRD. Esses itens formam uma **rodada legada**:

- Nunca os sobrescreva, mova ou apague.
- Não entram na contagem de `[N]` (a primeira rodada no layout novo é `review-1`/`qa-1`).
- Valem como "rodada anterior" para a seção de pendências (abaixo) quando não existir nenhuma rodada numerada anterior.

## Descobrir a rodada a partir de um arquivo de entrada (`do-execute-review-fix`, `do-execute-qa-bugfix`)

O arquivo recebido (fix task ou bug) define a rodada pelo seu caminho:

| Arquivo recebido | `<review-dir>` / `<qa-dir>` | Relatório / screenshots |
|------------------|-----------------------------|-------------------------|
| `.../review-[N]/fixes/fix-...md` | `.../review-[N]` | `<review-dir>/review-report.md` |
| `.../qa-[N]/bugs/bug-...md` | `.../qa-[N]` | `<qa-dir>/qa-report.md`, `<qa-dir>/screenshots/` |
| `.../review-fixes/fix-...md` (legado) | a raiz do PRD | `review-report.md` na raiz do PRD |
| `.../qa-bugs/bug-...md` (legado) | a raiz do PRD | `qa-report.md` e `qa-screenshots/` na raiz do PRD |

Todo arquivo criado ou atualizado durante a correção (relatório, evidências, novos bugs) fica **na mesma rodada do arquivo recebido**.

Se nenhum caminho for informado: liste os itens pendentes (`pendente` nos fixes, `aberto`/`novo` nos bugs) da rodada numerada mais recente — e, se existir, da rodada legada, rotulada como tal — e pergunte (ou escolha, conforme a regra da skill) qual corrigir.

## Pendências da rodada anterior (`do-execute-review`, `do-execute-qa`)

Antes de gerar o relatório da rodada nova:

1. Identifique a rodada anterior: a de maior número menor que `[N]`; se não houver, a rodada legada; se também não houver, não há pendências a reportar.
2. Leia o relatório anterior e os arquivos da rodada anterior (`fixes/` ou `bugs/`) e reúna os itens ainda abertos (`pendente` nos fixes; `aberto` ou `novo` nos bugs).
3. Verifique, na rodada atual, se cada item persiste (o problema continua no código/comportamento) ou não foi reproduzido.
4. Registre-os na seção **Pendências da Rodada Anterior** do relatório: ID, severidade, resumo, rodada de origem e situação (`persiste` / `não reproduzido`).
5. **Não** crie arquivos novos de fix ou de bug para esses itens: os arquivos da rodada anterior continuam válidos e podem ser corrigidos por `do-execute-review-fix`/`do-execute-qa-bugfix`. Só problemas **novos** geram arquivos na rodada atual.
6. Se não houver rodada anterior ou itens abertos, escreva "Nenhuma pendência da rodada anterior." na seção.

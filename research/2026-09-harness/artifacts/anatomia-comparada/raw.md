# Anatomia Comparada

**As 29 moléculas do gobbi-os contra os 32 subsistemas do LifeOS — e o que cada lado resolveu que o outro não resolveu.**

O documento anterior leu o LifeOS de fora. Este entra nos dois corpos. Ele responde três perguntas que ficaram abertas: o que exatamente é uma molécula nossa e para que serve cada uma; o que é o Hill Climb dele e por que ele deletou as fases para chegar nele; e o que são Atlas, Arbol, Synapse, Conduit, Feed, Bunker, Ledger, Router e Spinner — os nove nomes que o documento anterior descartou sem explicar.

**Todos os números foram medidos em 2026-09-14.** Nenhum veio de memória, nenhum foi herdado do documento anterior.

---

## A descoberta que reordena tudo

**O Router dele era o nosso classificador. Ele o aposentou em 11 de julho de 2026.**

Está escrito no topo de `DOCUMENTATION/Router/RouterSystem.md`, em caixa alta:

> **RETIRED 2026-07-11 (hooks + thinking-system BPE pass).** The Router hook, the mode/tier classifier, and the E1–E5 effort tiers no longer exist.

E a descrição do que o Router fazia, antes de morrer, é a descrição literal do nosso `PromptProcessing.hook.ts`:

> The Router is the layer that decides how every prompt gets handled — the mode, the effort tier, the stated goal, the model rung, and which agent or vendor runs the work. It runs first (classification happens the moment a prompt is submitted) and keeps deciding.

Isso muda a leitura do documento anterior. Lá, a pergunta "os tiers pararam mesmo?" parecia uma curiosidade sobre o estilo dele. Não é. **Ele construiu a mesma máquina que nós, rodou com ela, e a desmontou** — deixando o documento no repositório como histórico, com a frase "nothing below is operative".

Não é sinal de que precisamos matar o nosso. É sinal de que a pergunta certa já foi feita e respondida por alguém que pagou o custo. A resposta dele: **o classificador decidia coisas que o modelo decide melhor sozinho, e as duas coisas que ele realmente governava — qual modelo, qual profundidade — foram movidas para um arquivo de configuração versionado.**

---

## 01 · O que é uma molécula

Analogia direta: **uma molécula é um bloco de LEGO com instrução de montagem impressa nele.**

Ela não é um documento que alguém lê. É um pedaço de doutrina com um cabeçalho que diz à máquina onde ele entra, em que ordem, e com que profundidade. O `BuildHarness` (`harness/build/harness.ts`) lê os 29 blocos, ordena, renderiza e escreve os quatro artefatos que os agentes de fato leem.

```
  harness/molecules/rule/*.md   (29 blocos, 61.461 bytes)
                │
                │  frontmatter de cada bloco:
                │    slot · order · render · trigger · surfaces · gate · parity
                ▼
  harness/manifests/global.yaml (a lista de seleção — quem entra)
                │
                ▼
        BuildHarness (9 passos)
     resolve → ordena → renderiza → injeta
                │
        ┌───────┼────────┬──────────┐
        ▼       ▼        ▼          ▼
    CLAUDE.md  AGENTS.md  .agents/  NSM
     (Claude)   (Codex)   (AGY)   (Maestro)
```

O campo que carrega o peso é o `render`. Ele tem três valores e é a arquitetura inteira:

| `render` | O que acontece | Custo |
|---|---|---|
| **full** | O texto do bloco é colado por inteiro dentro do CLAUDE.md | Entra em toda sessão, sempre |
| **pointer** | Só uma LINHA na tabela de gatilhos, apontando o caminho absoluto do rule-book | ~200 bytes fixos; o corpo só é lido quando o gatilho dispara |
| **summary** | Um resumo curto no lugar do texto | Intermediário |

**São 10 blocos `full` e 19 blocos `pointer`.** Os 10 viram a espinha sempre-carregada; os 19 viram a tabela do §5 do CLAUDE.md, que aponta para 398 KB de rule-books que ficam no disco até alguém precisar.

É por isso que o nosso caminho quente é menor que o dele apesar da nossa doutrina total ser comparável. **Não é economia de texto. É progressive disclosure com frontmatter.**

### As 10 moléculas que entram sempre (a espinha)

| # | Molécula | Bytes | Para que serve |
|---|---|---|---|
| 1 | `os-contract` | 5.671 | O banner `gobbi-os` e o rodapé SITREP — o contrato de saída de toda resposta |
| 5 | `runtime-awareness` | 2.931 | Descobrir ONDE você está rodando (Mac, container, Maestro) antes de escrever qualquer arquivo |
| 10 | `identity-core` | 2.097 | Quem eu sou, quem é o Felipe, tom, PT-BR, sem enrolação |
| 20 | `prime-directive` | 3.354 | "Proibir erro, não mitigar" + os gates G1–G4 |
| 30 | `work-algorithm` | 10.222 | O funil de 7 passos (FRAME→CARD→QUALIFY→BRANCH→EXECUTE→REVIEW→CLOSE) + os gates por risco |
| 40 | `guardrails` | 2.660 | As listas NEVER / ALWAYS + a ordem de ensino WHAT→HOW→WHY |
| 42 | `topology-preflight` | 9.514 | O porteiro: branch cadáver, lastro, as três cadeiras, o SITREP |
| 49 | `module-library-intro` | 854 | A frase que explica como usar a tabela de gatilhos |
| 60 | `reading-rules` | 673 | "Puxe um módulo só quando o gatilho disparar; aí leia inteiro" |
| 90 | `references` | 784 | Os caminhos canônicos (CONTEXT.md, os dois rule-books carro-chefe) |

**Total da espinha: 38.760 bytes de 44.598.** O resto são as 19 linhas de ponteiro + o texto de emenda do §3.5.

### As 19 moléculas sob demanda (os ponteiros)

| Ordem | Molécula | Dispara quando | Rule-book |
|---|---|---|---|
| 50 | `conductor-profile` | Sessão começa, ou precisa agir sobre identidade Maestro | 24.904 B |
| 51 | `pai-memory` | Boot da sessão e na fase LEARN | 19.701 B |
| 52 | `pai-algorithm` | Classificador devolve ALGORITHM, ou você força um tier | 27.566 B |
| 53 | `pai-isa` | Precisa do formato de contrato de "pronto" | 22.259 B |
| 54 | `decision-log` | Vai agir numa tarefa ALGORITHM, ou uma decisão foi tomada | 3.690 B |
| 55 | `discovery-first` | Vai editar ou criar qualquer coisa | 12.156 B |
| 56 | `clickup-flow` | Toca ClickUp ou Auto Run | 35.070 B |
| 57 | `topology` | Cria lista, branch, worktree, ou faz bootstrap | 46.858 B |
| 58 | `github-flow` | O trabalho é gitable (branch→commit→PR→merge) | 10.068 B |
| 59 | `review` | Card chegou em `in review` | 29.261 B |
| 60 | `google-broker` | Qualquer acesso a Drive/Docs/Sheets/Calendar | 19.426 B |
| 61 | `shell-zsh-guard` | Comando de shell manipula um id | 15.155 B |
| 62 | `model-routing` | Vai spawnar subagente ou workflow | 17.113 B |
| 63 | `rtk` | Roda comandos dev-ops no shell | 15.580 B |
| 64 | `mcp-usage` | Usa qualquer ferramenta MCP | 17.470 B |
| 65 | `pms-crm` | Trabalho operacional de ClickUp (spaces, folders, perfis) | 25.511 B |
| 66 | `mermaid-visual-canon` | Gera, revisa ou audita qualquer diagrama Mermaid | skill |
| 67 | `weekly-goal` | Abre, lê, projeta ou fecha a SEMANA | 36.961 B |
| 68 | `ask-human` | Precisa que o Felipe decida, aprove ou opine | 10.175 B |

**19 rule-books, 398 KB, zero bytes no caminho quente até o gatilho disparar.**

### O que mais tem no frontmatter (e ninguém olha)

Quatro campos além de `render` fazem trabalho pesado e são invisíveis:

| Campo | O que faz | Exemplo vivo |
|---|---|---|
| `surfaces` | Em quais motores o bloco entra | `clickup-flow` vai para claude+agents+**nsm** (o Nudge do Maestro); `github-flow` só para claude+agents |
| `parity` | Se o bloco precisa ser byte-idêntico entre os motores | `true` em todos os 29 — Claude, Codex e AGY leem a MESMA regra |
| `gate` | Se existe um hook que ENFORCE aquele bloco | `enforce` em apenas **2**: `clickup-flow` e `pms-crm` |
| `requires` | Dependência entre blocos | vazio em todos hoje |

**O campo `gate` é a confissão mais útil da nossa arquitetura.** Vinte e nove moléculas, duas com dente. As outras 27 são texto pedindo boa-fé.

---

## 02 · O caminho quente, medido hoje

| | gobbi-os | LifeOS 8.20.2 |
|---|---|---|
| **Doutrina sempre-carregada** | 44.598 B (`CLAUDE.md`) | 61.179 B (3 arquivos) |
| | composto de 10 moléculas `full` | `CLAUDE.template.md` 7.293 |
| | + 19 linhas de ponteiro | `LIFEOS_SYSTEM_PROMPT.md` 25.584 |
| | | `ALGORITHM/v8.20.2.md` 28.302 |
| **Doutrina sob demanda** | 19 rule-books · 398 KB | 32 pastas DOCUMENTATION · 66 arquivos · ~1.260 KB |
| **Fonte de composição** | 29 moléculas · 61 KB · frontmatter + manifest | escrito à mão, sem composição |
| **Motores atendidos** | 4 (Claude · Codex · AGY · Maestro NSM) por build | 1 (Claude Code) |
| **Hooks** | 48 executáveis, **0 documentos** | ~91 KB só de documentação de hooks |
| **Skills** | 147 | 44 KB de doutrina de skills |

### A diferença que não é de tamanho

**Ele escreve o CLAUDE.md. Nós COMPILAMOS o nosso.**

É a diferença entre escrever um anúncio e ter um sistema de templates que gera o mesmo anúncio para Meta, Google e TikTok a partir de uma fonte só. Ele tem um motor, então escrever à mão é honesto. Nós temos quatro motores lendo a mesma regra — se fosse à mão, o Codex já teria divergido do Claude em três lugares.

**Isso é vantagem nossa e ele não tem equivalente.** Vale dizer em voz alta porque é fácil ler o documento anterior e sair achando que estamos atrás em tudo.

### A diferença que É de tamanho, e importa

**O `AlgorithmSystem.md` dele tem 26.706 bytes só explicando o Algorithm. O nosso `pai-algorithm.md` tem 27.566 — número quase idêntico.** Não estamos atrás em doutrina. Estamos atrás em UMA coisa: **quantos desses bytes viraram máquina.**

```
   DOUTRINA ESCRITA              O QUE VIRA MÁQUINA
   ─────────────────             ──────────────────
   nós:  29 moléculas   ────►    2 com gate: enforce
   ele:  32 subsistemas ────►    cada pasta É um componente com
                                 código .ts, plist de launchd, e um
                                 gate /ic que falha o build
```

Ele não escreveu mais que nós. Ele escreveu a mesma quantidade **e plugou cada pedaço num executável.**

---

## 03 · O Hill Climb, explicado

O que o Felipe chamou de "aquele Hill Climb" tem nome técnico: **Ascent States**, em `DOCUMENTATION/Algorithm/AscentStates.md`.

### A metáfora

Todo trabalho é **subir uma colina que você teve que nomear antes de começar a subir.**

- Você **marca o cume** — escreve o que "pronto" significa.
- Você **sobe** — constrói, explora, delega.
- Você **ancora cada apoio** — antes de confiar num apoio, você põe o peso nele. Traduzido: antes de fechar uma claim, você roda a prova.
- Você **deixa um marco de pedra** (cairn) — o registro de que a rota funcionou.

A frase que define o sistema inteiro:

> Everything here is one loop: move a thing from its current state to its ideal state by climbing a hill we define as we climb it. Runs are conjecture and refutation against the ISA.

**"Conjectura e refutação"** é Popper. A ISA não é um plano — é uma hipótese com o teste que a mataria escrito ao lado. Essa é a diferença inteira entre o Hill Climb e um checklist.

### Os seis estados

| Estado | Significado | Como é DERIVADO |
|---|---|---|
| 🥾 **Traverse** | Trabalho vivo sem ISA — andando sem rota declarada | untracked + live |
| 📐 **Marking** | Articulando o que é "pronto" — claims ainda não existem | tracked + live + zero claims |
| 🧗 **Ascending** | Na parede — explorando, construindo, delegando | tool stream construindo, ou `phase: climbing` |
| ⚓ **Anchoring** | Pondo peso no apoio — probes rodando, evidência chegando | tool stream verificando |
| 💤 **Camped** | Parado no meio da subida — retomável, não morto | tracked + ficou quieto |
| 🪨 **Cairn** | Fechado — toda claim segurou | closed ≥ total, ou `phase: complete` |

**A palavra-chave é DERIVADO.** Ninguém escreve o estado. O modelo escreve duas coisas no arquivo ISA: um `phase:` mínimo no começo e `complete` no fim. Tudo no meio é calculado por uma função, `deriveAscent()`, a partir dos dados da run.

```
   ISA phase:          ─┐
   claims fechadas/total┼──► deriveAscent() ──► AscentState
   tracked / active    ─┤                          │
   tool stream vivo    ─┘  (só o Pulse)            │
                                    ┌──────────────┼──────────────┐
                                    ▼              ▼              ▼
                              aba do terminal   dashboard    barra de status
```

### Por que ele matou as fases declaradas

Porque as oito "estações" declaradas produziram **cinco bugs reais**, não cosméticos. Todos com a mesma causa: cada consumidor mantinha a própria lista de nomes de fase à mão, e as listas pararam de bater com a realidade sem que nada quebrasse visivelmente.

| # | O bug |
|---|---|
| 1 | `ISASync` parou de repintar as abas no meio da run — o allowlist engolia valor desconhecido em silêncio |
| 2 | `PromptProcessing` apagava o estado da aba a cada prompt de follow-up — e um comentário no arquivo afirmava ter consertado exatamente isso |
| 3 | `ACTIVE_LOOKUP_PHASES` não casava com NENHUMA run moderna |
| 4 | **As duas linhas de nudge do ISA ficaram mudas por uma semana** — `stale-isa` não disparou uma única vez em **392 sessões registradas**; a irmã `late-isa` nunca disparou na vida |
| 5 | `WorkSweep` lia run moderna como scaffold abandonado e pulava |

E a frase que fecha, que é a lição mais transferível do documento inteiro:

> **A nudge that doesn't fire breaks nothing visibly — nothing fails, so nothing tells you.**

Um aviso que não dispara não quebra nada visivelmente. Nada falha, então nada te conta.

**E o conserto não foi o design.** Está escrito, com honestidade desconfortável: derivar da tabela removeu o bug onde a derivação é usada, e não fez nada por quem mantém a própria lista — "which is why this class has now recurred five times. **Enforcement, not design, is what closed it**". O que fechou foi um gate (`/ic ascent-vocabulary`) que FALHA o build se qualquer arquivo enumerar três ou mais chaves de fase à mão.

### Isso se aplica a nós?

**Parcialmente, e a distinção é toda a resposta.**

| | As fases dele | As nossas fases |
|---|---|---|
| O que eram | Cerimônia: o modelo anunciava estação e a estação não mudava nada | Funil: cada passo MOVE card, abre branch, dispara gate |
| Quem lia | Quatro superfícies de UI, cada uma com vocabulário próprio | ClickUp, o Regente, o porteiro, o SITREP |
| Custo de errar | Aba pintada errada | Card não despachado, branch órfã, merge sem review |

**Fase que aciona máquina fica. Fase que só anuncia, morre.** As nossas acionam.

**Mas o nosso SITREP tem exatamente a doença dele.** O rodapé que aparece a cada turno é montado por um script (`sitrep-build.sh`) e o dot 🟢/🟡/🔴 é calculado em outro lugar (`check.sh`), e as duas coisas já discordaram em produção — foi o caso do `TP_SKIP_LASTRO=1`, em que o painel lia 🟢 enquanto 15 commits iam para uma branch sem card. **É a mesma classe de bug: duas superfícies com a própria lista.** O conserto dele — uma derivação, um gate que falha o build se alguém escrever a lista à mão — é copiável direto.

---

## 04 · Os nove subsistemas, sem mistério

O documento anterior listou "Atlas · Arbol · Synapse · Conduit · Feed · Bunker · Ledger · Router · Spinner" e disse "subsistemas que não existem no nosso mundo". É verdade, mas não explica nada. Aqui está cada um, com o que já temos equivalente.

### A tese que organiza tudo

Antes dos nomes: o LifeOS inteiro tem **uma tese**, e ela explica por que esses subsistemas existem.

> Antes de você poder subir em direção a um estado ideal, você precisa de um retrato preciso do estado em que está.

Metade do sistema dele existe para conhecer o ESTADO ATUAL. A outra metade para subir até o IDEAL. Os nove nomes se distribuem nessas duas metades.

```
      ESTADO ATUAL                        ESTADO IDEAL
  ┌──────────────────┐              ┌──────────────────┐
  │ Conduit (sentido │              │  ISA  (as claims)│
  │  interno: você)  │              │  Algorithm (loop)│
  │ Feed (sentido    │   Synapse    │  Bunker (o chassi│
  │  externo: mundo) │ ──(roteador) │   de app)        │
  │ Atlas (inventário│      │       │  Arbol (execução │
  │  de ativos)      │      ▼       │   na nuvem)      │
  └──────────────────┘   Cortex     └──────────────────┘
                        (memória)
        Ledger = o que mudou, quando, versionado
        Spinner = cosmético (verbo animado na barra)
        Router = MORTO (era o classificador de tier)
```

### Um por um

| Nome | O que é, em uma frase | Nosso equivalente |
|---|---|---|
| **Atlas** (12 KB doc) | **Inventário grafo de tudo que ele possui** — workers Cloudflare, domínios, registros DNS, repos, serviços launchd, máquinas, equipamento. Um SQLite local reconciliado de fontes de autoridade, com carimbo de primeiro/último visto em cada fato. Modelado no Cartography da CNCF. | **Nada.** Temos `maestro-cli list agents` e `cup` cada um sabendo do próprio pedaço. Ninguém sabe o todo. |
| **Arbol** (30 KB doc) | **O LifeOS rodando enquanto ele dorme.** Cloudflare Workers agendados que observam fontes, transformam sinais e atualizam o estado a partir da borda, sem sessão aberta. *"Árbol" = árvore: um tronco de primitivas ramificando em workers agendados.* | **Maestro Cue.** Nosso Cue é exatamente isso, só que rodando na máquina dele em vez da nuvem. |
| **Synapse** (15 KB doc) | **O roteador de entrada.** Tudo que cruza sua atenção — URL, bookmark, pensamento falado, PDF — entra por um contrato só, é preservado na hora num diário append-only (o "âmbar"), resumido, **avaliado contra o que você está tentando fazer**, e roteado para o destino que merece. | **Nossa esteira X→Obsidian**, mas só para um tipo de entrada e sem a nota de relevância. |
| **Conduit** (13 KB doc) | **O sentido interno.** Captura local e contínua de para onde sua atenção realmente vai, consolidada num registro diário. Existe para responder "estamos trabalhando na coisa certa?". **Um espelho que você puxa, nunca um vigia que empurra** — ele se recusa a produzir um "score de alinhamento", porque score convida a gamear. | **Nada.** E o princípio "espelho, não score" é a melhor regra de produto do documento inteiro. |
| **Feed** (24 KB doc) | **O sentido externo.** Fontes que ele escolhe, observadas continuamente, avaliadas contra os objetivos dele: sinal relevante alerta na hora, ruído arquiva em silêncio. | **Nada.** É o que o Outlier Radar faz para UM nicho. |
| **Bunker** (6 KB doc) | **O chassi universal de aplicação.** O app é dono da experiência; o Bunker é dono da camada invisível — auth, dados, deploy, métricas, saúde, segurança. *"Padronize os canos, nunca a pintura."* Um app declara um TIPO e o tipo liga os componentes certos. | **O Bootstrapper**, na camada de projeto em vez de app. Mesma ideia: declarar o tipo e o sistema monta o esqueleto. |
| **Ledger** (14 KB doc) | **O que mudou, quando, em qual versão, verificado como — para tudo.** Classifica toda mudança em Major/Feature/Patch, sobe a versão do guarda-chuva e de cada componente tocado, registra num registro append-only, e **trava o envio se a integridade falhar.** | **git + `cup`**, parcialmente. Não temos a parte de versionar componentes nem o travamento por integridade. |
| **Router** | **MORTO.** Era o classificador de modo + tier E1–E5 + escolha de modelo. Aposentado 2026-07-11. | **Vivo e rodando aqui** (`PromptProcessing.hook.ts`). |
| **Spinner** (4 KB doc) | **O verbo animado na barra de status** enquanto o sistema trabalha ("Composing", "Forging", "Climbing") com ícone, cor e animação próprios, mais dicas rotativas sobre o estado real do sistema. Puramente cosmético — e ele escreveu 4 KB de doutrina para isso. | O nosso SITREP faz o trabalho informativo. |

### O que dá para profissionalizar

A pergunta do Felipe foi: dá para profissionalizar essa parte? **Três dos nove valem, e por motivos diferentes.**

**Atlas é o que mais vale e é o mais barato.** Hoje, se alguém perguntar "quantos worktrees existem, quais têm card, quais estão órfãos, quais agentes Maestro estão sentados onde, quais branches estão mortas" — a resposta exige rodar seis comandos e cruzar na cabeça. **Temos todas as fontes de autoridade** (`git worktree list`, `maestro-cli list agents`, `cup`, `gh`, `launchctl`). Falta o SQLite que reconcilia e o carimbo de primeiro/último visto. E o `worktree-reaper.sh` já é um coletor Atlas sem saber que é.

**Bunker vale como ELEVAÇÃO do Bootstrapper, não como coisa nova.** Hoje o Bootstrapper monta a topologia — grupo, cadeiras, worktrees, cards, ondas. Ele não monta a camada invisível de um app (auth, deploy, health, backup). A ideia dele — o app declara um TIPO e o tipo liga componentes em seis planos — é um upgrade natural do `bootstrapper.config.yaml`, que já tem o conceito de perfil.

**Conduit vale pelo princípio, não pela implementação.** "Espelho que você puxa, nunca vigia que empurra; distribuição e registro, você julga" é a regra que impede um dashboard de virar vaidade métrica. Vale colar isso no nosso SITREP e no relatório de semana como doutrina escrita.

**Os outros seis não.** Arbol é Cue com outro nome. Feed e Synapse são infraestrutura de vida pessoal dele. Ledger é git mais disciplina de release que não temos motivo para ter ainda. Router está morto. Spinner é enfeite.

---

## 05 · A tabela mestre

| Dimensão | gobbi-os | LifeOS 8.20.2 | Quem está à frente |
|---|---|---|---|
| **Composição da doutrina** | 29 moléculas com frontmatter → BuildHarness → 4 superfícies | escrito à mão, 1 superfície | **nós** |
| **Paridade entre motores** | `parity: true` em 29/29; Claude=Codex=AGY | não se aplica | **nós** |
| **Caminho quente** | 44.598 B | 61.179 B | **nós** |
| **Doutrina sob demanda** | 398 KB, 19 rule-books | 1.260 KB, 32 pastas | empate (naturezas diferentes) |
| **Regras com dente** | 2 de 29 (`gate: enforce`) | cada pasta tem código + gate `/ic` | **ele, com folga** |
| **Definição de "pronto"** | status do card + gates de review | ISA com claim + probe que a falsifica | **ele** |
| **Evidência para fechar** | exortação ("NEVER claim work is done") | 11 modalidades + `VerificationGate` (hook) | **ele** |
| **Estado da run visível** | SITREP montado por script | `deriveAscent()` — uma derivação, N superfícies | **ele** |
| **Esforço / modelo** | classificador Haiku + E1–E5 no banner | classificador MORTO; rung por papel em config versionada | **ele** (ele já matou o que temos) |
| **Governança de trabalho** | ClickUp: card, status, dependência nativa, Semana, BU | nenhuma — é um homem só | **nós, sem competição** |
| **Paralelismo** | worktree por frente, 3 cadeiras, porteiro, reaper | `IsaFrontier` com lock atômico por claim | empate (eixos diferentes) |
| **Bootstrap** | Bootstrapper v2: 8 fases, ondas, delegação, teardown | nenhum equivalente | **nós, sem competição** |
| **Inventário de ativos** | nenhum | Atlas (SQLite grafo) | **ele** |
| **Memória** | MEMORY.md + ~40 arquivos, frontmatter livre, zero validador | KnowledgeSchema + Lint + Graph + Cortex | **ele** |
| **Fronteira system/user** | 62 entradas não-versionadas | 4 zonas + hook em tempo de escrita | **ele** |
| **Auto-auditoria** | nenhuma | Doctor · IntegrityCheck · Invariant · SkillDriftLint | **ele** |
| **Documentação de hooks** | 48 hooks, 0 documentos | 91 KB | **ele** |
| **Doutrina de teste** | nenhuma | 24 KB | **ele** |
| **Multi-pessoa / multi-máquina** | é a premissa | não existe | **nós, sem competição** |

### A leitura honesta da tabela

**Ganhamos onde o problema é governar trabalho de várias pessoas.** Bootstrapper, ClickUp, worktrees, paridade entre motores, composição de doutrina — ele não tem nada disso porque não precisa: é um homem numa máquina.

**Ele ganha onde o problema é o sistema se policiar sozinho.** Gate, evidência, schema, auto-auditoria, derivação única. E ganha por um motivo estrutural, não por ser mais inteligente: **ele transformou doutrina em código, e nós transformamos doutrina em texto.**

Uma frase resume: **ele escreveu o manual da fábrica; nós escrevemos o manual do turno.** Ambos são necessários. Só que o manual da fábrica pode ser compilado e o do turno depende de alguém ler.

---

## 06 · O que fazer com isso

Quatro movimentos, em ordem de razão de valor por custo.

### Movimento 1 — ligar o que já está armado (horas)

Nada a construir:

- **`checkpoint-repos.txt`** — um arquivo de uma linha. Sem ele, `CheckpointPerISC.hook.ts` (13,8 KB, no disco, verificado hoje) é no-op. Com ele, cada claim fechada vira commit.
- **`lastro-guard` de `warn` para `block`** — o default hoje é `warn` (linha 93 do hook). É uma variável de ambiente.
- **ISA no Ready Gate** — vira o 12º item.

**Por que primeiro:** é a maior razão valor/custo da lista e não constrói nada novo.

### Movimento 2 — o gate de derivação única (dias)

**Não o Ascent dele. O princípio.** Um gate que falha se qualquer arquivo do harness enumerar à mão uma lista de estados que já existe numa tabela. Hoje o `sitrep-build.sh` e o `check.sh` têm listas paralelas e já discordaram em produção.

**Por que:** é o bug que se repetiu cinco vezes na casa dele antes de alguém conseguir fechar, e o conserto que funcionou foi o gate, não o redesenho. Já pagamos esse bug uma vez.

### Movimento 3 — a fronteira system/user (alto, sobe sozinho)

62 entradas não-versionadas com mapa de credencial dentro. O desenho dele — quatro zonas mais hook em tempo de escrita — é MIT.

**Por que:** é a única coisa da lista que **piora sozinha se ninguém fizer nada.**

### Movimento 4 — Atlas (alto, valor composto)

O inventário grafo. Todas as fontes de autoridade já existem; falta o reconciliador.

**Por que:** é o que responde "o que existe agora?" sem ninguém montar a resposta na cabeça — e é pré-requisito para qualquer coisa autônoma que precise saber o estado do mundo antes de agir.

### A decisão que continua sendo só do Felipe

**E1–E5.** Com a descoberta do Router morto, a pergunta ficou mais afiada: **ele construiu isso, rodou, e desmontou.** Continuam três saídas — virar portão real, virar etiqueta honesta de leitura, ou morrer — mas agora sabemos que alguém já testou a primeira e desistiu dela.

---

*gobbi-os · anatomia comparada · medido em 2026-09-14*

*fontes: `harness/molecules/rule/` (29 arquivos, 61.461 B) · `harness/manifests/global.yaml` · `harness/build/harness.ts` · `harness/_design/rulebooks/` (19 arquivos, 407.298 B) · `CLAUDE.md` (44.598 B) · árvore pública `danielmiessler/LifeOS` (2.658 entradas) · `ALGORITHM/v8.20.2.md` · `DOCUMENTATION/Algorithm/AscentStates.md` · `DOCUMENTATION/{Atlas,Arbol,Synapse,Conduit,Feed,Bunker,Ledger,Router,Spinner}/`*

*documento irmão: **Duas Versões à Frente** — a leitura de fora, mesma pasta*

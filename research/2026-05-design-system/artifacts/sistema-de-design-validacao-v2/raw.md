# Sistema de Design — Validação v2

A nova arquitetura de design publica conhecimento como **wiki navegável** ao invés de arquivos soltos. Esta página é o primeiro teste da v2 da skill.

## Por que mudou da v1

A v1 era boa pra publicar páginas isoladas, mas conhecimento isolado vira link perdido em pasta. A v2 conecta tudo num wiki interno que se atualiza sozinho a cada publicação.

::: comparator
### v1 (anterior)
- Páginas isoladas
- Sem navegação cruzada
- JetBrains Mono em tudo
- Sem busca
- Sem tags

### v2 (atual)
- Wiki navegável
- Sidebar com siblings do tema
- Inter (body) + Newsreader (títulos) + JetBrains Mono (code)
- Busca client-side
- Tags clicáveis
:::

## Como usar

Três passos pra publicar seu primeiro artefato:

::: accordion-seq
### Escreva o conteúdo em Markdown
Use frontmatter `tags: x,y,z` se quiser categorização. H1 vira título, primeiro parágrafo vira lead.

### Invoque a skill
`/global:skills:artifacts-publisher "Meu título" --content arquivo.md --tags strategy,growth`

### Receba a URL + senha
A skill gera senha randômica AES-GCM (ou usa `--password X`). Compartilhe em canal interno.
:::

## Arquitetura

::: mermaid-zoom
graph LR
    A[Markdown raw] --> B[Claude: enriquece]
    B --> C{--enrich?}
    C -->|sim| D[Codex: tabelas]
    C -->|sim| E[Gemini: research]
    D --> F[publish.sh]
    E --> F
    C -->|não| F
    F --> G[render-artifact.py]
    G --> H[gate.sh AES-GCM]
    H --> I[render-wiki.py]
    I --> J[git push + Pages]
    J --> K[URL pública]

    style A fill:#262121,stroke:#5b675b,color:#f2f2c0
    style F fill:#0f100f,stroke:#bed78e,color:#bed78e
    style K fill:#262121,stroke:#bed78e,color:#bed78e
:::

## Performance budget

| Métrica | Limite | Atual |
|---------|--------|-------|
| HTML inicial | < 30 KB | TBD |
| Total com gate | < 80 KB | TBD |
| JS inline | < 15 KB | TBD |
| Fontes | 3 (Inter, Newsreader, JBMono) | 3 |
| CDN externos | 0 (exceto mermaid sob demanda) | 1 condicional |

## Multi-model routing

Cada modelo tem seu papel:

- **Claude** — Criativo, escrita com nuance, decisões pedagógicas
- **Codex** — Formal, tabelas, números, specs técnicas
- **Gemini** — Pesquisa web, validação de fatos, refs externas

A flag `--enrich` ativa o pipeline completo. Sem ela, só Claude trabalha.

## Filtro pedagógico

> Antes de adicionar qualquer interatividade, pergunte: **esse componente vai aumentar DRASTICAMENTE a absorção do leitor?**

Se a resposta for "talvez", **remova**. Decoração não ensina.

---

Este artefato é o teste de fumaça da v2. Se você está lendo isso navegando do wiki, com a paleta dark green do Maestro, fontes editoriais, comparator funcionando, accordion expandindo, e diagrama com zoom — está tudo no lugar.

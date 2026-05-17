# POC FULL — Skill `artifacts-publisher`

Demonstração do modo `--full` da skill. Mesmo conteúdo do POC Simple + interatividade pesada.

## Features adicionadas (vs Simple)

- Sidebar TOC com scroll-spy
- Tabs navegáveis (Overview / Architecture / Design / Decisions)
- Mermaid diagrams (flowchart + sequence)
- Tabela de tokens com swatches clicáveis (copia hex)
- Filtros e sort em tabelas
- Code blocks com botão "copy"
- Collapse/expand de seções (clique no h2 ou botões collapse/expand all)
- Command palette ⌘K com fuzzy search
- Animações de entrada (fade-in nos tabs)
- Toast de feedback
- Keyboard navigation
- Stat cards
- Timeline visual

## Stack frontend

- HTML + CSS + JS vanilla (zero build step)
- Mermaid via CDN
- JetBrains Mono via Google Fonts
- Theme tokens hardcoded (snapshot Maestro)
- Tudo self-contained num único `index.html`

## Quando usar FULL vs Simple

- **Simple:** decisões rápidas, briefings, relatórios diários
- **Full:** estudos longos, handoffs, microsites, conteúdo navegável

## Senha
`vibework-poc-2026`

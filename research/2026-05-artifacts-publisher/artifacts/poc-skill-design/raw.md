# POC — Design da Skill `artifacts-publisher`

## Contexto

Felipe compartilhou print de documentação do repo `blessy-knowledge` mostrando padrão de artefatos navegáveis: cada estudo gera uma pasta `artifacts/` com `index.html` publicado no GitHub Pages com password gate. Workflow: gera Markdown (raw, edição/IA) + HTML (consumo, visualização) e publica.

## Objetivo

Adaptar esse padrão para o VibeworkV2 como skill global reutilizável, com dois modos:

- **Simple:** template básico, rápido, tipografia limpa
- **Full:** chama a skill `visual-explainer` para HTML "fodástico"

## Decisões tomadas

| Decisão | Escolha |
|---------|---------|
| Onde mora | `docs/research/YYYY-MM-tema/artifacts/` (padrão Blessy) |
| Password gate | Client-side JS hash (suficiente p/ uso interno) |
| Renderização | Dois modos: `--simple` (default) e `--full` (via visual-explainer) |
| Repo destino | `vibework-knowledge` (dedicado, mesmo modelo Blessy) |

## Arquitetura da skill

```
~/.claude/skills/artifacts-publisher/
├── SKILL.md                        ← orquestração
├── templates/
│   ├── artifact-readme.md.tpl
│   ├── password-gate.html.tpl
│   └── simple-page.html.tpl
└── scripts/
    ├── publish.sh
    ├── gate.sh
    └── slugify.sh
```

## Flags propostas

- sem flag → modo `--simple`
- `--full` → chama visual-explainer
- `--password <senha>` → define senha do gate
- `--no-gate` → publica público
- `--slug <nome>` → override do slug

## Pipeline comum

1. Gera `raw.md`
2. Gera `index.html` (simple ou full)
3. Aplica password gate JS
4. Cria/atualiza `artifacts/README.md`
5. Commit + push no repo
6. Habilita Pages se necessário
7. Retorna URL final

## Status

POC manual em execução para validar o fluxo end-to-end antes de codificar a skill.

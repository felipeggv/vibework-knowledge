# vibework-knowledge

Repositório de artefatos navegáveis do workspace **VibeworkV2**.

## Estrutura

```
research/
└── YYYY-MM-tema/
    └── artifacts/
        ├── README.md              ← origem, data, ferramenta, URLs
        └── <slug>/
            ├── index.html         ← HTML de consumo (com password gate)
            ├── raw.md             ← Markdown raw
            └── assets/            ← imagens, charts, etc.
```

## Convenções

- Cada artefato vive em uma **pasta própria** com `index.html`
- HTML tem **password gate** (client-side SHA-256) aplicado antes do publish
- Senha é compartilhada via **canal interno do time**, nunca em commit/issue/Markdown
- `artifacts/README.md` de cada estudo registra **URL publicada, origem e data**

## Publicação

GitHub Pages serve a `main` deste repo. URL padrão dos artefatos:

```
https://felipeggv.github.io/vibework-knowledge/research/YYYY-MM-tema/artifacts/<slug>/
```

## Histórico

| Data | Artefato | URL |
|------|----------|-----|
| 2026-05-17 | POC — Skill `artifacts-publisher` | [link](https://felipeggv.github.io/vibework-knowledge/research/2026-05-artifacts-publisher/artifacts/poc-skill-design/) |

# my-sites-action

GitHub Action para deploy de sites criados com o template [code7tecbr/my-sites](https://github.com/code7tecbr/my-sites).

## Como funciona

1. Clona o template `my-sites` (privado) usando um PAT
2. Injeta o `content/` e `public/` do repo do cliente
3. Executa `bun install` + `bun nuxt prepare` + `bun run deploy` (via Alchemy)
4. Publica no Cloudflare

O código-fonte do template nunca fica exposto ao cliente.

## Uso

Crie `.github/workflows/deploy.yml` no repo de conteúdo do cliente:

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: code7tecbr/my-sites-action@v1
        with:
          app_name: ${{ secrets.APP_NAME }}
          cf_api_token: ${{ secrets.CF_API_TOKEN }}
          cf_account_id: ${{ secrets.CF_ACCOUNT_ID }}
          template_token: ${{ secrets.TEMPLATE_TOKEN }}
```

## Secrets necessários

| Secret | Descrição |
|---|---|
| `APP_NAME` | Nome único do app no Cloudflare (ex: `acme-site`) |
| `CF_API_TOKEN` | Cloudflare API Token com permissão de Workers |
| `CF_ACCOUNT_ID` | Cloudflare Account ID |
| `TEMPLATE_TOKEN` | GitHub PAT (read-only) para acessar `code7tecbr/my-sites` |

### Como gerar o `TEMPLATE_TOKEN`

1. Acesse **GitHub → Settings → Developer settings → Fine-grained tokens**
2. Crie um token com acesso **read-only** ao repo `code7tecbr/my-sites`
3. Adicione o token como secret no repo do cliente com o nome `TEMPLATE_TOKEN`

## Estrutura esperada do repo do cliente

```
cliente-a/site-content/
├── content/
│   ├── brand.json
│   ├── nav.json
│   ├── seo.json
│   └── sections/
│       ├── hero.json
│       ├── about.json
│       └── contact.json
├── public/
│   └── (imagens e assets)
└── .github/
    └── workflows/
        └── deploy.yml
```

# my-sites-action

GitHub Action para deploy de sites criados com o template [code7tecbr/my-sites](https://github.com/code7tecbr/my-sites).

## Como funciona

1. Clona o template `my-sites` (privado) usando um PAT
2. Injeta o `content/` e `public/` do repo do cliente
3. Executa `bun install` + `bun nuxt prepare` + `bun run deploy` (via Alchemy)
4. Publica no Cloudflare Pages
5. Persiste o state do Alchemy de volta no repo do cliente (commit automático)

O código-fonte do template nunca fica exposto ao cliente.

---

## Setup de um novo cliente — passo a passo

### 1. Criar o repositório do cliente

Crie um repositório no GitHub (pode ser privado). A estrutura mínima é:

```
meu-site/
├── content/
│   ├── brand.json
│   ├── nav.json
│   ├── seo.json
│   ├── layout.json
│   └── sections/
│       ├── hero.json
│       ├── about.json
│       ├── services.json
│       ├── mission.json
│       ├── gallery.json
│       ├── contact.json
│       └── footer.json
├── public/
│   └── (imagens e assets)
└── .github/
    └── workflows/
        └── deploy.yml
```

Copie os JSONs de exemplo do template `code7tecbr/my-sites` (pasta `content/`) como ponto de partida.

### 2. Criar o workflow de deploy

Crie o arquivo `.github/workflows/deploy.yml` no repo do cliente:

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4

      - uses: code7tecbr/my-sites-action@v1
        with:
          app_name: ${{ secrets.APP_NAME }}
          cf_api_token: ${{ secrets.CF_API_TOKEN }}
          cf_account_id: ${{ secrets.CF_ACCOUNT_ID }}
          template_token: ${{ secrets.TEMPLATE_TOKEN }}
          alchemy_password: ${{ secrets.ALCHEMY_PASSWORD }}
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

> **Nota:** `permissions: contents: write` é necessário para o step de persistência do state do Alchemy fazer commit de volta no repo.

### 3. Configurar os secrets no GitHub

No repositório do cliente, vá em **Settings → Secrets and variables → Actions** e adicione:

| Secret | Obrigatório | Descrição |
|---|---|---|
| `APP_NAME` | sim | Nome único do app no Cloudflare (ex: `acme-site`, `banda-fulana`) |
| `CF_API_TOKEN` | sim | Cloudflare API Token com permissão de Workers e Pages |
| `CF_ACCOUNT_ID` | sim | Cloudflare Account ID |
| `TEMPLATE_TOKEN` | sim | GitHub PAT (read-only) para acessar `code7tecbr/my-sites` |
| `ALCHEMY_PASSWORD` | sim | Senha para criptografar o state do Alchemy. Escolha qualquer string forte e guarde-a |

#### Como gerar o `TEMPLATE_TOKEN`

1. Acesse **GitHub → Settings → Developer settings → Fine-grained tokens**
2. Crie um token com acesso **Contents: read** ao repo `code7tecbr/my-sites`
3. Adicione o token como secret no repo do cliente com o nome `TEMPLATE_TOKEN`

#### Como gerar o `CF_API_TOKEN`

1. Acesse **Cloudflare Dashboard → My Profile → API Tokens**
2. Crie um token com permissões: `Workers Scripts: Edit`, `Pages: Edit`, `Account Settings: Read`
3. Adicione como secret `CF_API_TOKEN`

### 4. Fazer o primeiro deploy

Commit e push qualquer alteração no `content/` para a branch `main`. O workflow será acionado automaticamente.

Na primeira execução, o Alchemy cria o Worker e o projeto no Cloudflare. O deploy subsequente é incremental.

### 5. Configurar o assistente de conteúdo (Claude Code)

Para que o cliente possa usar o Claude Code para editar o conteúdo com assistência, copie a pasta `.claude/` deste repositório para o repo do cliente:

```bash
cp -r .claude/ caminho/para/repo-do-cliente/
```

Com isso, o cliente pode abrir o repo no Claude Code e digitar `/content-guide` para receber orientações sobre como editar cada arquivo JSON.

---

## Inputs disponíveis

| Input | Obrigatório | Padrão | Descrição |
|---|---|---|---|
| `app_name` | sim | — | Nome único do app no Cloudflare |
| `cf_api_token` | sim | — | Cloudflare API Token |
| `cf_account_id` | sim | — | Cloudflare Account ID |
| `template_token` | sim | — | PAT para clonar `code7tecbr/my-sites` |
| `alchemy_password` | sim | — | Senha de criptografia do state do Alchemy |
| `stage` | não | `prod` | Stage do Alchemy (afeta o nome do Worker no Cloudflare) |
| `custom_domains` | não | `""` | Domínios customizados separados por vírgula (ex: `site.com.br,www.site.com.br`) |

### Usando domínio customizado

```yaml
- uses: code7tecbr/my-sites-action@v1
  with:
    app_name: ${{ secrets.APP_NAME }}
    cf_api_token: ${{ secrets.CF_API_TOKEN }}
    cf_account_id: ${{ secrets.CF_ACCOUNT_ID }}
    template_token: ${{ secrets.TEMPLATE_TOKEN }}
    alchemy_password: ${{ secrets.ALCHEMY_PASSWORD }}
    custom_domains: "meusite.com.br,www.meusite.com.br"
```

O domínio precisa estar configurado no Cloudflare (DNS apontando para o account).

---

## Versionamento

Use tags semânticas ao publicar atualizações desta action:

```bash
git tag v1.1
git push origin v1.1
```

Repos de clientes devem fixar a versão (`@v1`) para evitar quebras inesperadas ao atualizar a action.

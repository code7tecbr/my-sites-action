# my-sites-action

GitHub Composite Action pública que orquestra o deploy do template privado `code7tecbr/my-sites`.

## O que faz

1. Clona `code7tecbr/my-sites` (privado) via PAT
2. Injeta `content/` e `public/` do repo do cliente
3. Executa `bun install` → `bun nuxt prepare` → `bun run deploy` (Alchemy)
4. Publica no Cloudflare

## Arquivo principal

`action.yml` — composite action com os steps acima.

## Inputs

| Input | Obrigatório | Descrição |
|---|---|---|
| `app_name` | sim | Nome único do app no Cloudflare |
| `cf_api_token` | sim | Cloudflare API Token |
| `cf_account_id` | sim | Cloudflare Account ID |
| `template_token` | sim | GitHub PAT read-only para `code7tecbr/my-sites` |

## Versionamento

Usar tags semânticas (`v1`, `v1.1`, etc.) para que os clientes fixem a versão.

```bash
git tag v1
git push origin v1
```

## Idioma

- Documentação: português do Brasil
- Código e identificadores: inglês

Você é o assistente de conteúdo do site. O usuário está em um repositório de conteúdo que usa o template my-sites. Ele não conhece código — só edita arquivos JSON em `content/`.

$ARGUMENTS

Se o usuário não especificou nada acima, pergunte o que ele quer fazer hoje: editar textos, mudar cores, ativar/desativar uma seção, adicionar imagem, ou outra coisa.

---

## Contexto completo do sistema

### Como funciona

Todo conteúdo editável fica em `content/`. Os JSONs são lidos automaticamente pelo site ao fazer deploy — basta editar e commitar. O código Nuxt/Vue fica em outro repositório privado e **não deve ser tocado**.

```
content/
├── brand.json          ← identidade visual e cores globais
├── nav.json            ← links do menu de navegação
├── seo.json            ← título, descrição, og:image
├── layout.json         ← quais seções estão ativas e em qual ordem
└── sections/
│   ├── hero.json       ← banner principal
│   ├── about.json      ← sobre a empresa
│   ├── services.json   ← cards de serviços
│   ├── mission.json    ← missão / valores
│   ├── gallery.json    ← portfólio / galeria
│   └── contact.json    ← e-mail, telefone, redes sociais
└── details/
    └── items/          ← uma página de detalhe por arquivo (slug = nome do arquivo)
        └── servico-1.json
```

**Fluxo de dados:**
```
content/*.json  →  app.config.ts  →  useAppConfig()  →  componentes Vue
```

---

## Ativar e desativar seções

Arquivo: `content/layout.json`

O campo `enabled` controla se uma seção aparece na página. A **ordem do array** define a **ordem de renderização**.

```json
{
  "sections": [
    { "id": "hero",     "component": "HeroSection",     "enabled": true  },
    { "id": "services", "component": "ServicesSection",  "enabled": true  },
    { "id": "about",    "component": "AboutSection",     "enabled": true  },
    { "id": "mission",  "component": "MissionSection",   "enabled": true  },
    { "id": "gallery",  "component": "GallerySection",   "enabled": true  },
    { "id": "contact",  "component": "ContactSection",   "enabled": true  },
    { "id": "footer",   "component": "FooterSection",    "enabled": true  }
  ]
}
```

- `"enabled": false` → seção desaparece do site sem precisar deletar nada
- Para reordenar seções, basta mover o objeto no array
- Nunca altere `"component"` — apenas `"enabled"` e a ordem

---

## Cores e Theming

### Cores globais — `content/brand.json`

| Campo | CSS Variable | Uso | Padrão |
|---|---|---|---|
| `primaryColor` | `--brand-primary` | botões, links, destaques | `#7c3aed` |
| `accentColor` | `--brand-accent` | elementos secundários de destaque | `#ea580c` |
| `foregroundColor` | `--brand-foreground` | texto principal | `#ffffff` |
| `backgroundColor` | `--brand-background` | fundo geral do site | `#ffffff` |
| `secondaryBackground` | `--brand-bg-secondary` | fundo de seções alternadas | `#f5f5f7` |
| `mutedColor` | `--brand-muted` | textos secundários, legendas | `#6b7280` |

```json
{
  "primaryColor": "#7c3aed",
  "accentColor": "#ea580c",
  "foregroundColor": "#ffffff",
  "backgroundColor": "#0f0f0f",
  "secondaryBackground": "#1a1a2e",
  "mutedColor": "#9ca3af"
}
```

As cores são aplicadas como variáveis CSS em tempo de execução — troca de cores **sem rebuild**.

### Override de cores por seção

Cada `content/sections/*.json` tem um campo `colors: {}` que permite sobrescrever as cores globais apenas naquela seção. Útil para seções com fundo escuro em um site claro.

```json
{
  "colors": {
    "background": "#1a1a2e",
    "foreground": "#ffffff"
  },
  "title": "Sobre Nós"
}
```

### Nav com cores próprias — `content/nav.json`

O nav também tem campo `colors: {}` para override independente:

```json
{
  "colors": {
    "background": "#000000",
    "foreground": "#ffffff"
  },
  "links": [...]
}
```

---

## Identidade da Marca — `content/brand.json`

```json
{
  "name": "Nome da Empresa",
  "tagline": "O slogan da sua empresa aqui",
  "logo": "/logo.svg",
  "logoHeight": 40,
  "favicon": "/favicon.ico",
  "fontFamily": "'Montserrat', sans-serif",
  "primaryColor": "#7c3aed",
  "accentColor": "#ea580c",
  "foregroundColor": "#ffffff",
  "backgroundColor": "#ffffff",
  "secondaryBackground": "#f5f5f7",
  "mutedColor": "#6b7280"
}
```

| Campo | Descrição |
|---|---|
| `name` | Nome exibido quando não há logo |
| `tagline` | Slogan (usado em alguns temas) |
| `logo` | Caminho do arquivo em `public/` ou URL externa. String vazia = exibe `name` |
| `logoHeight` | Altura da logo em pixels |
| `favicon` | Ícone da aba do browser |
| `fontFamily` | Fonte principal. String vazia = usa a padrão |

---

## SEO — `content/seo.json`

```json
{
  "title": "Nome da Empresa",
  "description": "Descrição clara do site para Google (até 160 caracteres).",
  "ogImage": "/og-image.jpg",
  "twitterCard": "summary_large_image"
}
```

| Campo | Onde aparece |
|---|---|
| `title` | Aba do browser, Google, compartilhamentos |
| `description` | Snippet do Google, preview do WhatsApp/Telegram |
| `ogImage` | Imagem ao compartilhar no WhatsApp, Facebook, LinkedIn |
| `twitterCard` | Tipo de card no Twitter: `summary` ou `summary_large_image` |

A imagem `ogImage` deve ter 1200×630px para melhor compatibilidade.

---

## Navegação — `content/nav.json`

```json
{
  "colors": {},
  "links": [
    { "label": "Início",   "href": "#hero" },
    { "label": "Serviços", "href": "#services" },
    { "label": "Sobre",    "href": "#about" },
    { "label": "Missão",   "href": "#mission" },
    { "label": "Galeria",  "href": "#gallery" },
    { "label": "Contato",  "href": "#contact" }
  ]
}
```

- `href` com `#id` → ancora na seção da página (scroll suave)
- `href` com `/caminho` → navega para outra página
- Para remover um link do menu, delete o objeto — não precisa mexer no `layout.json`
- A ordem dos objetos define a ordem dos links no menu

---

## Seções — o que editar em cada uma

### Hero (`content/sections/hero.json`)

```json
{
  "colors": {},
  "headline": "Bem-vindo à Nossa Empresa",
  "subheadline": "Fazemos coisas incríveis. Venha conhecer o nosso trabalho.",
  "cta": {
    "label": "Saiba mais",
    "href": "#about"
  },
  "backgroundImage": "/hero-bg.jpg"
}
```

| Campo | Descrição |
|---|---|
| `headline` | Título principal (H1) |
| `subheadline` | Texto de apoio abaixo do título |
| `cta.label` | Texto do botão de chamada para ação |
| `cta.href` | Destino do botão (âncora, rota ou URL externa) |
| `backgroundImage` | Imagem de fundo. String vazia = fundo com cor `backgroundColor` |

### Sobre (`content/sections/about.json`)

```json
{
  "colors": {},
  "title": "Sobre Nós",
  "body": "Conte aqui a história da empresa. Use \\n para quebrar linhas.",
  "image": "/about.jpg",
  "imageAlt": "Foto da equipe"
}
```

### Serviços (`content/sections/services.json`)

```json
{
  "colors": {},
  "title": "Nossos Serviços",
  "subtitle": "O que podemos fazer por você",
  "items": [
    {
      "icon": "🏗️",
      "title": "Serviço 1",
      "description": "Descreva o que este serviço oferece.",
      "detailPage": "/item/servico-1",
      "cta": { "label": "Saiba mais", "href": "#" }
    },
    {
      "icon": "⚡",
      "title": "Serviço 2",
      "description": "Descreva o que este serviço oferece."
    }
  ]
}
```

| Campo | Obrigatório | Descrição |
|---|---|---|
| `icon` | sim | Emoji ou texto curto exibido no card |
| `title` | sim | Nome do serviço |
| `description` | sim | Texto curto do card |
| `detailPage` | não | Rota para página de detalhe. Omitir = sem link |
| `cta` | não | Botão de ação extra no card |

Para adicionar um serviço: adicione um objeto ao array `items`. Para remover: delete o objeto. O grid se adapta automaticamente a qualquer número de itens.

### Missão (`content/sections/mission.json`)

```json
{
  "colors": {},
  "title": "Nossa Missão",
  "items": [
    { "icon": "⭐", "title": "Excelência",   "description": "Entregamos o melhor em cada projeto." },
    { "icon": "⏰", "title": "Pontualidade", "description": "Cumprimos prazos." },
    { "icon": "❤️", "title": "Humanismo",    "description": "Pessoas no centro de tudo." }
  ]
}
```

Ideal para valores, diferenciais ou pilares da empresa.

### Galeria (`content/sections/gallery.json`)

```json
{
  "title": "Galeria",
  "items": [
    { "src": "/fotos/foto-1.jpg", "alt": "Descrição da foto 1" },
    { "src": "/fotos/foto-2.jpg", "alt": "Descrição da foto 2" }
  ]
}
```

- Imagens em `public/` são referenciadas com `/caminho/arquivo.jpg`
- O `alt` é obrigatório para acessibilidade e SEO
- A galeria abre lightbox ao clicar. Recomendamos até 12 fotos para melhor performance

### Contato (`content/sections/contact.json`)

```json
{
  "colors": {},
  "title": "Contato",
  "email": "contato@empresa.com",
  "phone": "(11) 9 0000-0000",
  "address": "Rua Exemplo, 123 — Cidade, Estado",
  "social": {
    "instagram": "usuario_sem_arroba",
    "facebook": "usuario_ou_url",
    "whatsapp": "11900000000"
  }
}
```

- Campos vazios (`""`) são omitidos da renderização
- `whatsapp` deve conter apenas números (DDI + DDD + número), sem espaços ou símbolos
- `instagram` e `facebook` aceitam apenas o username (sem `@` ou URL completa)

### Rodapé (`content/sections/footer.json`)

```json
{
  "colors": {
    "background": "#111111",
    "foreground": "#ffffff",
    "muted": "#9ca3af"
  },
  "slogan": "Sua empresa com excelência e inovação.",
  "address": "Rua Exemplo, 123 — Cidade, Estado",
  "phones": ["(11) 9 0000-0000"],
  "email": "contato@empresa.com",
  "quickLinks": [
    { "label": "Início",   "href": "#hero" },
    { "label": "Serviços", "href": "#services" },
    { "label": "Sobre",    "href": "#about" },
    { "label": "Missão",   "href": "#mission" },
    { "label": "Contato",  "href": "#contact" }
  ],
  "copyright": "© 2025 Empresa. Todos os direitos reservados."
}
```

| Campo | Descrição |
|---|---|
| `slogan` | Texto exibido abaixo do logo no rodapé |
| `address` | Endereço físico |
| `phones` | Array de telefones (pode ter mais de um) |
| `email` | E-mail de contato |
| `quickLinks` | Links rápidos da coluna de navegação |
| `copyright` | Texto de direitos autorais no rodapé inferior |

- `colors.muted` controla a cor dos textos secundários (endereço, e-mail, etc.)
- Para múltiplos telefones: `"phones": ["(11) 9 0000-0000", "(11) 9 1111-1111"]`

---

## Páginas de detalhe (`content/details/items/`)

Cada arquivo JSON cria uma página. O nome do arquivo vira a URL.

Exemplo: `servico-consultoria.json` → `/item/servico-consultoria`

```json
{
  "colors": {},
  "title": "Nome do Serviço",
  "description": "Descrição curta (aparece também no card de serviços).",
  "badge": "R$ 00,00",
  "images": [
    { "src": "/fotos/servico-1.jpg", "alt": "Serviço 1 — imagem 1" }
  ],
  "body": "Texto longo descrevendo o serviço em detalhes.",
  "features": [
    "Característica ou destaque 1",
    "Característica ou destaque 2"
  ],
  "cta": {
    "label": "Entrar em contato",
    "href": "#contact"
  }
}
```

| Campo | Descrição |
|---|---|
| `colors` | Override de cores da página (mesmas chaves das seções: `background`, `foreground`, etc.) |
| `title` | Nome do item (H1 da página de detalhe) |
| `description` | Texto curto (aparece também no card em `services.json`) |
| `badge` | Etiqueta destacada (preço, categoria, status) |
| `images` | Galeria de imagens da página de detalhe |
| `body` | Texto longo com descrição completa |
| `features` | Lista de características ou diferenciais |
| `cta` | Botão de chamada para ação no final da página |

Para **criar**: crie o arquivo JSON com o slug desejado.
Para **remover**: delete o arquivo. Verifique se nenhum `services.json` aponta para o slug removido via `detailPage`.

---

## Como adicionar uma nova seção

Siga estes passos em ordem:

**1. Crie o JSON de conteúdo**

```json
// content/sections/nova-secao.json
{
  "colors": {},
  "title": "Título da Nova Seção",
  "items": []
}
```

**2. Crie o componente Vue**

```vue
<!-- components/sections/NovaSectionSection.vue -->
<script setup lang="ts">
const { sections } = useAppConfig()
const data = sections.novaSection
</script>

<template>
  <section :id="'nova-section'">
    <h2>{{ data.title }}</h2>
  </section>
</template>
```

**3. Registre em `app.config.ts`**

```ts
import novaSection from './content/sections/nova-secao.json'

export default defineAppConfig({
  sections: {
    // ...existentes
    novaSection,
  },
})
```

**4. Registre em `pages/index.vue`**

```ts
import NovaSectionSection from '~/components/sections/NovaSectionSection.vue'

const sectionComponents: Record<string, Component> = {
  // ...existentes
  NovaSectionSection,
}
```

**5. Adicione ao `content/layout.json`**

```json
{ "id": "nova-section", "component": "NovaSectionSection", "enabled": true }
```

Posicione o objeto na posição desejada dentro do array.

---

## Histórico de funcionalidades

| Data | Feature | Descrição |
|---|---|---|
| 2025-04 | Sistema base de seções | `layout.json` com `enabled`/ordem, componentes dinâmicos via `pages/index.vue` |
| 2025-04 | Theming runtime | CSS vars via `app.vue`, `@theme inline` no Tailwind, override por seção |
| 2025-04 | Logo/favicon/fonte configuráveis | `brand.json` com `logo`, `favicon`, `fontFamily` |
| 2025-04 | Galeria com lightbox | `GallerySection` com transição e navegação por teclado |
| 2025-04 | Páginas de detalhe | `/item/[slug].vue` gerado a partir de `content/details/items/*.json` |
| 2026-04 | Versionamento do guia | Campo `version` no frontmatter + link para o repo público oficial |
| 2026-04 | `colors` nas páginas de detalhe | Campo `colors` suportado em `content/details/items/*.json` |
| 2026-04 | FooterSection | Seção de rodapé com 4 colunas + copyright config-driven via `content/sections/footer.json` |
| 2026-04 | Grid adaptativo em serviços | `auto-fit` no grid — se adapta a 1, 2 ou N itens sem espaço vazio |

> Ao implementar uma nova feature, adicione uma linha nesta tabela com a data e descrição resumida.

---

## Imagens

1. Coloque o arquivo em `public/` do repositório
2. Referencie nos JSONs como `/nome-do-arquivo.jpg`
3. Formatos aceitos: `.jpg`, `.png`, `.webp`, `.svg`, `.gif`
4. Para `ogImage` (compartilhamento social): use 1200×630px

---

## Após editar

Qualquer push na branch `main` dispara o deploy automático. O site fica no ar em alguns minutos.

Se precisar de ajuda com algum campo específico ou quiser ver como o JSON deve ficar após uma mudança, é só pedir.

---
name: content-guide
version: 1.12.0
description: >
  Guia completo de edição de conteúdo do projeto my-sites. Use este skill sempre que o
  usuário quiser editar textos, imagens, cores, seções ou qualquer configuração do site —
  mesmo que ele diga apenas "quero mudar o texto da hero" ou "como desativo a galeria".
  Também use quando alguém perguntar como adicionar uma nova seção, novo item de serviço,
  nova página de detalhe, ou como o sistema de theming funciona.
---

# Guia de Conteúdo — my-sites

> **Versão deste guia:** `1.12.0`
> Verifique se há uma versão mais recente no repositório oficial:
> https://github.com/code7tecbr/my-sites-action/blob/main/.claude/commands/content-guide.md

## Como o sistema funciona

Todo conteúdo editável fica em `content/`. Os arquivos JSON são importados **exclusivamente** por `app.config.ts`, que os expõe via `useAppConfig()`. Componentes nunca importam JSONs diretamente.

```
content/
├── brand.json          ← identidade visual e cores globais
├── nav.json            ← links do menu de navegação
├── seo.json            ← metadados de SEO e compartilhamento
├── analytics.json      ← IDs de rastreamento (GA4 e Cloudflare Analytics)
├── layout.json         ← quais seções estão ativas e em qual ordem
└── sections/
│   ├── hero.json
│   ├── about.json
│   ├── services.json
│   ├── mission.json
│   ├── gallery.json
│   ├── contact.json
│   ├── reviews.json
│   └── instagram.json
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
    { "id": "contact",   "component": "ContactSection",   "enabled": true  },
    { "id": "reviews",   "component": "ReviewsSection",   "enabled": true  },
    { "id": "instagram", "component": "InstagramSection",  "enabled": true  }
  ]
}
```

- `"enabled": false` → seção some da página sem precisar deletar nada
- Para reordenar seções, basta mover o objeto no array
- O campo `component` deve corresponder exatamente ao nome do componente registrado em `pages/index.vue`

---

## Sistema de Layouts

Cada seção suporta um campo `layout` que seleciona o template visual a usar. O padrão é sempre `"default"`.

```json
{ "layout": "default" }
```

Trocar o layout não exige nenhuma alteração no código — basta editar o JSON da seção.

### Layouts disponíveis por seção

| Seção | Layout | Descrição |
|---|---|---|
| `about` | `default` | Texto à esquerda + foto à direita |
| `about` | `pillars` | Título centralizado + 3 colunas de valores (sem foto) |
| todas as outras | `default` | Layout padrão (único por enquanto) |

### Campos de apresentação de ícones

Disponíveis em `services.json` e `mission.json`:

| Campo | Valores | Descrição |
|---|---|---|
| `iconLayout` | `"inline"` / `"stacked"` | `inline` = ícone e título na mesma linha; `stacked` = ícone acima do título |
| `align` | `"left"` / `"center"` / `"right"` | Alinhamento horizontal dos itens dentro do card |

> Novos layouts são adicionados por seção conforme a necessidade. O campo `layout` já existe em todos os JSONs como reserva para variantes futuras.

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

**Como funciona internamente:** `app.vue` injeta essas cores como CSS variables no `:root` em tempo de execução. O Tailwind usa `@theme inline` para mapear as vars como utilitários (`text-brand`, `bg-accent`, etc.). Resultado: troca de cores **sem rebuild**.

### Override de cores por seção

Cada `content/sections/*.json` tem um campo `colors: {}` que permite sobrescrever as cores globais apenas naquela seção. Útil para seções com fundo escuro em um site claro, por exemplo.

```json
{
  "colors": {
    "background": "#1a1a2e",
    "foreground": "#ffffff"
  },
  "title": "Sobre Nós"
}
```

> As chaves disponíveis em `colors` dependem do que cada componente consome. Consulte o componente correspondente em `components/sections/` para ver quais variáveis ele respeita.

> **Atenção:** `foreground` em `colors` é uma CSS variable aplicada à `<section>` inteira e cascateia para **todos** os textos filhos — não apenas o título. Para colorir somente o título da seção (H2), use o campo `titleColor` disponível em todas as seções com `title`. Para colorir somente o título de um item/pilar (H3), use `items[].titleColor` ou `pillars[].titleColor`.

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
  "backgroundColor": "#0f0f0f",
  "secondaryBackground": "#1a1a1a",
  "mutedColor": "#9ca3af"
}
```

| Campo | Descrição |
|---|---|
| `name` | Nome exibido quando não há logo |
| `tagline` | Slogan (usado em alguns temas) |
| `logo` | Caminho do arquivo ou URL externa. String vazia = exibe `name` |
| `logoHeight` | Altura da logo em pixels |
| `favicon` | Ícone da aba do browser |
| `fontFamily` | Fonte principal. String vazia = usa Inter (padrão) |
| `foregroundColor` | Cor do texto principal — deve contrastar com `backgroundColor` |
| `backgroundColor` | Cor de fundo geral do site |
| `secondaryBackground` | Fundo de seções alternadas (ex: galeria) |
| `mutedColor` | Cor de textos secundários, legendas e detalhes |

> Para usar uma fonte do Google Fonts, inclua o `@import` no campo `fontFamily` **ou** adicione a tag `<link>` manualmente em `app.vue`.

---

## SEO — `content/seo.json`

```json
{
  "title": "Nome da Empresa",
  "description": "Descrição objetiva para mecanismos de busca (até ~160 caracteres).",
  "ogImage": "/og-image.jpg",
  "twitterCard": "summary_large_image",
  "siteUrl": "https://seusite.com.br"
}
```

| Campo | Onde aparece |
|---|---|
| `title` | Aba do browser, Google, compartilhamentos |
| `description` | Snippet do Google, preview do WhatsApp/Telegram |
| `ogImage` | Imagem ao compartilhar no WhatsApp, Facebook, LinkedIn |
| `twitterCard` | Tipo de card no Twitter: `summary` ou `summary_large_image` |
| `siteUrl` | URL canônica do site — usada pelos dados estruturados (Schema.org) |

A imagem `ogImage` deve ter 1200×630px para melhor compatibilidade.

> **Dados estruturados (Schema.org):** o site injeta automaticamente JSON-LD no `<head>` com os schemas `WebSite` e `LocalBusiness`, incluindo catálogo de serviços, avaliações e redes sociais. Isso melhora a visibilidade no Google (rich results) e a compreensão por IAs como ChatGPT e Perplexity. Preencha `siteUrl` corretamente para que os schemas funcionem.

---

## Analytics — `content/analytics.json`

Controla os provedores de rastreamento do site. Ambos os campos são opcionais — string vazia desativa o provider correspondente.

```json
{
  "gaId": "G-XXXXXXXXXX",
  "cfToken": "seu-token-cf"
}
```

| Campo | Provedor | Como obter |
|---|---|---|
| `gaId` | Google Analytics 4 | Google Analytics → Admin → Fluxos de dados → ID de medição |
| `cfToken` | Cloudflare Web Analytics | Painel CF → Web Analytics → Sites → token do beacon |

**Características de cada provedor:**

- **GA4** (`gaId`): rastreamento completo com eventos, funis, relatórios avançados. Requer aceite de cookies dependendo da configuração de privacidade.
- **Cloudflare Web Analytics** (`cfToken`): analytics de privacidade, sem cookies, sem impacto na performance (script ~1 KB). Dados visíveis no painel da Cloudflare. Ideal como complemento leve ao GA4 ou substituto LGPD-friendly.

> Ambos os IDs são **públicos por design** — ficam no HTML final e qualquer visitante pode vê-los. Não são secrets e não precisam de variáveis de ambiente.

Os dois podem estar ativos simultaneamente. Deixar o campo vazio (`""`) desativa o provider sem precisar remover a chave do JSON.

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

- `href` com `#id` → ancora na seção da página (scroll suave via CSS)
- `href` com `/caminho` → navega para outra página
- Para remover um link do menu, delete o objeto — não precisa mexer no layout.json
- A ordem dos objetos define a ordem dos links no menu

---

## Seções — Referência de campos

### Hero — `content/sections/hero.json`

Primeira seção da página, com chamada principal e CTA. Suporta carrossel de até **5 imagens** com cross-fade automático a cada 5 segundos e indicadores clicáveis.

```json
{
  "colors": {},
  "headline": "Bem-vindo à Nossa Empresa",
  "subheadline": "Fazemos coisas incríveis. Venha conhecer o nosso trabalho.",
  "cta": {
    "label": "Saiba mais",
    "href": "#about"
  },
  "backgroundImages": [
    "/images/hero/slide-1.jpg",
    "/images/hero/slide-2.jpg"
  ]
}
```

| Campo | Descrição |
|---|---|
| `headline` | Título principal (H1) |
| `subheadline` | Texto de apoio abaixo do título |
| `cta.label` | Texto do botão de chamada para ação |
| `cta.href` | Destino do botão (âncora, rota ou URL externa) |
| `backgroundImages` | Array de imagens para o carrossel (máximo 5). Coloque os arquivos em `public/images/hero/`. Com apenas 1 item, os indicadores não aparecem |

---

### Sobre — `content/sections/about.json`

Suporta dois layouts selecionáveis via campo `layout`.

**`layout: "default"`** — texto à esquerda, foto à direita:

```json
{
  "colors": {},
  "layout": "default",
  "title": "Sobre Nós",
  "body": "Conte aqui a história da empresa...",
  "image": "/about.jpg",
  "imageAlt": "Foto da equipe"
}
```

**`layout: "pillars"`** — título centralizado + 3 colunas de valores, sem foto:

```json
{
  "colors": {},
  "layout": "pillars",
  "title": "Sobre Nós",
  "pillars": [
    { "icon": "💡", "title": "Inovação",        "text": "Soluções personalizadas e inovadoras." },
    { "iconSvg": "/icons/estrela.svg", "iconSize": "lg", "title": "Excelência", "text": "O melhor em cada projeto." },
    { "title": "Profissionalismo","text": "Equipe competente e dedicada às suas necessidades." }
  ]
}
```

| Campo | Layout | Descrição |
|---|---|---|
| `layout` | ambos | `"default"` ou `"pillars"`. Padrão: `"default"` |
| `title` | ambos | Título da seção |
| `titleColor` | ambos | Cor exclusiva do título da seção (H2). Não afeta outros textos |
| `body` | default | Texto longo. Suporta `\n` para quebras de linha |
| `image` | default | Foto ilustrativa ao lado do texto |
| `imageAlt` | default | Texto alternativo da imagem (acessibilidade e SEO) |
| `pillars` | pillars | Array de objetos com os valores da empresa |
| `pillars[].icon` | pillars | Emoji exibido acima do título (opcional; fallback quando não há `iconSvg`) |
| `pillars[].iconSvg` | pillars | Caminho para SVG em `public/` (opcional; tem prioridade sobre `icon`) |
| `pillars[].iconSize` | pillars | `"sm"`, `"md"`, `"lg"` ou `"xl"`. Padrão: `"md"` |
| `pillars[].bg` | pillars | Cor de fundo do card (qualquer valor CSS: `"#1a1a2e"`, `"rgba(0,0,0,0.3)"`). Quando presente, a borda do card é removida automaticamente |
| `pillars[].titleColor` | pillars | Cor exclusiva do título do pilar (H3). Não afeta outros textos |
| `pillars[].title` | pillars | Nome do pilar |
| `pillars[].text` | pillars | Descrição do pilar |

---

### Serviços — `content/sections/services.json`

```json
{
  "colors": {},
  "layout": "default",
  "iconLayout": "inline",
  "align": "left",
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
| `iconLayout` | não | `"inline"` (ícone e título na mesma linha) ou `"stacked"` (ícone acima do título). Padrão: `"inline"` |
| `align` | não | Alinhamento dos itens: `"left"`, `"center"` ou `"right"`. Padrão: `"left"` |
| `titleColor` | não | Cor exclusiva do título da seção (H2). Não afeta outros textos |
| `icon` | sim | Emoji ou texto curto exibido no card (fallback quando não há `iconSvg`) |
| `iconSvg` | não | Caminho para arquivo SVG em `public/` (ex: `"/icons/servico1.svg"`). Se presente, tem prioridade sobre `icon` |
| `iconSize` | não | Tamanho do SVG: `"sm"`, `"md"`, `"lg"` ou `"xl"`. Padrão: `"md"` |
| `title` | sim | Nome do serviço |
| `items[].titleColor` | não | Cor exclusiva do título do item (H3). Não afeta outros textos |
| `description` | sim | Texto curto do card |
| `detailPage` | não | Rota para página de detalhe. Omitir = sem link |
| `cta` | não | Botão de ação extra no card |

Para adicionar um serviço, adicione um objeto ao array `items`. Para remover, delete o objeto.

---

### Missão — `content/sections/mission.json`

```json
{
  "colors": {},
  "layout": "default",
  "iconLayout": "inline",
  "align": "center",
  "title": "Nossa Missão",
  "items": [
    { "icon": "⭐", "title": "Excelência",   "description": "Entregamos o melhor em cada projeto." },
    { "icon": "⏰", "title": "Pontualidade", "description": "Cumprimos prazos." },
    { "icon": "❤️", "title": "Humanismo",    "description": "Pessoas no centro de tudo." }
  ]
}
```

Mesma estrutura de serviços, mas sem `detailPage` ou `cta`. Ideal para valores, diferenciais ou pilares da empresa.

| Campo | Descrição |
|---|---|
| `iconLayout` | `"inline"` (ícone e título na mesma linha) ou `"stacked"` (ícone acima do título). Padrão: `"inline"` |
| `align` | Alinhamento dos itens: `"left"`, `"center"` ou `"right"`. Padrão: `"center"` |
| `titleColor` | Cor exclusiva do título da seção (H2). Não afeta outros textos |
| `icon` | Emoji ou texto curto (fallback quando não há `iconSvg`) |
| `iconSvg` | Caminho para arquivo SVG em `public/` (ex: `"/icons/missao1.svg"`). Se presente, tem prioridade sobre `icon` |
| `iconSize` | Tamanho do SVG: `"sm"`, `"md"`, `"lg"` ou `"xl"`. Padrão: `"md"` |
| `bg` | Cor de fundo do item (qualquer valor CSS: `"#1a1a2e"`, `"rgba(0,0,0,0.3)"`). Quando presente, adiciona `rounded-2xl p-6` ao card automaticamente |
| `items[].titleColor` | Cor exclusiva do título do item (H3). Não afeta outros textos |

---

### Galeria — `content/sections/gallery.json`

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
- A galeria abre lightbox ao clicar. Sem limite de itens, mas recomendamos até 12 para performance
- `titleColor` (opcional): cor exclusiva do título da seção. Ex: `"titleColor": "#7c3aed"`

---

### Contato — `content/sections/contact.json`

```json
{
  "colors": {},
  "title": "Contato",
  "email": "contato@empresa.com",
  "phone": "(11) 9 0000-0000",
  "address": "Rua Exemplo, 123 — Cidade, Estado",
  "social": {
    "instagram": "meu_perfil",
    "facebook": "meu_perfil",
    "whatsapp": "11900000000"
  }
}
```

- Campos vazios (`""`) são omitidos da renderização
- `titleColor` (opcional): cor exclusiva do título da seção. Ex: `"titleColor": "#7c3aed"`
- `instagram` e `facebook` → apenas o **username**, sem `@`, sem `https://`, sem URL completa (ex: `"meu_perfil"`, não `"instagram.com/meu_perfil"`)
- `whatsapp` → apenas números: DDI + DDD + número, sem espaços ou símbolos (ex: `"5511900000000"`)

---

### Footer — `content/sections/footer.json`

O footer é renderizado **automaticamente** no layout — não precisa estar em `layout.json`. Para desativar, use `"enabled": false`.

```json
{
  "enabled": true,
  "colors": {
    "background": "#111111",
    "foreground": "#ffffff",
    "muted": "#9ca3af"
  },
  "slogan": "Sua empresa com excelência e inovação.",
  "address": "Rua Exemplo, 123 — Cidade, Estado",
  "phones": ["(11) 9 0000-0000"],
  "email": "contato@empresa.com",
  "social": {
    "instagram": "meu_perfil",
    "facebook": "meu_perfil",
    "whatsapp": "11900000000"
  },
  "quickLinks": [
    { "label": "Início",   "href": "#hero" },
    { "label": "Serviços", "href": "#services" }
  ],
  "copyright": "© 2025 Empresa. Todos os direitos reservados."
}
```

| Campo | Descrição |
|---|---|
| `enabled` | `false` desativa o footer. Padrão: `true` |
| `slogan` | Texto exibido abaixo do nome da empresa na 1ª coluna |
| `phones` | Array de telefones (aceita mais de um) |
| `social.instagram` | Apenas o username, sem `@` ou URL |
| `social.facebook` | Apenas o username, sem `@` ou URL |
| `social.whatsapp` | Apenas números: DDI + DDD + número (ex: `"5511900000000"`) |
| `quickLinks` | Links da 4ª coluna. Mesma estrutura do `nav.json` |
| `copyright` | Texto da barra inferior à esquerda |

---

### Avaliações — `content/sections/reviews.json`

Seção de depoimentos e avaliações de clientes com estrelas, nota média e avatar.

```json
{
  "colors": {},
  "title": "O que dizem sobre nós",
  "subtitle": "Avaliações",
  "items": [
    {
      "name": "Ana Silva",
      "role": "Cliente",
      "avatar": "/fotos/ana.jpg",
      "rating": 5,
      "text": "Excelente atendimento! Superou todas as minhas expectativas."
    }
  ]
}
```

| Campo | Obrigatório | Descrição |
|---|---|---|
| `titleColor` | não | Cor exclusiva do título da seção (H2). Não afeta outros textos |
| `name` | sim | Nome do cliente |
| `role` | não | Cargo ou descrição (ex: "Cliente", "CEO da Empresa X") |
| `avatar` | não | Foto do cliente. Se vazio, exibe a inicial do nome |
| `rating` | sim | Nota de 1 a 5 |
| `text` | sim | Texto do depoimento |

> As avaliações alimentam automaticamente o schema `AggregateRating` do Google. Quanto mais avaliações e maior a média, maior a chance de aparecer a nota em estrelas nos resultados de busca.

---

### Instagram — `content/sections/instagram.json`

Feed curado do Instagram com grid de fotos e link para o perfil.

```json
{
  "colors": {},
  "title": "Instagram",
  "subtitle": "Nos siga nas redes",
  "username": "@seu_usuario",
  "profileUrl": "https://instagram.com/seu_usuario",
  "items": [
    {
      "url": "https://instagram.com/p/codigo-do-post",
      "image": "/fotos/insta-1.jpg",
      "alt": "Descrição do post"
    }
  ]
}
```

| Campo | Obrigatório | Descrição |
|---|---|---|
| `titleColor` | não | Cor exclusiva do título da seção (H2). Não afeta outros textos |
| `username` | não | Handle exibido abaixo do título (ex: `@minhaempresa`) |
| `profileUrl` | não | URL completa do perfil — ativa o botão "Ver perfil no Instagram" |
| `items[].url` | sim | Link direto para o post no Instagram |
| `items[].image` | sim | Imagem do post (salve em `public/` ou use URL externa) |
| `items[].alt` | sim | Descrição da imagem (acessibilidade e SEO) |

- Grid responsivo: 2 colunas no mobile, 3 no desktop
- Recomendamos múltiplos de 3 para preencher o grid sem espaços vazios (ex: 3, 6, 9)
- As fotos abrem o post original no Instagram em nova aba ao clicar

---

## Páginas de Detalhe — `content/details/items/`

Cada arquivo JSON cria uma página em `/item/[slug]`, onde o slug é o **nome do arquivo sem `.json`**.

Exemplo: `content/details/items/servico-consultoria.json` → `/item/servico-consultoria`

```json
{
  "colors": {},
  "title": "Serviço 1",
  "description": "Descrição curta exibida no card da seção de serviços.",
  "badge": "R$ 00,00",
  "images": [
    { "src": "/fotos/servico-1a.jpg", "alt": "Serviço 1 — imagem 1" },
    { "src": "/fotos/servico-1b.jpg", "alt": "Serviço 1 — imagem 2" }
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
| `colors` | Override de cores para a página (mesmas chaves das seções: `background`, `foreground`, etc.) |
| `title` | Nome do item (H1 da página de detalhe) |
| `description` | Texto curto (aparece também no card em `services.json`) |
| `badge` | Etiqueta destacada (preço, categoria, status) |
| `images` | Galeria de imagens da página de detalhe |
| `body` | Texto longo com descrição completa |
| `features` | Lista de características ou diferenciais |
| `cta` | Botão de chamada para ação no final da página |

Para **criar** uma nova página de detalhe: crie o arquivo JSON com o slug desejado.
Para **remover**: delete o arquivo. Garanta que nenhum `services.json` aponte para o slug removido via `detailPage`.

---

## Como adicionar uma nova seção

Siga estes passos em ordem:

**1. Crie o JSON de conteúdo**

```bash
# content/sections/nova-secao.json
{
  "colors": {},
  "title": "Título da Nova Seção",
  "items": []
}
```

**2. Crie o componente Vue**

```bash
# components/sections/NovaSectionSection.vue
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
  // ...existentes
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

## Roadmap de funcionalidades

<!-- TODO: atualizar esta seção conforme features forem implementadas -->

### Planejado

- **Admin Lite (Fase 2)**: painel `/admin` com edição visual dos JSONs via GitHub API. Save = commit automático → rebuild → deploy no CF Pages.
- **Internacionalização (i18n)**: suporte a múltiplos idiomas via arquivos `content/[locale]/`.
- **Temas predefinidos**: seletor de temas no admin que troca todas as cores de uma vez.
- **Suporte a vídeo no hero**: campo `backgroundVideo` alternativo ao `backgroundImages`.
- **Blog/notícias**: `content/posts/` com listagem e páginas de artigo.
- **Instagram feed dinâmico**: busca automática dos posts via Instagram Graph API (issue #2).

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
| 2026-04 | Footer no layout com `enabled` | Footer renderizado no layout por padrão; desativar com `"enabled": false` em `footer.json` |
| 2026-04 | `social` no footer | Campos `instagram`, `facebook`, `whatsapp` disponíveis em `footer.json` |
| 2026-04 | Analytics dual-provider | `content/analytics.json` com suporte a GA4 (`nuxt-gtag`) e Cloudflare Web Analytics simultâneos |
| 2026-04 | Release v0.1.3 | Versionamento sincronizado com package.json |
| 2026-04 | Release v0.1.4 | Versionamento sincronizado com package.json |
| 2026-04 | Carrossel no hero | `backgroundImages` (array até 5) com cross-fade e indicadores clicáveis |
| 2026-04 | CTA tracking na página de detalhe | Evento `cta_click` enviado ao GA4 via `useTrackEvent` com `item_slug` e `cta_label` |
| 2026-04 | ReviewsSection | Seção de avaliações com estrelas, nota média e avatar; alimenta `AggregateRating` no Schema.org |
| 2026-04 | InstagramSection | Feed curado do Instagram via `content/sections/instagram.json` com grid responsivo |
| 2026-04 | Schema.org / dados estruturados | `composables/useStructuredData.ts` injeta JSON-LD (`WebSite` + `LocalBusiness`) no `<head>` |
| 2026-04 | `siteUrl` no seo.json | Campo adicionado para alimentar os schemas de dados estruturados |
| 2026-04 | Sistema de layouts | Campo `layout` em todas as seções; `AboutSection` suporta `"default"` e `"pillars"` |
| 2026-04 | `iconLayout` e `align` | Campos de apresentação de ícones em ServicesSection e MissionSection |
| 2026-04 | `iconSvg` opcional nos ícones | Campo `iconSvg` em itens de ServicesSection e MissionSection — SVG tem prioridade sobre emoji `icon` |
| 2026-04 | `iconSize` por item | Campo `iconSize` (`sm`/`md`/`lg`/`xl`) em ServicesSection, MissionSection e AboutSection (pillars) |
| 2026-04 | Ícones nos pillars do AboutSection | Campos `icon`, `iconSvg` e `iconSize` disponíveis nos itens do layout `"pillars"` |
| 2026-04 | Campo `bg` por item | Fundo individual por item em MissionSection e por pilar em AboutSection (`"pillars"`) |
| 2026-04 | `titleColor` por título | Campo opcional em todas as seções com `title` (H2) e nos itens/pillars de Mission, Services e About (H3) |
| 2026-04 | Fix `bg` no AboutSection | Borda do card removida automaticamente quando `pillars[].bg` está presente (consistente com MissionSection) |
| 2026-04 | Fix ícone centralizado (stacked) | Ícones SVG agora centralizam corretamente com `mx-auto` em MissionSection e ServicesSection |

> Ao implementar uma nova feature, adicione uma linha nesta tabela com a data e descrição resumida.

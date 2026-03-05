# Ръководство: Поправка на критичните SEO проблеми (1.1)
## За Junior Developer

**Преди да започнеш:** Прочети цялото ръководство. Не прави промени директно на живия сайт — винаги работи през `shopify theme dev` и тествай локално.

---

## Съдържание
1. [Липсва meta description на началната страница](#1-meta-description)
2. [Липсва structured data (JSON-LD)](#2-structured-data)
3. [Само 3 продукта в каталога](#3-thin-content)
4. [Филтрите на колекцията са празни](#4-empty-filters)
5. [Дублиран Ahrefs analytics скрипт](#5-duplicate-ahrefs)

---

## Подготовка

### 1. Стартирай dev среда
```bash
cd d:\Nikolay\Temporary\shopify\app
shopify theme dev --store d20e1f.myshopify.com
```
Това отваря preview на `http://127.0.0.1:9292`. Всички промени се виждат тук в реално време.

### 2. Направи backup
Преди каквато и да е промяна, отиди в Shopify Admin > Online Store > Themes > Actions > **Duplicate**. Така имаш копие, ако нещо се обърка.

### 3. Файлова структура — къде какво е
```
layout/theme.liquid          <- Главният layout. Тук е <head> секцията.
snippets/social-meta-tags.liquid  <- OG и Twitter Card мета тагове.
snippets/product-json-ld.liquid   <- Съществуващ Product schema (JSON-LD).
sections/main-product.liquid      <- Продуктова страница (има microdata).
sections/collection-sidebar.liquid <- Sidebar филтри на колекции.
config/settings_schema.json       <- Дефиниции на theme settings.
```

---

## 1. Meta Description
**Файл:** [layout/theme.liquid](layout/theme.liquid) — редове 25-27

### Текущо състояние (проверено на 2026-03-05)
Кодът в темата вече работи правилно:
```liquid
{% if page_description %}
  <meta name="description" content="{{ page_description | escape }}">
{% endif %}
```

**Реална проверка на живия сайт:**

| # | Страница | Meta Description | Статус | Проблем |
|---|----------|-----------------|--------|---------|
| 1 | **Homepage** `/` | `Discover top-quality custom DTF printing services in South Florida. Our expert team delivers vibrant designs on apparel with precision and speed. Contact us today to elevate your brand with our innovative printing solutions!` | Има, но прекалено дълго | 222 символа — Google отрязва след ~160 |
| 2 | **Collections** `/collections/all` | — | **ЛИПСВА** | Няма meta description. `og:description` пада към homepage текста |
| 3 | **About Us** `/pages/about-us` | `Gravity Design was founded with a simple yet powerful vision: to provide individuals and teams with a platform to create personalized clothing that reflects...` | Има, но прекалено дълго | Копирано от body текста, отрязано. Не е написано специално за SEO |
| 4 | **DTF Gang Sheet Builder** (product) | `Build custom DTF gang sheets starting at $5. Upload your designs, choose your size (22" wide, up to 504"). 100% satisfaction guaranteed! Order now!` | **OK** | Добре написано, правилна дължина |
| 5 | **Custom Stickers** (product) | — | **404 NOT FOUND** | URL-ът `/products/custom-stickers-and-decals-high-quality-vinyl-printing` връща 404! |

**Допълнителна находка:** На продукта DTF Gang Sheet Builder, `og:description` е **различно** от `meta description` — OG дърпа от product description, докато meta description е от SEO полето. При споделяне в соц. мрежи се показва различен текст.

**Title тагове:** Навсякъде използват формат `Заглавие– Gravity Designs` (тире без спейс). Може да се коригира в theme.liquid ред 18-23.

### Какво казва документацията на Sofine (Alothemes)?
**Нищо.** Документацията на темата (https://alotheme.gitbook.io/docs-sofine) е крайно бедна — съдържа само промоционално описание и контакт за support (labertheme@gmail.com). Няма секция за SEO настройки.

**Находка в кода:** Темата дефинира setting `home_keywords` в [config/settings_schema.json](config/settings_schema.json) (ред 19), но **не го използва никъде в Liquid кода** — мъртъв код. Meta keywords не се ползват от Google от 2009 г.

Темата **НЕ предоставя** собствен механизъм за meta description — разчита на вградената Shopify променлива `page_description`.

### Откъде идва `page_description`?
Глобална Shopify променлива. Стойността зависи от типа страница:

| Тип страница | Откъде идва стойността |
|---|---|
| **Homepage** | Shopify Admin > Online Store > **Preferences** > "Homepage meta description" |
| **Product** | Products > [продукт] > "Search engine listing" > "Meta description" |
| **Collection** | Products > Collections > [колекция] > "Search engine listing" > "Meta description" |
| **Page** | Pages > [страница] > "Search engine listing" > "Meta description" |
| **Blog post** | Blog posts > [статия] > "Search engine listing" > "Meta description" |

Ако полето е празно — `page_description` е `nil` — и `<meta name="description">` **изобщо не се рендерира**.

### Какво трябва да се поправи

#### Проблем 1: Homepage описанието е прекалено дълго (222 символа)
1. Отиди в **Shopify Admin** > **Online Store** > **Preferences**
2. Намери полето **"Homepage meta description"**
3. Съкрати до max 155 символа. Примерен вариант:
```
Custom DTF heat transfers & vinyl stickers in South Florida. 24-48h turnaround, no minimums, free design support. Order online today!
```
(145 символа — перфектно)
4. **Save**

#### Проблем 2: Collections `/collections/all` НЯМА meta description
1. **Shopify Admin** > **Products** > **Collections** > избери "All" колекцията
2. Скролни до "Search engine listing" > натисни **"Edit website SEO"**
3. Попълни meta description:
```
Shop all custom DTF transfers, vinyl stickers and decals at Gravity Designs. Premium quality, fast turnaround. Starting from $2. Free shipping available.
```
4. **Save**

#### Проблем 3: About Us описанието е copy-paste от body текста
1. **Online Store** > **Pages** > **About Us**
2. "Search engine listing" > **"Edit website SEO"**
3. Замени с кратко, целенасочено описание:
```
Gravity Designs is a custom printing company in Coral Springs, FL. We specialize in DTF heat transfers, vinyl stickers & personalized apparel for teams and businesses.
```
(167 символа — малко дълго, скъси до 155)
4. **Save**

#### Проблем 4: Custom Stickers продуктът връща 404!
Това е **критичен проблем** — не е свързан с meta description, но е открит при проверката.
1. Провери в **Shopify Admin** > **Products** дали продуктът съществува
2. Ако URL-ът е бил сменен — добави redirect:
   - **Online Store** > **Navigation** > **URL Redirects** > **Add URL redirect**
   - From: `/products/custom-stickers-and-decals-high-quality-vinyl-printing`
   - To: `/products/НОВИЯТ-URL`
3. Провери навигацията и всички линкове на сайта, които сочат към стария URL

#### Проблем 5: OG description различно от meta description (DTF Gang Sheet)
Това е поведение на snippet-а [snippets/social-meta-tags.liquid](snippets/social-meta-tags.liquid) ред 11:
```liquid
{%- assign og_description = product.description | default: page_description | ... -%}
```
Приоритетът е: `product.description` > `page_description`. За да използва SEO описанието:

Отвори [snippets/social-meta-tags.liquid](snippets/social-meta-tags.liquid), замени ред 11:

**Преди:**
```liquid
  {%- assign og_description = product.description | default: page_description | default: shop.description | strip_html | truncate: 155 -%}
```

**След:**
```liquid
  {%- assign og_description = page_description | default: product.description | default: shop.description | strip_html | truncate: 155 -%}
```

Така `page_description` (SEO полето) ще има приоритет. Ако е празно — fallback към product.description.

### Допълнителна защита: Fallback в кода
За страниците, където `page_description` е празен (напр. ако забравиш да попълниш колекция), добави fallback. Редактирай [layout/theme.liquid](layout/theme.liquid) — замени редове 25-27:

**Преди:**
```liquid
{% if page_description %}
  <meta name="description" content="{{ page_description | escape }}">
{% endif %}
```

**След:**
```liquid
{%- assign meta_desc = page_description | default: shop.description -%}
{% if meta_desc != blank %}
  <meta name="description" content="{{ meta_desc | escape | truncate: 160 }}">
{% endif %}
```

**Какво прави:**
- Ако `page_description` е попълнен в Admin — използва го
- Ако е празен — използва `shop.description` (от Settings > General > "Store description")
- `| truncate: 160` — гарантира, че не надвишаваме лимита
- Така **никоя страница** не остава без meta description

**Важно:** `shop.description` също трябва да е попълнен! Провери в:
Shopify Admin > **Settings** (долу вляво) > **General** > **"Store description"**

### Бонус: Поправи title формата
В [layout/theme.liquid](layout/theme.liquid) ред 22, title-ът се генерира с `–` (тире) без спейс:
```liquid
{% unless page_title contains shop.name %}&ndash; {{ shop.name }}{% endunless %}
```
Резултат: `About us– Gravity Designs` (липсва спейс преди тирето).

**Поправка** — добави спейс преди `&ndash;`:
```liquid
{% unless page_title contains shop.name %} &ndash; {{ shop.name }}{% endunless %}
```
Резултат: `About us – Gravity Designs`

### Как да провериш
1. Отвори `http://127.0.0.1:9292` > View Page Source (Ctrl+U)
2. Търси `<meta name="description"` — трябва да го има на **всяка** страница
3. Провери дължината — трябва да е под 160 символа
4. Провери `<meta property="og:description"` — трябва да съвпада с meta description
5. Провери `<title>` — трябва да има спейс преди тирето

### Чести грешки
- **Copy-paste на едно и също описание** — ВСЯКА страница трябва да има УНИКАЛЕН текст
- **Описание над 160 символа** — Google го отрязва и показва "..."
- **Описание без ключови думи** — включи: "DTF transfers", "custom stickers", "heat press", "Florida", "Coral Springs"
- **Забравен 404 продукт** — проверявай редовно за счупени линкове с инструмент като Screaming Frog или Ahrefs Site Audit

---

## 2. Structured Data (JSON-LD)
**Файлове за промяна:** [layout/theme.liquid](layout/theme.liquid), нов snippet

### Какъв е проблемът?
Сайтът има `snippets/product-json-ld.liquid` за продуктови страници, **но** няма:
- **LocalBusiness** schema (за Google Maps, местни търсения)
- **Organization** schema (за brand knowledge panel)
- **BreadcrumbList** schema (за breadcrumbs в Google)
- **WebSite** schema (за sitelinks search box)

### Стъпка 1: Създай snippet `snippets/json-ld-global.liquid`

Създай нов файл `snippets/json-ld-global.liquid` със следното съдържание:

```liquid
{%- comment -%}
  Global JSON-LD structured data.
  Included in theme.liquid <head> on every page.
{%- endcomment -%}

{%- comment -%} === Organization + LocalBusiness === {%- endcomment -%}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "@id": "{{ shop.url }}/#organization",
  "name": "{{ shop.name | escape }}",
  "url": "{{ shop.url }}",
  {%- if settings.logo != blank -%}
  "logo": "{{ settings.logo | image_url: width: 600 | prepend: 'https:' }}",
  "image": "{{ settings.logo | image_url: width: 600 | prepend: 'https:' }}",
  {%- endif -%}
  "description": "{{ shop.description | strip_html | escape | truncate: 250 }}",
  "telephone": "+1-954-993-2041",
  "email": "info@gravity-designs.com",
  "address": {
    "@type": "PostalAddress",
    "addressLocality": "Coral Springs",
    "addressRegion": "FL",
    "addressCountry": "US"
  },
  "priceRange": "$$",
  "sameAs": [
    {%- if settings.social_facebook_link != blank -%}"{{ settings.social_facebook_link }}"{%- endif -%}
    {%- if settings.social_instagram_link != blank -%},"{{ settings.social_instagram_link }}"{%- endif -%}
    {%- if settings.social_twitter_link != blank -%},"{{ settings.social_twitter_link }}"{%- endif -%}
  ]
}
</script>

{%- comment -%} === WebSite (enables sitelinks search in Google) === {%- endcomment -%}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "{{ shop.name | escape }}",
  "url": "{{ shop.url }}",
  "potentialAction": {
    "@type": "SearchAction",
    "target": "{{ shop.url }}/search?q={search_term_string}",
    "query-input": "required name=search_term_string"
  }
}
</script>

{%- comment -%} === BreadcrumbList === {%- endcomment -%}
{%- if template.name != 'index' -%}
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {
      "@type": "ListItem",
      "position": 1,
      "name": "Home",
      "item": "{{ shop.url }}"
    }
    {%- if template.name == 'collection' -%}
    ,{
      "@type": "ListItem",
      "position": 2,
      "name": "{{ collection.title | escape }}",
      "item": "{{ shop.url }}{{ collection.url }}"
    }
    {%- elsif template.name == 'product' -%}
    {%- if product.collections.size > 0 -%}
    ,{
      "@type": "ListItem",
      "position": 2,
      "name": "{{ product.collections.first.title | escape }}",
      "item": "{{ shop.url }}{{ product.collections.first.url }}"
    }
    {%- endif -%}
    ,{
      "@type": "ListItem",
      "position": {% if product.collections.size > 0 %}3{% else %}2{% endif %},
      "name": "{{ product.title | escape }}",
      "item": "{{ shop.url }}{{ product.url }}"
    }
    {%- elsif template.name == 'page' -%}
    ,{
      "@type": "ListItem",
      "position": 2,
      "name": "{{ page.title | escape }}",
      "item": "{{ shop.url }}{{ page.url }}"
    }
    {%- elsif template.name == 'blog' -%}
    ,{
      "@type": "ListItem",
      "position": 2,
      "name": "{{ blog.title | escape }}",
      "item": "{{ shop.url }}{{ blog.url }}"
    }
    {%- elsif template.name == 'article' -%}
    ,{
      "@type": "ListItem",
      "position": 2,
      "name": "{{ blog.title | escape }}",
      "item": "{{ shop.url }}{{ blog.url }}"
    }
    ,{
      "@type": "ListItem",
      "position": 3,
      "name": "{{ article.title | escape }}",
      "item": "{{ shop.url }}{{ article.url }}"
    }
    {%- endif -%}
  ]
}
</script>
{%- endif -%}
</script>
```

### Стъпка 2: Включи snippet-а в theme.liquid

Отвори [layout/theme.liquid](layout/theme.liquid) и добави на ред 30 (след `{% include 'social-meta-tags' %}`):

```liquid
{% include 'social-meta-tags' %}
{% include 'json-ld-global' %}
```

### Стъпка 3: Поправи съществуващия Product JSON-LD

Файлът [snippets/product-json-ld.liquid](snippets/product-json-ld.liquid) вече съществува, но има проблеми:
- `"brand"` няма `@type`
- `"price"` може да има грешен формат (money_without_currency може да върне "2.00" или "$2.00")
- Няма `"sku"`, `"description"` е добре

Замени цялото съдържание на `snippets/product-json-ld.liquid`:

```liquid
{% assign current_variant = product.selected_or_first_available_variant %}

<script type="application/ld+json">
{
  "@context": "https://schema.org/",
  "@type": "Product",
  "@id": "{{ shop.url }}{{ product.url }}",
  "url": "{{ shop.url }}{{ product.url }}",
  "name": {{ product.title | json }},
  "image": [
    {%- for image in product.images limit: 5 -%}
      "{{ image | image_url: width: 1200 | prepend: 'https:' }}"{% unless forloop.last %},{% endunless %}
    {%- endfor -%}
  ],
  "description": {{ product.description | strip_html | truncate: 500 | json }},
  "brand": {
    "@type": "Brand",
    "name": {{ product.vendor | json }}
  },
  {%- if current_variant.sku != blank -%}
  "sku": {{ current_variant.sku | json }},
  {%- endif -%}
  "offers": {
    "@type": "Offer",
    "priceCurrency": "{{ shop.currency }}",
    "price": "{{ current_variant.price | divided_by: 100.0 }}",
    "availability": "https://schema.org/{% if current_variant.available %}InStock{% else %}OutOfStock{% endif %}",
    "url": "{{ shop.url }}{{ product.url }}",
    "seller": {
      "@type": "Organization",
      "name": {{ shop.name | json }}
    }
  }
}
</script>
```

**Ключови промени:**
- `"name"` и `"description"` използват `| json` филтъра вместо `| escape` — така специалните символи (кавички, нови редове) се escape-ват правилно за JSON
- `"brand"` вече има `"@type": "Brand"`
- `"image"` е масив с до 5 изображения
- `"availability"` използва `https://` вместо `http://`
- Добавен `"sku"` ако е наличен

### Как да провериш
1. Отвори `http://127.0.0.1:9292` > View Source > търси `application/ld+json`
2. Копирай JSON блоковете и ги провери в: https://search.google.com/test/rich-results
3. Или използвай: https://validator.schema.org/
4. Трябва да видиш **LocalBusiness**, **WebSite**, **BreadcrumbList** на всяка страница
5. На продуктова страница трябва да видиш и **Product**

### Чести грешки
- **Trailing comma в JSON** — ако `sameAs` масивът е празен, ще има проблем. Liquid условията по-горе се грижат за това.
- **Кавички в product title** — `| json` филтърът ги escape-ва автоматично. НИКОГА не използвай `"{{ product.title }}"` без филтър!
- **Price формат** — Shopify пази цените в центове. `divided_by: 100.0` ги конвертира (200 -> 2.0).

---

## 3. Само 3 продукта в каталога (Thin Content)
**Къде:** Shopify Admin (не е проблем на кода)

### Какъв е проблемът?
Google оценява сайтове с малко съдържание по-ниско. 3 продукта = 3 индексируеми продуктови страници. Конкурентите имат десетки/стотици.

### Какво трябва да се направи

#### А. Разшири продуктовия каталог (Shopify Admin)
Gravity Designs предлага DTF transfers, stickers и embroidery. Текущата структура е:
```
- Custom DTF Transfers by Size ($2-$7)
- Custom Stickers and Decals ($30)
- DTF Gang Sheet Builder ($5-$19.70)
```

**Препоръки за нови продукти/варианти:**
1. **DTF Transfers by Category** — раздели на отделни продукти:
   - DTF Transfers for Sports Teams
   - DTF Transfers for Business Logos
   - DTF Transfers for Events & Parties
   - DTF Transfers for Schools & PTA
2. **Custom Apparel** — добави конкретни продукти:
   - Custom T-Shirts
   - Custom Hoodies
   - Custom Caps/Hats
3. **Bundle продукти** — DTF Transfer + Application Kit

#### Б. Създай колекции (Shopify Admin)
1. Отиди в **Products > Collections**
2. Създай колекции:
   - "DTF Heat Transfers"
   - "Custom Stickers & Decals"
   - "Custom Apparel"
   - "For Sports Teams"
   - "For Businesses"
3. Всяка колекция трябва да има:
   - **Title** (с ключова дума)
   - **Description** (мин. 100 думи, с ключови думи)
   - **SEO title** и **meta description**
   - **Image**

#### В. Създай допълнителни страници
1. **FAQ страница** — Shopify Admin > Pages > Add page
   - Заглавие: "Frequently Asked Questions | DTF Transfers & Custom Printing"
   - Съдържание: 10-15 въпроса за DTF процеса, цени, сроковете, грижа за щампите
2. **Pricing/Size Guide** — детайлна таблица с цени по размер
3. **Gallery/Portfolio** — покажи реални проекти

#### Г. Стартирай блог
1. Shopify Admin > Online Store > Blog posts > "Manage blogs" > Add blog
2. Име: "DTF Printing Tips & Guides"
3. Публикувай статии като:
   - "What is DTF Printing? Complete Guide for Beginners"
   - "How to Apply DTF Heat Transfers: Step-by-Step"
   - "Custom Apparel for Sports Teams: Design Tips"
   - "DTF vs Screen Printing: Which is Better?"

**Важно:** Всяка статия мин. 800 думи, с изображения, и вътрешни линкове към продуктите.

### Как да провериш
- Google Search Console > Coverage — следи колко страници са индексирани
- Цел: от 3 на поне 15-20 индексируеми страници в рамките на месец

---

## 4. Филтрите на колекцията са празни
**Файл:** [sections/collection-sidebar.liquid](sections/collection-sidebar.liquid)

### Какъв е проблемът?
На страницата с колекции се показва "There are no collections" и "There are no tags". Това се случва, защото sidebar блоковете не са конфигурирани в Theme Editor.

### Как да го поправиш

#### Стъпка 1: Провери дали sidebar-ът е включен
1. Отиди в **Shopify Admin > Online Store > Themes > Customize**
2. В горния dropdown избери **Collection pages** (не Homepage)
3. В лявата колонка търси секция **"Collection sidebar"** или **"Sidebar"**
4. Ако я няма — натисни **"Add section"** и я добави

#### Стъпка 2: Конфигурирай блоковете в sidebar-а
Вътре в sidebar секцията, добави блокове:

1. **Collections блок:**
   - Натисни "Add block" > "Collections"
   - В полето **"Menu"** избери меню, което съдържа линкове към колекции
   - Ако такова меню не съществува — създай го: Shopify Admin > Online Store > Navigation > Add menu
   - Назови го "collection-sidebar" и добави линкове към колекциите

2. **Tags блок (ако го има):**
   - Tags в Shopify се добавят на самите продукти
   - Отиди в Products > [продукт] > Tags
   - Добави тагове като: "sports", "business", "custom", "vinyl"
   - След това sidebar-ът ще ги покаже автоматично

#### Стъпка 3: Провери в кода (ако sidebar-ът не работи)
Отвори [sections/collection-sidebar.liquid](sections/collection-sidebar.liquid) ред 11-13:
```liquid
{%- capture navigation_content -%}
    {%- assign menu = linklists[block.settings.menu] -%}
```

`block.settings.menu` трябва да съвпада с handle-а на менюто. Ако менюто се казва "Collection Sidebar", handle-ът е `collection-sidebar`.

#### Стъпка 4: Алтернатива — използвай Shopify Storefront Filtering
Ако искаш модерни филтри (по цена, наличност, тип):
1. Shopify Admin > Online Store > Navigation > **"Collection and search filters"**
2. Добави филтри: Price, Availability, Product type
3. Тези работят с Shopify Online Store 2.0 — темата Sofine може да не ги поддържа напълно

### Как да провериш
1. Отвори колекция (напр. `/collections/all`) в dev preview
2. В sidebar-а трябва да се виждат колекциите и таговете
3. Натисни на филтър — URL-ът трябва да се промени (напр. `/collections/all/sports`)
4. Продуктите трябва да се филтрират

---

## 5. Дублиран Ahrefs Analytics скрипт
**Файл:** [layout/theme.liquid](layout/theme.liquid) — редове 6-7

### Какъв е проблемът?
```html
<script src="https://analytics.ahrefs.com/analytics.js" data-key="FVHRFTjrlutHE4VwpcjXLA" async></script>
<script src="https://analytics.ahrefs.com/analytics.js" data-key="mFry0Q1SxDE13RjftgyMkg" async></script>
```

Два пъти се зарежда **един и същ скрипт** с различни API ключове. Това:
- Забавя зареждането на страницата (2 HTTP заявки към external CDN)
- Може да генерира дублирани данни в Ahrefs
- Скриптът е в самото начало на `<head>` — блокира парсинга

### Как да го поправиш

#### Стъпка 1: Разбери кой ключ е правилният
1. Влез в **Ahrefs акаунта** (попитай мениджъра/собственика за достъп)
2. Отиди в Site Audit > Settings > виж кой verification key е активен
3. Или отиди в Ahrefs Dashboard > Site Explorer > проверката за сайта

**Ако не знаеш кой е правилният** — попитай собственика. Не изтривай на random.

#### Стъпка 2: Премахни дублирания скрипт
Отвори [layout/theme.liquid](layout/theme.liquid) и промени редове 6-7.

**Преди:**
```html
<script src="https://analytics.ahrefs.com/analytics.js" data-key="FVHRFTjrlutHE4VwpcjXLA" async></script>
<script src="https://analytics.ahrefs.com/analytics.js" data-key="mFry0Q1SxDE13RjftgyMkg" async></script>
```

**След** (запази само правилния ключ):
```html
<script src="https://analytics.ahrefs.com/analytics.js" data-key="ПРАВИЛНИЯТ_КЛЮЧ_ТУК" async></script>
```

#### Стъпка 3: Премести го по-надолу в `<head>`
За по-добър performance, премести скрипта точно преди `</head>` (ред 47), след Google site verification тага:

```html
  <meta name="google-site-verification" content="WGz5j3O6uZho53wVYE1jaP7BBUkTExOjLzoB1RHu5uo" />
  <script src="https://analytics.ahrefs.com/analytics.js" data-key="ПРАВИЛНИЯТ_КЛЮЧ" async></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" integrity="..." crossorigin="anonymous" />
</head>
```

**Защо?** Analytics скриптовете не са критични за рендеринга. Мета таговете, CSS и JS на темата са по-важни и трябва да са по-нагоре.

### Как да провериш
1. Отвори dev preview > View Source
2. Търси "ahrefs" — трябва да е **само 1 резултат**
3. Отвори DevTools (F12) > Network tab > филтрирай по "ahrefs" — трябва да е 1 заявка
4. Отвори Ahrefs > Site Audit > провери дали verification-ът все още работи

---

---

## 6. Duplicate Product URLs (within: collection) — КРИТИЧНА SEO ГРЕШКА

### Какъв е проблемът?
Темата Sofine използва `{{ product.url | within: collection }}` на **40+ места** в кода. Това генерира дублирани URL адреси за всеки продукт:

```
/products/custom-dtf-gang-sheet                              <- каноничен URL
/collections/dtf-transfers/products/custom-dtf-gang-sheet    <- дублиран URL
```

Google вижда **две страници с едно и също съдържание**. Това:
- Разделя link authority между двата URL-а
- Губи crawl budget
- Може Google да индексира грешния URL
- Създава keyword канибализация

### Мащаб на проблема в нашата тема
Проверка с `grep` показва **40+ употреби** в следните файлове:

| Файл | Брой |
|------|------|
| [snippets/product-grid-item.liquid](snippets/product-grid-item.liquid) | 3 |
| [snippets/product-res-variables.liquid](snippets/product-res-variables.liquid) | 1 (задава `url` променлива, използвана навсякъде) |
| [snippets/product_style.liquid](snippets/product_style.liquid) | 2 |
| [snippets/product_menu.liquid](snippets/product_menu.liquid) | 3 |
| [snippets/product-list-item.liquid](snippets/product-list-item.liquid) | 4 |
| [snippets/product-column.liquid](snippets/product-column.liquid) | 1 |
| [snippets/product-grid-item-masonry.liquid](snippets/product-grid-item-masonry.liquid) | 5 |
| [snippets/product_lookbook_item.liquid](snippets/product_lookbook_item.liquid) | 3 |
| [snippets/product-wishlist.liquid](snippets/product-wishlist.liquid) | 1 |
| [snippets/product-outofstock.liquid](snippets/product-outofstock.liquid) | 1 |
| [snippets/collection-sidebar.liquid](snippets/collection-sidebar.liquid) | 2 |
| [sections/main-product.liquid](sections/main-product.liquid) | 1 |
| [templates/product.ajax-product.liquid](templates/product.ajax-product.liquid) | 3 |
| [templates/product.lookbook.liquid](templates/product.lookbook.liquid) | 4 |
| [templates/product.pr_lazy_load.liquid](templates/product.pr_lazy_load.liquid) | 2 |

### Как да го поправиш

#### Стъпка 1: Поправи централната променлива
Най-важният файл е [snippets/product-res-variables.liquid](snippets/product-res-variables.liquid), ред 19. Тук се задава `url` променливата, която се използва от много snippets.

**Преди:**
```liquid
{%- assign url = product.url | within: collection -%}
```

**След:**
```liquid
{%- assign url = product.url -%}
```

#### Стъпка 2: Замени `within: collection` навсякъде
Във **всеки файл** от таблицата по-горе, замени:

```liquid
{{ product.url | within: collection }}
```
с:
```liquid
{{ product.url }}
```

**Важно:** Правя масова замяна. Можеш да използваш Find & Replace в редактора:
- Find: `product.url | within: collection`
- Replace: `product.url`

#### Стъпка 3: Провери sidebar филтрите (изключение!)
В [sections/collection-sidebar.liquid](sections/collection-sidebar.liquid) ред 141 и [snippets/search-sidebar.liquid](snippets/search-sidebar.liquid) ред 141 има:
```liquid
{%- assign refresh_filter = refresh_tags_string | strip | prepend:'/' | within: collection -%}
```
Това е **различен случай** — тук `within: collection` се използва за филтриране по тагове вътре в колекция. Тази употреба е **правилна** и **НЕ трябва да се променя**, защото филтрирането зависи от collection context.

### Как да провериш
1. Отвори collection страница в dev preview
2. Hover върху продукт — URL-ът трябва да е `/products/product-name`, **НЕ** `/collections/xxx/products/product-name`
3. Щракни на продукт — адресната лента трябва да показва `/products/product-name`
4. Провери в Google Search Console > URL Inspection за стария collection URL — трябва да показва "Not indexed"

### Очакван SEO ефект
- По-добро индексиране на продуктовите страници
- Концентриран link authority
- По-ефективен crawl budget
- Ефект след **4-8 седмици**

---

## 7. SEO архитектура и Collection стратегия

### 7.1 За `/collections/all`
URL `/collections/all` е автоматична Shopify колекция — **не може да се изтрие**.

**Препоръка за Gravity Designs:**
- `/collections/all` да **съществува**, но да **НЕ е в навигацията**
- В менюто линквай директно към конкретни колекции
- НЕ прави redirect от `/collections/all` — остави го като системен fallback

### 7.2 Препоръчителна URL структура
```
Homepage
  /

Collections (Commercial intent keywords)
  /collections/dtf-transfers          <- "dtf transfers"
  /collections/gang-sheets            <- "dtf gang sheet"
  /collections/custom-stickers        <- "custom stickers"

Products
  /products/custom-dtf-gang-sheet
  /products/custom-dtf-transfer-by-size
  /products/custom-vinyl-stickers

SEO Landing Pages (B2B / commercial-informational)
  /pages/wholesale-dtf-transfers
  /pages/bulk-dtf-printing
  /pages/dtf-for-small-business

Blog (Informational keywords)
  /blogs/guides/how-to-apply-dtf-transfer
  /blogs/guides/dtf-vs-screen-printing
  /blogs/guides/best-heat-press-settings
```

### 7.3 Навигация (меню)
**Не правй:**
```
Shop → /collections/all
```

**Правй:**
```
Shop
  ├ DTF Transfers     → /collections/dtf-transfers
  ├ Gang Sheets       → /collections/gang-sheets
  └ Custom Stickers   → /collections/custom-stickers
```

### 7.4 Collection SEO оптимизация (носи +20-50% organic traffic)
Повечето магазини оставят collection страниците като чист product grid. За максимален SEO ефект, всяка collection трябва да има:

```
H1: DTF Transfers

Intro текст (50-100 думи)

[PRODUCT GRID]

SEO текстов блок (200-400 думи)

FAQ секция
```

**Стъпки за добавяне на SEO текст:**
1. Shopify Admin > Products > Collections > [колекция]
2. В полето **"Description"** добави intro текст
3. За SEO блок под продуктите — темата трябва да показва `collection.description` под grid-а (провери template)

**FAQ Schema за collection pages:**
Добави JSON-LD в collection template или като метаполе:

```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "What are DTF transfers?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "DTF (Direct-to-Film) transfers are heat-applied prints that can be pressed onto virtually any fabric color or material."
      }
    },
    {
      "@type": "Question",
      "name": "What temperature do you press DTF transfers?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Most DTF transfers are pressed at 300-320F for 10-15 seconds with medium pressure."
      }
    },
    {
      "@type": "Question",
      "name": "How long do DTF transfers last?",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "Properly applied DTF transfers last 50+ washes without cracking or fading."
      }
    }
  ]
}
```

### 7.5 Internal Linking стратегия (SEO silo)
Authority трябва да тече:

```
Blog article → Collection page → Product pages
```

Пример:
1. Blog: "How to Apply DTF Transfers" линква към `/collections/dtf-transfers`
2. Collection: `/collections/dtf-transfers` линква към продуктите
3. Продуктите линкват обратно към collection-а

**Не правй:**
- Collection за размери (`/collections/2x2-transfers`) — размерите трябва да са product variants
- Collection за цветове — използвай filters
- Твърде много collections с подобни имена (`/collections/dtf`, `/collections/dtf-transfer`, `/collections/dtf-transfers`)

### 7.6 Три SEO грешки с Collections, които 90% от магазините правят

**Грешка 1: Duplicate product URLs**
Вече описана в секция 6 — `within: collection` генерира дублирани URL адреси.

**Грешка 2: Collections без текстово съдържание (Thin Content)**
Google вижда само product grid без контекст — страницата трудно ранква.
Решение: добави SEO текст + FAQ (описано в 7.4).

**Грешка 3: Индексиране на филтри и сортирания**
Shopify генерира URL-и като:
```
/collections/dtf-transfers?sort_by=price-ascending
/collections/dtf-transfers?filter.v.price.gte=10
/collections/dtf-transfers?page=3
```
Google може да индексира стотици вариации на една страница.

**Решение:**
- Canonical тагът (вече е в theme.liquid ред 12-14) трябва да сочи към base URL
- Shopify robots.txt вече блокира `?sort_by=` по подразбиране
- Pagination pages (`?page=2`) трябва да имат canonical към себе си (Shopify го прави автоматично)

### 7.7 Blog стратегия — Topic Clusters
Публикувай 1-2 статии на седмица. Организирай като topic clusters:

```
DTF Printing (pillar topic)
  ├ What is DTF printing
  ├ How to apply DTF transfers
  ├ Best heat press settings for DTF
  ├ DTF vs sublimation
  ├ DTF vs screen printing
  └ DTF transfers care guide
```

Всички статии линкват към `/collections/dtf-transfers`.

---

## Обобщение: Чеклист

### Критични поправки (код)
| # | Задача | Файл | Статус |
|---|--------|------|--------|
| 1 | Съкрати homepage meta description до 155 символа | Shopify Admin > Preferences | [ ] |
| 2 | Добави meta description на Collections | Shopify Admin > Collections | [ ] |
| 3 | Пренапиши About Us meta description | Shopify Admin > Pages | [ ] |
| 4 | Поправи Custom Stickers 404 + добави redirect | Shopify Admin > Navigation > URL Redirects | [ ] |
| 5 | Поправи OG description приоритета | snippets/social-meta-tags.liquid:11 | [ ] |
| 6 | Добави fallback за meta description | layout/theme.liquid:25-27 | [ ] |
| 7 | Поправи title формата (липсващ спейс) | layout/theme.liquid:22 | [ ] |
| 8 | Създай snippets/json-ld-global.liquid | Нов файл | [ ] |
| 9 | Включи json-ld-global в theme.liquid | layout/theme.liquid:30 | [ ] |
| 10 | Обнови product-json-ld.liquid | snippets/product-json-ld.liquid | [ ] |
| 11 | **Замени `within: collection` с `product.url`** | 40+ файла (виж секция 6) | [ ] |
| 12 | Премахни дублирания Ahrefs скрипт | layout/theme.liquid:6-7 | [ ] |

### SEO архитектура (Admin)
| # | Задача | Статус |
|---|--------|--------|
| 13 | Създай колекции: DTF Transfers, Gang Sheets, Custom Stickers | [ ] |
| 14 | Добави SEO текст (200-400 думи) + FAQ на всяка колекция | [ ] |
| 15 | Добави FAQ Schema (JSON-LD) на collection pages | [ ] |
| 16 | Преструктурирай навигацията (Shop dropdown с конкретни collections) | [ ] |
| 17 | Премахни /collections/all от навигацията | [ ] |
| 18 | Добави тагове на продуктите | [ ] |
| 19 | Конфигурирай collection sidebar в Theme Editor | [ ] |

### Content стратегия (средносрочна)
| # | Задача | Статус |
|---|--------|--------|
| 20 | Създай SEO landing pages (wholesale, bulk, for-small-business) | [ ] |
| 21 | Стартирай блог с topic clusters | [ ] |
| 22 | Публикувай 1-2 статии/седмица с internal links | [ ] |
| 23 | Провери всичко с Rich Results Test | [ ] |

## Тестване след всички промени

### 1. Visual check
- Отвори всяка страница в dev preview
- Провери дали нищо не е счупено визуално
- **Hover върху продукти** — URL трябва да е `/products/...` не `/collections/.../products/...`

### 2. SEO validation
- **View Source** на всяка страница — провери meta tags и JSON-LD
- **Rich Results Test:** https://search.google.com/test/rich-results
- **Schema Validator:** https://validator.schema.org/

### 3. Performance check
- **DevTools > Network** — провери, че няма дублирани скриптове
- **DevTools > Lighthouse** — пусни SEO одит (трябва да е 90+)

### 4. Push
Когато всичко е ОК:
```bash
shopify theme push --unpublished --store d20e1f.myshopify.com
```
Това качва темата като **unpublished** — не засяга живия сайт. Тествай в preview линка от Shopify Admin преди да я публикуваш.

---

*Ръководство създадено на 2026-03-05. Обновено с SEO architecture best practices.*

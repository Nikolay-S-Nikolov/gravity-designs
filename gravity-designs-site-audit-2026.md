# Gravity Designs — Одит на сайта
**Дата:** 19.02.2026  
**URL:** https://gravity-designs.com  
**Платформа:** Shopify (тема: Gravity_Design_Trial1)

---

## 📋 Резюме

По време на пълна проверка на сайта бяха открити **9 проблема** — от критични функционални бъгове, блокиращи продажбите, до визуални и съдържателни слабости. По-долу са наредени по приоритет.

---

## 🔴 КРИТИЧЕН ПРИОРИТЕТ

### 1. Всички продукти показват Qty=0 и бутон "Add to cart Out Stock"

**Страниците засегнати:**
- /products/custom-stickers-and-decals
- /products/dtf-full-color-heat-transfer-sheets
- /products/dtf-gang-sheet-builder

**Симптоми:**
- Полето "Qty" се зарежда с **0** вместо 1
- `data-max="0"` в HTML кода — системата смята всички варианти за недостъпни
- Sticky бутонът в долната лента изписва буквално **"Add to cart Out Stock"**

**Защо е критично:**  
Клиент, дошъл да поръча, вижда Qty=0 и бутон с надпис "Out Stock" и най-вероятно ще се откаже, смятайки продуктите за неналични. Това е директна загуба на приходи.

> **Бележка:** Add to Cart функционира — при клик на бутона се появява валидация "Please upload a file", т.е. продуктите могат да се купят. Проблемът е само в initial стойността и sticky бутона.

**Решение:**  
В Shopify Admin → Products → за всеки продукт:
- Ако продуктите са custom услуга (без физически наличности): изключи inventory tracking напълно
- Или постави `inventory_policy: continue` (Allow customers to purchase when out of stock)
- Провери темата — QTY counter трябва да стартира от 1 (data-min=1, data-max трябва да е > 0 или неограничен)

---

### 2. Category Tiles на Homepage са некликаеми (8 плочки)

**Засегнати елементи:**

*Секция 1 — "LOOKING FOR CUSTOM PRINTED T-SHIRTS..."*
- CUSTOM DTF TRANSFERS
- EMBROIDERY
- CUSTOM STICKERS & DECALS
- CUSTOM APPAREL

*Секция 2 — Industries*
- SPORTS TEAMS
- BUSINESSES
- EVENTS
- SCHOOLS & PTA

**Симптом:**  
Нито една от плочките **няма `<a href>`**, onclick, или data атрибут. Потребителят кликва върху снимката или текста — нищо не се случва.

**Защо е критично:**  
Основната навигация от началната страница към категориите е счупена. Посетителят не може да продължи по интуитивния начин.

**Решение:**  
Shopify Admin → Online Store → Themes → Customize → всяка "Collection Featured" секция → задай URL линк към съответния продукт или колекция за всяка плочка.

---

### 3. "Embroidery" колекция дава 404 грешка

**URL:** https://gravity-designs.com/collections/embroidery → **404 Not Found**

**Проблемът:**  
Колекцията "Embroidery" е показана като категория на homepage, но не съществува в Shopify.

**Защо е критично:**  
Ако некликаемите плочки бъдат поправени (проблем #2) и Embroidery получи линк, клиентите ще стигнат до 404 страница.

**Решение:**  
- Създай колекция "Embroidery" в Shopify Admin → Products → Collections, или
- Насочи плочката към съществуващ продукт/страница

---

### 4. "Custom Apparel" колекция е напълно празна

**URL:** https://gravity-designs.com/collections/custom-apparel → "Sorry, there are no products in this collection."

**Проблемът:**  
Колекцията съществува, но не съдържа нито един продукт.

**Защо е критично:**  
"Custom Apparel" е една от 4-те основни категории на сайта. Ако клиент стигне до нея, ще намери само празна страница — лошо впечатление и пропусната продажба.

**Решение:**  
Shopify Admin → Products → Collections → Custom Apparel → добави продукти или условия за автоматично включване.

---

## 🟠 ВИСОК ПРИОРИТЕТ

### 5. Hero Banner текстовете се отрязват вдясно

**Засегнати слайдове:**
- "PREMIUM DTF TRANSFERS" → изрязано като "PREMIUM DTF TRANSFE"
- "CUSTOM STICKERS & DECALS" → изрязано като "CUSTOM STICKERS & DECA"
- "CUSTOM APPAREL" → видимо само при определена ширина

**Техническа причина:**  
Слайдер секцията (.laber_slidershow_...) има `overflow: visible` вместо `overflow: hidden`. Текстовите блокове се позиционират абсолютно и излизат от контейнера при резолюции под ~1440px.

**Защо е критично:**  
Hero секцията е първото нещо, което потребителят вижда. Отрязаният текст създава впечатление за непрофесионален, незавършен сайт.

**Решение (CSS):**
```css
/* В theme CSS или Custom CSS */
.shopify-section.slideshow,
[class*="laber_slidershow"] {
  overflow: hidden !important;
  max-width: 100%;
}
.slick-slide img {
  object-fit: cover;
  width: 100%;
}
```

---

### 6. "Upload a Gang Sheet" в навигацията сочи грешна страница

**Текущ линк:** /products/dtf-full-color-heat-transfer-sheets  
**Правилен линк трябва да е:** /products/dtf-gang-sheet-builder

**Проблемът:**  
В хедъра, под "DTF Heat Transfers" dropdown, линкът "Upload a Gang Sheet" води към "Order Transfers by Size" страницата вместо Gang Sheet Builder-а.

**Решение:**  
Shopify Admin → Online Store → Navigation → Main menu → DTF Heat Transfers → Upload a Gang Sheet → промени URL на /products/dtf-gang-sheet-builder

---

## 🟡 СРЕДЕН ПРИОРИТЕТ

### 7. About Us страницата е почти празна

**URL:** https://gravity-designs.com/pages/about-us

**Проблемът:**  
Страницата съдържа само 3 кратки параграфа с обща информация. Липсват:
- Снимки на екипа или работилницата
- История на компанията
- Сертификати или награди
- Защо да изберат Gravity Designs

**Препоръка:**  
Добави визуално съдържание — снимки, timeline на компанията, фотографии от производството. About страницата изгражда доверие и помага за конверсии.

---

### 8. Footer е минималистичен

**Текущо съдържание:**
- Contact info (телефон, имейл, адрес)
- Useful links (Refund Policy, Terms of service, Contact Information)

**Липсва:**
- Социални мрежи (Instagram, Facebook)
- Работно време
- Лого
- Copyright текст (© 2026 Gravity Designs)
- Приети начини на плащане

**Препоръка:**  
Обогати footer-а в Shopify editor → Footer style 1.

---

### 9. Contact страница — Google Maps зарежда бавно

**URL:** https://gravity-designs.com/pages/contact

**Проблемът:**  
Google Maps iframe (508px висок) понякога се показва като сив блок при първо зареждане. Адресът в картата (2161 NW 85th Ln) не съвпада с footer-а (само "Coral Springs, Florida 33071").

**Препоръка:**  
- Унифицирай адреса навсякъде
- Провери дали Maps Embed API ключът е валиден и активен в Google Cloud Console

---

## 📊 Обобщена таблица

| # | Проблем | Приоритет | Страница |
|---|---------|-----------|---------|
| 1 | Qty=0 и "Out Stock" бутон на всички продукти | 🔴 Критичен | Всички продуктови стр. |
| 2 | 8 category tiles некликаеми | 🔴 Критичен | Homepage |
| 3 | Embroidery колекция → 404 | 🔴 Критичен | /collections/embroidery |
| 4 | Custom Apparel колекция → празна | 🔴 Критичен | /collections/custom-apparel |
| 5 | Hero Banner текст отрязан | 🟠 Висок | Homepage |
| 6 | "Upload a Gang Sheet" → грешен URL | 🟠 Висок | Header nav |
| 7 | About Us — почти без съдържание | 🟡 Среден | /pages/about-us |
| 8 | Footer минималистичен | 🟡 Среден | Всички стр. |
| 9 | Google Maps бавно/непоследователен адрес | 🟡 Среден | /pages/contact |

---

## ✅ Препоръчителен ред на оправяне

1. **Веднага:** Оправи Qty/inventory проблема → директно влияе на продажбите
2. **Веднага:** Добави линкове към 8-те category tiles → основна навигация
3. **Тази седмица:** Оправи Embroidery 404 и добави продукти в Custom Apparel
4. **Тази седмица:** Поправи "Upload a Gang Sheet" URL в навигацията
5. **Следващата седмица:** Оправи Hero Banner overflow CSS
6. **По-нататък:** Обогати About Us и Footer

---

*Одитът е извършен на 19.02.2026 чрез пълна проверка на живия сайт и Shopify editor.*

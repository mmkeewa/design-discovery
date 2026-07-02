# source-urls.md

URL-шаблоны и инструкции для Claude in Chrome по каждому источнику. Все три слоя — через браузер.

## Подстановки

- `{flow}` — тип флоу из brief-parser (например, `onboarding`, `paywall`)
- `{industry}` — индустрия
- `{component}` — один из компонентов (для precision-поиска)
- `{tone}` — визуальный тон
- `{platform}` — платформа

---

# Layer 1 — Patterns

## Mobbin

**Приоритетный путь — Mobbin MCP** (если подключён на Шаге 0). Прямой API: возвращает структурированные метаданные + inline-превью, без парсинга страниц и без paywall на превью.

Три инструмента — выбирай по тому, что ищешь:
- `search_screens` — отдельные экраны (login, checkout, empty state). Параметры: `query` (обязателен), `platform` (обязателен), `mode` (по умолчанию `deep` — оставляй его для нюансных запросов), `limit` (до 30).
- `search_flows` — многошаговые флоу (onboarding, checkout end-to-end). Параметры: `query` (обязателен), `platform` (обязателен), `limit` (до 10).
- `search_sections` — секции сайтов (pricing, hero, footer). Параметры: `query` (обязателен). **Без `platform`** — это всегда web.

**Как формировать `query` — критично:**
- Одна фраза на естественном языке, описывающая ОДИН экран/флоу. Например: `"fintech onboarding with identity verification steps"`, `"checkout screen with Apple Pay and promo code field"`.
- ❌ НЕ склеивай ключи через `+` (`{flow}+{industry}+{platform}`) — это ровно тот антипаттерн, от которого тул предупреждает. Никаких списков ключей, отрицаний («without ads»), размытых слов («modern», «clean»).
- Индустрию вплетай в фразу словами (`"banking app onboarding"`), а не отдельным ключом.
- Конкретное приложение — тоже словами в query (`"Revolut onboarding"`), чтобы отфильтровать по нему.

**`platform` — отдельный параметр, enum `ios` | `web`.** Никакого `android` нет. НЕ клади платформу в `query`. Если бриф про мобильный — `ios`; если про веб/десктоп — `web`; если неясно — сделай два вызова.

**Что собрать с каждого результата:** название приложения, название экрана/флоу, **`mobbin_url`** (именно это поле — канонический линк, его цитируем как markdown-ссылку), inline-превью. Изучай сами картинки, а не только метаданные.

---

**Фолбэк — Claude in Chrome** (если Mobbin MCP не подключён):

**URL:**
```
https://mobbin.com/search/screens?keyword={flow}+{industry}
```

1. `navigate` на URL
2. `get_page_text` — извлеки список превью карточек
3. Если есть фильтры (платформа, индустрия) — `find` соответствующие чекбоксы, `computer` кликни их
4. Скролл вниз 2-3 раза, чтобы загрузить больше результатов
5. Для каждой карточки извлеки: название приложения, название экрана, URL, превью thumbnail
6. **Что брать:** базовая информация с listing-страницы. Не пытайся открывать детали — там paywall

**Лимиты Mobbin:**
- Free tier показывает превью без полных flow
- Скриншоты thumbnail видны без подписки
- Полные видео и архитектура флоу — paywall ($24/мес)

Skill собирает с listing — этого достаточно для первичной курации. Если пользователь хочет глубже — даёт ссылку на конкретный flow.

**Что вернуть:** 5-8 примеров от **разных компаний**. Не 3 разных flow Revolut.

---

## PageFlows

**URL:**
```
https://pageflows.com/?search={flow}+{industry}
```

**Инструкция:**
1. `navigate` на URL
2. `get_page_text` — список flow-карточек
3. Каждая карточка содержит: компания, название flow, длительность видео, превью
4. Скролл для подгрузки

**Лимиты PageFlows:**
- Превью бесплатно
- Полные видео — подписка ($25/мес)
- Описания флоу часто видны и без подписки

**Что вернуть:** 2-4 примера. Не больше — это длинные видео.

---

## Benchmarkee

**URL:**
```
https://benchmarkee.ru/search?q={flow}+across+apps
```

**Инструкция:**
1. `navigate` на URL
2. Benchmarkee показывает side-by-side сравнения 5+ приложений по одному и тому же экрану
3. `get_page_text` — извлеки доступные benchmark'и
4. Если есть конкретный benchmark под бриф — открой его, извлеки скриншоты всех компаний

**Особенность:** Benchmarkee хорош именно для "как 7 разных банков делают transfer flow". Уникальный угол.

**Что вернуть:** 1-2 benchmark'а, по одному взять 3-4 ключевые компании из сравнения.

---

# Layer 2 — Visual

## Awwwards

**URL-шаблоны:**
```
https://www.awwwards.com/inspiration_search/?text={tone}+{industry}
https://www.awwwards.com/websites/{tone}/
```

`{tone}` маппится на Awwwards-категории:
- `minimal` → `/websites/minimal/`
- `editorial` → `/websites/typography/`
- `brutalist` → `/websites/brutalism/`
- `motivational` → `/websites/colorful/` или `/websites/big-typography/`
- `data-dense` → не используй Awwwards, используй scrn.gallery
- `corporate` → не используй Awwwards, используй Behance

**Инструкция:**
1. `navigate`
2. `get_page_text` — извлеки галерею
3. Для каждой работы: автор/студия, название проекта, rating, thumbnail
4. **Фильтр:** rating > 6.5

**Что НЕ брать:**
- B2B-сайты, если бриф про мобильный продукт
- Студийные портфолио, если бриф про продуктовый дизайн

**Что вернуть:** 3-4 ссылки + одна строка "что взять" (типографика / композиция / motion / цвет).

---

## Recent (бывший Godly)

Godly ребрендился в **Recent** — `godly.website` редиректит на `recent.design`. Структура другая: разделы `/websites`, `/app-store-screenshots`, `/app-icons` + фильтр-теги сверху (Web, Interface, Branding, Product, Typography, Motion, Illustration, 3D, Editorial). У каждой работы — permalink `recent.design/i/{slug}` (это и берём как ссылку) + исходный пост в X.

**URL:**
```
https://recent.design/            # вся лента, фильтруй тегами
https://recent.design/app-store-screenshots   # мобильные экраны — самое ценное для app-брифов
```

Маппинг тонов на теги/разделы Recent:
- `editorial` → тег `Typography` + `Editorial`
- `motivational` / `motion` → тег `Motion`
- app-скриншоты (любой мобильный бриф) → раздел `/app-store-screenshots`
- `minimal` / `dark` → тег `Interface`, курируй по превью

**Инструкция:**
1. `navigate`. При первом заходе всплывает модалка подписки — закрой (`Escape`).
2. Кликни нужный тег/раздел, скролль (превью ленивые — дай 1-2 c после скролла).
3. `read_page` (filter `interactive`) — собери permalink'и `/i/{slug}` для понравившихся превью. `get_page_text` тут почти пустой (сайт image-heavy), опирайся на скриншот + ссылки.

**Что вернуть:** 3-4 ссылки. Особенно ценны для motion-моментов и премиум app-эстетики.

---

## Screen Gallery (scrn.gallery)

Домен — **`scrn.gallery`** (не `screen.gallery` — тот мёртв). Галерея мобильных app-скриншотов.

**URL:**
```
https://scrn.gallery/apps
```

**Инструкция:**
1. `navigate` на `/apps`.
2. Отфильтруй/найди приложения, близкие к брифу (по индустрии/типу флоу), открой нужное — внутри полный набор экранов приложения.
3. `get_page_text` + скриншот; собери: название приложения, экран, ссылку, превью.

**Что вернуть:** 3-5 мобильных скриншотов. Особенно ценны для iOS-native feel.

---

## FPS60

**URL:**
```
https://fps60.design/
```

Нет фильтров — просмотри последние 20-30 работ.

**Инструкция:**
1. `navigate`
2. `get_page_text` + скролл
3. Каждая работа: автор, видео-превью, описание момента

**Фильтр по релевантности:** выбирай только те motion-моменты, что применимы к компонентам из брифа:
- `wizard` → step transitions, progress animations
- `paywall` → CTA states, plan-switch animations
- `empty-state` → empty illustration animations
- `form` → inline validation, success states
- `dashboard` → data-load animations, chart transitions

**Что вернуть:** 2-3 motion-момента. Помечай: "смотреть на N секунде".

---

# Layer 3 — Mood

## Pinterest

**URL-шаблон:**
```
https://www.pinterest.com/search/pins/?q={tone}+{industry}+app+ui+design
```

**Инструкция:**
1. **Сначала проверь авторизацию.** `navigate` на `pinterest.com`, посмотри через `get_page_text`, видно ли "Sign up" или личный feed
2. Если не залогинен — **honest fail**. Скажи пользователю: "Pinterest требует залогиненного браузера. Перехожу на Behance." Не пытайся обойти
3. Если залогинен — `navigate` на search URL, `get_page_text`
4. Извлеки пины — каждый имеет URL и превью

**Что вернуть:** 4-6 пинов **без описания** (это mood, не функциональный layer).

---

## Dribbble

**URL-шаблон:**
```
https://dribbble.com/search/shots?q={tone}+{flow}&f=tags
```

**Инструкция:**
1. `navigate` на search URL
2. Dribbble частично доступен без login для shots
3. `get_page_text` — список shots
4. Фильтр: минимум 100 likes, дата — последние 12 месяцев

**Что вернуть:** 3-5 shots + автор.

**Hard rule:** не брать с Dribbble UX-паттерны. Только концепты, иллюстрации, типографика, mood.

---

## Behance

**URL-шаблон:**
```
https://www.behance.net/search/projects?search={tone}+{flow}+{industry}&field=ui-ux
```

**Жёсткие фильтры:**
- `field=ui-ux` обязательно (иначе придёт фотография и иллюстрация)
- Сортировка `recommended`, не `featured`

**Инструкция:**
1. `navigate`
2. `get_page_text`
3. Каждый проект: автор, название, превью, likes

**Что вернуть:** 3-5 проектов. Behance — главный fallback, если Pinterest/Dribbble недоступны.

---

## UI8

**URL-шаблон:**
```
https://ui8.net/search?q={tone}+{flow}
```

**Инструкция:**
1. `navigate`
2. `get_page_text` — список kit'ов и шаблонов
3. Каждый: название, автор, превью

**Особое использование UI8:**
- НЕ для копирования
- Используется как **mood-источник**: посмотреть, как продают дизайн в этой нише, как авторы упаковывают kit'ы, какие визуальные коды
- Один kit = один pin для настроения, не более

**Что вернуть:** 2-3 kit'а + ссылка.

---

# Параллельный запуск внутри одного слоя

В рамках **одного слоя** можно запускать источники через `browser_batch` параллельно, это сильно ускоряет. Между слоями — последовательно, с паузой на user-выбор.

Пример параллельного запуска Layer 1:

```
[browser_batch]
  - navigate Mobbin URL
  - navigate PageFlows URL
  - navigate Benchmarkee URL
```

После того как все 3 страницы загружены — `get_page_text` для каждой и сборка списка.

---

# Sanity check перед показом пользователю

Перед тем как показать пользователю результаты слоя:

1. **Минимум 3 разных источника отработали.** Если часть вернула пустоту — скажи об этом пользователю явно: "Mobbin и Behance вернули пустоту — возможно, бриф слишком узкий"
2. **Дедупликация по URL** в рамках слоя
3. **Дедупликация по компании** (Layer 1) — максимум 2 примера от одной компании
4. **Минимум 8 уникальных позиций для показа.** Если меньше — расширь запрос или скажи пользователю, что результатов мало

**Особый случай Layer 1 — почти пустая выдача.** Если во всём Layer 1 набралось **меньше 3 уникальных позиций** (источники за подпиской/логином, нет доступа), не застревай: сообщи причину и **автоматически перейди на Layer 2** — см. SKILL.md, Шаг 2, блок «Если Layer 1 ничего не нашёл». Layer 2 (Awwwards / Recent / scrn.gallery / FPS60) не требует этих подписок.

Если sanity check падает — не молчи. Расскажи пользователю, что произошло.

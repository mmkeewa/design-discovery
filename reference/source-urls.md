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

**URL:**
```
https://mobbin.com/search/screens?keyword={flow}+{industry}
```

**Инструкция для Claude in Chrome:**
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
- `data-dense` → не используй Awwwards, используй Screen Gallery
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

## Godly

**URL-шаблон:**
```
https://godly.website/?search={tone}
```

Маппинг тонов на Godly-теги:
- `brutalist` → `Brutalist`
- `editorial` → `Big Type` + `Typography`
- `motivational` → `Animated` + `Big Type`
- `glassmorphism` → `Glassmorphism`
- `playful` → `3D` + `Animated`

**Инструкция:**
1. `navigate`
2. `get_page_text`
3. Каждая работа: название, превью, теги
4. Скролл для больше результатов

**Что вернуть:** 3-4 ссылки. Особенно ценны для motion-моментов.

---

## Screen Gallery

**URL-шаблон:**
```
https://screen.gallery/category/{flow}/
```

Маппинг flow на категории Screen Gallery:
- `onboarding`, `tutorial` → `/category/onboarding/`
- `paywall`, `checkout` → `/category/paywall/`
- `settings`, `profile` → `/category/settings/`
- `empty-state` → `/category/empty/`
- `dashboard`, `analytics` → `/category/dashboard/`
- `feed`, `list` → `/category/feed/`

**Инструкция:**
1. `navigate`
2. `get_page_text` — мобильные скриншоты
3. Каждый: название приложения, скриншот, опциональное описание

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
  - navigate Liba URL
  - navigate Benchmarkee URL
```

После того как все 4 страницы загружены — `get_page_text` для каждой и сборка списка.

---

# Sanity check перед показом пользователю

Перед тем как показать пользователю результаты слоя:

1. **Минимум 3 разных источника отработали.** Если из 4 двое вернули пустоту — скажи об этом пользователю явно: "Mobbin и Behance вернули пустоту — возможно, бриф слишком узкий"
2. **Дедупликация по URL** в рамках слоя
3. **Дедупликация по компании** (Layer 1) — максимум 2 примера от одной компании
4. **Минимум 8 уникальных позиций для показа.** Если меньше — расширь запрос или скажи пользователю, что результатов мало

Если sanity check падает — не молчи. Расскажи пользователю, что произошло.

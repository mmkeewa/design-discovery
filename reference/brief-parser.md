# brief-parser.md

Извлечение структуры из брифа пользователя.

## Что приходит на вход

Пользователь даёт бриф **с компонентами**. Например:
> "Делаю онбординг для финтех-приложения. ЦА — gen-z. Компоненты: 4-шаговый wizard с прогресс-баром, биометрика на последнем шаге, motivational копирайтинг."

Или короче:
> "Paywall для медитативного приложения. Карточки тарифов, free trial, отзывы."

Skill извлекает пять категорий + список компонентов.

## Пять категорий

### 1. Тип экрана / флоу

Один из:

| Категория | Подтипы |
|---|---|
| Onboarding | first-launch, account-setup, tutorial, permission-priming |
| Auth | login, signup, password-reset, 2FA, biometric |
| Empty states | first-use, no-data, error, success |
| Settings | profile, preferences, security, notifications |
| Lists / feeds | infinite-scroll, paginated, grouped, filtered |
| Detail screens | product, post, profile, transaction |
| Forms | wizard, single-page, conditional, validation |
| Paywalls | hard, soft, free-trial, comparison |
| Search | global, in-context, filters |
| Navigation | tab-bar, drawer, breadcrumbs |
| Notifications | push, in-app, banner, toast |
| Dashboard | analytics, control-panel, status-overview |
| Checkout | cart, payment, confirmation |

Если не подходит — `custom: [описание]` и спроси пользователя пример похожего.

### 2. Индустрия

`fintech` `banking` `crypto` `edtech` `healthtech` `mental-health` `fitness` `productivity` `social` `dating` `gaming` `streaming` `e-commerce` `marketplace` `b2b-saas` `dev-tools` `creator-tools` `travel` `food-delivery` `real-estate` `automotive` `government` `non-profit`

Если редкая ниша — найди **ближайшую большую категорию** и добавь нишу уточнением. Пример: "сервис для подбора грумеров для собак" → `marketplace + niche services`.

### 3. Компоненты (новое — главная фишка этого skill)

**Это самая важная категория.** Пользователь явно перечисляет, какие UI-блоки будут — skill использует это для precision-поиска в Layer 1.

Парсинг идёт из явного списка пользователя, не выводится из типа экрана.

Примеры:
- "4-шаговый wizard с прогресс-баром" → `wizard, progress-indicator`
- "карточки тарифов" → `pricing-cards, comparison-grid`
- "biometric на последнем шаге" → `biometric-prompt, face-id`
- "отзывы пользователей" → `testimonials, social-proof`
- "переключатель валют" → `currency-picker, segmented-control`
- "график доходности" → `chart, data-viz`

Если пользователь не упомянул компоненты — **запроси их явно**:
> "Какие UI-блоки точно будут? Например: wizard, карточки, форма, график, переключатели."

Не выводи компоненты из типа экрана без подтверждения. Это снижает точность поиска.

### 4. Визуальный тон

| Основные | Когда выводится автоматически |
|---|---|
| `minimal` | если бриф нейтральный, без сильных триггеров |
| `editorial` | если упомянуты "серьёзный", "trustworthy", "premium" |
| `brutalist` | если упомянуто "raw", "experimental", "anti-design" |
| `glassmorphism` | если упомянуто "iOS-native feel" |
| `playful` | если ЦА — дети или teens |
| `motivational` | если упомянуто "gen-z", "energy", "ambitious" |
| `data-dense` | для b2b-saas, dev-tools, dashboards |
| `corporate` | для banking, b2b-enterprise |

Если тон не выводится — **спроси пользователя**:
> "Какой визуальный тон ближе? Минимализм / editorial / motivational / data-dense / brutalist / другое?"

### 5. Платформа

`iOS` `Android` `iPadOS` `web-desktop` `web-mobile` `responsive` `watchOS` `macOS`

Если в брифе не указана — **обязательно спроси**. Без платформы Layer 2 промахнётся.

---

## Output для подтверждения

После парсинга показать пользователю:

```
Поняла бриф как:
- Тип: [категория] / [подтип]
- Индустрия: [категория]
- Компоненты: [явный список от пользователя]
- Тон: [основной] (предполагаю / явно указан)
- Платформа: [список]

Правильно? Поправь, что не так, и запускаю Layer 1.
```

**Hard rule:** не запускай поиск без подтверждения.

---

## Если бриф слишком короткий

Если извлекается меньше 3 из 5 категорий — **уточняющие вопросы**, максимум 2 за один круг:

```
Из брифа извлекла:
- Тип: [есть]
- Индустрия: [есть]

Не хватает данных. Уточни:
1. Платформа — iOS / Android / веб?
2. Какие компоненты точно будут? (wizard, карточки, форма, график…)
```

Не больше двух вопросов за раз — пользователь устаёт.

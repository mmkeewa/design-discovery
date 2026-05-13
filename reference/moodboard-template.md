# moodboard-template.md

Финальный markdown — собирается **только из пользовательских выборов**.

## Имя файла

`outputs/{brief-slug}-moodboard.md`

`{brief-slug}` — kebab-case из первой части брифа. Примеры:
- "Онбординг для финтех gen-z" → `onboarding-fintech-genz`
- "Paywall для медитации" → `paywall-meditation`

## Структура

```markdown
# Moodboard: {Brief title}

Создан: {ISO date}
Бриф пользователя: «{original brief verbatim}»

---

## Distilled brief

- **Тип:** {flow}
- **Индустрия:** {industry}
- **Компоненты:** {explicit list from user}
- **Тон:** {tone}
- **Платформа:** {platform}

---

## Layer 1 — Patterns

{N выбранных пользователем} паттернов:

### [Company / app name] — [Screen / flow title]
🔗 [URL]
📌 **Взять:** [одна строка из layer 1 описания]

[Повторить для каждой выбранной ссылки]

---

## Layer 2 — Visual

{N выбранных пользователем} визуалов:

### [Project / site title]
🔗 [URL]
🖼️ ![preview]({image_url})
📌 **Взять:** [типографика / композиция / motion / цвет / микро-взаимодействие]

[Повторить для каждой выбранной ссылки]

---

## Layer 3 — Mood

{Если запускали — N выбранных пользователем}

### [Title or author]
🔗 [URL]
🖼️ ![preview]({image_url})

[Повторить без описаний — это mood]

---

## Synthesis

{Synthesis генерируется на основе выборов пользователя, не всего пула.}

Структура synthesis:

**Главное направление:**
{одна-две строки про то, куда тяготеют выборы пользователя — какой главный приём, какой основной тон, какое настроение}

**Конкретные приёмы взять:**
- {приём 1 из выборов Layer 1}
- {приём 2 из выборов Layer 2}
- {приём 3, если есть Layer 3}

**Что НЕ брать (видно из непопаданий):**
- {anti-pattern, который пользователь явно НЕ выбрал, хотя был в выдаче}

---

## Что дальше

- [ ] Открыть top-3 ссылки из Layer 1, сделать заметки по step-counts и copywriting
- [ ] Screenshot выборы из Layer 2, сложить в Figma как референс-полка
- [ ] Через 24 часа пересмотреть мудборд — что всё ещё ощущается правильным
- [ ] Запустить figma-generate-design с этим мудбордом как референсом

---

## Re-run

Бриф можно перезапустить с теми же параметрами или скорректировать:

```
design-discovery "{original brief}"
```

Или передай мудборд в figma-generate-design:

```
figma-generate-design with brief: {brief-slug}-moodboard.md
```
```

---

## Hard rules для финальной сборки

1. **Включаем только выбранные пользователем** ссылки. Если пользователь сказал "все" — берём все, но в synthesis пишем "пользователь не курировал, направление неоднозначное"
2. **Synthesis должен опираться на pattern в выборах**, а не на полный pool. Если из 14 Layer 1 пользователь выбрал 3 — анализируем общее в этих 3
3. **Anti-pattern блок** заполняется только если есть явные непопадания. Если в Layer 2 было 12, и пользователь выбрал 7, оставшиеся 5 — это не обязательно anti-patterns, может быть просто "не сейчас". Не натягивай sov'ы на глобусы
4. **"Что дальше" чек-лист обязателен** — это переход от research к action
5. **Re-run блок обязателен** — пользователь должен знать, как запустить снова

---

## Пример заполненного мудборда

```markdown
# Moodboard: Onboarding for Fintech (gen-z)

Создан: 2026-05-13
Бриф пользователя: «Делаю онбординг для финтех-приложения. ЦА — gen-z. Компоненты: 4-шаговый wizard с прогресс-баром, биометрика на последнем шаге, motivational копирайтинг.»

---

## Distilled brief

- **Тип:** onboarding / first-launch
- **Индустрия:** fintech
- **Компоненты:** wizard, progress-indicator, biometric-prompt, motivational copywriting
- **Тон:** motivational + gen-z
- **Платформа:** iOS

---

## Layer 1 — Patterns

4 выбранных паттерна:

### Revolut — Onboarding 2024
🔗 https://mobbin.com/flows/revolut-onboarding-2024
📌 **Взять:** 5-шаговый wizard, прогресс-бар сверху, один CTA внизу

### Step — Teen fintech onboarding
🔗 https://pageflows.com/flows/step-onboarding
📌 **Взять:** Разговорный copywriting, "Hey, what do you want to call your money?"

### N26 — Wizard component
🔗 https://liba.app/components/wizard/n26
📌 **Взять:** Сегментированный прогресс-бар с подписями шагов

### Benchmarkee — 7 fintech apps onboarding
🔗 https://benchmarkee.com/fintech-onboarding-2024
📌 **Взять:** Средняя длина 4 шага, 5 у premium-tier, 3 у challenger-банков

---

## Layer 2 — Visual

3 выбранных визуала:

### Arc browser onboarding
🔗 https://awwwards.com/sites/arc-browser
🖼️ ![preview](...)
📌 **Взять:** Motion — slide+scale на step transitions

### Calm app screenshot pack
🔗 https://screen.gallery/category/onboarding/calm
🖼️ ![preview](...)
📌 **Взять:** Один тёплый accent-цвет (коралл) + neutral palette

### iOS 18 Wallet add card animation
🔗 https://fps60.design/wallet-add-card
📌 **Взять:** Motion при подтверждении — карта прыгает в кошелёк на 0:02

---

## Layer 3 — Mood

2 выбранных:

### Shot by @balkangraph
🔗 https://dribbble.com/shots/...
🖼️ ![preview](...)

### Kit "Fintech Mobile UI 2024" (UI8)
🔗 https://ui8.net/...
🖼️ ![preview](...)

---

## Synthesis

**Главное направление:**
Editorial-motivational. Display typography как hero каждого шага, форма ввода занимает меньше 30% экрана. Один тёплый accent-цвет, motion как часть step-transition.

**Конкретные приёмы взять:**
- Wizard с сегментированным прогресс-баром (N26)
- Разговорный copywriting в режиме повелительного (Step)
- Slide+scale motion на step transitions (Arc)
- Один accent-цвет + neutral palette (Calm)
- Биометрика в отдельном контекстном экране, не в форме (выведено из выборов)

**Что НЕ брать:**
- Иллюстрации персонажей (Greenlight был в выдаче, не выбран — слишком "детский")
- Glassmorphism kit'ы (был в UI8, не выбран — не подходит ЦА)

---

## Что дальше

- [ ] Открыть Revolut, Step и N26 onboarding, записать step-counts
- [ ] Screenshot Arc motion и Calm color-palette в Figma
- [ ] Через 24 часа пересмотреть — действительно ли editorial-motivational
- [ ] Запустить figma-generate-design с этим мудбордом

---

## Re-run

```
design-discovery "Делаю онбординг для финтех-приложения. ЦА — gen-z..."
```

Или передай в figma-generate-design:

```
figma-generate-design with brief: onboarding-fintech-genz-moodboard.md
```
```

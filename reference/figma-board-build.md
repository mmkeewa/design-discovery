# figma-board-build.md

Как собрать визуальный референс-борд в **Figma design-файле** (не FigJam) из выбранных пользователем референсов. Результат: один тёмный артборд, разбитый на **разделы** (по слоям приложения / категориям брифа); в каждом разделе — карточки, где карточка = **скриншот референса + название источника + ссылка на источник**.

Формат единицы борда:

```
Раздел  →  референс (скриншот)  →  название источника (+ кликабельная ссылка)
```

Это точный рецепт, проверенный вживую. Соблюдай шаги и «подводные камни» — иначе борд выйдет с багами (белые заголовки на белом, прилипшие к краям экраны).

**Живой поток design-discovery (v0.4+).** Файл создаётся в самом начале (Шаг 1 скилла), а разделы = слои. Порядок: сразу построй **каркас** — тёмный артборд + заголовок + три пустых раздела-плейсхолдера «Layer 1 · Patterns», «Layer 2 · Visual», «Layer 3 · Mood». Затем наполняй их **по мере готовности слоя**: собрал Layer 1 → залил карточки в его раздел, собрал Layer 2 → в свой, и т.д. Пользователь видит, как файл наполняется вживую, и может остановить. Ссылки в чат не выводятся — всё уходит в файл.

---

## 0. Пререквизиты

- **Figma MCP подключён** (проверка на Шаге 0 скилла). Без него — только markdown-мудборд, борд не собрать.
- **Обязательно загрузи скилл `figma-use` ПЕРЕД каждым вызовом `use_figma`**, и `figma-create-new-file` ПЕРЕД `create_new_file`. Пропуск даёт трудноуловимые ошибки.
- У пользователя может быть несколько Figma-команд. Вызови `whoami`; если команд > 1 — спроси, в какой создавать файл; возьми её `key` как `planKey`. View-only команды исключай.

---

## 1. Создать design-файл

`create_new_file` c `editorType: "design"` (НЕ figjam), `fileName: "{Brief} — Reference Board"`, `planKey` из шага 0. Сохрани `file_key` из ответа — он нужен для всех последующих вызовов.

---

## 2. Загрузить скриншоты референсов в файл

`use_figma` **не умеет** тянуть картинки: `fetch` не определён, `figma.createImageAsync` не поддержан. Единственный рабочий путь — `upload_assets`:

1. Скачай превью каждого выбранного референса локально (`curl -sL -o l1a.webp "{image_url}"`). Для Mobbin MCP `image_url` приходит прямо в ответе `search_*`; для галерей — со страницы через Chrome. Форматы: WebP / PNG / JPG / GIF.
2. Вызови `upload_assets` с `count: N` (по числу картинок) → получишь N одноразовых `submitUrl` (живут 10 минут).
3. POST'ни байты каждого файла на его `submitUrl` и собери `imageHash` из ответа (`{ imageHash, placedOnNodeId }`).

**Подводные камни POST-загрузки (кусали вживую):**
- Bash-шелл здесь **zsh**: `${!arr[@]}` — bad substitution; фоновые `&`+`wait` на пачке curl'ов **зависают** за таймаут. Грузи **последовательным** циклом (≈1.7 c на файл, 19 штук ≈ 30 c). Например, `paste` ключей и url'ов в TSV и `while IFS=... read`.
- `submitUrl` **одноразовые и истекают за 10 минут** — не тяни между запросом url'ов и POST'ом.
- `placedOnNodeId` — это авто-созданный узел, разбросанный по холсту. Их потом сотрёшь при постройке (см. шаг 3, чистка).

Сопоставляй `imageHash` со своими референсами по стабильному ключу (`l1a`, `l1b`, …), иначе перепутаешь картинки.

---

## 3. Построить артборд (use_figma)

Тёмная премиум-база. **Соблюдай два критичных правила ниже**, иначе получишь баги из реальной сборки.

> ⚠️ **createAutoLayout создаёт фрейм с БЕЛОЙ заливкой по умолчанию.** У всех контейнеров (заголовок, секции-разделы, строки карточек) обязательно ставь `fills = []` — иначе белые заголовки сольются с белым контейнером, а тёмный артборд будет перекрыт. Заливку оставляй только у самого артборда (тёмная) и у карточек (тёмная).
>
> ⚠️ **Отступы.** Карточкам ставь `padding ≥ 16` со всех сторон, иначе скриншот прилипает к краю контейнера. Между карточками `itemSpacing` и (для переносов) `counterAxisSpacing` ≈ 28.

**Порядок (чтобы влезть в rate-limit, см. §4):**

1. **Вызов 1 — чистка + артборд + заголовок.** Сотри всё с холста (авто-узлы аплоада и тестовые): `[...figma.currentPage.children].forEach(n => n.remove())`. Создай артборд-фрейм и титул.
2. **Вызовы 2…N — по 1–2 раздела за вызов.** Каждый раздел = заголовок «`NN · Название раздела`» + строка-пояснение + строка карточек (`layoutWrap: "WRAP"`).

Разделы = слои приложения / категории из брифа (напр. «Онбординг», «Доступ», «Здоровье и показатели», «Тренировки», «Премиум / mood»). Название источника в карточке = имя приложения/сайта; ссылка ведёт на источник (для Mobbin — `mobbin_url`).

### Референс-код (проверенный)

Артборд + титул:

```js
[...figma.currentPage.children].forEach(n => n.remove());
await figma.loadFontAsync({family:"Inter",style:"Bold"});
await figma.loadFontAsync({family:"Inter",style:"Semi Bold"});
await figma.loadFontAsync({family:"Inter",style:"Regular"});
const board = figma.createFrame();
board.name = "{Brief} — Reference Board";
board.layoutMode = "VERTICAL"; board.primaryAxisSizingMode = "AUTO"; board.counterAxisSizingMode = "FIXED";
board.resize(1160, 100);
board.paddingLeft = board.paddingRight = 64; board.paddingTop = 64; board.paddingBottom = 72;
board.itemSpacing = 56;
board.fills = [{type:"SOLID", color:{r:0.055,g:0.059,b:0.071}}]; // тёмная база
board.cornerRadius = 4;
// title: Bold 40 white + Regular 16 muted — оба в контейнере с fills=[]
return { boardId: board.id };
```

Хелпер раздела (карточка = экран + источник + ссылка). Палитра: white `#FFF`, muted `#9AA0A8`, note `#B4B9C0`, card-bg `#16181D`, accent/ссылка золото `#C8A96A`.

```js
const GOLD={r:0.784,g:0.663,b:0.416}, WHITE={r:1,g:1,b:1}, MUT={r:0.604,g:0.627,b:0.659},
      NOTE={r:0.706,g:0.725,b:0.753}, CARDBG={r:0.086,g:0.094,b:0.114};
function txt(font,size,color,ch){const n=figma.createText();n.fontName={family:"Inter",style:font};n.characters=ch;n.fontSize=size;n.fills=[{type:"SOLID",color}];return n;}
function buildSection(num,title,purpose,cards){
  const sec=figma.createAutoLayout("VERTICAL"); sec.name=num+" · "+title; sec.itemSpacing=20;
  board.appendChild(sec); sec.layoutSizingHorizontal="FILL"; sec.fills=[];      // ← прозрачный контейнер
  const h=txt("Semi Bold",22,WHITE,num+" · "+title); sec.appendChild(h); h.textAutoResize="HEIGHT";
  const p=txt("Regular",14,MUT,purpose); sec.appendChild(p); p.layoutSizingHorizontal="FILL"; p.textAutoResize="HEIGHT";
  const row=figma.createAutoLayout("HORIZONTAL"); row.itemSpacing=28; row.counterAxisSpacing=28; row.layoutWrap="WRAP";
  sec.appendChild(row); row.layoutSizingHorizontal="FILL"; row.fills=[];         // ← прозрачный контейнер
  for(const c of cards){
    const card=figma.createAutoLayout("VERTICAL"); card.itemSpacing=10;
    card.paddingLeft=card.paddingRight=card.paddingTop=card.paddingBottom=16;    // ← воздух, экран не прилипает
    card.cornerRadius=16; card.fills=[{type:"SOLID",color:CARDBG}]; card.name="card "+c.app;
    row.appendChild(card); card.layoutSizingHorizontal="FIXED"; card.resize(248,100); card.layoutSizingVertical="HUG";
    const img=figma.createRectangle(); img.resize(224,508); img.cornerRadius=10;  // ~299×678 → ratio 0.44
    img.fills=[{type:"IMAGE",imageHash:c.hash,scaleMode:"FILL"}]; card.appendChild(img);
    const an=txt("Semi Bold",16,WHITE,c.app); card.appendChild(an); an.layoutSizingHorizontal="FILL"; an.textAutoResize="HEIGHT"; // название источника
    const nt=txt("Regular",13,NOTE,c.note); card.appendChild(nt); nt.layoutSizingHorizontal="FILL"; nt.textAutoResize="HEIGHT";   // что взять
    const ln=txt("Medium",12,GOLD,"↗ Открыть источник"); card.appendChild(ln); ln.layoutSizingHorizontal="FILL"; ln.textAutoResize="HEIGHT";
    ln.setRangeHyperlink(0,ln.characters.length,{type:"URL",value:c.url});        // ← кликабельная ссылка на источник
  }
  return sec.id;
}
```

`c` = `{app: "название источника", hash: "imageHash", note: "что взять", url: "ссылка на источник"}`. Для ссылки `Medium`-шрифт грузи (`loadFontAsync {style:"Medium"}`) в том же вызове.

### Чистка авто-узлов

Авто-узлы от `upload_assets` (`placedOnNodeId`) — верхнеуровневые на странице; вызов 1 (`children.forEach(remove)`) их сметает, если построение идёт ПОСЛЕ загрузки картинок. Если что-то осталось — удали фреймы с именем «Uploaded Image».

---

## 4. Rate-limit Figma MCP

Pro Full seat: **15 вызовов/мин, 200/день**. Чтобы не упереться — **собирай по 2 раздела за один `use_figma`**, а не по одному. Скриншоть для проверки не каждый раздел, а один раз в конце (`get_screenshot` по id артборда). Если поймал лимит 15/мин — подожди ~60 c.

---

## 5. Проверка

`get_screenshot` по id артборда. Убедись: заголовки читаются (белое на тёмном), экраны не прилипают к краям карточек, у каждой карточки есть название источника и золотая ссылка «↗». При багах — точечный фикс: у контейнера забыли `fills=[]` или карточке не хватает padding.

---

## Итог для пользователя

```
Борд собран в Figma (design): {file_url}

Внутри: тёмный артборд, {N} разделов, {M} экранов.
Карточка = скриншот референса + название источника + ссылка на источник.
```

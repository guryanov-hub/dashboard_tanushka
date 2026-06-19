# Dashboard Tanya — документація

Файл: `index.html`
Мова інтерфейсу: українська
Останнє оновлення: червень 2026

---

## Загальний опис

Один HTML-файл без бекенду, npm та API-ключів. Всі залежності через CDN. П'ять блоків: Погода · Валюта · Лунний календар · Теніс WTA · Цей день в історії.

---

## CDN залежності

```html
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://unpkg.com/lucide@latest/dist/umd/lucide.js"></script>
```

---

## Дизайн-система

### CSS-змінні (теми)

Всі кольори визначені через CSS custom properties. Тема перемикається атрибутом `data-theme` на `<html>`.

| Змінна | Темна тема | Світла тема |
|---|---|---|
| `--bg` | `#0f172a` | `#f0f4f8` |
| `--surface` | `#1e293b` | `#ffffff` |
| `--border` | `#334155` | `#d1dae3` |
| `--inset` | `#0f172a` | `#f1f5f9` |
| `--text` | `#f8fafc` | `#1e293b` |
| `--muted` | `#94a3b8` | `#64748b` |
| `--accent` | `#f97316` | `#f97316` |
| `--divider` | `#1e293b` | `transparent` |
| `--tab-off` | `#64748b` | `#94a3b8` |

Горизонтальні лінії в секціях Теніс і Історія використовують `var(--divider)` — у світлій темі зникають автоматично.

Тема зберігається в `localStorage` під ключем `theme`. Відновлення відбувається через inline `<script>` у `<head>` — до рендеру, без мерехтіння.

### Спільні класи

| Клас | Властивість |
|---|---|
| `.card` | `background: var(--surface); border: 1px solid var(--border)` |
| `.inset` | `background: var(--inset)` |
| `.accent` | `color: var(--accent)` |
| `.muted` | `color: var(--muted)` |
| `.hist-link` | Посилання в блоці Історія з hover-акцентом |
| `.hdr-btn` | Кнопки хедера (оновлення, тема) |

**Анімації:**
- `.skeleton` — shimmer-лоадер (`var(--sk1)` / `var(--sk2)`)
- `.fade-in` — fadeIn + translateY(6px → 0), 0.4s ease
- `@keyframes spin` — оберт іконки оновлення (0.6s linear)

---

## Структура сторінки

### Desktop (≥ 769px)

```
<header>                      — назва + кнопка ↻ + перемикач теми + таймер оновлення

<div id="main-layout">        — CSS grid: 40% | 60%

  Рядок 1 (align-items: start — природна висота):
    #tab-pair  [40%]          — внутрішній grid 1fr 1fr
      #tab-weather            — Погода (20%)
      #tab-currency           — Валюта (20%)
    #tab-moon  [60%]          — Лунний календар

  Рядок 2 (align-self: stretch — однакова висота):
    #tab-tennis  [40%]        — Теніс WTA
    #tab-history [60%]        — Цей день в історії

<nav class="mob-nav">         — прихована на desktop

<footer>                      — підпис джерел
```

Висота карток у кожному рядку вирівнюється через `height: 100%` на `.mob-tab` і `.card`.

### Mobile (≤ 768px)

`#main-layout` стає `display: block`. Показується фіксована нижня навігаційна панель (`mob-nav`) з 5 вкладками:

| Вкладка | id панелі | Lucide іконка |
|---|---|---|
| Погода | `#tab-weather` | `cloud-sun` |
| Валюта | `#tab-currency` | `banknote` |
| Місяць | `#tab-moon` | `moon` |
| Теніс | `#tab-tennis` | `trophy` |
| Історія | `#tab-history` | `book-open` |

Одночасно видно лише одна вкладка. Активна вкладка підсвічується акцентним кольором. Після оновлення даних залишається поточна вкладка (`activeMobileTab`).

`#tab-pair` на mobile використовує `display: contents` — Weather і Currency стають прямими дочірніми елементами блоку та керуються окремо.

---

## Хедер

```html
<header>
  <div>Live Dashboard / Київ</div>
  <div>
    [↻ Оновити]         — manualRefresh() + spin-анімація
    [☀️/🌙 Тема]        — toggleTheme() + збереження в localStorage
    [Оновлено HH:MM:SS] — #last-updated
  </div>
</header>
```

---

## Блок 1 — Погода (Київ)

### HTML id: `#weather-card`, контент: `#weather-content`

### API
```
GET https://api.open-meteo.com/v1/forecast
  ?latitude=50.45
  &longitude=30.52
  &current=temperature_2m,relative_humidity_2m,wind_speed_10m,weather_code
  &wind_speed_unit=kmh
  &timezone=Europe/Kyiv
```
Безкоштовно, без ключа, CORS відкритий.

### Що відображається
- Емодзі погоди (функція `wIcon(code)`) + температура великим шрифтом (48px, акцент)
- Текстовий опис (`WEATHER_CODES[code]`) поруч із назвою "Київ, UA"
- Вологість і Вітер — два рядки `.inset` **під** температурою, вирівняні по лівому краю

### Функція
```js
async function fetchWeather()
```
Викликається при старті та кожні 60 секунд через `setInterval`.

### Таблиця weather_code → текст (WEATHER_CODES)
| Код | Текст |
|---|---|
| 0 | Ясно |
| 1 | Переважно ясно |
| 2 | Мінлива хмарність |
| 3 | Хмарно |
| 45, 48 | Туман / Крижаний туман |
| 51–55 | Мряка |
| 61–65 | Дощ |
| 71–75 | Сніг |
| 80–82 | Злива |
| 95–99 | Гроза |

---

## Блок 2 — Валюта

### HTML id: `#currency-card`, контент: `#currency-content`

### Джерела даних

| Курс | API | Ключ |
|---|---|---|
| USD/UAH, EUR/UAH | NBU StatService | не потрібен |
| BTC/USD | CoinGecko public | не потрібен |

```
# Поточний курс NBU
GET https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?valcode=USD&json

# Курс на конкретну дату (YYYYMMDD)
GET https://bank.gov.ua/NBUStatService/v1/statdirectory/exchange?valcode=USD&date=20260618&json

# BTC поточний + 24h зміна
GET https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd&include_24hr_change=true

# BTC історія (90 днів)
GET https://api.coingecko.com/api/v3/coins/bitcoin/market_chart?vs_currency=usd&days=90
```

**Важливо:** NBU range-параметри (`start`/`end`) не працюють — повертають порожній масив. Використовуються індивідуальні запити на кожну дату через `Promise.all`.

### Що відображається

Три рядки один під одним:
```
USD / UAH    44.9125    ▲ +0.29%
EUR / UAH    51.4563    ▼ -0.93%
BTC / USD    62 952     ▼ -2.11%
```

Зміна порівнюється з попереднім **робочим** днем NBU (вихідні пропускаються).

### Модальне вікно (клік на рядок)

При кліці на будь-який рядок відкривається `#currency-modal` з графіком динаміки курсу. Перемикачі: **Тиждень** (7 днів, відкривається за замовчуванням) · **Місяць** (30 днів) · **Квартал** (90 днів).

Графік будується через `<canvas>` з ручним малюванням (без Chart.js).

### Ключові функції

```js
async function fetchCurrency()          // завантажує поточні курси
function renderCurrency()               // малює три рядки
async function getCurrencyHistory(code, days)  // NBU/CoinGecko, кешується
async function openCurrencyModal(code)  // відкриває модальне вікно
function loadCurrencyChart()            // перемальовує canvas
```

---

## Блок 3 — Лунний календар

### HTML id: `#moon-card`

Єдиний об'єднаний блок з трьома зонами: шапка з вибором дати, фазова смуга, секції рекомендацій.

---

### 3.1 Вибір дати

```html
<button onclick="shiftDay(-1)">‹</button>
<input type="date" id="moon-date" />
<button onclick="shiftDay(1)">›</button>
<button onclick="goToday()">Сьогодні</button>
```

**Стан:** `let selectedDate = new Date()` (встановлюється в 12:00:00 щоб уникнути UTC-зсуву).

**Логіка:**
- При зміні дати → `renderMoon()` перераховує все локально, без запитів
- `shiftDay(delta)` — переміщення на ±1 день кнопками ‹ ›
- `goToday()` скидає `selectedDate` до сьогодні та оновлює `input.value`

---

### 3.2 Астрономічний розрахунок (локальний, без API)

```js
function calcMoonData(date) → { day, illum, waxing, signIdx, fraction }
```

**Алгоритм:**
1. Відома точка відліку: новолуння `2000-01-06T18:14:00Z`
2. Синодичний місяць: `29.53058867` діб
3. `fraction` = дробова частина кількості повних місяців від точки відліку (0..1)
4. `day` = `Math.floor(fraction * 29.53) + 1`
5. `illum` = `round((0.5 - 0.5 * cos(fraction * 2π)) * 100)` %
6. `waxing` = `fraction < 0.5`
7. Знак зодіаку: спрощена формула еклептичної довготи Місяця

**Точність:** похибка ~0.5–1 лунного дня. Достатньо для практичних рекомендацій.

---

### 3.3 Фазова смуга (Phase Strip)

| id | Вміст |
|---|---|
| `#moon-emoji` | Емодзі фази (🌑🌒🌓🌔🌕🌖🌗🌘) |
| `#moon-day-num` | Номер лунного дня (акцент, 36px) |
| `#moon-day-sub` | "лунний день · {назва фази}" |
| `#moon-phase-label` | Емодзі + назва у заголовку картки |
| `#moon-sign-badge` | "♈ Місяць у Овні" |
| `#moon-illum` | "Освітленість 73%" |
| `#moon-wax` | ↑ Зростаючий / ↓ Спадаючий (з кольором) |

**Таблиця фаз (`fraction` → назва/емодзі):**
| fraction | Назва | Емодзі |
|---|---|---|
| < 0.03 або > 0.97 | Новолуння | 🌑 |
| 0.03–0.22 | Зростаючий серп | 🌒 |
| 0.22–0.28 | Перша чверть | 🌓 |
| 0.28–0.47 | Зростаючий горбатий | 🌔 |
| 0.47–0.53 | Повний місяць | 🌕 |
| 0.53–0.72 | Спадаючий горбатий | 🌖 |
| 0.72–0.78 | Остання чверть | 🌗 |
| 0.78–0.97 | Спадаючий серп | 🌘 |

---

### 3.4 Попередження (`#moon-warning`)

Показується автоматично при:
- **Новолуння** (`fraction < 0.04 || > 0.96`) — червоний банер
- **Повнолуння** (`fraction > 0.46 && < 0.54`) — жовтий банер

---

### 3.5 Знаки зодіаку та типи днів

| Тип | Назва | Колір |
|---|---|---|
| leaf | 🌿 Листовий день | #22c55e |
| flower | 🌸 Квітковий день | #e879f9 |
| fruit | 🍅 Плодовий день | #f97316 |
| root | 🥕 Кореневий день | #f59e0b |

---

### 3.6 Секції рекомендацій (2×2 сітка)

Чотири плитки `.inset` з рекомендаціями:
- ✂️ **Стрижка волосся** — оцінка + текст + підказка про зростаючий/спадаючий місяць
- 🦷 **Лікування зубів** — оцінка + текст
- 💧 **Полив рослин** — рекомендація по типу дня
- 🌱 **Висадка рослин** — рекомендація по типу дня + фазі місяця

**Оцінки (`pill(rate)`):** `great` (★ Відмінно, зелений) · `good` (✓ Добре, помаранчевий) · `bad` (~ Небажано, жовтий) · `avoid` (✕ Уникати, червоний)

---

## Блок 4 — Теніс WTA

### HTML id: `#tennis-card`

### Джерело даних

**ESPN WTA API** — безкоштовно, без ключа, CORS відкритий.

```
GET https://site.api.espn.com/apis/site/v2/sports/tennis/wta/scoreboard?dates=YYYYMMDD
```

Виводяться матчі **всіх українських гравчинь** (фільтрація за `athlete.flag.alt === 'Ukraine'`).

### Вкладки дат

Три вкладки: **Сьогодні** + 2 наступні дні. Всі дані завантажуються одним запуском `fetchTennis()`.

### Структура ESPN API

```
response.events[]
  └── .name                        — назва турніру
  └── .groupings[].competitions[]  — окремі матчі
       ├── .id                     — унікальний ID (для дедуплікації)
       ├── .date                   — ISO timestamp UTC
       ├── .timeValid              — bool: false = час не підтверджено
       └── .competitors[]
            └── .athlete.displayName
            └── .athlete.flag.alt  — "Ukraine", "Germany" …
```

Дублі усуваються через `Set` унікальних `comp.id`.

### Конвертація часу UTC → Київ

Україна постійно UTC+3. `utcToKyiv(t)` додає 3 години. Час береться лише якщо `timeValid === true`, інакше показується "час уточнюється".

---

## Блок 5 — Цей день в історії

### HTML id: `#history-card`

### Структура

Завжди дві події:
1. 🌍 **Подія зі світової історії** — з Wikipedia (Ukrainian)
2. 🇺🇦 **Подія з історії України** — з пулу: `UA_HISTORY` + Wikipedia з фільтром `/україн/i`

### API

```
GET https://uk.wikipedia.org/api/rest_v1/feed/onthisday/events/MM/DD
```

### Логіка відбору подій

При кожному виклику `fetchHistory()` події вибираються **випадково** з доступного пулу:

```js
const pick = arr => arr.length ? arr[Math.floor(Math.random() * arr.length)] : null;
renderHistory(pick(worldPool), pick(uaPool));
```

- **worldPool** — всі Wikipedia-події дня без слова "Україн"
- **uaPool** — hardcoded запис з `UA_HISTORY` (якщо є на цю дату) + Wikipedia-події з `/україн/i` у тексті

Кожне оновлення (↻ або при старті) показує нову випадкову пару подій.

### Посилання

Кожна подія — клікабельна (клас `.hist-link`). При кліці відкривається **Google пошук** по рядку `"рік текст події"`:

```js
`https://www.google.com/search?q=${encodeURIComponent(year + ' ' + text)}`
```

### Hardcoded UA_HISTORY (20 ключових дат)

| Дата | Рік | Подія |
|---|---|---|
| 01-22 | 1919 | Акт Злуки УНР і ЗУНР |
| 01-25 | 1918 | Бій під Крутами |
| 02-09 | 1918 | Брестський мир — перше визнання УНР |
| 02-19 | 1954 | Передача Криму Українській РСР |
| 02-20 | 2014 | Найкривавіший день Майдану |
| 02-24 | 2022 | Початок повномасштабного вторгнення |
| 03-09 | 1814 | Народження Тараса Шевченка |
| 04-26 | 1986 | Вибух на Чорнобильській АЕС |
| 05-22 | 1861 | Перепоховання Шевченка в Каневі |
| 06-28 | 1996 | Прийнята Конституція України |
| 07-16 | 1990 | Декларація про державний суверенітет |
| 07-28 | 988 | Хрещення Русі |
| 08-24 | 1991 | Акт проголошення незалежності України |
| 09-22 | 1941 | Початок масових страт у Бабиному Яру |
| 10-14 | 1942 | Створення УПА |
| 11-21 | 2013 | Початок Євромайдану |
| 11-22 | 2004 | Початок Помаранчевої революції |
| 12-01 | 1991 | Референдум про незалежність (90.3%) |
| 12-05 | 1994 | Будапештський меморандум |
| 12-25 | 1991 | Офіційний розпад СРСР |

### Ключові функції

```js
async function fetchHistory()           // завантаження, відбір, випадковий вибір
function renderHistory(worldEvt, uaEvt) // рендер двох рядків
```

---

## JS функції — зведена таблиця

| Функція | Що робить |
|---|---|
| `calcMoonData(date)` | Астрономічний розрахунок: day, illum, waxing, signIdx, fraction |
| `renderMoon()` | Оновлює всі DOM-елементи лунного блоку для `selectedDate` |
| `shiftDay(delta)` | Зсуває `selectedDate` на ±1 день (кнопки ‹ ›) |
| `goToday()` | Скидає selectedDate до сьогодні, оновлює input і renderMoon |
| `fetchWeather()` | async, запит до Open-Meteo, оновлює `#weather-content` |
| `fetchCurrency()` | async, NBU + CoinGecko, оновлює `currencyData` і рендерить |
| `renderCurrency()` | Малює три рядки курсів |
| `getCurrencyHistory(code, days)` | async, NBU/CoinGecko з кешем |
| `openCurrencyModal(code)` | Відкриває модальне вікно з графіком |
| `loadCurrencyChart()` | Перемальовує canvas у модальному вікні |
| `fetchTennis()` | async, ESPN WTA API за 3 дні, оновлює `tennisData` |
| `renderTennisDay()` | Малює рядки матчів для поточної вкладки |
| `buildTennisTabs()` | Будує горизонтальну смугу з 3 вкладок дат |
| `setTennisDate(iso)` | Перемикає активну дату тенісу |
| `fetchHistory()` | async, Wikipedia + UA_HISTORY, випадковий відбір |
| `renderHistory(w, ua)` | Рендер двох подій (світова + українська) |
| `toggleTheme()` | Перемикає `data-theme` на `<html>`, зберігає в localStorage |
| `updateThemeIcon()` | Оновлює іконку кнопки теми (sun/moon) |
| `manualRefresh()` | Spin-анімація + `refreshAll()` |
| `switchMobileTab(tab)` | Показує відповідну вкладку, оновлює `activeMobileTab` |
| `utcToKyiv(t)` | "HH:MM" UTC → "HH:MM" Київ (+3h) |
| `getDateStr(offset)` | Повертає ISO-дату зі зміщенням у днях від сьогодні |
| `dayLabel(iso)` | ISO дата → "Сьогодні" або "15 чер" |
| `nbuDateStr(d)` | Date → "YYYYMMDD" для NBU API |
| `parseNBUDate(str)` | "DD.MM.YYYY" NBU → "15 червня" |
| `toDateVal(d)` | Date → "YYYY-MM-DD" для input[type=date] |
| `phaseEmoji(f)` | fraction → емодзі місяця |
| `phaseName(f)` | fraction → назва фази українською |
| `pill(rate)` | rate → HTML-span з кольоровою міткою оцінки |
| `wIcon(code)` | weather_code → емодзі погоди |
| `updateTime()` | Оновлює `#last-updated` поточним часом |
| `refreshAll()` | updateTime + fetchWeather + fetchCurrency + renderMoon + fetchTennis + fetchHistory |

---

## Автооновлення

```js
refreshAll();  // при старті
setInterval(() => { updateTime(); fetchWeather(); }, 60000);  // кожні 60 с
```

`fetchCurrency`, `fetchTennis`, `fetchHistory` — тільки при старті або вручну через кнопку ↻ (`manualRefresh()`).

---

## Як відтворити з нуля

1. Один файл `index.html`, відкривається напряму в браузері
2. Інтернет потрібен для CDN (Tailwind, Lucide, Inter) + Open-Meteo + NBU + CoinGecko + ESPN + Wikipedia
3. Лунний календар працює повністю офлайн після завантаження CDN
4. Щоб змінити місто погоди — замінити `latitude`, `longitude`, `timezone` в `fetchWeather()`
5. Щоб додати нову дату в UA_HISTORY — додати рядок формату `'MM-DD': [рік, 'текст']`

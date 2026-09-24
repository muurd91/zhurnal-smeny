# Лендинг «Журнал смены» — план реализации

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Собрать статичный одностраничный лендинг «Журнал смены» в одном файле `index.html`, который объясняет идею мастерам смен на других производствах и собирает контакт через `mailto:`.

**Architecture:** Один HTML-файл с инлайновыми `<style>` и `<script>`, без сборки и внешних зависимостей. Страница состоит из четырёх секций (`hero`, `solution`, `cta`) внутри `<main class="page">`, стили — mobile-first с брейкпоинтом на десктоп, форма собирает `mailto:`-ссылку через одну чистую JS-функцию.

**Tech Stack:** Чистые HTML5 / CSS3 / vanilla JS. Никаких фреймворков, CDN, npm-зависимостей.

**Spec:** [`docs/superpowers/specs/2026-09-24-zhurnal-smeny-landing-design.md`](./2026-09-24-zhurnal-smeny-landing-design.md)

## Global Constraints

- Один файл `index.html` — HTML, `<style>` и `<script>` в одном файле, без сборки.
- Никаких внешних зависимостей: ни CDN-шрифтов, ни внешних скриптов/стилей — страница должна открываться офлайн через `file://`.
- Никакого backend и сторонних форм-сервисов — заявка уходит только через `mailto:`.
- Приоритет мобильной вёрстки (~360–414px), десктоп не должен ломаться, но не является основным сценарием.
- Текст не придумывает факты о читателе — опирается только на идею 1 из `yc-rfs/product-ideas.md`.
- Email в CTA: `myrdoc911@gmail.com` (подтверждено при брейнсторминге, будет виден в исходном коде страницы).

## Review Focus

- Пользователь вводит только пробелы в поле имени/контакта — HTML `required` пропускает непустую строку из пробелов, и мимо него уйдёт пустое письмо. Тест в Task 4.
- SVG-схема «3 касания» ломает горизонтальный скролл на узком экране (360px), если не резиновая. Тест в Task 3.
- JS не выполнился (заблокирован/ошибка) — отправка формы не должна кидать ошибку или уводить на битую страницу, максимум безопасная перезагрузка. Тест в Task 4.
- Общая раскладка страницы переполняет ширину экрана на 360px ещё до появления контента (сам каркас/шрифты). Тест в Task 1, подтверждается в Task 5.
- Пользователь идёт по странице с клавиатуры (Tab) — поля формы и кнопка CTA должны иметь видимый фокус. Тест в Task 4.

---

### Task 1: Каркас и стили

**Files:**
- Create: `index.html`

**Interfaces:**
- Produces: CSS-переменные `--bg`, `--bg-panel`, `--text`, `--text-muted`, `--accent`, `--accent-text`, `--radius`, `--max-w`; контейнер `main.page`; базовые селекторы `section`, `h1`, `h2`, `p`, `p.lead`, `.eyebrow`, `.btn` — все следующие задачи вставляют разметку внутрь `main.page` и используют эти классы.

- [ ] **Step 1: Создать `index.html` с каркасом и базовыми стилями**

```html
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Журнал смены — учёт простоев без облака</title>
<meta name="description" content="Планшет в центре цеха вместо журнала простоев на бумаге или в голове. Три касания на событие, без облака и аккаунтов.">
<style>
  :root {
    --bg: #14181c;
    --bg-panel: #1c2127;
    --text: #eef1f4;
    --text-muted: #8b95a1;
    --accent: #e0a83f;
    --accent-text: #14181c;
    --radius: 12px;
    --max-w: 720px;
  }

  * { box-sizing: border-box; }

  body {
    margin: 0;
    background: var(--bg);
    color: var(--text);
    font-family: -apple-system, "Segoe UI", Roboto, Arial, sans-serif;
    line-height: 1.55;
  }

  main.page {
    max-width: var(--max-w);
    margin: 0 auto;
    padding: 0 20px;
  }

  section {
    padding: 56px 0;
    border-bottom: 1px solid rgba(255, 255, 255, 0.06);
  }

  section:last-of-type { border-bottom: none; }

  h1 {
    font-size: clamp(1.6rem, 5vw, 2.4rem);
    line-height: 1.2;
    margin: 0 0 16px;
  }

  h2 {
    font-size: clamp(1.2rem, 3.5vw, 1.6rem);
    margin: 0 0 20px;
  }

  p {
    margin: 0 0 16px;
    color: var(--text-muted);
  }

  p.lead {
    color: var(--text);
    font-size: 1.05rem;
  }

  .eyebrow {
    color: var(--accent);
    text-transform: uppercase;
    letter-spacing: 0.08em;
    font-size: 0.75rem;
    font-weight: 600;
    margin: 0 0 12px;
  }

  .btn {
    display: inline-block;
    background: var(--accent);
    color: var(--accent-text);
    font-weight: 600;
    padding: 14px 28px;
    border-radius: var(--radius);
    border: none;
    text-decoration: none;
    font-size: 1rem;
    cursor: pointer;
  }

  .btn:hover { filter: brightness(1.08); }

  @media (max-width: 480px) {
    section { padding: 40px 0; }
  }
</style>
</head>
<body>
<main class="page">
</main>
</body>
</html>
```

- [ ] **Step 2: Запустить локальный сервер для проверки**

Run: `python -m http.server 8000` (Windows: `py -m http.server 8000`) из папки `zhurnal-smeny`, в фоне.
Если Python недоступен: `npx --yes serve -l 8000 .`

- [ ] **Step 3: Проверить каркас через Playwright MCP**

Открыть `mcp__playwright__browser_navigate` на `http://localhost:8000/index.html`, затем:
- `mcp__playwright__browser_console_messages` — ожидать пустой список ошибок.
- `mcp__playwright__browser_evaluate`: `() => getComputedStyle(document.body).backgroundColor` — ожидать `"rgb(20, 24, 28)"`.
- `mcp__playwright__browser_resize` на `360x740`, затем `browser_evaluate`: `() => document.documentElement.scrollWidth <= window.innerWidth` — ожидать `true` (Review Focus: базовая раскладка не должна давать горизонтальный скролл).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add page shell and base styles"
```

---

### Task 2: Первый экран (шапка-проблема)

**Files:**
- Modify: `index.html` (внутри `<main class="page">`)

**Interfaces:**
- Consumes: `.eyebrow`, `h1`, `p.lead` из Task 1.
- Produces: `<section id="hero">` — Task 3 вставляется сразу после неё.

- [ ] **Step 1: Проверить, что секции ещё нет**

`mcp__playwright__browser_evaluate`: `() => document.getElementById('hero')` — ожидать `null`.

- [ ] **Step 2: Добавить секцию hero**

Вставить внутрь `<main class="page">` (сейчас пустого):

```html
  <section id="hero">
    <p class="eyebrow">Журнал смены</p>
    <h1>Простои, брак и передача смены — вы это вообще где-то записываете?</h1>
    <p class="lead">На большинстве участков ведётся только один журнал — объёма продукции. Что стояло, почему и сколько времени — теряется в течение смены и не всплывает, пока кто-то не спросит «а что случилось на прошлой неделе». Журнал смены закрывает это одним планшетом в центре цеха.</p>
  </section>
```

- [ ] **Step 3: Проверить секцию**

`mcp__playwright__browser_navigate` заново на `http://localhost:8000/index.html`, затем:
- `browser_evaluate`: `() => document.querySelector('main.page > section:first-child').id` — ожидать `"hero"`.
- `browser_evaluate`: `() => document.querySelector('#hero h1').textContent.length > 0` — ожидать `true`.
- `browser_evaluate`: `() => getComputedStyle(document.querySelector('#hero .eyebrow')).color` — ожидать цвет акцента `"rgb(224, 168, 63)"`.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add hero section"
```

---

### Task 3: Блок проблемы и решения (как работает + почему устойчиво)

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `h2`, `p`, `.eyebrow`-стили из Task 1; секция вставляется после `#hero` из Task 2.
- Produces: `<section id="solution">` с SVG-схемой `.steps-diagram` и списком `.why-list` — ни один следующий шаг эти классы не переиспользует.

- [ ] **Step 1: Проверить, что секции ещё нет**

`browser_evaluate`: `() => document.getElementById('solution')` — ожидать `null`.

- [ ] **Step 2: Добавить CSS для схемы и списка**

Вставить перед закрывающим `</style>` из Task 1:

```css
  .steps-diagram {
    display: block;
    width: 100%;
    height: auto;
    margin: 24px 0;
  }

  .steps-diagram .step-line { stroke: var(--text-muted); stroke-width: 2; }
  .steps-diagram .step-circle { fill: var(--bg-panel); stroke: var(--accent); stroke-width: 3; }
  .steps-diagram .step-num { fill: var(--text); font-size: 20px; font-weight: 700; text-anchor: middle; }
  .steps-diagram .step-label { fill: var(--text-muted); font-size: 14px; text-anchor: middle; }

  .why-list {
    list-style: none;
    padding: 0;
    margin: 0;
    display: grid;
    gap: 16px;
  }

  .why-list li {
    padding-left: 20px;
    position: relative;
    color: var(--text-muted);
  }

  .why-list li::before {
    content: "—";
    position: absolute;
    left: 0;
    color: var(--accent);
  }
```

- [ ] **Step 3: Добавить секцию solution после `#hero`**

```html
  <section id="solution">
    <h2>Как это работает</h2>
    <svg viewBox="0 0 600 160" role="img" aria-label="Три касания: оборудование, что случилось, износ или человек и длительность" class="steps-diagram">
      <line class="step-line" x1="100" y1="80" x2="300" y2="80"/>
      <line class="step-line" x1="300" y1="80" x2="500" y2="80"/>
      <circle class="step-circle" cx="100" cy="80" r="36"/>
      <circle class="step-circle" cx="300" cy="80" r="36"/>
      <circle class="step-circle" cx="500" cy="80" r="36"/>
      <text class="step-num" x="100" y="87">1</text>
      <text class="step-num" x="300" y="87">2</text>
      <text class="step-num" x="500" y="87">3</text>
      <text class="step-label" x="100" y="140">оборудование</text>
      <text class="step-label" x="300" y="140">что случилось</text>
      <text class="step-label" x="500" y="140">износ / человек + время</text>
    </svg>
    <p>Одно событие — три касания на планшете. Мастер отмечает, какая единица оборудования, что случилось, и износ это или человеческий фактор, плюс сколько длилось. Запись постфактум, одним заходом — разбираться посреди смены некогда.</p>

    <h2>Почему это устойчиво</h2>
    <ul class="why-list">
      <li>Один планшет в центре цеха — если участки стоят в одном зале, хватает одного устройства на всех, а не телефона в кармане у каждого.</li>
      <li>Данные остаются на планшете. Без облака, без аккаунтов и согласований с ИТ.</li>
      <li>Список причин простоя не придуман заранее — он вырастает из того, что реально записывают.</li>
    </ul>
  </section>
```

Вставить сразу после `</section>` секции `hero`.

- [ ] **Step 4: Проверить секцию и адаптивность SVG**

`browser_navigate` заново, затем:
- `browser_evaluate`: `() => document.querySelectorAll('#hero, #solution').length === 2 && Array.from(document.querySelectorAll('main.page > section')).map(s => s.id).indexOf('solution') === 1` — ожидать `true` (идёт сразу после hero).
- `browser_evaluate`: `() => document.querySelectorAll('#solution .step-circle').length` — ожидать `3`.
- `browser_evaluate`: `() => document.querySelectorAll('#solution .why-list li').length` — ожидать `3`.
- `browser_resize` на `360x740`, `browser_evaluate`: `() => document.documentElement.scrollWidth <= window.innerWidth` — ожидать `true` (Review Focus: SVG не ломает узкий экран).

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add solution section with steps diagram and sustainability list"
```

---

### Task 4: Блок призыва к действию (CTA)

**Files:**
- Modify: `index.html`

**Interfaces:**
- Consumes: `.btn` из Task 1; секция вставляется после `#solution` из Task 3.
- Produces: `<section id="cta">` с формой `#contact-form`, полями `#cta-name` / `#cta-contact`, и функцией `window.buildMailtoHref(name, contact)`, которую использует Task 5 для финальной проверки.

- [ ] **Step 1: Проверить, что секции и функции ещё нет**

`browser_evaluate`: `() => document.getElementById('cta') === null && typeof window.buildMailtoHref === 'undefined'` — ожидать `true`.

- [ ] **Step 2: Добавить CSS формы**

Вставить перед закрывающим `</style>`:

```css
  form {
    display: grid;
    gap: 16px;
    max-width: 420px;
  }

  label {
    display: block;
    font-size: 0.85rem;
    color: var(--text-muted);
    margin-bottom: 6px;
  }

  input {
    width: 100%;
    background: var(--bg-panel);
    border: 1px solid rgba(255, 255, 255, 0.12);
    border-radius: 8px;
    padding: 12px 14px;
    color: var(--text);
    font-size: 1rem;
  }

  input:focus,
  .btn:focus-visible {
    outline: 2px solid var(--accent);
    outline-offset: 2px;
  }
```

- [ ] **Step 3: Добавить секцию cta и скрипт после `#solution`**

```html
  <section id="cta">
    <h2>Хотите попробовать на своей смене?</h2>
    <p>Оставьте, как с вами связаться — вышлю минимальную версию, когда она будет готова к показу.</p>
    <form id="contact-form">
      <div>
        <label for="cta-name">Имя</label>
        <input id="cta-name" name="name" type="text" required>
      </div>
      <div>
        <label for="cta-contact">Как связаться (телефон, telegram, почта)</label>
        <input id="cta-contact" name="contact" type="text" required>
      </div>
      <button type="submit" class="btn">Оставить контакт</button>
    </form>
  </section>
```

Затем перед `</body>`:

```html
<script>
(function () {
  var TARGET_EMAIL = 'myrdoc911@gmail.com';

  function buildMailtoHref(name, contact) {
    var subject = encodeURIComponent('Журнал смены — хочу попробовать');
    var body = encodeURIComponent('Имя: ' + name + '\nКак связаться: ' + contact);
    return 'mailto:' + TARGET_EMAIL + '?subject=' + subject + '&body=' + body;
  }

  window.buildMailtoHref = buildMailtoHref;

  var form = document.getElementById('contact-form');
  form.addEventListener('submit', function (event) {
    event.preventDefault();
    var name = document.getElementById('cta-name').value.trim();
    var contact = document.getElementById('cta-contact').value.trim();

    if (!name || !contact) {
      var emptyField = !name ? document.getElementById('cta-name') : document.getElementById('cta-contact');
      emptyField.focus();
      return;
    }

    window.location.href = buildMailtoHref(name, contact);
  });
})();
</script>
```

Форма не получает атрибут `action` — если скрипт не выполнится, обычная отправка перезагрузит страницу с параметрами в URL, без ошибок (Review Focus: JS не выполнился).

- [ ] **Step 4: Проверить функцию сборки mailto и защиту от пробелов**

`browser_navigate` заново, затем:
- `browser_evaluate`:
  ```js
  () => {
    var href = window.buildMailtoHref('Иван', '+7 900 000-00-00');
    var url = new URL(href);
    return url.protocol === 'mailto:'
      && url.pathname === 'myrdoc911@gmail.com'
      && decodeURIComponent(url.searchParams.get('subject')) === 'Журнал смены — хочу попробовать'
      && decodeURIComponent(url.searchParams.get('body')) === 'Имя: Иван\nКак связаться: +7 900 000-00-00';
  }
  ```
  — ожидать `true`. Так проверка не зависит от ручного расчёта percent-encoding.
- Заполнить `#cta-name` пробелами (`"   "`) и `#cta-contact` реальным текстом, кликнуть кнопку `.btn`, затем `browser_evaluate`: `() => document.activeElement.id` — ожидать `"cta-name"` (форма не ушла в mailto, фокус вернулся на пустое поле — Review Focus: пробелы не проходят как валидное значение).
- `mcp__playwright__browser_press_key` — с фокусом на `#cta-name`, нажать `Tab` дважды до кнопки, `browser_evaluate`: `() => getComputedStyle(document.activeElement).outlineStyle` — ожидать `"solid"` (Review Focus: видимый фокус с клавиатуры).
- `browser_evaluate`: `() => document.getElementById('contact-form').hasAttribute('action')` — ожидать `false`.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add CTA form with mailto submission"
```

---

### Task 5: Финальная проверка

**Files:**
- Modify: `index.html` (только если проверка найдёт проблему)

**Interfaces:**
- Consumes: весь `index.html` из Task 1–4. Ничего не производит для последующих задач — это последняя задача плана.

- [ ] **Step 1: Проверить консоль и заголовки на десктопе**

`browser_resize` на `1280x800`, `browser_navigate` на `http://localhost:8000/index.html`, `mcp__playwright__browser_console_messages` — ожидать пустой список ошибок и предупреждений.

- [ ] **Step 2: Проверить мобильную раскладку целиком**

`browser_resize` на `360x740`, затем:
- `browser_evaluate`: `() => document.documentElement.scrollWidth <= window.innerWidth` — ожидать `true` (весь документ, а не отдельные секции).
- `mcp__playwright__browser_take_screenshot` — визуально свериться, что все 3 секции (`hero`, `solution`, `cta`) читаемы и не наезжают друг на друга.

- [ ] **Step 3: Свериться с фактами из идеи**

Открыть `../../../yc-rfs/product-ideas.md`, раздел «1. Журнал смены», и построчно сверить текст `#hero` и `#solution` — убедиться, что ни одно утверждение не придумано (три касания, один планшет, список причин растёт сам, без облака).

- [ ] **Step 4: Проверить порядок и полноту секций**

`browser_evaluate`: `() => Array.from(document.querySelectorAll('main.page > section')).map(s => s.id)` — ожидать `["hero", "solution", "cta"]`.

- [ ] **Step 5: Остановить локальный сервер**

Остановить процесс `python -m http.server 8000` (или `serve`), запущенный в Task 1.

- [ ] **Step 6: Commit (если были правки)**

```bash
git add index.html
git commit -m "fix: address final review findings"
```

Если правок не было — коммита нет, задача закрывается на предыдущем шаге.

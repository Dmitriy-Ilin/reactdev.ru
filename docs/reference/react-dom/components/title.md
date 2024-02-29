---
status: experimental
description: Встроенный компонент браузера title позволяет указать заголовок документа.
---

# &lt;title&gt;

!!!example "Canary"

    Расширения React для `<title>` в настоящее время доступны только в канале React canary и экспериментальном канале. В стабильных релизах React `<title>` работает только как [встроенный в браузер HTML-компонент](./index.md#all-html-components). Подробнее о [каналах выпуска React здесь](https://react.dev/community/versioning-policy#all-release-channels).

<big>Встроенный компонент браузера [`<title>`](https://hcdev.ru/html/title) позволяет указать заголовок документа.</big>

```js
<title>My Blog</title>
```

## Описание {#reference}

### `<title>` {#title}

Чтобы указать заголовок документа, отобразите [встроенный в браузер компонент `<title>`](https://hcdev.ru/html/title). Вы можете рендерить `<title>` из любого компонента, и React всегда будет помещать соответствующий элемент DOM в голову документа.

```js
<title>My Blog</title>
```

**Параметры**

`<title>` поддерживает все [общие свойства элемента](./common.md#props)

-   `children`: `<title>` принимает в качестве дочернего элемента только текст. Этот текст станет заголовком документа. Вы также можете передавать свои собственные компоненты, если они отображают только текст.

#### Особое поведение при рендеринге {#special-rendering-behavior}

React всегда будет помещать элемент DOM, соответствующий компоненту `<title>`, в `<head>` документа, независимо от того, в каком месте дерева React он отрисовывается. `<head>` - единственное допустимое место для `<title>` в DOM, но это удобно и позволяет сохранить композитность, если компонент, представляющий определенную страницу, может сам рендерить свой `<title>`.

Есть два исключения из этого правила:

-   Если `<title>` находится внутри компонента `<svg>`, то особого поведения не требуется, поскольку в данном контексте он не представляет собой заголовок документа, а является [аннотацией доступности для этой SVG-графики](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/title).
-   Если у `<title>` есть свойство [`itemProp`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/itemprop), то особого поведения не будет, потому что в этом случае он представляет не заголовок документа, а метаданные о конкретной части страницы.

!!!warning ""

    Одновременно отображается только один `<title>`. Если несколько компонентов рендерят тег `<title>` одновременно, React поместит все эти заголовки в голову документа. В этом случае поведение браузеров и поисковых систем будет неопределенным.

## Использование {#usage}

### Set the document title {#set-the-document-title}

Отрисуйте компонент `<title>` из любого компонента с текстом в качестве дочерних элементов. React поместит DOM-узел `<title>` в документ `<head>`.

=== "App.js"

    ```js
    import ShowRenderedHTML from './ShowRenderedHTML.js';

    export default function ContactUsPage() {
    	return (
    		<ShowRenderedHTML>
    			<title>My Site: Contact Us</title>
    			<h1>Contact Us</h1>
    			<p>Email us at support@example.com</p>
    		</ShowRenderedHTML>
    	);
    }
    ```

=== "ShowRenderedHTML.js"

    ```js
    import { renderToStaticMarkup } from 'react-dom/server';
    import formatHTML from './formatHTML.js';

    export default function ShowRenderedHTML({ children }) {
    	const markup = renderToStaticMarkup(
    		<html>
    			<head />
    			<body>{children}</body>
    		</html>
    	);
    	return (
    		<>
    			<h1>Rendered HTML:</h1>
    			<pre>{formatHTML(markup)}</pre>
    		</>
    	);
    }
    ```

=== "CodeSandbox"

    <iframe src="https://codesandbox.io/embed/9r9nsh?view=Editor+%2B+Preview&module=%2Fsrc%2FShowRenderedHTML.js" style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;" title="silly-joana-9r9nsh" allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking" sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"></iframe>

### Использование переменных в заголовке {#use-variables-in-the-title}

Дочерние элементы компонента `<title>` должны быть одной строкой текста. (Или одно число, или один объект с методом `toString`.) Это может быть неочевидно, но использование фигурных скобок JSX выглядит следующим образом:

```js
// 🔴 Problem: This is not a single string
<title>Results page {pageNumber}</title>
```

... фактически приводит к тому, что компонент `<title>` получает в качестве своих дочерних элементов массив из двух элементов (строка `" Results page"` и значение `pageNumber`). Это приведет к ошибке. Вместо этого используйте строковую интерполяцию для передачи `<title>` одной строки:

```js
<title>{`Results page ${pageNumber}`}</title>
```

<small>:material-information-outline: Источник &mdash; [https://react.dev/reference/react-dom/components/title](https://react.dev/reference/react-dom/components/title)</small>

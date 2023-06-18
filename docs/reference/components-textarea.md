# &lt;textarea&gt;

Встроенный компонент браузера [`<textarea>`](https://hcdev.ru/html/textarea/) позволяет отображать многострочный текстовый ввод.

```js
<textarea />
```

## Описание

### `<textarea>`

Чтобы отобразить текстовую область, создайте компонент [встроенный в браузер `<textarea>`](https://hcdev.ru/html/textarea/).

```js
<textarea name="postContent" />
```

#### Свойства

`<textarea>` поддерживает все [общие пропсы элементов](components-common.md#props)

Вы можете сделать текстовую область управляемой, передав ей пропс `value`:

-   `value`: Строка. Управляет текстом внутри текстовой области.

Когда вы передаете `value`, вы также должны передать обработчик `onChange`, который обновляет переданное значение.

Если ваша `<textarea>` является неконтролируемой, вы можете передать вместо нее параметр `defaultValue`:

-   `defaultValue`: Строка. Определяет начальное значение для текстовой области.

Эти пропсы `<textarea>` актуальны как для неконтролируемых, так и для контролируемых текстовых областей:

-   [`autoComplete`](https://hcdev.ru/html/textarea/#autocomplete): Либо `on`, либо `off`. Определяет поведение автозаполнения.
-   [`autoFocus`](https://hcdev.ru/html/textarea/#autofocus): Булево значение. Если значение `true`, React будет фокусировать элемент на монтировании.
-   `children`: `<textarea>` не принимает дочерних элементов. Для установки начального значения используйте `defaultValue`.
-   [`cols`](https://hcdev.ru/html/textarea/#cols): Число. Задает ширину по умолчанию в средних значениях ширины символов. По умолчанию `20`.
-   [`disabled`](https://hcdev.ru/html/textarea/#disabled): Булево значение. Если значение `true`, ввод не будет интерактивным и будет отображаться затемненным.
-   [`form`](https://hcdev.ru/html/textarea/#form): Строка. Указывает `id` `<form>`, к которой принадлежит этот ввод. Если опущено, то это ближайшая родительская форма.
-   [`maxLength`](https://hcdev.ru/html/textarea/#maxlength): Число. Определяет максимальную длину текста.
-   [`minLength`](https://hcdev.ru/html/textarea/#minlength): Число. Определяет минимальную длину текста.
-   [`name`](https://hcdev.ru/html/textarea/#name): Строка. Задает имя для этого ввода, который передается вместе с формой.
-   `onChange`: Функция обработчика [`event`](components-common.md#event-handler). Требуется для управляемых текстовых областей. Срабатывает немедленно, когда значение ввода изменяется пользователем (например, срабатывает при каждом нажатии клавиши). Поведение аналогично браузерному событию [`input`](https://developer.mozilla.org/docs/Web/API/HTMLElement/input_event).
-   `onChangeCapture`: Версия `onChange`, которая срабатывает в [фазе захвата](../learn/responding-to-events.md#capture-phase-events).
-   [`onInput`](https://developer.mozilla.org/docs/Web/API/HTMLElement/input_event): Функция обработчика [`event`](components-common.md#event-handler). функция. Срабатывает немедленно, когда значение изменяется пользователем. По историческим причинам в React идиоматично использовать `onChange`, которая работает аналогично.
-   `onInputCapture`: Версия `onInput`, которая срабатывает в [фазе захвата](../learn/responding-to-events.md#capture-phase-events).
-   [`onInvalid`](https://developer.mozilla.org/docs/Web/API/HTMLInputElement/invalid_event): Функция обработчика [`event`](components-common.md#event-handler). Срабатывает, если ввод не прошел валидацию при отправке формы. В отличие от встроенного события `invalid`, событие React `onInvalid` всплывает.
-   `onInvalidCapture`: Версия `onInvalid`, которая срабатывает в [фазе захвата.](../learn/responding-to-events.md#capture-phase-events)
-   [`onSelect`](https://developer.mozilla.org/docs/Web/API/HTMLTextAreaElement/select_event): Функция обработчика [`Event` handler](components-common.md#event-handler). Срабатывает после изменения выделения внутри `<textarea>`. В React событие `onSelect` расширяется, чтобы также срабатывать при пустом выделении и при редактировании (которое может повлиять на выделение).
-   `onSelectCapture`: Версия `onSelect`, которая срабатывает в [фазе захвата.](../learn/responding-to-events.md#capture-phase-events)
-   [`placeholder`](https://hcdev.ru/html/textarea/#placeholder): Строка. Отображается затемненным цветом, когда значение текстовой области пусто.
-   [`readOnly`](https://hcdev.ru/html/textarea/#readonly): Булево значение. Если значение `true`, текстовая область не редактируется пользователем.
-   [`required`](https://hcdev.ru/html/textarea/#required): Булево значение. Если `true`, то для отправки формы необходимо указать значение.
-   [`rows`](https://hcdev.ru/html/textarea/#rows): Число. Задает высоту по умолчанию в средних высотах символов. По умолчанию `2`.
-   [`wrap`](https://hcdev.ru/html/textarea/#wrap): Либо `'hard'`, либо `'soft'`, либо `'off'`. Определяет, как должен быть обернут текст при отправке формы.

#### Ограничения

-   Передача дочерних элементов типа `<textarea>something</textarea>` не допускается. Используйте `defaultValue` для начального содержимого.
-   Если текстовая область получает строковое свойство `value`, она будет рассматриваться как управляемая.
-   Текстовая область не может быть одновременно управляемой и неуправляемой.
-   Текстовая область не может переключаться между управляемой и неуправляемой в течение своего существования.
-   Каждая управляемая текстовая область должна иметь обработчик события `onChange`, который синхронно обновляет ее базовое значение.

## Использование

### Отображение текстовой области

Рендеринг `<textarea>` отображает текстовую область. Вы можете указать ее размер по умолчанию с помощью атрибутов [`rows`](https://hcdev.ru/html/textarea/#rows) и [`cols`](https://hcdev.ru/html/textarea/#cols), но по умолчанию пользователь сможет изменить ее размер. Чтобы отключить изменение размера, в CSS можно указать `resize: none`.

```js
export default function NewPost() {
    return (
        <label>
            Write your post:
            <textarea
                name="postContent"
                rows={4}
                cols={40}
            />
        </label>
    );
}
```

### Предоставление метки для текстовой области

Обычно вы помещаете каждую `<textarea>` внутри тега [`<label>`](https://hcdev.ru/html/label/). Это сообщает браузеру, что данная метка связана с этой текстовой областью. Когда пользователь нажимает на метку, браузер фокусирует текстовую область. Это также необходимо для обеспечения доступности: программа чтения с экрана будет объявлять надпись на ярлыке, когда пользователь наводит фокус на текстовую область.

Если вы не можете вложить `<textarea>` в `<label>`, свяжите их, передав одинаковый ID в `<textarea id>` и [`<label htmlFor>`.](https://developer.mozilla.org/docs/Web/API/HTMLLabelElement/htmlFor) Чтобы избежать конфликтов между экземплярами одного компонента, создайте такой ID с помощью [`useId`.](useId.md)

```js
import { useId } from 'react';

export default function Form() {
    const postTextAreaId = useId();
    return (
        <>
            <label htmlFor={postTextAreaId}>
                Write your post:
            </label>
            <textarea
                id={postTextAreaId}
                name="postContent"
                rows={4}
                cols={40}
            />
        </>
    );
}
```

### Предоставление начального значения для текстовой области

Вы можете опционально указать начальное значение для текстовой области. Передайте его как строку `defaultValue`.

```js
export default function EditPost() {
    return (
        <label>
            Edit your post:
            <textarea
                name="postContent"
                defaultValue="I really enjoyed biking yesterday!"
                rows={4}
                cols={40}
            />
        </label>
    );
}
```

!!!note ""

    В отличие от HTML, передача начального текста типа `<textarea>Some content</textarea>` не поддерживается.

### Чтение значения текстовой области при отправке формы

Добавьте [`<form>`](https://hcdev.ru/html/form/) вокруг вашей текстовой области с [`<button type="submit">`](https://hcdev.ru/html/button/) внутри. Это вызовет обработчик события `<form onSubmit>`. По умолчанию браузер отправит данные формы на текущий URL и обновит страницу. Вы можете отменить это поведение, вызвав `e.preventDefault()`. Считайте данные формы с помощью [`new FormData(e.target)`](https://developer.mozilla.org/docs/Web/API/FormData).

```js
export default function EditPost() {
    function handleSubmit(e) {
        // Prevent the browser from reloading the page
        e.preventDefault();

        // Read the form data
        const form = e.target;
        const formData = new FormData(form);

        // You can pass formData as a fetch body directly:
        fetch('/some-api', {
            method: form.method,
            body: formData,
        });

        // Or you can work with it as a plain object:
        const formJson = Object.fromEntries(
            formData.entries()
        );
        console.log(formJson);
    }

    return (
        <form method="post" onSubmit={handleSubmit}>
            <label>
                Post title:{' '}
                <input
                    name="postTitle"
                    defaultValue="Biking"
                />
            </label>
            <label>
                Edit your post:
                <textarea
                    name="postContent"
                    defaultValue="I really enjoyed biking yesterday!"
                    rows={4}
                    cols={40}
                />
            </label>
            <hr />
            <button type="reset">Reset edits</button>
            <button type="submit">Save post</button>
        </form>
    );
}
```

!!!note ""

    Дайте `name` вашему `<textarea>`, например `<textarea name="postContent" />`. Указанное вами `name` будет использоваться в качестве ключа в данных формы, например `{ postContent: "Ваш пост" }`.

!!!note ""

    По умолчанию _любая_ `<кнопка>` внутри `<формы>` отправит ее. Это может быть неожиданно! Если у вас есть собственный пользовательский компонент React `Button`, подумайте о возврате [`<button type="button">`](https://hcdev.ru/html/button/) вместо `<button>`. Затем, чтобы быть явным, используйте `<button type="submit">` для кнопок, которые _должны_ отправлять форму.

### Управление текстовой областью с помощью переменной состояния

Текстовая область типа `<textarea />` является _неуправляемой._ Даже если вы передадите начальное значение, например `<textarea defaultValue="Initial text" />`, ваш JSX определяет только начальное значение, а не значение прямо сейчас.

**Чтобы отобразить _управляемую_ текстовую область, передайте ей свойство `value`.** React заставит текстовую область всегда иметь переданное вами `value`. Обычно вы управляете текстовой областью, объявляя [переменную состояния:](useState.md)s

```js
function NewPost() {
    const [postContent, setPostContent] = useState(''); // Declare a state variable...
    // ...
    return (
        <textarea
            value={postContent} // ...force the input's value to match the state variable...
            onChange={(e) => setPostContent(e.target.value)} // ... and update the state variable on any edits!
        />
    );
}
```

Это полезно, если вы хотите перерисовывать часть пользовательского интерфейса в ответ на каждое нажатие клавиши.

=== "App.js"

    ```js
    import { useState } from 'react';
    import MarkdownPreview from './MarkdownPreview.js';

    export default function MarkdownEditor() {
    	const [postContent, setPostContent] = useState(
    		'_Hello,_ **Markdown**!'
    	);
    	return (
    		<>
    			<label>
    				Enter some markdown:
    				<textarea
    					value={postContent}
    					onChange={(e) =>
    						setPostContent(e.target.value)
    					}
    				/>
    			</label>
    			<hr />
    			<MarkdownPreview markdown={postContent} />
    		</>
    	);
    }
    ```

=== "MarkdownPreview.js"

    ```js
    import { Remarkable } from 'remarkable';

    const md = new Remarkable();

    export default function MarkdownPreview({ markdown }) {
    	const renderedHTML = md.render(markdown);
    	return (
    		<div
    			dangerouslySetInnerHTML={{
    				__html: renderedHTML,
    			}}
    		/>
    	);
    }
    ```

!!!note ""

    **Если вы передадите `value` без `onChange`, в текстовой области будет невозможно набирать текст.** Когда вы управляете текстовой областью, передавая ей некоторое `value`, вы _принуждаете_ ее всегда иметь то значение, которое вы передали. Поэтому если вы передадите переменную состояния в качестве `значения`, но забудете синхронно обновить эту переменную состояния в обработчике события `onChange`, React будет возвращать текстовую область после каждого нажатия клавиши к указанному вами `значению`.

## Устранение неполадок

### Моя текстовая область не обновляется, когда я ввожу текст

Если вы отображаете текстовую область с `value`, но без `onChange`, вы увидите ошибку в консоли:

```js
// 🔴 Bug: controlled text area with no onChange handler
<textarea value={something} />
```

!!!danger "Ошибка"

    Вы предоставили свойство `value` полю формы без обработчика `onChange`. Это приведет к тому, что поле будет доступно только для чтения. Если поле должно быть изменяемым, используйте `defaultValue`. В противном случае установите либо `onChange`, либо `readOnly`.

Как следует из сообщения об ошибке, если вы хотите указать только _начальное_ значение, передайте `defaultValue` вместо этого:

```js
// ✅ Good: uncontrolled text area with an initial value
<textarea defaultValue={something} />
```

Если вы хотите управлять этой текстовой областью с помощью переменной состояния, укажите обработчик `onChange`:

```js
// ✅ Good: controlled text area with onChange
<textarea
    value={something}
    onChange={(e) => setSomething(e.target.value)}
/>
```

Если значение намеренно предназначено только для чтения, добавьте параметр `readOnly`, чтобы подавить ошибку:

```js
// ✅ Good: readonly controlled text area without on change
<textarea value={something} readOnly={true} />
```

### Моя текстовая область переходит в начало при каждом нажатии клавиши

Если вы управляете текстовой областью, вы должны обновить ее переменную состояния до значения текстовой области из DOM во время `onChange`.

Вы не можете обновить ее до значения, отличного от `e.target.value`:

```js
function handleChange(e) {
    // 🔴 Bug: updating an input to something other than e.target.value
    setFirstName(e.target.value.toUpperCase());
}
```

Вы также не можете обновлять его асинхронно:

```js
function handleChange(e) {
    // 🔴 Bug: updating an input asynchronously
    setTimeout(() => {
        setFirstName(e.target.value);
    }, 100);
}
```

Чтобы исправить свой код, обновите его синхронно с `e.target.value`:

```js
function handleChange(e) {
    // ✅ Updating a controlled input to e.target.value synchronously
    setFirstName(e.target.value);
}
```

Если это не устраняет проблему, возможно, текстовая область удаляется и добавляется из DOM при каждом нажатии клавиши. Это может произойти, если вы случайно [сбрасываете состояние](../learn/preserving-and-resetting-state.md) при каждом повторном рендере. Например, это может произойти, если текстовая область или один из ее родителей всегда получает другой атрибут `key`, или если вы вложили определения компонентов (что не разрешено в React и заставляет "внутренний" компонент перемонтироваться при каждом рендере).

### Я получаю ошибку: "Компонент изменяет неконтролируемый вход на контролируемый"

Если вы передаете компоненту `value`, оно должно оставаться строкой в течение всего времени его работы.

Вы не можете сначала передать `value={undefined}`, а затем передать `value="some string"`, потому что React не будет знать, хотите ли вы, чтобы компонент был неконтролируемым или контролируемым. Управляемый компонент всегда должен получать строковое `value`, а не `null` или `undefined`.

Если ваше `value` приходит из API или переменной состояния, оно может быть инициализировано в `null` или `undefined`. В этом случае либо изначально установите его в пустую строку (`''`), либо передайте `value={someValue ?? ''}`, чтобы убедиться, что `value` является строкой.

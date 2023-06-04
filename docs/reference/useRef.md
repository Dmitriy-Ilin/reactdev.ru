# useRef

**`useRef`** - это хук React, позволяющий ссылаться на значение, которое не нужно для рендеринга.

<!-- 0001.part.md -->

```js
const ref = useRef(initialValue);
```

## Описание

### `useRef(initialValue)`

Вызовите `useRef` на верхнем уровне вашего компонента, чтобы объявить [ссылку](../learn/referencing-values-with-refs.md)

<!-- 0003.part.md -->

```js
import { useRef } from 'react';

function MyComponent() {
    const intervalRef = useRef(0);
    const inputRef = useRef(null);
    // ...
}
```

#### Параметры

-   `initialValue`: Значение, которое вы хотите, чтобы свойство `current` объекта ref было первоначальным. Это может быть значение любого типа. Этот аргумент игнорируется после первоначального рендеринга.

#### Возвращает

Функция `useRef` возвращает объект с одним свойством:

-   `current`: Изначально оно установлено на `initialValue`, которое вы передали. Позже вы можете установить его в другое значение. Если вы передадите объект ref в React как атрибут `ref` узла JSX, React установит его свойство `current`.

При следующем рендере `useRef` вернет тот же объект.

#### Предупреждения

-   Вы можете изменить свойство `ref.current`. В отличие от состояния, оно является изменяемым. Однако, если оно содержит объект, который используется для рендеринга (например, часть вашего состояния), то не стоит мутировать этот объект.
-   Когда вы изменяете свойство `ref.current`, React не перерисовывает ваш компонент. React не знает, когда вы изменяете его, потому что ref - это обычный объект JavaScript.
-   Не записывайте _или читайте_ `ref.current` во время рендеринга, кроме инициализации. Это делает поведение вашего компонента непредсказуемым.
-   В строгом режиме React будет **вызывать функцию вашего компонента дважды**, чтобы помочь вам найти случайные примеси. Это поведение только для разработки и не влияет на производство. Каждый объект ref будет создан дважды, но одна из версий будет отброшена. Если ваша функция компонента является чистой (как и должно быть), это не должно повлиять на поведение.

## Использование

### Ссылка на значение с помощью ссылки

Вызовите `useRef` на верхнем уровне вашего компонента, чтобы объявить одну или более [ссылок](../learn/referencing-values-with-refs.md).

<!-- 0005.part.md -->

```js
import { useRef } from 'react';

function Stopwatch() {
    const intervalRef = useRef(0);
    // ...
}
```

<!-- 0006.part.md -->

`useRef` возвращает объект ref с одним `current` property, первоначально установленным на начальное значение, которое вы указали.

При последующих рендерах `useRef` будет возвращать тот же объект. Вы можете изменить его свойство `current`, чтобы сохранить информацию и прочитать ее позже. Это может напомнить вам [state](useState.md), но есть важное отличие.

**Изменение ссылки не вызывает повторного рендеринга.** Это означает, что ссылки идеально подходят для хранения информации, которая не влияет на визуальный вывод вашего компонента. Например, если вам нужно хранить [ID интервала](https://developer.mozilla.org/docs/Web/API/setInterval) и получить его позже, вы можете поместить его в ссылку. Чтобы обновить значение внутри ссылки, вам нужно вручную изменить ее `current` свойство:

<!-- 0007.part.md -->

```js
function handleStartClick() {
    const intervalId = setInterval(() => {
        // ...
    }, 1000);
    intervalRef.current = intervalId;
}
```

<!-- 0008.part.md -->

Позже вы можете прочитать ID этого интервала из реферера, чтобы вызвать [clear that interval](https://developer.mozilla.org/docs/Web/API/clearInterval):

<!-- 0009.part.md -->

```js
function handleStopClick() {
    const intervalId = intervalRef.current;
    clearInterval(intervalId);
}
```

<!-- 0010.part.md -->

Используя ссылку, вы гарантируете, что:

-   Вы можете **сохранять информацию** между повторными рендерами (в отличие от обычных переменных, которые сбрасываются при каждом рендере).
-   Ее изменение **не вызывает повторного рендеринга** (в отличие от переменных состояния, которые вызывают повторный рендеринг).
-   Информация **является локальной** для каждой копии вашего компонента (в отличие от внешних переменных, которые являются общими).

Изменение ссылки не вызывает повторного рендеринга, поэтому ссылки не подходят для хранения информации, которую вы хотите вывести на экран. Для этого лучше использовать состояние. Подробнее о [выбор между `useRef` и `useState`](../learn/referencing-values-with-refs.md)

### Примеры ссылок на значение с помощью useRef

#### 1. Счетчик кликов

Этот компонент использует ссылку для отслеживания того, сколько раз была нажата кнопка. Обратите внимание, что здесь можно использовать ссылку вместо state, потому что счетчик кликов считывается и записывается только в обработчике событий.

<!-- 0011.part.md -->

```js
import { useRef } from 'react';

export default function Counter() {
    let ref = useRef(0);

    function handleClick() {
        ref.current = ref.current + 1;
        alert('You clicked ' + ref.current + ' times!');
    }

    return <button onClick={handleClick}>Click me!</button>;
}
```

<!-- 0012.part.md -->

Если вы покажете `{ref.current}` в JSX, число не будет обновляться по щелчку. Это происходит потому, что установка `ref.current` не вызывает повторного рендеринга. Вместо этого информация, которая используется для рендеринга, должна быть состоянием.

#### 2. Секундомер

В этом примере используется комбинация state и refs. И `startTime` и `now` являются переменными состояния, потому что они используются для рендеринга. Но нам также нужно хранить [ID интервала](https://developer.mozilla.org/docs/Web/API/setInterval), чтобы мы могли остановить интервал по нажатию кнопки. Поскольку идентификатор интервала не используется для рендеринга, целесообразно хранить его в ссылке и обновлять вручную.

<!-- 0013.part.md -->

```js
import { useState, useRef } from 'react';

export default function Stopwatch() {
    const [startTime, setStartTime] = useState(null);
    const [now, setNow] = useState(null);
    const intervalRef = useRef(null);

    function handleStart() {
        setStartTime(Date.now());
        setNow(Date.now());

        clearInterval(intervalRef.current);
        intervalRef.current = setInterval(() => {
            setNow(Date.now());
        }, 10);
    }

    function handleStop() {
        clearInterval(intervalRef.current);
    }

    let secondsPassed = 0;
    if (startTime != null && now != null) {
        secondsPassed = (now - startTime) / 1000;
    }

    return (
        <>
            <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
            <button onClick={handleStart}>Start</button>
            <button onClick={handleStop}>Stop</button>
        </>
    );
}
```

!!!warning ""

    **Не пишите и не читайте `ref.current` во время рендеринга.**.

    React ожидает, что тело вашего компонента [ведет себя как чистая функция](../learn/keeping-components-pure.md):

    -   Если входные данные ([props](../learn/passing-props-to-a-component.md), [state](../learn/state-a-components-memory.md) и [context](../learn/passing-data-deeply-with-context.md)) одинаковы, оно должно возвращать точно такой же JSX.
    -   Вызов в другом порядке или с другими аргументами не должен влиять на результаты других вызовов.

    Чтение или запись ссылки **во время рендеринга** нарушает эти ожидания.

    ```js
    function MyComponent() {
    	// ...
    	// 🚩 Don't write a ref during rendering
    	myRef.current = 123;
    	// ...
    	// 🚩 Don't read a ref during rendering
    	return <h1>{myOtherRef.current}</h1>;
    }
    ```

    Вместо этого вы можете читать или записывать ссылки **из обработчиков событий или эффектов**.

    ```js
    function MyComponent() {
    	// ...
    	useEffect(() => {
    		// ✅ You can read or write refs in effects
    		myRef.current = 123;
    	});
    	// ...
    	function handleClick() {
    		// ✅ You can read or write refs in event handlers
    		doSomething(myOtherRef.current);
    	}
    	// ...
    }
    ```

    Если вам _нужно_ прочитать [или записать](useState.md) что-то во время рендеринга, [используйте state](useState.md) вместо этого.

    Если вы нарушите эти правила, ваш компонент может продолжать работать, но большинство новых функций, которые мы добавляем в React, будут опираться на эти ожидания. Подробнее о [соблюдении чистоты компонентов](../learn/keeping-components-pure.md)

### Манипулирование DOM с помощью ссылки

Особенно часто ссылка используется для манипуляций с [DOM](https://developer.mozilla.org/docs/Web/API/HTML_DOM_API). React имеет встроенную поддержку для этого.

Сначала объявите ref object с начальным значением равным `null`:

<!-- 0019.part.md -->

```js
import { useRef } from 'react';

function MyComponent() {
    const inputRef = useRef(null);
    // ...
}
```

<!-- 0020.part.md -->

Затем передайте ваш объект `ref` в качестве атрибута `ref` в JSX узла DOM, которым вы хотите манипулировать:

<!-- 0021.part.md -->

```js
// ...
return <input ref={inputRef} />;
```

<!-- 0022.part.md -->

После того как React создаст DOM-узел и поместит его на экран, React установит свойство `current` вашего объекта ref на этот DOM-узел. Теперь вы можете получить доступ к DOM-узлу `input` и вызвать такие методы, как [`focus()`](https://developer.mozilla.org/docs/Web/API/HTMLElement/focus):

<!-- 0023.part.md -->

```js
function handleClick() {
    inputRef.current.focus();
}
```

<!-- 0024.part.md -->

React установит свойство `current` обратно в `null`, когда узел будет удален с экрана.

Подробнее о [манипулирование DOM с помощью ссылок](../learn/manipulating-the-dom-with-refs.md)

### Примеры манипулирования DOM с useRef

#### 1. Фокусировка текстового ввода

В этом примере нажатие на кнопку фокусирует ввод:

<!-- 0025.part.md -->

```js
import { useRef } from 'react';

export default function Form() {
    const inputRef = useRef(null);

    function handleClick() {
        inputRef.current.focus();
    }

    return (
        <>
            <input ref={inputRef} />
            <button onClick={handleClick}>
                Focus the input
            </button>
        </>
    );
}
```

#### 2. Прокрутка изображения в режиме просмотра

В этом примере нажатие на кнопку прокручивает изображение в окне просмотра. Он использует ссылку на DOM-узел list, а затем вызывает API DOM [`querySelectorAll`](https://developer.mozilla.org/docs/Web/API/Document/querySelectorAll), чтобы найти изображение, которое мы хотим прокрутить.

<!-- 0027.part.md -->

<div markdown style="max-height: 400px; overflow-y: auto;">

```js
import { useRef } from 'react';

export default function CatFriends() {
    const listRef = useRef(null);

    function scrollToIndex(index) {
        const listNode = listRef.current;
        // This line assumes a particular DOM structure:
        const imgNode = listNode.querySelectorAll(
            'li > img'
        )[index];
        imgNode.scrollIntoView({
            behavior: 'smooth',
            block: 'nearest',
            inline: 'center',
        });
    }

    return (
        <>
            <nav>
                <button onClick={() => scrollToIndex(0)}>
                    Tom
                </button>
                <button onClick={() => scrollToIndex(1)}>
                    Maru
                </button>
                <button onClick={() => scrollToIndex(2)}>
                    Jellylorum
                </button>
            </nav>
            <div>
                <ul ref={listRef}>
                    <li>
                        <img
                            src="https://placekitten.com/g/200/200"
                            alt="Tom"
                        />
                    </li>
                    <li>
                        <img
                            src="https://placekitten.com/g/300/200"
                            alt="Maru"
                        />
                    </li>
                    <li>
                        <img
                            src="https://placekitten.com/g/250/200"
                            alt="Jellylorum"
                        />
                    </li>
                </ul>
            </div>
        </>
    );
}
```

</div>

#### Воспроизведение и приостановка видео

Этот пример использует ссылку для вызова [`play()`](https://developer.mozilla.org/docs/Web/API/HTMLMediaElement/play) и [`pause()`](https://developer.mozilla.org/docs/Web/API/HTMLMediaElement/pause) на DOM-узле `<video>`.

<!-- 0031.part.md -->

```js
import { useState, useRef } from 'react';

export default function VideoPlayer() {
    const [isPlaying, setIsPlaying] = useState(false);
    const ref = useRef(null);

    function handleClick() {
        const nextIsPlaying = !isPlaying;
        setIsPlaying(nextIsPlaying);

        if (nextIsPlaying) {
            ref.current.play();
        } else {
            ref.current.pause();
        }
    }

    return (
        <>
            <button onClick={handleClick}>
                {isPlaying ? 'Pause' : 'Play'}
            </button>
            <video
                width="250"
                ref={ref}
                onPlay={() => setIsPlaying(true)}
                onPause={() => setIsPlaying(false)}
            >
                <source
                    src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
                    type="video/mp4"
                />
            </video>
        </>
    );
}
```

#### 4. Выставление ссылки на собственный компонент

Иногда вы можете захотеть позволить родительскому компоненту манипулировать DOM внутри вашего компонента. Например, вы пишете компонент `MyInput`, но хотите, чтобы родительский компонент мог фокусироваться на вводе (к которому родительский компонент не имеет доступа). Вы можете использовать комбинацию `useRef` для хранения ввода и [`forwardRef`](forwardRef.md) для передачи его родительскому компоненту. Читайте [подробное руководство](../learn/manipulating-the-dom-with-refs.md) здесь.

<!-- 0035.part.md -->

```js
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {
    return <input {...props} ref={ref} />;
});

export default function Form() {
    const inputRef = useRef(null);

    function handleClick() {
        inputRef.current.focus();
    }

    return (
        <>
            <MyInput ref={inputRef} />
            <button onClick={handleClick}>
                Focus the input
            </button>
        </>
    );
}
```

### Избегание воссоздания содержимого ссылки

React сохраняет начальное значение ссылки один раз и игнорирует его при последующих рендерах.

<!-- 0037.part.md -->

```js
function Video() {
    const playerRef = useRef(new VideoPlayer());
    // ...
}
```

<!-- 0038.part.md -->

Хотя результат `new VideoPlayer()` используется только для начального рендеринга, вы все равно вызываете эту функцию при каждом рендеринге. Это может быть расточительно, если создаются дорогие объекты.

Чтобы решить эту проблему, вы можете инициализировать ссылку следующим образом:

<!-- 0039.part.md -->

```js
function Video() {
    const playerRef = useRef(null);
    if (playerRef.current === null) {
        playerRef.current = new VideoPlayer();
    }
    // ...
}
```

<!-- 0040.part.md -->

Обычно запись или чтение `ref.current` во время рендеринга не допускается. Однако в данном случае это нормально, потому что результат всегда один и тот же, а условие выполняется только во время инициализации, поэтому оно полностью предсказуемо.

!!!note "Как избежать проверки нуля при инициализации useRef позже"

    Если вы используете программу проверки типов и не хотите всегда проверять `null`, вы можете попробовать такой шаблон:

    ```js
    function Video() {
    	const playerRef = useRef(null);

    	function getPlayer() {
    		if (playerRef.current !== null) {
    			return playerRef.current;
    		}
    		const player = new VideoPlayer();
    		playerRef.current = player;
    		return player;
    	}
    	// ...
    }
    ```

    Здесь сам `playerRef` является нулевым. Однако вы должны быть в состоянии убедить вашу программу проверки типов, что не существует случая, когда `getPlayer()` возвращает `null`. Затем используйте `getPlayer()` в обработчиках событий.

## Устранение неполадок

### Я не могу получить ссылку на пользовательский компонент

Если вы попытаетесь передать `ref` своему собственному компоненту следующим образом:

<!-- 0043.part.md -->

```js
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

<!-- 0044.part.md -->

Вы можете получить ошибку в консоли:

```
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

По умолчанию ваши собственные компоненты не передают ссылки на узлы DOM внутри них.

Чтобы исправить это, найдите компонент, к которому вы хотите получить ссылку:

<!-- 0045.part.md -->

```js
export default function MyInput({ value, onChange }) {
    return <input value={value} onChange={onChange} />;
}
```

<!-- 0046.part.md -->

А затем оберните его в [`forwardRef`](forwardRef.md) следующим образом:

<!-- 0047.part.md -->

```js
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {
    return (
        <input
            value={value}
            onChange={onChange}
            ref={ref}
        />
    );
});

export default MyInput;
```

<!-- 0048.part.md -->

Тогда родительский компонент сможет получить ссылку на него.

Подробнее о [доступе к узлам DOM другого компонента](../learn/manipulating-the-dom-with-refs.md)

<!-- 0049.part.md -->

## Ссылки

-   [https://react.dev/reference/react/useRef](https://react.dev/reference/react/useRef)

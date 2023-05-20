# Отделение событий от эффектов

Обработчики событий запускаются повторно только при повторном выполнении того же взаимодействия. В отличие от обработчиков событий, эффекты повторно синхронизируются, если какое-либо значение, которое они считывают, например реквизит или переменная состояния, отличается от того, что было во время последнего рендеринга. Иногда требуется сочетание обоих типов поведения: Эффект, который повторно запускается в ответ на некоторые значения, но не на другие. На этой странице вы узнаете, как это сделать.

-   Как выбрать между обработчиком событий и эффектом
-   Почему эффекты являются реактивными, а обработчики событий - нет
-   Что делать, если вы хотите, чтобы часть кода вашего Эффекта не была реактивной
-   Что такое события эффектов и как извлекать их из эффектов
-   Как считывать последние реквизиты и состояние из Эффектов с помощью Событий Эффектов

## Выбор между обработчиками событий и Эффектами {/_choosing-betosing-between-event-handlers-and-effects_/}

Во-первых, давайте вспомним разницу между обработчиками событий и Эффектами.

Представьте, что вы реализуете компонент чата. Ваши требования выглядят следующим образом:

1.  Ваш компонент должен автоматически подключаться к выбранному чату.
2.  Когда вы нажимаете кнопку "Отправить", он должен отправлять сообщение в чат.

Допустим, вы уже реализовали код для них, но не уверены, куда его поместить. Следует ли вам использовать обработчики событий или эффекты? Каждый раз, когда вам нужно ответить на этот вопрос, подумайте [_почему_ код должен выполняться](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events)

### Обработчики событий запускаются в ответ на конкретные взаимодействия {/_event-handlers-run-in-response-to-specific-interactions_/}

С точки зрения пользователя, отправка сообщения должна произойти _потому что_ была нажата конкретная кнопка "Отправить". Пользователь будет очень расстроен, если вы отправите его сообщение в любое другое время или по любой другой причине. Вот почему отправка сообщения должна быть обработчиком событий. Обработчики событий позволяют вам обрабатывать определенные взаимодействия:

<!-- 0001.part.md -->

```js
function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');
    // ...
    function handleSendClick() {
        sendMessage(message);
    }
    // ...
    return (
        <>
            <input
                value={message}
                onChange={(e) => setMessage(e.target.value)}
            />
            <button onClick={handleSendClick}>Send</button>;
        </>
    );
}
```

<!-- 0002.part.md -->

С обработчиком событий вы можете быть уверены, что `sendMessage(message)` будет _только_ если пользователь нажмет на кнопку.

### Эффекты запускаются всякий раз, когда необходима синхронизация {/_эффекты запускаются всякий раз, когда необходима синхронизация_/}

Вспомните, что вам также нужно, чтобы компонент был подключен к чату. Где находится этот код?

Причиной* для запуска этого кода не является какое-то конкретное взаимодействие. Неважно, почему или как пользователь перешел на экран чата. Теперь, когда они смотрят на него и могут взаимодействовать с ним, компонент должен оставаться подключенным к выбранному серверу чата. Даже если компонент чата был начальным экраном вашего приложения, и пользователь не выполнял никаких действий, вам *все равно\* потребуется подключение. Вот почему это Эффект:

<!-- 0003.part.md -->

```js
function ChatRoom({ roomId }) {
    // ...
    useEffect(() => {
        const connection = createConnection(
            serverUrl,
            roomId
        );
        connection.connect();
        return () => {
            connection.disconnect();
        };
    }, [roomId]);
    // ...
}
```

<!-- 0004.part.md -->

С помощью этого кода вы можете быть уверены, что всегда будет активное соединение с выбранным сервером чата, _независимо_ от конкретных действий, выполняемых пользователем. Открыл ли пользователь ваше приложение, выбрал другую комнату или перешел на другой экран и обратно, ваш эффект гарантирует, что компонент будет _оставаться синхронизированным_ с текущей выбранной комнатой и будет [переподключаться, когда это необходимо](/learn/lifecycle-of-reactive-effects#why-synchronization-may-need-to-happen-more-than-once).

<!-- 0005.part.md -->

```js
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');

    useEffect(() => {
        const connection = createConnection(
            serverUrl,
            roomId
        );
        connection.connect();
        return () => connection.disconnect();
    }, [roomId]);

    function handleSendClick() {
        sendMessage(message);
    }

    return (
        <>
            <h1>Welcome to the {roomId} room!</h1>
            <input
                value={message}
                onChange={(e) => setMessage(e.target.value)}
            />
            <button onClick={handleSendClick}>Send</button>
        </>
    );
}

export default function App() {
    const [roomId, setRoomId] = useState('general');
    const [show, setShow] = useState(false);
    return (
        <>
            <label>
                Choose the chat room:{' '}
                <select
                    value={roomId}
                    onChange={(e) =>
                        setRoomId(e.target.value)
                    }
                >
                    <option value="general">general</option>
                    <option value="travel">travel</option>
                    <option value="music">music</option>
                </select>
            </label>
            <button onClick={() => setShow(!show)}>
                {show ? 'Close chat' : 'Open chat'}
            </button>
            {show && <hr />}
            {show && <ChatRoom roomId={roomId} />}
        </>
    );
}
```

<!-- 0006.part.md -->

<!-- 0007.part.md -->

```js
export function sendMessage(message) {
    console.log('🔵 You sent: ' + message);
}

export function createConnection(serverUrl, roomId) {
    // A real implementation would actually connect to the server
    return {
        connect() {
            console.log(
                '✅ Connecting to "' +
                    roomId +
                    '" room at ' +
                    serverUrl +
                    '...'
            );
        },
        disconnect() {
            console.log(
                '❌ Disconnected from "' +
                    roomId +
                    '" room at ' +
                    serverUrl
            );
        },
    };
}
```

<!-- 0008.part.md -->

<!-- 0009.part.md -->

```css
input,
select {
    margin-right: 20px;
}
```

<!-- 0010.part.md -->

## Реактивные значения и реактивная логика {/_reactive-values-and-reactive-logic_/}

Интуитивно можно сказать, что обработчики событий всегда запускаются "вручную", например, при нажатии на кнопку. Эффекты, с другой стороны, являются "автоматическими": они запускаются и перезапускаются так часто, как это необходимо для сохранения синхронизации.

Есть более точный способ подумать об этом.

Реквизиты, состояния и переменные, объявленные в теле компонента, называются \<CodeStep step={2}\>реактивными значениями\</CodeStep\>. В этом примере `serverUrl` не является реактивным значением, но `roomId` и `message` являются таковыми. Они участвуют в потоке данных рендеринга:

<!-- 0011.part.md -->

```js
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');

    // ...
}
```

<!-- 0012.part.md -->

Реактивные значения, такие как эти, могут измениться в результате повторного рендеринга. Например, пользователь может изменить `сообщение` или выбрать другой `номер комнаты` в выпадающем списке. Обработчики событий и эффекты по-разному реагируют на изменения:

-   **Логика внутри обработчиков событий _не реактивна._** Она не будет запущена снова, пока пользователь не выполнит то же самое действие (например, щелчок) снова. Обработчики событий могут считывать реактивные значения, не "реагируя" на их изменения.
-   Если ваш эффект считывает реактивное значение, [вы должны указать его как зависимость](/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values), то, если при повторном рендеринге это значение изменится, React запустит логику вашего эффекта заново с новым значением.

Давайте вернемся к предыдущему примеру, чтобы проиллюстрировать эту разницу.

### Логика внутри обработчиков событий не является реактивной {/_logic-inside-event-handlers-is-not-reactive_/}

Взгляните на эту строку кода. Должна ли эта логика быть реактивной или нет?

<!-- 0013.part.md -->

```js
// ...
sendMessage(message);
// ...
```

<!-- 0014.part.md -->

С точки зрения пользователя, **изменение `message` **не\* означает, что он хочет отправить сообщение.\*\* Это означает только то, что пользователь набирает текст. Другими словами, логика, отправляющая сообщение, не должна быть реактивной. Она не должна запускаться снова только потому, что \<CodeStep step={2}\>реактивное значение\</CodeStep\> изменилось. Вот почему она должна находиться в обработчике событий:

<!-- 0015.part.md -->

```js
function handleSendClick() {
    sendMessage(message);
}
```

<!-- 0016.part.md -->

Обработчики событий не являются реактивными, поэтому `sendMessage(message)` будет выполняться только тогда, когда пользователь нажмет кнопку Send.

### Логика внутри Effects является реактивной {/_logic-inside-effects-is-reactive_/}

Теперь давайте вернемся к этим строкам:

<!-- 0017.part.md -->

```js
// ...
const connection = createConnection(serverUrl, roomId);
connection.connect();
// ...
```

<!-- 0018.part.md -->

С точки зрения пользователя, **изменение `roomId` **означает*, что он хочет подключиться к другой комнате.\*\* Другими словами, логика подключения к комнате должна быть реактивной. Вы *хотите\*, чтобы эти строки кода "шли в ногу" с \<CodeStep step={2}\>реактивным значением\</CodeStep\>, и запускались снова, если это значение изменилось. Вот почему это должно быть в Эффекте:

<!-- 0019.part.md -->

```js
useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
        connection.disconnect();
    };
}, [roomId]);
```

<!-- 0020.part.md -->

Эффекты являются реактивными, поэтому `createConnection(serverUrl, roomId)` и `connection.connect()` будут выполняться для каждого отдельного значения `roomId`. Ваш эффект сохраняет соединение чата синхронизированным с текущей выбранной комнатой.

## Извлечение нереактивной логики из эффектов {/_extracting-non-reactive-logic-out-of-effects_/}

Ситуация усложняется, когда вы хотите смешать реактивную логику с нереактивной.

Например, представьте, что вы хотите показать уведомление, когда пользователь подключается к чату. Вы считываете текущую тему (темную или светлую) из реквизита, чтобы показать уведомление нужного цвета:

<!-- 0021.part.md -->

```js
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    // ...
```

<!-- 0022.part.md -->

Однако `theme` является реактивным значением (оно может меняться в результате перерисовки), и [каждое реактивное значение, считываемое Эффектом, должно быть объявлено его зависимостью](/learn/lifecycle-of-reactive-effects#react-verifies-that-you-specified-every-reactive-value-as-a-dependency) Теперь вы должны указать `theme` как зависимость вашего Эффекта:

<!-- 0023.part.md -->

```js
function ChatRoom({ roomId, theme }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      showNotification('Connected!', theme);
    });
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId, theme]); // ✅ All dependencies declared
  // ...
```

<!-- 0024.part.md -->

Поиграйте с этим примером и посмотрите, сможете ли вы найти проблему в этом пользовательском опыте:

<!-- 0025.part.md -->

```js
{
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

<!-- 0026.part.md -->

<!-- 0027.part.md -->

```js
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
    useEffect(() => {
        const connection = createConnection(
            serverUrl,
            roomId
        );
        connection.on('connected', () => {
            showNotification('Connected!', theme);
        });
        connection.connect();
        return () => connection.disconnect();
    }, [roomId, theme]);

    return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
    const [roomId, setRoomId] = useState('general');
    const [isDark, setIsDark] = useState(false);
    return (
        <>
            <label>
                Choose the chat room:{' '}
                <select
                    value={roomId}
                    onChange={(e) =>
                        setRoomId(e.target.value)
                    }
                >
                    <option value="general">general</option>
                    <option value="travel">travel</option>
                    <option value="music">music</option>
                </select>
            </label>
            <label>
                <input
                    type="checkbox"
                    checked={isDark}
                    onChange={(e) =>
                        setIsDark(e.target.checked)
                    }
                />
                Use dark theme
            </label>
            <hr />
            <ChatRoom
                roomId={roomId}
                theme={isDark ? 'dark' : 'light'}
            />
        </>
    );
}
```

<!-- 0028.part.md -->

<!-- 0029.part.md -->

```js
export function createConnection(serverUrl, roomId) {
    // A real implementation would actually connect to the server
    let connectedCallback;
    let timeout;
    return {
        connect() {
            timeout = setTimeout(() => {
                if (connectedCallback) {
                    connectedCallback();
                }
            }, 100);
        },
        on(event, callback) {
            if (connectedCallback) {
                throw Error(
                    'Cannot add the handler twice.'
                );
            }
            if (event !== 'connected') {
                throw Error(
                    'Only "connected" event is supported.'
                );
            }
            connectedCallback = callback;
        },
        disconnect() {
            clearTimeout(timeout);
        },
    };
}
```

<!-- 0030.part.md -->

<!-- 0031.part.md -->

```js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
    Toastify({
        text: message,
        duration: 2000,
        gravity: 'top',
        position: 'right',
        style: {
            background:
                theme === 'dark' ? 'black' : 'white',
            color: theme === 'dark' ? 'white' : 'black',
        },
    }).showToast();
}
```

<!-- 0032.part.md -->

<!-- 0033.part.md -->

```css
label {
    display: block;
    margin-top: 10px;
}
```

<!-- 0034.part.md -->

При изменении `roomId` чат переподключается, как и следовало ожидать. Но поскольку `theme` также является зависимостью, чат _также_ переподключается каждый раз, когда вы переключаетесь между темной и светлой темой. Это не здорово\!

Другими словами, вы _не_ хотите, чтобы эта строка была реактивной, даже если она находится внутри Effect (который является реактивным):

<!-- 0035.part.md -->

```js
// ...
showNotification('Connected!', theme);
// ...
```

<!-- 0036.part.md -->

Вам нужен способ отделить эту нереактивную логику от реактивного Эффекта вокруг нее.

### Объявление события эффекта {/_declaring-an-effect-event_/}

\<Wip\>

Этот раздел описывает **экспериментальный API, который еще не был выпущен** в стабильной версии React.

\</Wip\>

Используйте специальный хук под названием [`useEffectEvent`](/reference/react/experimental_useEffectEvent), чтобы извлечь эту нереактивную логику из вашего Эффекта:

<!-- 0037.part.md -->

```js
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });
  // ...
```

<!-- 0038.part.md -->

Здесь `onConnected` называется _событием эффекта._ Это часть логики вашего эффекта, но она ведет себя гораздо больше как обработчик событий. Логика внутри него не является реактивной, и он всегда "видит" последние значения ваших реквизитов и состояния.

Теперь вы можете вызывать событие Эффекта `onConnected` изнутри вашего Эффекта:

<!-- 0039.part.md -->

```js
function ChatRoom({ roomId, theme }) {
  const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      onConnected();
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

<!-- 0040.part.md -->

Это решает проблему. Обратите внимание, что вам пришлось _удалить_ `onConnected` из списка зависимостей вашего Эффекта. **События эффектов не являются реактивными и должны быть исключены из зависимостей.**.

Проверьте, что новое поведение работает так, как вы ожидаете:

<!-- 0041.part.md -->

```js
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

<!-- 0042.part.md -->

<!-- 0043.part.md -->

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
    const onConnected = useEffectEvent(() => {
        showNotification('Connected!', theme);
    });

    useEffect(() => {
        const connection = createConnection(
            serverUrl,
            roomId
        );
        connection.on('connected', () => {
            onConnected();
        });
        connection.connect();
        return () => connection.disconnect();
    }, [roomId]);

    return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
    const [roomId, setRoomId] = useState('general');
    const [isDark, setIsDark] = useState(false);
    return (
        <>
            <label>
                Choose the chat room:{' '}
                <select
                    value={roomId}
                    onChange={(e) =>
                        setRoomId(e.target.value)
                    }
                >
                    <option value="general">general</option>
                    <option value="travel">travel</option>
                    <option value="music">music</option>
                </select>
            </label>
            <label>
                <input
                    type="checkbox"
                    checked={isDark}
                    onChange={(e) =>
                        setIsDark(e.target.checked)
                    }
                />
                Use dark theme
            </label>
            <hr />
            <ChatRoom
                roomId={roomId}
                theme={isDark ? 'dark' : 'light'}
            />
        </>
    );
}
```

<!-- 0044.part.md -->

<!-- 0045.part.md -->

```js
export function createConnection(serverUrl, roomId) {
    // A real implementation would actually connect to the server
    let connectedCallback;
    let timeout;
    return {
        connect() {
            timeout = setTimeout(() => {
                if (connectedCallback) {
                    connectedCallback();
                }
            }, 100);
        },
        on(event, callback) {
            if (connectedCallback) {
                throw Error(
                    'Cannot add the handler twice.'
                );
            }
            if (event !== 'connected') {
                throw Error(
                    'Only "connected" event is supported.'
                );
            }
            connectedCallback = callback;
        },
        disconnect() {
            clearTimeout(timeout);
        },
    };
}
```

<!-- 0046.part.md -->

<!-- 0047.part.md -->

```js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
    Toastify({
        text: message,
        duration: 2000,
        gravity: 'top',
        position: 'right',
        style: {
            background:
                theme === 'dark' ? 'black' : 'white',
            color: theme === 'dark' ? 'white' : 'black',
        },
    }).showToast();
}
```

<!-- 0048.part.md -->

<!-- 0049.part.md -->

```css
label {
    display: block;
    margin-top: 10px;
}
```

<!-- 0050.part.md -->

События эффектов можно считать очень похожими на обработчики событий. Основное различие заключается в том, что обработчики событий запускаются в ответ на действия пользователя, в то время как События эффектов запускаются вами из Эффектов. События эффектов позволяют вам "разорвать цепь" между реактивностью эффектов и кодом, который не должен быть реактивным.

### Чтение последних реквизитов и состояния с помощью событий эффектов {/_reading-latest-props-and-state-with-effect-events_/}

\<Wip\>

Этот раздел описывает **экспериментальный API, который еще не был выпущен** в стабильной версии React.

\</Wip\>

События Effect Events позволяют исправить многие паттерны, где может возникнуть соблазн подавить линтер зависимостей.

Например, у вас есть эффект для регистрации посещений страницы:

<!-- 0051.part.md -->

```js
function Page() {
    useEffect(() => {
        logVisit();
    }, []);
    // ...
}
```

<!-- 0052.part.md -->

Позже вы добавляете несколько маршрутов на свой сайт. Теперь ваш компонент `Page` получает реквизит `url` с текущим путем. Вы хотите передать `url` как часть вашего вызова `logVisit`, но линтер зависимостей жалуется:

<!-- 0053.part.md -->

```js
function Page({ url }) {
    useEffect(() => {
        logVisit(url);
    }, []); // 🔴 React Hook useEffect has a missing dependency: 'url'
    // ...
}
```

<!-- 0054.part.md -->

Подумайте, что вы хотите сделать с помощью кода. Вы _хотите_ регистрировать отдельные посещения для разных URL, поскольку каждый URL представляет собой отдельную страницу. Другими словами, этот вызов `logVisit` _должен_ быть реактивным по отношению к `url`. Вот почему в данном случае имеет смысл следовать указателю зависимостей и добавить `url` в качестве зависимости:

<!-- 0055.part.md -->

```js
function Page({ url }) {
    useEffect(() => {
        logVisit(url);
    }, [url]); // ✅ All dependencies declared
    // ...
}
```

<!-- 0056.part.md -->

Теперь предположим, что вы хотите включать количество товаров в корзине вместе с каждым посещением страницы:

<!-- 0057.part.md -->

```js
function Page({ url }) {
    const { items } = useContext(ShoppingCartContext);
    const numberOfItems = items.length;

    useEffect(() => {
        logVisit(url, numberOfItems);
    }, [url]); // 🔴 React Hook useEffect has a missing dependency: 'numberOfItems'
    // ...
}
```

<!-- 0058.part.md -->

Вы использовали `numberOfItems` внутри Effect, поэтому линтер просит вас добавить его в качестве зависимости. Однако, вы _не_ хотите, чтобы вызов `logVisit` был реактивным по отношению к `numberOfItems`. Если пользователь положил что-то в корзину, и `numberOfItems` изменилось, это _не означает_, что пользователь снова посетил страницу. Другими словами, _посещение страницы_ - это, в некотором смысле, "событие". Оно происходит в определенный момент времени.

Разделите код на две части:

<!-- 0059.part.md -->

```js
function Page({ url }) {
    const { items } = useContext(ShoppingCartContext);
    const numberOfItems = items.length;

    const onVisit = useEffectEvent((visitedUrl) => {
        logVisit(visitedUrl, numberOfItems);
    });

    useEffect(() => {
        onVisit(url);
    }, [url]); // ✅ All dependencies declared
    // ...
}
```

<!-- 0060.part.md -->

Здесь `onVisit` - это событие эффекта. Код внутри него не является реактивным. Вот почему вы можете использовать `numberOfItems` (или любое другое реактивное значение\!), не беспокоясь о том, что оно заставит окружающий код повторно выполняться при изменениях.

С другой стороны, сам Эффект остается реактивным. Код внутри Эффекта использует реквизит `url`, поэтому Эффект будет запускаться заново после каждого повторного рендеринга с другим `url`. Это, в свою очередь, вызовет событие эффекта `onVisit`.

В результате вы будете вызывать `logVisit` для каждого изменения `url`, и всегда считывать последнее `numberOfItems`. Однако, если `numberOfItems` изменится сам по себе, это не приведет к повторному выполнению кода.

Вам может быть интересно, можно ли вызвать функцию `onVisit()` без аргументов и прочитать `url` внутри нее:

<!-- 0061.part.md -->

```js
const onVisit = useEffectEvent(() => {
    logVisit(url, numberOfItems);
});

useEffect(() => {
    onVisit();
}, [url]);
```

<!-- 0062.part.md -->

Это сработает, но лучше передавать `url` в событие Effect Event явно. ** Передавая `url` в качестве аргумента в событие Effect Event, вы говорите, что посещение страницы с другим `url` представляет собой отдельное "событие" с точки зрения пользователя.** `visitedUrl` является _частью_ произошедшего "события":

<!-- 0063.part.md -->

```js
const onVisit = useEffectEvent((visitedUrl) => {
    logVisit(visitedUrl, numberOfItems);
});

useEffect(() => {
    onVisit(url);
}, [url]);
```

<!-- 0064.part.md -->

Поскольку событие вашего эффекта явно "запрашивает" `visitedUrl`, теперь вы не сможете случайно удалить `url` из зависимостей эффекта. Если вы удалите зависимость `url` (в результате чего посещение разных страниц будет считаться за одно), линтер предупредит вас об этом. Вы хотите, чтобы `onVisit` был реактивным по отношению к `url`, поэтому вместо того, чтобы читать `url` внутри (где он не будет реактивным), вы передаете его _из_ вашего Эффекта.

Это становится особенно важным, если внутри Эффекта есть какая-то асинхронная логика:

<!-- 0065.part.md -->

```js
const onVisit = useEffectEvent((visitedUrl) => {
    logVisit(visitedUrl, numberOfItems);
});

useEffect(() => {
    setTimeout(() => {
        onVisit(url);
    }, 5000); // Delay logging visits
}, [url]);
```

<!-- 0066.part.md -->

Здесь `url` внутри `onVisit` соответствует _последнему_ `url` (который мог уже измениться), но `visitedUrl` соответствует `url`, который первоначально вызвал запуск этого Effect (и этого вызова `onVisit`).

#### Можно ли вместо этого подавить линтер зависимостей? {/_is-it-okay-to-suppress-the-dependency-linter-instead_/}

В существующих кодовых базах иногда можно встретить подавление правила lint следующим образом:

<!-- 0067.part.md -->

```js
function Page({ url }) {
    const { items } = useContext(ShoppingCartContext);
    const numberOfItems = items.length;

    useEffect(() => {
        logVisit(url, numberOfItems);
        // 🔴 Avoid suppressing the linter like this:
        // eslint-disable-next-line react-hooks/exhaustive-deps
    }, [url]);
    // ...
}
```

<!-- 0068.part.md -->

После того, как `useEffectEvent` станет стабильной частью React, мы рекомендуем **никогда не подавлять линтер**.

Первый недостаток подавления правила заключается в том, что React больше не будет предупреждать вас, когда ваш Эффект должен "отреагировать" на новую реактивную зависимость, которую вы добавили в свой код. В предыдущем примере вы добавили `url` к зависимостям _потому что_ React напомнил вам сделать это. Если вы отключите линтер, вы больше не будете получать такие напоминания для любых будущих правок этого Эффекта. Это приводит к ошибкам.

Вот пример запутанной ошибки, вызванной подавлением линтера. В этом примере функция `handleMove` должна прочитать текущее значение переменной состояния `canMove`, чтобы решить, должна ли точка следовать за курсором. Однако, `canMove` всегда `true` внутри `handleMove`.

Вы можете понять, почему?

<!-- 0069.part.md -->

```js
import { useState, useEffect } from 'react';

export default function App() {
    const [position, setPosition] = useState({
        x: 0,
        y: 0,
    });
    const [canMove, setCanMove] = useState(true);

    function handleMove(e) {
        if (canMove) {
            setPosition({ x: e.clientX, y: e.clientY });
        }
    }

    useEffect(() => {
        window.addEventListener('pointermove', handleMove);
        return () =>
            window.removeEventListener(
                'pointermove',
                handleMove
            );
        // eslint-disable-next-line react-hooks/exhaustive-deps
    }, []);

    return (
        <>
            <label>
                <input
                    type="checkbox"
                    checked={canMove}
                    onChange={(e) =>
                        setCanMove(e.target.checked)
                    }
                />
                The dot is allowed to move
            </label>
            <hr />
            <div
                style={{
                    position: 'absolute',
                    backgroundColor: 'pink',
                    borderRadius: '50%',
                    opacity: 0.6,
                    transform: `translate(${position.x}px, ${position.y}px)`,
                    pointerEvents: 'none',
                    left: -20,
                    top: -20,
                    width: 40,
                    height: 40,
                }}
            />
        </>
    );
}
```

<!-- 0070.part.md -->

<!-- 0071.part.md -->

```css
body {
    height: 200px;
}
```

<!-- 0072.part.md -->

Проблема с этим кодом заключается в подавлении линтера зависимостей. Если вы уберете подавление, то увидите, что этот Effect должен зависеть от функции `handleMove`. Это имеет смысл: `handleMove` объявлена внутри тела компонента, что делает ее реактивным значением. Каждое реактивное значение должно быть указано как зависимость, иначе оно может устареть со временем\!

Автор оригинального кода "соврал" React, сказав, что Effect не зависит (`[]`) ни от каких реактивных значений. Именно поэтому React не пересинхронизировал Effect после изменения `canMove` (и `handleMove` вместе с ним). Поскольку React не пересинхронизировал Effect, `handleMove`, прикрепленная в качестве слушателя, является функцией `handleMove`, созданной во время первоначального рендеринга. Во время первоначального рендера `canMove` было `true`, поэтому `handleMove` из первоначального рендера всегда будет видеть это значение.

\*_Если вы никогда не подавляете линтер, вы никогда не увидите проблем с устаревшими значениями_.

С `useEffectEvent` нет необходимости "врать" линтеру, и код работает так, как вы ожидаете:

<!-- 0073.part.md -->

```js
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

<!-- 0074.part.md -->

<!-- 0075.part.md -->

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function App() {
    const [position, setPosition] = useState({
        x: 0,
        y: 0,
    });
    const [canMove, setCanMove] = useState(true);

    const onMove = useEffectEvent((e) => {
        if (canMove) {
            setPosition({ x: e.clientX, y: e.clientY });
        }
    });

    useEffect(() => {
        window.addEventListener('pointermove', onMove);
        return () =>
            window.removeEventListener(
                'pointermove',
                onMove
            );
    }, []);

    return (
        <>
            <label>
                <input
                    type="checkbox"
                    checked={canMove}
                    onChange={(e) =>
                        setCanMove(e.target.checked)
                    }
                />
                The dot is allowed to move
            </label>
            <hr />
            <div
                style={{
                    position: 'absolute',
                    backgroundColor: 'pink',
                    borderRadius: '50%',
                    opacity: 0.6,
                    transform: `translate(${position.x}px, ${position.y}px)`,
                    pointerEvents: 'none',
                    left: -20,
                    top: -20,
                    width: 40,
                    height: 40,
                }}
            />
        </>
    );
}
```

<!-- 0076.part.md -->

<!-- 0077.part.md -->

```css
body {
    height: 200px;
}
```

<!-- 0078.part.md -->

Это не означает, что `useEffectEvent` - это _всегда_ правильное решение. Вы должны применять его только к тем строкам кода, которые вы не хотите, чтобы были реактивными. В приведенной выше песочнице вы не хотели, чтобы код Эффекта был реактивным в отношении `canMove`. Вот почему имело смысл извлечь событие эффекта.

Читайте [Удаление зависимостей эффектов](/learn/removing-effect-dependencies) о других правильных альтернативах подавления линтера.

### Ограничения событий эффектов {/_limitations-of-effect-events_/}

\<Wip\>

Этот раздел описывает **экспериментальный API, который еще не был выпущен** в стабильной версии React.

\</Wip\>

События эффектов очень ограничены в том, как вы можете их использовать:

-   **Вызывать их только изнутри Эффектов.**.
-   **Никогда не передавайте их другим компонентам или крючкам.**.

Например, не объявляйте и не передавайте событие эффекта следующим образом:

<!-- 0079.part.md -->

```js
function Timer() {
    const [count, setCount] = useState(0);

    const onTick = useEffectEvent(() => {
        setCount(count + 1);
    });

    useTimer(onTick, 1000); // 🔴 Avoid: Passing Effect Events

    return <h1>{count}</h1>;
}

function useTimer(callback, delay) {
    useEffect(() => {
        const id = setInterval(() => {
            callback();
        }, delay);
        return () => {
            clearInterval(id);
        };
    }, [delay, callback]); // Need to specify "callback" in dependencies
}
```

<!-- 0080.part.md -->

Вместо этого всегда объявляйте события эффектов непосредственно рядом с эффектами, которые их используют:

<!-- 0081.part.md -->

```js
function Timer() {
    const [count, setCount] = useState(0);
    useTimer(() => {
        setCount(count + 1);
    }, 1000);
    return <h1>{count}</h1>;
}

function useTimer(callback, delay) {
    const onTick = useEffectEvent(() => {
        callback();
    });

    useEffect(() => {
        const id = setInterval(() => {
            onTick(); // ✅ Good: Only called locally inside an Effect
        }, delay);
        return () => {
            clearInterval(id);
        };
    }, [delay]); // No need to specify "onTick" (an Effect Event) as a dependency
}
```

<!-- 0082.part.md -->

События эффектов - это нереактивные "куски" кода вашего эффекта. Они должны находиться рядом с использующим их Эффектом.

\<Recap\>

-   Обработчики событий запускаются в ответ на определенные взаимодействия.
-   Эффекты запускаются всякий раз, когда требуется синхронизация.
-   Логика внутри обработчиков событий не является реактивной.
-   Логика внутри эффектов реактивна.
-   Вы можете перенести нереактивную логику из Эффектов в События Эффектов.
-   Вызывайте события эффектов только из самих эффектов.
-   Не передавайте события эффектов другим компонентам или крючкам.

\</Recap\>

\<Проблемы\>

#### Исправление переменной, которая не обновляется {/_fix-a-variable-that-doesnt-update_/}

Этот компонент `Timer` хранит переменную состояния `count`, которая увеличивается каждую секунду. Значение, на которое она увеличивается, хранится в переменной состояния `increment`. Вы можете управлять переменной `increment` с помощью кнопок плюс и минус.

Однако, сколько бы раз вы ни нажали на кнопку с плюсом, счетчик все равно увеличивается на единицу каждую секунду. Что не так с этим кодом? Почему `increment` всегда равен `1` в коде Effect'а? Найдите ошибку и исправьте ее.

\<Hint\>

Чтобы исправить этот код, достаточно следовать правилам.

\</Hint\>

<!-- 0083.part.md -->

```js
import { useState, useEffect } from 'react';

export default function Timer() {
    const [count, setCount] = useState(0);
    const [increment, setIncrement] = useState(1);

    useEffect(() => {
        const id = setInterval(() => {
            setCount((c) => c + increment);
        }, 1000);
        return () => {
            clearInterval(id);
        };
        // eslint-disable-next-line react-hooks/exhaustive-deps
    }, []);

    return (
        <>
            <h1>
                Counter: {count}
                <button onClick={() => setCount(0)}>
                    Reset
                </button>
            </h1>
            <hr />
            <p>
                Every second, increment by:
                <button
                    disabled={increment === 0}
                    onClick={() => {
                        setIncrement((i) => i - 1);
                    }}
                >
                    –
                </button>
                <b>{increment}</b>
                <button
                    onClick={() => {
                        setIncrement((i) => i + 1);
                    }}
                >
                    +
                </button>
            </p>
        </>
    );
}
```

<!-- 0084.part.md -->

<!-- 0085.part.md -->

```css
button {
    margin: 10px;
}
```

<!-- 0086.part.md -->

\<Решение\>

Как обычно, когда вы ищете ошибки в Эффектах, начните с поиска подавлений линтера.

Если вы удалите комментарий о подавлении, React скажет вам, что код этого Эффекта зависит от `increment`, но вы "солгали" React, утверждая, что этот Эффект не зависит ни от каких реактивных значений (`[]`). Добавьте `increment` в массив зависимостей:

<!-- 0087.part.md -->

```js
import { useState, useEffect } from 'react';

export default function Timer() {
    const [count, setCount] = useState(0);
    const [increment, setIncrement] = useState(1);

    useEffect(() => {
        const id = setInterval(() => {
            setCount((c) => c + increment);
        }, 1000);
        return () => {
            clearInterval(id);
        };
    }, [increment]);

    return (
        <>
            <h1>
                Counter: {count}
                <button onClick={() => setCount(0)}>
                    Reset
                </button>
            </h1>
            <hr />
            <p>
                Every second, increment by:
                <button
                    disabled={increment === 0}
                    onClick={() => {
                        setIncrement((i) => i - 1);
                    }}
                >
                    –
                </button>
                <b>{increment}</b>
                <button
                    onClick={() => {
                        setIncrement((i) => i + 1);
                    }}
                >
                    +
                </button>
            </p>
        </>
    );
}
```

<!-- 0088.part.md -->

<!-- 0089.part.md -->

```css
button {
    margin: 10px;
}
```

<!-- 0090.part.md -->

Теперь, когда `increment` изменится, React пересинхронизирует ваш Effect, что перезапустит интервал.

\</Solution\>

#### Fix a freezing counter {/_fix-a-freezing-counter_/}

Этот компонент `Timer` хранит переменную состояния `count`, которая увеличивается каждую секунду. Значение, на которое она увеличивается, хранится в переменной состояния `increment`, которой вы можете управлять с помощью кнопок плюс и минус. Например, попробуйте нажать кнопку плюс девять раз и заметите, что теперь `count` увеличивается каждую секунду на десять, а не на один.

Есть небольшая проблема с этим пользовательским интерфейсом. Вы можете заметить, что если вы продолжаете нажимать на кнопки плюс или минус быстрее, чем один раз в секунду, то сам таймер как бы приостанавливается. Он возобновляется только после того, как пройдет секунда с момента последнего нажатия любой из кнопок. Выясните, почему это происходит, и устраните проблему, чтобы таймер тикал _каждую_ секунду без перерывов.

\< Подсказка\>

Похоже, что Effect, который устанавливает таймер, "реагирует" на значение `increment`. Действительно ли строка, которая использует текущее значение `increment` для вызова `setCount` должна быть реактивной?

\</Hint\>

<!-- 0091.part.md -->

```js
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

<!-- 0092.part.md -->

<!-- 0093.part.md -->

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function Timer() {
    const [count, setCount] = useState(0);
    const [increment, setIncrement] = useState(1);

    useEffect(() => {
        const id = setInterval(() => {
            setCount((c) => c + increment);
        }, 1000);
        return () => {
            clearInterval(id);
        };
    }, [increment]);

    return (
        <>
            <h1>
                Counter: {count}
                <button onClick={() => setCount(0)}>
                    Reset
                </button>
            </h1>
            <hr />
            <p>
                Every second, increment by:
                <button
                    disabled={increment === 0}
                    onClick={() => {
                        setIncrement((i) => i - 1);
                    }}
                >
                    –
                </button>
                <b>{increment}</b>
                <button
                    onClick={() => {
                        setIncrement((i) => i + 1);
                    }}
                >
                    +
                </button>
            </p>
        </>
    );
}
```

<!-- 0094.part.md -->

<!-- 0095.part.md -->

```css
button {
    margin: 10px;
}
```

<!-- 0096.part.md -->

\<Решение\>

Проблема в том, что код внутри Эффекта использует переменную состояния `increment`. Поскольку она зависит от вашего Эффекта, каждое изменение `increment` заставляет Эффект пересинхронизироваться, что приводит к очистке интервала. Если вы будете очищать интервал каждый раз, прежде чем он успеет сработать, то будет казаться, что таймер остановился.

Чтобы решить эту проблему, извлеките из Эффекта событие Эффекта `onTick`:

<!-- 0097.part.md -->

```js
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

<!-- 0098.part.md -->

<!-- 0099.part.md -->

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function Timer() {
    const [count, setCount] = useState(0);
    const [increment, setIncrement] = useState(1);

    const onTick = useEffectEvent(() => {
        setCount((c) => c + increment);
    });

    useEffect(() => {
        const id = setInterval(() => {
            onTick();
        }, 1000);
        return () => {
            clearInterval(id);
        };
    }, []);

    return (
        <>
            <h1>
                Counter: {count}
                <button onClick={() => setCount(0)}>
                    Reset
                </button>
            </h1>
            <hr />
            <p>
                Every second, increment by:
                <button
                    disabled={increment === 0}
                    onClick={() => {
                        setIncrement((i) => i - 1);
                    }}
                >
                    –
                </button>
                <b>{increment}</b>
                <button
                    onClick={() => {
                        setIncrement((i) => i + 1);
                    }}
                >
                    +
                </button>
            </p>
        </>
    );
}
```

<!-- 0100.part.md -->

<!-- 0101.part.md -->

```css
button {
    margin: 10px;
}
```

<!-- 0102.part.md -->

Поскольку `onTick` является событием Эффекта, код внутри него не является реактивным. Изменение `increment` не вызывает никаких Эффектов.

\</Solution\>

#### Исправление нерегулируемой задержки {/_fix-a-non-adjustable-delay_/}

В этом примере вы можете настроить интервальную задержку. Она хранится в переменной состояния `delay`, которая обновляется двумя кнопками. Однако, даже если вы будете нажимать кнопку "плюс 100 мс", пока `delay` не станет равной 1000 миллисекунд (то есть секунде), вы заметите, что таймер все равно увеличивается очень быстро (каждые 100 мс). Как будто ваши изменения `задержки` игнорируются. Найдите и исправьте ошибку.

\<Примечание\>

Код внутри Effect Events не является реактивным. Есть ли случаи, в которых вы _хотели бы_, чтобы вызов `setInterval` выполнялся повторно?

\</Hint\>

<!-- 0103.part.md -->

```js
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

<!-- 0104.part.md -->

<!-- 0105.part.md -->

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function Timer() {
    const [count, setCount] = useState(0);
    const [increment, setIncrement] = useState(1);
    const [delay, setDelay] = useState(100);

    const onTick = useEffectEvent(() => {
        setCount((c) => c + increment);
    });

    const onMount = useEffectEvent(() => {
        return setInterval(() => {
            onTick();
        }, delay);
    });

    useEffect(() => {
        const id = onMount();
        return () => {
            clearInterval(id);
        };
    }, []);

    return (
        <>
            <h1>
                Counter: {count}
                <button onClick={() => setCount(0)}>
                    Reset
                </button>
            </h1>
            <hr />
            <p>
                Increment by:
                <button
                    disabled={increment === 0}
                    onClick={() => {
                        setIncrement((i) => i - 1);
                    }}
                >
                    –
                </button>
                <b>{increment}</b>
                <button
                    onClick={() => {
                        setIncrement((i) => i + 1);
                    }}
                >
                    +
                </button>
            </p>
            <p>
                Increment delay:
                <button
                    disabled={delay === 100}
                    onClick={() => {
                        setDelay((d) => d - 100);
                    }}
                >
                    –100 ms
                </button>
                <b>{delay} ms</b>
                <button
                    onClick={() => {
                        setDelay((d) => d + 100);
                    }}
                >
                    +100 ms
                </button>
            </p>
        </>
    );
}
```

<!-- 0106.part.md -->

<!-- 0107.part.md -->

```css
button {
    margin: 10px;
}
```

<!-- 0108.part.md -->

\<Решение\>

Проблема приведенного выше примера заключается в том, что он извлек событие Effect Event под названием `onMount`, не задумываясь о том, что на самом деле должен делать код. Вы должны извлекать события Effect Events только по определенной причине: когда вы хотите сделать часть вашего кода нереактивной. Однако вызов `setInterval` _должен_ быть реактивным по отношению к переменной состояния `delay`. Если `delay` меняется, вы хотите установить интервал с нуля\! Чтобы исправить этот код, перенесите весь реактивный код обратно в Effect:

<!-- 0109.part.md -->

```js
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

<!-- 0110.part.md -->

<!-- 0111.part.md -->

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export default function Timer() {
    const [count, setCount] = useState(0);
    const [increment, setIncrement] = useState(1);
    const [delay, setDelay] = useState(100);

    const onTick = useEffectEvent(() => {
        setCount((c) => c + increment);
    });

    useEffect(() => {
        const id = setInterval(() => {
            onTick();
        }, delay);
        return () => {
            clearInterval(id);
        };
    }, [delay]);

    return (
        <>
            <h1>
                Counter: {count}
                <button onClick={() => setCount(0)}>
                    Reset
                </button>
            </h1>
            <hr />
            <p>
                Increment by:
                <button
                    disabled={increment === 0}
                    onClick={() => {
                        setIncrement((i) => i - 1);
                    }}
                >
                    –
                </button>
                <b>{increment}</b>
                <button
                    onClick={() => {
                        setIncrement((i) => i + 1);
                    }}
                >
                    +
                </button>
            </p>
            <p>
                Increment delay:
                <button
                    disabled={delay === 100}
                    onClick={() => {
                        setDelay((d) => d - 100);
                    }}
                >
                    –100 ms
                </button>
                <b>{delay} ms</b>
                <button
                    onClick={() => {
                        setDelay((d) => d + 100);
                    }}
                >
                    +100 ms
                </button>
            </p>
        </>
    );
}
```

<!-- 0112.part.md -->

<!-- 0113.part.md -->

```css
button {
    margin: 10px;
}
```

<!-- 0114.part.md -->

В целом, вы должны с подозрением относиться к функциям типа `onMount`, которые сосредоточены на _времени_, а не на _цели_ части кода. Поначалу это может показаться "более описательным", но это затуманивает ваш замысел. Как правило, события эффектов должны соответствовать чему-то, что происходит с точки зрения _пользователя_. Например, `onMessage`, `onTick`, `onVisit` или `onConnected` - хорошие названия событий эффектов. Код внутри них, скорее всего, не должен быть реактивным. С другой стороны, `onMount`, `onUpdate`, `onUnmount` или `onAfterRender` настолько общие, что в них легко случайно поместить код, который _должен_ быть реактивным. Вот почему вы должны называть свои события Effect Events в соответствии с тем, _что, по мнению пользователя, произошло,_ а не с тем, когда был запущен какой-то код.

\</Solution\>

#### Исправление отложенного уведомления {/_fix-a-delayed-notification_/}

Когда вы присоединяетесь к чату, этот компонент показывает уведомление. Однако он не показывает уведомление немедленно. Вместо этого уведомление искусственно задерживается на две секунды, чтобы у пользователя была возможность осмотреться в пользовательском интерфейсе.

Это почти работает, но есть ошибка. Попробуйте быстро изменить выпадающий список с "общие" на "путешествия", а затем на "музыка". Если вы сделаете это достаточно быстро, вы увидите два уведомления (как и ожидалось\!), но в обоих будет написано "Добро пожаловать в музыку".

Исправьте это так, чтобы при быстром переключении с "общего" на "путешествия" и затем на "музыку" вы видели два уведомления, первое из которых было бы "Добро пожаловать в путешествие", а второе - "Добро пожаловать в музыку". (Для дополнительной сложности, если вы _уже_ сделали так, чтобы уведомления показывали правильные комнаты, измените код так, чтобы отображалось только последнее уведомление).

\<Hint\>

Ваш Эффект знает, к какой комнате он подключился. Есть ли какая-нибудь информация, которую вы могли бы передать в событие вашего Эффекта?

\</Hint\>

<!-- 0115.part.md -->

```js
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

<!-- 0116.part.md -->

<!-- 0117.part.md -->

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
    const onConnected = useEffectEvent(() => {
        showNotification('Welcome to ' + roomId, theme);
    });

    useEffect(() => {
        const connection = createConnection(
            serverUrl,
            roomId
        );
        connection.on('connected', () => {
            setTimeout(() => {
                onConnected();
            }, 2000);
        });
        connection.connect();
        return () => connection.disconnect();
    }, [roomId]);

    return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
    const [roomId, setRoomId] = useState('general');
    const [isDark, setIsDark] = useState(false);
    return (
        <>
            <label>
                Choose the chat room:{' '}
                <select
                    value={roomId}
                    onChange={(e) =>
                        setRoomId(e.target.value)
                    }
                >
                    <option value="general">general</option>
                    <option value="travel">travel</option>
                    <option value="music">music</option>
                </select>
            </label>
            <label>
                <input
                    type="checkbox"
                    checked={isDark}
                    onChange={(e) =>
                        setIsDark(e.target.checked)
                    }
                />
                Use dark theme
            </label>
            <hr />
            <ChatRoom
                roomId={roomId}
                theme={isDark ? 'dark' : 'light'}
            />
        </>
    );
}
```

<!-- 0118.part.md -->

<!-- 0119.part.md -->

```js
export function createConnection(serverUrl, roomId) {
    // A real implementation would actually connect to the server
    let connectedCallback;
    let timeout;
    return {
        connect() {
            timeout = setTimeout(() => {
                if (connectedCallback) {
                    connectedCallback();
                }
            }, 100);
        },
        on(event, callback) {
            if (connectedCallback) {
                throw Error(
                    'Cannot add the handler twice.'
                );
            }
            if (event !== 'connected') {
                throw Error(
                    'Only "connected" event is supported.'
                );
            }
            connectedCallback = callback;
        },
        disconnect() {
            clearTimeout(timeout);
        },
    };
}
```

<!-- 0120.part.md -->

<!-- 0121.part.md -->

```js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
    Toastify({
        text: message,
        duration: 2000,
        gravity: 'top',
        position: 'right',
        style: {
            background:
                theme === 'dark' ? 'black' : 'white',
            color: theme === 'dark' ? 'white' : 'black',
        },
    }).showToast();
}
```

<!-- 0122.part.md -->

<!-- 0123.part.md -->

```css
label {
    display: block;
    margin-top: 10px;
}
```

<!-- 0124.part.md -->

\<Решение\>

Внутри вашего события Effect Event, `roomId` - это значение _в момент вызова Effect Event._.

Ваше Событие Эффекта вызывается с двухсекундной задержкой. Если вы быстро переключаетесь из комнаты для путешествий в музыкальную комнату, то к тому времени, когда появится уведомление комнаты для путешествий, `roomId` уже будет `"music"`. Вот почему оба уведомления говорят "Добро пожаловать в музыкальную комнату".

Чтобы решить эту проблему, вместо чтения _последнего_ `roomId` внутри события эффекта, сделайте его параметром события эффекта, как `connectedRoomId` ниже. Затем передавайте `roomId` из вашего Эффекта, вызывая `onConnected(roomId)`:

<!-- 0125.part.md -->

```js
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

<!-- 0126.part.md -->

<!-- 0127.part.md -->

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
    const onConnected = useEffectEvent(
        (connectedRoomId) => {
            showNotification(
                'Welcome to ' + connectedRoomId,
                theme
            );
        }
    );

    useEffect(() => {
        const connection = createConnection(
            serverUrl,
            roomId
        );
        connection.on('connected', () => {
            setTimeout(() => {
                onConnected(roomId);
            }, 2000);
        });
        connection.connect();
        return () => connection.disconnect();
    }, [roomId]);

    return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
    const [roomId, setRoomId] = useState('general');
    const [isDark, setIsDark] = useState(false);
    return (
        <>
            <label>
                Choose the chat room:{' '}
                <select
                    value={roomId}
                    onChange={(e) =>
                        setRoomId(e.target.value)
                    }
                >
                    <option value="general">general</option>
                    <option value="travel">travel</option>
                    <option value="music">music</option>
                </select>
            </label>
            <label>
                <input
                    type="checkbox"
                    checked={isDark}
                    onChange={(e) =>
                        setIsDark(e.target.checked)
                    }
                />
                Use dark theme
            </label>
            <hr />
            <ChatRoom
                roomId={roomId}
                theme={isDark ? 'dark' : 'light'}
            />
        </>
    );
}
```

<!-- 0128.part.md -->

<!-- 0129.part.md -->

```js
export function createConnection(serverUrl, roomId) {
    // A real implementation would actually connect to the server
    let connectedCallback;
    let timeout;
    return {
        connect() {
            timeout = setTimeout(() => {
                if (connectedCallback) {
                    connectedCallback();
                }
            }, 100);
        },
        on(event, callback) {
            if (connectedCallback) {
                throw Error(
                    'Cannot add the handler twice.'
                );
            }
            if (event !== 'connected') {
                throw Error(
                    'Only "connected" event is supported.'
                );
            }
            connectedCallback = callback;
        },
        disconnect() {
            clearTimeout(timeout);
        },
    };
}
```

<!-- 0130.part.md -->

<!-- 0131.part.md -->

```js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
    Toastify({
        text: message,
        duration: 2000,
        gravity: 'top',
        position: 'right',
        style: {
            background:
                theme === 'dark' ? 'black' : 'white',
            color: theme === 'dark' ? 'white' : 'black',
        },
    }).showToast();
}
```

<!-- 0132.part.md -->

<!-- 0133.part.md -->

```css
label {
    display: block;
    margin-top: 10px;
}
```

<!-- 0134.part.md -->

Эффект, у которого `roomId` был установлен на `"travel"` (поэтому он подключился к комнате `"travel"`), покажет уведомление для `"travel"`. Эффект, у которого `roomId` установлен в `"music"` (поэтому он подключился к комнате `"music"`), покажет уведомление для `"music"`. Другими словами, `connectedRoomId` приходит от вашего Эффекта (который является реактивным), в то время как `theme` всегда использует последнее значение.

Чтобы решить дополнительную проблему, сохраните идентификатор таймаута уведомления и очистите его в функции очистки вашего Эффекта:

<!-- 0135.part.md -->

```js
{
  "dependencies": {
    "react": "experimental",
    "react-dom": "experimental",
    "react-scripts": "latest",
    "toastify-js": "1.12.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

<!-- 0136.part.md -->

<!-- 0137.part.md -->

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection, sendMessage } from './chat.js';
import { showNotification } from './notifications.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId, theme }) {
    const onConnected = useEffectEvent(
        (connectedRoomId) => {
            showNotification(
                'Welcome to ' + connectedRoomId,
                theme
            );
        }
    );

    useEffect(() => {
        const connection = createConnection(
            serverUrl,
            roomId
        );
        let notificationTimeoutId;
        connection.on('connected', () => {
            notificationTimeoutId = setTimeout(() => {
                onConnected(roomId);
            }, 2000);
        });
        connection.connect();
        return () => {
            connection.disconnect();
            if (notificationTimeoutId !== undefined) {
                clearTimeout(notificationTimeoutId);
            }
        };
    }, [roomId]);

    return <h1>Welcome to the {roomId} room!</h1>;
}

export default function App() {
    const [roomId, setRoomId] = useState('general');
    const [isDark, setIsDark] = useState(false);
    return (
        <>
            <label>
                Choose the chat room:{' '}
                <select
                    value={roomId}
                    onChange={(e) =>
                        setRoomId(e.target.value)
                    }
                >
                    <option value="general">general</option>
                    <option value="travel">travel</option>
                    <option value="music">music</option>
                </select>
            </label>
            <label>
                <input
                    type="checkbox"
                    checked={isDark}
                    onChange={(e) =>
                        setIsDark(e.target.checked)
                    }
                />
                Use dark theme
            </label>
            <hr />
            <ChatRoom
                roomId={roomId}
                theme={isDark ? 'dark' : 'light'}
            />
        </>
    );
}
```

<!-- 0138.part.md -->

<!-- 0139.part.md -->

```js
export function createConnection(serverUrl, roomId) {
    // A real implementation would actually connect to the server
    let connectedCallback;
    let timeout;
    return {
        connect() {
            timeout = setTimeout(() => {
                if (connectedCallback) {
                    connectedCallback();
                }
            }, 100);
        },
        on(event, callback) {
            if (connectedCallback) {
                throw Error(
                    'Cannot add the handler twice.'
                );
            }
            if (event !== 'connected') {
                throw Error(
                    'Only "connected" event is supported.'
                );
            }
            connectedCallback = callback;
        },
        disconnect() {
            clearTimeout(timeout);
        },
    };
}
```

<!-- 0140.part.md -->

<!-- 0141.part.md -->

```js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme) {
    Toastify({
        text: message,
        duration: 2000,
        gravity: 'top',
        position: 'right',
        style: {
            background:
                theme === 'dark' ? 'black' : 'white',
            color: theme === 'dark' ? 'white' : 'black',
        },
    }).showToast();
}
```

<!-- 0142.part.md -->

<!-- 0143.part.md -->

```css
label {
    display: block;
    margin-top: 10px;
}
```

<!-- 0144.part.md -->

Это гарантирует, что уже запланированные (но еще не отображенные) уведомления будут отменены, когда вы смените комнату.

\</Solution\>

\</Challenges\>

<!-- 0145.part.md -->

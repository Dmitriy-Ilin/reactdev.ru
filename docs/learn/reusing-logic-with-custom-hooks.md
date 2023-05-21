# Переиспользование логики с помощью пользовательских хуков

React поставляется с несколькими встроенными хуками, такими как `useState`, `useContext` и `useEffect`. Иногда вам захочется иметь хук для какой-то более конкретной цели: например, для получения данных, отслеживания того, находится ли пользователь в сети, или для подключения к чату. Возможно, вы не найдете таких хуков в React, но вы можете создать свои собственные хуки для нужд вашего приложения.

-   Что такое пользовательские хуки и как написать свой собственный
-   Как повторно использовать логику между компонентами
-   Как назвать и структурировать пользовательские хуки
-   Когда и зачем извлекать пользовательские хуки

## Пользовательские хуки: Совместное использование логики между компонентами {/_custom-hooks-sharing-logic-between-components_/}

Представьте, что вы разрабатываете приложение, которое сильно зависит от сети (как и большинство приложений). Вы хотите предупредить пользователя, если его сетевое соединение случайно прервалось во время работы с вашим приложением. Как вы собираетесь это сделать? Похоже, что вам понадобятся две вещи в вашем компоненте:

1.  Элемент состояния, который отслеживает, находится ли сеть в сети.
2.  Эффект, который подписывается на глобальные события [`online`](https://developer.mozilla.org/en-US/docs/Web/API/Window/online_event) и [`offline`](https://developer.mozilla.org/en-US/docs/Web/API/Window/offline_event) и обновляет это состояние.

Это позволит вашему компоненту [синхронизироваться](/learn/synchronizing-with-effects) со статусом сети. Вы можете начать с чего-то подобного:

<!-- 0001.part.md -->

```js
import { useState, useEffect } from 'react';

export default function StatusBar() {
    const [isOnline, setIsOnline] = useState(true);
    useEffect(() => {
        function handleOnline() {
            setIsOnline(true);
        }
        function handleOffline() {
            setIsOnline(false);
        }
        window.addEventListener('online', handleOnline);
        window.addEventListener('offline', handleOffline);
        return () => {
            window.removeEventListener(
                'online',
                handleOnline
            );
            window.removeEventListener(
                'offline',
                handleOffline
            );
        };
    }, []);

    return (
        <h1>
            {isOnline ? '✅ Online' : '❌ Disconnected'}
        </h1>
    );
}
```

<!-- 0002.part.md -->

Попробуйте включить и выключить сеть, и обратите внимание, как эта `StatusBar` обновляется в ответ на ваши действия.

Теперь представьте, что вы _также_ хотите использовать ту же логику в другом компоненте. Вы хотите реализовать кнопку Save, которая будет отключена и показывать "Reconnecting..." вместо "Save", пока сеть выключена.

Для начала вы можете скопировать и вставить состояние `isOnline` и эффект в `SaveButton`:

<!-- 0003.part.md -->

```js
import { useState, useEffect } from 'react';

export default function SaveButton() {
    const [isOnline, setIsOnline] = useState(true);
    useEffect(() => {
        function handleOnline() {
            setIsOnline(true);
        }
        function handleOffline() {
            setIsOnline(false);
        }
        window.addEventListener('online', handleOnline);
        window.addEventListener('offline', handleOffline);
        return () => {
            window.removeEventListener(
                'online',
                handleOnline
            );
            window.removeEventListener(
                'offline',
                handleOffline
            );
        };
    }, []);

    function handleSaveClick() {
        console.log('✅ Progress saved');
    }

    return (
        <button
            disabled={!isOnline}
            onClick={handleSaveClick}
        >
            {isOnline ? 'Save progress' : 'Reconnecting...'}
        </button>
    );
}
```

<!-- 0004.part.md -->

Убедитесь, что при отключении сети кнопка изменит свой вид.

Эти два компонента работают нормально, но дублирование логики между ними вызывает сожаление. Похоже, что даже если они имеют разный _визуальный вид_, вы хотите повторно использовать логику между ними.

### Извлечение собственного пользовательского хука из компонента {/_extracting-your-own-custom-hook-from-a-component_/}

Представьте на секунду, что, подобно [`useState`](/reference/react/useState) и [`useEffect`](/reference/react/useEffect), существует встроенный хук `useOnlineStatus`. Тогда оба этих компонента можно было бы упростить и убрать дублирование между ними:

<!-- 0005.part.md -->

```js
function StatusBar() {
    const isOnline = useOnlineStatus();
    return (
        <h1>
            {isOnline ? '✅ Online' : '❌ Disconnected'}
        </h1>
    );
}

function SaveButton() {
    const isOnline = useOnlineStatus();

    function handleSaveClick() {
        console.log('✅ Progress saved');
    }

    return (
        <button
            disabled={!isOnline}
            onClick={handleSaveClick}
        >
            {isOnline ? 'Save progress' : 'Reconnecting...'}
        </button>
    );
}
```

<!-- 0006.part.md -->

Хотя такого встроенного Hook не существует, вы можете написать его самостоятельно. Объявите функцию `useOnlineStatus` и перенесите в нее весь дублирующийся код из компонентов, которые вы написали ранее:

<!-- 0007.part.md -->

```js
function useOnlineStatus() {
    const [isOnline, setIsOnline] = useState(true);
    useEffect(() => {
        function handleOnline() {
            setIsOnline(true);
        }
        function handleOffline() {
            setIsOnline(false);
        }
        window.addEventListener('online', handleOnline);
        window.addEventListener('offline', handleOffline);
        return () => {
            window.removeEventListener(
                'online',
                handleOnline
            );
            window.removeEventListener(
                'offline',
                handleOffline
            );
        };
    }, []);
    return isOnline;
}
```

<!-- 0008.part.md -->

В конце функции верните `isOnline`. Это позволит вашим компонентам прочитать это значение:

<!-- 0009.part.md -->

```js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
    const isOnline = useOnlineStatus();
    return (
        <h1>
            {isOnline ? '✅ Online' : '❌ Disconnected'}
        </h1>
    );
}

function SaveButton() {
    const isOnline = useOnlineStatus();

    function handleSaveClick() {
        console.log('✅ Progress saved');
    }

    return (
        <button
            disabled={!isOnline}
            onClick={handleSaveClick}
        >
            {isOnline ? 'Save progress' : 'Reconnecting...'}
        </button>
    );
}

export default function App() {
    return (
        <>
            <SaveButton />
            <StatusBar />
        </>
    );
}
```

<!-- 0010.part.md -->

<!-- 0011.part.md -->

```js
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
    const [isOnline, setIsOnline] = useState(true);
    useEffect(() => {
        function handleOnline() {
            setIsOnline(true);
        }
        function handleOffline() {
            setIsOnline(false);
        }
        window.addEventListener('online', handleOnline);
        window.addEventListener('offline', handleOffline);
        return () => {
            window.removeEventListener(
                'online',
                handleOnline
            );
            window.removeEventListener(
                'offline',
                handleOffline
            );
        };
    }, []);
    return isOnline;
}
```

<!-- 0012.part.md -->

Убедитесь, что включение и выключение сети обновляет оба компонента.

Теперь в ваших компонентах не так много повторяющейся логики. **Более того, код внутри них описывает _что они хотят сделать_ (использовать сетевой статус\!), а не _как это сделать_ (подписываясь на события браузера)**.

Когда вы извлекаете логику в пользовательские Hooks, вы можете скрыть ужасные детали того, как вы работаете с какой-то внешней системой или API браузера. Код ваших компонентов выражает ваше намерение, а не реализацию.

### Имена хуков всегда начинаются с `use` {/_hook-names-always-start-with-use_/}

Приложения React строятся из компонентов. Компоненты строятся из хуков, встроенных или пользовательских. Скорее всего, вы часто будете использовать пользовательские хуки, созданные другими, но иногда вы можете написать один самостоятельно\!

Вы должны следовать этим соглашениям об именовании:

1.  **Имена компонентов React должны начинаться с заглавной буквы,** например, `StatusBar` и `SaveButton`. Компоненты React также должны возвращать что-то, что React умеет отображать, например, кусок JSX.
2.  **Имена хуков должны начинаться с `use`, за которым следует заглавная буква,** например [`useState`](/reference/react/useState) (встроенный) или `useOnlineStatus` (пользовательский, как ранее на этой странице). Крючки могут возвращать произвольные значения.

Это соглашение гарантирует, что вы всегда сможете посмотреть на компонент и узнать, где может "прятаться" его состояние, Эффекты и другие возможности React. Например, если вы видите вызов функции `getColor()` внутри вашего компонента, вы можете быть уверены, что она не может содержать внутри себя состояние React, потому что ее имя не начинается с `use`. Однако вызов такой функции, как `useOnlineStatus()`, скорее всего, будет содержать вызовы других Hooks внутри\!

Если ваш линтер [настроен на React,](/learn/editor-setup#linting), он будет применять это соглашение об именовании. Прокрутите вверх до песочницы выше и переименуйте `useOnlineStatus` в `getOnlineStatus`. Обратите внимание, что линтер больше не позволит вам вызывать `useState` или `useEffect` внутри него. Только Hooks и компоненты могут вызывать другие Hooks\!

#### Должны ли все функции, вызываемые во время рендеринга, начинаться с префикса use? {/_should-all-functions called-during-rendering-start-with-the-use-prefix_/}

Нет. Функции, которые не _вызывают_ Hooks, не обязаны _быть_ Hooks.

Если ваша функция не вызывает никаких хуков, избегайте префикса `use`. Вместо этого напишите ее как обычную функцию _без_ префикса `use`. Например, `useSorted` ниже не вызывает хуков, поэтому вместо этого назовите ее `getSorted`:

<!-- 0013.part.md -->

```js
// 🔴 Avoid: A Hook that doesn't use Hooks
function useSorted(items) {
    return items.slice().sort();
}

// ✅ Good: A regular function that doesn't use Hooks
function getSorted(items) {
    return items.slice().sort();
}
```

<!-- 0014.part.md -->

Это гарантирует, что ваш код сможет вызвать эту регулярную функцию в любом месте, включая условия:

<!-- 0015.part.md -->

```js
function List({ items, shouldSort }) {
    let displayedItems = items;
    if (shouldSort) {
        // ✅ It's ok to call getSorted() conditionally because it's not a Hook
        displayedItems = getSorted(items);
    }
    // ...
}
```

<!-- 0016.part.md -->

Вы должны дать префикс `use` функции (и таким образом сделать ее хуком), если она использует хотя бы один хук внутри себя:

<!-- 0017.part.md -->

```js
// ✅ Good: A Hook that uses other Hooks
function useAuth() {
    return useContext(Auth);
}
```

<!-- 0018.part.md -->

Технически, React этого не делает. В принципе, вы можете сделать хук, который не вызывает другие хуки. Это часто запутывает и ограничивает, поэтому лучше избегать такого шаблона. Однако в редких случаях это может быть полезно. Например, возможно, ваша функция сейчас не использует никаких хуков, но в будущем вы планируете добавить в нее несколько вызовов хуков. Тогда имеет смысл назвать ее с префиксом `use`:

<!-- 0019.part.md -->

```js
// ✅ Good: A Hook that will likely use some other Hooks later
function useAuth() {
    // TODO: Replace with this line when authentication is implemented:
    // return useContext(Auth);
    return TEST_USER;
}
```

<!-- 0020.part.md -->

Тогда компоненты не смогут вызывать его условно. Это станет важным, когда вы действительно добавите вызовы Hook внутри. Если вы не планируете использовать в нем хуки (сейчас или позже), не делайте его хуком.

### Пользовательские хуки позволяют вам делиться логикой состояния, а не самим состоянием {/_custom-hooks-let-you-share-stateful-logic-not-state-itself_/}

В предыдущем примере, когда вы включали и выключали сеть, оба компонента обновлялись вместе. Однако неправильно думать, что одна переменная состояния `isOnline` разделяется между ними. Посмотрите на этот код:

<!-- 0021.part.md -->

```js
function StatusBar() {
    const isOnline = useOnlineStatus();
    // ...
}

function SaveButton() {
    const isOnline = useOnlineStatus();
    // ...
}
```

<!-- 0022.part.md -->

Он работает так же, как и до извлечения дубликата:

<!-- 0023.part.md -->

```js
function StatusBar() {
    const [isOnline, setIsOnline] = useState(true);
    useEffect(() => {
        // ...
    }, []);
    // ...
}

function SaveButton() {
    const [isOnline, setIsOnline] = useState(true);
    useEffect(() => {
        // ...
    }, []);
    // ...
}
```

<!-- 0024.part.md -->

Это две совершенно независимые переменные состояния и Effects\! Они имеют одинаковое значение в одно и то же время, потому что вы синхронизировали их с одним и тем же внешним значением (включена ли сеть).

Чтобы лучше проиллюстрировать это, нам понадобится другой пример. Рассмотрим компонент `Form`:

<!-- 0025.part.md -->

```js
import { useState } from 'react';

export default function Form() {
    const [firstName, setFirstName] = useState('Mary');
    const [lastName, setLastName] = useState('Poppins');

    function handleFirstNameChange(e) {
        setFirstName(e.target.value);
    }

    function handleLastNameChange(e) {
        setLastName(e.target.value);
    }

    return (
        <>
            <label>
                First name:
                <input
                    value={firstName}
                    onChange={handleFirstNameChange}
                />
            </label>
            <label>
                Last name:
                <input
                    value={lastName}
                    onChange={handleLastNameChange}
                />
            </label>
            <p>
                <b>
                    Good morning, {firstName} {lastName}.
                </b>
            </p>
        </>
    );
}
```

<!-- 0026.part.md -->

<!-- 0027.part.md -->

```css
label {
    display: block;
}
input {
    margin-left: 10px;
}
```

<!-- 0028.part.md -->

Есть несколько повторяющихся логических операций для каждого поля формы:

1.  Есть часть состояния (`firstName` и `lastName`).
2.  Есть обработчик изменений (`handleFirstNameChange` и `handleLastNameChange`).
3.  Есть кусок JSX, который определяет атрибуты `value` и `onChange` для этого входа.

Вы можете извлечь повторяющуюся логику в этот пользовательский хук `useFormInput`:

<!-- 0029.part.md -->

```js
import { useFormInput } from './useFormInput.js';

export default function Form() {
    const firstNameProps = useFormInput('Mary');
    const lastNameProps = useFormInput('Poppins');

    return (
        <>
            <label>
                First name:
                <input {...firstNameProps} />
            </label>
            <label>
                Last name:
                <input {...lastNameProps} />
            </label>
            <p>
                <b>
                    Good morning, {firstNameProps.value}{' '}
                    {lastNameProps.value}.
                </b>
            </p>
        </>
    );
}
```

<!-- 0030.part.md -->

<!-- 0031.part.md -->

```js
import { useState } from 'react';

export function useFormInput(initialValue) {
    const [value, setValue] = useState(initialValue);

    function handleChange(e) {
        setValue(e.target.value);
    }

    const inputProps = {
        value: value,
        onChange: handleChange,
    };

    return inputProps;
}
```

<!-- 0032.part.md -->

<!-- 0033.part.md -->

```css
label {
    display: block;
}
input {
    margin-left: 10px;
}
```

<!-- 0034.part.md -->

Обратите внимание, что здесь объявлена только _одна_ переменная состояния под названием `value`.

Однако, компонент `Form` вызывает `useFormInput` _два раза:_

<!-- 0035.part.md -->

```js
function Form() {
  const firstNameProps = useFormInput('Mary');
  const lastNameProps = useFormInput('Poppins');
  // ...
```

<!-- 0036.part.md -->

Вот почему это работает как объявление двух отдельных переменных состояния\!

**Настроенные хуки позволяют вам делиться _логикой состояния_, но не самим состоянием.\* Каждый вызов хука полностью независим от любого другого вызова того же хука.** Вот почему две вышеприведенные песочницы полностью эквивалентны. Если хотите, прокрутите страницу назад и сравните их. Поведение до и после извлечения пользовательского хука идентично.

Если вам нужно разделить само состояние между несколькими компонентами, вместо этого [lift it up and pass it down](/learn/sharing-state-between-components).

## Передача реактивных значений между хуками {/_passing-reactive-values-between-hooks_/}

Код внутри ваших пользовательских хуков будет выполняться заново при каждом повторном рендеринге компонента. Вот почему, как и компоненты, пользовательские хуки [должны быть чистыми.](/learn/keeping-components-pure) Думайте о коде пользовательских хуков как о части тела\ вашего компонента!

Поскольку пользовательские хуки перерисовываются вместе с вашим компонентом, они всегда получают последние реквизиты и состояние. Чтобы понять, что это значит, рассмотрим пример с чатом. Измените URL сервера или чата:

<!-- 0037.part.md -->

```js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
    const [roomId, setRoomId] = useState('general');
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
            <hr />
            <ChatRoom roomId={roomId} />
        </>
    );
}
```

<!-- 0038.part.md -->

<!-- 0039.part.md -->

```js
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';
import { showNotification } from './notifications.js';

export default function ChatRoom({ roomId }) {
    const [serverUrl, setServerUrl] = useState(
        'https://localhost:1234'
    );

    useEffect(() => {
        const options = {
            serverUrl: serverUrl,
            roomId: roomId,
        };
        const connection = createConnection(options);
        connection.on('message', (msg) => {
            showNotification('New message: ' + msg);
        });
        connection.connect();
        return () => connection.disconnect();
    }, [roomId, serverUrl]);

    return (
        <>
            <label>
                Server URL:
                <input
                    value={serverUrl}
                    onChange={(e) =>
                        setServerUrl(e.target.value)
                    }
                />
            </label>
            <h1>Welcome to the {roomId} room!</h1>
        </>
    );
}
```

<!-- 0040.part.md -->

<!-- 0041.part.md -->

```js
export function createConnection({ serverUrl, roomId }) {
    // A real implementation would actually connect to the server
    if (typeof serverUrl !== 'string') {
        throw Error(
            'Expected serverUrl to be a string. Received: ' +
                serverUrl
        );
    }
    if (typeof roomId !== 'string') {
        throw Error(
            'Expected roomId to be a string. Received: ' +
                roomId
        );
    }
    let intervalId;
    let messageCallback;
    return {
        connect() {
            console.log(
                '✅ Connecting to "' +
                    roomId +
                    '" room at ' +
                    serverUrl +
                    '...'
            );
            clearInterval(intervalId);
            intervalId = setInterval(() => {
                if (messageCallback) {
                    if (Math.random() > 0.5) {
                        messageCallback('hey');
                    } else {
                        messageCallback('lol');
                    }
                }
            }, 3000);
        },
        disconnect() {
            clearInterval(intervalId);
            messageCallback = null;
            console.log(
                '❌ Disconnected from "' +
                    roomId +
                    '" room at ' +
                    serverUrl +
                    ''
            );
        },
        on(event, callback) {
            if (messageCallback) {
                throw Error(
                    'Cannot add the handler twice.'
                );
            }
            if (event !== 'message') {
                throw Error(
                    'Only "message" event is supported.'
                );
            }
            messageCallback = callback;
        },
    };
}
```

<!-- 0042.part.md -->

<!-- 0043.part.md -->

```js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme = 'dark') {
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

<!-- 0044.part.md -->

<!-- 0045.part.md -->

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

<!-- 0046.part.md -->

<!-- 0047.part.md -->

```css
input {
    display: block;
    margin-bottom: 20px;
}
button {
    margin-left: 10px;
}
```

<!-- 0048.part.md -->

Когда вы изменяете `serverUrl` или `roomId`, Эффект ["реагирует" на ваши изменения] (/learn/lifecycle-of-reactive-effects#effects-react-to-reactive-values) и пересинхронизируется. По сообщениям консоли вы можете определить, что чат переподключается каждый раз, когда вы изменяете зависимости вашего Эффекта.

Теперь переместите код эффекта в пользовательский хук:

<!-- 0049.part.md -->

```js
export function useChatRoom({ serverUrl, roomId }) {
    useEffect(() => {
        const options = {
            serverUrl: serverUrl,
            roomId: roomId,
        };
        const connection = createConnection(options);
        connection.connect();
        connection.on('message', (msg) => {
            showNotification('New message: ' + msg);
        });
        return () => connection.disconnect();
    }, [roomId, serverUrl]);
}
```

<!-- 0050.part.md -->

Это позволит вашему компоненту `ChatRoom` вызывать ваш пользовательский Hook, не беспокоясь о том, как он работает внутри:

<!-- 0051.part.md -->

```js
export default function ChatRoom({ roomId }) {
    const [serverUrl, setServerUrl] = useState(
        'https://localhost:1234'
    );

    useChatRoom({
        roomId: roomId,
        serverUrl: serverUrl,
    });

    return (
        <>
            <label>
                Server URL:
                <input
                    value={serverUrl}
                    onChange={(e) =>
                        setServerUrl(e.target.value)
                    }
                />
            </label>
            <h1>Welcome to the {roomId} room!</h1>
        </>
    );
}
```

<!-- 0052.part.md -->

Это выглядит намного проще\! (Но делает то же самое.)

Обратите внимание, что логика _все еще реагирует_ на изменения реквизита и состояния. Попробуйте отредактировать URL сервера или выбранной комнаты:

<!-- 0053.part.md -->

```js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
    const [roomId, setRoomId] = useState('general');
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
            <hr />
            <ChatRoom roomId={roomId} />
        </>
    );
}
```

<!-- 0054.part.md -->

<!-- 0055.part.md -->

```js
import { useState } from 'react';
import { useChatRoom } from './useChatRoom.js';

export default function ChatRoom({ roomId }) {
    const [serverUrl, setServerUrl] = useState(
        'https://localhost:1234'
    );

    useChatRoom({
        roomId: roomId,
        serverUrl: serverUrl,
    });

    return (
        <>
            <label>
                Server URL:
                <input
                    value={serverUrl}
                    onChange={(e) =>
                        setServerUrl(e.target.value)
                    }
                />
            </label>
            <h1>Welcome to the {roomId} room!</h1>
        </>
    );
}
```

<!-- 0056.part.md -->

<!-- 0057.part.md -->

```js
import { useEffect } from 'react';
import { createConnection } from './chat.js';
import { showNotification } from './notifications.js';

export function useChatRoom({ serverUrl, roomId }) {
    useEffect(() => {
        const options = {
            serverUrl: serverUrl,
            roomId: roomId,
        };
        const connection = createConnection(options);
        connection.connect();
        connection.on('message', (msg) => {
            showNotification('New message: ' + msg);
        });
        return () => connection.disconnect();
    }, [roomId, serverUrl]);
}
```

<!-- 0058.part.md -->

<!-- 0059.part.md -->

```js
export function createConnection({ serverUrl, roomId }) {
    // A real implementation would actually connect to the server
    if (typeof serverUrl !== 'string') {
        throw Error(
            'Expected serverUrl to be a string. Received: ' +
                serverUrl
        );
    }
    if (typeof roomId !== 'string') {
        throw Error(
            'Expected roomId to be a string. Received: ' +
                roomId
        );
    }
    let intervalId;
    let messageCallback;
    return {
        connect() {
            console.log(
                '✅ Connecting to "' +
                    roomId +
                    '" room at ' +
                    serverUrl +
                    '...'
            );
            clearInterval(intervalId);
            intervalId = setInterval(() => {
                if (messageCallback) {
                    if (Math.random() > 0.5) {
                        messageCallback('hey');
                    } else {
                        messageCallback('lol');
                    }
                }
            }, 3000);
        },
        disconnect() {
            clearInterval(intervalId);
            messageCallback = null;
            console.log(
                '❌ Disconnected from "' +
                    roomId +
                    '" room at ' +
                    serverUrl +
                    ''
            );
        },
        on(event, callback) {
            if (messageCallback) {
                throw Error(
                    'Cannot add the handler twice.'
                );
            }
            if (event !== 'message') {
                throw Error(
                    'Only "message" event is supported.'
                );
            }
            messageCallback = callback;
        },
    };
}
```

<!-- 0060.part.md -->

<!-- 0061.part.md -->

```js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme = 'dark') {
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

<!-- 0062.part.md -->

<!-- 0063.part.md -->

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

<!-- 0064.part.md -->

<!-- 0065.part.md -->

```css
input {
    display: block;
    margin-bottom: 20px;
}
button {
    margin-left: 10px;
}
```

<!-- 0066.part.md -->

Обратите внимание, что вы берете возвращаемое значение одного Hook:

<!-- 0067.part.md -->

```js
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

<!-- 0068.part.md -->

и передать его в качестве входного сигнала в другой Hook:

<!-- 0069.part.md -->

```js
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

<!-- 0070.part.md -->

Каждый раз, когда ваш компонент `ChatRoom` перерисовывается, он передает последние значения `roomId` и `serverUrl` вашему Hook. Именно поэтому ваш Эффект повторно подключается к чату, когда их значения меняются после повторного рендеринга. (Если вы когда-нибудь работали с программами для обработки аудио или видео, то такое построение цепочки хуков может напомнить вам построение цепочки визуальных или звуковых эффектов. Это как если бы выход `useState` "вливался" во вход `useChatRoom`).

### Передача обработчиков событий пользовательским крючкам {/_passing-event-handlers-to-custom-hooks_/}

\<Wip\>

Этот раздел описывает **экспериментальный API, который еще не был выпущен** в стабильной версии React.

\</Wip\>

Когда вы начнете использовать `useChatRoom` в большем количестве компонентов, вы, возможно, захотите позволить компонентам настраивать его поведение. Например, в настоящее время логика того, что делать, когда приходит сообщение, жестко закодирована внутри Hook:

<!-- 0071.part.md -->

```js
export function useChatRoom({ serverUrl, roomId }) {
    useEffect(() => {
        const options = {
            serverUrl: serverUrl,
            roomId: roomId,
        };
        const connection = createConnection(options);
        connection.connect();
        connection.on('message', (msg) => {
            showNotification('New message: ' + msg);
        });
        return () => connection.disconnect();
    }, [roomId, serverUrl]);
}
```

<!-- 0072.part.md -->

Допустим, вы хотите перенести эту логику обратно в ваш компонент:

<!-- 0073.part.md -->

```js
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    onReceiveMessage(msg) {
      showNotification('New message: ' + msg);
    }
  });
  // ...
```

<!-- 0074.part.md -->

Чтобы это работало, измените свой пользовательский хук так, чтобы он принимал `onReceiveMessage` в качестве одной из именованных опций:

<!-- 0075.part.md -->

```js
export function useChatRoom({
    serverUrl,
    roomId,
    onReceiveMessage,
}) {
    useEffect(() => {
        const options = {
            serverUrl: serverUrl,
            roomId: roomId,
        };
        const connection = createConnection(options);
        connection.connect();
        connection.on('message', (msg) => {
            onReceiveMessage(msg);
        });
        return () => connection.disconnect();
    }, [roomId, serverUrl, onReceiveMessage]); // ✅ All dependencies declared
}
```

<!-- 0076.part.md -->

Это будет работать, но есть еще одно улучшение, которое вы можете сделать, когда ваш пользовательский Hook принимает обработчики событий.

Добавление зависимости от `onReceiveMessage` не является идеальным, потому что это заставит чат переподключаться каждый раз, когда компонент перерендерится. [Заверните этот обработчик события в событие эффекта, чтобы избавить его от зависимостей:](/learn/removing-effect-dependencies#wrapping-an-event-handler-from-the-props)

<!-- 0077.part.md -->

```js
import { useEffect, useEffectEvent } from 'react';
// ...

export function useChatRoom({
    serverUrl,
    roomId,
    onReceiveMessage,
}) {
    const onMessage = useEffectEvent(onReceiveMessage);

    useEffect(() => {
        const options = {
            serverUrl: serverUrl,
            roomId: roomId,
        };
        const connection = createConnection(options);
        connection.connect();
        connection.on('message', (msg) => {
            onMessage(msg);
        });
        return () => connection.disconnect();
    }, [roomId, serverUrl]); // ✅ All dependencies declared
}
```

<!-- 0078.part.md -->

Теперь чат не будет подключаться заново каждый раз, когда компонент `ChatRoom` перерисовывается. Вот полностью рабочий демонстрационный пример передачи обработчика события пользовательскому Hook, с которым вы можете поиграть:

<!-- 0079.part.md -->

```js
import { useState } from 'react';
import ChatRoom from './ChatRoom.js';

export default function App() {
    const [roomId, setRoomId] = useState('general');
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
            <hr />
            <ChatRoom roomId={roomId} />
        </>
    );
}
```

<!-- 0080.part.md -->

<!-- 0081.part.md -->

```js
import { useState } from 'react';
import { useChatRoom } from './useChatRoom.js';
import { showNotification } from './notifications.js';

export default function ChatRoom({ roomId }) {
    const [serverUrl, setServerUrl] = useState(
        'https://localhost:1234'
    );

    useChatRoom({
        roomId: roomId,
        serverUrl: serverUrl,
        onReceiveMessage(msg) {
            showNotification('New message: ' + msg);
        },
    });

    return (
        <>
            <label>
                Server URL:
                <input
                    value={serverUrl}
                    onChange={(e) =>
                        setServerUrl(e.target.value)
                    }
                />
            </label>
            <h1>Welcome to the {roomId} room!</h1>
        </>
    );
}
```

<!-- 0082.part.md -->

<!-- 0083.part.md -->

```js
import { useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';
import { createConnection } from './chat.js';

export function useChatRoom({
    serverUrl,
    roomId,
    onReceiveMessage,
}) {
    const onMessage = useEffectEvent(onReceiveMessage);

    useEffect(() => {
        const options = {
            serverUrl: serverUrl,
            roomId: roomId,
        };
        const connection = createConnection(options);
        connection.connect();
        connection.on('message', (msg) => {
            onMessage(msg);
        });
        return () => connection.disconnect();
    }, [roomId, serverUrl]);
}
```

<!-- 0084.part.md -->

<!-- 0085.part.md -->

```js
export function createConnection({ serverUrl, roomId }) {
    // A real implementation would actually connect to the server
    if (typeof serverUrl !== 'string') {
        throw Error(
            'Expected serverUrl to be a string. Received: ' +
                serverUrl
        );
    }
    if (typeof roomId !== 'string') {
        throw Error(
            'Expected roomId to be a string. Received: ' +
                roomId
        );
    }
    let intervalId;
    let messageCallback;
    return {
        connect() {
            console.log(
                '✅ Connecting to "' +
                    roomId +
                    '" room at ' +
                    serverUrl +
                    '...'
            );
            clearInterval(intervalId);
            intervalId = setInterval(() => {
                if (messageCallback) {
                    if (Math.random() > 0.5) {
                        messageCallback('hey');
                    } else {
                        messageCallback('lol');
                    }
                }
            }, 3000);
        },
        disconnect() {
            clearInterval(intervalId);
            messageCallback = null;
            console.log(
                '❌ Disconnected from "' +
                    roomId +
                    '" room at ' +
                    serverUrl +
                    ''
            );
        },
        on(event, callback) {
            if (messageCallback) {
                throw Error(
                    'Cannot add the handler twice.'
                );
            }
            if (event !== 'message') {
                throw Error(
                    'Only "message" event is supported.'
                );
            }
            messageCallback = callback;
        },
    };
}
```

<!-- 0086.part.md -->

<!-- 0087.part.md -->

```js
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

export function showNotification(message, theme = 'dark') {
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

<!-- 0088.part.md -->

<!-- 0089.part.md -->

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

<!-- 0090.part.md -->

<!-- 0091.part.md -->

```css
input {
    display: block;
    margin-bottom: 20px;
}
button {
    margin-left: 10px;
}
```

<!-- 0092.part.md -->

Обратите внимание, что вам больше не нужно знать, как работает `useChatRoom`, чтобы использовать его. Вы можете добавить его в любой другой компонент, передать любые другие параметры, и он будет работать точно так же. В этом и заключается сила пользовательских хуков.

## Когда использовать пользовательские хуки {/_when-to-use-custom-hooks_/}

Вам не нужно извлекать пользовательский хук для каждого маленького дублирующегося кусочка кода. Некоторое дублирование вполне нормально. Например, извлечение хука `useFormInput` для обертывания одного вызова `useState`, как это было ранее, вероятно, не нужно.

Однако всякий раз, когда вы пишете Эффект, подумайте, не будет ли яснее, если его также обернуть в пользовательский Хук. [Эффекты не должны требоваться очень часто,](/learn/you-might-not-need-an-effect) поэтому если вы пишете эффект, это означает, что вам нужно "выйти за пределы React", чтобы синхронизироваться с какой-то внешней системой или сделать что-то, для чего у React нет встроенного API. Обернув это в пользовательский хук, вы можете точно передать свое намерение и то, как данные проходят через него.

Например, рассмотрим компонент `ShippingForm`, который отображает два выпадающих списка: один показывает список городов, а другой - список областей в выбранном городе. Вы можете начать с кода, который выглядит следующим образом:

<!-- 0093.part.md -->

```js
function ShippingForm({ country }) {
  const [cities, setCities] = useState(null);
  // This Effect fetches cities for a country
  useEffect(() => {
    let ignore = false;
    fetch(`/api/cities?country=${country}`)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setCities(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [country]);

  const [city, setCity] = useState(null);
  const [areas, setAreas] = useState(null);
  // This Effect fetches areas for the selected city
  useEffect(() => {
    if (city) {
      let ignore = false;
      fetch(`/api/areas?city=${city}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setAreas(json);
          }
        });
      return () => {
        ignore = true;
      };
    }
  }, [city]);

  // ...
```

<!-- 0094.part.md -->

Хотя этот код довольно повторяющийся, [правильно держать эти Эффекты отдельно друг от друга.](/learn/removing-effect-dependencies#is-your-effect-doing-several-unrelated-things) Они синхронизируют две разные вещи, поэтому не стоит объединять их в один Эффект. Вместо этого вы можете упростить компонент `ShippingForm` выше, извлекая общую логику между ними в свой собственный хук `useData`:

<!-- 0095.part.md -->

```js
function useData(url) {
    const [data, setData] = useState(null);
    useEffect(() => {
        if (url) {
            let ignore = false;
            fetch(url)
                .then((response) => response.json())
                .then((json) => {
                    if (!ignore) {
                        setData(json);
                    }
                });
            return () => {
                ignore = true;
            };
        }
    }, [url]);
    return data;
}
```

<!-- 0096.part.md -->

Теперь вы можете заменить оба Effects в компоненте `ShippingForm` вызовами `useData`:

<!-- 0097.part.md -->

```js
function ShippingForm({ country }) {
  const cities = useData(`/api/cities?country=${country}`);
  const [city, setCity] = useState(null);
  const areas = useData(city ? `/api/areas?city=${city}` : null);
  // ...
```

<!-- 0098.part.md -->

Извлечение пользовательского Hook делает поток данных явным. Вы вводите `url` и получаете `data`. "Пряча" свой Эффект внутри `useData`, вы также не позволяете кому-то, работающему над компонентом `ShippingForm`, добавить к нему [ненужные зависимости](/learn/removing-effect-dependencies). Со временем большая часть Эффектов вашего приложения будет находиться в пользовательских Hooks.

#### Сосредоточьте ваши пользовательские хуки на конкретных высокоуровневых сценариях использования {/_keep-your-custom-hooks-focused-on-concrete-high-level-use-cases_/}

Начните с выбора имени вашего пользовательского хука. Если вы не можете выбрать четкое имя, это может означать, что ваш Эффект слишком связан с остальной логикой вашего компонента и еще не готов к извлечению.

В идеале название вашего пользовательского хука должно быть достаточно понятным, чтобы даже человек, который не часто пишет код, мог догадаться, что делает ваш пользовательский хук, что он принимает и что возвращает:

-   ✅ `useData(url)`
-   ✅ `useImpressionLog(eventName, extraData)`
-   ✅ `useChatRoom(options)`.

Когда вы синхронизируетесь с внешней системой, ваше пользовательское имя Hook может быть более техническим и использовать жаргон, характерный для этой системы. Это хорошо, если оно будет понятно человеку, знакомому с этой системой:

-   ✅ `useMediaQuery(query)`.
-   ✅ `useSocket(url)`
-   ✅ `useIntersectionObserver(ref, options)`

\*\*Избегайте создания и использования пользовательских крючков "жизненного цикла", которые действуют как альтернативы и удобные обертки для самого API `useEffect`:

-   🔴 `useMount(fn)`
-   🔴 `useEffectOnce(fn)`
-   🔴 `useUpdateEffect(fn)`.

Например, этот хук `useMount` пытается обеспечить выполнение некоторого кода только "при монтировании":

<!-- 0099.part.md -->

```js
function ChatRoom({ roomId }) {
    const [serverUrl, setServerUrl] = useState(
        'https://localhost:1234'
    );

    // 🔴 Avoid: using custom "lifecycle" Hooks
    useMount(() => {
        const connection = createConnection({
            roomId,
            serverUrl,
        });
        connection.connect();

        post('/analytics/event', {
            eventName: 'visit_chat',
        });
    });
    // ...
}

// 🔴 Avoid: creating custom "lifecycle" Hooks
function useMount(fn) {
    useEffect(() => {
        fn();
    }, []); // 🔴 React Hook useEffect has a missing dependency: 'fn'
}
```

<!-- 0100.part.md -->

** Пользовательские хуки "жизненного цикла", такие как `useMount`, плохо вписываются в парадигму React.** Например, в этом примере кода есть ошибка (он не "реагирует" на изменения `roomId` или `serverUrl`), но линтер не предупредит вас об этом, потому что линтер проверяет только прямые вызовы `useEffect`. Он не будет знать о вашем крючке.

Если вы пишете эффект, начните с прямого использования React API:

<!-- 0101.part.md -->

```js
function ChatRoom({ roomId }) {
    const [serverUrl, setServerUrl] = useState(
        'https://localhost:1234'
    );

    // ✅ Good: two raw Effects separated by purpose

    useEffect(() => {
        const connection = createConnection({
            serverUrl,
            roomId,
        });
        connection.connect();
        return () => connection.disconnect();
    }, [serverUrl, roomId]);

    useEffect(() => {
        post('/analytics/event', {
            eventName: 'visit_chat',
            roomId,
        });
    }, [roomId]);

    // ...
}
```

<!-- 0102.part.md -->

Затем вы можете (но не обязаны) извлекать пользовательские крючки для различных высокоуровневых сценариев использования:

<!-- 0103.part.md -->

```js
function ChatRoom({ roomId }) {
    const [serverUrl, setServerUrl] = useState(
        'https://localhost:1234'
    );

    // ✅ Great: custom Hooks named after their purpose
    useChatRoom({ serverUrl, roomId });
    useImpressionLog('visit_chat', { roomId });
    // ...
}
```

<!-- 0104.part.md -->

\*\*Например, `useChatRoom(options)` может только подключаться к чату, а `useImpressionLog(eventName, extraData)` может только отправлять журнал впечатлений аналитику. Если ваш пользовательский API Hook не ограничивает сценарии использования и является очень абстрактным, в долгосрочной перспективе он, скорее всего, создаст больше проблем, чем решит.

### Пользовательские хуки помогают перейти на лучшие паттерны {/_custom-hooks-help-you-migrate-to-better-patterns_/}

Эффекты - это ["аварийный люк"] (/learn/escape-hatches): вы используете их, когда вам нужно "выйти за пределы React" и когда нет лучшего встроенного решения для вашего случая использования. Со временем цель команды React - сократить количество Эффектов в вашем приложении до минимума, предоставляя более конкретные решения для более конкретных проблем. Обертывание ваших Эффектов в пользовательские Hooks упрощает обновление вашего кода, когда эти решения становятся доступными.

Давайте вернемся к этому примеру:

<!-- 0105.part.md -->

```js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
    const isOnline = useOnlineStatus();
    return (
        <h1>
            {isOnline ? '✅ Online' : '❌ Disconnected'}
        </h1>
    );
}

function SaveButton() {
    const isOnline = useOnlineStatus();

    function handleSaveClick() {
        console.log('✅ Progress saved');
    }

    return (
        <button
            disabled={!isOnline}
            onClick={handleSaveClick}
        >
            {isOnline ? 'Save progress' : 'Reconnecting...'}
        </button>
    );
}

export default function App() {
    return (
        <>
            <SaveButton />
            <StatusBar />
        </>
    );
}
```

<!-- 0106.part.md -->

<!-- 0107.part.md -->

```js
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
    const [isOnline, setIsOnline] = useState(true);
    useEffect(() => {
        function handleOnline() {
            setIsOnline(true);
        }
        function handleOffline() {
            setIsOnline(false);
        }
        window.addEventListener('online', handleOnline);
        window.addEventListener('offline', handleOffline);
        return () => {
            window.removeEventListener(
                'online',
                handleOnline
            );
            window.removeEventListener(
                'offline',
                handleOffline
            );
        };
    }, []);
    return isOnline;
}
```

<!-- 0108.part.md -->

В приведенном выше примере `useOnlineStatus` реализован с помощью пары [`useState`](/reference/react/useState) и [`useEffect`.](/reference/react/useEffect) Однако это не лучшее из возможных решений. Оно не учитывает ряд побочных ситуаций. Например, предполагается, что когда компонент монтируется, `isOnline` уже `true`, но это может быть неверно, если сеть уже отключилась. Вы можете использовать API браузера [`navigator.onLine`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/onLine) для проверки этого, но его использование напрямую не будет работать на сервере для генерации начального HTML. Короче говоря, этот код можно улучшить.

К счастью, React 18 включает специальный API под названием [`useSyncExternalStore`](/reference/react/useSyncExternalStore), который решает все эти проблемы за вас. Вот как выглядит ваш хук `useOnlineStatus`, переписанный для использования преимуществ этого нового API:

<!-- 0109.part.md -->

```js
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
    const isOnline = useOnlineStatus();
    return (
        <h1>
            {isOnline ? '✅ Online' : '❌ Disconnected'}
        </h1>
    );
}

function SaveButton() {
    const isOnline = useOnlineStatus();

    function handleSaveClick() {
        console.log('✅ Progress saved');
    }

    return (
        <button
            disabled={!isOnline}
            onClick={handleSaveClick}
        >
            {isOnline ? 'Save progress' : 'Reconnecting...'}
        </button>
    );
}

export default function App() {
    return (
        <>
            <SaveButton />
            <StatusBar />
        </>
    );
}
```

<!-- 0110.part.md -->

<!-- 0111.part.md -->

```js
import { useSyncExternalStore } from 'react';

function subscribe(callback) {
    window.addEventListener('online', callback);
    window.addEventListener('offline', callback);
    return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
    };
}

export function useOnlineStatus() {
    return useSyncExternalStore(
        subscribe,
        () => navigator.onLine, // How to get the value on the client
        () => true // How to get the value on the server
    );
}
```

<!-- 0112.part.md -->

Обратите внимание, что **вам не нужно было менять ни один из компонентов**, чтобы осуществить этот переход:

<!-- 0113.part.md -->

```js
function StatusBar() {
    const isOnline = useOnlineStatus();
    // ...
}

function SaveButton() {
    const isOnline = useOnlineStatus();
    // ...
}
```

<!-- 0114.part.md -->

Это еще одна причина, по которой обертывание эффектов в пользовательские крючки часто оказывается полезным:

1.  Вы делаете поток данных к эффектам и от них очень явным.
2.  Вы позволяете компонентам сосредоточиться на замысле, а не на точной реализации ваших эффектов.
3.  Когда React добавляет новые возможности, вы можете удалить эти Эффекты, не меняя ни одного из своих компонентов.

По аналогии с [системой проектирования] (https://uxdesign.cc/everything-you-need-to-know-about-design-systems-54b109851969) вы можете найти полезным начать извлекать общие идиомы из компонентов вашего приложения в пользовательские Hooks. Это позволит сфокусировать код ваших компонентов на замысле и избежать частого написания сырых эффектов. Сообщество React поддерживает множество отличных пользовательских Hooks.

#### Предоставит ли React какое-либо встроенное решение для получения данных? {/_will-react-provide-any-built-in-solution-for-data-fetching_/}

Мы все еще прорабатываем детали, но ожидаем, что в будущем вы будете писать выборку данных следующим образом:

<!-- 0115.part.md -->

```js
import { use } from 'react'; // Not available yet!

function ShippingForm({ country }) {
  const cities = use(fetch(`/api/cities?country=${country}`));
  const [city, setCity] = useState(null);
  const areas = city ? use(fetch(`/api/areas?city=${city}`)) : null;
  // ...
```

<!-- 0116.part.md -->

Если вы используете в своем приложении пользовательские хуки, такие как `useData` выше, то для перехода на рекомендуемый подход потребуется меньше изменений, чем если бы вы писали необработанные Эффекты в каждом компоненте вручную. Однако старый подход все еще будет работать, так что если вам нравится писать необработанные Эффекты, вы можете продолжать это делать.

### Существует более одного способа сделать это {/_there-is-more-than-one-way-to-do-it_/}

Допустим, вы хотите реализовать анимацию затухания _с нуля_, используя API браузера [`requestAnimationFrame`](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame). Вы можете начать с Эффекта, который устанавливает цикл анимации. Во время каждого кадра анимации вы можете изменять непрозрачность узла DOM, который вы [держите в ссылке](/learn/manipulating-the-dom-with-refs), пока она не достигнет `1`. Ваш код может начинаться следующим образом:

<!-- 0117.part.md -->

```js
import { useState, useEffect, useRef } from 'react';

function Welcome() {
    const ref = useRef(null);

    useEffect(() => {
        const duration = 1000;
        const node = ref.current;

        let startTime = performance.now();
        let frameId = null;

        function onFrame(now) {
            const timePassed = now - startTime;
            const progress = Math.min(
                timePassed / duration,
                1
            );
            onProgress(progress);
            if (progress < 1) {
                // We still have more frames to paint
                frameId = requestAnimationFrame(onFrame);
            }
        }

        function onProgress(progress) {
            node.style.opacity = progress;
        }

        function start() {
            onProgress(0);
            startTime = performance.now();
            frameId = requestAnimationFrame(onFrame);
        }

        function stop() {
            cancelAnimationFrame(frameId);
            startTime = null;
            frameId = null;
        }

        start();
        return () => stop();
    }, []);

    return (
        <h1 className="welcome" ref={ref}>
            Welcome
        </h1>
    );
}

export default function App() {
    const [show, setShow] = useState(false);
    return (
        <>
            <button onClick={() => setShow(!show)}>
                {show ? 'Remove' : 'Show'}
            </button>
            <hr />
            {show && <Welcome />}
        </>
    );
}
```

<!-- 0118.part.md -->

<!-- 0119.part.md -->

```css
label,
button {
    display: block;
    margin-bottom: 20px;
}
html,
body {
    min-height: 300px;
}
.welcome {
    opacity: 0;
    color: white;
    padding: 50px;
    text-align: center;
    font-size: 50px;
    background-image: radial-gradient(
        circle,
        rgba(63, 94, 251, 1) 0%,
        rgba(252, 70, 107, 1) 100%
    );
}
```

<!-- 0120.part.md -->

Чтобы сделать компонент более читаемым, вы можете извлечь логику в пользовательский хук `useFadeIn`:

<!-- 0121.part.md -->

```js
import { useState, useEffect, useRef } from 'react';
import { useFadeIn } from './useFadeIn.js';

function Welcome() {
    const ref = useRef(null);

    useFadeIn(ref, 1000);

    return (
        <h1 className="welcome" ref={ref}>
            Welcome
        </h1>
    );
}

export default function App() {
    const [show, setShow] = useState(false);
    return (
        <>
            <button onClick={() => setShow(!show)}>
                {show ? 'Remove' : 'Show'}
            </button>
            <hr />
            {show && <Welcome />}
        </>
    );
}
```

<!-- 0122.part.md -->

<!-- 0123.part.md -->

```js
import { useEffect } from 'react';

export function useFadeIn(ref, duration) {
    useEffect(() => {
        const node = ref.current;

        let startTime = performance.now();
        let frameId = null;

        function onFrame(now) {
            const timePassed = now - startTime;
            const progress = Math.min(
                timePassed / duration,
                1
            );
            onProgress(progress);
            if (progress < 1) {
                // We still have more frames to paint
                frameId = requestAnimationFrame(onFrame);
            }
        }

        function onProgress(progress) {
            node.style.opacity = progress;
        }

        function start() {
            onProgress(0);
            startTime = performance.now();
            frameId = requestAnimationFrame(onFrame);
        }

        function stop() {
            cancelAnimationFrame(frameId);
            startTime = null;
            frameId = null;
        }

        start();
        return () => stop();
    }, [ref, duration]);
}
```

<!-- 0124.part.md -->

<!-- 0125.part.md -->

```css
label,
button {
    display: block;
    margin-bottom: 20px;
}
html,
body {
    min-height: 300px;
}
.welcome {
    opacity: 0;
    color: white;
    padding: 50px;
    text-align: center;
    font-size: 50px;
    background-image: radial-gradient(
        circle,
        rgba(63, 94, 251, 1) 0%,
        rgba(252, 70, 107, 1) 100%
    );
}
```

<!-- 0126.part.md -->

Можно оставить код `useFadeIn` как есть, но можно и рефакторить его. Например, вы можете извлечь логику установки анимационного цикла из `useFadeIn` в пользовательский хук `useAnimationLoop`:

<!-- 0127.part.md -->

```js
import { useState, useEffect, useRef } from 'react';
import { useFadeIn } from './useFadeIn.js';

function Welcome() {
    const ref = useRef(null);

    useFadeIn(ref, 1000);

    return (
        <h1 className="welcome" ref={ref}>
            Welcome
        </h1>
    );
}

export default function App() {
    const [show, setShow] = useState(false);
    return (
        <>
            <button onClick={() => setShow(!show)}>
                {show ? 'Remove' : 'Show'}
            </button>
            <hr />
            {show && <Welcome />}
        </>
    );
}
```

<!-- 0128.part.md -->

<!-- 0129.part.md -->

```js
import { useState, useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export function useFadeIn(ref, duration) {
    const [isRunning, setIsRunning] = useState(true);

    useAnimationLoop(isRunning, (timePassed) => {
        const progress = Math.min(timePassed / duration, 1);
        ref.current.style.opacity = progress;
        if (progress === 1) {
            setIsRunning(false);
        }
    });
}

function useAnimationLoop(isRunning, drawFrame) {
    const onFrame = useEffectEvent(drawFrame);

    useEffect(() => {
        if (!isRunning) {
            return;
        }

        const startTime = performance.now();
        let frameId = null;

        function tick(now) {
            const timePassed = now - startTime;
            onFrame(timePassed);
            frameId = requestAnimationFrame(tick);
        }

        tick();
        return () => cancelAnimationFrame(frameId);
    }, [isRunning]);
}
```

<!-- 0130.part.md -->

<!-- 0131.part.md -->

```css
label,
button {
    display: block;
    margin-bottom: 20px;
}
html,
body {
    min-height: 300px;
}
.welcome {
    opacity: 0;
    color: white;
    padding: 50px;
    text-align: center;
    font-size: 50px;
    background-image: radial-gradient(
        circle,
        rgba(63, 94, 251, 1) 0%,
        rgba(252, 70, 107, 1) 100%
    );
}
```

<!-- 0132.part.md -->

<!-- 0133.part.md -->

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

<!-- 0134.part.md -->

Однако вы не обязаны это делать. Как и в случае с обычными функциями, в конечном итоге вы сами решаете, где проводить границы между различными частями вашего кода. Вы также можете использовать совершенно другой подход. Вместо того чтобы держать логику в Effect, вы можете перенести большую часть императивной логики внутрь JavaScript [class:](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes)

<!-- 0135.part.md -->

```js
import { useState, useEffect, useRef } from 'react';
import { useFadeIn } from './useFadeIn.js';

function Welcome() {
    const ref = useRef(null);

    useFadeIn(ref, 1000);

    return (
        <h1 className="welcome" ref={ref}>
            Welcome
        </h1>
    );
}

export default function App() {
    const [show, setShow] = useState(false);
    return (
        <>
            <button onClick={() => setShow(!show)}>
                {show ? 'Remove' : 'Show'}
            </button>
            <hr />
            {show && <Welcome />}
        </>
    );
}
```

<!-- 0136.part.md -->

<!-- 0137.part.md -->

```js
import { useState, useEffect } from 'react';
import { FadeInAnimation } from './animation.js';

export function useFadeIn(ref, duration) {
    useEffect(() => {
        const animation = new FadeInAnimation(ref.current);
        animation.start(duration);
        return () => {
            animation.stop();
        };
    }, [ref, duration]);
}
```

<!-- 0138.part.md -->

<!-- 0139.part.md -->

```js
export class FadeInAnimation {
    constructor(node) {
        this.node = node;
    }
    start(duration) {
        this.duration = duration;
        this.onProgress(0);
        this.startTime = performance.now();
        this.frameId = requestAnimationFrame(() =>
            this.onFrame()
        );
    }
    onFrame() {
        const timePassed =
            performance.now() - this.startTime;
        const progress = Math.min(
            timePassed / this.duration,
            1
        );
        this.onProgress(progress);
        if (progress === 1) {
            this.stop();
        } else {
            // We still have more frames to paint
            this.frameId = requestAnimationFrame(() =>
                this.onFrame()
            );
        }
    }
    onProgress(progress) {
        this.node.style.opacity = progress;
    }
    stop() {
        cancelAnimationFrame(this.frameId);
        this.startTime = null;
        this.frameId = null;
        this.duration = 0;
    }
}
```

<!-- 0140.part.md -->

<!-- 0141.part.md -->

```css
label,
button {
    display: block;
    margin-bottom: 20px;
}
html,
body {
    min-height: 300px;
}
.welcome {
    opacity: 0;
    color: white;
    padding: 50px;
    text-align: center;
    font-size: 50px;
    background-image: radial-gradient(
        circle,
        rgba(63, 94, 251, 1) 0%,
        rgba(252, 70, 107, 1) 100%
    );
}
```

<!-- 0142.part.md -->

Эффекты позволяют подключать React к внешним системам. Чем больше координации между эффектами требуется (например, для цепочки нескольких анимаций), тем больше смысла извлекать эту логику из эффектов и хуков _полностью_, как в песочнице выше. Тогда извлеченный вами код _станет_ "внешней системой". Это позволяет вашим Эффектам оставаться простыми, потому что им нужно только посылать сообщения системе, которую вы перенесли за пределы React.

Приведенные выше примеры предполагают, что логика затухания должна быть написана на JavaScript. Однако эту конкретную анимацию затухания проще и гораздо эффективнее реализовать с помощью простой [CSS-анимации:](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations/Using_CSS_animations)

<!-- 0143.part.md -->

```js
import { useState, useEffect, useRef } from 'react';
import './welcome.css';

function Welcome() {
    return <h1 className="welcome">Welcome</h1>;
}

export default function App() {
    const [show, setShow] = useState(false);
    return (
        <>
            <button onClick={() => setShow(!show)}>
                {show ? 'Remove' : 'Show'}
            </button>
            <hr />
            {show && <Welcome />}
        </>
    );
}
```

<!-- 0144.part.md -->

<!-- 0145.part.md -->

```css
label,
button {
    display: block;
    margin-bottom: 20px;
}
html,
body {
    min-height: 300px;
}
```

<!-- 0146.part.md -->

<!-- 0147.part.md -->

```css
.welcome {
    color: white;
    padding: 50px;
    text-align: center;
    font-size: 50px;
    background-image: radial-gradient(
        circle,
        rgba(63, 94, 251, 1) 0%,
        rgba(252, 70, 107, 1) 100%
    );

    animation: fadeIn 1000ms;
}

@keyframes fadeIn {
    0% {
        opacity: 0;
    }
    100% {
        opacity: 1;
    }
}
```

<!-- 0148.part.md -->

Иногда даже не нужен Hook\!

\<Recap\>

-   Пользовательские хуки позволяют обмениваться логикой между компонентами.
-   Имена пользовательских хуков должны начинаться с `use` и заканчиваться заглавной буквой.
-   Пользовательские хуки передают только логику состояния, но не само состояние.
-   Вы можете передавать реактивные значения от одного хука к другому, и они остаются актуальными.
-   Все хуки перезапускаются каждый раз, когда ваш компонент перерендеривается.
-   Код ваших пользовательских хуков должен быть чистым, как и код вашего компонента.
-   Оберните обработчики событий, получаемые пользовательскими хуками, в события Effect Events.
-   Не создавайте пользовательские хуки типа `useMount`. Их назначение должно быть конкретным.
-   Вам решать, как и где выбирать границы вашего кода.

\</Recap\>

\<Проблемы\>

#### Extract a `useCounter` Hook {/_extract-a-usecounter-hook_/}

Этот компонент использует переменную состояния и Эффект для отображения числа, которое увеличивается каждую секунду. Извлеките эту логику в пользовательский хук под названием `useCounter`. Ваша цель состоит в том, чтобы реализация компонента `Counter` выглядела именно так:

<!-- 0149.part.md -->

```js
export default function Counter() {
    const count = useCounter();
    return <h1>Seconds passed: {count}</h1>;
}
```

<!-- 0150.part.md -->

Вам нужно будет написать свой пользовательский Hook в файле `useCounter.js` и импортировать его в файл `Counter.js`.

<!-- 0151.part.md -->

```js
import { useState, useEffect } from 'react';

export default function Counter() {
    const [count, setCount] = useState(0);
    useEffect(() => {
        const id = setInterval(() => {
            setCount((c) => c + 1);
        }, 1000);
        return () => clearInterval(id);
    }, []);
    return <h1>Seconds passed: {count}</h1>;
}
```

<!-- 0152.part.md -->

<!-- 0153.part.md -->

```js
// Write your custom Hook in this file!
```

<!-- 0154.part.md -->

\<Решение\>

Ваш код должен выглядеть следующим образом:

<!-- 0155.part.md -->

```js
import { useCounter } from './useCounter.js';

export default function Counter() {
    const count = useCounter();
    return <h1>Seconds passed: {count}</h1>;
}
```

<!-- 0156.part.md -->

<!-- 0157.part.md -->

```js
import { useState, useEffect } from 'react';

export function useCounter() {
    const [count, setCount] = useState(0);
    useEffect(() => {
        const id = setInterval(() => {
            setCount((c) => c + 1);
        }, 1000);
        return () => clearInterval(id);
    }, []);
    return count;
}
```

<!-- 0158.part.md -->

Обратите внимание, что `App.js` больше не нужно импортировать `useState` или `useEffect`.

\</Solution\>

#### Сделайте задержку счетчика настраиваемой {/_make-the-counter-delay-configurable_/}

В этом примере есть переменная состояния `delay`, управляемая ползунком, но ее значение не используется. Передайте значение `delay` в ваш пользовательский хук `useCounter`, и измените хук `useCounter`, чтобы он использовал переданную `delay` вместо жесткого кодирования `1000` мс.

<!-- 0159.part.md -->

```js
import { useState } from 'react';
import { useCounter } from './useCounter.js';

export default function Counter() {
    const [delay, setDelay] = useState(1000);
    const count = useCounter();
    return (
        <>
            <label>
                Tick duration: {delay} ms
                <br />
                <input
                    type="range"
                    value={delay}
                    min="10"
                    max="2000"
                    onChange={(e) =>
                        setDelay(Number(e.target.value))
                    }
                />
            </label>
            <hr />
            <h1>Ticks: {count}</h1>
        </>
    );
}
```

<!-- 0160.part.md -->

<!-- 0161.part.md -->

```js
import { useState, useEffect } from 'react';

export function useCounter() {
    const [count, setCount] = useState(0);
    useEffect(() => {
        const id = setInterval(() => {
            setCount((c) => c + 1);
        }, 1000);
        return () => clearInterval(id);
    }, []);
    return count;
}
```

<!-- 0162.part.md -->

\<Решение\>

Передайте `задержку` вашему хуку с помощью `useCounter(delay)`. Затем, внутри хука, используйте `delay` вместо жестко заданного значения `1000`. Вам нужно будет добавить `delay` в зависимости вашего Эффекта. Это гарантирует, что изменение `delay` сбросит интервал.

<!-- 0163.part.md -->

```js
import { useState } from 'react';
import { useCounter } from './useCounter.js';

export default function Counter() {
    const [delay, setDelay] = useState(1000);
    const count = useCounter(delay);
    return (
        <>
            <label>
                Tick duration: {delay} ms
                <br />
                <input
                    type="range"
                    value={delay}
                    min="10"
                    max="2000"
                    onChange={(e) =>
                        setDelay(Number(e.target.value))
                    }
                />
            </label>
            <hr />
            <h1>Ticks: {count}</h1>
        </>
    );
}
```

<!-- 0164.part.md -->

<!-- 0165.part.md -->

```js
import { useState, useEffect } from 'react';

export function useCounter(delay) {
    const [count, setCount] = useState(0);
    useEffect(() => {
        const id = setInterval(() => {
            setCount((c) => c + 1);
        }, delay);
        return () => clearInterval(id);
    }, [delay]);
    return count;
}
```

<!-- 0166.part.md -->

\</Solution\>

#### Извлечение `useInterval` из `useCounter` {/_extract-useinterval-out-of-usecounter_/}

В настоящее время ваш хук `useCounter` делает две вещи. Он устанавливает интервал, а также увеличивает переменную состояния при каждом тике интервала. Выделите логику, которая устанавливает интервал, в отдельный хук под названием `useInterval`. Он должен принимать два аргумента: обратный вызов `onTick` и `delay`. После этого изменения ваша реализация `useCounter` должна выглядеть следующим образом:

<!-- 0167.part.md -->

```js
export function useCounter(delay) {
    const [count, setCount] = useState(0);
    useInterval(() => {
        setCount((c) => c + 1);
    }, delay);
    return count;
}
```

<!-- 0168.part.md -->

Напишите `useInterval` в файле `useInterval.js` и импортируйте его в файл `useCounter.js`.

<!-- 0169.part.md -->

```js
import { useState } from 'react';
import { useCounter } from './useCounter.js';

export default function Counter() {
    const count = useCounter(1000);
    return <h1>Seconds passed: {count}</h1>;
}
```

<!-- 0170.part.md -->

<!-- 0171.part.md -->

```js
import { useState, useEffect } from 'react';

export function useCounter(delay) {
    const [count, setCount] = useState(0);
    useEffect(() => {
        const id = setInterval(() => {
            setCount((c) => c + 1);
        }, delay);
        return () => clearInterval(id);
    }, [delay]);
    return count;
}
```

<!-- 0172.part.md -->

<!-- 0173.part.md -->

```js
// Write your Hook here!
```

<!-- 0174.part.md -->

\<Решение\>

Логика внутри `useInterval` должна установить и очистить интервал. Больше ничего делать не нужно.

<!-- 0175.part.md -->

```js
import { useCounter } from './useCounter.js';

export default function Counter() {
    const count = useCounter(1000);
    return <h1>Seconds passed: {count}</h1>;
}
```

<!-- 0176.part.md -->

<!-- 0177.part.md -->

```js
import { useState } from 'react';
import { useInterval } from './useInterval.js';

export function useCounter(delay) {
    const [count, setCount] = useState(0);
    useInterval(() => {
        setCount((c) => c + 1);
    }, delay);
    return count;
}
```

<!-- 0178.part.md -->

<!-- 0179.part.md -->

```js
import { useEffect } from 'react';

export function useInterval(onTick, delay) {
    useEffect(() => {
        const id = setInterval(onTick, delay);
        return () => clearInterval(id);
    }, [onTick, delay]);
}
```

<!-- 0180.part.md -->

Обратите внимание, что в этом решении есть небольшая проблема, которую вы решите в следующей задаче.

\</Solution\>

#### Исправить интервал сброса {/_fix-a-resetting-interval_/}

В этом примере есть _два_ отдельных интервала.

Компонент `App` вызывает `useCounter`, который вызывает `useInterval` для обновления счетчика каждую секунду. Но компонент `App` _также_ вызывает `useInterval` для случайного обновления цвета фона страницы каждые две секунды.

По какой-то причине обратный вызов, обновляющий фон страницы, никогда не выполняется. Добавьте несколько журналов внутри `useInterval`:

<!-- 0181.part.md -->

```js
useEffect(() => {
    console.log(
        '✅ Setting up an interval with delay ',
        delay
    );
    const id = setInterval(onTick, delay);
    return () => {
        console.log(
            '❌ Clearing an interval with delay ',
            delay
        );
        clearInterval(id);
    };
}, [onTick, delay]);
```

<!-- 0182.part.md -->

Совпадают ли журналы с тем, что вы ожидаете? Если некоторые из ваших Эффектов, кажется, пересинхронизируются без необходимости, можете ли вы предположить, какая зависимость вызывает это? Есть ли способ [удалить эту зависимость](/learn/removing-effect-dependencies) из вашего Эффекта?

После устранения проблемы, вы должны ожидать, что фон страницы будет обновляться каждые две секунды.

\<Hint\>

Похоже, что ваш хук `useInterval` принимает в качестве аргумента слушатель событий. Можете ли вы придумать, как обернуть этот слушатель событий так, чтобы он не был зависим от вашего Effect?

\</Hint\>

<!-- 0183.part.md -->

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

<!-- 0184.part.md -->

<!-- 0185.part.md -->

```js
import { useCounter } from './useCounter.js';
import { useInterval } from './useInterval.js';

export default function Counter() {
    const count = useCounter(1000);

    useInterval(() => {
        const randomColor = `hsla(${
            Math.random() * 360
        }, 100%, 50%, 0.2)`;
        document.body.style.backgroundColor = randomColor;
    }, 2000);

    return <h1>Seconds passed: {count}</h1>;
}
```

<!-- 0186.part.md -->

<!-- 0187.part.md -->

```js
import { useState } from 'react';
import { useInterval } from './useInterval.js';

export function useCounter(delay) {
    const [count, setCount] = useState(0);
    useInterval(() => {
        setCount((c) => c + 1);
    }, delay);
    return count;
}
```

<!-- 0188.part.md -->

<!-- 0189.part.md -->

```js
import { useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export function useInterval(onTick, delay) {
    useEffect(() => {
        const id = setInterval(onTick, delay);
        return () => {
            clearInterval(id);
        };
    }, [onTick, delay]);
}
```

<!-- 0190.part.md -->

\<Решение\>

Внутри `useInterval` оберните обратный вызов тика в событие эффекта, как вы делали [ранее на этой странице](/learn/reusing-logic-with-custom-hooks#passing-event-handlers-to-custom-hooks).

Это позволит вам опустить `onTick` из зависимостей вашего Эффекта. Эффект не будет пересинхронизироваться при каждом повторном рендере компонента, поэтому интервал изменения цвета фона страницы не будет сбрасываться каждую секунду, прежде чем успеет сработать.

Благодаря этому изменению оба интервала работают, как и ожидалось, и не мешают друг другу:

<!-- 0191.part.md -->

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

<!-- 0192.part.md -->

<!-- 0193.part.md -->

```js
import { useCounter } from './useCounter.js';
import { useInterval } from './useInterval.js';

export default function Counter() {
    const count = useCounter(1000);

    useInterval(() => {
        const randomColor = `hsla(${
            Math.random() * 360
        }, 100%, 50%, 0.2)`;
        document.body.style.backgroundColor = randomColor;
    }, 2000);

    return <h1>Seconds passed: {count}</h1>;
}
```

<!-- 0194.part.md -->

<!-- 0195.part.md -->

```js
import { useState } from 'react';
import { useInterval } from './useInterval.js';

export function useCounter(delay) {
    const [count, setCount] = useState(0);
    useInterval(() => {
        setCount((c) => c + 1);
    }, delay);
    return count;
}
```

<!-- 0196.part.md -->

<!-- 0197.part.md -->

```js
import { useEffect } from 'react';
import { experimental_useEffectEvent as useEffectEvent } from 'react';

export function useInterval(callback, delay) {
    const onTick = useEffectEvent(callback);
    useEffect(() => {
        const id = setInterval(onTick, delay);
        return () => clearInterval(id);
    }, [delay]);
}
```

<!-- 0198.part.md -->

\</Solution\>

#### Реализация шагающего движения {/_implement-a-staggering-movement_/}

В этом примере хук `usePointerPosition()` отслеживает текущую позицию указателя. Попробуйте переместить курсор или палец по области предварительного просмотра и увидите, как красная точка следует за вашим движением. Ее положение сохраняется в переменной `pos1`.

На самом деле, в данный момент отображается пять (\!) различных красных точек. Вы не видите их, потому что в настоящее время все они отображаются в одном и том же положении. Это то, что вам нужно исправить. Вместо этого вы хотите реализовать "ступенчатое" движение: каждая точка должна "следовать" по пути предыдущей точки. Например, если вы быстро перемещаете курсор, первая точка должна следовать за ним немедленно, вторая точка должна следовать за первой с небольшой задержкой, третья точка должна следовать за второй и так далее.

Вам необходимо реализовать пользовательский хук `useDelayedValue`. Его текущая реализация возвращает предоставленное ему `значение`. Вместо этого вы хотите возвращать значение, полученное от `задержки` миллисекунды назад. Для этого вам может понадобиться некоторое состояние и Эффект.

После реализации `useDelayedValue`, вы должны увидеть, как точки движутся друг за другом.

\<Намек\>

Вам нужно будет хранить `delayedValue` как переменную состояния внутри вашего пользовательского Hook. Когда `значение` изменится, вы захотите запустить Эффект. Этот Эффект должен обновить `delayedValue` после `задержки`. Возможно, вам будет полезно вызвать `setTimeout`.

Нужно ли очистить этот Эффект? Почему или почему нет?

\</Hint\>

<!-- 0199.part.md -->

```js
import { usePointerPosition } from './usePointerPosition.js';

function useDelayedValue(value, delay) {
    // TODO: Implement this Hook
    return value;
}

export default function Canvas() {
    const pos1 = usePointerPosition();
    const pos2 = useDelayedValue(pos1, 100);
    const pos3 = useDelayedValue(pos2, 200);
    const pos4 = useDelayedValue(pos3, 100);
    const pos5 = useDelayedValue(pos3, 50);
    return (
        <>
            <Dot position={pos1} opacity={1} />
            <Dot position={pos2} opacity={0.8} />
            <Dot position={pos3} opacity={0.6} />
            <Dot position={pos4} opacity={0.4} />
            <Dot position={pos5} opacity={0.2} />
        </>
    );
}

function Dot({ position, opacity }) {
    return (
        <div
            style={{
                position: 'absolute',
                backgroundColor: 'pink',
                borderRadius: '50%',
                opacity,
                transform: `translate(${position.x}px, ${position.y}px)`,
                pointerEvents: 'none',
                left: -20,
                top: -20,
                width: 40,
                height: 40,
            }}
        />
    );
}
```

<!-- 0200.part.md -->

<!-- 0201.part.md -->

```js
import { useState, useEffect } from 'react';

export function usePointerPosition() {
    const [position, setPosition] = useState({
        x: 0,
        y: 0,
    });
    useEffect(() => {
        function handleMove(e) {
            setPosition({ x: e.clientX, y: e.clientY });
        }
        window.addEventListener('pointermove', handleMove);
        return () =>
            window.removeEventListener(
                'pointermove',
                handleMove
            );
    }, []);
    return position;
}
```

<!-- 0202.part.md -->

<!-- 0203.part.md -->

```css
body {
    min-height: 300px;
}
```

<!-- 0204.part.md -->

\<Решение\>

Вот рабочая версия. Вы храните `delayedValue` как переменную состояния. Когда `значение` обновляется, ваш Effect планирует таймаут для обновления `отложенного значения`. Вот почему `delayedValue` всегда "отстает" от фактического `value`.

<!-- 0205.part.md -->

```js
import { useState, useEffect } from 'react';
import { usePointerPosition } from './usePointerPosition.js';

function useDelayedValue(value, delay) {
    const [delayedValue, setDelayedValue] = useState(value);

    useEffect(() => {
        setTimeout(() => {
            setDelayedValue(value);
        }, delay);
    }, [value, delay]);

    return delayedValue;
}

export default function Canvas() {
    const pos1 = usePointerPosition();
    const pos2 = useDelayedValue(pos1, 100);
    const pos3 = useDelayedValue(pos2, 200);
    const pos4 = useDelayedValue(pos3, 100);
    const pos5 = useDelayedValue(pos3, 50);
    return (
        <>
            <Dot position={pos1} opacity={1} />
            <Dot position={pos2} opacity={0.8} />
            <Dot position={pos3} opacity={0.6} />
            <Dot position={pos4} opacity={0.4} />
            <Dot position={pos5} opacity={0.2} />
        </>
    );
}

function Dot({ position, opacity }) {
    return (
        <div
            style={{
                position: 'absolute',
                backgroundColor: 'pink',
                borderRadius: '50%',
                opacity,
                transform: `translate(${position.x}px, ${position.y}px)`,
                pointerEvents: 'none',
                left: -20,
                top: -20,
                width: 40,
                height: 40,
            }}
        />
    );
}
```

<!-- 0206.part.md -->

<!-- 0207.part.md -->

```js
import { useState, useEffect } from 'react';

export function usePointerPosition() {
    const [position, setPosition] = useState({
        x: 0,
        y: 0,
    });
    useEffect(() => {
        function handleMove(e) {
            setPosition({ x: e.clientX, y: e.clientY });
        }
        window.addEventListener('pointermove', handleMove);
        return () =>
            window.removeEventListener(
                'pointermove',
                handleMove
            );
    }, []);
    return position;
}
```

<!-- 0208.part.md -->

<!-- 0209.part.md -->

```css
body {
    min-height: 300px;
}
```

<!-- 0210.part.md -->

Обратите внимание, что этот Эффект _не_ нуждается в очистке. Если бы вы вызвали `clearTimeout` в функции очистки, то при каждом изменении `значения` сбрасывался бы уже запланированный таймаут. Чтобы движение было непрерывным, нужно, чтобы срабатывали все таймауты.

\</Solution\>

\</Challenges\>

<!-- 0211.part.md -->

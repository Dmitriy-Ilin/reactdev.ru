# useSyncExternalStore

**`useSyncExternalStore`** - это хук React, позволяющий подписаться на внешнее хранилище.

<!-- 0001.part.md -->

```js
const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)
```

## Описание

### `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`

Вызовите `useSyncExternalStore` на верхнем уровне вашего компонента для чтения значения из внешнего хранилища данных.

<!-- 0003.part.md -->

```js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
    const todos = useSyncExternalStore(
        todosStore.subscribe,
        todosStore.getSnapshot
    );
    // ...
}
```

<!-- 0004.part.md -->

Она возвращает моментальный снимок данных в хранилище. В качестве аргументов необходимо передать две функции:

1.  Функция `subscribe должна подписываться на стор и возвращать функцию, которая отписывается.
2.  Функция `getSnapshot` должна считывать моментальный снимок данных из хранилища.

#### Параметры

-   `subscribe`: Функция, принимающая один аргумент `callback` и подписывающаяся на стор. Когда стор изменяется, он должен вызвать предоставленный `callback`. Это приведет к повторному отображению компонента. Функция `subscribe` должна возвращать функцию, которая очищает подписку.

-   `getSnapshot`: Функция, которая возвращает снимок данных в хранилище, необходимых компоненту. Пока хранилище не изменилось, повторные вызовы `getSnapshot` должны возвращать одно и то же значение. Если стор изменился, а возвращаемое значение отличается (по сравнению с [`Object.is`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), React перерисовывает компонент.

-   **опциональная** `getServerSnapshot`: Функция, возвращающая начальный снимок данных в хранилище. Он будет использоваться только при рендеринге сервера и при гидратации рендеримого сервером контента на клиенте. Серверный снимок должен быть одинаковым для клиента и сервера, и обычно сериализуется и передается от сервера к клиенту. Если вы опустите этот аргумент, рендеринг компонента на сервере приведет к ошибке.

#### Возвращает

Текущий снимок хранилища, который вы можете использовать в своей логике рендеринга.

#### Ограничения

-   Снимок хранилища, возвращаемый `getSnapshot`, должен быть неизменяемым. Если базовое хранилище имеет изменяемые данные, верните новый неизменяемый снимок, если данные изменились. В противном случае возвращается кэшированный последний снимок.

-   Если во время повторного рендеринга передается другая функция `subscribe`, React повторно подпишется на стор, используя новую переданную функцию `subscribe`. Вы можете предотвратить это, объявив `subscribe` вне компонента.

## Использование

### Подписка на внешний стор

Большинство ваших компонентов React будут читать данные только из своих [props,](../../learn/passing-props-to-a-component.md) [state](useState.md), и [context](useContext.md). Однако иногда компоненту необходимо читать данные из какого-то хранилища вне React, которое меняется со временем. К ним относятся:

-   Сторонние библиотеки управления состоянием, которые хранят состояние вне React.
-   Браузерные API, которые предоставляют изменяемое значение и события для подписки на его изменения.

Вызовите `useSyncExternalStore` на верхнем уровне вашего компонента, чтобы прочитать значение из внешнего хранилища данных.

<!-- 0005.part.md -->

```js
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
    const todos = useSyncExternalStore(
        todosStore.subscribe,
        todosStore.getSnapshot
    );
    // ...
}
```

<!-- 0006.part.md -->

Она возвращает моментальный снимок данных в хранилище. Вам нужно передать две функции в качестве аргументов:

1.  Функция `subscribe` должна подписываться на стор и возвращать функцию, которая отписывается.
2.  `getSnapshot` функция должна прочитать снимок данных из стора.

React будет использовать эти функции, чтобы сохранить ваш компонент подписанным на стор и перерисовывать его при изменениях.

Например, в песочнице ниже, `todosStore` реализован как внешний стор, который хранит данные вне React. Компонент `TodosApp` подключается к этому внешнему хранилищу с помощью хука `useSyncExternalStore`.

=== "App.js"

    ```js
    import { useSyncExternalStore } from 'react';
    import { todosStore } from './todoStore.js';

    export default function TodosApp() {
    	const todos = useSyncExternalStore(
    		todosStore.subscribe,
    		todosStore.getSnapshot
    	);
    	return (
    		<>
    			<button onClick={() => todosStore.addTodo()}>
    				Add todo
    			</button>
    			<hr />
    			<ul>
    				{todos.map((todo) => (
    					<li key={todo.id}>{todo.text}</li>
    				))}
    			</ul>
    		</>
    	);
    }
    ```

=== "todoStore.js"

    ```js
    // This is an example of a third-party store
    // that you might need to integrate with React.

    // If your app is fully built with React,
    // we recommend using React state instead.

    let nextId = 0;
    let todos = [{ id: nextId++, text: 'Todo #1' }];
    let listeners = [];

    export const todosStore = {
    	addTodo() {
    		todos = [
    			...todos,
    			{ id: nextId++, text: 'Todo #' + nextId },
    		];
    		emitChange();
    	},
    	subscribe(listener) {
    		listeners = [...listeners, listener];
    		return () => {
    			listeners = listeners.filter(
    				(l) => l !== listener
    			);
    		};
    	},
    	getSnapshot() {
    		return todos;
    	},
    };

    function emitChange() {
    	for (let listener of listeners) {
    		listener();
    	}
    }
    ```

!!!note ""

    Когда это возможно, мы рекомендуем использовать встроенное состояние React с помощью [`useState`](useState.md) и [`useReducer`](useReducer.md). API `useSyncExternalStore` в основном полезен, если вам нужно интегрироваться с существующим не-React кодом.

### Подписка на API браузера

Еще одна причина добавить `useSyncExternalStore` - это когда вы хотите подписаться на какое-то значение, предоставляемое браузером, которое меняется со временем. Например, предположим, что вы хотите, чтобы ваш компонент отображал, активно ли сетевое соединение. Браузер предоставляет эту информацию через свойство [`navigator.onLine`](https://developer.mozilla.org/docs/Web/API/Navigator/onLine).

Это значение может меняться без ведома React, поэтому вы должны считывать его с помощью `useSyncExternalStore`.

<!-- 0011.part.md -->

```js
import { useSyncExternalStore } from 'react';

function ChatIndicator() {
    const isOnline = useSyncExternalStore(
        subscribe,
        getSnapshot
    );
    // ...
}
```

<!-- 0012.part.md -->

Чтобы реализовать функцию `getSnapshot`, прочитайте текущее значение из API браузера:

<!-- 0013.part.md -->

```js
function getSnapshot() {
    return navigator.onLine;
}
```

<!-- 0014.part.md -->

Далее необходимо реализовать функцию `subscribe`. Например, при изменении `navigator.onLine` браузер запускает события [`online`](https://developer.mozilla.org/docs/Web/API/Window/online_event) и [`offline`](https://developer.mozilla.org/docs/Web/API/Window/offline_event) на объекте `window`. Вам нужно подписать аргумент `callback` на соответствующие события, а затем вернуть функцию, которая очистит подписки:

<!-- 0015.part.md -->

```js
function subscribe(callback) {
    window.addEventListener('online', callback);
    window.addEventListener('offline', callback);
    return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
    };
}
```

<!-- 0016.part.md -->

Теперь React знает, как прочитать значение из внешнего API `navigator.onLine` и как подписаться на его изменения. Отключите устройство от сети и обратите внимание на то, что в ответ на это компонент снова отображается:

<!-- 0017.part.md -->

```js
import { useSyncExternalStore } from 'react';

export default function ChatIndicator() {
    const isOnline = useSyncExternalStore(
        subscribe,
        getSnapshot
    );
    return (
        <h1>
            {isOnline ? '✅ Online' : '❌ Disconnected'}
        </h1>
    );
}

function getSnapshot() {
    return navigator.onLine;
}

function subscribe(callback) {
    window.addEventListener('online', callback);
    window.addEventListener('offline', callback);
    return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
    };
}
```

### Извлечение логики в пользовательский хук

Обычно вы не будете писать `useSyncExternalStore` непосредственно в своих компонентах. Вместо этого вы, как правило, вызываете его из своего собственного пользовательского хука. Это позволяет вам использовать одно и то же внешнее хранилище в разных компонентах.

Например, этот пользовательский хук `useOnlineStatus` отслеживает, находится ли сеть в режиме онлайн:

<!-- 0019.part.md -->

```js
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
    const isOnline = useSyncExternalStore(
        subscribe,
        getSnapshot
    );
    return isOnline;
}

function getSnapshot() {
    // ...
}

function subscribe(callback) {
    // ...
}
```

<!-- 0020.part.md -->

Теперь различные компоненты могут вызывать `useOnlineStatus` без повторения базовой реализации:

=== "App.js"

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

=== "useOnlineStatus.js"

    ```js
    import { useSyncExternalStore } from 'react';

    export function useOnlineStatus() {
    	const isOnline = useSyncExternalStore(
    		subscribe,
    		getSnapshot
    	);
    	return isOnline;
    }

    function getSnapshot() {
    	return navigator.onLine;
    }

    function subscribe(callback) {
    	window.addEventListener('online', callback);
    	window.addEventListener('offline', callback);
    	return () => {
    		window.removeEventListener('online', callback);
    		window.removeEventListener('offline', callback);
    	};
    }
    ```

### Добавление поддержки серверного рендеринга

Если ваше приложение React использует [серверный рендеринг](server.md), ваши компоненты React также будут запускаться вне среды браузера для генерации начального HTML. Это создает несколько проблем при подключении к внешнему стору:

-   Если вы подключаетесь к API только для браузера, он не будет работать, поскольку не существует на сервере.
-   Если вы подключаетесь к стороннему хранилищу данных, вам потребуется, чтобы его данные совпадали на сервере и клиенте.

Чтобы решить эти проблемы, передайте функцию `getServerSnapshot` в качестве третьего аргумента в `useSyncExternalStore`:

<!-- 0025.part.md -->

```js
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
    const isOnline = useSyncExternalStore(
        subscribe,
        getSnapshot,
        getServerSnapshot
    );
    return isOnline;
}

function getSnapshot() {
    return navigator.onLine;
}

function getServerSnapshot() {
    return true; // Always show "Online" for server-generated HTML
}

function subscribe(callback) {
    // ...
}
```

<!-- 0026.part.md -->

Функция `getServerSnapshot` похожа на `getSnapshot`, но запускается только в двух ситуациях:

-   Она запускается на сервере при генерации HTML.
-   Она запускается на клиенте во время [hydration](client-hydrateRoot.md), т. е. когда React берет серверный HTML и делает его интерактивным.

Это позволяет указать начальное значение snapshot, которое будет использоваться до того, как приложение станет интерактивным. Если нет значимого начального значения для серверного рендеринга, опустите этот аргумент для [принудительного рендеринга на клиенте](Suspense.md).

!!!note ""

    Убедитесь, что `getServerSnapshot` возвращает те же данные при первоначальном рендеринге на клиенте, что и на сервере. Например, если `getServerSnapshot` вернул некоторое предварительно заполненное содержимое стора на сервере, вам нужно передать это содержимое клиенту. Один из способов сделать это - выдать тег `<script>` во время рендеринга сервера, который устанавливает глобал типа `window.MY_STORE_DATA`, и читать из этого глобала на клиенте в `getServerSnapshot`. Ваш внешний стор должен предоставить инструкции о том, как это сделать.

## Устранение неполадок

### Я получаю ошибку: "The result of `getSnapshot` should be cached"

Эта ошибка означает, что ваша функция `getSnapshot` возвращает новый объект при каждом вызове, например:

<!-- 0027.part.md -->

```js
function getSnapshot() {
    // 🔴 Do not return always different objects from getSnapshot
    return {
        todos: myStore.todos,
    };
}
```

<!-- 0028.part.md -->

React будет перерисовывать компонент, если возвращаемое значение `getSnapshot` отличается от предыдущего. Вот почему, если вы всегда возвращаете другое значение, вы попадете в бесконечный цикл и получите эту ошибку.

Ваш объект `getSnapshot` должен возвращать другой объект только в том случае, если что-то действительно изменилось. Если ваше хранилище содержит неизменяемые данные, вы можете возвращать эти данные напрямую:

<!-- 0029.part.md -->

```js
function getSnapshot() {
    // ✅ You can return immutable data
    return myStore.todos;
}
```

<!-- 0030.part.md -->

Если данные вашего стора изменчивы, ваша функция `getSnapshot` должна возвращать неизменяемый снимок. Это означает, что ей _нужно_ создавать новые объекты, но она не должна делать это при каждом вызове. Вместо этого она должна хранить последний вычисленный снимок и возвращать тот же снимок, что и в прошлый раз, если данные в хранилище не изменились. Как определить, изменились ли изменяемые данные, зависит от вашего хранилища изменяемых данных.

### Моя функция `subscribe` вызывается после каждого рендеринга

Эта функция `subscribe` определена _внутри_ компонента, поэтому при каждом повторном рендере она будет другой:

<!-- 0031.part.md -->

```js
function ChatIndicator() {
    const isOnline = useSyncExternalStore(
        subscribe,
        getSnapshot
    );

    // 🚩 Always a different function, so React will resubscribe on every re-render
    function subscribe() {
        // ...
    }

    // ...
}
```

<!-- 0032.part.md -->

React повторно подпишется на ваш стор, если вы передадите другую функцию `subscribe` между повторными рендерами. Если это вызывает проблемы с производительностью и вы хотите избежать повторной подписки, переместите функцию `subscribe` наружу:

<!-- 0033.part.md -->

```js
function ChatIndicator() {
    const isOnline = useSyncExternalStore(
        subscribe,
        getSnapshot
    );
    // ...
}

// ✅ Always the same function, so React won't need to resubscribe
function subscribe() {
    // ...
}
```

<!-- 0034.part.md -->

Как вариант, оберните `subscribe` в [`useCallback`](useCallback.md), чтобы повторно подписываться только при изменении какого-либо аргумента:

<!-- 0035.part.md -->

```js
function ChatIndicator({ userId }) {
    const isOnline = useSyncExternalStore(
        subscribe,
        getSnapshot
    );

    // ✅ Same function as long as userId doesn't change
    const subscribe = useCallback(() => {
        // ...
    }, [userId]);

    // ...
}
```

<!-- 0036.part.md -->

<!-- 0037.part.md -->

## Ссылки

-   [https://react.dev/reference/react/useSyncExternalStore](https://react.dev/reference/react/useSyncExternalStore)

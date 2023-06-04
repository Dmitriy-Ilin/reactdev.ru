# useState

**`useState`** - это хук React, который позволяет вам добавить [переменную состояния](../learn/state-a-components-memory.md) в ваш компонент.

<!-- 0001.part.md -->

```js
const [state, setState] = useState(initialState);
```

## Описание

### `useState(initialState)`

Вызовите `useState` на верхнем уровне вашего компонента, чтобы объявить [переменную состояния.](../learn/state-a-components-memory.md)

<!-- 0003.part.md -->

```js
import { useState } from 'react';

function MyComponent() {
    const [age, setAge] = useState(28);
    const [name, setName] = useState('Taylor');
    const [todos, setTodos] = useState(() => createTodos());
    // ...
}
```

<!-- 0004.part.md -->

Принято называть переменные состояния типа `[something, setSomething]`, используя [деструктуризацию массива.](https://javascript.info/destructuring-assignment)

#### Параметры

-   `initialState`: Значение, которое вы хотите, чтобы состояние было первоначальным. Это может быть значение любого типа, но для функций предусмотрено особое поведение. Этот аргумент игнорируется после первоначального рендеринга.
    -   Если вы передадите функцию в качестве `initialState`, она будет рассматриваться как _инициализирующая функция_. Она должна быть чистой, не принимать никаких аргументов и возвращать значение любого типа. React будет вызывать вашу функцию инициализатора при инициализации компонента и сохранять возвращаемое значение в качестве начального состояния.

#### Возвраты

`useState` возвращает массив, содержащий ровно два значения:

1.  Текущее состояние. Во время первого рендера оно будет соответствовать переданному вами `initialState`.
2.  Функция `set`, которая позволяет обновить состояние до другого значения и вызвать повторный рендеринг.

#### Предупреждения

-   `useState` - это хук, поэтому вы можете вызывать его только **на верхнем уровне вашего компонента** или ваших собственных хуков. Вы не можете вызывать его внутри циклов или условий. Если вам это нужно, создайте новый компонент и переместите состояние в него.
-   В строгом режиме React будет **вызывать вашу функцию инициализатора дважды**, чтобы помочь вам найти случайные примеси. Это поведение только для разработки и не влияет на производство. Если ваша функция инициализатора чиста (как и должно быть), это не должно повлиять на поведение. Результат одного из вызовов будет проигнорирован.

### `set` функции, такие как `setSomething(nextState)`

Функция `set`, возвращаемая `useState`, позволяет обновить состояние до другого значения и вызвать повторный рендеринг. Вы можете передать следующее состояние напрямую или функцию, которая вычисляет его на основе предыдущего состояния:

<!-- 0005.part.md -->

```js
const [name, setName] = useState('Edward');

function handleClick() {
    setName('Taylor');
    setAge((a) => a + 1);
    // ...
}
```

<!-- 0006.part.md -->

#### Параметры

-   `nextState`: Значение, которое вы хотите видеть в состоянии. Это может быть значение любого типа, но есть особое поведение для функций.
    -   Если вы передадите функцию в качестве `nextState`, она будет рассматриваться как _функция обновления_. Она должна быть чистой, принимать состояние ожидания в качестве единственного аргумента и возвращать следующее состояние. React поместит вашу функцию обновления в очередь и перерендерит ваш компонент. Во время следующего рендеринга React вычислит следующее состояние, применив все стоящие в очереди функции обновления к предыдущему состоянию.

#### Возврат

Функции `set` не имеют возвращаемого значения.

#### Предостережения

-   Функция `set` **только обновляет переменную состояния для _следующего_ рендера**. Если вы прочитаете переменную состояния после вызова функции `set`, вы получите старое значение, которое было на экране до вашего вызова.

-   Если новое значение, которое вы предоставили, идентично текущему `состоянию`, что определяется сравнением [`Object.is`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/is), React **пропустит повторное отображение компонента и его дочерних элементов.** Это оптимизация. Хотя в некоторых случаях React может потребоваться вызвать ваш компонент, прежде чем пропустить дочерние компоненты, это не должно повлиять на ваш код.

-   React [batches state updates.](../learn/queueing-a-series-of-state-updates.md) Он обновляет экран **после того, как все обработчики событий запущены** и вызвали свои функции `set`. Это предотвращает множественные повторные рендеринги во время одного события. В редких случаях, когда вам нужно заставить React обновить экран раньше, например, для доступа к DOM, вы можете использовать [`flushSync`](flushSync.md).

-   Вызов функции `set` _во время рендеринга_ разрешен только внутри текущего рендерингового компонента. React отбросит его вывод и немедленно попытается отрендерить его снова с новым состоянием. Этот паттерн нужен редко, но вы можете использовать его для **сохранения информации из предыдущих рендеров**.

-   В строгом режиме React будет **вызывать вашу функцию обновления дважды**, чтобы помочь вам найти случайные примеси. Это поведение только для разработчиков и не влияет на производство. Если ваша функция обновления является чистой (как и должно быть), это не должно повлиять на поведение. Результат одного из вызовов будет проигнорирован.

## Использование

### Добавление состояния в компонент

Вызовите `useState` на верхнем уровне вашего компонента для объявления одной или более [переменных состояния.](../learn/state-a-components-memory.md)

<!-- 0007.part.md -->

```js
import { useState } from 'react';

function MyComponent() {
    const [age, setAge] = useState(42);
    const [name, setName] = useState('Taylor');
    // ...
}
```

<!-- 0008.part.md -->

Принято называть переменные состояния типа `[something, setSomething]`, используя [деструктуризацию массива](https://javascript.info/destructuring-assignment).

`useState` возвращает массив, содержащий ровно два элемента:

1.  текущее состояние этой переменной состояния, первоначально установленное в начальное состояние, которое вы указали.
2.  `set` функция, которая позволяет вам изменить ее на любое другое значение в ответ на взаимодействие.

Чтобы обновить то, что отображается на экране, вызовите функцию `set` с некоторым следующим состоянием:

<!-- 0009.part.md -->

```js
function handleClick() {
    setName('Robin');
}
```

<!-- 0010.part.md -->

React сохранит следующее состояние, снова отобразит ваш компонент с новыми значениями и обновит пользовательский интерфейс.

!!!warning ""

    Вызов функции `set` **не** изменяет текущее состояние в уже выполняющемся коде:

    ```js
    function handleClick() {
    	setName('Robin');
    	console.log(name); // Still "Taylor"!
    }
    ```

    Это влияет только на то, что `useState` будет возвращать, начиная со _следующего_ рендера.

### Базовые примеры useState

#### 1. Счетчик (число)

В этом примере переменная состояния `count` содержит число. Нажатие на кнопку увеличивает его.

<!-- 0013.part.md -->

```js
import { useState } from 'react';

export default function Counter() {
    const [count, setCount] = useState(0);

    function handleClick() {
        setCount(count + 1);
    }

    return (
        <button onClick={handleClick}>
            You pressed me {count} times
        </button>
    );
}
```

#### 2. Текстовое поле (строка)

В этом примере переменная состояния `text` содержит строку. Когда вы вводите текст, `handleChange` считывает последнее значение ввода из DOM-элемента ввода браузера и вызывает `setText` для обновления состояния. Это позволяет отобразить текущий `текст` ниже.

<!-- 0015.part.md -->

```js
import { useState } from 'react';

export default function MyInput() {
    const [text, setText] = useState('hello');

    function handleChange(e) {
        setText(e.target.value);
    }

    return (
        <>
            <input value={text} onChange={handleChange} />
            <p>You typed: {text}</p>
            <button onClick={() => setText('hello')}>
                Reset
            </button>
        </>
    );
}
```

#### 3. Checkbox (boolean)

В этом примере переменная состояния `liked` содержит булево значение. Когда вы нажимаете на вход, `setLiked` обновляет переменную состояния `liked`, чтобы узнать, установлен ли флажок в чекбоксе браузера. Переменная `liked` используется для вывода текста под чекбоксом.

<!-- 0017.part.md -->

```js
import { useState } from 'react';

export default function MyCheckbox() {
    const [liked, setLiked] = useState(true);

    function handleChange(e) {
        setLiked(e.target.checked);
    }

    return (
        <>
            <label>
                <input
                    type="checkbox"
                    checked={liked}
                    onChange={handleChange}
                />
                I liked this
            </label>
            <p>
                You {liked ? 'liked' : 'did not like'} this.
            </p>
        </>
    );
}
```

#### 4. Форма (две переменные)

Вы можете объявить более одной переменной состояния в одном компоненте. Каждая переменная состояния полностью независима.

<!-- 0019.part.md -->

```js
import { useState } from 'react';

export default function Form() {
    const [name, setName] = useState('Taylor');
    const [age, setAge] = useState(42);

    return (
        <>
            <input
                value={name}
                onChange={(e) => setName(e.target.value)}
            />
            <button onClick={() => setAge(age + 1)}>
                Increment age
            </button>
            <p>
                Hello, {name}. You are {age}.
            </p>
        </>
    );
}
```

### Обновление состояния на основе предыдущего состояния

Предположим, что `возраст` равен `42`. Этот обработчик вызывает `setAge(age + 1)` три раза:

<!-- 0023.part.md -->

```js
function handleClick() {
    setAge(age + 1); // setAge(42 + 1)
    setAge(age + 1); // setAge(42 + 1)
    setAge(age + 1); // setAge(42 + 1)
}
```

<!-- 0024.part.md -->

Однако после одного клика `age` будет только `43`, а не `45`! Это происходит потому, что вызов функции `set` [не обновляет](../learn/state-as-a-snapshot.md) переменную состояния `age` в уже запущенном коде. Поэтому каждый вызов `setAge(age + 1)` становится `setAge(43)`.

Чтобы решить эту проблему, **вы можете передать функцию _updater_** в `setAge` вместо следующего состояния:

<!-- 0025.part.md -->

```js
function handleClick() {
    setAge((a) => a + 1); // setAge(42 => 43)
    setAge((a) => a + 1); // setAge(43 => 44)
    setAge((a) => a + 1); // setAge(44 => 45)
}
```

<!-- 0026.part.md -->

Здесь `a => a + 1` - это ваша функция обновления. Она берет отложенное состояние и вычисляет следующее состояние из него.

React помещает ваши функции обновления в [очередь](../learn/queueing-a-series-of-state-updates.md) Затем, во время следующего рендеринга, он будет вызывать их в том же порядке:

1.  `a => a + 1` получит `42` в качестве ожидающего состояния и вернет `43` в качестве следующего состояния.
2.  `a => a + 1` получит `43` как ожидающее состояние и вернет `44` как следующее состояние.
3.  `a => a + 1` получит `44` как состояние ожидания и вернет `45` как следующее состояние.

Других обновлений в очереди нет, поэтому в конце React сохранит `45` как текущее состояние.

По традиции принято называть аргумент ожидающего состояния по первой букве имени переменной состояния, например `a` для `age`. Однако вы также можете назвать его `prevAge` или как-то иначе, что вам кажется более понятным.

React может вызывать ваши апдейтеры дважды в процессе разработки, чтобы убедиться, что они [чистые](../learn/keeping-components-pure.md).

!!!note "Всегда ли использование программы обновления предпочтительнее?"

    Вы можете услышать рекомендацию всегда писать код типа `setAge(a => a + 1)`, если состояние, которое вы устанавливаете, вычисляется из предыдущего состояния. В этом нет ничего плохого, но это также не всегда необходимо.

    В большинстве случаев между этими двумя подходами нет никакой разницы. React всегда гарантирует, что для намеренных действий пользователя, таких как щелчки, переменная состояния `age` будет обновлена перед следующим щелчком. Это означает, что нет риска того, что обработчик щелчка увидит "устаревший" `age` в начале обработчика события.

    Однако, если вы делаете несколько обновлений в рамках одного события, обновляющие устройства могут быть полезны. Они также полезны, если доступ к самой переменной состояния неудобен (вы можете столкнуться с этим при оптимизации повторных рендеров).

    Если вы предпочитаете последовательность, а не более многословный синтаксис, разумно всегда писать обновляющее устройство, если состояние, которое вы устанавливаете, вычисляется из предыдущего состояния. Если оно вычисляется из предыдущего состояния некоторой _другой_ переменной состояния, вы можете объединить их в один объект и [использовать редуктор](../learn/extracting-state-logic-into-a-reducer.md).

### Разница между передачей функции обновления и передачей следующего состояния напрямую

#### 1. Передача функции обновления

В этом примере передается функция обновления, поэтому кнопка "+3" работает.

<!-- 0027.part.md -->

```js
import { useState } from 'react';

export default function Counter() {
    const [age, setAge] = useState(42);

    function increment() {
        setAge((a) => a + 1);
    }

    return (
        <>
            <h1>Your age: {age}</h1>
            <button
                onClick={() => {
                    increment();
                    increment();
                    increment();
                }}
            >
                +3
            </button>
            <button
                onClick={() => {
                    increment();
                }}
            >
                +1
            </button>
        </>
    );
}
```

#### 2. Передача следующего состояния напрямую

В этом примере **не** передается функция обновления, поэтому кнопка "+3" **не работает так, как задумано**.

<!-- 0031.part.md -->

```js
import { useState } from 'react';

export default function Counter() {
    const [age, setAge] = useState(42);

    function increment() {
        setAge(age + 1);
    }

    return (
        <>
            <h1>Your age: {age}</h1>
            <button
                onClick={() => {
                    increment();
                    increment();
                    increment();
                }}
            >
                +3
            </button>
            <button
                onClick={() => {
                    increment();
                }}
            >
                +1
            </button>
        </>
    );
}
```

### Обновление объектов и массивов в состоянии

Вы можете помещать объекты и массивы в состояние. В React состояние считается доступным только для чтения, поэтому **вам следует _заменить_ его, а не _изменять_ существующие объекты**. Например, если у вас есть объект `form` в состоянии, не изменяйте его:

<!-- 0035.part.md -->

```js
// 🚩 Don't mutate an object in state like this:
form.firstName = 'Taylor';
```

<!-- 0036.part.md -->

Вместо этого замените весь объект, создав новый:

<!-- 0037.part.md -->

```js
// ✅ Replace state with a new object
setForm({
    ...form,
    firstName: 'Taylor',
});
```

<!-- 0038.part.md -->

Прочитайте [обновление объектов в состоянии](../learn/updating-objects-in-state.md) и [обновление массивов в состоянии](../learn/updating-arrays-in-state.md), чтобы узнать больше.

### Примеры объектов и массивов в штате

#### 1. Форма (объект)

В этом примере переменная состояния `form` содержит объект. Каждый вход имеет обработчик изменений, который вызывает `setForm` со следующим состоянием всей формы. Синтаксис распространения `{ ...form }` гарантирует, что объект состояния заменяется, а не изменяется.

<!-- 0039.part.md -->

```js
import { useState } from 'react';

export default function Form() {
    const [form, setForm] = useState({
        firstName: 'Barbara',
        lastName: 'Hepworth',
        email: 'bhepworth@sculpture.com',
    });

    return (
        <>
            <label>
                First name:
                <input
                    value={form.firstName}
                    onChange={(e) => {
                        setForm({
                            ...form,
                            firstName: e.target.value,
                        });
                    }}
                />
            </label>
            <label>
                Last name:
                <input
                    value={form.lastName}
                    onChange={(e) => {
                        setForm({
                            ...form,
                            lastName: e.target.value,
                        });
                    }}
                />
            </label>
            <label>
                Email:
                <input
                    value={form.email}
                    onChange={(e) => {
                        setForm({
                            ...form,
                            email: e.target.value,
                        });
                    }}
                />
            </label>
            <p>
                {form.firstName} {form.lastName} (
                {form.email})
            </p>
        </>
    );
}
```

#### 2. Форма (вложенный объект)

В этом примере состояние более вложенное. Когда вы обновляете вложенное состояние, вам необходимо создать копию объекта, который вы обновляете, а также любых объектов, "содержащих" его на пути вверх. Прочитайте [обновление вложенного объекта](../learn/updating-objects-in-state.md), чтобы узнать больше.

<!-- 0043.part.md -->

<div markdown style="max-height: 400px; overflow-y: auto;">

```js
import { useState } from 'react';

export default function Form() {
    const [person, setPerson] = useState({
        name: 'Niki de Saint Phalle',
        artwork: {
            title: 'Blue Nana',
            city: 'Hamburg',
            image: 'https://i.imgur.com/Sd1AgUOm.jpg',
        },
    });

    function handleNameChange(e) {
        setPerson({
            ...person,
            name: e.target.value,
        });
    }

    function handleTitleChange(e) {
        setPerson({
            ...person,
            artwork: {
                ...person.artwork,
                title: e.target.value,
            },
        });
    }

    function handleCityChange(e) {
        setPerson({
            ...person,
            artwork: {
                ...person.artwork,
                city: e.target.value,
            },
        });
    }

    function handleImageChange(e) {
        setPerson({
            ...person,
            artwork: {
                ...person.artwork,
                image: e.target.value,
            },
        });
    }

    return (
        <>
            <label>
                Name:
                <input
                    value={person.name}
                    onChange={handleNameChange}
                />
            </label>
            <label>
                Title:
                <input
                    value={person.artwork.title}
                    onChange={handleTitleChange}
                />
            </label>
            <label>
                City:
                <input
                    value={person.artwork.city}
                    onChange={handleCityChange}
                />
            </label>
            <label>
                Image:
                <input
                    value={person.artwork.image}
                    onChange={handleImageChange}
                />
            </label>
            <p>
                <i>{person.artwork.title}</i>
                {' by '}
                {person.name}
                <br />
                (located in {person.artwork.city})
            </p>
            <img
                src={person.artwork.image}
                alt={person.artwork.title}
            />
        </>
    );
}
```

</div>

#### 3. Список (массив)

В этом примере переменная состояния `todos` содержит массив. Каждый обработчик кнопки вызывает `setTodos` со следующей версией этого массива. Синтаксис распространения `[...todos]`, `todos.map()` и `todos.filter()` гарантируют, что массив состояний заменяется, а не изменяется.

=== "App.js"

    <div markdown style="max-height: 400px; overflow-y: auto;">

    ```js
    import { useState } from 'react';
    import AddTodo from './AddTodo.js';
    import TaskList from './TaskList.js';

    let nextId = 3;
    const initialTodos = [
    	{ id: 0, title: 'Buy milk', done: true },
    	{ id: 1, title: 'Eat tacos', done: false },
    	{ id: 2, title: 'Brew tea', done: false },
    ];

    export default function TaskApp() {
    	const [todos, setTodos] = useState(initialTodos);

    	function handleAddTodo(title) {
    		setTodos([
    			...todos,
    			{
    				id: nextId++,
    				title: title,
    				done: false,
    			},
    		]);
    	}

    	function handleChangeTodo(nextTodo) {
    		setTodos(
    			todos.map((t) => {
    				if (t.id === nextTodo.id) {
    					return nextTodo;
    				} else {
    					return t;
    				}
    			})
    		);
    	}

    	function handleDeleteTodo(todoId) {
    		setTodos(todos.filter((t) => t.id !== todoId));
    	}

    	return (
    		<>
    			<AddTodo onAddTodo={handleAddTodo} />
    			<TaskList
    				todos={todos}
    				onChangeTodo={handleChangeTodo}
    				onDeleteTodo={handleDeleteTodo}
    			/>
    		</>
    	);
    }
    ```

    </div>

=== "AddTodo.js"

    ```js
    import { useState } from 'react';

    export default function AddTodo({ onAddTodo }) {
    	const [title, setTitle] = useState('');
    	return (
    		<>
    			<input
    				placeholder="Add todo"
    				value={title}
    				onChange={(e) => setTitle(e.target.value)}
    			/>
    			<button
    				onClick={() => {
    					setTitle('');
    					onAddTodo(title);
    				}}
    			>
    				Add
    			</button>
    		</>
    	);
    }
    ```

=== "TaskList.js"

    <div markdown style="max-height: 400px; overflow-y: auto;">

    ```js
    import { useState } from 'react';

    export default function TaskList({
    	todos,
    	onChangeTodo,
    	onDeleteTodo,
    }) {
    	return (
    		<ul>
    			{todos.map((todo) => (
    				<li key={todo.id}>
    					<Task
    						todo={todo}
    						onChange={onChangeTodo}
    						onDelete={onDeleteTodo}
    					/>
    				</li>
    			))}
    		</ul>
    	);
    }

    function Task({ todo, onChange, onDelete }) {
    	const [isEditing, setIsEditing] = useState(false);
    	let todoContent;
    	if (isEditing) {
    		todoContent = (
    			<>
    				<input
    					value={todo.title}
    					onChange={(e) => {
    						onChange({
    							...todo,
    							title: e.target.value,
    						});
    					}}
    				/>
    				<button onClick={() => setIsEditing(false)}>
    					Save
    				</button>
    			</>
    		);
    	} else {
    		todoContent = (
    			<>
    				{todo.title}
    				<button onClick={() => setIsEditing(true)}>
    					Edit
    				</button>
    			</>
    		);
    	}
    	return (
    		<label>
    			<input
    				type="checkbox"
    				checked={todo.done}
    				onChange={(e) => {
    					onChange({
    						...todo,
    						done: e.target.checked,
    					});
    				}}
    			/>
    			{todoContent}
    			<button onClick={() => onDelete(todo.id)}>
    				Delete
    			</button>
    		</label>
    	);
    }
    ```

    </div>

#### 4. Написание краткой логики обновления с помощью Immer

Если обновление массивов и объектов без мутации кажется утомительным, вы можете использовать библиотеку типа [Immer](https://github.com/immerjs/use-immer) для сокращения повторяющегося кода. Immer позволяет вам писать лаконичный код, как если бы вы мутировали объекты, но под капотом она выполняет неизменяемые обновления:

<!-- 0055.part.md -->

```js
import { useState } from 'react';
import { useImmer } from 'use-immer';

let nextId = 3;
const initialList = [
    { id: 0, title: 'Big Bellies', seen: false },
    { id: 1, title: 'Lunar Landscape', seen: false },
    { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
    const [list, updateList] = useImmer(initialList);

    function handleToggle(artworkId, nextSeen) {
        updateList((draft) => {
            const artwork = draft.find(
                (a) => a.id === artworkId
            );
            artwork.seen = nextSeen;
        });
    }

    return (
        <>
            <h1>Art Bucket List</h1>
            <h2>My list of art to see:</h2>
            <ItemList
                artworks={list}
                onToggle={handleToggle}
            />
        </>
    );
}

function ItemList({ artworks, onToggle }) {
    return (
        <ul>
            {artworks.map((artwork) => (
                <li key={artwork.id}>
                    <label>
                        <input
                            type="checkbox"
                            checked={artwork.seen}
                            onChange={(e) => {
                                onToggle(
                                    artwork.id,
                                    e.target.checked
                                );
                            }}
                        />
                        {artwork.title}
                    </label>
                </li>
            ))}
        </ul>
    );
}
```

### Избегание воссоздания начального состояния

React сохраняет начальное состояние один раз и игнорирует его при последующих рендерах.

<!-- 0059.part.md -->

```js
function TodoList() {
    const [todos, setTodos] = useState(
        createInitialTodos()
    );
    // ...
}
```

<!-- 0060.part.md -->

Хотя результат `createInitialTodos()` используется только для начального рендеринга, вы все равно вызываете эту функцию при каждом рендеринге. Это может быть расточительно, если она создает большие массивы или выполняет дорогие вычисления.

Чтобы решить эту проблему, вы можете передать ее в качестве функции инициализатора в `useState` вместо этого:

<!-- 0061.part.md -->

```js
function TodoList() {
    const [todos, setTodos] = useState(createInitialTodos);
    // ...
}
```

<!-- 0062.part.md -->

Обратите внимание, что вы передаете `createInitialTodos`, которая является самой _функцией_, а не `createInitialTodos()`, которая является результатом ее вызова. Если вы передадите функцию в `useState`, React будет вызывать ее только во время инициализации.

React может вызвать ваши инициализаторы дважды в процессе разработки, чтобы убедиться, что они [чистые](../learn/keeping-components-pure.md).

### Разница между передачей инициализатора и передачей начального состояния напрямую

#### 1. Передача функции инициализатора

Этот пример передает функцию инициализатора, поэтому функция `createInitialTodos` выполняется только во время инициализации. Она не выполняется при повторном рендеринге компонента, например, когда вы вводите текст в поле ввода.

<!-- 0063.part.md -->

```js
import { useState } from 'react';

function createInitialTodos() {
    const initialTodos = [];
    for (let i = 0; i < 50; i++) {
        initialTodos.push({
            id: i,
            text: 'Item ' + (i + 1),
        });
    }
    return initialTodos;
}

export default function TodoList() {
    const [todos, setTodos] = useState(createInitialTodos);
    const [text, setText] = useState('');

    return (
        <>
            <input
                value={text}
                onChange={(e) => setText(e.target.value)}
            />
            <button
                onClick={() => {
                    setText('');
                    setTodos([
                        {
                            id: todos.length,
                            text: text,
                        },
                        ...todos,
                    ]);
                }}
            >
                Add
            </button>
            <ul>
                {todos.map((item) => (
                    <li key={item.id}>{item.text}</li>
                ))}
            </ul>
        </>
    );
}
```

#### 2. Передача начального состояния напрямую

В этом примере не передается функция инициализатора, поэтому функция `createInitialTodos` запускается при каждом рендере, например, когда вы вводите текст в input. Никакой заметной разницы в поведении нет, но этот код менее эффективен.

<!-- 0065.part.md -->

```js
import { useState } from 'react';

function createInitialTodos() {
    const initialTodos = [];
    for (let i = 0; i < 50; i++) {
        initialTodos.push({
            id: i,
            text: 'Item ' + (i + 1),
        });
    }
    return initialTodos;
}

export default function TodoList() {
    const [todos, setTodos] = useState(
        createInitialTodos()
    );
    const [text, setText] = useState('');

    return (
        <>
            <input
                value={text}
                onChange={(e) => setText(e.target.value)}
            />
            <button
                onClick={() => {
                    setText('');
                    setTodos([
                        {
                            id: todos.length,
                            text: text,
                        },
                        ...todos,
                    ]);
                }}
            >
                Add
            </button>
            <ul>
                {todos.map((item) => (
                    <li key={item.id}>{item.text}</li>
                ))}
            </ul>
        </>
    );
}
```

### Сброс состояния с помощью ключа

Вы часто сталкиваетесь с атрибутом `key` при [рендеринге списков](../learn/rendering-lists.md). Однако он также служит для другой цели.

Вы можете **сбросить состояние компонента, передав компоненту другой `key`.** В этом примере кнопка Reset изменяет переменную состояния `version`, которую мы передаем в качестве `key` в `Form`. Когда `ключ` меняется, React заново создает компонент `Form` (и все его дочерние элементы) с нуля, поэтому его состояние сбрасывается.

Прочитайте [сохранение и сброс состояния](../learn/preserving-and-resetting-state.md), чтобы узнать больше.

<!-- 0067.part.md -->

```js
import { useState } from 'react';

export default function App() {
    const [version, setVersion] = useState(0);

    function handleReset() {
        setVersion(version + 1);
    }

    return (
        <>
            <button onClick={handleReset}>Reset</button>
            <Form key={version} />
        </>
    );
}

function Form() {
    const [name, setName] = useState('Taylor');

    return (
        <>
            <input
                value={name}
                onChange={(e) => setName(e.target.value)}
            />
            <p>Hello, {name}.</p>
        </>
    );
}
```

### Хранение информации из предыдущих рендеров

Обычно вы обновляете состояние в обработчиках событий. Однако в редких случаях вы можете захотеть изменить состояние в ответ на рендеринг - например, вы можете захотеть изменить переменную состояния при изменении реквизита.

В большинстве случаев это не нужно:

-   **Если нужное вам значение может быть вычислено полностью из текущего реквизита или другого состояния, [удалите лишнее состояние вообще](../learn/choosing-the-state-structure.md)** Если вас беспокоит слишком частое повторное вычисление, вам поможет хук [`useMemo`](useMemo.md).
-   Если вы хотите сбросить состояние всего дерева компонентов, передайте своему компоненту другой `ключ`.
-   Если можете, обновите все соответствующие состояния в обработчиках событий.

В редких случаях, когда ни один из этих способов не применим, существует шаблон, который можно использовать для обновления состояния на основе значений, которые были отображены на данный момент, путем вызова функции `set` во время рендеринга компонента.

Вот пример. Этот компонент `CountLabel` отображает переданный ему параметр `count`:

<!-- 0071.part.md -->

```js
export default function CountLabel({ count }) {
    return <h1>{count}</h1>;
}
```

<!-- 0072.part.md -->

Допустим, вы хотите показать, увеличился или уменьшился счетчик с момента последнего изменения. Реквизит `count` не говорит вам об этом - вам нужно отслеживать его предыдущее значение. Добавьте переменную состояния `prevCount`, чтобы отслеживать его. Добавьте еще одну переменную состояния `trend`, чтобы отслеживать, увеличился или уменьшился счетчик. Сравните `prevCount` с `count`, и если они не равны, обновите и `prevCount`, и `trend`. Теперь вы можете показать как текущее значение счетчика, так и _как оно изменилось с момента последнего рендера_.

=== "App.js"

    ```js
    import { useState } from 'react';
    import CountLabel from './CountLabel.js';

    export default function App() {
    	const [count, setCount] = useState(0);
    	return (
    		<>
    			<button onClick={() => setCount(count + 1)}>
    				Increment
    			</button>
    			<button onClick={() => setCount(count - 1)}>
    				Decrement
    			</button>
    			<CountLabel count={count} />
    		</>
    	);
    }
    ```

=== "CountLabel.js"

    ```js
    import { useState } from 'react';

    export default function CountLabel({ count }) {
    	const [prevCount, setPrevCount] = useState(count);
    	const [trend, setTrend] = useState(null);
    	if (prevCount !== count) {
    		setPrevCount(count);
    		setTrend(
    			count > prevCount ? 'increasing' : 'decreasing'
    		);
    	}
    	return (
    		<>
    			<h1>{count}</h1>
    			{trend && <p>The count is {trend}</p>}
    		</>
    	);
    }
    ```

Обратите внимание, что если вы вызываете функцию `set` во время рендеринга, она должна быть внутри условия типа `prevCount !== count`, а внутри условия должен быть вызов типа `setPrevCount(count)`. В противном случае ваш компонент будет перерисовываться в цикле до тех пор, пока не произойдет сбой. Кроме того, вы можете обновить состояние _текущего рендеринга_ компонента только таким образом. Вызов функции `set` другого компонента во время рендеринга является ошибкой. Наконец, ваш вызов `set` должен обновлять состояние без мутации - это не означает, что вы можете нарушать другие правила [чистых функций](../learn/keeping-components-pure.md).

Этот паттерн может быть трудным для понимания, и обычно его лучше избегать. Однако это лучше, чем обновлять состояние в эффекте. Когда вы вызываете функцию `set` во время рендеринга, React перерендерит этот компонент сразу после того, как ваш компонент выйдет с оператором `return`, и до рендеринга дочерних компонентов. Таким образом, дочерние компоненты не нужно рендерить дважды. Оставшаяся часть функции вашего компонента будет по-прежнему выполняться (а результат будет отброшен). Если ваше условие находится ниже всех вызовов Hook, вы можете добавить ранний `return;`, чтобы перезапустить рендеринг раньше.

## Устранение неполадок

### Я обновил состояние, но логирование выдает мне старое значение

Вызов функции `set` **не изменяет состояние в работающем коде**:

<!-- 0079.part.md -->

```js
function handleClick() {
    console.log(count); // 0

    setCount(count + 1); // Request a re-render with 1
    console.log(count); // Still 0!

    setTimeout(() => {
        console.log(count); // Also 0!
    }, 5000);
}
```

<!-- 0080.part.md -->

Это происходит потому, что [states ведет себя как моментальный снимок] (/learn/state-as-a-snapshot). Обновление состояния запрашивает другой рендер с новым значением состояния, но не влияет на переменную JavaScript `count` в уже запущенном обработчике событий.

Если вам нужно использовать следующее состояние, вы можете сохранить его в переменной перед передачей в функцию `set`:

<!-- 0081.part.md -->

```js
const nextCount = count + 1;
setCount(nextCount);

console.log(count); // 0
console.log(nextCount); // 1
```

### Я обновил состояние, но экран не обновляется

React будет **игнорировать ваше обновление, если следующее состояние равно предыдущему,** что определяется сравнением [`Object.is`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/is). Обычно это происходит, когда вы изменяете объект или массив в состоянии напрямую:

<!-- 0083.part.md -->

```js
obj.x = 10; // 🚩 Wrong: mutating existing object
setObj(obj); // 🚩 Doesn't do anything
```

<!-- 0084.part.md -->

Вы изменили существующий объект `obj` и передали его обратно в `setObj`, поэтому React проигнорировал обновление. Чтобы исправить это, вам нужно убедиться, что вы всегда _заменяете_ объекты и массивы в состоянии вместо того, чтобы _мутировать_ их:

<!-- 0085.part.md -->

```js
// ✅ Correct: creating a new object
setObj({
    ...obj,
    x: 10,
});
```

<!-- 0086.part.md -->

### Я получаю ошибку: "Слишком много повторных рендеров"

Вы можете получить ошибку, которая гласит: `Слишком много рендеров. React ограничивает количество рендеров для предотвращения бесконечного цикла.` Обычно это означает, что вы безоговорочно устанавливаете состояние _во время рендера_, поэтому ваш компонент попадает в цикл: рендер, установка состояния (что вызывает рендер), рендер, установка состояния (что вызывает рендер) и так далее. Очень часто причиной этого является ошибка в определении обработчика событий:

<!-- 0087.part.md -->

```js
// 🚩 Wrong: calls the handler during render
return <button onClick={handleClick()}>Click me</button>;

// ✅ Correct: passes down the event handler
return <button onClick={handleClick}>Click me</button>;

// ✅ Correct: passes down an inline function
return (
    <button onClick={(e) => handleClick(e)}>
        Click me
    </button>
);
```

<!-- 0088.part.md -->

Если вы не можете найти причину этой ошибки, нажмите на стрелку рядом с ошибкой в консоли и просмотрите стек JavaScript, чтобы найти конкретный вызов функции `set`, ответственный за ошибку.

### Моя функция инициализатора или обновления выполняется дважды

В [Строгом режиме](StrictMode.md) React будет вызывать некоторые из ваших функций дважды вместо одного раза:

<!-- 0089.part.md -->

```js
function TodoList() {
    // This component function will run twice for every render.

    const [todos, setTodos] = useState(() => {
        // This initializer function will run twice during initialization.
        return createTodos();
    });

    function handleClick() {
        setTodos((prevTodos) => {
            // This updater function will run twice for every click.
            return [...prevTodos, createTodo()];
        });
    }
    // ...
}
```

<!-- 0090.part.md -->

Это ожидаемо и не должно нарушать ваш код.

Это **поведение только для разработчиков** помогает вам [поддерживать чистоту компонентов](../learn/keeping-components-pure.md). React использует результат одного из вызовов и игнорирует результат другого вызова. Пока ваши функции компонентов, инициализаторов и обновлений чисты, это не должно влиять на вашу логику. Однако если они случайно оказались нечистыми, это поможет вам заметить ошибки.

Например, эта нечистая функция обновления мутирует массив в состоянии:

<!-- 0091.part.md -->

```js
setTodos((prevTodos) => {
    // 🚩 Mistake: mutating state
    prevTodos.push(createTodo());
});
```

<!-- 0092.part.md -->

Поскольку React дважды вызывает вашу функцию обновления, вы увидите, что todo было добавлено дважды, поэтому вы будете знать, что произошла ошибка. В этом примере вы можете исправить ошибку, заменив массив вместо его мутации:

<!-- 0093.part.md -->

```js
setTodos((prevTodos) => {
    // ✅ Correct: replacing with new state
    return [...prevTodos, createTodo()];
});
```

<!-- 0094.part.md -->

Теперь, когда эта функция обновления является чистой, ее повторный вызов не изменит поведение. Вот почему двойной вызов React помогает найти ошибки. **Только функции компонентов, инициализаторов и обновлений должны быть чистыми.** Обработчики событий не должны быть чистыми, поэтому React никогда не будет вызывать ваши обработчики событий дважды.

Для получения дополнительной информации читайте [keeping components pure](../learn/keeping-components-pure.md).

---

### Я пытаюсь установить состояние на функцию, но вместо этого вызывается она

Вы не можете поместить функцию в состояние таким образом:

<!-- 0095.part.md -->

```js
const [fn, setFn] = useState(someFunction);

function handleClick() {
    setFn(someOtherFunction);
}
```

<!-- 0096.part.md -->

Поскольку вы передаете функцию, React предполагает, что `someFunction` является инициализирующей функцией, а `someOtherFunction` является обновляющей функцией, поэтому он пытается вызвать их и сохранить результат. Чтобы действительно _сохранить_ функцию, вы должны поставить `() =>` перед ними в обоих случаях. Тогда React будет хранить переданные вами функции.

<!-- 0097.part.md -->

```js
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
    setFn(() => someOtherFunction);
}
```

## Ссылки

-   [https://react.dev/reference/react/useState](https://react.dev/reference/react/useState)

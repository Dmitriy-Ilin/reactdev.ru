---
description: createContext позволяет вам создать контекст, который компоненты могут предоставить или прочитать
---

# createContext

<big>`createContext` позволяет вам создать [контекст](../../learn/passing-data-deeply-with-context.md), который компоненты могут предоставить или прочитать.</big>

```js
const SomeContext = createContext(defaultValue);
```

## Описание {#reference}

### `createContext(defaultValue)` {#createcontext}

Вызовите `createContext` вне каких-либо компонентов для создания контекста.

```js
import { createContext } from 'react';

const ThemeContext = createContext('light');
```

#### Параметры {#parameters}

-   `defaultValue`: Значение, которое вы хотите, чтобы контекст имел, если в дереве над компонентом, читающим контекст, нет подходящего поставщика контекста. Если у вас нет никакого значимого значения по умолчанию, укажите `null`. Значение по умолчанию используется в качестве запасного варианта "на крайний случай". Оно статично и никогда не изменяется с течением времени.

#### Возвращает {#returns}

`createContext` возвращает объект контекста.

**Сам объект контекста не содержит никакой информации.** Он представляет _какой_ контекст читают или предоставляют другие компоненты. Обычно вы используете `SomeContext.Provider` в компонентах выше, чтобы указать значение контекста, и вызываете [`useContext(SomeContext)`](useContext.md) в компонентах ниже, чтобы прочитать его. Объект контекста имеет несколько свойств:

-   `SomeContext.Provider` позволяет вам предоставлять значение контекста компонентам.
-   `SomeContext.Consumer` является альтернативным и редко используемым способом чтения значения контекста.

### `SomeContext.Provider` {#provider}

Оберните ваши компоненты в провайдер контекста, чтобы указать значение этого контекста для всех компонентов внутри:

```js
function App() {
    const [theme, setTheme] = useState('light');
    // ...
    return (
        <ThemeContext.Provider value={theme}>
            <Page />
        </ThemeContext.Provider>
    );
}
```

#### Параметры {#provider-props}

-   `value`: Значение, которое вы хотите передать всем компонентам, читающим данный контекст внутри данного провайдера, независимо от его глубины. Значение контекста может быть любого типа. Компонент, вызывающий [`useContext(SomeContext)`](useContext.md) внутри провайдера, получает `value` самого внутреннего соответствующего провайдера контекста над ним.

### `SomeContext.Consumer` {#consumer}

До появления `useContext` существовал более старый способ чтения контекста:

```js
function Button() {
    // 🟡 Legacy way (not recommended)
    return (
        <ThemeContext.Consumer>
            {(theme) => <button className={theme} />}
        </ThemeContext.Consumer>
    );
}
```

Хотя этот старый способ все еще работает, но **новый написанный код должен читать контекст с помощью [`useContext()`](useContext.md) вместо этого:**

```js
function Button() {
    // ✅ Recommended way
    const theme = useContext(ThemeContext);
    return <button className={theme} />;
}
```

#### Параметры {#consumer-props}

-   `children`: Функция. React будет вызывать переданную вами функцию с текущим значением контекста, определяемым по тому же алгоритму, что и [`useContext()`](useContext.md), и отображать результат, возвращаемый этой функцией. React также будет повторно запускать эту функцию и обновлять пользовательский интерфейс при каждом изменении контекста из родительских компонентов.

## Использование {#usage}

### Создание контекста {#creating-context}

Контекст позволяет компонентам [передавать информацию вглубь](../../learn/passing-data-deeply-with-context.md) без явной передачи пропсов.

Вызовите `createContext` вне любых компонентов для создания одного или нескольких контекстов.

```js
import { createContext } from 'react';

const ThemeContext = createContext('light');
const AuthContext = createContext(null);
```

`createContext` возвращает объект контекста. Компоненты могут читать контекст, передавая его в [`useContext()`](useContext.md):

```js
function Button() {
    const theme = useContext(ThemeContext);
    // ...
}

function Profile() {
    const currentUser = useContext(AuthContext);
    // ...
}
```

По умолчанию, значения, которые они получают, будут значениями по умолчанию, которые вы указали при создании контекстов. Однако, само по себе это не полезно, потому что значения по умолчанию никогда не меняются.

Контекст полезен, потому что вы можете **предоставить другие, динамические значения из ваших компонентов:**.

```js hl_lines="10-11 13-14"
function App() {
    const [theme, setTheme] = useState('dark');
    const [currentUser, setCurrentUser] = useState({
        name: 'Taylor',
    });

    // ...

    return (
        <ThemeContext.Provider value={theme}>
            <AuthContext.Provider value={currentUser}>
                <Page />
            </AuthContext.Provider>
        </ThemeContext.Provider>
    );
}
```

Теперь компонент `Page` и все компоненты внутри него, независимо от глубины, будут "видеть" переданные значения контекста. Если переданные значения контекста изменяются, React будет перерисовывать компоненты, читающие контекст.

Подробнее о [чтении и предоставлении контекста и примеры](useContext.md)

### Импорт и экспорт контекста из файла {#importing-and-exporting-context-from-a-file}

Часто компонентам в разных файлах требуется доступ к одному и тому же контексту. Поэтому обычно контексты объявляются в отдельном файле. Затем вы можете использовать оператор [`export`](https://developer.mozilla.org/docs/web/javascript/reference/statements/export), чтобы сделать контекст доступным для других файлов:

```js hl_lines="4-5"
// Contexts.js
import { createContext } from 'react';

export const ThemeContext = createContext('light');
export const AuthContext = createContext(null);
```

Компоненты, объявленные в других файлах, могут использовать оператор [`импорт`](https://developer.mozilla.org/docs/web/javascript/reference/statements/import) для чтения или предоставления этого контекста:

```js hl_lines="2"
// Button.js
import { ThemeContext } from './Contexts.js';

function Button() {
    const theme = useContext(ThemeContext);
    // ...
}
```

---

```js hl_lines="2"
// App.js
import { ThemeContext, AuthContext } from './Contexts.js';

function App() {
    // ...
    return (
        <ThemeContext.Provider value={theme}>
            <AuthContext.Provider value={currentUser}>
                <Page />
            </AuthContext.Provider>
        </ThemeContext.Provider>
    );
}
```

Это работает аналогично [импорту и экспорту компонентов](../../learn/importing-and-exporting-components.md).

## Устранение неполадок {#troubleshooting}

### Я не могу найти способ изменить значение контекста {#i-cant-find-a-way-to-change-the-context-value}

Код, подобный этому, определяет значение контекста _по умолчанию_:

```js
const ThemeContext = createContext('light');
```

Это значение никогда не меняется. React использует это значение только в качестве запасного варианта, если не может найти подходящего провайдера выше.

Чтобы контекст менялся со временем, [добавьте состояние и оберните компоненты в провайдера контекста](useContext.md).

<small>:material-information-outline: Источник &mdash; [https://react.dev/reference/react/createContext](https://react.dev/reference/react/createContext)</small>

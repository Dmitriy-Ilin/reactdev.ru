# createContext

`createContext` позволяет вам создать [контекст](../learn/passing-data-deeply-with-context.md), который компоненты могут предоставить или прочитать.

<!-- 0001.part.md -->

```js
const SomeContext = createContext(defaultValue);
```

<!-- 0002.part.md -->

## Описание

### `createContext(defaultValue)`

Вызовите `createContext` вне каких-либо компонентов для создания контекста.

<!-- 0003.part.md -->

```js
import { createContext } from 'react';

const ThemeContext = createContext('light');
```

#### Параметры

-   `defaultValue`: Значение, которое вы хотите, чтобы контекст имел, если в дереве над компонентом, читающим контекст, нет подходящего поставщика контекста. Если у вас нет никакого значимого значения по умолчанию, укажите `null`. Значение по умолчанию используется в качестве запасного варианта "на крайний случай". Оно статично и никогда не изменяется с течением времени.

#### Возвращает

`createContext` возвращает объект контекста.

**Сам объект контекста не содержит никакой информации.** Он представляет _какой_ контекст читают или предоставляют другие компоненты. Обычно вы используете `SomeContext.Provider` в компонентах выше, чтобы указать значение контекста, и вызываете [`useContext(SomeContext)`](useContext.md) в компонентах ниже, чтобы прочитать его. Объект контекста имеет несколько свойств:

-   `SomeContext.Provider` позволяет вам предоставлять значение контекста компонентам.
-   `SomeContext.Consumer` является альтернативным и редко используемым способом чтения значения контекста.

### `SomeContext.Provider`

Оберните ваши компоненты в провайдер контекста, чтобы указать значение этого контекста для всех компонентов внутри:

<!-- 0005.part.md -->

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

<!-- 0006.part.md -->

#### Реквизиты

-   `value`: Значение, которое вы хотите передать всем компонентам, читающим данный контекст внутри данного провайдера, независимо от его глубины. Значение контекста может быть любого типа. Компонент, вызывающий [`useContext(SomeContext)`](useContext.md) внутри провайдера, получает `значение` самого внутреннего соответствующего провайдера контекста над ним.

### `SomeContext.Consumer`

До появления `useContext` существовал более старый способ чтения контекста:

<!-- 0007.part.md -->

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

<!-- 0008.part.md -->

Хотя этот старый способ все еще работает, но **новый написанный код должен читать контекст с помощью [`useContext()`](useContext.md) вместо этого:**.

<!-- 0009.part.md -->

```js
function Button() {
    // ✅ Recommended way
    const theme = useContext(ThemeContext);
    return <button className={theme} />;
}
```

<!-- 0010.part.md -->

#### Реквизиты

-   `children`: Функция. React будет вызывать переданную вами функцию с текущим значением контекста, определяемым по тому же алгоритму, что и [`useContext()`](useContext.md), и отображать результат, возвращаемый этой функцией. React также будет повторно запускать эту функцию и обновлять пользовательский интерфейс при каждом изменении контекста из родительских компонентов.

## Использование

### Создание контекста

Контекст позволяет компонентам [передавать информацию вглубь](../learn/passing-data-deeply-with-context.md) без явной передачи реквизитов.

Вызовите `createContext` вне любых компонентов для создания одного или нескольких контекстов.

<!-- 0011.part.md -->

```js
import { createContext } from 'react';

const ThemeContext = createContext('light');
const AuthContext = createContext(null);
```

<!-- 0012.part.md -->

`createContext` возвращает объект контекста. Компоненты могут читать контекст, передавая его в [`useContext()`](useContext.md):

<!-- 0013.part.md -->

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

<!-- 0014.part.md -->

По умолчанию, значения, которые они получают, будут значениями по умолчанию, которые вы указали при создании контекстов. Однако, само по себе это не полезно, потому что значения по умолчанию никогда не меняются.

Контекст полезен, потому что вы можете **предоставить другие, динамические значения из ваших компонентов:**.

<!-- 0015.part.md -->

```js
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

<!-- 0016.part.md -->

Теперь компонент `Page` и все компоненты внутри него, независимо от глубины, будут "видеть" переданные значения контекста. Если переданные значения контекста изменяются, React будет перерисовывать компоненты, читающие контекст.

[Подробнее о чтении и предоставлении контекста и примеры](useContext.md)

### Импорт и экспорт контекста из файла

Часто компонентам в разных файлах требуется доступ к одному и тому же контексту. Поэтому обычно контексты объявляются в отдельном файле. Затем вы можете использовать оператор [`export`](https://developer.mozilla.org/docs/web/javascript/reference/statements/export), чтобы сделать контекст доступным для других файлов:

<!-- 0017.part.md -->

```js
// Contexts.js
import { createContext } from 'react';

export const ThemeContext = createContext('light');
export const AuthContext = createContext(null);
```

<!-- 0018.part.md -->

Компоненты, объявленные в других файлах, могут использовать оператор [`импорт`](https://developer.mozilla.org/docs/web/javascript/reference/statements/import) для чтения или предоставления этого контекста:

<!-- 0019.part.md -->

```js
// Button.js
import { ThemeContext } from './Contexts.js';

function Button() {
    const theme = useContext(ThemeContext);
    // ...
}
```

<!-- 0020.part.md -->

<!-- 0021.part.md -->

```js
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

<!-- 0022.part.md -->

Это работает аналогично [импорту и экспорту компонентов](../learn/importing-and-exporting-components.md).

## Устранение неполадок

### Я не могу найти способ изменить значение контекста

Код, подобный этому, определяет значение контекста _по умолчанию_:

<!-- 0023.part.md -->

```js
const ThemeContext = createContext('light');
```

<!-- 0024.part.md -->

Это значение никогда не меняется. React использует это значение только в качестве запасного варианта, если не может найти подходящего провайдера выше.

Чтобы контекст менялся со временем, [добавьте состояние и оберните компоненты в провайдера контекста](useContext.md).

<!-- 0025.part.md -->

## Ссылки

-   [https://react.dev/reference/react/createContext](https://react.dev/reference/react/createContext)

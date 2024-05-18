---
description: Чистые функции выполняют только вычисления и ничего больше. Это облегчает понимание и отладку кода, а также позволяет React автоматически оптимизировать компоненты и хуки
---

# Компоненты и хуки должны быть чистыми

<big>
Чистые функции выполняют только вычисления и ничего больше. Это облегчает понимание и отладку кода, а также позволяет React автоматически оптимизировать компоненты и хуки.
</big>

!!!note "Чистота компонентов"

    Эта справочная страница охватывает расширенные темы и требует знакомства с концепциями, рассмотренными на странице [«Чистота компонентов»](../../learn/keeping-components-pure.md).

## Почему чистота имеет значение? {#why-does-purity-matter}

Одна из ключевых концепций, которая делает React _React_, - это _чистота_. Чистый компонент или хук - это тот, который:

-   **Идемпотентен** - Вы [всегда получаете один и тот же результат каждый раз](../../learn/keeping-components-pure.md#purity-components-as-formulas), когда запускаете его с теми же входными данными - props, state, context для компонентных входов; и аргументы для входов хука.
-   **Не имеет побочных эффектов в рендере** - Код с побочными эффектами должен выполняться [**отдельно от рендеринга**](#how-does-react-run-your-code). Например, как [обработчик событий](../../learn/responding-to-events.md) - когда пользователь взаимодействует с пользовательским интерфейсом и заставляет его обновляться; или как [эффект](../react/useEffect.md) - который запускается после рендера.
-   **Не мутирует нелокальные значения**: Компоненты и хуки должны [никогда не изменять значения, которые не создаются локально](#mutation) при рендере.

Когда рендер остается чистым, React может понять, как расставить приоритеты, какие обновления наиболее важны для пользователя, чтобы увидеть их первыми. Это становится возможным благодаря чистоте рендеринга: поскольку компоненты не имеют побочных эффектов [в рендеринге](#how-does-react-run-your-code), React может приостановить рендеринг компонентов, которые не так важно обновлять, и вернуться к ним позже, когда это будет необходимо.

Конкретно это означает, что логика рендеринга может быть запущена несколько раз, что позволяет React обеспечить пользователю приятный пользовательский опыт. Однако если у вашего компонента есть неотслеживаемый побочный эффект - например, изменение значения глобальной переменной [во время рендеринга](#how-does-react-run-your-code) - когда React снова запустит ваш код рендеринга, ваши побочные эффекты будут вызваны не так, как вы хотели. Это часто приводит к неожиданным ошибкам, которые могут ухудшить восприятие приложения пользователями. Вы можете увидеть [пример этого на странице Keeping Components Pure](../../learn/keeping-components-pure.md#side-effects-unintended-consequences).

### Как React выполняет ваш код? {#how-does-react-run-your-code}

React является декларативным: вы указываете React, что именно нужно отобразить, а React сам решает, как лучше показать это пользователю. Для этого у React есть несколько этапов, на которых он выполняет ваш код. Вам не нужно знать обо всех этих фазах, чтобы хорошо использовать React. Но на высоком уровне вы должны знать, какой код выполняется в _render_, а какой - за его пределами.

Под _рендерингом_ понимается вычисление того, как должна выглядеть следующая версия вашего пользовательского интерфейса. После рендеринга [эффекты](../react/useEffect.md) _промываются_ (то есть выполняются до тех пор, пока их не останется) и могут обновить расчеты, если эффекты влияют на макет. React берет этот новый расчет и сравнивает его с расчетом, использованным для создания предыдущей версии вашего пользовательского интерфейса, а затем _фиксирует_ только минимальные изменения, необходимые для [DOM](https://developer.mozilla.org/docs/Web/API/Document_Object_Model) (то, что пользователь видит на самом деле), чтобы привести его в соответствие с последней версией.

!!!note "Как определить, выполняется ли код при рендеринге"

    Одна из быстрых эвристик, позволяющих определить, выполняется ли код во время рендеринга, - это посмотреть, где он находится: если он написан на верхнем уровне, как в примере ниже, то велика вероятность, что он выполняется во время рендеринга.

    ```js hl_lines="2"
    function Dropdown() {
    	const selectedItems = new Set(); // created during render
    	// ...
    }
    ```

    Обработчики событий и эффекты не запускаются при рендеринге:

    ```js hl_lines="4-5"
    function Dropdown() {
    	const selectedItems = new Set();
    	const onSelect = (item) => {
    		// this code is in an event handler,
    		// so it's only run when the user triggers this
    		selectedItems.add(item);
    	};
    }
    ```

    ---

    ```js hl_lines="4"
    function Dropdown() {
    	const selectedItems = new Set();
    	useEffect(() => {
    		// this code is inside of an Effect, so it only runs after rendering
    		logForAnalytics(selectedItems);
    	}, [selectedItems]);
    }
    ```

## Компоненты и хуки должны быть идемпотентными {#components-and-hooks-must-be-idempotent}

Компоненты должны всегда возвращать один и тот же результат по отношению к своим входным данным - реквизитам, состоянию и контексту. Это известно как _идемпотентность_. [Идемпотентность](https://ru.wikipedia.org/wiki/Идемпотентность) - термин, получивший распространение в функциональном программировании. Он относится к идее, что вы [всегда получаете один и тот же результат каждый раз](../../learn/keeping-components-pure.md), когда запускаете этот кусок кода с одними и теми же входными данными.

Это означает, что _весь_ код, который выполняется [во время рендеринга](#how-does-react-run-your-code), должен быть идемпотентным, чтобы это правило выполнялось. Например, эта строка кода не является идемпотентной (и, следовательно, компонент тоже):

```js hl_lines="2"
function Clock() {
    const time = new Date(); // 🔴 Bad: always returns a different result!
    return <span>{time.toLocaleString()}</span>;
}
```

Функция `new Date()` не является идемпотентной, поскольку всегда возвращает текущую дату и меняет свой результат при каждом вызове. При рендеринге вышеуказанного компонента время, отображаемое на экране, будет зациклено на том времени, когда компонент был рендерирован. Аналогично, функции вроде `Math.random()` также не являются идемпотентными, поскольку при каждом вызове они возвращают разные результаты, даже если входные данные одинаковы.

Это не означает, что вы не должны использовать неидемпотентные функции вроде `new Date()` вообще - вы просто должны избегать их использования [во время рендеринга](#how-does-react-run-your-code). В данном случае мы можем _синхронизировать_ последнюю дату для этого компонента с помощью [Effect](../react/useEffect.md):

=== "App.js"

    ```js
    import { useState, useEffect } from 'react';

    function useTime() {
    	// 1. Keep track of the current date's state. `useState`
    	//    receives an initializer function as its
    	//    initial state. It only runs once when the hook is called,
    	//    so only the current date at the
    	//    time the hook is called is set first.
    	const [time, setTime] = useState(() => new Date());

    	useEffect(() => {
    		// 2. Update the current date every second using `setInterval`.
    		const id = setInterval(() => {
    			setTime(new Date()); // ✅ Good: non-idempotent code
    			// no longer runs in render
    		}, 1000);
    		// 3. Return a cleanup function so we don't leak the
    		// `setInterval` timer.
    		return () => clearInterval(id);
    	}, []);

    	return time;
    }

    export default function Clock() {
    	const time = useTime();
    	return <span>{time.toLocaleString()}</span>;
    }
    ```

=== "CodeSandbox"

    <iframe src="https://codesandbox.io/embed/wp6f4j?view=Editor+%2B+Preview&module=%2Fsrc%2FApp.js" style="width:100%; height: 500px; border:0; border-radius: 4px; overflow:hidden;" title="react.dev" allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking" sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"></iframe>

Обернув неэдемпотентный вызов `new Date()` в Effect, он переносит это вычисление [за пределы рендеринга](#how-does-react-run-your-code).

Если вам не нужно синхронизировать внешнее состояние с React, вы также можете рассмотреть возможность использования [обработчика событий](../../learn/responding-to-events.md), если оно должно обновляться только в ответ на взаимодействие с пользователем.

## Побочные эффекты должны выполняться вне рендера {#side-effects-must-run-outside-of-render}

[Побочные эффекты](../../learn/keeping-components-pure.md#side-effects-unintended-consequences) не должны запускаться [в рендере](#how-does-react-run-your-code), так как React может рендерить компоненты несколько раз, чтобы создать наилучший пользовательский опыт.

!!!note "Побочные эффекты"

    Побочные эффекты - это более широкое понятие, чем эффекты. Эффекты относятся к коду, обернутому в `useEffect`, в то время как побочный эффект - это общий термин для кода, который имеет любой наблюдаемый эффект, кроме его основного результата - возврата значения вызывающей стороне.

    Побочные эффекты обычно пишутся внутри [обработчиков событий](../../learn/responding-to-events.md) или Эффектов. Но никогда - во время рендеринга.

Хотя рендеринг должен быть чистым, побочные эффекты необходимы, чтобы в какой-то момент ваше приложение сделало что-нибудь интересное, например показало что-то на экране! Ключевой момент этого правила заключается в том, что побочные эффекты не должны выполняться [в рендере](#how-does-react-run-your-code), так как React может рендерить компоненты несколько раз. В большинстве случаев для обработки побочных эффектов вы будете использовать [обработчики событий](../../learn/responding-to-events.md). Использование обработчика событий явно говорит React, что этот код не нужно запускать во время рендеринга, сохраняя чистоту рендеринга. Если вы исчерпали все варианты - и только в крайнем случае - вы также можете обрабатывать побочные эффекты с помощью `useEffect`.

### Когда можно использовать мутацию? {#mutation}

#### Локальная мутация {#local-mutation}

Одним из распространенных примеров побочного эффекта является мутация, которая в JavaScript означает изменение значения не [примитивного](https://developer.mozilla.org/docs/Glossary/Primitive) значения. В целом, хотя мутация не является идиоматической в React, _локальная_ мутация абсолютно нормальна:

```js hl_lines="2 7"
function FriendList({ friends }) {
    const items = []; // ✅ Good: locally created
    for (let i = 0; i < friends.length; i++) {
        const friend = friends[i];
        items.push(
            <Friend key={friend.id} friend={friend} />
        ); // ✅ Good: local mutation is okay
    }
    return <section>{items}</section>;
}
```

Нет необходимости искажать код, чтобы избежать локальной мутации. Для краткости здесь можно было бы использовать и [`Array.map`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/map), но нет ничего плохого в том, чтобы создать локальный массив и затем заталкивать в него элементы [во время рендеринга](#how-does-react-run-your-code).

Несмотря на то, что выглядит так, будто мы мутируем `items`, важно отметить, что этот код делает это только _локально_ - мутация не «запоминается» при повторном рендеринге компонента. Другими словами, `items` будет существовать только до тех пор, пока существует компонент. Поскольку `items` всегда _восстанавливается_ при каждом рендеринге `<FriendList />`, компонент всегда будет возвращать один и тот же результат.

С другой стороны, если `items` был создан вне компонента, он сохраняет свои предыдущие значения и запоминает изменения:

```js hl_lines="1 7"
const items = []; // 🔴 Bad: created outside of the component
function FriendList({ friends }) {
    for (let i = 0; i < friends.length; i++) {
        const friend = friends[i];
        items.push(
            <Friend key={friend.id} friend={friend} />
        ); // 🔴 Bad: mutates a value created outside of render
    }
    return <section>{items}</section>;
}
```

Когда `<FriendList />` будет запущен снова, мы продолжим добавлять `friends` к `items` при каждом запуске этого компонента, что приведет к многочисленным дублирующимся результатам. Эта версия `<FriendList />` имеет наблюдаемые побочные эффекты [во время рендеринга](#how-does-react-run-your-code) и **нарушает правило**.

#### Ленивая инициализация {#lazy-initialization}

Ленивая инициализация также подходит, несмотря на то, что не является полностью «чистой»:

```js hl_lines="2"
function ExpenseForm() {
    SuperCalculator.initializeIfNotReady(); // ✅ Good: if it
    // doesn't affect other components
    // Continue rendering...
}
```

#### Изменение DOM {#changing-the-dom}

В логике рендеринга компонентов React не допускаются побочные эффекты, которые видны непосредственно пользователю. Другими словами, простой вызов функции компонента сам по себе не должен приводить к изменениям на экране.

```js hl_lines="2"
function ProductDetailPage({ product }) {
    document.window.title = product.title; // 🔴 Bad: Changes the DOM
}
```

Один из способов добиться желаемого результата обновления `window.title` вне рендеринга - это [синхронизировать компонент с `window`](../../learn/synchronizing-with-effects.md).

Пока вызов компонента несколько раз безопасен и не влияет на рендеринг других компонентов, React не волнует, является ли он на 100% чистым в строгом смысле функционального программирования. Важнее, что [компоненты должны быть идемпотентными](./components-and-hooks-must-be-pure.md).

## Пропсы и состояние неизменяемы {#props-and-state-are-immutable}

Пропсы и состояние компонента неизменяемы [snapshots](../../learn/state-as-a-snapshot.md). Никогда не изменяйте их напрямую. Вместо этого передавайте новые пропсы вниз и используйте функцию setter из `useState`.

Можно рассматривать пропсы и значения состояния как моментальные снимки, которые обновляются после рендеринга. По этой причине вы не изменяете переменные props или state напрямую: вместо этого вы передаете новые props или используете предоставленную вам функцию setter, чтобы сообщить React, что состояние должно быть обновлено при следующем рендеринге компонента.

### Не мутируйте пропсы {#props}

Пропсы неизменяемы, потому что если вы их измените, приложение будет выдавать непоследовательный вывод, который может быть трудно отладить, поскольку он может работать или не работать в зависимости от обстоятельств.

```js hl_lines="2"
function Post({ item }) {
    item.url = new Url(item.url, base); // 🔴 Bad: never mutate props directly
    return <Link url={item.url}>{item.title}</Link>;
}
```

---

```js hl_lines="2"
function Post({ item }) {
    const url = new Url(item.url, base); // ✅ Good: make a copy instead
    return <Link url={url}>{item.title}</Link>;
}
```

### Не мутируйте состояние {#state}

`useState` возвращает переменную state и сеттер для обновления этого состояния.

```js
const [stateVariable, setter] = useState(0);
```

Вместо того чтобы обновлять переменную state на месте, нам нужно обновить ее с помощью функции setter, которую возвращает `useState`. Изменение значения переменной state не приводит к обновлению компонента, оставляя пользователей с устаревшим пользовательским интерфейсом. Использование функции setter информирует React о том, что состояние изменилось, и что нам нужно поставить в очередь повторный рендеринг для обновления пользовательского интерфейса.

```js hl_lines="5"
function Counter() {
    const [count, setCount] = useState(0);

    function handleClick() {
        count = count + 1; // 🔴 Bad: never mutate state directly
    }

    return (
        <button onClick={handleClick}>
            You pressed me {count} times
        </button>
    );
}
```

---

```js hl_lines="5-6"
function Counter() {
    const [count, setCount] = useState(0);

    function handleClick() {
        setCount(count + 1); // ✅ Good: use the setter function
        // returned by useState
    }

    return (
        <button onClick={handleClick}>
            You pressed me {count} times
        </button>
    );
}
```

## Возвращаемые значения и аргументы хуков неизменяемы {#return-values-and-arguments-to-hooks-are-immutable}

После передачи значений в хук их нельзя изменять. Как и пропсы в JSX, значения становятся неизменяемыми при передаче хуку.

```js hl_lines="4-5"
function useIconStyle(icon) {
    const theme = useContext(ThemeContext);
    if (icon.enabled) {
        // 🔴 Bad: never mutate hook arguments directly
        icon.className = computeStyle(icon, theme);
    }
    return icon;
}
```

---

```js hl_lines="3"
function useIconStyle(icon) {
    const theme = useContext(ThemeContext);
    const newIcon = { ...icon }; // ✅ Good: make a copy instead
    if (icon.enabled) {
        newIcon.className = computeStyle(icon, theme);
    }
    return newIcon;
}
```

Одним из важных принципов React является _локальное рассуждение_: способность понять, что делает компонент или хук, глядя на его код в изоляции. К хукам следует относиться как к «черным ящикам», когда они вызываются. Например, пользовательский хук может использовать свои аргументы в качестве зависимостей для мемоизации значений внутри него:

```js
function useIconStyle(icon) {
    const theme = useContext(ThemeContext);

    return useMemo(() => {
        const newIcon = { ...icon };
        if (icon.enabled) {
            newIcon.className = computeStyle(icon, theme);
        }
        return newIcon;
    }, [icon, theme]);
}
```

Если вы измените аргументы Hooks, мемоизация пользовательского хука станет некорректной, поэтому важно избегать этого.

```js hl_lines="2"
style = useIconStyle(icon); // `style` is memoized based on `icon`
icon.enabled = false; // Bad: 🔴 never mutate hook arguments directly
style = useIconStyle(icon); // previously memoized result is returned
```

---

```js hl_lines="2"
style = useIconStyle(icon); // `style` is memoized based on `icon`
icon = { ...icon, enabled: false }; // Good: ✅ make a copy instead
style = useIconStyle(icon); // new value of `style` is calculated
```

Аналогично, важно не изменять возвращаемые значения хуков, так как они могут быть мемоизированы.

## Значения неизменяемы после передачи в JSX {#values-are-immutable-after-being-passed-to-jsx}

Не мутируйте значения после того, как они были использованы в JSX. Переместите мутацию до создания JSX.

Когда вы используете JSX в выражении, React может нетерпеливо оценить JSX до того, как компонент закончит рендеринг. Это означает, что мутация значений после их передачи в JSX может привести к устаревшему пользовательскому интерфейсу, поскольку React не будет знать, что нужно обновить вывод компонента.

```js hl_lines="4-5"
function Page({ colour }) {
    const styles = { colour, size: 'large' };
    const header = <Header styles={styles} />;
    // 🔴 Bad: styles was already used in the JSX above
    styles.size = 'small';
    const footer = <Footer styles={styles} />;
    return (
        <>
            {header}
            <Content />
            {footer}
        </>
    );
}
```

---

```js hl_lines="4-5"
function Page({ colour }) {
    const headerStyles = { colour, size: 'large' };
    const header = <Header styles={headerStyles} />;
    // ✅ Good: we created a new value
    const footerStyles = { colour, size: 'small' };
    const footer = <Footer styles={footerStyles} />;
    return (
        <>
            {header}
            <Content />
            {footer}
        </>
    );
}
```

<small>:material-information-outline: Источник &mdash; <https://react.dev/reference/rules/components-and-hooks-must-be-pure></small>

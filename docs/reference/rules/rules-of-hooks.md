---
description: Хуки определяются с помощью функций JavaScript, но они представляют собой особый тип многократно используемой логики пользовательского интерфейса с ограничениями на то, где они могут быть вызваны
---

# Правила работы с хуками

<big>
Хуки определяются с помощью функций JavaScript, но они представляют собой особый тип многократно используемой логики пользовательского интерфейса с ограничениями на то, где они могут быть вызваны.
</big>

## Вызывайте хуки только на верхнем уровне {#only-call-hooks-at-the-top-level}

Функции, названия которых начинаются с `use`, в React называются [_Hooks_](../react/hooks.md).

**Не вызывайте хуки внутри циклов, условий, вложенных функций или блоков `try`/`catch`/`finally`.** Вместо этого всегда используйте хуки на верхнем уровне вашей функции React, перед любым ранним возвратом. Вы можете вызывать хуки только во время рендеринга компонента функции React:

-   ✅ Вызывайте их на верхнем уровне в теле [функционального компонента](../../learn/your-first-component.md).
-   ✅ Вызовите их на верхнем уровне в теле [пользовательского хука](../../learn/reusing-logic-with-custom-hooks.md).

```js
function Counter() {
    // ✅ Good: top-level in a function component
    const [count, setCount] = useState(0);
    // ...
}

function useWindowWidth() {
    // ✅ Good: top-level in a custom Hook
    const [width, setWidth] = useState(window.innerWidth);
    // ...
}
```

Вызов хуков (функций, начинающихся с `use`) в любых других случаях не поддерживается, например:

-   🔴 Не вызывайте хуки внутри условий или циклов.
-   🔴 Не вызывайте хуки после условного оператора `возврата`.
-   🔴 Не вызывайте хуки в обработчиках событий.
-   🔴 Не вызывайте хуки в компонентах классов.
-   🔴 Не вызывайте хуки в функциях, передаваемых в `useMemo`, `useReducer` или `useEffect`.
-   🔴 Не вызывайте хуки внутри блоков `try`/`catch`/`finally`.

Если вы нарушите эти правила, вы можете увидеть эту ошибку.

```js
function Bad({ cond }) {
    if (cond) {
        // 🔴 Bad: inside a condition (to fix, move it outside!)
        const theme = useContext(ThemeContext);
    }
    // ...
}

function Bad() {
    for (let i = 0; i < 10; i++) {
        // 🔴 Bad: inside a loop (to fix, move it outside!)
        const theme = useContext(ThemeContext);
    }
    // ...
}

function Bad({ cond }) {
    if (cond) {
        return;
    }
    // 🔴 Bad: after a conditional return (to fix, move it before the return!)
    const theme = useContext(ThemeContext);
    // ...
}

function Bad() {
    function handleClick() {
        // 🔴 Bad: inside an event handler (to fix, move it outside!)
        const theme = useContext(ThemeContext);
    }
    // ...
}

function Bad() {
    const style = useMemo(() => {
        // 🔴 Bad: inside useMemo (to fix, move it outside!)
        const theme = useContext(ThemeContext);
        return createStyle(theme);
    });
    // ...
}

class Bad extends React.Component {
    render() {
        // 🔴 Bad: inside a class component (to fix, write a function component instead of a class!)
        useEffect(() => {});
        // ...
    }
}

function Bad() {
    try {
        // 🔴 Bad: inside try/catch/finally block (to fix, move it outside!)
        const [x, setX] = useState(0);
    } catch {
        const [x, setX] = useState(1);
    }
}
```

Вы можете использовать плагин [`eslint-plugin-react-hooks`](https://www.npmjs.com/package/eslint-plugin-react-hooks), чтобы отловить эти ошибки.

!!!note ""

    [Пользовательские хуки](../../learn/reusing-logic-with-custom-hooks.md) _могут_ вызывать другие хуки (в этом и заключается их назначение). Это работает, потому что пользовательские хуки также должны вызываться только во время рендеринга компонента функции.

## Вызывайте хуки только из функций React {#only-call-hooks-from-react-functions}

Не вызывайте хуки из обычных функций JavaScript. Вместо этого вы можете:

✅ Вызывать хуки из компонентов функций React.
✅ Вызывать хуки из [пользовательских хуков](../../learn/reusing-logic-with-custom-hooks.md#extracting-your-own-custom-hook-from-a-component).

Следуя этому правилу, вы гарантируете, что вся логика с состоянием в компоненте будет четко видна из его исходного кода.

```js
function FriendList() {
    const [
        onlineStatus,
        setOnlineStatus,
    ] = useOnlineStatus(); // ✅
}

function setOnlineStatus() {
    // ❌ Not a component or custom Hook!
    const [
        onlineStatus,
        setOnlineStatus,
    ] = useOnlineStatus();
}
```

<small>:material-information-outline: Источник &mdash; <https://react.dev/reference/rules/rules-of-hooks></small>

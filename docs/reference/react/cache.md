---
status: experimental
description: cache позволяет кэшировать результат выборки данных или вычислений
---

# cache

!!!example "Canary"

    -   `cache` предназначен только для использования с [React Server Components](https://react.dev/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components). Смотрите [фреймворки](../../learn/start-a-new-react-project.md#bleeding-edge-react-frameworks), которые поддерживают React Server Components.

    -   `cache` доступен только в каналах React [Canary](https://react.dev/community/versioning-policy#canary-channel) и [experimental](https://react.dev/community/versioning-policy#experimental-channel). Пожалуйста, убедитесь, что вы понимаете ограничения, прежде чем использовать `cache` в производстве. Узнайте больше о [каналах выпуска React здесь](https://react.dev/community/versioning-policy#all-release-channels).

<big>`cache` позволяет кэшировать результат выборки данных или вычислений.</big>

```js
const cachedFn = cache(fn);
```

## Описание {#reference}

### `cache(fn)` {#cache}

Вызовите `cache` вне каких-либо компонентов, чтобы создать версию функции с кэшированием.

```js hl_lines="4 7"
import { cache } from 'react';
import calculateMetrics from 'lib/metrics';

const getMetrics = cache(calculateMetrics);

function Chart({ data }) {
    const report = getMetrics(data);
    // ...
}
```

При первом вызове `getMetrics` с `data`, `getMetrics` вызовет `calculateMetrics(data)` и сохранит результат в кэше. Если `getMetrics` будет вызвана снова с теми же `data`, она вернет кэшированный результат вместо повторного вызова `calculateMetrics(data)`.

#### Параметры {#parameters}

-   `fn`: Функция, для которой вы хотите кэшировать результаты. Функция `fn` может принимать любые аргументы и возвращать любое значение.

#### Возвращает {#returns}

`cache` возвращает кэшированную версию `fn` с той же сигнатурой типа. При этом вызов `fn` не производится.

При вызове `cachedFn` с заданными аргументами сначала проверяется, существует ли кэшированный результат в кэше. Если кэшированный результат существует, он возвращает его. Если нет, он вызывает `fn` с аргументами, сохраняет результат в кэше и возвращает его. Единственный раз, когда вызывается `fn`, это когда происходит пропуск кэша.

!!!note "Мемоизация"

    Оптимизация кэширования возвращаемых значений на основе входных данных известна как [_мемоизация_](https://ru.wikipedia.org/wiki/Мемоизация). Мы называем функцию, возвращаемую из `cache`, мемоизированной функцией.

#### Замечания {#caveats}

-   React аннулирует кэш для всех мемоизированных функций при каждом запросе сервера.
-   Каждый вызов `cache` создает новую функцию. Это означает, что вызов `cache` с одной и той же функцией несколько раз будет возвращать разные мемоизированные функции, которые не используют один и тот же кэш.
-   `cachedFn` также будет кэшировать ошибки. Если `fn` выбрасывает ошибку для определенных аргументов, она будет кэширована, и та же ошибка будет повторно выброшена, когда `cachedFn` будет вызвана с теми же аргументами.
-   `cache` предназначен только для использования в [Серверных компонентах](https://react.dev/blog/2023/03/22/react-labs-what-we-have-been-working-on-march-2023#react-server-components).

## Использование {#usage}

### Кэширование дорогих вычислений {#cache-expensive-computation}

Используйте `cache` для пропуска дублирующей работы.

```js hl_lines="7 13"
import { cache } from 'react';
import calculateUserMetrics from 'lib/user';

const getUserMetrics = cache(calculateUserMetrics);

function Profile({ user }) {
    const metrics = getUserMetrics(user);
    // ...
}

function TeamReport({ users }) {
    for (let user in users) {
        const metrics = getUserMetrics(user);
        // ...
    }
    // ...
}
```

Если один и тот же объект `user` отображается и в `Profile`, и в `TeamReport`, оба компонента могут разделить работу и вызвать `calculateUserMetrics` только один раз для этого `user`.

Предположим, что первым рендерится `Profile`. Он вызовет `getUserMetrics` и проверит, есть ли кэшированный результат. Поскольку `getUserMetrics` вызывается впервые для этого 'user', произойдет пропуск кэша. Затем `getUserMetrics` вызовет `calculateUserMetrics` с этим `пользователем` и запишет результат в кэш.

Когда `TeamReport` отобразит свой список 'users' и достигнет того же самого объекта `user`, он вызовет `getUserMetrics` и прочитает результат из кэша.

#### Вызов разных мемоизированных функций будет считывать данные из разных кэшей {#pitfall-different-memoized-functions}

Чтобы получить доступ к одному и тому же кэшу, компоненты должны вызывать одну и ту же мемоизированную функцию.

```js hl_lines="8-9"
// Temperature.js
import { cache } from 'react';
import { calculateWeekReport } from './report';

export function Temperature({ cityData }) {
    // 🚩 Wrong: Calling `cache` in component creates
    // new `getWeekReport` for each render
    const getWeekReport = cache(calculateWeekReport);
    const report = getWeekReport(cityData);
    // ...
}
```

---

```js hl_lines="7 10"
// Precipitation.js
import { cache } from 'react';
import { calculateWeekReport } from './report';

// 🚩 Wrong: `getWeekReport` is only accessible
// for `Precipitation` component.
const getWeekReport = cache(calculateWeekReport);

export function Precipitation({ cityData }) {
    const report = getWeekReport(cityData);
    // ...
}
```

В приведенном выше примере `Precipitation` и `Temperature` каждый вызывает `cache` для создания новой мемоизированной функции с собственным поиском в кэше. Если оба компонента выполняют рендеринг для одного и того же `cityData`, они будут выполнять дублирующую работу по вызову `calculateWeekReport`.

Кроме того, `Temperature` создает новую мемоизированную функцию каждый раз, когда компонент рендерится, что не позволяет разделить кэш.

Чтобы максимизировать количество обращений к кэшу и сократить объем работы, оба компонента должны вызывать одну и ту же мемоизированную функцию для доступа к одному и тому же кэшу. Вместо этого определите мемоизированную функцию в специальном модуле, который можно [`import`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/import) для всех компонентов.

```js hl_lines="5"
// getWeekReport.js
import { cache } from 'react';
import { calculateWeekReport } from './report';

export default cache(calculateWeekReport);
```

---

```js hl_lines="2 5"
// Temperature.js
import getWeekReport from './getWeekReport';

export default function Temperature({ cityData }) {
    const report = getWeekReport(cityData);
    // ...
}
```

---

```js hl_lines="2 5"
// Precipitation.js
import getWeekReport from './getWeekReport';

export default function Precipitation({ cityData }) {
    const report = getWeekReport(cityData);
    // ...
}
```

Здесь оба компонента вызывают одну и ту же мемоизированную функцию, экспортированную из `./getWeekReport.js`, для чтения и записи в один и тот же кэш.

### Совместное использование снимка данных {#take-and-share-snapshot-of-data}

Чтобы поделиться снимком данных между компонентами, вызовите `cache` с функцией получения данных, например `fetch`. Когда несколько компонентов выполняют одну и ту же выборку данных, выполняется только один запрос, а возвращаемые данные кэшируются и разделяются между компонентами. Все компоненты обращаются к одному и тому же снимку данных во время рендеринга сервера.

```js
import { cache } from 'react';
import { fetchTemperature } from './api.js';

const getTemperature = cache(async (city) => {
    return await fetchTemperature(city);
});

async function AnimatedWeatherCard({ city }) {
    const temperature = await getTemperature(city);
    // ...
}

async function MinimalWeatherCard({ city }) {
    const temperature = await getTemperature(city);
    // ...
}
```

Если `AnimatedWeatherCard` и `MinimalWeatherCard` рендерятся для одного и того же города, то они получат один и тот же снимок данных из мемоизированной функции.

Если `AnimatedWeatherCard` и `MinimalWeatherCard` передают разные аргументы города в `getTemperature`, то `fetchTemperature` будет вызван дважды, и каждый сайт вызова получит разные данные.

Город действует как ключ кэша.

!!!note "Асинхронный рендеринг"

    Асинхронный рендеринг поддерживается только для серверных компонентов.

    ```js
    async function AnimatedWeatherCard({ city }) {
    	const temperature = await getTemperature(city);
    	// ...
    }
    ```

### Предварительная загрузка данных {#preload-data}

Кэширование длительной выборки данных позволяет запустить асинхронную работу до рендеринга компонента.

```js
const getUser = cache(async (id) => {
    return await db.user.query(id);
});

async function Profile({ id }) {
    const user = await getUser(id);
    return (
        <section>
            <img src={user.profilePic} />
            <h2>{user.name}</h2>
        </section>
    );
}

function Page({ id }) {
    // ✅ Good: start fetching the user data
    getUser(id);
    // ... some computational work
    return (
        <>
            <Profile id={id} />
        </>
    );
}
```

При рендеринге `Page` компонент вызывает `getUser`, но обратите внимание, что он не использует возвращенные данные. Этот ранний вызов `getUser` запускает асинхронный запрос к базе данных, который происходит, пока `Page` выполняет другую вычислительную работу и рендерит дочерние страницы.

При рендеринге `Profile` мы снова вызываем `getUser`. Если первоначальный вызов `getUser` уже вернул и кэшировал данные о пользователе, то когда `Profile` запрашивает и ждет эти данные, он может просто прочитать их из кэша, не требуя повторного вызова удаленной процедуры. Если первоначальный запрос данных не был завершен, предварительная загрузка данных в этом шаблоне уменьшает задержку в получении данных.

#### Кэширование асинхронной работы {#caching-asynchronous-work}

При выполнении [асинхронной функции](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function) вы получите [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) для этой работы. Promise содержит состояние этой работы (_в ожидании_, _выполнена_, _не выполнена_) и ее конечный результат.

В этом примере асинхронная функция `fetchData` возвращает обещание, которое ожидает `fetch`.

```js
async function fetchData() {
    return await fetch(`https://...`);
}

const getData = cache(fetchData);

async function MyComponent() {
    getData();
    // ... some computational work
    await getData();
    // ...
}
```

При первом вызове `getData` обещание, возвращенное из `fetchData`, кэшируется. Последующие вызовы будут возвращать то же обещание.

Обратите внимание, что в первом вызове `getData` не используется `await`, тогда как во втором - используется. [`await`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/await) - это оператор JavaScript, который будет ждать и вернет готовый результат обещания. Первый вызов `getData` просто инициирует `fetch` для кэширования обещания, чтобы второй `getData` мог его просмотреть.

Если ко второму вызову обещание все еще _ожидает_, то `await` сделает паузу для получения результата. Оптимизация заключается в том, что пока мы ждем `fetch`, React может продолжать вычислительную работу, тем самым сокращая время ожидания второго вызова.

Если обещание уже выполнено, либо ошибка, либо _выполненный_ результат, `await` вернет это значение немедленно. В обоих случаях выигрыш в производительности налицо.

#### Вызов мемоизированной функции вне компонента не будет использовать кэш {#pitfall-memoized-call-outside-component}

```js
import { cache } from 'react';

const getUser = cache(async (userId) => {
    return await db.user.query(userId);
});

// 🚩 Wrong: Calling memoized function outside
// of component will not memoize.
getUser('demo-id');

async function DemoProfile() {
    // ✅ Good: `getUser` will memoize.
    const user = await getUser('demo-id');
    return <Profile user={user} />;
}
```

React предоставляет доступ к кэшу только для мемоизированной функции в компоненте. При вызове `getUser` вне компонента, он по-прежнему будет оценивать функцию, но не будет считывать или обновлять кэш.

Это происходит потому, что доступ к кэшу осуществляется через [контекст](../../learn/passing-data-deeply-with-context.md), который доступен только из компонента.

### Когда следует использовать `cache`, [`memo`](./memo.md) или [`useMemo`](./useMemo.md)? {#cache-memo-usememo}

Все упомянутые API предлагают мемоизацию, но разница заключается в том, что они предназначены для мемоизации, кто может получить доступ к кэшу и когда их кэш будет аннулирован.

#### `useMemo` {#deep-dive-use-memo}

В общем случае для кэширования дорогостоящих вычислений в клиентском компоненте при разных рендерах следует использовать [`useMemo`](./useMemo.md). Например, для мемоизации преобразования данных внутри компонента.

```js hl_lines="4"
'use client';

function WeatherReport({ record }) {
    const avgTemp = useMemo(
        () => calculateAvg(record),
        record
    );
    // ...
}

function App() {
    const record = getRecord();
    return (
        <>
            <WeatherReport record={record} />
            <WeatherReport record={record} />
        </>
    );
}
```

В этом примере `App` отображает два `WeatherReport` с одной и той же записью. Несмотря на то, что оба компонента выполняют одну и ту же работу, они не могут делиться ею. Кэш `useMemo` является локальным только для компонента.

Однако `useMemo` гарантирует, что если `App` перерендерится и объект `record` не изменится, каждый экземпляр компонента пропустит работу и использует мемоизированное значение `avgTemp`. `useMemo` будет кэшировать только последнее вычисление `avgTemp` с заданными зависимостями.

#### `cache` {#deep-dive-cache}

В общем случае следует использовать `cache` в серверных компонентах для запоминания работы, которая может быть разделена между компонентами.

```js
const cachedFetchReport = cache(fetchReport);

function WeatherReport({ city }) {
    const report = cachedFetchReport(city);
    // ...
}

function App() {
    const city = 'Los Angeles';
    return (
        <>
            <WeatherReport city={city} />
            <WeatherReport city={city} />
        </>
    );
}
```

Если переписать предыдущий пример и использовать `cache`, то в этом случае второй экземпляр `WeatherReport` сможет пропустить дублирование работы и читать из того же кэша, что и первый `WeatherReport`. Еще одним отличием от предыдущего примера является то, что `cache` также рекомендуется для мемоизации поиска данных, в отличие от `useMemo`, который должен использоваться только для вычислений.

В настоящее время `cache` следует использовать только в серверных компонентах, и при запросах к серверу кэш будет аннулирован.

#### `memo` {#deep-dive-memo}

Вы должны использовать [`memo`](./memo.md) для предотвращения повторного рендеринга компонента, если его реквизиты не изменились.

```js
'use client';

function WeatherReport({ record }) {
    const avgTemp = calculateAvg(record);
    // ...
}

const MemoWeatherReport = memo(WeatherReport);

function App() {
    const record = getRecord();
    return (
        <>
            <MemoWeatherReport record={record} />
            <MemoWeatherReport record={record} />
        </>
    );
}
```

В этом примере оба компонента `MemoWeatherReport` вызовут `calculateAvg` при первом рендеринге. Однако если `App` перерендерится без изменений в `record`, ни один из реквизитов не изменится, и `MemoWeatherReport` не перерендерится.

По сравнению с `useMemo`, `memo` мемоизирует рендеринг компонента на основе реквизитов, а не конкретных вычислений. Как и в случае с `useMemo`, компонент с мемоизацией кэширует только последний рендер с последними значениями реквизитов. Как только реквизит меняется, кэш аннулируется и компонент рендерится заново.

## Устранение неполадок {#troubleshooting}

### Моя мемоизированная функция все еще выполняется, даже если я вызывал ее с теми же аргументами {#memoized-function-still-runs}

См. ранее упомянутые подводные камни

-   [Вызов разных мемоизированных функций будет считываться из разных кэшей](#pitfall-different-memoized-functions)
-   [Вызов мемоизированной функции вне компонента не будет использовать кэш](#pitfall-memoized-call-outside-component).

Если ничего из вышеперечисленного не работает, возможно, проблема в том, как React проверяет, существует ли что-то в кэше.

Если ваши аргументы не являются [примитивами](https://developer.mozilla.org/docs/Glossary/Primitive) (например, объекты, функции, массивы), убедитесь, что вы передаете одну и ту же ссылку на объект.

При вызове мемоизированной функции React просмотрит входные аргументы на предмет того, не кэширован ли уже результат. React будет использовать неглубокое равенство аргументов, чтобы определить, есть ли попадание в кэш.

```js
import { cache } from 'react';

const calculateNorm = cache((vector) => {
    // ...
});

function MapMarker(props) {
    // 🚩 Wrong: props is an object that changes every render.
    const length = calculateNorm(props);
    // ...
}

function App() {
    return (
        <>
            <MapMarker x={10} y={10} z={10} />
            <MapMarker x={10} y={10} z={10} />
        </>
    );
}
```

В данном случае два `MapMarker` выглядят так, как будто они выполняют одну и ту же работу и вызывают `calculateNorm` с одним и тем же значением `{x: 10, y: 10, z:10}`. Несмотря на то, что объекты содержат одинаковые значения, они не являются одной и той же объектной ссылкой, поскольку каждый компонент создает свой собственный объект `props`.

React вызовет [`Object.is`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/is) на входе, чтобы проверить, есть ли попадание в кэш.

```js hl_lines="3 9"
import { cache } from 'react';

const calculateNorm = cache((x, y, z) => {
    // ...
});

function MapMarker(props) {
    // ✅ Good: Pass primitives to memoized function
    const length = calculateNorm(props.x, props.y, props.z);
    // ...
}

function App() {
    return (
        <>
            <MapMarker x={10} y={10} z={10} />
            <MapMarker x={10} y={10} z={10} />
        </>
    );
}
```

Одним из способов решения этой проблемы может быть передача размеров вектора в `calculateNorm`. Это работает, потому что сами размеры являются примитивами.

Другим решением может быть передача компоненту самого объекта вектора в качестве параметра. Нам нужно будет передать один и тот же объект обоим экземплярам компонента.

```js hl_lines="3 9"
import { cache } from 'react';

const calculateNorm = cache((vector) => {
    // ...
});

function MapMarker(props) {
    // ✅ Good: Pass the same `vector` object
    const length = calculateNorm(props.vector);
    // ...
}

function App() {
    const vector = [10, 10, 10];
    return (
        <>
            <MapMarker vector={vector} />
            <MapMarker vector={vector} />
        </>
    );
}
```

<small>:material-information-outline: Источник &mdash; [https://react.dev/reference/react/cache](https://react.dev/reference/react/cache)</small>

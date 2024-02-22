# Suspense

`<Suspense>` позволяет отображать фалбэк до тех пор, пока его дочерние элементы не закончат загрузку.

<!-- 0001.part.md -->

```js
<Suspense fallback={<Loading />}>
    <SomeComponent />
</Suspense>
```

<!-- 0002.part.md -->

## Описание

### `<Suspense>`

#### пропсы

-   `children`: Фактический пользовательский интерфейс, который вы собираетесь рендерить. Если `children` приостановится во время рендеринга, граница Suspense переключится на рендеринг `fallback`.
-   `fallback`: Альтернативный пользовательский интерфейс, который будет отображаться вместо реального пользовательского интерфейса, если он не закончил загрузку. Принимается любой допустимый узел React, хотя на практике запасной вариант - это легковесное представление-заполнитель, например, загрузочный спиннер или скелет. Приостановка будет автоматически переключаться на `fallback`, когда `children` приостанавливает работу, и обратно на `children`, когда данные будут готовы. Если `fallback` приостанавливает работу во время рендеринга, он активирует ближайшую родительскую границу Suspense.

#### Ограничения

-   React не сохраняет состояние для рендеров, которые были приостановлены до того, как они смогли смонтироваться в первый раз. Когда компонент загрузится, React повторит попытку рендеринга приостановленного дерева с нуля.
-   Если `Suspense` отображал содержимое для дерева, но затем снова приостановился, то `откат` будет показан снова, если только обновление, вызвавшее его, не было вызвано [`startTransition`](startTransition.md) или [`useDeferredValue`](useDeferredValue.md).
-   Если React необходимо скрыть уже видимый контент из-за повторного приостановления, он очистит [layout Effects](useLayoutEffect.md) в дереве контента. Когда контент снова будет готов к показу, React снова запустит Эффекты компоновки. Это гарантирует, что Эффекты, измеряющие макет DOM, не попытаются сделать это, пока содержимое скрыто.
-   React включает в себя такие "подкапотные" оптимизации, как _Streaming Server Rendering_ и _Selective Hydration_, которые интегрированы в Suspense. Чтобы узнать больше, прочитайте [архитектурный обзор](https://github.com/reactwg/react-18/discussions/37) и посмотрите [технический доклад](https://www.youtube.com/watch?v=pj5N-Khihgc).

## Использование

### Отображение фалбэка во время загрузки контента

Вы можете обернуть любую часть вашего приложения границей Suspense:

<!-- 0003.part.md -->

```js
<Suspense fallback={<Loading />}>
    <Albums />
</Suspense>
```

<!-- 0004.part.md -->

React будет отображать ваш loading fallback до тех пор, пока весь код и данные, необходимые потомкам не будут загружены.

В приведенном ниже примере компонент `Albums` _приостанавливается_ на время получения списка альбомов. Пока он не готов к рендерингу, React переключает ближайшую границу Suspense выше, чтобы показать отступающий компонент - ваш компонент `Loading`. Затем, когда данные загружаются, React скрывает компонент `Loading` и отображает компонент `Albums` с данными.

```js
import { Suspense } from 'react';
import Albums from './Albums.js';

export default function ArtistPage({ artist }) {
    return (
        <>
            <h1>{artist.name}</h1>
            <Suspense fallback={<Loading />}>
                <Albums artistId={artist.id} />
            </Suspense>
        </>
    );
}

function Loading() {
    return <h2>🌀 Loading...</h2>;
}
```

!!!note ""

    **Только источники данных с поддержкой Suspense активируют компонент Suspense.** К ним относятся:

    -   Получение данных с помощью фреймворков с поддержкой Suspense, таких как [Relay](https://relay.dev/docs/guided-tour/rendering/loading-states/) и [Next.js](https://nextjs.org/docs/getting-started/react-essentials).
    -   Ленивая загрузка кода компонента с помощью [`lazy`](lazy.md).

    Suspense **не** обнаруживает, когда данные извлекаются внутри Effect или обработчика события.

    Точный способ загрузки данных в компонент `Albums`, описанный выше, зависит от вашего фреймворка. Если вы используете фреймворк с поддержкой Suspense, вы найдете подробности в документации по получению данных.

    Получение данных с поддержкой Suspense без использования мнений фреймворка пока не поддерживается. Требования к реализации источника данных с поддержкой Suspense нестабильны и не документированы. Официальный API для интеграции источников данных с Suspense будет выпущен в одной из будущих версий React.

### Раскрытие содержимого сразу

По умолчанию все дерево внутри Suspense рассматривается как единое целое. Например, даже если _только один_ из этих компонентов приостановится в ожидании каких-то данных, _все_ они вместе будут заменены индикатором загрузки:

<!-- 0015.part.md -->

```js
<Suspense fallback={<Loading />}>
    <Biography />
    <Panel>
        <Albums />
    </Panel>
</Suspense>
```

<!-- 0016.part.md -->

Затем, когда все они будут готовы к отображению, они появятся все вместе одновременно.

В приведенном ниже примере и `Biography`, и `Albums` получают некоторые данные. Однако, поскольку они сгруппированы под одной границей Suspense, эти компоненты всегда "всплывают" вместе в одно и то же время.

=== "ArtistPage.js"

    ```js
    import { Suspense } from 'react';
    import Albums from './Albums.js';
    import Biography from './Biography.js';
    import Panel from './Panel.js';

    export default function ArtistPage({ artist }) {
    	return (
    		<>
    			<h1>{artist.name}</h1>
    			<Suspense fallback={<Loading />}>
    				<Biography artistId={artist.id} />
    				<Panel>
    					<Albums artistId={artist.id} />
    				</Panel>
    			</Suspense>
    		</>
    	);
    }

    function Loading() {
    	return <h2>🌀 Loading...</h2>;
    }
    ```

=== "Panel.js"

    ```js
    export default function Panel({ children }) {
    	return <section className="panel">{children}</section>;
    }
    ```

Компоненты, загружающие данные, не обязательно должны быть прямыми дочерними компонентами границы Suspense. Например, вы можете переместить `Biography` и `Albums` в новый компонент `Details`. Это не изменит поведение. `Biography` и `Albums` имеют одну и ту же ближайшую родительскую границу Suspense, поэтому их раскрытие координируется вместе.

<!-- 0033.part.md -->

```js
<Suspense fallback={<Loading />}>
    <Details artistId={artist.id} />
</Suspense>;

function Details({ artistId }) {
    return (
        <>
            <Biography artistId={artistId} />
            <Panel>
                <Albums artistId={artistId} />
            </Panel>
        </>
    );
}
```

### Раскрытие вложенного содержимого по мере загрузки

Когда компонент приостанавливается, ближайший родительский Suspense-компонент показывает запасной вариант. Это позволяет вложить несколько компонентов Suspense для создания последовательности загрузки. Падение каждой границы Suspense будет заполняться по мере того, как становится доступным содержимое следующего уровня. Например, вы можете дать списку альбомов свой собственный откат:

<!-- 0035.part.md -->

```js
<Suspense fallback={<BigSpinner />}>
    <Biography />
    <Suspense fallback={<AlbumsGlimmer />}>
        <Panel>
            <Albums />
        </Panel>
    </Suspense>
</Suspense>
```

<!-- 0036.part.md -->

С этим изменением отображение `Биографии` не должно "ждать" загрузки `Альбомов`.

Последовательность будет следующей:

1.  Если `Биография` еще не загрузилась, `BigSpinner` отображается вместо всей области содержимого.
2.  Как только `Биография` завершает загрузку, `BigSpinner` заменяется содержимым.
3.  Если `Альбомы` еще не загрузились, `AlbumsGlimmer` отображается вместо `Альбомов` и его родительской `Панели`.
4.  Наконец, когда `Albums` завершает загрузку, он заменяет `AlbumsGlimmer`.

=== "ArtistPage.js"

    ```js
    import { Suspense } from 'react';
    import Albums from './Albums.js';
    import Biography from './Biography.js';
    import Panel from './Panel.js';

    export default function ArtistPage({ artist }) {
    	return (
    		<>
    			<h1>{artist.name}</h1>
    			<Suspense fallback={<BigSpinner />}>
    				<Biography artistId={artist.id} />
    				<Suspense fallback={<AlbumsGlimmer />}>
    					<Panel>
    						<Albums artistId={artist.id} />
    					</Panel>
    				</Suspense>
    			</Suspense>
    		</>
    	);
    }

    function BigSpinner() {
    	return <h2>🌀 Loading...</h2>;
    }

    function AlbumsGlimmer() {
    	return (
    		<div className="glimmer-panel">
    			<div className="glimmer-line" />
    			<div className="glimmer-line" />
    			<div className="glimmer-line" />
    		</div>
    	);
    }
    ```

=== "Panel.js"

    ```js
    export default function Panel({ children }) {
    	return <section className="panel">{children}</section>;
    }
    ```

Приостановочные границы позволяют вам координировать, какие части пользовательского интерфейса должны всегда "всплывать" одновременно, а какие - постепенно раскрывать больше содержимого в последовательности состояний загрузки. Вы можете добавлять, перемещать или удалять Suspense-границы в любом месте дерева, не влияя на поведение остального приложения.

Не ставьте приостанавливающую границу вокруг каждого компонента. Границы приостановки не должны быть более детализированными, чем последовательность загрузки, которую вы хотите, чтобы испытал пользователь. Если вы работаете с дизайнером, спросите его, где должны располагаться состояния загрузки - скорее всего, они уже включили их в свои эскизы.

### Показ устаревшего контента во время загрузки свежего

В этом примере компонент `SearchResults` приостанавливается на время получения результатов поиска. Введите `"a"`, дождитесь результатов, а затем измените его на `"ab"`. Результаты для `"a"` будут заменены загрузочным фалбэком.

```js
import { Suspense, useState } from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
    const [query, setQuery] = useState('');
    return (
        <>
            <label>
                Search albums:
                <input
                    value={query}
                    onChange={(e) =>
                        setQuery(e.target.value)
                    }
                />
            </label>
            <Suspense fallback={<h2>Loading...</h2>}>
                <SearchResults query={query} />
            </Suspense>
        </>
    );
}
```

Распространенным альтернативным шаблоном пользовательского интерфейса является _отложенное_ обновление списка и отображение предыдущих результатов до тех пор, пока не будут готовы новые результаты. Хук [`useDeferredValue`](useDeferredValue.md) позволяет вам передать отложенную версию запроса вниз:

<!-- 0063.part.md -->

```js
export default function App() {
    const [query, setQuery] = useState('');
    const deferredQuery = useDeferredValue(query);
    return (
        <>
            <label>
                Search albums:
                <input
                    value={query}
                    onChange={(e) =>
                        setQuery(e.target.value)
                    }
                />
            </label>
            <Suspense fallback={<h2>Loading...</h2>}>
                <SearchResults query={deferredQuery} />
            </Suspense>
        </>
    );
}
```

<!-- 0064.part.md -->

Запрос `query` будет обновлен немедленно, поэтому на входе будет отображаться новое значение. Однако `deferredQuery` сохранит свое предыдущее значение до тех пор, пока данные не загрузятся, поэтому `SearchResults` будет отображать устаревшие результаты некоторое время.

Чтобы сделать это более очевидным для пользователя, вы можете добавить визуальную индикацию, когда отображается список несвежих результатов:

<!-- 0065.part.md -->

```js
<div
    style={{
        opacity: query !== deferredQuery ? 0.5 : 1,
    }}
>
    <SearchResults query={deferredQuery} />
</div>
```

<!-- 0066.part.md -->

Введите `"a"` в примере ниже, дождитесь загрузки результатов, а затем измените ввод на `"ab"`. Обратите внимание, что вместо отката на приостановку вы теперь видите затемненный список несвежих результатов, пока не загрузятся новые результаты:

```js
import {
    Suspense,
    useState,
    useDeferredValue,
} from 'react';
import SearchResults from './SearchResults.js';

export default function App() {
    const [query, setQuery] = useState('');
    const deferredQuery = useDeferredValue(query);
    const isStale = query !== deferredQuery;
    return (
        <>
            <label>
                Search albums:
                <input
                    value={query}
                    onChange={(e) =>
                        setQuery(e.target.value)
                    }
                />
            </label>
            <Suspense fallback={<h2>Loading...</h2>}>
                <div style={{ opacity: isStale ? 0.5 : 1 }}>
                    <SearchResults query={deferredQuery} />
                </div>
            </Suspense>
        </>
    );
}
```

!!!note ""

    И отложенные значения, и transitions позволяют вам избежать отображения Suspense fallback в пользу встроенных индикаторов. Переходы помечают все обновление как несрочное, поэтому они обычно используются фреймворками и библиотеками маршрутизаторов для навигации. Отложенные значения, с другой стороны, в основном полезны в коде приложений, где вы хотите пометить часть пользовательского интерфейса как несрочную и позволить ей "отстать" от остальной части пользовательского интерфейса.

### Предотвращение скрытия уже раскрытого содержимого

Когда компонент приостанавливается, ближайшая родительская граница приостановки переключается на отображение резервного копирования. Это может привести к искажению пользовательского опыта, если уже отображалось какое-то содержимое. Попробуйте нажать эту кнопку:

=== "App.js"

    ```js
    import { Suspense, useState } from 'react';
    import IndexPage from './IndexPage.js';
    import ArtistPage from './ArtistPage.js';
    import Layout from './Layout.js';

    export default function App() {
    	return (
    		<Suspense fallback={<BigSpinner />}>
    			<Router />
    		</Suspense>
    	);
    }

    function Router() {
    	const [page, setPage] = useState('/');

    	function navigate(url) {
    		setPage(url);
    	}

    	let content;
    	if (page === '/') {
    		content = <IndexPage navigate={navigate} />;
    	} else if (page === '/the-beatles') {
    		content = (
    			<ArtistPage
    				artist={{
    					id: 'the-beatles',
    					name: 'The Beatles',
    				}}
    			/>
    		);
    	}
    	return <Layout>{content}</Layout>;
    }

    function BigSpinner() {
    	return <h2>🌀 Loading...</h2>;
    }
    ```

=== "Layout.js"

    ```js
    export default function Layout({ children }) {
    	return (
    		<div className="layout">
    			<section className="header">
    				Music Browser
    			</section>
    			<main>{children}</main>
    		</div>
    	);
    }
    ```

=== "IndexPage.js"

    ```js
    export default function IndexPage({ navigate }) {
    	return (
    		<button onClick={() => navigate('/the-beatles')}>
    			Open The Beatles artist page
    		</button>
    	);
    }
    ```

=== "ArtistPage.js"

    ```js
    import { Suspense } from 'react';
    import Albums from './Albums.js';
    import Biography from './Biography.js';
    import Panel from './Panel.js';

    export default function ArtistPage({ artist }) {
    	return (
    		<>
    			<h1>{artist.name}</h1>
    			<Biography artistId={artist.id} />
    			<Suspense fallback={<AlbumsGlimmer />}>
    				<Panel>
    					<Albums artistId={artist.id} />
    				</Panel>
    			</Suspense>
    		</>
    	);
    }

    function AlbumsGlimmer() {
    	return (
    		<div className="glimmer-panel">
    			<div className="glimmer-line" />
    			<div className="glimmer-line" />
    			<div className="glimmer-line" />
    		</div>
    	);
    }
    ```

При нажатии кнопки компонент `Router` отображал `ArtistPage` вместо `IndexPage`. Компонент внутри `ArtistPage` приостанавливался, поэтому ближайшая граница Suspense начинала показывать откат. Ближайшая Suspense-граница находилась рядом с корнем, поэтому весь макет сайта заменялся на `BigSpinner`.

Чтобы предотвратить это, вы можете пометить обновление состояния навигации как _переход_ с помощью [`startTransition`:](startTransition.md)

<!-- 0097.part.md -->

```js
function Router() {
    const [page, setPage] = useState('/');

    function navigate(url) {
        startTransition(() => {
            setPage(url);
        });
    }
    // ...
}
```

<!-- 0098.part.md -->

Это говорит React, что переход состояния не является срочным, и лучше продолжать показывать предыдущую страницу вместо того, чтобы скрывать уже открытое содержимое. Теперь нажатие на кнопку "ждет" загрузки `Биографии`:

=== "App.js"

    ```js
    import { Suspense, startTransition, useState } from 'react';
    import IndexPage from './IndexPage.js';
    import ArtistPage from './ArtistPage.js';
    import Layout from './Layout.js';

    export default function App() {
    	return (
    		<Suspense fallback={<BigSpinner />}>
    			<Router />
    		</Suspense>
    	);
    }

    function Router() {
    	const [page, setPage] = useState('/');

    	function navigate(url) {
    		startTransition(() => {
    			setPage(url);
    		});
    	}

    	let content;
    	if (page === '/') {
    		content = <IndexPage navigate={navigate} />;
    	} else if (page === '/the-beatles') {
    		content = (
    			<ArtistPage
    				artist={{
    					id: 'the-beatles',
    					name: 'The Beatles',
    				}}
    			/>
    		);
    	}
    	return <Layout>{content}</Layout>;
    }

    function BigSpinner() {
    	return <h2>🌀 Loading...</h2>;
    }
    ```

=== "Layout.js"

    ```js
    export default function Layout({ children }) {
    	return (
    		<div className="layout">
    			<section className="header">
    				Music Browser
    			</section>
    			<main>{children}</main>
    		</div>
    	);
    }
    ```

=== "IndexPage.js"

    ```js
    export default function IndexPage({ navigate }) {
    	return (
    		<button onClick={() => navigate('/the-beatles')}>
    			Open The Beatles artist page
    		</button>
    	);
    }
    ```

=== "ArtistPage.js"

    ```js
    import { Suspense } from 'react';
    import Albums from './Albums.js';
    import Biography from './Biography.js';
    import Panel from './Panel.js';

    export default function ArtistPage({ artist }) {
    	return (
    		<>
    			<h1>{artist.name}</h1>
    			<Biography artistId={artist.id} />
    			<Suspense fallback={<AlbumsGlimmer />}>
    				<Panel>
    					<Albums artistId={artist.id} />
    				</Panel>
    			</Suspense>
    		</>
    	);
    }

    function AlbumsGlimmer() {
    	return (
    		<div className="glimmer-panel">
    			<div className="glimmer-line" />
    			<div className="glimmer-line" />
    			<div className="glimmer-line" />
    		</div>
    	);
    }
    ```

Переход не ждет, пока загрузится _все_ содержимое. Он ждет только достаточно долго, чтобы не скрыть уже открытое содержимое. Например, сайт `Layout` уже был показан, поэтому было бы плохо скрывать его за загружающимся волчком. Однако вложенная граница `Suspense` вокруг `Albums` является новой, поэтому переход не ждет ее.

!!!note ""

    Ожидается, что маршрутизаторы с поддержкой Suspense по умолчанию будут оборачивать обновления навигации в переходы.

---

### Индикация того, что переход происходит

В приведенном выше примере после нажатия на кнопку нет визуальной индикации того, что происходит переход. Чтобы добавить индикатор, вы можете заменить [`startTransition`](startTransition.md) на [`useTransition`](useTransition.md), что даст вам булево значение `isPending`. В примере ниже это используется для изменения стиля заголовка сайта во время перехода:

=== "App.js"

    ```js
    import { Suspense, useState, useTransition } from 'react';
    import IndexPage from './IndexPage.js';
    import ArtistPage from './ArtistPage.js';
    import Layout from './Layout.js';

    export default function App() {
    	return (
    		<Suspense fallback={<BigSpinner />}>
    			<Router />
    		</Suspense>
    	);
    }

    function Router() {
    	const [page, setPage] = useState('/');
    	const [isPending, startTransition] = useTransition();

    	function navigate(url) {
    		startTransition(() => {
    			setPage(url);
    		});
    	}

    	let content;
    	if (page === '/') {
    		content = <IndexPage navigate={navigate} />;
    	} else if (page === '/the-beatles') {
    		content = (
    			<ArtistPage
    				artist={{
    					id: 'the-beatles',
    					name: 'The Beatles',
    				}}
    			/>
    		);
    	}
    	return <Layout isPending={isPending}>{content}</Layout>;
    }

    function BigSpinner() {
    	return <h2>🌀 Loading...</h2>;
    }
    ```

=== "Layout.js"

    ```js
    export default function Layout({ children, isPending }) {
    	return (
    		<div className="layout">
    			<section
    				className="header"
    				style={{
    					opacity: isPending ? 0.7 : 1,
    				}}
    			>
    				Music Browser
    			</section>
    			<main>{children}</main>
    		</div>
    	);
    }
    ```

=== "IndexPage.js"

    ```js
    export default function IndexPage({ navigate }) {
    	return (
    		<button onClick={() => navigate('/the-beatles')}>
    			Open The Beatles artist page
    		</button>
    	);
    }
    ```

=== "ArtistPage.js"

    ```js
    import { Suspense } from 'react';
    import Albums from './Albums.js';
    import Biography from './Biography.js';
    import Panel from './Panel.js';

    export default function ArtistPage({ artist }) {
    	return (
    		<>
    			<h1>{artist.name}</h1>
    			<Biography artistId={artist.id} />
    			<Suspense fallback={<AlbumsGlimmer />}>
    				<Panel>
    					<Albums artistId={artist.id} />
    				</Panel>
    			</Suspense>
    		</>
    	);
    }

    function AlbumsGlimmer() {
    	return (
    		<div className="glimmer-panel">
    			<div className="glimmer-line" />
    			<div className="glimmer-line" />
    			<div className="glimmer-line" />
    		</div>
    	);
    }
    ```

### Сброс границ приостановки при навигации

Во время перехода React будет избегать скрытия уже показанного содержимого. Однако, если вы переходите на маршрут с другими параметрами, вы можете захотеть сказать React, что это _другой_ контент. Вы можете выразить это с помощью `key`:

<!-- 0139.part.md -->

```js
<ProfilePage key={queryParams.id} />
```

<!-- 0140.part.md -->

Представьте, что вы перемещаетесь по странице профиля пользователя, и что-то приостанавливается. Если это обновление завернуто в переход, оно не вызовет откат для уже видимого содержимого. Это ожидаемое поведение.

Однако теперь представьте, что вы перемещаетесь между двумя разными профилями пользователей. В этом случае имеет смысл показать откат. Например, временная шкала одного пользователя представляет собой _различное содержимое_, чем временная шкала другого пользователя. Указывая `ключ`, вы гарантируете, что React рассматривает профили разных пользователей как разные компоненты и сбрасывает границы Suspense во время навигации. Маршрутизаторы, интегрированные в Suspense, должны делать это автоматически.

### Предоставление обратного хода для ошибок сервера и контента только для сервера

Если вы используете один из [API потокового серверного рендеринга](../react-dom/server/index.md) (или фреймворк, который полагается на них), React также будет использовать ваши границы `<Suspense>` для обработки ошибок на сервере. Если компонент выдает ошибку на сервере, React не будет прерывать серверный рендеринг. Вместо этого он найдет ближайший компонент `<Suspense>` над ним и включит его фалбэк (например, спиннер) в сгенерированный серверный HTML. Пользователь сначала увидит спиннер.

На клиенте React попытается отрисовать тот же компонент еще раз. Если и на клиенте произойдет ошибка, React выдаст ошибку и отобразит ближайшую [границу ошибки](Component.md). Однако, если ошибка не произойдет на клиенте, React не будет отображать ошибку пользователю, так как содержимое в итоге было отображено успешно.

Вы можете использовать это, чтобы исключить некоторые компоненты из рендеринга на сервере. Для этого бросьте ошибку в серверное окружение, а затем оберните их в границу `<Suspense>`, чтобы заменить их HTML фалбэками:

<!-- 0141.part.md -->

```js
<Suspense fallback={<Loading />}>
    <Chat />
</Suspense>;

function Chat() {
    if (typeof window === 'undefined') {
        throw Error(
            'Chat should only render on the client.'
        );
    }
    // ...
}
```

<!-- 0142.part.md -->

HTML сервера будет включать индикатор загрузки. На клиенте он будет заменен компонентом `Chat`.

## Устранение неполадок

### Как предотвратить замену пользовательского интерфейса на fallback во время обновления?

Замена видимого пользовательского интерфейса на фалбэк приводит к резким изменениям в работе пользователя. Это может произойти, когда обновление приводит к приостановке компонента, а ближайшая граница приостановки уже показывает пользователю содержимое.

Чтобы этого не произошло, пометьте обновление как несрочное с помощью `startTransition`. Во время перехода React будет ждать, пока загрузится достаточно данных, чтобы предотвратить появление нежелательного отката:

<!-- 0143.part.md -->

```js
function handleNextPageClick() {
    // If this update suspends, don't hide the already displayed content
    startTransition(() => {
        setCurrentPage(currentPage + 1);
    });
}
```

<!-- 0144.part.md -->

Это позволит избежать скрытия существующего содержимого. Тем не менее, все новые границы `Suspense` будут немедленно отображать отступления, чтобы избежать блокировки пользовательского интерфейса и позволить пользователю видеть содержимое по мере его появления.

**React будет предотвращать нежелательные отступления только во время несрочных обновлений**. Он не будет задерживать рендеринг, если он является результатом срочного обновления. Вы должны выбрать API, например [`startTransition`](startTransition.md) или [`useDeferredValue`](useDeferredValue.md).

Если ваш маршрутизатор интегрирован с Suspense, он должен автоматически обернуть свои обновления в [`startTransition`](startTransition.md).

## Ссылки

-   [https://react.dev/reference/react/Suspense](https://react.dev/reference/react/Suspense)

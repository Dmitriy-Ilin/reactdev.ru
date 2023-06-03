# useCallback

**`useCallback`** - это хук React, который позволяет кэшировать определение функции между повторными рендерингами.

<!-- 0001.part.md -->

```js
const cachedFn = useCallback(fn, dependencies);
```

## Описание

### `useCallback(fn, dependencies)`

Вызовите `useCallback` на верхнем уровне вашего компонента, чтобы кэшировать определение функции между рендерингами:

<!-- 0003.part.md -->

```js
import { useCallback } from 'react';

export default function ProductPage({
    productId,
    referrer,
    theme,
}) {
    const handleSubmit = useCallback(
        (orderDetails) => {
            post('/product/' + productId + '/buy', {
                referrer,
                orderDetails,
            });
        },
        [productId, referrer]
    );
    // ...
}
```

#### Параметры

-   `fn`: Значение функции, которое вы хотите кэшировать. Она может принимать любые аргументы и возвращать любые значения. React вернет (не вызовет!) вашу функцию обратно во время первоначального рендера. При последующих рендерах React вернет вам ту же функцию, если `зависимости` не изменились с момента последнего рендера. В противном случае, он отдаст вам функцию, которую вы передали во время текущего рендеринга, и сохранит ее на случай, если она может быть использована позже. React не будет вызывать вашу функцию. Функция возвращается вам, чтобы вы могли решить, когда и стоит ли ее вызывать.

-   `dependencies`: Список всех реактивных значений, на которые ссылается код `fn`. Реактивные значения включают реквизиты, состояние, а также все переменные и функции, объявленные непосредственно в теле вашего компонента. Если ваш линтер [настроен на React](../learn/editor-setup.md), он проверит, что каждое реактивное значение правильно указано в качестве зависимости. Список зависимостей должен иметь постоянное количество элементов и быть написан inline по типу `[dep1, dep2, dep3]`. React будет сравнивать каждую зависимость с предыдущим значением, используя алгоритм сравнения [`Object.is`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/is).

#### Возвраты

При первоначальном рендере `useCallback` возвращает переданную вами функцию `fn`.

При последующих рендерах она либо вернет уже сохраненную функцию `fn` из последнего рендера (если зависимости не изменились), либо вернет функцию `fn`, которую вы передали во время этого рендера.

#### Ограничения

-   `useCallback` - это хук, поэтому вы можете вызывать его только **на верхнем уровне вашего компонента** или ваших собственных хуков. Вы не можете вызывать его внутри циклов или условий. Если вам это нужно, создайте новый компонент и перенесите состояние в него.
-   React **не будет выбрасывать кэшированную функцию, если для этого нет особой причины.** Например, в разработке React выбрасывает кэш, когда вы редактируете файл вашего компонента. Как в разработке, так и в продакшене, React отбрасывает кэш, если ваш компонент приостанавливается во время начального монтирования. В будущем React может добавить больше функций, которые будут использовать преимущества отбрасывания кэша - например, если React в будущем добавит встроенную поддержку виртуализированных списков, то будет иметь смысл отбрасывать кэш для элементов, которые прокручиваются из области просмотра виртуализированной таблицы. Это должно соответствовать вашим ожиданиям, если вы полагаетесь на `useCallback` в качестве оптимизации производительности. В противном случае, более подходящими могут быть [state variable](useState.md) или [ref](useRef.md).

## Использование

### Пропуск повторного рендеринга компонентов

При оптимизации производительности рендеринга иногда необходимо кэшировать функции, которые вы передаете в команду

<!-- 0005.part.md -->

Чтобы кэшировать функцию между повторными рендерингами компонента, оберните ее определение в хук `useCallback`:

<!-- 0006.part.md -->

```js
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
    const handleSubmit = useCallback(
        (orderDetails) => {
            post('/product/' + productId + '/buy', {
                referrer,
                orderDetails,
            });
        },
        [productId, referrer]
    );
    // ...
}
```

<!-- 0007.part.md -->

Вам нужно передать две вещи в `useCallback`:

1.  Определение функции, которую вы хотите кэшировать между повторными рендерами.
2.  Список зависимостей, включающий каждое значение в вашем компоненте, которое используется в вашей функции.

При первом рендере возвращаемая функция, которую вы получите от `useCallback`, будет функцией, которую вы передали.

При последующих рендерах React будет сравнивать зависимости с зависимостями, которые вы передали во время предыдущего рендера. Если ни одна из зависимостей не изменилась (по сравнению с [`Object.is`](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/is)), `useCallback` вернет ту же функцию, что и раньше. В противном случае, `useCallback` вернет функцию, которую вы передали на _этом_ рендере.

Другими словами, `useCallback` кэширует функцию между повторными рендерами, пока ее зависимости не изменятся.

**Давайте рассмотрим пример, чтобы увидеть, когда это полезно.**

Скажем, вы передаете функцию `handleSubmit` от `ProductPage` компоненту `ShippingForm`:

<!-- 0008.part.md -->

```js
function ProductPage({ productId, referrer, theme }) {
    // ...
    return (
        <div className={theme}>
            <ShippingForm onSubmit={handleSubmit} />
        </div>
    );
    // ...
}
```

<!-- 0009.part.md -->

Вы заметили, что при переключении параметра `theme` приложение на мгновение замирает, но если убрать `<ShippingForm />` из JSX, то все работает быстро. Это говорит о том, что стоит попробовать оптимизировать компонент `ShippingForm`.

**По умолчанию, когда компонент рендерится, React рекурсивно рендерит все его дочерние компоненты.** Вот почему, когда `ProductPage` рендерится с другой `темой`, компонент `ShippingForm` _также_ рендерится. Это хорошо для компонентов, которые не требуют больших вычислений для повторного рендеринга. Но если вы убедились, что повторный рендеринг медленный, вы можете сказать `ShippingForm` пропустить повторный рендеринг, когда его реквизиты такие же, как и при последнем рендере, обернув его в [`memo`:](memo.md)

<!-- 0010.part.md -->

```js
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({
    onSubmit,
}) {
    // ...
});
```

<!-- 0011.part.md -->

**После этого изменения `ShippingForm` будет пропускать повторный рендеринг, если все его реквизиты _те же_, что и при последнем рендеринге.** Вот когда кэширование функции становится важным! Допустим, вы определили `handleSubmit` без `useCallback`:

<!-- 0012.part.md -->

```js
function ProductPage({ productId, referrer, theme }) {
    // Every time the theme changes, this will be a different function...
    function handleSubmit(orderDetails) {
        post('/product/' + productId + '/buy', {
            referrer,
            orderDetails,
        });
    }

    return (
        <div className={theme}>
            {/* ... so ShippingForm's props will never be the same,
			and it will re-render every time */}
            <ShippingForm onSubmit={handleSubmit} />
        </div>
    );
}
```

<!-- 0013.part.md -->

**В JavaScript `функция () {}` или `() => {}` всегда создает _разную_ функцию,** подобно тому, как объектный литерал `{}` всегда создает новый объект. Обычно это не является проблемой, но это означает, что реквизит `ShippingForm` никогда не будет одинаковым, и ваша оптимизация [`memo`](memo.md) не будет работать. Вот здесь-то и пригодится `useCallback`:

<!-- 0014.part.md -->

```js
function ProductPage({ productId, referrer, theme }) {
    // Tell React to cache your function between re-renders...
    const handleSubmit = useCallback(
        (orderDetails) => {
            post('/product/' + productId + '/buy', {
                referrer,
                orderDetails,
            });
        },
        [productId, referrer]
    ); // ...so as long as these dependencies don't change...

    return (
        <div className={theme}>
            {/* ...ShippingForm will receive
			the same props and can skip re-rendering */}
            <ShippingForm onSubmit={handleSubmit} />
        </div>
    );
}
```

<!-- 0015.part.md -->

**Вернув `handleSubmit` в `useCallback`, вы гарантируете, что это будет _одна и та же_ функция между повторными рендерами** (пока не изменятся зависимости). Вы не обязаны _обертывать_ функцию в `useCallback`, если только вы не делаете это по какой-то конкретной причине. В данном примере причина в том, что вы передаете ее компоненту, обернутому в [`memo`](memo.md), и это позволяет ему пропустить повторный рендеринг. Есть и другие причины, по которым вам может понадобиться `useCallback`, которые описаны далее на этой странице.

!!!note ""

    **Вы должны полагаться на `useCallback` только в качестве оптимизации производительности.** Если ваш код не работает без него, найдите основную проблему и сначала устраните ее. Затем вы можете добавить `useCallback` обратно.

!!!note "Как useCallback связан с useMemo?"

    Вы часто будете видеть [`useMemo`](useMemo.md) вместе с `useCallback`. Они оба полезны, когда вы пытаетесь оптимизировать дочерний компонент. Они позволяют вам [memoize](https://ru.wikipedia.org/wiki/%D0%9C%D0%B5%D0%BC%D0%BE%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F) (или, другими словами, кэшировать) то, что вы передаете вниз:

    ```js
    import { useMemo, useCallback } from 'react';

    function ProductPage({ productId, referrer }) {
    	const product = useData('/product/' + productId);

    	const requirements = useMemo(() => {
    		// Calls your function and caches its result
    		return computeRequirements(product);
    	}, [product]);

    	const handleSubmit = useCallback(
    		(orderDetails) => {
    			// Caches your function itself
    			post('/product/' + productId + '/buy', {
    				referrer,
    				orderDetails,
    			});
    		},
    		[productId, referrer]
    	);

    	return (
    		<div className={theme}>
    			<ShippingForm
    				requirements={requirements}
    				onSubmit={handleSubmit}
    			/>
    		</div>
    	);
    }
    ```

    Разница в том, что именно они позволяют вам кэшировать:

    -   **[`useMemo`](useMemo.md) кэширует _результат_ вызова вашей функции.** В этом примере он кэширует результат вызова `computeRequirements(product)`, чтобы он не изменился, если `product` не изменился. Это позволяет передавать объект `requirements` вниз без ненужного повторного рендеринга `ShippingForm`. При необходимости React будет вызывать переданную вами функцию во время рендеринга для вычисления результата.
    -   **`useCallback` кэширует _саму функцию._** В отличие от `useMemo`, он не вызывает предоставленную вами функцию. Вместо этого она кэширует предоставленную вами функцию, так что `handleSubmit` _сама по себе_ не изменяется, если только `productId` или `referrer` не изменились. Это позволяет вам передавать функцию `handleSubmit` вниз без ненужного повторного рендеринга `ShippingForm`. Ваш код не будет выполняться до тех пор, пока пользователь не отправит форму.

    Если вы уже знакомы с [`useMemo`](useMemo.md), вам будет полезно представить `useCallback` следующим образом:

    ```js
    // Simplified implementation (inside React)
    function useCallback(fn, dependencies) {
    	return useMemo(() => fn, dependencies);
    }
    ```

    [Подробнее о разнице между `useMemo` и `useCallback`.](useMemo.md)

!!!note "Должны ли вы везде добавлять useCallback?"

    Если ваше приложение похоже на этот сайт, и большинство взаимодействий являются грубыми (например, замена страницы или целого раздела), мемоизация обычно не нужна. С другой стороны, если ваше приложение больше похоже на редактор рисунков, и большинство взаимодействий являются гранулированными (например, перемещение фигур), то мемоизация может оказаться очень полезной.

    Кэширование функции с `useCallback` полезно только в нескольких случаях:

    -   Вы передаете ее в качестве параметра компоненту, обернутому в [`memo`](memo.md). Вы хотите пропустить повторный рендеринг, если значение не изменилось. Мемоизация позволяет вашему компоненту повторно отображаться только в том случае, если изменились зависимости.
    -   Функция, которую вы передаете, позже будет использоваться как зависимость какого-то Hook. Например, другая функция, обернутая в `useCallback`, зависит от нее, или вы зависите от этой функции из [`useEffect`](useEffect.md).

    В других случаях нет никакой пользы от обертывания функции в `useCallback`. Вреда от этого тоже нет, поэтому некоторые команды предпочитают не думать об отдельных случаях и мемоизировать как можно больше. Недостатком является то, что код становится менее читабельным. Кроме того, не вся мемоизация эффективна: одного значения, которое "всегда новое", достаточно, чтобы сломать мемоизацию для всего компонента.

    Обратите внимание, что `useCallback` не предотвращает _создание_ функции. Вы всегда создаете функцию (и это fine!), но React игнорирует это и возвращает вам кэшированную функцию, если ничего не изменилось.

    **На практике вы можете сделать ненужной мемоизацию, следуя нескольким принципам:**.

    1.  Когда компонент визуально оборачивает другие компоненты, пусть он [принимает JSX в качестве дочерних компонентов](../learn/passing-props-to-a-component.md) Тогда, если компонент-обертка обновляет свое состояние, React знает, что его дочерние компоненты не нужно перерисовывать.
    2.  Предпочитайте локальное состояние и не [поднимайте состояние вверх](../learn/sharing-state-between-components.md) дальше, чем это необходимо. Не храните переходные состояния, такие как формы и то, наведен ли элемент на вершину вашего дерева или в глобальной библиотеке состояний.
    3.  Сохраняйте чистоту [логики рендеринга](../learn/keeping-components-pure.md) Если повторный рендеринг компонента вызывает проблему или приводит к заметным визуальным артефактам, это ошибка в вашем компоненте! Исправьте ошибку вместо того, чтобы добавлять мемоизацию.
    4.  Избегайте [ненужных Эффектов, обновляющих состояние](../learn/you-might-not-need-an-effect.md) Большинство проблем с производительностью в приложениях React вызваны цепочками обновлений, исходящих от Эффектов, которые заставляют ваши компоненты рендериться снова и снова.
    5.  Попробуйте [удалить ненужные зависимости из ваших Эффектов](../learn/removing-effect-dependencies.md). Например, вместо мемоизации часто проще переместить какой-то объект или функцию внутрь Эффекта или за пределы компонента.

    Если конкретное взаимодействие все еще кажется нестабильным, [используйте профилировщик React Developer Tools](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html), чтобы увидеть, какие компоненты больше всего выигрывают от мемоизации, и добавьте мемоизацию там, где это необходимо. Эти принципы облегчают отладку и понимание ваших компонентов, поэтому следовать им полезно в любом случае. В перспективе мы исследуем [возможность автоматической мемоизации](https://www.youtube.com/watch?v=lGEMwh32soc), чтобы решить эту проблему раз и навсегда.

<!-- 0020.part.md -->

### Разница между useCallback и объявлением функции напрямую

#### 1. Пропуск повторного рендеринга с `useCallback` и `memo`

В этом примере компонент `ShippingForm` **искусственно замедлен**, чтобы вы могли увидеть, что происходит, когда React-компонент, который вы рендерите, действительно медленный. Попробуйте увеличить счетчик и переключить тему.

Увеличение счетчика кажется медленным, потому что это заставляет замедленный `ShippingForm` перерисовываться. Это ожидаемо, потому что счетчик изменился, и вам нужно отразить новый выбор пользователя на экране.

Далее попробуйте переключить тему. **Благодаря `useCallback` вместе с [`memo`](memo.md), это происходит быстро, несмотря на искусственное замедление!** `ShippingForm` пропускает повторное отображение, потому что функция `handleSubmit` не изменилась. Функция `handleSubmit` не изменилась, потому что `productId` и `referrer` (ваши зависимости `useCallback`) не изменились с момента последнего рендеринга.

=== "App.js"

    ```js
    import { useState } from 'react';
    import ProductPage from './ProductPage.js';

    export default function App() {
    	const [isDark, setIsDark] = useState(false);
    	return (
    		<>
    			<label>
    				<input
    					type="checkbox"
    					checked={isDark}
    					onChange={(e) =>
    						setIsDark(e.target.checked)
    					}
    				/>
    				Dark mode
    			</label>
    			<hr />
    			<ProductPage
    				referrerId="wizard_of_oz"
    				productId={123}
    				theme={isDark ? 'dark' : 'light'}
    			/>
    		</>
    	);
    }
    ```

=== "ProductPage.js"

    ```js
    import { useCallback } from 'react';
    import ShippingForm from './ShippingForm.js';

    export default function ProductPage({
    	productId,
    	referrer,
    	theme,
    }) {
    	const handleSubmit = useCallback(
    		(orderDetails) => {
    			post('/product/' + productId + '/buy', {
    				referrer,
    				orderDetails,
    			});
    		},
    		[productId, referrer]
    	);

    	return (
    		<div className={theme}>
    			<ShippingForm onSubmit={handleSubmit} />
    		</div>
    	);
    }

    function post(url, data) {
    	// Imagine this sends a request...
    	console.log('POST /' + url);
    	console.log(data);
    }
    ```

=== "ShippingForm.js"

    ```js
    import { memo, useState } from 'react';

    const ShippingForm = memo(function ShippingForm({
    	onSubmit,
    }) {
    	const [count, setCount] = useState(1);

    	console.log(
    		'[ARTIFICIALLY SLOW] Rendering <ShippingForm />'
    	);
    	let startTime = performance.now();
    	while (performance.now() - startTime < 500) {
    		// Do nothing for 500 ms to emulate extremely slow code
    	}

    	function handleSubmit(e) {
    		e.preventDefault();
    		const formData = new FormData(e.target);
    		const orderDetails = {
    			...Object.fromEntries(formData),
    			count,
    		};
    		onSubmit(orderDetails);
    	}

    	return (
    		<form onSubmit={handleSubmit}>
    			<p>
    				<b>
    					Note: <code>ShippingForm</code> is
    					artificially slowed down!
    				</b>
    			</p>
    			<label>
    				Number of items:
    				<button
    					type="button"
    					onClick={() => setCount(count - 1)}
    				>
    					–
    				</button>
    				{count}
    				<button
    					type="button"
    					onClick={() => setCount(count + 1)}
    				>
    					+
    				</button>
    			</label>
    			<label>
    				Street:
    				<input name="street" />
    			</label>
    			<label>
    				City:
    				<input name="city" />
    			</label>
    			<label>
    				Postal code:
    				<input name="zipCode" />
    			</label>
    			<button type="submit">Submit</button>
    		</form>
    	);
    });

    export default ShippingForm;
    ```

#### 2. Всегда перерендеринг компонента

В этом примере реализация `ShippingForm` также **искусственно замедлена**, чтобы вы могли увидеть, что происходит, когда какой-либо компонент React, который вы рендерите, действительно медленный. Попробуйте увеличить счетчик и переключить тему.

В отличие от предыдущего примера, переключение темы теперь также происходит медленно! Это происходит потому, что **в этой версии нет вызова `useCallback`,** поэтому `handleSubmit` всегда является новой функцией, и замедленный компонент `ShippingForm` не может пропустить повторный рендеринг.

=== "App.js"

    ```js
    import { useState } from 'react';
    import ProductPage from './ProductPage.js';

    export default function App() {
    	const [isDark, setIsDark] = useState(false);
    	return (
    		<>
    			<label>
    				<input
    					type="checkbox"
    					checked={isDark}
    					onChange={(e) =>
    						setIsDark(e.target.checked)
    					}
    				/>
    				Dark mode
    			</label>
    			<hr />
    			<ProductPage
    				referrerId="wizard_of_oz"
    				productId={123}
    				theme={isDark ? 'dark' : 'light'}
    			/>
    		</>
    	);
    }
    ```

=== "ProductPage.js"

    ```js
    import ShippingForm from './ShippingForm.js';

    export default function ProductPage({
    	productId,
    	referrer,
    	theme,
    }) {
    	function handleSubmit(orderDetails) {
    		post('/product/' + productId + '/buy', {
    			referrer,
    			orderDetails,
    		});
    	}

    	return (
    		<div className={theme}>
    			<ShippingForm onSubmit={handleSubmit} />
    		</div>
    	);
    }

    function post(url, data) {
    	// Imagine this sends a request...
    	console.log('POST /' + url);
    	console.log(data);
    }
    ```

=== "ShippingForm.js"

    ```js
    import { memo, useState } from 'react';

    const ShippingForm = memo(function ShippingForm({
    	onSubmit,
    }) {
    	const [count, setCount] = useState(1);

    	console.log(
    		'[ARTIFICIALLY SLOW] Rendering <ShippingForm />'
    	);
    	let startTime = performance.now();
    	while (performance.now() - startTime < 500) {
    		// Do nothing for 500 ms to emulate extremely slow code
    	}

    	function handleSubmit(e) {
    		e.preventDefault();
    		const formData = new FormData(e.target);
    		const orderDetails = {
    			...Object.fromEntries(formData),
    			count,
    		};
    		onSubmit(orderDetails);
    	}

    	return (
    		<form onSubmit={handleSubmit}>
    			<p>
    				<b>
    					Note: <code>ShippingForm</code> is
    					artificially slowed down!
    				</b>
    			</p>
    			<label>
    				Number of items:
    				<button
    					type="button"
    					onClick={() => setCount(count - 1)}
    				>
    					–
    				</button>
    				{count}
    				<button
    					type="button"
    					onClick={() => setCount(count + 1)}
    				>
    					+
    				</button>
    			</label>
    			<label>
    				Street:
    				<input name="street" />
    			</label>
    			<label>
    				City:
    				<input name="city" />
    			</label>
    			<label>
    				Postal code:
    				<input name="zipCode" />
    			</label>
    			<button type="submit">Submit</button>
    		</form>
    	);
    });

    export default ShippingForm;
    ```

Однако, вот тот же код **с искусственным замедлением.** Отсутствие `useCallback` ощущается заметно или нет?

=== "App.js"

    ```js
    import { useState } from 'react';
    import ProductPage from './ProductPage.js';

    export default function App() {
    	const [isDark, setIsDark] = useState(false);
    	return (
    		<>
    			<label>
    				<input
    					type="checkbox"
    					checked={isDark}
    					onChange={(e) =>
    						setIsDark(e.target.checked)
    					}
    				/>
    				Dark mode
    			</label>
    			<hr />
    			<ProductPage
    				referrerId="wizard_of_oz"
    				productId={123}
    				theme={isDark ? 'dark' : 'light'}
    			/>
    		</>
    	);
    }
    ```

=== "ProductPage.js"

    ```js
    import ShippingForm from './ShippingForm.js';

    export default function ProductPage({
    	productId,
    	referrer,
    	theme,
    }) {
    	function handleSubmit(orderDetails) {
    		post('/product/' + productId + '/buy', {
    			referrer,
    			orderDetails,
    		});
    	}

    	return (
    		<div className={theme}>
    			<ShippingForm onSubmit={handleSubmit} />
    		</div>
    	);
    }

    function post(url, data) {
    	// Imagine this sends a request...
    	console.log('POST /' + url);
    	console.log(data);
    }
    ```

=== "ShippingForm.js"

    ```js
    import { memo, useState } from 'react';

    const ShippingForm = memo(function ShippingForm({
    	onSubmit,
    }) {
    	const [count, setCount] = useState(1);

    	console.log('Rendering <ShippingForm />');

    	function handleSubmit(e) {
    		e.preventDefault();
    		const formData = new FormData(e.target);
    		const orderDetails = {
    			...Object.fromEntries(formData),
    			count,
    		};
    		onSubmit(orderDetails);
    	}

    	return (
    		<form onSubmit={handleSubmit}>
    			<label>
    				Number of items:
    				<button
    					type="button"
    					onClick={() => setCount(count - 1)}
    				>
    					–
    				</button>
    				{count}
    				<button
    					type="button"
    					onClick={() => setCount(count + 1)}
    				>
    					+
    				</button>
    			</label>
    			<label>
    				Street:
    				<input name="street" />
    			</label>
    			<label>
    				City:
    				<input name="city" />
    			</label>
    			<label>
    				Postal code:
    				<input name="zipCode" />
    			</label>
    			<button type="submit">Submit</button>
    		</form>
    	);
    });

    export default ShippingForm;
    ```

Довольно часто код без мемоизации работает нормально. Если ваши взаимодействия достаточно быстрые, мемоизация не нужна.

Помните, что вам нужно запустить React в производственном режиме, отключить [React Developer Tools](../learn/react-developer-tools.md) и использовать устройства, похожие на те, которые есть у пользователей вашего приложения, чтобы получить реальное представление о том, что на самом деле замедляет работу вашего приложения.

### Обновление состояния из мемоизированного обратного вызова

Иногда вам может потребоваться обновить состояние на основе предыдущего состояния из мемоизированного обратного вызова.

Эта функция `handleAddTodo` указывает `todos` как зависимость, потому что она вычисляет следующий todos из него:

<!-- 0045.part.md -->

```js
function TodoList() {
    const [todos, setTodos] = useState([]);

    const handleAddTodo = useCallback(
        (text) => {
            const newTodo = { id: nextId++, text };
            setTodos([...todos, newTodo]);
        },
        [todos]
    );
    // ...
}
```

<!-- 0046.part.md -->

Обычно вы хотите, чтобы мемоизированные функции имели как можно меньше зависимостей. Когда вы читаете некоторое состояние только для вычисления следующего состояния, вы можете устранить эту зависимость, передав вместо него функцию [updater](useState.md):

<!-- 0047.part.md -->

```js
function TodoList() {
    const [todos, setTodos] = useState([]);

    const handleAddTodo = useCallback((text) => {
        const newTodo = { id: nextId++, text };
        setTodos((todos) => [...todos, newTodo]);
    }, []); // ✅ No need for the todos dependency
    // ...
}
```

<!-- 0048.part.md -->

Здесь вместо того, чтобы сделать `todos` зависимостью и читать ее внутри, вы передаете в React инструкцию о том, _как_ обновить состояние (`todos => [...todos, newTodo]`). [Подробнее о функциях обновления](useState.md)

### Предотвращение слишком частого срабатывания эффекта

Иногда вы можете захотеть вызвать функцию внутри [Effect:](../learn/synchronizing-with-effects.md)

<!-- 0049.part.md -->

```js
function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');

    function createOptions() {
        return {
            serverUrl: 'https://localhost:1234',
            roomId: roomId,
        };
    }

    useEffect(() => {
        const options = createOptions();
        const connection = createConnection();
        connection.connect();
        // ...
    });
}
```

<!-- 0050.part.md -->

Это создает проблему. [Каждое реактивное значение должно быть объявлено зависимостью вашего Эффекта](../learn/lifecycle-of-reactive-effects.md) Однако, если вы объявите `createOptions` как зависимость, это заставит ваш Эффект постоянно переподключаться к чату:

<!-- 0051.part.md -->

```js
useEffect(() => {
    const options = createOptions();
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
}, [createOptions]); // 🔴 Problem: This dependency changes on every render
// ...
```

<!-- 0052.part.md -->

Чтобы решить эту проблему, вы можете обернуть функцию, которую нужно вызвать из Effect, в `useCallback`:

<!-- 0053.part.md -->

```js
function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');

    const createOptions = useCallback(() => {
        return {
            serverUrl: 'https://localhost:1234',
            roomId: roomId,
        };
    }, [roomId]); // ✅ Only changes when roomId changes

    useEffect(() => {
        const options = createOptions();
        const connection = createConnection();
        connection.connect();
        return () => connection.disconnect();
    }, [createOptions]); // ✅ Only changes when createOptions changes
    // ...
}
```

<!-- 0054.part.md -->

Это гарантирует, что функция `createOptions` будет одинаковой между повторными рендерингами, если `roomId` одинаков. **Однако, еще лучше устранить необходимость в зависимости от функции.** Переместите вашу функцию _внутрь_ Effect:

<!-- 0055.part.md -->

```js
function ChatRoom({ roomId }) {
    const [message, setMessage] = useState('');

    useEffect(() => {
        function createOptions() {
            // ✅ No need for useCallback or function dependencies!
            return {
                serverUrl: 'https://localhost:1234',
                roomId: roomId,
            };
        }

        const options = createOptions();
        const connection = createConnection();
        connection.connect();
        return () => connection.disconnect();
    }, [roomId]); // ✅ Only changes when roomId changes
    // ...
}
```

<!-- 0056.part.md -->

Теперь ваш код стал проще и не нуждается в `useCallback`. [Подробнее об удалении зависимостей от эффектов.](../learn/removing-effect-dependencies.md)

### Оптимизация пользовательского хука

Если вы пишете [пользовательский хук](../learn/reusing-logic-with-custom-hooks.md), рекомендуется обернуть все функции, которые он возвращает, в `useCallback`:

<!-- 0057.part.md -->

```js
function useRouter() {
    const { dispatch } = useContext(RouterStateContext);

    const navigate = useCallback(
        (url) => {
            dispatch({ type: 'navigate', url });
        },
        [dispatch]
    );

    const goBack = useCallback(() => {
        dispatch({ type: 'back' });
    }, [dispatch]);

    return {
        navigate,
        goBack,
    };
}
```

<!-- 0058.part.md -->

Это гарантирует, что потребители вашего Hook смогут при необходимости оптимизировать свой собственный код.

## Устранение неполадок

### Каждый раз, когда мой компонент рендерится, `useCallback` возвращает другую функцию

Убедитесь, что вы указали массив зависимостей в качестве второго аргумента!

Если вы забудете про массив зависимостей, `useCallback` будет возвращать каждый раз новую функцию:

<!-- 0059.part.md -->

```js
function ProductPage({ productId, referrer }) {
    const handleSubmit = useCallback((orderDetails) => {
        post('/product/' + productId + '/buy', {
            referrer,
            orderDetails,
        });
    }); // 🔴 Returns a new function every time: no dependency array
    // ...
}
```

<!-- 0060.part.md -->

Это исправленная версия, передающая массив зависимостей в качестве второго аргумента:

<!-- 0061.part.md -->

```js
function ProductPage({ productId, referrer }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // ✅ Does not return a new function unnecessarily
  // ...
```

<!-- 0062.part.md -->

Если это не помогло, то проблема в том, что по крайней мере одна из ваших зависимостей отличается от предыдущего рендера. Вы можете отладить эту проблему, вручную записав логи зависимостей в консоль:

<!-- 0063.part.md -->

```js
const handleSubmit = useCallback(
    (orderDetails) => {
        // ..
    },
    [productId, referrer]
);

console.log([productId, referrer]);
```

<!-- 0064.part.md -->

Затем вы можете щелкнуть правой кнопкой мыши на массивах из разных рендеров в консоли и выбрать "Store as a global variable" для обоих. Предположив, что первый массив был сохранен как `temp1`, а второй - как `temp2`, вы можете использовать консоль браузера, чтобы проверить, является ли каждая зависимость в обоих массивах одинаковой:

<!-- 0065.part.md -->

```js
Object.is(temp1[0], temp2[0]); // Is the first dependency the same between the arrays?
Object.is(temp1[1], temp2[1]); // Is the second dependency the same between the arrays?
Object.is(temp1[2], temp2[2]); // ... and so on for every dependency ...
```

<!-- 0066.part.md -->

Когда вы обнаружите, какая зависимость нарушает мемоизацию, либо найдите способ удалить ее, либо [мемоизируйте и ее](useMemo.md).

### Мне нужно вызвать `useCallback` для каждого элемента списка в цикле, но это не разрешено

Предположим, что компонент `Chart` обернут в [`memo`](memo.md). Вы хотите пропустить повторное отображение каждого `Chart` в списке при повторном отображении компонента `ReportList`. Однако вы не можете вызвать `useCallback` в цикле:

<!-- 0067.part.md -->

```js
function ReportList({ items }) {
    return (
        <article>
            {items.map((item) => {
                // 🔴 You can't call useCallback in a loop like this:
                const handleClick = useCallback(() => {
                    sendReport(item);
                }, [item]);

                return (
                    <figure key={item.id}>
                        <Chart onClick={handleClick} />
                    </figure>
                );
            })}
        </article>
    );
}
```

<!-- 0068.part.md -->

Вместо этого извлеките компонент для отдельного элемента и поместите туда `useCallback`:

<!-- 0069.part.md -->

```js
function ReportList({ items }) {
    return (
        <article>
            {items.map((item) => (
                <Report key={item.id} item={item} />
            ))}
        </article>
    );
}

function Report({ item }) {
    // ✅ Call useCallback at the top level:
    const handleClick = useCallback(() => {
        sendReport(item);
    }, [item]);

    return (
        <figure>
            <Chart onClick={handleClick} />
        </figure>
    );
}
```

<!-- 0070.part.md -->

В качестве альтернативы можно убрать `useCallback` в последнем фрагменте и вместо этого обернуть сам `Report` в [`memo`.](memo.md) Если параметр `item` не меняется, `Report` пропускает повторный рендеринг, поэтому `Chart` тоже пропускает повторный рендеринг:

<!-- 0071.part.md -->

```js
function ReportList({ items }) {
    // ...
}

const Report = memo(function Report({ item }) {
    function handleClick() {
        sendReport(item);
    }

    return (
        <figure>
            <Chart onClick={handleClick} />
        </figure>
    );
});
```

## Ссылки

-   [https://react.dev/reference/react/useCallback](https://react.dev/reference/react/useCallback)

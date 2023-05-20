# Возможно, вам не нужен эффект

Эффекты - это люк для выхода из парадигмы React. Они позволяют вам "выйти за пределы" React и синхронизировать ваши компоненты с какой-либо внешней системой, например, с не-React виджетом, сетью или DOM браузера. Если внешняя система не задействована (например, если вы хотите обновить состояние компонента при изменении некоторых реквизитов или состояния), вам не нужен Эффект. Удаление ненужных Эффектов сделает ваш код проще для понимания, быстрее для выполнения и менее подверженным ошибкам.

-   Зачем и как удалять ненужные Эффекты из ваших компонентов
-   Как кэшировать дорогостоящие вычисления без эффектов
-   Как сбрасывать и корректировать состояние компонента без Эффектов
-   Как разделить логику между обработчиками событий
-   Какую логику следует перенести в обработчики событий
-   Как уведомлять родительские компоненты об изменениях

## Как удалить ненужные эффекты {/_how-to-remove-unnecessary-effects_/}

Есть два распространенных случая, в которых вам не нужны Эффекты:

-   **Вам не нужны эффекты для преобразования данных для рендеринга.** Например, допустим, вы хотите отфильтровать список перед его отображением. У вас может возникнуть соблазн написать эффект, который обновляет переменную состояния при изменении списка. Однако это неэффективно. Когда вы обновляете состояние, React сначала вызовет функции вашего компонента, чтобы вычислить, что должно быть на экране. Затем React ["зафиксирует"](/learn/render-and-commit) эти изменения в DOM, обновляя экран. Затем React запустит ваши Эффекты. Если ваш Эффект _также_ немедленно обновляет состояние, это перезапускает весь процесс с нуля\! Чтобы избежать ненужных проходов рендеринга, преобразуйте все данные на верхнем уровне ваших компонентов. Этот код будет автоматически запускаться заново при каждом изменении реквизита или состояния.
-   **Вам не нужны Effects для обработки событий пользователя.** Например, допустим, вы хотите отправить POST-запрос `/api/buy` и показать уведомление, когда пользователь покупает товар. В обработчике события нажатия кнопки "Купить" вы точно знаете, что произошло. К моменту выполнения Effect вы не знаете, _что_ сделал пользователь (например, какая кнопка была нажата). Вот почему вы обычно обрабатываете события пользователя в соответствующих обработчиках событий.

Эффекты _действительно_ нужны для [синхронизации](/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events) с внешними системами. Например, вы можете написать эффект, который синхронизирует виджет jQuery с состоянием React. С помощью эффектов можно также получать данные: например, синхронизировать результаты поиска с текущим поисковым запросом. Имейте в виду, что современные [фреймворки](/learn/start-a-new-react-project#production-grade-react-frameworks) предоставляют более эффективные встроенные механизмы получения данных, чем написание Эффектов непосредственно в ваших компонентах.

Чтобы помочь вам обрести правильную интуицию, давайте рассмотрим несколько распространенных конкретных примеров\!

### Обновление состояния на основе реквизитов или состояния {/_updating-state-based-on-props-or-state_/}

Предположим, у вас есть компонент с двумя переменными состояния: `firstName` и `lastName`. Yo

<!-- 0001.part.md -->

<!-- 0002.part.md -->

```js
function Form() {
    const [firstName, setFirstName] = useState('Taylor');
    const [lastName, setLastName] = useState('Swift');

    // 🔴 Avoid: redundant state and unnecessary Effect
    const [fullName, setFullName] = useState('');
    useEffect(() => {
        setFullName(firstName + ' ' + lastName);
    }, [firstName, lastName]);
    // ...
}
```

<!-- 0003.part.md -->

Это сложнее, чем необходимо. Кроме того, это неэффективно: он выполняет целый проход рендеринга с устаревшим значением для `fullName`, а затем сразу же выполняет повторный рендеринг с обновленным значением. Удалите переменную state и Effect:

<!-- 0004.part.md -->

```js
function Form() {
    const [firstName, setFirstName] = useState('Taylor');
    const [lastName, setLastName] = useState('Swift');
    // ✅ Good: calculated during rendering
    const fullName = firstName + ' ' + lastName;
    // ...
}
```

<!-- 0005.part.md -->

**Если что-то можно вычислить из существующих реквизитов или состояния, [не помещайте это в состояние](/learn/choosing-the-state-structure#avoid-redundant-state) Вместо этого вычисляйте это во время рендеринга.** Это сделает ваш код быстрее (вы избежите лишних "каскадных" обновлений), проще (вы удалите часть кода) и менее подверженным ошибкам (вы избежите ошибок, вызванных рассинхронизацией различных переменных состояния друг с другом). Если такой подход кажется вам новым, в [Thinking in React](/learn/thinking-in-react#step-3-find-the-minimal-but-complete-representation-of-ui-state) объясняется, что должно входить в состояние.

### Кэширование дорогих вычислений {/_caching-expensive-calculations_/}

Этот компонент вычисляет `visibleTodos`, принимая `todos`, которые он получает по реквизитам, и фильтруя их в соответствии с реквизитом `filter`. У вас может возникнуть желание хранить результат в состоянии и обновлять его из Effect:

<!-- 0006.part.md -->

```js
function TodoList({ todos, filter }) {
    const [newTodo, setNewTodo] = useState('');

    // 🔴 Avoid: redundant state and unnecessary Effect
    const [visibleTodos, setVisibleTodos] = useState([]);
    useEffect(() => {
        setVisibleTodos(getFilteredTodos(todos, filter));
    }, [todos, filter]);

    // ...
}
```

<!-- 0007.part.md -->

Как и в предыдущем примере, это и ненужно, и неэффективно. Сначала удалите состояние и Эффект:

<!-- 0008.part.md -->

```js
function TodoList({ todos, filter }) {
    const [newTodo, setNewTodo] = useState('');
    // ✅ This is fine if getFilteredTodos() is not slow.
    const visibleTodos = getFilteredTodos(todos, filter);
    // ...
}
```

<!-- 0009.part.md -->

Обычно этот код является fine\! Но, возможно, `getFilteredTodos()` работает медленно или у вас много `todos`. В таком случае вы не хотите пересчитывать `getFilteredTodos()`, если изменилась какая-то несвязанная переменная состояния, например `newTodo`.

Вы можете кэшировать (или ["memoize"](https://en.wikipedia.org/wiki/Memoization)) дорогостоящее вычисление, обернув его в хук [`useMemo`](/reference/react/useMemo):

<!-- 0010.part.md -->

```js
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
    const [newTodo, setNewTodo] = useState('');
    const visibleTodos = useMemo(() => {
        // ✅ Does not re-run unless todos or filter change
        return getFilteredTodos(todos, filter);
    }, [todos, filter]);
    // ...
}
```

<!-- 0011.part.md -->

Или написанные в одну строку:

<!-- 0012.part.md -->

```js
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
    const [newTodo, setNewTodo] = useState('');
    // ✅ Does not re-run getFilteredTodos() unless todos or filter change
    const visibleTodos = useMemo(
        () => getFilteredTodos(todos, filter),
        [todos, filter]
    );
    // ...
}
```

<!-- 0013.part.md -->

**Это говорит React, что вы не хотите, чтобы внутренняя функция запускалась повторно, если `todos` или `filter` не изменились.** React будет помнить возвращаемое значение `getFilteredTodos()` во время первоначального рендеринга. При последующих рендерах он будет проверять, не изменились ли `todos` или `filter`. Если они те же, что и в прошлый раз, `useMemo` вернет последний сохраненный результат. Если же они отличаются, React снова вызовет внутреннюю функцию (и сохранит ее результат).

Функция, которую вы обернули в [`useMemo`](/reference/react/useMemo), запускается во время рендеринга, поэтому это работает только для [чистых вычислений](/learn/keeping-components-pure).

#### Как определить, является ли вычисление дорогим? {/_how-to-tell-if-a-calculation-is-expensive_/}

В общем, если вы не создаете тысячи объектов и не выполняете циклы, то, скорее всего, это не дорого. Если вы хотите получить больше уверенности, вы можете добавить консольный журнал, чтобы измерить время, затраченное на часть кода:

<!-- 0014.part.md -->

```js
console.time('filter array');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('filter array');
```

<!-- 0015.part.md -->

Выполните измеряемое действие (например, введите текст в input). После этого вы увидите в консоли журналы типа `фильтр массива: 0.15ms` в вашей консоли. Если общее время, записанное в журнал, составляет значительную величину (скажем, `1 мс` или больше), возможно, имеет смысл запомнить этот расчет. В качестве эксперимента вы можете обернуть расчет в `useMemo`, чтобы проверить, уменьшилось ли общее время регистрации для данного взаимодействия или нет:

<!-- 0016.part.md -->

```js
console.time('filter array');
const visibleTodos = useMemo(() => {
    return getFilteredTodos(todos, filter); // Skipped if todos and filter haven't changed
}, [todos, filter]);
console.timeEnd('filter array');
```

<!-- 0017.part.md -->

`useMemo` не сделает _первый_ рендеринг быстрее. Это только поможет вам пропустить ненужную работу над обновлениями.

Имейте в виду, что ваша машина, скорее всего, быстрее, чем у ваших пользователей, поэтому хорошей идеей будет проверить производительность с помощью искусственного замедления. Например, Chrome предлагает для этого опцию [CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling).

Также обратите внимание, что измерение производительности в процессе разработки не даст вам наиболее точных результатов. (Например, если включен [Строгий режим](/reference/react/StrictMode), каждый компонент будет отображаться дважды, а не один раз). Чтобы получить наиболее точные результаты, создайте приложение для производства и протестируйте его на устройстве, которое есть у ваших пользователей.

### Сброс всех состояний при изменении реквизита {/_resetting-all-state-when-a-prop-changes_/}

Этот компонент `ProfilePage` получает реквизит `userId`. Страница содержит входной комментарий, и вы используете переменную состояния `comment` для хранения его значения. Однажды вы заметили проблему: когда вы переходите от одного профиля к другому, состояние `comment` не сбрасывается. В результате легко случайно опубликовать комментарий в профиле не того пользователя. Чтобы решить эту проблему, вы хотите очищать переменную состояния `comment` при каждом изменении `userId`:

<!-- 0018.part.md -->

```js
export default function ProfilePage({ userId }) {
    const [comment, setComment] = useState('');

    // 🔴 Avoid: Resetting state on prop change in an Effect
    useEffect(() => {
        setComment('');
    }, [userId]);
    // ...
}
```

<!-- 0019.part.md -->

Это неэффективно, потому что `ProfilePage` и его дочерние компоненты будут сначала отображаться с устаревшим значением, а затем снова отображаться. Это также сложно, потому что вам придется делать это в _каждом_ компоненте, который имеет какое-то состояние внутри `ProfilePage`. Например, если пользовательский интерфейс комментариев является вложенным, вы захотите очистить и состояние вложенных комментариев.

Вместо этого вы можете сказать React, что профиль каждого пользователя концептуально является _отдельным_ профилем, задав ему явный ключ. Разделите ваш компонент на две части и передайте атрибут `key` от внешнего компонента к внутреннему:

<!-- 0020.part.md -->

```js
export default function ProfilePage({ userId }) {
    return <Profile userId={userId} key={userId} />;
}

function Profile({ userId }) {
    // ✅ This and any other state below will reset on key change automatically
    const [comment, setComment] = useState('');
    // ...
}
```

<!-- 0021.part.md -->

Обычно React сохраняет состояние, когда один и тот же компонент отображается в одном и том же месте. **Передавая `userId` в качестве `ключа` компоненту `Profile`, вы просите React рассматривать два компонента `Profile` с разными `userId` как два разных компонента, которые не должны иметь общего состояния.** Каждый раз, когда ключ (который вы установили в `userId`) меняется, React будет воссоздавать DOM и [сбрасывать состояние](/learn/preserving-and-resetting-state#option-2-resetting-state-with-a-key) компонента `Profile` и всех его дочерних компонентов. Теперь поле `комментарий` будет автоматически очищаться при переходе между профилями.

Обратите внимание, что в этом примере только внешний компонент `ProfilePage` экспортируется и виден другим файлам в проекте. Компонентам, рендерящим `ProfilePage`, не нужно передавать ключ: они передают `userId` как обычный реквизит. Тот факт, что `ProfilePage` передает его как `ключ` внутреннему компоненту `Profile`, является деталью реализации.

### Настройка состояния при изменении реквизита {/_adjusting-some-state-when-a-prop-changes_/}

Иногда при изменении реквизита вы можете захотеть сбросить или скорректировать часть состояния, но не все.

Этот компонент `List` получает список `элементов` в качестве реквизита и сохраняет выбранный элемент в переменной состояния `selection`. Вы хотите сбрасывать `selection` в `null` всякий раз, когда реквизит `items` получает другой массив:

<!-- 0022.part.md -->

```js
function List({ items }) {
    const [isReverse, setIsReverse] = useState(false);
    const [selection, setSelection] = useState(null);

    // 🔴 Avoid: Adjusting state on prop change in an Effect
    useEffect(() => {
        setSelection(null);
    }, [items]);
    // ...
}
```

<!-- 0023.part.md -->

Это тоже не идеально. Каждый раз, когда `items` меняется, `List` и его дочерние компоненты будут сначала отображаться с устаревшим значением `selection`. Затем React обновит DOM и запустит Effects. Наконец, вызов `setSelection(null)` приведет к повторному рендерингу `List` и его дочерних компонентов, и весь процесс начнется заново.

Начните с удаления эффекта. Вместо этого измените состояние непосредственно во время рендеринга:

<!-- 0024.part.md -->

```js
function List({ items }) {
    const [isReverse, setIsReverse] = useState(false);
    const [selection, setSelection] = useState(null);

    // Better: Adjust the state while rendering
    const [prevItems, setPrevItems] = useState(items);
    if (items !== prevItems) {
        setPrevItems(items);
        setSelection(null);
    }
    // ...
}
```

<!-- 0025.part.md -->

[Storing information from previous renders](/reference/react/useState#storing-information-from-previous-renders) подобное может быть трудно понять, но это лучше, чем обновление того же состояния в Effect. В приведенном выше примере `setSelection` вызывается непосредственно во время рендеринга. React повторно отобразит `List` _сразу же_ после выхода из него с оператором `return`. React еще не рендерил дочерние элементы `List` и не обновлял DOM, поэтому это позволяет дочерним элементам `List` пропустить рендеринг устаревшего значения `selection`.

Когда вы обновляете компонент во время рендеринга, React выбрасывает возвращенный JSX и немедленно повторяет рендеринг. Чтобы избежать очень медленных каскадных повторов, React позволяет обновлять состояние только _одного_ компонента во время рендеринга. Если вы обновите состояние другого компонента во время рендеринга, вы увидите ошибку. Условие типа `items !== prevItems` необходимо, чтобы избежать циклов. Вы можете корректировать состояние подобным образом, но любые другие побочные эффекты (например, изменение DOM или установка таймаутов) должны оставаться в обработчиках событий или Effects для [сохранения чистоты компонентов](/learn/keeping-components-pure).

**Хотя этот паттерн более эффективен, чем Effect, большинству компонентов он также не нужен.** Независимо от того, как вы это делаете, изменение состояния на основе реквизитов или других состояний делает ваш поток данных более сложным для понимания и отладки. Всегда проверяйте, можно ли вместо этого [сбросить все состояние с помощью ключа] (#resetting-all-state-when-a-prop-changes) или [вычислить все во время рендеринга] (#updating-state-based-on-props-or-state). Например, вместо хранения (и сброса) выбранного _элемента_, вы можете хранить выбранный _идентификатор элемента:_.

<!-- 0026.part.md -->

```js
function List({ items }) {
    const [isReverse, setIsReverse] = useState(false);
    const [selectedId, setSelectedId] = useState(null);
    // ✅ Best: Calculate everything during rendering
    const selection =
        items.find((item) => item.id === selectedId) ??
        null;
    // ...
}
```

<!-- 0027.part.md -->

Теперь нет необходимости "корректировать" состояние вообще. Если элемент с выбранным ID находится в списке, он остается выбранным. Если нет, то `выборка`, вычисленная при рендеринге, будет равна `null`, потому что не было найдено ни одного подходящего элемента. Это поведение отличается, но, возможно, лучше, потому что большинство изменений `items` сохраняют выбор.

### Совместное использование логики между обработчиками событий {/_sharing-logic-between-event-handlers_/}

Допустим, у вас есть страница продукта с двумя кнопками (Купить и Оформить заказ), которые позволяют вам купить этот продукт. Вы хотите показывать уведомление каждый раз, когда пользователь кладет товар в корзину. Вызов `showNotification()` в обработчиках нажатия обеих кнопок кажется повторяющимся, поэтому у вас может возникнуть соблазн поместить эту логику в Effect:

<!-- 0028.part.md -->

```js
function ProductPage({ product, addToCart }) {
    // 🔴 Avoid: Event-specific logic inside an Effect
    useEffect(() => {
        if (product.isInCart) {
            showNotification(
                `Added ${product.name} to the shopping cart!`
            );
        }
    }, [product]);

    function handleBuyClick() {
        addToCart(product);
    }

    function handleCheckoutClick() {
        addToCart(product);
        navigateTo('/checkout');
    }
    // ...
}
```

<!-- 0029.part.md -->

Этот эффект не нужен. Кроме того, он, скорее всего, приведет к ошибкам. Например, предположим, что ваше приложение "запоминает" корзину между перезагрузками страницы. Если вы один раз добавите товар в корзину и обновите страницу, уведомление появится снова. Оно будет появляться каждый раз, когда вы обновляете страницу этого товара. Это происходит потому, что `product.isInCart` уже будет `true` при загрузке страницы, поэтому Эффект выше вызовет `showNotification()`.

**Когда вы не уверены, должен ли какой-то код находиться в Эффекте или в обработчике событий, спросите себя, _почему_ этот код должен выполняться. Используйте эффекты только для кода, который должен выполняться _по причине_ отображения компонента пользователю.** В данном примере уведомление должно появиться потому, что пользователь _нажал на кнопку_, а не потому, что страница была отображена\! Удалите эффект и поместите общую логику в функцию, вызываемую из обоих обработчиков событий:

<!-- 0030.part.md -->

```js
function ProductPage({ product, addToCart }) {
    // ✅ Good: Event-specific logic is called from event handlers
    function buyProduct() {
        addToCart(product);
        showNotification(
            `Added ${product.name} to the shopping cart!`
        );
    }

    function handleBuyClick() {
        buyProduct();
    }

    function handleCheckoutClick() {
        buyProduct();
        navigateTo('/checkout');
    }
    // ...
}
```

<!-- 0031.part.md -->

Это и убирает ненужный Effect, и исправляет ошибку.

### Отправка POST-запроса {/_sending-a-post-request_/}

Этот компонент `Form` отправляет два вида POST-запросов. При монтировании он посылает событие аналитики. Когда вы заполняете форму и нажимаете кнопку Submit, он отправляет POST-запрос на конечную точку `/api/register`:

<!-- 0032.part.md -->

```js
function Form() {
    const [firstName, setFirstName] = useState('');
    const [lastName, setLastName] = useState('');

    // ✅ Good: This logic should run because the component was displayed
    useEffect(() => {
        post('/analytics/event', {
            eventName: 'visit_form',
        });
    }, []);

    // 🔴 Avoid: Event-specific logic inside an Effect
    const [jsonToSubmit, setJsonToSubmit] = useState(null);
    useEffect(() => {
        if (jsonToSubmit !== null) {
            post('/api/register', jsonToSubmit);
        }
    }, [jsonToSubmit]);

    function handleSubmit(e) {
        e.preventDefault();
        setJsonToSubmit({ firstName, lastName });
    }
    // ...
}
```

<!-- 0033.part.md -->

Применим те же критерии, что и в предыдущем примере.

POST-запрос аналитики должен оставаться в Эффекте. Это потому, что _причина_ отправки события аналитики заключается в том, что форма была отображена. (При разработке оно сработает дважды, но [см. здесь](/learn/synchronizing-with-effects#sending-analytics) о том, как с этим справиться).

Однако POST-запрос `/api/register` не вызван тем, что форма _отображается_. Вы хотите отправить запрос только в один конкретный момент времени: когда пользователь нажимает кнопку. Это должно происходить только _при этом конкретном взаимодействии_. Удалите второй Effect и переместите POST-запрос в обработчик события:

<!-- 0034.part.md -->

```js
function Form() {
    const [firstName, setFirstName] = useState('');
    const [lastName, setLastName] = useState('');

    // ✅ Good: This logic runs because the component was displayed
    useEffect(() => {
        post('/analytics/event', {
            eventName: 'visit_form',
        });
    }, []);

    function handleSubmit(e) {
        e.preventDefault();
        // ✅ Good: Event-specific logic is in the event handler
        post('/api/register', { firstName, lastName });
    }
    // ...
}
```

<!-- 0035.part.md -->

Когда вы выбираете, куда поместить логику - в обработчик событий или в Effect, главный вопрос, на который вам нужно ответить, - это _какого рода логика_ с точки зрения пользователя. Если эта логика вызвана определенным взаимодействием, поместите ее в обработчик события. Если она вызвана тем, что пользователь _видит_ компонент на экране, сохраните ее в Effect.

### Цепочки вычислений {/_chains-of-computations_/}

Иногда у вас может возникнуть соблазн выстроить цепочку Эффектов, каждый из которых корректирует часть состояния на основе другого состояния:

<!-- 0036.part.md -->

```js
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 Avoid: Chains of Effects that adjust the state solely to trigger each other
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

<!-- 0037.part.md -->

В этом коде есть две проблемы.

Одна проблема заключается в том, что он очень неэффективен: компоненту (и его дочерним элементам) приходится заново выполнять рендеринг между каждым вызовом `set` в цепочке. В примере выше, в худшем случае (`setCard` → render → `setGoldCardCount` → render → `setRound` → render → `setIsGameOver` → render) происходит три ненужных повторных рендеринга дерева ниже.

Даже если бы это не было медленно, по мере развития вашего кода вы столкнетесь со случаями, когда написанная вами "цепочка" не будет соответствовать новым требованиям. Представьте, что вы добавляете способ просмотреть историю ходов игры. Вы сделаете это, обновив каждую переменную состояния на значение из прошлого. Однако установка состояния `card` в значение из прошлого снова запустит цепочку Effect и изменит данные, которые вы показываете. Такой код часто бывает жестким и хрупким.

В этом случае лучше вычислить все, что можно, во время рендеринга, а состояние изменить в обработчике событий:

<!-- 0038.part.md -->

```js
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ Calculate what you can during rendering
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calculate all the next state in the event handler
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```

<!-- 0039.part.md -->

Это намного эффективнее. Кроме того, если вы реализуете способ просмотра истории игры, теперь вы сможете установить каждую переменную состояния на ход из прошлого, не запуская цепочку Effect, которая корректирует все остальные значения. Если вам нужно повторно использовать логику между несколькими обработчиками событий, вы можете [извлечь функцию](#sharing-logic-between-event-handlers) и вызвать ее из этих обработчиков.

Помните, что внутри обработчиков событий [состояние ведет себя как снимок](/learn/state-as-a-snapshot). Например, даже после вызова `setRound(round + 1)`, переменная `round` будет отражать значение на момент нажатия пользователем кнопки. Если вам нужно использовать следующее значение для вычислений, определите его вручную, например, `const nextRound = round + 1`.

В некоторых случаях вы _не можете_ вычислить следующее состояние непосредственно в обработчике события. Например, представьте форму с несколькими выпадающими списками, где опции следующего выпадающего списка зависят от выбранного значения предыдущего выпадающего списка. Тогда цепочка эффектов будет уместна, поскольку вы синхронизируетесь с сетью.

### Инициализация приложения {/_initializing-the-application_/}

Некоторая логика должна выполняться только один раз при загрузке приложения.

У вас может возникнуть соблазн поместить ее в Эффект в компоненте верхнего уровня:

<!-- 0040.part.md -->

```js
function App() {
    // 🔴 Avoid: Effects with logic that should only ever run once
    useEffect(() => {
        loadDataFromLocalStorage();
        checkAuthToken();
    }, []);
    // ...
}
```

<!-- 0041.part.md -->

Однако вы быстро обнаружите, что он [запускается дважды в разработке](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development) Это может вызвать проблемы - например, он может аннулировать токен аутентификации, поскольку функция не была рассчитана на двойной вызов. В целом, ваши компоненты должны быть устойчивы к перемонтированию. Это относится и к компоненту верхнего уровня `App`.

Хотя на практике в производстве его, возможно, никогда не будут перемонтировать, соблюдение одинаковых ограничений во всех компонентах облегчает перемещение и повторное использование кода. Если какая-то логика должна выполняться _один раз при загрузке приложения_, а не _один раз при монтировании компонента_, добавьте переменную верхнего уровня для отслеживания того, была ли она уже выполнена:

<!-- 0042.part.md -->

```js
let didInit = false;

function App() {
    useEffect(() => {
        if (!didInit) {
            didInit = true;
            // ✅ Only runs once per app load
            loadDataFromLocalStorage();
            checkAuthToken();
        }
    }, []);
    // ...
}
```

<!-- 0043.part.md -->

Вы также можете запустить его во время инициализации модуля и перед рендерингом приложения:

<!-- 0044.part.md -->

```js
if (typeof window !== 'undefined') {
    // Check if we're running in the browser.
    // ✅ Only runs once per app load
    checkAuthToken();
    loadDataFromLocalStorage();
}

function App() {
    // ...
}
```

<!-- 0045.part.md -->

Код на верхнем уровне выполняется один раз при импорте компонента - даже если он не будет отображаться. Чтобы избежать замедления работы или неожиданного поведения при импорте произвольных компонентов, не злоупотребляйте этим шаблоном. Оставьте логику инициализации для всего приложения в корневых модулях компонентов, таких как `App.js` или в точке входа вашего приложения.

### Уведомление родительских компонентов об изменениях состояния {/_notifying-parent-components-about-state-changes_/}

Допустим, вы пишете компонент `Toggle` с внутренним состоянием `isOn`, которое может быть либо `true`, либо `false`. Есть несколько различных способов переключить его (щелчком или перетаскиванием). Вы хотите уведомлять родительский компонент каждый раз, когда внутреннее состояние `Toggle` меняется, поэтому вы выставляете событие `onChange` и вызываете его из Effect:

<!-- 0046.part.md -->

```js
function Toggle({ onChange }) {
    const [isOn, setIsOn] = useState(false);

    // 🔴 Avoid: The onChange handler runs too late
    useEffect(() => {
        onChange(isOn);
    }, [isOn, onChange]);

    function handleClick() {
        setIsOn(!isOn);
    }

    function handleDragEnd(e) {
        if (isCloserToRightEdge(e)) {
            setIsOn(true);
        } else {
            setIsOn(false);
        }
    }

    // ...
}
```

<!-- 0047.part.md -->

Как и в предыдущем случае, это не идеально. Сначала `Toggle` обновляет свое состояние, а React обновляет экран. Затем React запускает Effect, который вызывает функцию `onChange`, переданную от родительского компонента. Теперь родительский компонент обновит свое собственное состояние, запуская еще один проход рендеринга. Было бы лучше сделать все за один проход.

Удалите Effect и вместо этого обновите состояние _обоих_ компонентов в одном обработчике событий:

<!-- 0048.part.md -->

```js
function Toggle({ onChange }) {
    const [isOn, setIsOn] = useState(false);

    function updateToggle(nextIsOn) {
        // ✅ Good: Perform all updates during the event that caused them
        setIsOn(nextIsOn);
        onChange(nextIsOn);
    }

    function handleClick() {
        updateToggle(!isOn);
    }

    function handleDragEnd(e) {
        if (isCloserToRightEdge(e)) {
            updateToggle(true);
        } else {
            updateToggle(false);
        }
    }

    // ...
}
```

<!-- 0049.part.md -->

При таком подходе и компонент `Toggle`, и его родительский компонент обновляют свое состояние во время события. React [batches updates](/learn/queueing-a-series-of-state-updates) от разных компонентов вместе, так что будет только один проход рендеринга.

Возможно, вы также сможете полностью убрать состояние и вместо этого получать `isOn` от родительского компонента:

<!-- 0050.part.md -->

```js
// ✅ Also good: the component is fully controlled by its parent
function Toggle({ isOn, onChange }) {
    function handleClick() {
        onChange(!isOn);
    }

    function handleDragEnd(e) {
        if (isCloserToRightEdge(e)) {
            onChange(true);
        } else {
            onChange(false);
        }
    }

    // ...
}
```

<!-- 0051.part.md -->

["Lifting state up"](/learn/sharing-state-between-components) позволяет родительскому компоненту полностью контролировать `Toggle`, переключая собственное состояние родительского компонента. Это означает, что родительский компонент будет содержать больше логики, но в целом будет меньше состояния, о котором нужно беспокоиться. Всякий раз, когда вы пытаетесь синхронизировать две разные переменные состояния, попробуйте поднять состояние вверх\!

### Передача данных родителю {/_passing-data-to-the-parent_/}

Этот компонент `Child` получает некоторые данные и затем передает их компоненту `Parent` в Effect:

<!-- 0052.part.md -->

```js
function Parent() {
    const [data, setData] = useState(null);
    // ...
    return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
    const data = useSomeAPI();
    // 🔴 Avoid: Passing data to the parent in an Effect
    useEffect(() => {
        if (data) {
            onFetched(data);
        }
    }, [onFetched, data]);
    // ...
}
```

<!-- 0053.part.md -->

В React данные передаются от родительских компонентов к их дочерним компонентам. Когда вы видите на экране что-то неправильное, вы можете проследить, откуда пришла информация, поднимаясь по цепочке компонентов, пока не найдете, какой компонент передает неправильный реквизит или имеет неправильное состояние. Когда дочерние компоненты обновляют состояние своих родительских компонентов в Effects, отследить поток данных становится очень сложно. Поскольку и дочернему, и родительскому компоненту нужны одни и те же данные, позвольте родительскому компоненту получить эти данные и _передать их вниз_ дочернему компоненту:

<!-- 0054.part.md -->

```js
function Parent() {
    const data = useSomeAPI();
    // ...
    // ✅ Good: Passing data down to the child
    return <Child data={data} />;
}

function Child({ data }) {
    // ...
}
```

<!-- 0055.part.md -->

Это проще и сохраняет предсказуемость потока данных: данные идут вниз от родителя к ребенку.

### Подписка на внешний магазин {/_subscribing-to-an-external-store_/}

Иногда вашим компонентам может понадобиться подписаться на данные, находящиеся за пределами состояния React. Эти данные могут быть получены из сторонней библиотеки или встроенного API браузера. Поскольку эти данные могут меняться без ведома React, вам необходимо вручную подписать на них свои компоненты. Это часто делается, например, с помощью Effect:

<!-- 0056.part.md -->

```js
function useOnlineStatus() {
    // Not ideal: Manual store subscription in an Effect
    const [isOnline, setIsOnline] = useState(true);
    useEffect(() => {
        function updateState() {
            setIsOnline(navigator.onLine);
        }

        updateState();

        window.addEventListener('online', updateState);
        window.addEventListener('offline', updateState);
        return () => {
            window.removeEventListener(
                'online',
                updateState
            );
            window.removeEventListener(
                'offline',
                updateState
            );
        };
    }, []);
    return isOnline;
}

function ChatIndicator() {
    const isOnline = useOnlineStatus();
    // ...
}
```

<!-- 0057.part.md -->

Здесь компонент подписывается на внешнее хранилище данных (в данном случае API браузера `navigator.onLine`). Поскольку этот API не существует на сервере (поэтому он не может быть использован для исходного HTML), первоначально состояние устанавливается в `true`. Каждый раз, когда значение этого хранилища данных изменяется в браузере, компонент обновляет свое состояние.

Хотя обычно для этого используются эффекты, в React есть специально созданный Hook для подписки на внешнее хранилище, который предпочтительнее использовать. Удалите эффект и замените его вызовом [`useSyncExternalStore`](/reference/react/useSyncExternalStore):

<!-- 0058.part.md -->

```js
function subscribe(callback) {
    window.addEventListener('online', callback);
    window.addEventListener('offline', callback);
    return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
    };
}

function useOnlineStatus() {
    // ✅ Good: Subscribing to an external store with a built-in Hook
    return useSyncExternalStore(
        subscribe, // React won't resubscribe for as long as you pass the same function
        () => navigator.onLine, // How to get the value on the client
        () => true // How to get the value on the server
    );
}

function ChatIndicator() {
    const isOnline = useOnlineStatus();
    // ...
}
```

<!-- 0059.part.md -->

Этот подход менее подвержен ошибкам, чем ручная синхронизация изменяемых данных с состоянием React с помощью Effect. Обычно вы пишете пользовательский хук, подобный `useOnlineStatus()` выше, чтобы вам не нужно было повторять этот код в отдельных компонентах. [Подробнее о подписке на внешние хранилища из компонентов React.](/reference/react/useSyncExternalStore)

### Получение данных {/_fetching-data_/}

Многие приложения используют Effects для запуска получения данных. Довольно часто Эффект для получения данных пишется следующим образом:

<!-- 0060.part.md -->

```js
function SearchResults({ query }) {
    const [results, setResults] = useState([]);
    const [page, setPage] = useState(1);

    useEffect(() => {
        // 🔴 Avoid: Fetching without cleanup logic
        fetchResults(query, page).then((json) => {
            setResults(json);
        });
    }, [query, page]);

    function handleNextPageClick() {
        setPage(page + 1);
    }
    // ...
}
```

<!-- 0061.part.md -->

Вам _не нужно_ переносить эту выборку в обработчик событий.

Это может показаться противоречием с предыдущими примерами, где вам нужно было поместить логику в обработчики событий\! Однако подумайте о том, что не _событие ввода_ является основной причиной для выборки. Входы поиска часто заполняются из URL, и пользователь может перемещаться назад и вперед, не касаясь входа.

Не имеет значения, откуда берутся `page` и `query`. Пока этот компонент виден, вы хотите сохранить `результаты` [синхронизированными](/learn/synchronizing-with-effects) с данными из сети для текущей `страницы` и `запроса`. Вот почему это Эффект.

Однако в приведенном выше коде есть ошибка. Представьте, что вы быстро набираете `"hello"`. Тогда `запрос` изменится с `"h"` на `"he"`, `"hel"`, `"hell"` и `"hello"`. Это приведет к запуску отдельных запросов, но нет никакой гарантии, в каком порядке будут получены ответы. Например, ответ `"hell"` может прийти _после_ ответа `"hello"`. Поскольку вызов `setResults()` будет последним, вы отобразите неправильные результаты поиска. Это называется ["условие гонки"](https://en.wikipedia.org/wiki/Race_condition): два разных запроса "мчатся" друг за другом и приходят в другом порядке, чем вы ожидали.

**Чтобы исправить состояние гонки, вам нужно [добавить функцию очистки](/learn/synchronizing-with-effects#fetching-data) для игнорирования несвежих ответов:**.

<!-- 0062.part.md -->

```js
function SearchResults({ query }) {
    const [results, setResults] = useState([]);
    const [page, setPage] = useState(1);
    useEffect(() => {
        let ignore = false;
        fetchResults(query, page).then((json) => {
            if (!ignore) {
                setResults(json);
            }
        });
        return () => {
            ignore = true;
        };
    }, [query, page]);

    function handleNextPageClick() {
        setPage(page + 1);
    }
    // ...
}
```

<!-- 0063.part.md -->

Это гарантирует, что при выборке данных вашим Effect все ответы, кроме последнего запрошенного, будут проигнорированы.

Обработка условий гонки - не единственная сложность при реализации выборки данных. Вам также может понадобиться подумать о кэшировании ответов (чтобы пользователь мог нажать кнопку Back и мгновенно увидеть предыдущий экран), о том, как получать данные на сервере (чтобы первоначальный HTML, отрисованный на сервере, содержал полученное содержимое вместо спиннера), и о том, как избежать сетевых водопадов (чтобы дочерний элемент мог получать данные, не дожидаясь каждого родительского элемента).

**Эти проблемы относятся к любой библиотеке пользовательского интерфейса, не только к React. Их решение не является тривиальным, поэтому современные [фреймворки](/learn/start-a-new-react-project#production-grade-react-frameworks) предоставляют более эффективные встроенные механизмы выборки данных, чем выборка данных в Effects.**.

Если вы не используете фреймворк (и не хотите создавать свой собственный), но хотите сделать получение данных из Effects более эргономичным, рассмотрите возможность извлечения логики получения данных в пользовательский Hook, как в этом примере:

<!-- 0064.part.md -->

```js
function SearchResults({ query }) {
    const [page, setPage] = useState(1);
    const params = new URLSearchParams({ query, page });
    const results = useData(`/api/search?${params}`);

    function handleNextPageClick() {
        setPage(page + 1);
    }
    // ...
}

function useData(url) {
    const [data, setData] = useState(null);
    useEffect(() => {
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
    }, [url]);
    return data;
}
```

<!-- 0065.part.md -->

Скорее всего, вы также захотите добавить некоторую логику для обработки ошибок и отслеживания загрузки содержимого. Вы можете создать подобный Hook самостоятельно или использовать одно из многочисленных решений, уже доступных в экосистеме React. **Несмотря на то, что этот способ не будет столь же эффективным, как использование встроенного во фреймворк механизма получения данных, перенос логики получения данных в пользовательский хук облегчит принятие эффективной стратегии получения данных в дальнейшем**.

В общем, всякий раз, когда вам приходится прибегать к написанию Эффектов, следите за тем, когда вы можете извлечь часть функциональности в пользовательский Хук с более декларативным и специально разработанным API, как `useData` выше. Чем меньше необработанных вызовов `useEffect` будет в ваших компонентах, тем легче вам будет поддерживать ваше приложение.

\<Recap\>

-   Если вы можете вычислить что-то во время рендеринга, вам не нужен Эффект.
-   Чтобы кэшировать дорогостоящие вычисления, добавьте `useMemo` вместо `useEffect`.
-   Чтобы сбросить состояние всего дерева компонентов, передайте ему другой `key`.
-   Чтобы сбросить определенный бит состояния в ответ на изменение реквизита, установите его во время рендеринга.
-   Код, который выполняется потому, что компонент был _отображен_, должен быть в Effects, остальное - в событиях.
-   Если вам нужно обновить состояние нескольких компонентов, лучше сделать это во время одного события.
-   Всякий раз, когда вы пытаетесь синхронизировать переменные состояния в разных компонентах, подумайте о поднятии состояния вверх.
-   Вы можете получать данные с помощью Effects, но вам необходимо реализовать очистку, чтобы избежать условий гонки.

\</Recap\>

\<Проблемы\>

#### Преобразование данных без эффектов {/_transform-data-without-effects_/}

Приведенный ниже `TodoList` отображает список дел. Когда установлен флажок "Показывать только активные задания", завершенные задания не отображаются в списке. Независимо от того, какие из них видны, в нижнем колонтитуле отображается количество еще не завершенных дел.

Упростите этот компонент, удалив все ненужные состояния и эффекты.

<!-- 0066.part.md -->

```js
import { useState, useEffect } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
    const [todos, setTodos] = useState(initialTodos);
    const [showActive, setShowActive] = useState(false);
    const [activeTodos, setActiveTodos] = useState([]);
    const [visibleTodos, setVisibleTodos] = useState([]);
    const [footer, setFooter] = useState(null);

    useEffect(() => {
        setActiveTodos(
            todos.filter((todo) => !todo.completed)
        );
    }, [todos]);

    useEffect(() => {
        setVisibleTodos(showActive ? activeTodos : todos);
    }, [showActive, todos, activeTodos]);

    useEffect(() => {
        setFooter(
            <footer>{activeTodos.length} todos left</footer>
        );
    }, [activeTodos]);

    return (
        <>
            <label>
                <input
                    type="checkbox"
                    checked={showActive}
                    onChange={(e) =>
                        setShowActive(e.target.checked)
                    }
                />
                Show only active todos
            </label>
            <NewTodo
                onAdd={(newTodo) =>
                    setTodos([...todos, newTodo])
                }
            />
            <ul>
                {visibleTodos.map((todo) => (
                    <li key={todo.id}>
                        {todo.completed ? (
                            <s>{todo.text}</s>
                        ) : (
                            todo.text
                        )}
                    </li>
                ))}
            </ul>
            {footer}
        </>
    );
}

function NewTodo({ onAdd }) {
    const [text, setText] = useState('');

    function handleAddClick() {
        setText('');
        onAdd(createTodo(text));
    }

    return (
        <>
            <input
                value={text}
                onChange={(e) => setText(e.target.value)}
            />
            <button onClick={handleAddClick}>Add</button>
        </>
    );
}
```

<!-- 0067.part.md -->

<!-- 0068.part.md -->

```js
let nextId = 0;

export function createTodo(text, completed = false) {
    return {
        id: nextId++,
        text,
        completed,
    };
}

export const initialTodos = [
    createTodo('Get apples', true),
    createTodo('Get oranges', true),
    createTodo('Get carrots'),
];
```

<!-- 0069.part.md -->

<!-- 0070.part.md -->

```css
label {
    display: block;
}
input {
    margin-top: 10px;
}
```

<!-- 0071.part.md -->

\<Hint\>

Если вы можете рассчитать что-то во время рендеринга, вам не нужно состояние или Эффект, который его обновляет.

\</Hint\>

\<Решение\>

В этом примере есть только две важные части состояния: список `todos` и переменная состояния `showActive`, которая показывает, установлен ли флажок. Все остальные переменные состояния являются [избыточными](/learn/choosing-the-state-structure#avoid-redundant-state) и могут быть вычислены во время рендеринга. Это включает `footer`, который вы можете перенести непосредственно в окружающий JSX.

В итоге ваш результат должен выглядеть следующим образом:

<!-- 0072.part.md -->

```js
import { useState } from 'react';
import { initialTodos, createTodo } from './todos.js';

export default function TodoList() {
    const [todos, setTodos] = useState(initialTodos);
    const [showActive, setShowActive] = useState(false);
    const activeTodos = todos.filter(
        (todo) => !todo.completed
    );
    const visibleTodos = showActive ? activeTodos : todos;

    return (
        <>
            <label>
                <input
                    type="checkbox"
                    checked={showActive}
                    onChange={(e) =>
                        setShowActive(e.target.checked)
                    }
                />
                Show only active todos
            </label>
            <NewTodo
                onAdd={(newTodo) =>
                    setTodos([...todos, newTodo])
                }
            />
            <ul>
                {visibleTodos.map((todo) => (
                    <li key={todo.id}>
                        {todo.completed ? (
                            <s>{todo.text}</s>
                        ) : (
                            todo.text
                        )}
                    </li>
                ))}
            </ul>
            <footer>{activeTodos.length} todos left</footer>
        </>
    );
}

function NewTodo({ onAdd }) {
    const [text, setText] = useState('');

    function handleAddClick() {
        setText('');
        onAdd(createTodo(text));
    }

    return (
        <>
            <input
                value={text}
                onChange={(e) => setText(e.target.value)}
            />
            <button onClick={handleAddClick}>Add</button>
        </>
    );
}
```

<!-- 0073.part.md -->

<!-- 0074.part.md -->

```js
let nextId = 0;

export function createTodo(text, completed = false) {
    return {
        id: nextId++,
        text,
        completed,
    };
}

export const initialTodos = [
    createTodo('Get apples', true),
    createTodo('Get oranges', true),
    createTodo('Get carrots'),
];
```

<!-- 0075.part.md -->

<!-- 0076.part.md -->

```css
label {
    display: block;
}
input {
    margin-top: 10px;
}
```

<!-- 0077.part.md -->

\</Solution\>

#### Кэширование расчета без эффектов {/_cache-a-calculation-without-effects_/}

В этом примере фильтрация тодосов была вынесена в отдельную функцию под названием `getVisibleTodos()`. Эта функция содержит внутри себя вызов `console.log()`, который поможет вам заметить, когда она вызывается. Установите флажок "Показывать только активные тодосы" и обратите внимание, что это вызывает повторный запуск `getVisibleTodos()`. Это ожидаемо, поскольку видимые тодосы меняются, когда вы переключаете, какие из них показывать.

Ваша задача - удалить эффект, который пересчитывает список `visibleTodos` в компоненте `TodoList`. Однако, вам нужно убедиться, что `getVisibleTodos()` _не_ повторно запускается (и поэтому не печатает никаких логов), когда вы вводите данные в input.

\<Hint\>

Одно из решений - добавить вызов `useMemo` для кэширования видимых задач. Есть и другое, менее очевидное решение.

\</Hint\>

<!-- 0078.part.md -->

```js
import { useState, useEffect } from 'react';
import {
    initialTodos,
    createTodo,
    getVisibleTodos,
} from './todos.js';

export default function TodoList() {
    const [todos, setTodos] = useState(initialTodos);
    const [showActive, setShowActive] = useState(false);
    const [text, setText] = useState('');
    const [visibleTodos, setVisibleTodos] = useState([]);

    useEffect(() => {
        setVisibleTodos(getVisibleTodos(todos, showActive));
    }, [todos, showActive]);

    function handleAddClick() {
        setText('');
        setTodos([...todos, createTodo(text)]);
    }

    return (
        <>
            <label>
                <input
                    type="checkbox"
                    checked={showActive}
                    onChange={(e) =>
                        setShowActive(e.target.checked)
                    }
                />
                Show only active todos
            </label>
            <input
                value={text}
                onChange={(e) => setText(e.target.value)}
            />
            <button onClick={handleAddClick}>Add</button>
            <ul>
                {visibleTodos.map((todo) => (
                    <li key={todo.id}>
                        {todo.completed ? (
                            <s>{todo.text}</s>
                        ) : (
                            todo.text
                        )}
                    </li>
                ))}
            </ul>
        </>
    );
}
```

<!-- 0079.part.md -->

<!-- 0080.part.md -->

```js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
    console.log(
        `getVisibleTodos() was called ${++calls} times`
    );
    const activeTodos = todos.filter(
        (todo) => !todo.completed
    );
    const visibleTodos = showActive ? activeTodos : todos;
    return visibleTodos;
}

export function createTodo(text, completed = false) {
    return {
        id: nextId++,
        text,
        completed,
    };
}

export const initialTodos = [
    createTodo('Get apples', true),
    createTodo('Get oranges', true),
    createTodo('Get carrots'),
];
```

<!-- 0081.part.md -->

<!-- 0082.part.md -->

```css
label {
    display: block;
}
input {
    margin-top: 10px;
}
```

<!-- 0083.part.md -->

\<Решение\>

Удалите переменную state и Effect, а вместо этого добавьте вызов `useMemo` для кэширования результата вызова `getVisibleTodos()`:

<!-- 0084.part.md -->

```js
import { useState, useMemo } from 'react';
import {
    initialTodos,
    createTodo,
    getVisibleTodos,
} from './todos.js';

export default function TodoList() {
    const [todos, setTodos] = useState(initialTodos);
    const [showActive, setShowActive] = useState(false);
    const [text, setText] = useState('');
    const visibleTodos = useMemo(
        () => getVisibleTodos(todos, showActive),
        [todos, showActive]
    );

    function handleAddClick() {
        setText('');
        setTodos([...todos, createTodo(text)]);
    }

    return (
        <>
            <label>
                <input
                    type="checkbox"
                    checked={showActive}
                    onChange={(e) =>
                        setShowActive(e.target.checked)
                    }
                />
                Show only active todos
            </label>
            <input
                value={text}
                onChange={(e) => setText(e.target.value)}
            />
            <button onClick={handleAddClick}>Add</button>
            <ul>
                {visibleTodos.map((todo) => (
                    <li key={todo.id}>
                        {todo.completed ? (
                            <s>{todo.text}</s>
                        ) : (
                            todo.text
                        )}
                    </li>
                ))}
            </ul>
        </>
    );
}
```

<!-- 0085.part.md -->

<!-- 0086.part.md -->

```js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
    console.log(
        `getVisibleTodos() was called ${++calls} times`
    );
    const activeTodos = todos.filter(
        (todo) => !todo.completed
    );
    const visibleTodos = showActive ? activeTodos : todos;
    return visibleTodos;
}

export function createTodo(text, completed = false) {
    return {
        id: nextId++,
        text,
        completed,
    };
}

export const initialTodos = [
    createTodo('Get apples', true),
    createTodo('Get oranges', true),
    createTodo('Get carrots'),
];
```

<!-- 0087.part.md -->

<!-- 0088.part.md -->

```css
label {
    display: block;
}
input {
    margin-top: 10px;
}
```

<!-- 0089.part.md -->

С этим изменением `getVisibleTodos()` будет вызываться только при изменении `todos` или `showActive`. Ввод текста в input изменяет только переменную состояния `text`, поэтому он не вызывает вызов `getVisibleTodos()`.

Есть и другое решение, которое не требует использования `useMemo`. Поскольку переменная состояния `text` не может повлиять на список дел, вы можете выделить форму `NewTodo` в отдельный компонент и переместить переменную состояния `text` в него:

<!-- 0090.part.md -->

```js
import { useState, useMemo } from 'react';
import {
    initialTodos,
    createTodo,
    getVisibleTodos,
} from './todos.js';

export default function TodoList() {
    const [todos, setTodos] = useState(initialTodos);
    const [showActive, setShowActive] = useState(false);
    const visibleTodos = getVisibleTodos(todos, showActive);

    return (
        <>
            <label>
                <input
                    type="checkbox"
                    checked={showActive}
                    onChange={(e) =>
                        setShowActive(e.target.checked)
                    }
                />
                Show only active todos
            </label>
            <NewTodo
                onAdd={(newTodo) =>
                    setTodos([...todos, newTodo])
                }
            />
            <ul>
                {visibleTodos.map((todo) => (
                    <li key={todo.id}>
                        {todo.completed ? (
                            <s>{todo.text}</s>
                        ) : (
                            todo.text
                        )}
                    </li>
                ))}
            </ul>
        </>
    );
}

function NewTodo({ onAdd }) {
    const [text, setText] = useState('');

    function handleAddClick() {
        setText('');
        onAdd(createTodo(text));
    }

    return (
        <>
            <input
                value={text}
                onChange={(e) => setText(e.target.value)}
            />
            <button onClick={handleAddClick}>Add</button>
        </>
    );
}
```

<!-- 0091.part.md -->

<!-- 0092.part.md -->

```js
let nextId = 0;
let calls = 0;

export function getVisibleTodos(todos, showActive) {
    console.log(
        `getVisibleTodos() was called ${++calls} times`
    );
    const activeTodos = todos.filter(
        (todo) => !todo.completed
    );
    const visibleTodos = showActive ? activeTodos : todos;
    return visibleTodos;
}

export function createTodo(text, completed = false) {
    return {
        id: nextId++,
        text,
        completed,
    };
}

export const initialTodos = [
    createTodo('Get apples', true),
    createTodo('Get oranges', true),
    createTodo('Get carrots'),
];
```

<!-- 0093.part.md -->

<!-- 0094.part.md -->

```css
label {
    display: block;
}
input {
    margin-top: 10px;
}
```

<!-- 0095.part.md -->

Этот подход также удовлетворяет требованиям. Когда вы вводите текст в input, обновляется только переменная состояния `text`. Поскольку переменная состояния `text` находится в дочернем компоненте `NewTodo`, родительский компонент `TodoList` не будет перерисован. Вот почему `getVisibleTodos()` не вызывается при вводе текста. (Она будет вызвана, если `TodoList` будет рендериться по другой причине).

\</Solution\>

#### Сброс состояния без эффектов {/_reset-state-without-effects_/}

Этот компонент `EditContact` получает объект контакта, имеющий форму `{ id, name, email }` в качестве реквизита `savedContact`. Попробуйте отредактировать поля ввода имени и электронной почты. Когда вы нажмете кнопку Save, кнопка контакта над формой обновится на отредактированное имя. При нажатии кнопки Reset все изменения в форме будут отменены. Поиграйте с этим пользовательским интерфейсом, чтобы почувствовать его.

Когда вы выбираете контакт с помощью кнопок вверху, форма сбрасывается, чтобы отразить данные этого контакта. Это делается с помощью эффекта внутри `EditContact.js`. Удалите этот эффект. Найдите другой способ сброса формы при изменении `savedContact.id`.

<!-- 0096.part.md -->

```js
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
    const [contacts, setContacts] = useState(
        initialContacts
    );
    const [selectedId, setSelectedId] = useState(0);
    const selectedContact = contacts.find(
        (c) => c.id === selectedId
    );

    function handleSave(updatedData) {
        const nextContacts = contacts.map((c) => {
            if (c.id === updatedData.id) {
                return updatedData;
            } else {
                return c;
            }
        });
        setContacts(nextContacts);
    }

    return (
        <div>
            <ContactList
                contacts={contacts}
                selectedId={selectedId}
                onSelect={(id) => setSelectedId(id)}
            />
            <hr />
            <EditContact
                savedContact={selectedContact}
                onSave={handleSave}
            />
        </div>
    );
}

const initialContacts = [
    { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
    { id: 1, name: 'Alice', email: 'alice@mail.com' },
    { id: 2, name: 'Bob', email: 'bob@mail.com' },
];
```

<!-- 0097.part.md -->

<!-- 0098.part.md -->

```js
export default function ContactList({
    contacts,
    selectedId,
    onSelect,
}) {
    return (
        <section>
            <ul>
                {contacts.map((contact) => (
                    <li key={contact.id}>
                        <button
                            onClick={() => {
                                onSelect(contact.id);
                            }}
                        >
                            {contact.id === selectedId ? (
                                <b>{contact.name}</b>
                            ) : (
                                contact.name
                            )}
                        </button>
                    </li>
                ))}
            </ul>
        </section>
    );
}
```

<!-- 0099.part.md -->

<!-- 0100.part.md -->

```js
import { useState, useEffect } from 'react';

export default function EditContact({
    savedContact,
    onSave,
}) {
    const [name, setName] = useState(savedContact.name);
    const [email, setEmail] = useState(savedContact.email);

    useEffect(() => {
        setName(savedContact.name);
        setEmail(savedContact.email);
    }, [savedContact]);

    return (
        <section>
            <label>
                Name:{' '}
                <input
                    type="text"
                    value={name}
                    onChange={(e) =>
                        setName(e.target.value)
                    }
                />
            </label>
            <label>
                Email:{' '}
                <input
                    type="email"
                    value={email}
                    onChange={(e) =>
                        setEmail(e.target.value)
                    }
                />
            </label>
            <button
                onClick={() => {
                    const updatedData = {
                        id: savedContact.id,
                        name: name,
                        email: email,
                    };
                    onSave(updatedData);
                }}
            >
                Save
            </button>
            <button
                onClick={() => {
                    setName(savedContact.name);
                    setEmail(savedContact.email);
                }}
            >
                Reset
            </button>
        </section>
    );
}
```

<!-- 0101.part.md -->

<!-- 0102.part.md -->

```css
ul,
li {
    list-style: none;
    margin: 0;
    padding: 0;
}
li {
    display: inline-block;
}
li button {
    padding: 10px;
}
label {
    display: block;
    margin: 10px 0;
}
button {
    margin-right: 10px;
    margin-bottom: 10px;
}
```

<!-- 0103.part.md -->

\<Hint\>

Было бы неплохо, если бы существовал способ сказать React, что когда `savedContact.id` отличается, форма `EditContact` концептуально является _другой формой контакта_ и не должна сохранять состояние. Не припомните ли вы какой-нибудь подобный способ?

\</Hint\>

\<Решение\>

Разделите компонент `EditContact` на две части. Переместите все состояние формы во внутренний компонент `EditForm`. Экспортируйте внешний компонент `EditContact` и заставьте его передавать `savedContact.id` в качестве `ключа` внутреннему компоненту `EditContact`. В результате внутренний компонент `EditForm` сбросит все состояние формы и пересоздаст DOM каждый раз, когда вы выбираете другой контакт.

<!-- 0104.part.md -->

```js
import { useState } from 'react';
import ContactList from './ContactList.js';
import EditContact from './EditContact.js';

export default function ContactManager() {
    const [contacts, setContacts] = useState(
        initialContacts
    );
    const [selectedId, setSelectedId] = useState(0);
    const selectedContact = contacts.find(
        (c) => c.id === selectedId
    );

    function handleSave(updatedData) {
        const nextContacts = contacts.map((c) => {
            if (c.id === updatedData.id) {
                return updatedData;
            } else {
                return c;
            }
        });
        setContacts(nextContacts);
    }

    return (
        <div>
            <ContactList
                contacts={contacts}
                selectedId={selectedId}
                onSelect={(id) => setSelectedId(id)}
            />
            <hr />
            <EditContact
                savedContact={selectedContact}
                onSave={handleSave}
            />
        </div>
    );
}

const initialContacts = [
    { id: 0, name: 'Taylor', email: 'taylor@mail.com' },
    { id: 1, name: 'Alice', email: 'alice@mail.com' },
    { id: 2, name: 'Bob', email: 'bob@mail.com' },
];
```

<!-- 0105.part.md -->

<!-- 0106.part.md -->

```js
export default function ContactList({
    contacts,
    selectedId,
    onSelect,
}) {
    return (
        <section>
            <ul>
                {contacts.map((contact) => (
                    <li key={contact.id}>
                        <button
                            onClick={() => {
                                onSelect(contact.id);
                            }}
                        >
                            {contact.id === selectedId ? (
                                <b>{contact.name}</b>
                            ) : (
                                contact.name
                            )}
                        </button>
                    </li>
                ))}
            </ul>
        </section>
    );
}
```

<!-- 0107.part.md -->

<!-- 0108.part.md -->

```js
import { useState } from 'react';

export default function EditContact(props) {
    return (
        <EditForm {...props} key={props.savedContact.id} />
    );
}

function EditForm({ savedContact, onSave }) {
    const [name, setName] = useState(savedContact.name);
    const [email, setEmail] = useState(savedContact.email);

    return (
        <section>
            <label>
                Name:{' '}
                <input
                    type="text"
                    value={name}
                    onChange={(e) =>
                        setName(e.target.value)
                    }
                />
            </label>
            <label>
                Email:{' '}
                <input
                    type="email"
                    value={email}
                    onChange={(e) =>
                        setEmail(e.target.value)
                    }
                />
            </label>
            <button
                onClick={() => {
                    const updatedData = {
                        id: savedContact.id,
                        name: name,
                        email: email,
                    };
                    onSave(updatedData);
                }}
            >
                Save
            </button>
            <button
                onClick={() => {
                    setName(savedContact.name);
                    setEmail(savedContact.email);
                }}
            >
                Reset
            </button>
        </section>
    );
}
```

<!-- 0109.part.md -->

<!-- 0110.part.md -->

```css
ul,
li {
    list-style: none;
    margin: 0;
    padding: 0;
}
li {
    display: inline-block;
}
li button {
    padding: 10px;
}
label {
    display: block;
    margin: 10px 0;
}
button {
    margin-right: 10px;
    margin-bottom: 10px;
}
```

<!-- 0111.part.md -->

\</Solution\>

#### Отправить форму без эффектов {/_submit-a-form-without-effects_/}

Этот компонент `Form` позволяет вам отправить сообщение другу. Когда вы отправляете форму, переменная состояния `showForm` устанавливается в `false`. Это вызывает Эффект, вызывающий `sendMessage(message)`, который отправляет сообщение (вы можете увидеть его в консоли). После отправки сообщения вы видите диалог "Спасибо" с кнопкой "Открыть чат", которая позволяет вам вернуться к форме.

Пользователи вашего приложения отправляют слишком много сообщений. Чтобы немного усложнить общение в чате, вы решили показывать диалог "Спасибо" _первым_, а не форму. Измените переменную состояния `showForm` так, чтобы она инициализировалась значением `false` вместо `true`. Как только вы сделаете это изменение, консоль покажет, что было отправлено пустое сообщение. Что-то в этой логике неправильно\!

В чем первопричина этой проблемы? И как вы можете ее устранить?

\< Подсказка\>

Должно ли сообщение быть отправлено _потому что_ пользователь увидел диалог "Спасибо"? Или наоборот?

\</Hint\>

<!-- 0112.part.md -->

```js
import { useState, useEffect } from 'react';

export default function Form() {
    const [showForm, setShowForm] = useState(true);
    const [message, setMessage] = useState('');

    useEffect(() => {
        if (!showForm) {
            sendMessage(message);
        }
    }, [showForm, message]);

    function handleSubmit(e) {
        e.preventDefault();
        setShowForm(false);
    }

    if (!showForm) {
        return (
            <>
                <h1>Thanks for using our services!</h1>
                <button
                    onClick={() => {
                        setMessage('');
                        setShowForm(true);
                    }}
                >
                    Open chat
                </button>
            </>
        );
    }

    return (
        <form onSubmit={handleSubmit}>
            <textarea
                placeholder="Message"
                value={message}
                onChange={(e) => setMessage(e.target.value)}
            />
            <button type="submit" disabled={message === ''}>
                Send
            </button>
        </form>
    );
}

function sendMessage(message) {
    console.log('Sending message: ' + message);
}
```

<!-- 0113.part.md -->

<!-- 0114.part.md -->

```css
label,
textarea {
    margin-bottom: 10px;
    display: block;
}
```

<!-- 0115.part.md -->

\<Решение\>

Переменная состояния `showForm` определяет, показывать ли форму или диалог "Спасибо". Однако, вы отправляете сообщение не потому, что диалог "Спасибо" был _показан_. Вы хотите отправить сообщение, потому что пользователь _отправил форму._ Удалите вводящий в заблуждение Эффект и переместите вызов `sendMessage` в обработчик события `handleSubmit`:

<!-- 0116.part.md -->

```js
import { useState, useEffect } from 'react';

export default function Form() {
    const [showForm, setShowForm] = useState(true);
    const [message, setMessage] = useState('');

    function handleSubmit(e) {
        e.preventDefault();
        setShowForm(false);
        sendMessage(message);
    }

    if (!showForm) {
        return (
            <>
                <h1>Thanks for using our services!</h1>
                <button
                    onClick={() => {
                        setMessage('');
                        setShowForm(true);
                    }}
                >
                    Open chat
                </button>
            </>
        );
    }

    return (
        <form onSubmit={handleSubmit}>
            <textarea
                placeholder="Message"
                value={message}
                onChange={(e) => setMessage(e.target.value)}
            />
            <button type="submit" disabled={message === ''}>
                Send
            </button>
        </form>
    );
}

function sendMessage(message) {
    console.log('Sending message: ' + message);
}
```

<!-- 0117.part.md -->

<!-- 0118.part.md -->

```css
label,
textarea {
    margin-bottom: 10px;
    display: block;
}
```

<!-- 0119.part.md -->

Обратите внимание, что в этой версии только _отправка формы_ (которая является событием) вызывает отправку сообщения. Это работает одинаково хорошо независимо от того, установлено ли `showForm` изначально в `true` или `false`. (Установите значение `false` и не заметите никаких дополнительных консольных сообщений).

\</Solution\>

\</Challenges\>

<!-- 0120.part.md -->

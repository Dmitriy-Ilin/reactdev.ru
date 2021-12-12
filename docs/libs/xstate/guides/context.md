# Контекст

Хотя _конечные_ состояния четко определены в конечных автоматах и ​​диаграммах состояний, состояние, которое представляет _количественные данные_ (например, произвольные строки, числа, объекты и т. д.), которые могут быть потенциально бесконечными, представлено как [расширенное состояние](https://en.wikipedia.org/wiki/UML_state_machine#Extended_states). Это делает диаграммы состояний более полезными для реальных приложений.

В XState расширенное состояние известно как **контекст** (_context_). Ниже приведен пример использования `context` для имитации наполнения стакана водой:

```js
import { createMachine, assign } from 'xstate';

// Action to increment the context amount
const addWater = assign({
  amount: (context, event) => context.amount + 1,
});

// Guard to check if the glass is full
function glassIsFull(context, event) {
  return context.amount >= 10;
}

const glassMachine = createMachine(
  {
    id: 'glass',
    // the initial context (extended state) of the statechart
    context: {
      amount: 0,
    },
    initial: 'empty',
    states: {
      empty: {
        on: {
          FILL: {
            target: 'filling',
            actions: 'addWater',
          },
        },
      },
      filling: {
        // Transient transition
        always: {
          target: 'full',
          cond: 'glassIsFull',
        },
        on: {
          FILL: {
            target: 'filling',
            actions: 'addWater',
          },
        },
      },
      full: {},
    },
  },
  {
    actions: { addWater },
    guards: { glassIsFull },
  }
);
```

Текущий контекст ссылается на `State` как `state.context`:

```js
const nextState = glassMachine.transition(
  glassMachine.initialState,
  {
    type: 'FILL',
  }
);

nextState.context;
// => { amount: 1 }
```

## Начальный контекст

Начальный контекст указывается в свойстве `context` автомата:

```js
const counterMachine = createMachine({
  id: 'counter',
  // initial context
  context: {
    count: 0,
    message: 'Currently empty',
    user: {
      name: 'David',
    },
    allowedToIncrement: true,
    // ... etc.
  },
  states: {
    // ...
  },
});
```

Для динамического контекста (то есть контекста, начальное значение которого задается извне) вы можете использовать фабричную функцию, которая создает автомат с предоставленными значениями контекста, например:

```js
const createCounterMachine = (count, time) => {
  return createMachine({
    id: 'counter',
    // values provided from function arguments
    context: {
      count,
      time,
    },
    // ...
  });
};

const counterMachine = createCounterMachine(42, Date.now());
```

Для существующих автоматов следует использовать `machine.withContext(...)`:

```js
const counterMachine = createMachine({
  /* ... */
});

// retrieved dynamically
const someContext = { count: 42, time: Date.now() };

const dynamicCounterMachine = counterMachine.withContext(
  someContext
);
```

Исходный контекст машины можно получить из ее начального состояния:

```js
dynamicCounterMachine.initialState.context;
// => { count: 42, time: 1543687816981 }
```

Этот способ предпочтительнее прямого доступа к `machine.context`, так как начальное состояние вычисляется с помощью начальных действий `assign(...)` и проходных переходов, если таковые имеются.

## Действие assign()

Действие `assign()` используется для обновления контекста автомата. Оно принимает контекст `assigner`, который указывает, как должны быть присвоены значения в текущем контексте.

| Параметр   | Тип                  | Описание                                                   |
| ---------- | -------------------- | ---------------------------------------------------------- |
| `assigner` | object<br />function | Объект или функция, которые присваивают значения контексту |

`assigner` может быть объектом (рекомендовано):

```js
import { createMachine, assign } from 'xstate';
// example: property assigner

// ...
  actions: assign({
    // increment the current count by the event value
    count: (context, event) => context.count + event.value,

    // assign static value to the message (no function needed)
    message: 'Count changed'
  }),
// ...
```

Или функция, которая возвращает обновленное состояние:

```js
// example: context assigner

// ...

  // return a partial (or full) updated context
  actions: assign((context, event) => {
    return {
      count: context.count + event.value,
      message: 'Count changed'
    }
  }),
// ...
```

Приведенная выше функция принимает три параметра: `context`, `event` и `meta`

| Параметр  | Тип         | Описание                                          |
| --------- | ----------- | ------------------------------------------------- |
| `context` | TContext    | Текущий контекст (расширенное состояние) автомата |
| `event`   | EventObject | Событие, вызвавшее действие `assign`              |
| `meta`    | AssignMeta  | объект с мета-данными, _начиная с версии 4.7+_    |

Объект мета-данных содержит:

- `state` — текущее состояние при нормальном переходе (`undefined` для перехода начального состояния)
- `action` — связанное действие `assign`

!!!warning "Внимание"

    Функция `assign(...)` является **создателем действия**; это чистая функция, которая возвращает только объект действия и *не делает* обязательных присваиваний контексту.

## Порядок действий

Пользовательские действия всегда выполняются в отношении _следующего состояния_ в переходе. Когда переход состояния имеет действия `assign(...)`, эти действия всегда группируются и вычисляются _первыми_, чтобы определить следующее состояние. Так происходит потому, что состояние - это комбинация конечного состояния и расширенного состояния (контекста).

Например, на этом счетчике пользовательские действия не будут работать должным образом:

```js
const counterMachine = createMachine({
  id: 'counter',
  context: { count: 0 },
  initial: 'active',
  states: {
    active: {
      on: {
        INC_TWICE: {
          actions: [
            (context) =>
              console.log(`Before: ${context.count}`),
            assign({
              count: (context) => context.count + 1,
            }), // count === 1
            assign({
              count: (context) => context.count + 1,
            }), // count === 2
            (context) =>
              console.log(`After: ${context.count}`),
          ],
        },
      },
    },
  },
});

interpret(counterMachine)
  .start()
  .send({ type: 'INC_TWICE' });
// => "Before: 2"
// => "After: 2"
```

Это связано с тем, что оба действия `assign(...)` группируются по порядку и выполняются первыми (на микрошаге), поэтому следующим `context` состояния является `{count: 2}`, который передается обоим настраиваемым действиям. Другой способ думать об этом переходе - читать его так:

> Когда в состоянии `active` происходит событие `INC_TWICE`, следующее состояние — это состояние `active` с обновленным `context.count`, а затем _эти_ настраиваемые действия выполняются в этом состоянии.

Хороший способ рефакторинга этого для получения желаемого результата — моделирование `context` с явными _предыдущими_ значениями, если они необходимы:

```js
const counterMachine = createMachine({
  id: 'counter',
  context: { count: 0, prevCount: undefined },
  initial: 'active',
  states: {
    active: {
      on: {
        INC_TWICE: {
          actions: [
            (context) =>
              console.log(`Before: ${context.prevCount}`),
            assign({
              count: (context) => context.count + 1,
              prevCount: (context) => context.count,
            }), // count === 1, prevCount === 0
            assign({
              count: (context) => context.count + 1,
            }), // count === 2
            (context) =>
              console.log(`After: ${context.count}`),
          ],
        },
      },
    },
  },
});

interpret(counterMachine)
  .start()
  .send({ type: 'INC_TWICE' });
// => "Before: 0"
// => "After: 2"
```

Преимущества от этого:

1. Расширенное состояние (контекст) моделируется более явно
2. Отсутствуют неявные промежуточные состояния, предотвращающие появление трудноуловимых ошибок.
3. Порядок действий более независим (Логирование «До» может идти даже после логирования «После»!)
4. Облегчает тестирование и изучение состояния

## Примечания

- 🚫 Никогда не изменяйте контекст `context` автомата извне. У всего есть причина, и каждое изменение контекста должно происходить явно из-за события.
- Предпочтителен синтаксис объекта `assign({...})`. Это позволяет будущим инструментам анализа предсказывать, _как_ определенные свойства могут измениться декларативно.
- Задания `assign` можно складывать, и они будут выполняться последовательно:

```js
// ...
  actions: [
    assign({ count: 3 }), // context.count === 3
    assign({ count: context => context.count * 2 }) // context.count === 6
  ],
// ...
```

- Как и в случае с `actions`, лучше всего представлять действия `assign()` в виде строк или функций, а затем ссылаться на них в параметрах автомата:

```js hl_lines="6"
const countMachine = createMachine({
  initial: 'start',
  context: { count: 0 }
  states: {
    start: {
      entry: 'increment'
    }
  }
}, {
  actions: {
    increment: assign({ count: context => context.count + 1 }),
    decrement: assign({ count: context => context.count - 1 })
  }
});
```

Или через именованные функции:

```js hl_lines="9"
const increment = assign({ count: context => context.count + 1 });
const decrement = assign({ count: context => context.count - 1 });

const countMachine = createMachine({
  initial: 'start',
  context: { count: 0 }
  states: {
    start: {
      // Named function
      entry: increment
    }
  }
});
```

- В идеале `context` должен быть представлен как простой объект JavaScript, т. е. он должен быть сериализуемым как JSON.
- Поскольку вызываются действия `assign()`, контекст обновляется перед выполнением других действий. Это означает, что другие действия на том же шаге получат обновленный контекст, а не тот, который был до выполнения действия `assign()`. Вы не должны полагаться на порядок действий для своих состояний, но имейте это в виду.

## TypeScript

Для правильного вывода типа, добавьте тип контекста в качестве первого параметра типа в `createMachine<TContext, ...>`:

```ts
interface CounterContext {
  count: number;
  user?: {
    name: string;
  };
}

const machine = createMachine<CounterContext>({
  // ...
  context: {
    count: 0,
    user: undefined,
  },
  // ...
});
```

Если возможно, вы также можете использовать `typeof ...` как сокращение:

```ts
const context = {
  count: 0,
  user: { name: '' },
};

const machine = createMachine<typeof context>({
  // ...
  context,
  // ...
});
```

В большинстве случаев типы контекста `context` и события `event` в действиях `assign(...)` будут автоматически выведены из параметров типа, переданных в `createMachine<TContext, TEvent>`:

```ts
interface CounterContext {
  count: number;
}

const machine = createMachine<CounterContext>({
  // ...
  context: {
    count: 0
  },
  // ...
  {
    on: {
      INCREMENT: {
        // Inferred automatically in most cases
        actions: assign({
          count: (context) => {
            // context: { count: number }
            return context.count + 1;
          }
        })
      }
    }
  }
});
```

Однако вывод TypeScript не идеален, поэтому можно добавить контекст и событие в качестве обобщений в `assign<Context, Event>(...)`:

```ts hl_lines="3"
// ...
on: {
  INCREMENT: {
    // Generics guarantee proper inference
    actions: assign<CounterContext, CounterEvent>({
      count: (context) => {
        // context: { count: number }
        return context.count + 1;
      },
    });
  }
}
// ...
```

## Краткий справочник

**Установка начального контекста**

```js
const machine = createMachine({
  // ...
  context: {
    count: 0,
    user: undefined,
    // ...
  },
});
```

**Установка динамического начального контекста**

```js
const createSomeMachine = (count, user) => {
  return createMachine({
    // ...
    // Provided from arguments; your implementation may vary
    context: {
      count,
      user,
      // ...
    },
  });
};
```

**Установка пользовательского контекста автомату**

```js
const machine = createMachine({
  // ...
  // Provided from arguments; your implementation may vary
  context: {
    count: 0,
    user: undefined,
    // ...
  },
});

const myMachine = machine.withContext({
  count: 10,
  user: {
    name: 'David',
  },
});
```

**Связывание контекста**

```js
const machine = createMachine({
  // ...
  context: {
    count: 0,
    user: undefined,
    // ...
  },
  // ...
  on: {
    INCREMENT: {
      actions: assign({
        count: (context, event) => context.count + 1,
      }),
    },
  },
});
```

**Статичное связывание**

```js
// ...
actions: assign({
  counter: 42
}),
// ...
```

**Связывание через функцию**

```js
// ...
actions: assign({
  counter: (context, event) => {
    return context.count + event.value;
  }
}),
// ...
```

**Связывание через контекст**

```js
// ...
actions: assign((context, event) => {
  return {
    counter: context.count + event.value,
    time: event.time,
    // ...
  }
}),
// ...
```

**Множественное связывание**

```js
// ...
// assume context.count === 1
actions: [
  // assigns context.count to 1 + 1 = 2
  assign({ count: (context) => context.count + 1 }),
  // assigns context.count to 2 * 3 = 6
  assign({ count: (context) => context.count * 3 })
],
// ...
```

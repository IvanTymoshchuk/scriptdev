# Индексные сигнатуры

К `Object` в JavaScript (и, следовательно, в TypeScript) можно получить доступ с помощью **строки**, содержащей ссылку на любой другой JavaScript **объект**.

Вот краткий пример:

```ts
let foo: any = {};
foo['Hello'] = 'World';
console.log(foo['Hello']); // World
```

Мы храним строку `"World"` под ключом `"Hello"`. Помните, мы говорили, что он может хранить любой JavaScript **объект**, поэтому давайте сохраним экземпляр класса, чтобы показать концепцию:

```ts
class Foo {
    constructor(public message: string) {}
    log() {
        console.log(this.message);
    }
}

let foo: any = {};
foo['Hello'] = new Foo('World');
foo['Hello'].log(); // World
```

Помните мы сказали, что к нему можно получить доступ с помощью **строки**. Если вы передадите в сигнатуру индекса любой другой объект, среда выполнения JavaScript вызовет для него `.toString` перед получением результата. Это показано ниже:

```ts
let obj = {
    toString() {
        console.log('toString вызывается');
        return 'Hello';
    },
};

let foo: any = {};
foo[obj] = 'World'; // toString вызывается
console.log(foo[obj]); // toString вызывается, World
console.log(foo['Hello']); // World
```

Обратите внимание, что `toString` будет вызываться всякий раз, когда `obj` используется в позиции индекса.

Массивы немного отличаются. Для индексации `числа` виртуальные машины JavaScript будут пытаться оптимизировать (в зависимости от того, действительно ли это массив, совпадают ли структуры хранимых элементов и т.д.). Таким образом, `число` следует рассматривать как действительный метод доступа к объекту сам по себе (в отличие от `строки`). Вот простой пример c массивом:

```ts
let foo = ['World'];
console.log(foo[0]); // World
```

Итак, это JavaScript. Теперь давайте посмотрим на изящную обработку этой концепции в TypeScript.

## Индексные сигнатуры в TypeScript

Во-первых, поскольку JavaScript _неявно_ вызывает `toString` для любой сигнатуры индекса в виде объекта, TypeScript выдаст вам ошибку, чтобы новички не стреляли себе в ногу (я вижу, как пользователи стреляют себе в ногу при постоянном использовании JavaScript на stackoverflow):

```ts
let obj = {
    toString() {
        return 'Hello';
    },
};

let foo: any = {};

// ОШИБКА: сигнатура индекса должна быть строкой, числом ...
foo[obj] = 'World';

// ИСПРАВЛЕНИЕ: TypeScript заставляет вас быть явным
foo[obj.toString()] = 'World';
```

Причина для такого принуждения пользователя быть явным в том, что реализация `toString` по умолчанию для объекта довольно ужасна, например в v8 он всегда возвращает `[object Object]`:

```ts
let obj = { message: 'Hello' };
let foo: any = {};

// ОШИБКА: сигнатура индекса должна быть строкой, числом ...
foo[obj] = 'World';

// Вот где вы на самом деле хранили!
console.log(foo['[object Object]']); // World
```

Конечно, поддерживается `число`, потому что

1. Это необходимо для отличной поддержки массивов / кортежей.
2. Даже если вы используете его для `obj`, его реализация `toString` по умолчанию нормальна (не `[object Object]`).

Пункт 2 показан ниже:

```ts
console.log((1).toString()); // 1
console.log((2).toString()); // 2
```

Итак, урок 1:

> Сигнатуры индекса TypeScript должны быть либо строкой, либо числом.

Краткое примечание: `символы` также действительны и поддерживаются TypeScript. Но пока не будем туда заходить. Будем двигаться маленькими шажками.

## Объявление индексной сигнатуры

Итак, мы использовали `any`, чтобы сказать TypeScript, что мы можем делать все, что захотим. Фактически мы можем явно указать сигнатуру для _index_. Например, скажем, вы хотите убедиться, что все, что хранится в объекте с использованием строки, соответствует структуре `{message: string}`. Это можно сделать с помощью объявления `{ [index:string] : {message: string} }`. Это показано ниже:

```ts
let foo: { [index: string]: { message: string } } = {};

/**
 * Необходимо хранить в соответствии структуре
 */
/** Ok */
foo['a'] = { message: 'some message' };
/** Ошибка: должно содержать `message` типа string. У вас опечатка в
 * `message` */
foo['a'] = { messages: 'some message' };

/**
 * Тип также проверяется на чтение
 */
/** Ok */
foo['a'].message;
/** Ошибка: messages не существует. У вас опечатка в `message` */
foo['a'].messages;
```

> СОВЕТ: имя сигнатуры индекса, например `index` в `{ [index:string] : {message: string} }` не имеет значения для TypeScript и предназначен только для удобства чтения. Например если это имена пользователей, вы можете использовать `{ [username:string] : {message: string} }`, чтобы помочь следующему разработчику, который просматривает код (который кстати может оказаться вами).

Конечно, числовые индексы также поддерживаются, например `{ [count: number] : SomeOtherTypeYouWantToStoreEgRebate }`

## Все члены должны соответствовать сигнатуре индекса `string`

Как только у вас есть сигнатура индекса `string`, все явные элементы также должны соответствовать этой сигнатуре индекса. Это показано ниже:

```ts
/** Okay */
interface Foo {
    [key: string]: number;
    x: number;
    y: number;
}
/** Ошибка */
interface Bar {
    [key: string]: number;
    x: number;
    y: string; // ОШИБКА: Свойство `y` должно иметь тип number
}
```

Это сделано для обеспечения безопасности, чтобы любой доступ к строке давал одинаковый результат:

```ts
interface Foo {
    [key: string]: number;
    x: number;
}
let foo: Foo = { x: 1, y: 2 };

// Напрямую
foo['x']; // число

// Опосредственно
let x = 'x';
foo[x]; // число
```

## Использование ограниченного набора строковых литералов

Сигнатура индекса может требовать, чтобы строки индекса были частями объединения литеральных строк, используя _замапленные типы_, например:

```ts
type Index = 'a' | 'b' | 'c';
type FromIndex = { [k in Index]?: number };

const good: FromIndex = { b: 1, c: 2 };

// Ошибка:
// Тип '{ b: number; c: number; d: number; }' нельзя присвоить типу 'FromIndex'.
// Литерал объекта может указывать только известные свойства, а 'd'
// не существует в типе 'FromIndex'.
const bad: FromIndex = { b: 1, c: 2, d: 3 };
```

Это часто используется вместе с `keyof typeof` для захвата типов словаря, описанных далее.

В общем случае определение словаря может быть отложено:

```ts
type FromSomeIndex<K extends string> = {
    [key in K]: number;
};
```

## Используйте и строки и числа в качестве индексов

Это не стандартный вариант использования, но компилятор TypeScript, тем не менее, его поддерживает.

Однако у него есть ограничение: индексатор `string` более строгий, чем индексатор `number`. Это сделано намеренно, например чтобы разрешить следующий код:

```ts
interface ArrStr {
    [key: string]: string | number; // Должен согласовываться со всеми
    // элементами

    [index: number]: string; // Может быть подмножеством индексатора строк

    // Просто пример элемента
    length: number;
}
```

## Шаблон проектирования: вложенная индексная сигнатура

> Рекомендации для API при добавлении индексных сигнатур

Довольно часто в сообществе JS можно встретить API, которые неправильно употребляют строковые индексаторы. Например стандартный шаблон для CSS в библиотеках JS:

```ts
interface NestedCSS {
    color?: string;
    [selector: string]: string | NestedCSS | undefined;
}

const example: NestedCSS = {
    color: 'red',
    '.subclass': {
        color: 'blue',
    },
};
```

Постарайтесь не смешивать таким образом индексаторы строк с _валидными_ значениями. Например, опечатка останется невыявленной:

```ts
const failsSilently: NestedCSS = {
    colour: 'red', // Ошибок нет, так как colour является допустимым
    // селектором строки
};
```

Вместо этого поместите вложение в отдельное свойство, например в имя `nest` (или `children`, или `subnodes` и т.д.):

```ts
interface NestedCSS {
    color?: string;
    nest?: {
        [selector: string]: NestedCSS;
    };
}

const example: NestedCSS = {
    color: 'red',
    nest: {
        '.subclass': {
            color: 'blue',
        },
    },
};

const failsSilently: NestedCSS = {
    colour: 'red', // Ошибка TS: неизвестное свойство `colour`
};
```

## Исключение определенных свойств из сигнатуры индекса

Иногда вам нужно объединить свойства в сигнатуру индекса. Это не рекомендуется, и вам _следует_ использовать упомянутый выше шаблон вложенной индексной сигнатуры.

Однако, если вы моделируете _существующий JavaScript_, вы можете обойти это с помощью типа пересечения. Ниже показан пример ошибки, с которой вы столкнетесь без использования пересечения:

```ts
type FieldState = {
    value: string;
};

type FormState = {
    isValid: boolean; // Ошибка: не соответствует сигнатуре индекса
    [fieldName: string]: FieldState;
};
```

Вот обходной путь с использованием типа пересечения:

```ts
type FieldState = {
    value: string;
};

type FormState = { isValid: boolean } & {
    [fieldName: string]: FieldState;
};
```

Обратите внимание, что даже если вы можете объявить его для моделирования существующего JavaScript, вы не можете создать такой объект с помощью TypeScript:

```ts
type FieldState = {
    value: string;
};

type FormState = { isValid: boolean } & {
    [fieldName: string]: FieldState;
};

// Используйте это для какого-нибудь объекта JavaScript, который вы откуда-то
// получаете
declare const foo: FormState;

const isValidBool = foo.isValid;
const somethingFieldState = foo['something'];

// Использование его для создания объекта TypeScript не сработает
const bar: FormState = {
    // Ошибка: `isValid` не может быть присвоен `FieldState`
    isValid: false,
};
```
# `@types`

[Definitely Typed](https://github.com/DefinitelyTyped/DefinitelyTyped), безусловно, одна из самых сильных сторон TypeScript. Сообщество продуктивно продвинулось вперед и **задокументировало** сущность почти 90% лучших проектов на JavaScript.

Это означает, что вы можете использовать эти проекты в очень интерактивном и исследовательском режиме, не нужно открывать документы в отдельном окне чтобы убедиться что вы не сделали опечатку.

## Использование `@types`

Установка довольно проста, так как она работает поверх `npm`. В качестве примера вы можете установить определения типов для `jquery` просто как:

```
npm install @types/jquery --save-dev
```

`@types` поддерживает определения обоих видов _глобальные_ и _модульные_.

### Глобальные `@types`

По умолчанию любые определения, которые поддерживают глобальное употребление, включаются автоматически. Например, для `jquery` вы сможете использовать `$` _глобально_ в своем проекте.

Однако для _библиотек_ (например, `jquery`) я обычно рекомендую использовать _модули_:

### Модульные `@types`

После установки никаких специальных настроек не требуется. Вы просто используете его как модуль, например:

```ts
import * as $ from 'jquery';

// Используйте $ по своему усмотрению в этом модуле :)
```

## Контроль глобальных типов

Как можно видеть, наличие определения, которое позволяет глобальный доступ автоматически, может быть проблемой для некоторых команд. Что ж, вы можете _явно_ вводить только те типы, которые необходимы, используя `tsconfig.json` `compilerOptions.types`, например:

```json
{
    "compilerOptions": {
        "types": ["jquery"]
    }
}
```

Выше показан пример, в котором будет разрешено использовать только `jquery`. Даже если человек установит другие определения, такие как `npm install @types/node`, его глобальные переменные (например, [`process`](https://nodejs.org/api/process.html)) не попадут в ваш код, пока вы не добавите их в опцию типов `tsconfig.json`.
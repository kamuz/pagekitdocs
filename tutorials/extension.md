# Как разрабатывать Pagekit расширение

<p class="uk-article-lead">Данный учебник выполняет все необходимые шаги, необходимые для настройки и разработки полного расширения для управления элементов Todo в админке Pagekit. Вы узнаете о базовых понятиях расширения, контроллерах, маршрутизации, рендерингу видов и фреймворку Vue.js.</p>

## Содержимое


* [Использование модулей](extension.html#extending-pagekit-using-modules)
* [Роутинг и контроллеры](extension.html#routing-and-controller)
* [Рендеринг видов и конфигурация модуля](extension.html#view-rendering-and-module-config)
* [Использование Vue.js](extension.html#using-vue-js-in-a-pagekit-extension)
* [Итоговый пример](extension.html#the-completed-example)

**Внимание!** Весь код примера [доступен на Github](https://github.com/pagekit/example-todo).

## Использование модулей {#extending-pagekit-using-modules}

Как разработчик, вы можете легко расширять функционал тем что уже предлагает Pagekit или создавать собственные **темы** (*англ.* theme) и **расширения** (*англ.* extension) для добавления дополнительного функционала (оба они имеют одинаковые подходы). На этом шаге мы познакомимся с понятием **пакет** (*англ.* package) и узнаем что такое **модуль** (*англ.* module) - это основные понятия в Pagekit.

### Пакеты и модули

Темы и расширения могут отличатся снаружи, но они оба являются **пакетом** (*англ.* package). Пакет - это просто папка, которая связывает множество файлов и содержит мета-данные в файле с именем *composer.json*.

Чтобы мы могли что-нибудь зарегистрировать с помощью системы Pagekit, вам также включить файл с именем *index.php*. Он определяет то что в Pagekit называется **модулем** (*англ.* module). Модуль является наиболее важной составляющей кода Pagekit. Модули  - это автономные блоки, который имеют определённый функционал и объединяются, чтобы сформировать всю систему. Модули используются как в ядре Pagekit так и в других расширениях сторонних разработчиков.

Каждый пакет относится к определенному наименование поставщика (*англ.* vendor name), что становится более очевидным, после того как вы посмотрите на структуру директорий, установив свежую версию Pagekit.

Пакеты ядра являются частью пространства имён `pagekit` и находится в соответствующем подкаталоге внутри `packages/`. Любые другие пакеты сторонних разработчиков (например, ваши) будут иметь свое собственное имя поставщика и поэтому будут находится в отдельной подпапке.

```html
packages/
    pagekit/
        blog/
        theme-one/
    your-vendor/
        your-theme/
        your-extension/
```

В каждом пакете вы должны найти как минимум два файла: *composer.json* и *index.php*.

### Мета данные пакета

Файл *composer.json* содержит необходимые мета-данные для пакета. Это отображается в админке Pagekit и используется, если вы загрузите ваше расширение на маркетплэйс Pagekit.

*packages/pagekit/todo/composer.json*

```json
{
  "name": "pagekit/todo",
  "version": "0.1",
  "type": "pagekit-extension",
  "title": "ToDo management"
}
```

### Определение модуля

Определение модуля - фактически представляет собой массив PHP, у которого установлены определённые свойства. Файл *index.php* всегда должен возвращать валидный массив.

*packages/pagekit/todo/index.php*

```php
<?php

use Pagekit\Application;

return [
  'name' => 'todo',
  'type' => 'extension',
  // called when Pagekit initializes the module
  'main' => function (Application $app) {
    echo "It's alive";
  }
];
```

Для того, чтобы протестировать функциональность, убедитесь в том, что вы включили расширение в панели управления Pagekit. Как только вы его включите, вы увидите сверху экрана надпись `It's alive`.

Этот пример показывает, насколько маленьким может быть полностью функциональный модуль. У него есть доступ к экземпляру объекта `Application`. Используя этот объект, вы можете получать доступ ко всем сервисам, вызывать события и слушать события, вызванные другими модулями.

## Роутинг и контроллер {#routing-and-controller}

После того как была создана базовая структура расширения, следующим шагом, как правило, является регистрация ваших собственных контроллеров и добавление элементов меню в админ-панель. Для этого необходимо добавить несколько дополнительных свойств в определение модуля в вашем *index.php*.

### Добавление контроллера

Контроллер в Pagekit - это простой PHP класс. Любой публичный метод, названный должным образом (например, `someAction()`) будет подключен роутером Pagekit (например, `/todo/some`).

Создадим файл *TodoController.php* со следующим кодом:

*packages/pagekit/todo/src/Controller/TodoController.php*

```php
<?php

namespace Pagekit\Todo\Controller;

/**
 * @Access(admin=true)
 */

class TodoController{

  public function indexAction(){
    return "Yay.";
  }

}
```

Чтобы ограничить доступ к админ-панели и для подключения контроллера к админскому URL, мы используем аннотацию `@Access(admin=true)`. Аннотации - это ключевые слова, которые размещаются в блоках комментариев над классом или методом. Подробнее об аннотациях [читайте здесь](http://pagekit.com/docs/developer/routing#annotations).

Для использования этого контроллера мы должны добавить в автозагрузку наше пространство имён и подключить контроллер к роуту. Добавьте следующие свойства в конфигурационный массив вашего *index.php*:

*packages/pagekit/todo/index.php*

```php
<?php

use Pagekit\Application;

return [
  'name' => 'todo',
  'main' => function(Application $app) {
  },
  // array of namespaces to autoload from given folders
  'autoload' => [
    'Pagekit\\Todo\\' => 'src'
  ],
  // array of routes
  'routes' => [
    // identifier to reference the route from your code
    '@todo' => [
      // which path this extension should be mounted to
      'path' => '/todo',
      // which controller to mount
      'controller' => 'Pagekit\\Todo\\Controller\\TodoController'
    ]
  ],
];
```

Теперь вы можете получить доступ к новосозданному контроллеру по следующему URL `sitename.loc/admin/todo`. Этот URL генерируется из четырёх частей:

1. `sitename.loc` - базовый ULR сайта
1. `/admin` - указывает на админский URL, благодаря использованию анотации `@Access(admin=true)`
2. `/todo` - подключаемый контроллер
3. `indexAction` - дефолтный роут данного URL

Если вы не можете получить доступ к вашему роуту, тогда попробуйте [почистить кэш](http://pagekit.com/docs/user-interface/system#cache). Pagekit кэширует маршруты для лучшей производительности.

### Панель отладки

Чтобы во время разработки, видеть все зарегистрированные маршруты, полезно включить панель отладки (Debug toolbar) в системных настройках Pagekit. Панель отладки появляется внизу экрана и показывает все зарегистрированные маршруты и другую полезную информацию.

Не забудьте выключить её на живом сайте.

### Добавление пункта меню

Для добавления пункта меню используйте свойство `menu` в определении модуля. Добавьте следующее в *index.php*:

*packages/pagekit/todo/index.php*

```php
'menu' => [
  'example' => [
    'label' => 'ToDo',
    'icon' => 'app/system/assets/images/placeholder-icon.svg',
    'url' => '@todo',
  ]
],
```

Обновите бэкенд Pagekit и вы увидиите новый пункт меню, который ссылается на маршрут `@todo`.

## Рендеринг видов и конфигурация модуля {#view-rendering-and-module-config}

Мы рассмотрели основы модулей и маршрутизации. Однако наш первый контроллер возвращал только чистую строку. Давайте рассмотрим рендеринг видов/представлений (view).

### Рендеринг представления из действия контроллера

Для передачи параметров в рендерер вида можно использовать возвращаемый массив в экшене нашего контроллера. Зарезервированный ключ `$view` содержит параметры для рендерера представления, например имя файла представления для рендеринга.

*packages/pagekit/todo/src/Controller/TodoController.php*

```php
public function indexAction(){
  return [
    '$view' => [
      'title' => 'TODO management',
      'name' => 'todo:views/admin/index.php'
    ],
    'message' => 'Hello how\'s it going?'
  ];
}
```


Отображенное представление может быть простым PHP шаблоном. Простые параметры (например, `message`) доступны как переменные PHP (например, `$message`).

*packages/pagekit/todo/views/admin/index.php*

```php
<h1><?php echo $message; ?></h1>
```

### Сокращение ресурсов

Вместо того чтобы ссылаться на полный путь файла, мы можем использовать сокращённый синтаксис, тоесть `packages/pagekit/todo/views/admin/index.php` превращается в `todo:views/admin/index.php`. Так быстрее набирать и проще читать.

Вы можете зарегистрировать сокращения с помощью целевого пути в вашем *index.php*. В нашем примере мы хотим указать `todo:` на текущий путь *index.php*.

*packages/pagekit/todo/index.php*

```php
'resources' => [
  'todo:' => ''
],
```

### Конфигурация модуля

У модуля может быть конфигурационный массив, который содержит любые типы настроек. Мы можем использовать это как простое хранилище наших TODO элементов:

*packages/pagekit/todo/index.php*

```php
'config' => [
  'entries' => [
    ['message' => 'Buy milk', 'done' => false],
    ['message' => 'Write code', 'done' => false],
    ['message' => 'Drink coffee', 'done' => true]
  ]
],
```

Из контроллера мы можем получить доступ к этому конфигу в качестве свойства объекта модуля:

*packages/pagekit/todo/src/Controller/TodoController.php*

```php
use Pagekit\Application as App;

// ...

  $module = App::module('todo');
  $config = $module->config;

// ...
```

Мы можем сохранять изменения конфига модуля в базу данных. Изменения из дефолтного конфига модуля сливаются с сохранёнными изменениями, так что мы всегда имеем валидную конфигурацию доступную в контроллере.

```php
// modifying the module config
App::config('todo')->set('entries', $entries);
```

В итоге *index.php* у нас имеет следующий код:

*packages/pagekit/todo/index.php*

```php
<?php

use Pagekit\Application;

return [
  'name' => 'todo',
  'autoload' => [
    'Pagekit\\Todo\\' => 'src'
  ],
  'routes' => [
    '@todo' => [
      'path' => '/todo',
      'controller' => 'Pagekit\\Todo\\Controller\\TodoController'
    ]
  ],
  'menu' => [
    'example' => [
      'label' => 'ToDo',
      'icon' => 'app/system/assets/images/placeholder-icon.svg',
      'url' => '@todo',
    ]
  ],
  'resources' => [
    'todo:' => ''
  ],
  'config' => [
    'entries' => [
      ['message' => 'Buy milk', 'done' => false],
      ['message' => 'Write code', 'done' => false],
      ['message' => 'Drink coffee', 'done' => true]
    ]
  ],
];
```

Контроллер приобретает такой:

*packages/pagekit/todo/src/Controller/TodoController.php*

```php
<?php

namespace Pagekit\Todo\Controller;
use Pagekit\Application as App;

/**
 * @Access(admin=true)
 */

class TodoController{

  public function indexAction(){

    $module = App::module('todo');
    $config = $module->config;

    return [
      '$view' => [
        'title' => 'TODO management',
        'name' => 'todo:views/admin/index.php'
      ],
      'message' => 'Coffee time?',
      'entries' => $config['entries']
    ];
  }
}
```

В виде мы с помощью `foreach` перебираем элементы TODO.

*packages/pagekit/todo/views/admin/index.php*

```php
<h1><?php echo $message; ?></h1>

<ul>
  <?php foreach ($entries as $entry): ?>
    <li><?php echo $entry['message'] ?>: <?php echo $entry['done'] ? 'Done' : 'Not done' ?></li>
  <?php endforeach; ?>
</ul>
```

## Использование Vue.js{#using-vue-js-in-a-pagekit-extension}

При создании пользовательского интерфейса админ-панели Pagekit, используется JavaScript библиотека Vue.js, которая имеет простой, но мощным API. Поэтому имеет смысл её изучить, чтобы понять хороший ли это выбор для вас. В этой части, мы будем изучать базовые концепции работы с Vue.js внутри Pagekit.

### Передача данных в JavaScript

Ранее, мы рассмотрели, как данные могут быть доступны в PHP контроллерах. Для использования этих данных в JavaScript используйте ключевое слово `$data` для передачи PHP массивов в рендер представления. Pagekit автоматически сконвертирует их в JSON и выведет в раздел `head` сгенерированного HTML.

*packages/pagekit/todo/src/Controller/TodoController.php*


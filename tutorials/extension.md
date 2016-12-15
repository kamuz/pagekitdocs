<p class="uk-article-lead">Данный учебник выполняет все необходимые шаги, необходимые для настройки и разработки полного расширения для управления элементов Todo в админке Pagekit. Вы узнаете о базовых понятиях расширения, контроллерах, маршрутизации, рендерингу видов и фреймворку Vue.js.</p>

## Содержимое


* [Использование модулей](#extending-pagekit-using-modules)
* [Роутинг и контроллеры](#routing-and-controller)
* [Рендеринг видов и конфигурация модуля](#view-rendering-and-module-config)
* [Использование Vue.js](#using-vue-js-in-a-pagekit-extension)
* [Итоговый пример](#the-completed-example)

**Внимание!** Весь код примера [доступен на Github](https://github.com/pagekit/example-todo).

<h2 id="extending-pagekit-using-modules">Использование модулей</h2>

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

<h2 id="routing-and-controller">Роутинг и контроллер</h2>

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

class TodoController
{

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

<h2 id="view-rendering-and-module-config">Рендеринг видов и конфигурация модуля</h2>

Мы рассмотрели основы модулей и маршрутизации. Однако наш первый контроллер возвращал только чистую строку. Давайте рассмотрим рендеринг видов/представлений (view).

### Рендеринг представления из действия контроллера

Для передачи параметров в рендерер вида можно использовать возвращаемый массив в экшене нашего контроллера. Зарезервированный ключ `$view` содержит параметры для рендерера представления, например имя файла представления для рендеринга.

*packages/pagekit/todo/src/Controller/TodoController.php*

```php
public function indexAction()
{
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

    public function indexAction()
    {

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

<h2 id="using-vue-js-in-a-pagekit-extension">Использование Vue.js</h2>

Уже вышла 2 версия Vue.js, но в Pagekit пока что используется 1, поэтому если вам будут не понятны какие-то вещи с Vue.js, смотрите документацию по первой версии.

При создании пользовательского интерфейса админ-панели Pagekit, используется JavaScript библиотека Vue.js, которая имеет простой, но мощным API. Поэтому имеет смысл её изучить, чтобы понять хороший ли это выбор для вас. В этой части, мы будем изучать базовые концепции работы с Vue.js внутри Pagekit.

### Передача данных в JavaScript

Ранее, мы рассмотрели, как данные могут быть доступны в PHP контроллерах. Для использования этих данных в JavaScript используйте ключевое слово `$data` для передачи PHP массивов в рендер представления. Pagekit автоматически сконвертирует их в JSON и выведет в раздел `head` сгенерированного HTML.

*packages/pagekit/todo/src/Controller/TodoController.php*

```php

// ...

public function indexAction()
{
    $module = App::module('todo');

    return [
        '$view' => [
            'name' => 'todo:views/admin/index.php'
        ],
        '$data' => $module->config
    ];
}
```

Это выведет следующее в раздел `<head>`:

```html
<script>var $data = {"entries": [{"message":"Buy milk","done":false},{"message":"Drink coffee","done":true}] };</script>
```

Чтобы не было ошибок, нужно очистить вид от вывода каких-либо переменных, так как мы их не передаём.

### Объединение представления и модели

`vm` (ViewModel) – это экземпляр класса Vue, который синхронизирует данные вашей модели с интерфейсом вашего представления. Это называется **reactivity** и это одна из ключевых фичей Vue. При корректном использовании это помогает держать ваши JavaScript компоненты маленькими и читаемыми.

Вы можете подключить экземпляр Vue к элементу DOM (`el: '#todo'`). Моделью будет всё, что вы передадите в параметр `data`. Чтобы использовать данные из Pagekit, используйте глобальный объект `window.$data`, который отрендерил ваше представление.

Любые созданные `methods` могут быть вызваны из вашего шаблона.

*packages/pagekit/todo/js/todo.js*

```js
$(function(){

    var vm = new Vue({

        el: '#todo',

        data: {
            entries: [{ message: 'Dummy', done: false}],
        },

        methods: {
        }

    });

});
```

Мы создадим разметку шаблона чуть позже.

### Разметка для Vue

Из PHP мы всё ещё рендерим файл представления *views/admin/index.php*. А здесь мы подключим необходимые JavaScript файлы. Для того сделать Vue доступным в вашем скрипте, добавьте зависимость `vue` в качестве третьего параметра. Добавьте следующий код в самом верху файла представления:

*packages/pagekit/todo/views/admin/index.php*

```php
<?php $view->script('todo', 'todo:js/todo.js', 'vue') ?>
```

В вашем HTML подключите DOM элемент, который будет искать экземпляр Vue (`<div id="todo"></div>`). 
Внутри элемента вы можете использовать директивы Vue. Директивы – это определённые ключевые слова, которые говорят Vue, что делать с элементом:

*packages/pagekit/todo/views/admin/index.php*

```html
<div id="todo">
    <p v-if="entry.done">This will be displayed if the item has been done.</p>
</div>
```

С помощью `@click` вы можете забиндить событие клика и вызвать методы вашей модели представления.

Для вывода значений из модели, вы можете использовать фигурные скобки (`{{ entry.message }}`). 

*packages/pagekit/todo/views/admin/index.php*

```php
<?php $view->script('todo', 'todo:js/todo.js', 'vue') ?>

<div id="todo">
    <ul class="uk-list uk-list-space">
        <li v-for="entry in entries" v-bind:class="{'uk-text-muted' : entry.done}">{{ entry.message }}</li>
    </ul>
</div>
```

Теперь у нас должны вывестись данные с нашей модели Vue.

Давайте создадим новое свойство `$data` в нашем контроллере и присвоим ему наш массив `$config` и затем посмотрим что теперь будет выводится внутри тега `<head>`:

*packages/pagekit/todo/src/Controller/TodoController.php*

```php
//...
return [
    '$view' => [
        'title' => 'TODO management',
        'name' => 'todo:views/admin/index.php'
    ],
    '$data' => $config,
    'message' => 'Coffee time?',
    'entries' => $config['entries']
];
//...
```

Если мы заглянем в исходный код, то мы получим следующее:

```html
<script>var $data = {"entries":[{"message":"Buy milk","done":false},{"message":"Write code","done":false},{"message":"Drink coffee","done":true}]};</script>
```

Теперь в нашем скрипте мы меняем модель

*packages/pagekit/todo/js/todo.js*

```js
//...
data: {
    entries: $data.entries,
},
//...
```

И как видим, теперь уже данные выводятся из нашего конфига.

Теперь давайте выведем кнопки `Do` или `Undo` в зависимости от значения свойства `done` нашего конфига.

*packages/pagekit/todo/views/admin/index.php*

```php
<li v-for="entry in entries" v-bind:class="{'uk-text-muted' : entry.done}">
{{ entry.message }}
<button class="uk-button">{{ entry.done ? 'Undo' : 'Do' }}</button>
</li>
```

Забиндим метод `toggle()` по клику на кнопку.

*packages/pagekit/todo/views/admin/index.php*

```html
<button class="uk-button" @click="toggle(entry)">{{ entry.done ? 'Undo' : 'Do' }}</button>
```

Теперь давайте определим этот метод, который фактически будет выполнять роль переключателя.

*packages/pagekit/todo/js/todo.js*

```js
//...
methods: {
    toggle: function(entry) {
        entry.done = !entry.done;
    }
}
//...
```

Теперь по клику у нас будет менятся значение наших кнопок и происходить переключение классов CSS.

Но пока у нас не сохраняется состояние наших TODO элементов, тоесть если мы перезагрузим страницу, то увидим, что значения сбросились до тех, которые указанны в `$config`. Чтобы решить этот вопрос, давайте создадим новый метод в нашем контроллере.

*packages/pagekit/todo/src/Controller/TodoController.php*

```php
//...
/**
 * @Access(admin=true)
 * @Request({"entries": "array"}, csrf=true)
 */

public function saveAction($entries=[])
{
    App::config('todo')->set('entries', $entries);

    return ['success' => true];
}
//...
```

Не забываем использовать аннотации, о которых мы говорили ранее.

Pagekit предоставляет фильтр `trans`, который заменит строку на переведённую альтернативу, если она есть для выбранного языка. Теперь в виде, перед началом нумерованного списка, создаём кнопку **Save**, которая будет вызывать вызывать метод `save()` по клику.

*packages/pagekit/todo/views/admin/index.php*

```html
<button class="uk-button uk-button-primary" @click="save">{{ 'Save' | trans }}</button>
```

Осталось создать метод `save()` в JS файле:

*packages/pagekit/todo/js/todo.js*

```js
save: function() {
    this.$http.post('admin/todo/save', { entries: this.entries },
        function() {
            UIkit.notify('Saved');
        }).error(function() {
            UIkit.notify('Oops');
        })
}
```

Теперь состояние у нас сохраняется, а если мы откроем исходный код, то увидим, что в переменной `$data` у нас находятся текущие состояния элементов TODO.

```js
$data = {"entries":[{
    "message":"Buy milk","done":true},
    {"message":"Write code","done":true},
    {"message":"Drink coffee","done":false}
]}
```

### Узнайте больше о Vue

Vue.js – это мощная библиотека, которая позволяет создавать реактивные веб-интерфейсы. Больше можете узнать в [официальной документации](http://vuejs.org/guide/).

<h2 id="the-completed-example">Полный пример</h2>

В полном примере реализовано сохранение в бэкенде и чистый исходный код. Он объединяет все предыдущие шаги, где было описано создание простого расширения Todo. Результат [доступен на GitHub](https://github.com/pagekit/example-todo).
# Как разработать тему Pagekit

<p class="uk-article-lead">Это учебное пособие научит вас как создавать собственную тему на основе нашей заготовки темы Hello Theme. Вы узнаете о структуре темы и следуя необходимым шагам добавите новые опции и позиции в теме.</p>

<div class="uk-badge">Внимание</div> Завершённая тема [находится на Github](https://github.com/pagekit/example-theme).

## Начало работы

Перед началом обучения, мы предполагаем что у вас уже установлена Pagekit на локальном сервере, а если это не так, то вам нужно, [скачать](https://pagekit.com/download) пакет инсталятора Pagekit и [установить](getting-started/installation) её. После чего зайдите в панель администратора в раздел *System / Theme*, который интегрирован с Marketplace.

[Hello Theme](https://pagekit.com/marketplace/package/pagekit/theme-hello) - является шаблоном-заготовкой для разработки будущих тем, которая включает примеры кода и общую основу чтобы помочь вам начать работу.

Тема Hello не предоставляет никакого стилевого оформления, но при этом является хорошей отправной точкой для разработки собственной темы.

В первую очередь после установки темы нужно ознакомится с файловой структурой. После установки, тема Hello будет находится в папке *packages/pagekit/theme-hello/*. Если вы будете разрабатывать свои собственные темы в будущем, мы рекомендуем установить простую тему, чтобы использовать её для справки, например [Theme One](https://pagekit.com/marketplace/package/pagekit/theme-one).

## Структура

Давайте взглянем на некоторые основные файлы и папки с которыми вы будете иметь дело при разработке собственной темы.

```txt
theme-name/
├─ css/
│  └─ theme.css
├─ js/
│  └─ theme.js
├─ views
│  └─ template.php
├─ composer.json
├─ image.jpg
├─ index.php
├─ CHANGELOG.md
└─ README.md
```

* *css/theme.css* - основной CSS файл с предопределёнными классами
* *js/theme.js* - пустой JS файл для ваших скриптов
* *views/template.php* - рендерит вывод темы; позиции лого, меню, основного контейнера, сайдбара и футера доступны по умолчанию
* *composer.json* - определение темы
* *image.jpg* - скриншот темы
* *index.php* - основной конфиг темы
* *CHANGELOG.md* - логи изменений
* *README.md* - описание темы

### Переименование вашей темы

Если вы просто измените файлы темы Hello, то любые обновления в Marketplace для этой темы перезапишет ваши собственные изменения. Кроме этого, вы вероятно захотите указать для темы собственное название. Если вы решите загрузить собственную тему на Marketplace, то вам также понадобится назвать вашу тему как то иначе. Для этого потребуется три простых шага:

1. Скопировать все файлы с *packages/pagekit/theme-hello* в *packages/your-name/your-theme* (вам нужно будет создать данную папку)
2. Открыть файл *composer.json* и заменить `"name": "pagekit/theme-hello"` на `"name": "your-name/your-theme"`. А также изменить `"title": "Hello"` на `"title": "Your theme"`.
3. Открыть *index.php* и заменить `'name' => 'theme-hello'` на `'name' => 'your-theme'`.

Для простоты, дальше в примерах всё ещё будет использоваться `theme-hello`.

### CSS

Тема Hello не содержит каких либо стилей. Она включает файл *css/theme.css* со списком всех CSS классов, которые рендерят расширения ядра Pagekit без дополнительной стилизации. Вы можете добавить свой собственный CSS к этим классам.

Админка Pagekit и дефолтная тема Pagekit созданны с использованием [фронт-энд фреймворка UIkit](http://getuikit.com/), поэтому вы можете использовать и его для своих проектов. В дальнейших примерах, мы будем создавать нашу собственную тема с использованием UIkit. Если вы будете использовать другой фреймворк или не будете использовать какой-либо фреймворк вообще, основные принципы подключения CSS будут похожи.

Есть множество всевозможных способов настроить файловою структуру вашей темы. Мы можем рекомендовать два подхода. Первый - это простое подключение CSS файлов. Второй - более сложный, но предоставляет гораздо больше гибкости.

#### Простой способ: Подключение CSS файлов

Перейдите на [веб-сайт UIkit](http://getuikit.com/), скачайте и распакуйте последнюю версию фреймворка. UIkit идёт с тремя темами: Default, Gradient и Almost Flat. Для подключения дефолной темы (Default), скопируйте файл *css/uikit.min.css* из архива UIkit и переместите в папку *theme-hello/css/* вашей темы.

Чтобы убедиться что файл Pagekit загружается, откройте основной файл макета темы *theme-hello/views/template.php*

Внутри `<head>` файла макета, мы видим что один CSS файл уже подключён.

```php
<?php $view->style('theme', 'theme:css/theme.css') ?>
```

Теперь добавьте еще одну строку, чтобы подключить CSS файл UIKit. Вам нужно подключить его (*css/uikit.min.css*) выше основного файла CSS.

```php
<?php $view->style('custom-uikit', 'theme:css/uikit.min.css') ?>
<?php $view->style('theme', 'theme:css/theme.css') ?>
```

Теперь ваша тема содержит CSS UIkit. Для добавления ваших собственных CSS правил, вам нужно отредактировать файл *theme-hello/css/theme.css*.

#### Продвинутый способ: Использование Gulp и LESS

В предыдущем разделе мы увидели, как легко можно добавить простые CSS файлы к вашей теме. Если у вас есть опыт создания веб-сайтов, то вы возможно знакомы с более гибкими способами стилизации вашего контента, например применение CSS препроцессора, такого как [LESS] (http://lesscss.org/). Это позволяет использовать такие вещи, как переменные, которые делают ваш код более удобным для чтения и управления.

При использовании UIKit, это даёт вам большое преимущество, потому что вы можете просто изменить переменную, чтобы применить глобальные изменения, например, изменяя основной цвет темы.

Чтобы работать комфортно с предпроцессором LESS, вам необходимо установить несколько инструментов, которыми можно работать через командную строку: `npm`, `gulp` и `bower`. Если они у вас ещё не установленны, тогда вам следует это сделать используя Google, чтобы найти необходимые учебные пособия.

Имеется множество возможных вариантов огранизовать файловую структуру. Мы предлагаем структуру, которая также используется для официальных тем Pagekit. В первый раз это создаёт впечатление, что вы выполняете много шагов при создании темы, но на практике это позволяет хорошо структурировать ваш код, легко кастомизировать и поддерживать UIKit в актуальном состоянии.

#### Шаг №1

В вашей теме, создайте новые файлы *package.json*, *bower.json*, *.bowerrc*, *gulpfile.js* и *less/theme.less*. Вставьте следующий конент в ваши файлы. Если ваша тема названа по другому, замените любый вхождения (occurences) строки `theme-hello` на название вашей темы.

*package.json*. Этот файл определяет все JavaScript зависимости, которые будут устанавливаться во время запуска команды `npm install` и подключает несколько npm пакетов, которые мы будем использовать, например при компиляции LESS в CSS:

*package.json*

```json
{
	"name": "theme-hello",
	"devDependencies": {
		"bower": "*",
		"gulp": "*",
		"gulp-less": "*",
		"gulp-rename": "*"
	 }
}
```

*bower.json* говорит Bower получать новые версии UIkit. В таком случае, вы всегда можете использовать `bower install` для получения текущих LESS файлов от UIkit:

*bower.json*

```json
{
	"name": "theme-hello",
	"dependencies": {
		"uikit": "*"
	},
	"private": true
}
```

*.bowerrc* подключает настройки конфигурации для Bower. По умолчанию, Bower устанавливает всё в директорию */bower_components* внутри директории темы. Давайте изменим эту дефолтную директорию:

*.bowerrc*

```json
{
    "directory": "app/assets"
}
```

*gulpfile.js* содержит все задачи, которые мы можем запускать используя Gulp. Нам нужна только задача для компиляции LESS в CSS. Для удобства мы также добавим задачу `watch`, которая будет автоматически запускать перекомпиляцию LESS при любых изменениях в файлах с расширением _*.less_ внутри директории *less/*:

*gulpfile.js*

```js
var gulp = require('gulp'),
	less = require('gulp-less'),
	rename = require('gulp-rename');

	gulp.task('default', function () {
		return gulp.src('less/theme.less', {base: __dirname})
			.pipe(less({compress: true}))
			.pipe(rename(function (file) {
				// the compiled file should be stored in the css/ folder instead of the less/ folder
				file.dirname = file.dirname.replace('less', 'css');
			}))
			.pipe(gulp.dest(__dirname));
	});

	gulp.task('watch', function () {
		gulp.watch('less/*.less', ['default']);
	});
```

*less/theme.less* это место где будут храниться все стили темы. В первую очередь нам нужно импортировать UIkit, который также скомпилируется при помощи задачи Gulp, которую мы определили выше.

*less/theme.less*

```less
@import "uikit/uikit.less";

// use icon font from system
@icon-font-path: "../../../../app/assets/uikit/fonts";

// your theme styles will follow here...
```

*.gitignore* - это не объязательный файл, который будет нам полезен, когда вы будете хранить историю изменений с помощью Git.


В теме Hello, уже существует версионность файлов. Вы можете добавить новые точки изменений, поэтому это будет выглядеть как продолжение. Возможно вы не захотите коммитить скачанные пакеты с помощью Bower и генерируемые CSS. Только будьте уверенными что включаете сгенерируемые CSS, когда будете загружать данную тему на ваш сервер или на Marketplace Pagekit.

*.gitignore*

```txt
/app/bundle/*
/app/assets/*
/node_modules
/css
.DS_Store
.idea
*.zip
```

#### Шаг №2

После создания файлов конфигурации, перейдите на [UIkit Github репозиторий](https://github.com/uikit/uikit) и скачайте и распакуйте zip архив, после чего найдите папку *theme/default* (или другую тему, если пожелаете). Обратите внимание что вам нужна именно GitHub версия для этого, а не просто CSS версия, которую мы скачивали для простой установки.

#### Шаг №3

Создайте папку */less* внутри вашей темы, скопируйте и вставьте сюда папку темы *default/* и переименуйте её в *uikit/*, чтобы она находилась в *less/uikit/* внутри папки с вашей темой.

#### Шаг №4

В файле стилей мы просто импортировали ядро LESS UIKit, так что теперь оно может быть успешно скомпилировано. Для этого вам необходимо обновить путь импорта в файле вашей темы `less/uikit/uikit.less`. Убедитесь в том, что на 4 строке вы изменили путь к импортируемого файла: `@import "../../app/assets/uikit/less/uikit.less";`.

#### Шаг №5

Откройте вашу тему в новой вкладке консоли/терминала (например `cd pagekit/packages/theme-hello`) и запустите `npm install`, `bower install` и `gulp`.

Сейчас мы уже проделали несколько шагов. Вам нужно убедиться в том, что файловая структура у вас имеет следующий вид (плюс дополнительные файлы темы):

* *app/*
	* *assets/* - загруженные библиотеки - результат команды `bower install`
		* *jquery/*
		* *uikit/*
* *css/*
	* *theme.css* - скомпилированный файл CSS - результат команды `gulp`
* *less/*
	* *uikit/*
		* ... uikit компоненты
		* *uikit.less*
	* *theme.less*
* *node_modules/* - модули Node.js - результат команды `npm install`
* *.bowerrc*
* *bower.json*
* *gulpfile.js*
* *package.json*
* ... другие файлы темы

С помощью вышеуказанных команд и данной файловой структуры мы достигнули следующего:

- **Разделения** файлов стилей темы и кастомизации UIKit. Добавляейте свои собственные стили в *less/theme.less* и выполняйте кастомизацию UIKit в *less/uikit/*
- **Упрощённой настройки UIKit** - настройки каждого UIKit компонента находятся в своем собственном _*.less_ файле. Например, чтобы изменить размер шрифта для `body`, откройте *less/uikit/base.less* и измените значение`@base-body-font-size`, а затем повторно запустить `gulp`. Для использования каких-либо из [дополнительных UIKit компонентов](http://getuikit.com/docs/components.html), откройте *less/uikit.less* и импортируйте дополнительный LESS файл из директории *app/assets*, например, для слайд-шоу (slideshow), добавьте строку: `@import "../../app/assets/uikit/less/components/slideshow.less";` и повторно запустите `gulp.`
- **Вы можете легко обновлять UIKit**: Выполните команду `bower install`, чтобы получить последнюю версию UIKit и запустите команду `gulp` для перекомпиляции ваших LESS файлов.

## Добавление JavaScript

В теме Hello, вы найдёте пустой JavaScript файл *js/script.js*. Сюда вы можете добавлять ваш собственный JavaScript код. В теме Hello этот файл загружается потому что он уже подключён в *template.php*:

```php
<?php $view->script('theme', 'theme:js/theme.js') ?>
```

Когда вы подключаете скрипт, требуется уникальный идентификатор (`theme`) и путь к файлу скрипта (`theme:js/theme.js`). Как видете, вы можете использовать `theme:` в качестве короткой версии файлового пути к директории темы. Чтобы добавить больше JavaScript файлов, просто добавьте несколько строк таким же образом. Убедитесь в том, что вы назначили другие идентификаторы для скриптов. Если бы вы назвали их все `theme`, то будет подключён только последний.

```php
<?php $view->script('theme', 'theme:js/theme.js') ?>
<?php $view->script('plugins', 'theme:js/plugins.js') ?>
<?php $view->script('slideshow', 'theme:js/slideshow.js') ?>
```

В то время как добавление нескольких вызовов `script()` - это самый простой способ, чтобы подключить JavaScript, при этом хелпер `script()` предоставляет более мощные способы подключения файлов JavaScript, которые также могут зависеть друг от друга. Более подробно об этом в следующих разделах.

### Добавление несколько JavaScript файлов с зависимостями

В ранее описанных примерах мы не работали с CSS из UIkit. Если вы хотите использовать UIkit JavaScript компоненты и утилиты, имеет смысл добавить UIkit JavaScript файлы. Обратите внимание, что UIkit требует загрузки jQuery перед использованием UIkit JavaScript компонентов.

Locate the previous line `script('theme', ...)` in the head of `views/template.php` and replace it with the following three lines.

```php
<?php $view->script('theme-jquery', 'theme:app/assets/jquery/dist/jquery.min.js') ?>
<?php $view->script('theme-uikit', 'theme:app/assets/uikit/js/uikit.min.js', 'theme-jquery') ?>
<?php $view->script('theme', 'theme:js/theme.js', 'theme-uikit') ?>
```

Этот пример предполагает чтобы вы используете настройки выше, где Bower установил UIkit и jQuery в папку *app/assets/*. Если вместо этого вы использовали простую установку, просто скачайте и скопируйте *jquery.min.js* и *uikit.min.js* в определённое место в вашей темы и измените к ним пути.

Обратите внимание, как теперь мы добавляем третий параметр, который определяет зависимости скрипта, который мы загружаем. Зависимости - это другие JavaScript файлы, которые должны быть загружены ранее. Таким образом, в данном примере, Pagekit будет загружать данные три файла в следующем порядке: jQuery, UIKit, а затем наш *theme.js*. В данном конкретном примере, этот механизм не кажется очень полезным, потому что скрипты, вероятно, будут загружены точно в том порядке, в котором мы поместили их в `template.php`, не так ли? Но, представьте себе, данные строки расположены в разных файлах и суб-шаблонах вашей темы. Поэтому, определяя зависимости убедитесь в том, что Pagekit всегда загружает файлы в правильном порядке.

Как вы уже могли видеть в выше приведенном примере, зависимости ссылаются, используя уникальный строковый идентификатор (например, `theme-jquery`). В нашем примере этот идентификатор дается скрипту в тот момент, когда он подключается с помощью метода `script()`. Как видите, этот метод принимает три параметра: `$view->script($identifier, $path_to_script, $dependencies)`.

Для того, чтобы убедиться что это работает, откройте `views/template.php` и добавьте следующие строки (`data-uk-*`является префиксом для JavaScript компонентов в UIKit).

```html
<!-- ADD id="up" to body -->
<body id="up">

	<!-- LEAVE existing content ... -->
	...

	<!-- ADD to-top-scroller -->
	<div class="uk-text-center">
       <a href="#up" data-uk-smooth-scroll=""><i class="uk-icon-caret-up"></i></a>
    </div>

	<!-- LEAVE rendering of footer section  -->
    <?= $view->render('footer') ?>

</body>
```

При обновлении браузера, вы увидите небольшую стрелку, которую можно использовать для плавного перехода к верхней части окна браузера. Если браузер не прокручивается плавно, а вместо этого резко перемещается вверх, пожалуйста проверьте что вы все написали также как и в примерах.

### Adding third party scripts, like jQuery

You may ask yourself why we called the included jQuery script `theme-jquery` and not simply `jquery`. In general, it is always useful to prefix your own identifiers, to avoid collisions with other extensions. However, in this specific example, the identifiers `jquery` and `uikit` are already taken, because Pagekit itself includes jQuery and UIkit. This means that you can already load these JavaScript files without including them in your theme. That way, all themes and extensions can share a single version of jQuery (and UIkit, if they use UIkit) to avoid conflicts.

```
<?php $view->script('theme', 'theme:js/theme.js', ['uikit', 'jquery']) ?>
```

As you can see in the example, the third parameter of the `script()` method can also take a list of multiple dependencies. In the earlier example we have only passed in a single string (for example `theme-jquery`). Pass in a string for a single dependency, or a list for multiple depencies — both are possible.

The currently loaded version of jQuery and UIkit depend on the current version of Pagekit. With new releases of Pagekit, the versions of these libraries will continually be updated. While this allows for always having a current version available, a potential downside would be that you need to make sure your code also works for the new versions of these libraries.

## Layout
The central files for your theme's layout are `views/template.php` and `index.php`. The actual rendering happens in the `template.php`.

When you open the `template.php`, you see a very basic setup for you to get started. Let's wrapping a container around our main content and divide the system output and sidebar into a grid.

Around **line 30** the `views/template.php` file renders the sidebar position and the actual content.

```
<!-- Render widget position -->
<?php if ($view->position()->exists('sidebar')) : ?>
    <?= $view->position('sidebar') ?>
<?php endif; ?>

<!-- Render content -->
<?= $view->render('content') ?>
```

Using UIkit's [Block](http://getuikit.com/docs/block.html) and [Utility](http://getuikit.com/docs/utility.html) components, we will create a position block and a container with a fluid width.

It is always a good idea to prefix your own classes, so they will not collide with other CSS you might be using. For example, all UIkit classes are prefixed `uk-`. To distinguish classes or IDs that come from this theme, we will use the prefix `tm-`. Consequently, we add the class and ID `tm-main` to identify the section.

```
<div id="tm-main" class="tm-main uk-block">
    <div class="uk-container uk-container-center">

        <!-- Render widget position -->
        <?php if ($view->position()->exists('sidebar')) : ?>
            <?= $view->position('sidebar') ?>
        <?php endif; ?>

        <!-- Render content -->
        <?= $view->render('content') ?>

    </div>
</div>
```

Now we want the system output and sidebar to actually be side by side. The [Grid](http://getuikit.com/docs/grid.html) component can help us here. For a more semantic layout, we will use `<main>` and `<aside>` elements for the containers.

```
<div id="tm-main" class="tm-main uk-block">
    <div class="uk-container uk-container-center">

        <div class="uk-grid" data-uk-grid-match data-uk-grid-margin>

            <main class="<?= $view->position()->exists('sidebar') ? 'uk-width-medium-3-4' : 'uk-width-1-1'; ?>">

                <!-- Render content -->
                <?= $view->render('content') ?>

            </main>

            <?php if ($view->position()->exists('sidebar')) : ?>
            <aside class="uk-width-medium-1-4">
                <?= $view->position('sidebar') ?>
            </aside>
            <?php endif; ?>

        </div>

    </div>
</div>
```

### Theme elements

To create more complex layouts, you can add your own widget positions, menus and options for both. A regular theme basically consists of widgets, menus and the page content itself.

The page *content* is nothing other than Pagekit's system output. That means that the content of any page you create will be rendered in this area.

*Widgets* are small chunks of content that you can render in different positions of your site, so that they will be displayed in specific locations of your site's markup.

To navigate through any site, you first need to set up a *menu*. For this purpose, Pagekit provides different menu positions that allow users to publish menus in several locations of the theme markup.

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-elements.png" alt="Typical theme elements">
    <figcaption class="uk-thumbnail-caption">A typical site layout consists of the main navigation, the page content and several widget positions</figcaption>
</figure>

However, your theme needs to register all positions before. This happens in the `index.php` file through the `menus` and `positions` properties. These contain arrays of the position name and a label, which is displayed in the admin panel. This file is also used to load additional scripts and much more.

## Navbar

One of the first things you will want to render in your theme is the main navigation.

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-menu-unstyled.png" alt="Main navigation unstyled">
    <figcaption class="uk-thumbnail-caption">By default, Hello theme renders menu items in a very simple vertical navigation.</figcaption>
</figure>

#### Step 1

Hello theme comes with the predefined *Main* menu position. When adding a new position it needs to be defined by an identifier (i.e. `main`) and a label to be displayed to the user (i.e. *Main*).

```
'menu' => [

    'main' => 'Main',

]
```

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-menu-position.png" alt="Menu position in Site Tree">
    <figcaption class="uk-thumbnail-caption">A menu can be published to the defined positions in the Pagekit Site Tree.</figcaption>
</figure>


#### Step 2

With the concept of modularity in mind, Pagekit renders position layouts in separate files. For the navigation, create the `views/menu-navbar.php` file containing the following:

```
<?php if ($root->getDepth() === 0) : ?>
<ul class="uk-navbar-nav">
<?php endif ?>

    <?php foreach ($root->getChildren() as $node) : ?>
    <li class="<?= $node->hasChildren() ? 'uk-parent' : '' ?><?= $node->get('active') ? ' uk-active' : '' ?>" <?= ($root->getDepth() === 0 && $node->hasChildren()) ? 'data-uk-dropdown':'' ?>>
        <a href="<?= $node->getUrl() ?>"><?= $node->title ?></a>

        <?php if ($node->hasChildren()) : ?>

            <?php if ($root->getDepth() === 0) : ?>
            <div class="uk-dropdown uk-dropdown-navbar">
            <?php endif ?>

                <?php if ($root->getDepth() === 0) : ?>
                <ul class="uk-nav uk-nav-navbar">
                <?php elseif ($root->getDepth() === 1) : ?>
                <ul class="uk-nav-sub">
                <?php else : ?>
                <ul>
                <?php endif ?>
                    <?= $view->render('menu-navbar.php', ['root' => $node]) ?>
                </ul>

            <?php if ($root->getDepth() === 0) : ?>
            </div>
            <?php endif ?>

        <?php endif ?>

    </li>
    <?php endforeach ?>

<?php if ($root->getDepth() === 0) : ?>
</ul>
<?php endif ?>
```

#### Step 3

To render the actual navbar in the `template.php` file, create a `<nav>` element and add the `.uk-navbar` class. Load the `menu-navbar.php` file inside the element as follows (you can remove the existing block where `$view->menu('main')` is rendered).

```
<nav class="uk-navbar">

    <?php if ($view->menu()->exists('main')) : ?>
    <div class="uk-navbar-flip">
        <?= $view->menu('main', 'menu-navbar.php') ?>
    </div>
    <?php endif ?>

</nav>
```

The main menu should now automatically be rendered in the new *Navbar* position.

#### Step 4

You will probably also want the logo to appear inside the navbar. So wrap the `<nav>` element around the logo as well and add the `.uk-navbar-brand` class, to apply the appropriate spacing.

```
<nav class="uk-navbar">

    <!-- Render logo or title with site URL -->
    <a class="uk-navbar-brand" href="<?= $view->url()->get() ?>">
        <?php if ($logo = $params['logo']) : ?>
            <img src="<?= $this->escape($logo) ?>" alt="">
        <?php else : ?>
            <?= $params['title'] ?>
        <?php endif ?>
    </a>

    <?php if ($view->menu()->exists('main')) : ?>
    <div class="uk-navbar-flip">
        <?= $view->menu('main', 'menu-navbar.php') ?>
    </div>
    <?php endif ?>

</nav>
```

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-navbar.png" alt="Horizontal navbar">
    <figcaption class="uk-thumbnail-caption">With our changes, menu items are now rendered in a horizontal navbar.</figcaption>
</figure>

### Adding theme options

Pagekit uses [Vue.js](https://vuejs.org/) to build its administration interface. Here is a [Video tutorial](https://youtu.be/3gPGyhTroSA?list=PL2rU5GxE-MQ7aYIcxTmDh4-BTHRM-9lto) on Vue.js in Pagekit.

A frequently requested feature is for the navbar to remain fixed at the top of the browser window when scrolling down the site. In the following steps, we are going to add this as an option to our theme.

#### Step 1

First, we need to create the folder `app/components` and in it the file `site-theme.vue`. Settings stored in this file affect the entire website and can be found under *Theme* in the *Settings* tab of the Site Tree. They cannot be applied to a specific page.

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-site-tree.png" alt="Site Tree">
    <figcaption class="uk-thumbnail-caption">You can add any kind of Settings screen to the admin area.</figcaption>
</figure>

In the freshly created file `site-theme.vue` we add the option which will be displayed in the Pagekit administration.

```
<template>

    <div class="uk-margin uk-flex uk-flex-space-between uk-flex-wrap" data-uk-margin>
        <div data-uk-margin>
            <h2 class="uk-margin-remove">{{ 'Theme' | trans }}</h2>
        </div>
        <div data-uk-margin>
            <button class="uk-button uk-button-primary" type="submit">{{ 'Save' | trans }}</button>
        </div>
    </div>

    <div class="uk-form uk-form-horizontal">

        <div class="uk-form-row">
            <label for="form-navbar-mode" class="uk-form-label">{{ 'Navbar Mode' | trans }}</label>
            <p class="uk-form-controls-condensed">
                <label><input type="checkbox" v-model="config.navbar_sticky"> {{ 'Sticky Navigation' | trans }}</label>
            </p>
        </div>

    </div>

</template>
```

#### Step 2

Now we still have to make this option available in the Site Tree. To do so, we can create a *Theme* tab in the interface by adding the following script below the option in the `site-theme.vue` file.

```
<script>

    module.exports = {

        section: {
            label: 'Theme',
            icon: 'pk-icon-large-brush',
            priority: 15
        },

        data: function () {
            return _.extend({config: {}}, window.$theme);
        },

        events: {

            save: function() {

                this.$http.post('admin/system/settings/config', {name: this.name, config: this.config}).catch(function (res) {
                    this.$notify(res.data, 'danger');
                });

            }

        }

    };

    window.Site.components['site-theme'] = module.exports;

</script>
```

#### Step 3

To load the script and add the option to the Site Tree, you also need to add the following to the `index.php` file.

```
'events' => [

    'view.system/site/admin/settings' => function ($event, $view) use ($app) {
        $view->script('site-theme', 'theme:app/bundle/site-theme.js', 'site-settings');
        $view->data('$theme', $this);
    }

]
```

#### Step 4

Add the default setting for the navbar mode in the `index.php` file.

```
'config' => [
    'navbar_sticky' => false
],
```

#### Step 5

Vue components need to be compiled into JavaScript using Webpack. To do so, create the file `webpack.config.js` in your theme folder:

```
module.exports = [

    {
        entry: {
            "site-theme": "./app/components/site-theme.vue"
        },
        output: {
            filename: "./app/bundle/[name].js"
        },
        module: {
            loaders: [
                { test: /\.vue$/, loader: "vue" }
            ]
        }
    }

];
```

#### Step 6

After that, run the command `webpack` inside the theme folder and `site-theme.vue` will be compiled into `/bundle/site-theme.js` with the template markup converted to an inline string.

If you haven't worked with it before, you will quickly need to [install webpack globally](http://webpack.github.io/docs/installation.html) and then also run the following in your project directory to have a local webpack version and a Vue compiler available.

```
npm install webpack vue-loader vue-html-loader babel-core babel-loader babel-preset-es2015 babel-plugin-transform-runtime --save-dev
```

Whenever you now apply changes to the Vue component, you need to run this task again. Alternatively, you can run `webpack --watch` or `webpack -w` which will stay active and automatically recompile whenever you change the Vue component. You can quit this command with the shortcut *Ctrl + C* For more information on Vue and Webpack, take a closer look at [this doc](https://pagekit.com/docs/developer/vuejs-and-webpack).

#### Step 7

Lastly, we want to load the necessary JavaScript dependencies in the head of our `views/template.php` file. In our case we are using the [Sticky component](http://getuikit.com/docs/sticky.html) from UIkit. Since it is not included in the framework core, it needs to be loaded explicitely. YOu can do that by adding a dependency to the sticky component when loading the theme's JavaScript.

```
<?php $view->script('theme', 'theme:js/theme.js', ['uikit-sticky']) ?>
```

Now all you need to do is render the option into the actual navbar in `template.php`.

```
<nav class="uk-navbar uk-position-z-index" <?= $params['navbar_sticky'] ? ' data-uk-sticky' : '' ?>>
```

In the admin area, go to *Site &gt; Settings &gt; Theme*  and enable the _Sticky Navigation_ option to see it take effect on your website.

## Widgets

Widget positions allow users to publish widgets in several locations of your theme markup. They appear in the *Widgets* area of the Pagekit admin panel and can be selected by the user when setting up a widget.

#### Step 1

To render a new widget position, you first need to register it with the `index.php` file. For example, if we want a create a new *Top* position, we will define it through the `positions` property.

```
'positions' => [

    'sidebar' => 'Sidebar',
    'top' => 'Top'

],
```

#### Step 2

Now that we have made the new position known to Pagekit, we need also to create a position renderer. We could skip this step and use a default renderer, but then all widgets would simply render below each pther. So to lay out the widgets in a grid, create the file `views/position-grid.php`:

```
<?php foreach ($widgets as $widget) : ?>
<div class="uk-width-medium-1-<?= count($widgets) ?>">

    <div class="uk-panel">

        <h3 class="uk-panel-title"><?= $widget->title ?></h3>

        <?= $widget->get('result') ?>

    </div>

</div>
<?php endforeach ?>
```

#### Step 3

To render the new position in the theme's markup, we need to add it to the `views/template.php` file, after the closing `</nav>` tag:

```
<?php if ($view->position()->exists('top')) : ?>
<div id="top" class="tm-top">
    <div class="uk-container uk-container-center">

        <section class="uk-grid uk-grid-match" data-uk-grid-margin>
            <?= $view->position('top', 'position-grid.php') ?>
        </section>

    </div>
</div>
<?php endif ?>
```

Now can select *Top* for any widget that you want to render in the newly created position.

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-select-position.png" alt="Widget position">
    <figcaption class="uk-thumbnail-caption">Select the new position when creating a widget.</figcaption>
</figure>

### Adding position options
The example before added configuration options which applied for the whole site. We can also extend the Site Tree with configuration that only applies for a certain page. Let us add the option to apply a different background color to our new Top position.

#### Step 1

First, we need to create the file `node-theme.vue` inside the folder `app/components`. Here we add the option which will be displayed in the Pagekit administration. Settings that are stored in this file can be applied to each page separately by entering the appropriate item in the site tree and clicking the *Theme* tab.

```
<template>

    <div class="uk-form-horizontal">

        <div class="uk-form-row">
            <label for="form-top-style" class="uk-form-label">Top {{ 'Position' | trans }}</label>
            <div class="uk-form-controls">
                <select id="form-top-style" class="uk-form-width-large" v-model="node.theme.top_style">
                    <option value="uk-block-default">{{ 'Default' | trans }}</option>
                    <option value="uk-block-muted">{{ 'Muted' | trans }}</option>
                </select>
            </div>
        </div>

    </div>

</template>
```

#### Step 2

Now we still have to make this option available in the Site Tree. To do so, we can create a *Theme* tab in the interface by adding the following to the `node-theme.vue` file.

```
<script>

    module.exports = {

        section: {
            label: 'Theme',
            priority: 90
        },

        props: ['node']

    };

    window.Site.components['node-theme'] = module.exports;

</script>
```

#### Step 3

In the chapter about theme options, we inserted the event listener to the `index.php` file in order to load the script and add the option to the Site Tree. Now we need to do the same thing for the site setting. The complete `events` section should then look like this.

```
'events' => [

    'view.system/site/admin/settings' => function ($event, $view) use ($app) {
        $view->script('site-theme', 'theme:app/bundle/site-theme.js', 'site-settings');
        $view->data('$theme', $this);
    },

    'view.system/site/admin/edit' => function ($event, $view) {
        $view->script('node-theme', 'theme:app/bundle/node-theme.js', 'site-edit');
    }

]
```

#### Step 4

The default setting for the widget position also needs to be added in the `index.php`.

```
'node' => [

    'top_style' => 'uk-block-muted'

],
```

#### Step 5

In the chapter about theme options we created the file `webpack.config.js`. Our `node-theme.vue` file also needs to be registered to be compiled into JavaScript.

```
entry: {
    "node-theme": "./app/components/node-theme.vue",
    "site-theme": "./app/components/site-theme.vue"
},
```

#### Step 6

Now you can run the command *webpack* inside the theme folder and `node-theme.vue` will be compiled into `app/bundle/node-theme.js`.

#### Step 7

Lastly, to actually render the chosen setting into the widget position, we need to add the `.uk-block` class and the style parameter to the position itself in the `template.php` file.

```
<div id="top" class="tm-top uk-block <?= $params['top_style'] ?>">
```

#### Step 8

In the site tree, you now see a _Theme_ tab when editing a page. Here you can configure the new option. This configuration only applies for this specific page.

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-node.png" alt="Node options added by the theme">
    <figcaption class="uk-thumbnail-caption">Node options added by the theme.</figcaption>
</figure>

### Adding widget options
You can also add specific options to widgets themselves. In this case, we would like to provide a panel style option that can be selected for each widget.

<figure class="uk-thumbnail">
    <img src="assets/tutorial-theme-widget-options.png" alt="Widget options">
    <figcaption class="uk-thumbnail-caption">A theme can add any kind of options to the Widget editor.</figcaption>
</figure>

#### Step 1

First, we need to create a `app/components/widget-theme.vue` file inside the folder `app/components`. This renders the select box from which we will be able to choose the widget's style.

```
<template>

    <div class="uk-form-horizontal">

        <div class="uk-form-row">
            <label for="form-theme-panel" class="uk-form-label">{{ 'Panel Style' | trans }}</label>
            <div class="uk-form-controls">
                <select id="form-theme-panel" class="uk-form-width-large" v-model="widget.theme.panel">
                    <option value="">{{ 'None' | trans }}</option>
                    <option value="uk-panel-box">{{ 'Box' | trans }}</option>
                    <option value="uk-panel-box uk-panel-box-primary">{{ 'Box Primary' | trans }}</option>
                    <option value="uk-panel-box uk-panel-box-secondary">{{ 'Box Secondary' | trans }}</option>
                    <option value="uk-panel-header">{{ 'Header' | trans }}</option>
                </select>
            </div>
        </div>

    </div>

</template>
```

#### Step 2

Now we still have to make this option available in the widget administration. To do so, we can create a *Theme* tab in the interface by adding the following to the `widget-theme.vue` file.

```
<script>

    module.exports = {

        section: {
            label: 'Theme',
            priority: 90
        },

        props: ['widget', 'config']

    };

    window.Widgets.components['theme'] = module.exports;

</script>
```

#### Step 3

To make the option available in the widget administration, we can create a *Theme* tab to the interface by adding the following to the `events` section in the file `index.php`.

```
'view.system/widget/edit' => function ($event, $view) {
    $view->script('widget-theme', 'theme:app/bundle/widget-theme.js', 'widget-edit');
}
```

#### Step 4

The default setting for the widget also needs to be added in the `index.php`.

```
'widget' => [

    'panel' => ''

],
```

#### Step 5

Add the `widget-theme` entry to the `webpack.config.json` file, so it will be compiled to JavaScript (don't forget to run `webpack` in the theme folder afterwards).

```
entry: {
    "node-theme": "./app/components/node-theme.vue",
    "site-theme": "./app/components/site-theme.vue",
    "widget-theme": "./app/components/widget-theme.vue"
},
```

#### Step 6

Now all we need to do is render the chosen setting into the widget in the `position-grid.php` file.

```
<div class="uk-panel <?= $widget->theme['panel'] ?>">
```

#### Step 7

When editing a widget, you will now see a _Theme_ tab on top of the editor, where you can access the _Panel Style_ configuration that we have just added.

## Overwrite system views

A theme can also customize the way how Pagekit renders static pages and blog posts. You simply overwrite system view files to apply your own layout. To do so, create corresponding folders inside your theme to mimic the original structure of the Pagekit system and put the template files there:

File                         | Original view file                       | Description
---------------------------- | ---------------------------------------- | ------------------------
`views/system/site/page.php` | `/app/system/site/views/page.php`        | Default static page view
`views/blog/post.php`        | `/packages/pagekit/blog/views/post.php`  | Blog post single view
`views/blog/posts.php`       | `/packages/pagekit/blog/views/posts.php` | Blog posts list view

To understand which variables are available in these views, look at the markup in the original view file.

## Wrapping up

In this tutorial, you have learned the basic knowledge and tools to create themes for Pagekit. Let us summarize which topics we have covered.

**Note** The [completed theme](https://github.com/pagekit/example-theme) can be found on Github.

- You are now familiar with the **file structure** of a Pagekit theme. The main entry point for configuration and custom code is the theme's `index.php`, the main template file is located at `views/template.php` inside the theme.
- We have presented an approach for a modern **front-end workflow** with tools such as LESS, Gulp and Bower. You can customize this setup to your own preferences.
- To **add JavaScript** to your theme, you can add your own code and include the script files in your template. You can also add third party libraries like UIkit and jQuery that help you to add interaction to your website.
- **Widgets and menus** are managed from the Pagekit admin area and are rendered in special positions of your theme. You have learned how to create these positions and how to change the default widget rendering.
- To make sure your theme is customizable from the admin area, you can add your own **settings screens**. These can be added to the Site Tree, to the Site settings and to the Widget editor.
- To **overwrite system views**, your theme can include custom view files to alter the way static pages and blog posts are rendered.

With these skills you are now in a position to create Pagekit themes, both for client projects and also for the Pagekit marketplace.

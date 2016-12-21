# Файл конфигурации

<p class="uk-article-lead">Файл конфигурации автоматические создаётся во время установки Pagekit. Если вы хотите вручную изменить настройки конфигурации, то вам поможет эта статья, которая объяснит синтаксис и содержимое данного файла.</p>

Файл конфигурации Pagekit расположен в корневой директории с именем *config.php*. Как правило, вам не нужно возиться с этим файлом после того как он был создан инсталятором. Обычно конфигурацию изменяют в разделе *Система / Настройки* в панели управления Pagekit.

Но иногда этой файл всё таки нужно редактировать вручную, например во время появления каких-либо ошибок во время установки или переноса существующего сайта на новый сервер.

Ниже, вы увидите пример конфигурации с наиболее распостранёнными настройками.

Обычно у вас имеется только одно соединение с базой данных. Но, для примера, включенны обе конфигурации (`sqlite`, `mysql`). При этом, будет использоваться только соединение переданное в `default` (в данном случае `sqlite`).

<div class="uk-badge">Примечание</div> Я не сторонник переводить комментарии, но я надеюсь что и без перевода понятно какая настройка за что отвечает.

```php
'database' => [
  'default' => 'sqlite', // default database connection
  'connections' => [ // array of database connections
    'sqlite' => [ // database driver name, here: sqlite
      'prefix' => 'pk_', // prefix in front of every table
    ],
    'mysql' => [ // database driver name, here: mysql
      'host' => 'localhost', // server host name
      'user' => 'user', // server user name
      'password' => 'pass', // server user password
      'dbname' => 'pagekit', // database name
      'prefix' => 'pk_' // prefix in front of every table
    ],
  ]
],
'system' => [
  'secret' => 'secret' // a secret string generated during installation
],
'system/cache' => [
  'caches' => [
    'cache' => [
      'storage' => 'auto' // the cache method to be used, if enabled
    ]
  ],
  'nocache' => false // the cache state - disable entirely by setting to true
],
'system/finder' => [
  'storage' => '/storage' // relative path to a folder used for uploads, cache etc.
],
'application' => [
  'debug' => false // debug mode state, enable while developing to get debug output
],
'debug' => [
  'enabled' => false // debug toolbar state, enable to get information, about requests, routes etc.
]
```
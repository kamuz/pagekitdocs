# Требования

<p class="uk-article-lead">Pagekit весьма не требовательный и как правило, всё что ему нужно, уже имеется на любом современном сервере.</p>

## Системные требования

Убедитесь, что ваш сервер соответствует следующим требованиям.

* Apache 2.2+ или nginx
* MySQL Server 5.1+ или SQLite 3
* PHP Version 5.5.9+

## PHP расширения

В дополнение к этому, необходимо чтобы были включены следующие PHP расширения:

* [JSON](http://php.net/manual/book.json.php)
* [Session](http://php.net/manual/book.session.php)
* [ctype](http://php.net/manual/book.ctype.php)
* [Tokenizer](http://php.net/manual/book.tokenizer.php)
* [SimpleXML](http://php.net/manual/book.simplexml.php)
* [DOM](http://php.net/manual/book.dom.php)
* [mbstring](http://php.net/manual/book.mbstring.php)
* [PCRE](http://php.net/manual/book.pcre.php) 8.0+
* [ZIP](http://php.net/manual/book.zip.php)
* [PDO](http://php.net/manual/book.pdo.php) с драйвером [MySQL](http://php.net/manual/ref.pdo-mysql) или [SQLite](http://php.net/manual/ref.pdo-sqlite).

Хотя не объязательно, но мы также настроятельно рекомендуем включить следующие PHP расширения:

* [cURL](http://php.net/manual/book.curl.php)
* [iconv](http://php.net/manual/book.iconv.php)
* [XML Parser](http://php.net/manual/book.xml.php)
* [APC](http://php.net/manual/book.apc.php) или [XCache](http://xcache.lighttpd.net/) для кэширования.

## Требования к браузерам

Админ-панель Pagekit совместима с Internet Explorer 10+. Chrome, Firefox, Opera и Safari автоматически обновляются, поэтому мы поддерживаем только последние версии этих браузеров.

Поддержка браузерами, также зависит от используемой вами темы оформления.
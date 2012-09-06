# Скелет приложения

Набор методов, которые позволяют организовать код.

## namespace()

> .namespace( namespace )
>
> .namespace( namespace, origin )

Модули и функции приложения не захламляют глобальное пространство имен, а
группируются в специально отведенной для них переменной. Базовое имя задается
в константе **NS**. Для совместимости с минимизаторами кода она определяется
как параметр замыкания.

    App.namespace("App.Module1");

Можно использовать укороченный синтаксис, чтобы получить аналогичный результат:

    App.namespace("Module1");

Базовое имя будет подставляться автоматически, если он не будет найдено до
первого разделителя.

Второй опциональный параметр задает исходный объект, который нужно расширить
существующими полями и методами, если они уже есть.

    App.namespace("App.Module1", {
        field: "field 1",
        method: function (a, b) {
            return a + b;
        }
    });

Метод `namespace` возвращает новый объект, если указанное имя не было найдено,
или объект, который был передан во втором параметре. Он так же может быть
вызван и без параметров. В этом случае он вернет глобальный объект приложения.

    App.namespace() === App    /* true */

## defaults()

> .defaults( namespace )
>
> .defaults( namespace, object )
>
> .defaults( namespace, field, stub )

Вспомогательный метод для доступа к хранилищу настроек компонент приложения.
Его поведение полностью аналогично `namespace` за исключением того, что данные
сохраняются и извлекаются не из глобальной переменной, а из локальной,
которая «спрятана» в замыкании.

    App.defauls("Module1", {url: "/search/"});

Если `defaults` вызвать без параметров, то можно получить доступ ко всем
сохраненным ранее данным.

Основное применение этого метода заключается в том, что приложение может
объявлять настройки отдельных компонент до их загрузки и инициализации.
В дальнейшем такие компоненты при инициализации должны будут обратиться к
этому хранилищу и получить их.

Эти методом можно пользоваться для безопасного получения значений ключей,
которые могут отсутствовать по каким-либо причинам.

    App.defaults("App.Text", {
        error: {
            not_found: "Not found",
            connection_lost: "Connection lost"
        },
        notice: {
            updated: "Updated"
        }
    }

    var msg1 = App.defaults("Text", "error.not_found", "Not found");
    var msg2 = App.defaults("Text", "error.unknown", "Unknown error");

Первый ключ найден в хранилище и вернется значение, которое там определено.
А второго ключа в хранилище нет. В этом случае будет использовано значение по умолчанию.

Для однотипных действий в приложении можно сделать обертку для них.

    App.namespace("App.Text", {
        getError: function (key, stub) {
            return App.defaults("Text.error", key, stub);
        },
        getNotice: function (key, stub) {
            return App.defaults("Text.notice", key, stub);
        }
    });

    var msg = App.Text.getError("not_found", "Not found");

## register()

> .register( module )
>
> .register( [module1, module2, ...] )
>
> .register( [module1, ...], callback )
>
> .register( [module1, ...], "version" )

Добавляем описание компоненты приложения

    App.register({
        // имя компоненты, которое в дальнейшем будет использоваться
        // при построении списка зависимостей
        name: "module1",
        // путь до файла компоненты
        path: "/scripts/module1.js",
        // список зависимостей. порядок следования определяет приоритет загрузки
        // может быть пустым или отсутствовать вообще
        requires: ["jQuery", "common", "module2"]
    });

Компоненты можно добавлять как по одной, так и массивом. В поле `path` так же
можно передать массив строк, определяющих адреса всех обязательных файлов компоненты.

    App.register([
        {
            name: "showroom",
            path: ["css/showroom.css", "js/showroom.js"],
            requires: ["jquery-imageload", "showroom-settings"]
        }, {
            name: "showroom-settings",
            path: "js/showroom-settings.js"
        }, {
            name: "jquery-imageload",
            path: "js/libs/jquery.imageload.js"
        }
    ]);

Вторым параметром в метод можно передать функцию, которая будет изменять информацию о модуле
перед фактическим добавлением её в реестр.

    var buildVersion = "20120725";
    App.register(modules, function (module, name, index) {
        // this === module
        module.path = module.path + "?" + buildVersion;
        return module;
    });

В этом примере все URL ресурсов будут дополнены GET-параметром с номером версии,
сбрасывающим кеширование файлов на стороне клиента.

Вместо функции может быть передана строка. В этом случае будет использован встроенный
механизм добавления указанной версии в имя файла. Эта строка будет добавлена перед «расширением» ресурса.
Чтобы оставить путь до ресурса без изменений, в начало строки следует добавить символ `!`.

    App.register([
        {
            name: "showroom",
            path: ["css/showroom.css", "js/showroom.js"]
        }, {
            name: "jquery",
            path: "!http://code.jquery.com/jquery-1.8.1.min.js"
        }
    ], "20120906");

В приведённом примере в результате будут сформированы следующие пути:

    css/showroom.20120906.css
    js/showroom.20120906.js
    http://code.jquery.com/jquery-1.8.1.min.js

Метод всегда возвращает объект со всеми описаниями компонент приложения.
Не рекомендуется изменять его содержимое, но можно использовать для различного
рода проверок и тестирования фреймворка.

Вызвав его без параметров, можно получить объект-хранилище без каких
либо предварительных изменений в нем.

    var storage = App.register();

## calculate()

> calculate( moduleName )

Построение списка компонент, от которых зависит данный модуль

    var list = App.calculate("module1");

В результате работы получаем массив со значениями `path`, отсортированный в порядке
значимости.

Как только компонента оказывается в числе прочих зависимостей, то она помечается
специальным флагом и больше не выдается в списке. Это исключает двойные загрузки.

    App.calculate("module1").length === 0   /* true */

Чтобы не помечать такие компоненты, метод `calculate` можно вызвать с дополнительным
параметром `true`. Использовать его стоит только для отладки и тестирования.

## load() и bootstrap()

> load( needs )
>
> bootstrap( needs )

Методы `load` и `bootstrap` выполняют роль оберток над методом `load` загрузчика
[yepnope.js](http://yepnopejs.com/).

Ключевая особенность методов состоит в том, что они формируют и управляют
внутренней очередью загрузки приложения. Различные компоненты приложения могу
инициировать загрузку на любой стадии выполнении скриптов. Но так как в них
могут присутствовать жесткие зависимости, например, от [jQuery](http://jquery.com/),
то сперва нужно позаботиться о загрузке таких общих для всего приложения ресурсов.

Метод `bootstrap` загружает общие для всего приложения ресурсы. После этого
начинается загрузка все остальных ресурсов, которые успели встать в очередь
с помощью метода `load`.

    // ставим компонент шоурума в очередь на загрузку.
    // реальная загрузка начнется только после загрузки базовых скриптов
    App.load({
        load: App.calculate("showroom"),
        complete: function () {
            // инициализируем шоурум
            $(function () {
                ...
            });
        }
    });

    // стартуем загрузку базовых скриптов, например, jQuery
    App.bootstrap({
        load: "http://code.jquery.com/jquery-1.7.1.min.js",
        complete: function () {
            // в этом месте уже можно декорировать страницу
            // с применением jQuery.
        }
    });

Вызовы `load` после выполнения `bootstrap` будут выполняться без какой-либо
задержки в соответствии с логикой работы *yepnope.js*.

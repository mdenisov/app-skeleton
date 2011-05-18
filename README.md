# Скелет приложения

Набор методов, которые позволяют организовать код.

## namespace()

Модули и функции приложения не захламляют глобальное
пространство имен, а группируются в специально отведенной
для них переменной. Базовое имя задается в константе **NS**.

    APP.namespace("APP.Module1");

Можно использовать укороченный синтаксис, чтобы получить аналогичный результат:

    APP.namespace("Module1");

Второй опциональный параметр задает исходный объект, который
нужно расширить существующими полями и методами, если они уже есть.

    APP.namespace("APP.Module1", {
        field: "field 1",
        method: function (a, b) {
            return a + b;
        }
    });

## module()

Функция возвращает новый объект, поля и методы которого унаследованы
от исходного. Подробнее о схеме работы можно прочесть в статье
Дугласа Крокфорда «[Наследование через прототип](http://javascript.crockford.com/prototypal.html)».

    var M = APP.module({
        method: function (a, b) {
            return a + b;
        }
    });

    APP.namespace("APP.Module", M);

# APP.Dependency

Модуля для хранения зависимостей компонент приложения друг от друга

## add()

Добавляем компоненту

    APP.Dependency.add({
        // имя компоненты, которое в дальнейшем будет использоваться
        // при построении списка зависимостей
        name: "module1",
        // путь до файла компоненты
        path: "/scripts/module1.js",
        // список зависимостей. порядок следования определяет приоритет загрузки
        // может быть пустым или отсутствовать вообще
        requires: ["jQuery", "common", "module2"]
    });

## calculate()

Построение списка компонент, от которых зависит данный модуль

    var list = APP.Dependency.calculate("module1");

В результате работы получаем массив со значениями `path`, отсортированный в порядке
значимости.

[![Build & Test](https://github.com/TonyCardio/CodeVsZombies/actions/workflows/dotnet.yml/badge.svg)](https://github.com/TonyCardio/CodeVsZombies/actions/workflows/dotnet.yml)

# starter-kit
Заготовка для написания ботов CodinGame.com

### Инструкция по использованию

1. Сделайте копию репозитория для конкретной игры.
2. Пишите код в проекте bot. Типичное начало контеста — вы копируете код ввода данных в класс StateReader.ReadState и если надо в ReadInit (если не надо — просто удаляете его). И наполняете State и StateInit всякими полями / свойствами.
3. Тесты запускайте в проекте tests.
4. Обобщённые алгоритмы ищите в lib.
5. builder сделайте стартовым проектом, при каждом запуске он собирает все файлы bot и lib в один файл и копирует его содержимое в буфер обмена.
6. После окончания соревнования, все новое полезное обобщённое, что можно было бы поместить в lib, оформите в виде pull request к этому репозиторию. 
Не присылайте реализацию обобщённых алгоритмов поиска, только вспомогательные примитивы, которые могут оказаться полезными в будущем.


## Что тут есть?

###  Общая архитектура решения

1. ISolver — общая абстркция для алгоритмов поиска. 
2. State — состояние игры, StateInit — часть состояния, которое неизменно и вводится перед началом игры.
3. StateReader — чтение State и StateInit.
4. App — точка входа. Создает Solver, организует ввод и вывод.

### ISolver

Абстракция позволяет комбинировать разные солверы друг с другом. Некоторым солверам нужен другой солвер, для работы.
Можно делать солверы обёртки. Например, вот так можно добавить логгирование 10 лучших найденных решений к любому солверу:

`var solver = new MyCustomSolver(new SomeOtherSolver(...), ...).WithLogging(bestSolutionsCountToLog: 10);`

Многие фичи лучше добавлять не в конкретный солвер, а делать оберткой, которую можно будет применить к любым солверам в будущем.

В проекте реализованы несколько базовых классов для алгоритмов: жадного алгоритма, случайного поиска и поиска восхождением.

ISolver возвращает решения. У каждого решения есть отладочная информация, как правило помогающая в отладке:

1. Время и номер итерации, на которой было найдено решение.
2. Количество улучшений найденного решения (не для всех алгоритмов поиска имеет смысл, но для HillClimbing и MonteCarlo —— имеет)
3. Имя солвера (ISolver.ShortName), который нашел решение (удобно, когда вы начинаете комбинировать солверы)

Кроме того, уже реализованные солверы собирают статистику в течение всей игры про свою работу и на ToString возвращают строковое представление этой статистики.

### Базовый клас для команд бота

В разных контестах CG команды, которые вы отдаёте боту устроены однообразно. Поэтому есть общий класс BotCommand для всех команд, который за вас реализует форматирование команды в строку.
Вам остается только определить класс команды с полями вот так:

```
    public class Move : BotCommand<Move>
    {
        public Move(V pos) => Pos = pos;
        public V Pos;
    }

    public class Wait : BotCommand<Wait>
    {
    }

    public class Use : BotCommand<Use>
    {
        public Use(V pos) => Pos = pos;
        public V Pos;
    }

```
С появлением Record-ов на CG, вероятно, можно будет все это ещё упростить и укоротить.


### StatValue

Регистрируйте в объекте StatValue наблюдаемые значения случайной величины, а он посчитает матожидание, стандартное отклонение, доверительный интервал для матожидания и т.п.

### Вектор

Для того, чтобы код, активно манипулирующий векторами не был громоздким, класс вектора называется супер-кратко: V.
В нем реализованы все операции, над целочисленными двумерными векторами, которые обычно нужны.

Из необычного — рассчет времени до столкновения с другим объектом, к которому мы движемся со скоростью speed: `double GetCollisionTime(V speed, V obstacle, double radius)`

Иногда нужен не целочисленный вектор. Для этого есть аналогичный класс VD.

### Extension-методы

Много мелочей пригождаются постоянно во многих играх. Они собраны в файле Extensions.cs и оформлены методами расширения.

### MaxHeap

Бинарная куча. Её нет в net5, но нужна в некоторых алгоритмах. Вероятно, станет не нужна, когда CodingGame перейдет на net6 с его PriorityQueue.

## Что из этого действительно пригождается?

1. Архитектуру: App + State + StateReader + Countdown + Solver + BotCommand — получается использовать всегда.
2. Часто пригождаются StatValue, V, VD, некоторые Extension-методы.
3. ISolver и его готовые реализации — получается применять далеко не в каждом контесте. Но когда они подходят, сразу получаешь кучу отладочной информации из коробки.

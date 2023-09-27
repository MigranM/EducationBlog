# Linq

`LINQ (Language-Integrated Query)` предоставляет возможности выполнения запросов на уровне языка и API функции высшего порядка.

## Отложенное выполнение

У Linq есть методы, которые выполняются не сразу, а в момент обращения к результирующей выборке каким-либо способом.

Предположим, есть метод, который возвращает перечисление некоторых значений - `GetAllLines()`, тогда следующий код не всегда будет корректным:

```c#
    public class MyService
    {
        public void DoWork()
        {
            var lines = GetAllLines();
            foreach (var line in lines)
            {
                // some logic
            }

            // more logic

            foreach (var line in lines)
            {
                // some logic
            }
        }

        public static IEnumerable<string> GetAllLines()
        {
            // ...
        }
    }

```

В примере выше мы обращаемся к коллекции `GetAllLines()` два раза и если метод `GetAllLines()` возвращал, например, результат запроса к БД, то в разных foreach будет обход разных коллекция. Такое код имеет место быть, если есть уверенность, что нам вернется одна и та же коллекция или логикой приложения подразумевается обход коллекции в одном ее состоянии, а затем в другом.

Чаще всего исправить ситуация можно оператора немедленного выполнения, например:

```c#
    public class MyService
    {
        public void DoWork()
        {
            var lines = GetAllLines().ToList();

            foreach (var line in lines)
            {
                // some logic
            }

            // more logic

            foreach (var line in lines)
            {
                // some logic
            }
        }

        public static IEnumerable<string> GetAllLines()
        {
            // ...
        }
    }

```

В таком случае, `lines` будет не ссылкой на `IEnumerable<string>`, вернувшейся из `GetAllLines()`, а непосредственно списком значений, которые скопированы из `GetAllLines()` разовым обходом это колекции методом `ToList()`.

## Extras

| Операторы отложенного выполнения (deferred) | Операторы немедленного выполнения (immediate) |
|---|---|
| <ul><li>AsEnumerable</li><li>Cast</li><li>Concat</li><li>DefaultIfEmpty</li><li>Distinct</li><li>Except</li><li>GroupBy</li><li>GroupJoin</li><li>Intersect</li><li>Join</li><li>OfType</li><li>OrderBy</li><li>OrderByDescending</li><li>Range</li><li>Repeat</li><li>Reverse</li><li>Select</li><li>SelectMany</li><li>Skip</li><li>SkipWhile</li><li>Take</li><li>TakeWhile</li><li>ThenBy</li><li>ThenByDescending</li><li>Union</li><li>Where</li></ul> | <ul><li>Aggregate</li><li>All</li><li>Any</li><li>Average</li><li>Contains</li><li>Count</li><li>ElementAt</li><li>ElementAtOrDefault</li><li>Empty</li><li>First</li><li>FirstOrDefault</li><li>Last</li><li>LastOrDefault</li><li>LongCount</li><li>Max</li><li>Min</li><li>SequenceEqual</li><li>Single</li><li>SingleOrDefault</li><li>Sum</li><li>ToArray</li><li>ToDictionary</li><li>ToList</li><li>ToLookup</li></ul> |

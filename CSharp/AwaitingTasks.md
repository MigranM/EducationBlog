# void, Task, ValueTask

Методы с модификатором async могут возвращать void, Task и ValueTask.

- Если вызывающему коду не нужно дожидаться окончания выполнения асинхронного метода, то такой метод можно пометить как async void. Такой подход удобен в различных обработчиках событий.
```c#
    public async void OnKeyDown()
    {
        // ...
    }
```
- Если необходимо иметь возможность дождаться выполнение метода, то метод необходимо пометить как async Task. Чаще всего на практике используется именно такой вариант, например, для методов чтения данных из БД, отправки http-запросов и прочего.
```c#
    public async Task<string> GetAllRows()
    {
        // ...
    }
```
- Если ожидается частый вызов метода, а также в большинстве вызовов этот метод будет возвращать значение сразу, в обход всех await, то следует использовать ValueTask.
```c#
    public async ValueTask<string> GetFromCache(string key)
    {
        // ...
    }
```

## ValueTask

ValueTask - инструмент, позволяющий оптимизировать асинхронные методы, в которых при некоторых условиях результат будет вычислен синхронно, без ожиданий. Например, необходимо получить данные из кеша, чаще всего эти данные уже есть, но иногда их нужно получить при помощи вызова асинхронного метода.

```c#
    // Не совсем оптимизированный вариант
    public async Task<object> GetFromCache(string key)
    {
        if (this.cacheRepository.TryGetValue(key, out var result))
        {
            return result;
        }
        return await this.cacheRepository.Query(key);
    }
```

```c#
    // Оптимизированный вариант
    public ValueTask<object> GetFromCache(string key)
    {
        if (this.cacheRepository.TryGetValue(key, out var result))
        {
            return new ValueTask<object>(result);
        }
        return new ValueTask<object>(this.cacheRepository.Query(key));
    }
```

При использовании ValueTask необходимо обратить внимание на некоторые его ограничения:
- Не допускается повторное ожидание ValueTask.
- Параллельное ожидание (по сути повторное) ValueTask не допускается.
- Использование .GetAwaiter().GetResult() не допускается.

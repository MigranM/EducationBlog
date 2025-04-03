# Связянность (Coupling) адаптеров.

## Проблематика
Зависимость адаптера от другого адаптера не допускается в гексагональной архитектуре, но почему?

Самый частый ответ на этот вопрос (после "потому") - это нарушение направления зависимостей, а следом за этим нарушение единственной отвественности.

- Однако принцип единой отвественности может восприниматься в разных проектах по-разному, а границы этой отвественности размыты.

- Что касается направления зависимостей, на первый взгляд может показаться, что DI решает эту проблему.

## Используемый пример
Предположим, что существует контракт:

```c#
namespace Project.Application;

public interface IDbAdapter {
    Task SaveRecord(DomainRecord record);
}
```

Его реализация:

```c#
namespace Project.Adapters;

internal class DbAdapter : IDbAdapter {
    public async Task SaveRecord(DomainRecord record){
        ...
    }
}
```

И еще один контракт с реализацией соответственно:

```c#
namespace Project.Application;

public interface IMessagingAdapter {
    Task OnRecordSaved(DomainRecord record);
}
```

```c#
namespace Project.Adapters;

internal class RmqAdapter : IMessagingAdapter {
    public async Task OnRecordSaved(DomainRecord record){
        ...
    }
}
```

Допустим, необходимо при сохранении записи формировать событие в RMQ. Тогда реализацию IDbAdapter можно было бы представить следующим образом:

## Плохое решение

```c#
namespace Project.Adapters;

internal class DbAdapter : IDbAdapter {
    private readonly IMessagingAdapter _messagingAdapter;

    public async Task SaveRecord(DomainRecord record){
        ...

        await _messagingAdapter.OnRecordSaved(record);
    }
}
```

По коду выше можно заметить, что ссылка у DbAdapter направлена "внутрь" луковицы, а именно на слой приложения и контракт IMessagingAdapter, можно подумать, что DI решил нашу задачу, однако, он лишь оказал нам медвежью услугу. Рассмотрим подробнее какие подводные камни будут скрываться за такой реализацией.

## Проблемы данного решения

### Изолированность адаптера DbAdapter

Если DbAdapter зависит от IMessagingAdapter, то изменение в RmqMessagingAdapter может повлиять на DbAdapter, хотя они оба должны быть изолированы.

### Неявные (скрытые) точки связанности
Адаптеры становятся связанными через неявные контракты. Например, если RmqMessagingAdapter изменит формат сообщения, это может сломать логику DbAdapter, хотя формально IMessagingAdapter остался тем же.

### Усложнение тестирования
Теперь для теста DbAdapter нужно мокать не только БД, но и IMessagingAdapter, хотя он должен отвечать только за сохранение данных.

## Улучшения решения

### Способ 1

Создание сервиса на уровне приложения, который будет содержать в себе бизнес правило (сохранили запись - сформировали событие):

```c#
namespace Project.Application;

internal class RecordService : IRecordService{
    private readonly IDbAdapter _db;
    private readonly IMessagingAdapter _messaging;
    
    ...

    public void Execute(Record record)
    {
        _db.SaveRecord(record);
        _messaging.OnRecordSaved(record.Id);
    }

    ...
}
```

Теперь, бизнес правило находится на нужном уровне, адаптеры более не связаны и зависят только от ядра, а обмен сообщениями между ними (их координация) проходит через слой приложения.

Также, если на проекте используется Use-Case архитектура, то вместо довольно обобщенного RecordService, который со временем обязательно разрестется можно сделать реализацию в отдельно классе use-case'е:

```c#
namespace Project.Application;

internal class SaveRecordUseCase : ISaveRecordUseCase{
    private readonly IDbAdapter _db;
    private readonly IMessagingAdapter _messaging;
    
    public void Execute(Record record)
    {
        _db.SaveRecord(record);
        _messaging.OnRecordSaved(record.Id);
    }
}
```

### Способ 2

Создание событийной модели.

```c#
namespace Project.Domain;

public record RecordSavedEvent(Guid RecordId) : IDomainEvent;
```

```c#
namespace Project.Adapters;

internal class DbAdapter : IDbAdapter {
    private readonly IEventDispatcher _eventDispatcher;

    public async Task SaveRecord(DomainRecord record){
        ...

        _eventDispatcher.Publish(new RecordSavedEvent(record.Id));
    }
}
```

```c#
namespace Project.Application;

internal class RecordSavedEventHandler : IEventHandler<RecordSavedEvent>
{
    private readonly IMessagingAdapter _messaging;
    
    public void Handle(RecordSavedEvent @event) 
    {
        _messaging.OnRecordSaved(@event.RecordId);
    }
}
```

Таким образом, адаптер DbAdapter вызывает доменное событие, при этом не зная ничего про IMessagingAdapter, а слой приложения реагирует на это событие и вызывает IMessagingAdapter.

## Вывод

Запрет на зависимости между адаптерами существует для:
- Сохранения изоляции адаптеров
- Предотвращения создания скрытых точек связанности
- Упрощения тестирования и подмены реализации
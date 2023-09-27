# Простой Singleton

Такой вариант подходит, например, для синглтон-объекта IEqualityComparer<T>, если нужно вызвать его несколько раз, но при этом не хочется в классе создавать поле под ссылку на объект компаратор:

```c#
    public sealed class DocumentInfo
    {
        public Guid ID { get; private set; }

        public Guid TypeID { get; private set; }

        public string? Name { get; private set; }
    }

    public sealed class DocumentInfoTypeComparer : IEqualityComparer<DocumentInfo>
    {
        public static DocumentInfoTypeComparer Instance { get; } = new();

        public bool Equals(DocumentInfo? x, DocumentInfo? y) => x.TypeID == y.TypeID;

        public int GetHashCode([DisallowNull] DocumentInfo obj) => obj.TypeID.GetHashCode();
    }
```

Так как .NET гарантирует вызов статичного конструктора лишь единожды, такой вариант реализации можно назвать потокобезопасным.
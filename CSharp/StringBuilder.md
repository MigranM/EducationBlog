# Тип данных String

Сам по себе примитив string является неизменяемым ссылочным типом. Каждый раз, когда строка изменяется будет возвращена новая строка, созданная в памяти.

При небольшом объеме данных и относительно редком изменении эту проблему можно игнорировать, однако на больших количестах данных с частыми изменениями могут появиться проблемы с производительностью.

Для начала разберем выделение памяти и указатели на строки:

```c#
static void Main(string[] args)
{
    
    var firstText = "Some text";
    var secondText = "Another" +  " text";
}
```
А данном примере в памяти будут созданы две строки, потому что secondText компилятор сразу преобразует в строку "Another text", IL код при этом будет выглядеть следующим образом:

```as
 IL_0000: nop
        IL_0001: ldstr "Some text"
        IL_0006: stloc.0
        IL_0007: ldstr "Another text"
        IL_000c: stloc.1
        IL_000d: ret
```

# Интернирование строк
Среда CLR хранит таблицу, которая называется пул интернирования. Данная таблица содержит уникальные записи с ссылками на строки, которые были объявлены, либо создана в процессе выполнения программы.

Рассмотрим следующий пример:
```c#
static void Main(string[] args)
{

    var firstText = "Some text";
    var firstTextCopy = "Some text";

    var secondText = "Another" +  " text";
    var secondTextCopy = "Another tex";

    secondTextCopy += "t";
}
```
Данный пример демонстрирует интернирование строк в C#. В пуле интернирования будет выделена всего одна ссылка на строку "Some text", однако две "Another text" и еще одна "t". Наглядно убедиться в этом можно сравнив указатели firstText и secondText с их копиями:
```c#
static void Main(string[] args)
{
    
    var firstText = "Some text";
    var firstTextCopy = "Some text";

    var secondText = "Another" +  " text";
    var secondTextCopy = "Another tex";

    secondTextCopy += "t";

    unsafe
    {
        fixed (char* pointerToFirst = firstText)
        { 
            fixed(char* pointerToFirstCopy = firstTextCopy)
            {
                var firstTextResult = pointerToFirst == pointerToFirstCopy;
            }
        }
    }

    unsafe
    {
        fixed (char* pointerToSecond = secondText)
        {
            fixed (char* pointerToSecondCopy = secondTextCopy)
            {
                var secondTextResult = pointerToSecond == pointerToSecondCopy;
            }
        }
    }
}
```
В данном примере firstTextResult будет равен true, в то время как secondTextResult будет равен false. Это происходит потому что secondTextCopy после конкотенации с "t" не будет занесена в пул интернирования строк. Это можно исправить явно добавив строку в пул интернирования, однако это приведет в дополнительным затратам вычислительной мощности, а также строки в пуле интернирования будут существовать до конца работы процесса.

```c#
static void Main(string[] args)
{
    
    var firstText = "Some text";
    var firstTextCopy = "Some text";

    var secondText = "Another" +  " text";
    var secondTextCopy = "Another tex";

    secondTextCopy += "t";

    secondTextCopy = string.Intern(secondTextCopy);

    unsafe
    {
        fixed (char* pointerToFirst = firstText)
        { 
            fixed(char* pointerToFirstCopy = firstTextCopy)
            {
                var firstTextResult = pointerToFirst == pointerToFirstCopy;
            }
        }
    }

    unsafe
    {
        fixed (char* pointerToSecond = secondText)
        {
            fixed (char* pointerToSecondCopy = secondTextCopy)
            {
                var secondTextResult = pointerToSecond == pointerToSecondCopy;
            }
        }
    }
}
```
В данном примере обе переменные firstTextResult и secondTextResult будут иметь значение true.

# StringBuilder
Когда требуется проводить большое количество операций с измененем строк, особенно больших строк, то интернирование строк не поможет решить проблему производительности и затрат памяти, так как преобразованные строки могут быть разные. В таком случае следует использовать класс StringBuilder.

Класс StringBuilder позволяет работать с "изменяемой" строкой символов.

Рассмотрим следующий пример:
```c#
internal class Program
{
    static void Main(string[] args)
    {
        BuildStringStringBuild();
        BuildStringRaw();
    }

    private static string BuildStringStringBuild()
    {
        var stringBuilder = new StringBuilder("Some text");
        for (var i = 0; i < 1000; i++)
        {
            stringBuilder.Append("Some text" + Environment.NewLine);
        }
        return stringBuilder.ToString();
    }

    private static string BuildStringRaw()
    {
        var result = "Some text";
        for (var i = 0; i < 1000; i++)
        {
            result += "Some text" + Environment.NewLine;
        }

        return result;
    }
}
```
В данном примере формируются одинаковые строки, одна при помощи StringBuilder, а одна при помощи конкатенации строк класса String. Результаты производительности представлены ниже:

| Method        | Mean      | Error    | StdDev   | Gen0      | Gen1     | Allocated   |
|-------------- |----------:|---------:|---------:|----------:|---------:|------------:|
| BuildStringStringBuild |  21.49 us | 0.416 us | 0.462 us |   21.8811 |   2.4109 |    100.8 KB |
| 'BuildStringRaw' | 826.09 us | 8.141 us | 7.217 us | 2343.7500 | 119.1406 | 10794.92 KB |

---
Как можно заменить исходя из данных в таблице BuildStringRaw занял на порядок больше времени, а также на несколько порядков больше памяти.

По рекомендациям Microsoft StringBuilder рекомендуется использовать в указанных случаях:
- При неизвестном количестве операций и изменений над строками во время выполнения программы;
- Когда предполагается, что приложению придется сделать множество подобных операций.

Класс string использовать в следующих случаях:
- При небольшом количестве операций и изменений над строками;
- При выполнении фиксированного количества операций объединения. В этом случае компилятор может объединить все операции объединения в одну;
- Когда надо выполнять масштабные операции поиска при построении строки, например IndexOf или StartsWith. Класс StringBuilder не имеет подобных методов.


# struct vs ref struct

Передача в параметр метода struct и ref struct имеет отличия.

Рассмотрим пример:
```c#
public struct Square
{
    public int Size;
    
    public Square(int size)
    {
        this.Size= size;
    }
}

internal class Program
{

    static void Main(string[] args)
    {
        var square = new Square(10);
        ChangeStruct(square);
        ChangeStructRef(ref square);
    }

    public static void ChangeStruct(Square square)
    {
        square.Size = 5;
    }

    public static void ChangeStructRef(ref Square square)
    {
        square.Size = 5;
    }
}
```
В данном примере при передаче в метод ChangeStruct() структуры Square в метод будет передано лишь значение square. Таким образом, внути данного метода будет изменена лишь копия square.
Ситуация обстоит иначе с ChangeStructRef(), в данном случае в метод будет передана ссылка на square и внутри метода изменится поле Size переданной структуры.

Это отчетливо видно при просмотре IL кода данной программы. Метод Main:
```as
IL_0000: nop
IL_0001: ldloca.s 0
IL_0003: ldc.i4.s 10
IL_0005: call instance void RefStruct.Square::.ctor(int32)
IL_000a: ldloc.0
IL_000b: call void RefStruct.Program::ChangeStruct(valuetype RefStruct.Square)
IL_0010: nop
IL_0011: ldloca.s 0
IL_0013: call void RefStruct.Program::ChangeStructRef(valuetype RefStruct.Square&)
IL_0018: nop
IL_0019: ret
```
В коде метода Main можно заметить, как при вызове ChangeStruct() в метод передается значение переменной, а в ChangeStructRef() передается адрес Square& данной структуры.

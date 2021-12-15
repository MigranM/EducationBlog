# const vs readonly

Ключевые слова *const* и *readonly* в C# довольно схожи, но имеют ряд отличий

------

## Ключевое слово *const*

- Ключевое слово *const* используется для объявления константного поля, такое поле не может быть изменено в результате выполнения программы;
- Поля с ключевым словом *const* должны быть инициализированы каким-либо значения сразу во время объявления.

------

Константные поля удобно использовать для хранения сообщений об ошибках, как на примере снизу:

```c#
        public class WebClient
        {
            public const string URLTooLongMessage = "Request URL too long";

            public WebClient()
            {
                //some logic
            }

            public bool getResponse(string request,  out string errorCode, out string errorMessage)
            {
                bool Succes = true;
                errorCode = string.Empty;
                errorMessage = string.Empty;

                //more logic

                if (request.Length> 2048)
                {
                    errorCode = "404.14";
                    errorMessage = "Request URL too long";
                    Succes = false;
                }

                return Succes;
            }
            
        }
```

Константные поля не привязаны к конкретному экземпляру класса, а являются полем самого класса, как если бы они были помечены ключевым словом "static". 

По этой причине нельзя создавать поля вида: *static const double PI*.

Благодаря этой особенности константные поля можно без проблем использовать в статических методах, как на примере снизу:

```c#
    public class SalaryCalculator
        {
            private const decimal PersonalIncomeTaxOnRegularSalary = 13m;

            public static decimal GetSalaryAfterTax(decimal salary)
            {
                salary = salary * (1 - PersonalIncomeTaxOnRegularSalary);
                return salary;
            }
        }
```

------

## Ключевое слово *readonly*

Модификатор *readonly* так же как и ключевое слово *const* гарантирует неизменность поля во время выполнения, однако, *readonly* позволяет создавать неизменяемые поля на уровне экземпляров класса.

- Поля помеченные модификатором *readonly* должны быть проинициализированы либо сразу при объявлении, либо в конструкторе экземпляра;
- Для данных ссылочного типа модификатор *readonly* гарантирует лишь неизменность ссылки, в то время как сам экзмепляр может быть изменен, к примеру, если в нем есть открытые поля или свойства.

Поля с модификатором readonly удобно использовать, если есть необходимость добавить какие-либо неизменяемые аттрибуты экземпляру.

Например, у нас есть необходимость создавать объекты вражеских сущностей с определенными параметрами. Здоровье и показатели защиты данной сущности могут быть изменены в процессе выполнения программы, однако изменение имени данной сущности будет запрещено.

```c#
   public class EnemyEntity
        {
            public readonly string name;
            public int health;
            public int defence;

            public EnemyEntity(string name, int health, int defence)
            {
                this.name = name;
                this.health = health;
                this.defence = defence;
            }

            //Entity actions
        }
```

Главным отличием *readonly* от const является то, что в *readonly* поле можно поместить данные времени выполнения, что позволяет создавать выражения как типа:

```c#
public static readonly uint timeStamp = (uint)DateTime.Now.Ticks;
```

------

## static *readonly*

Поля с ключевыми словами static *readonly* аналогичны полям с ключевым словом *const*, но, как уже было сказано выше, могут быть проинициализированы во время выполнения. К примеру, можно записать временную метку первого обращения к классу при помощи статического конструктора. Для примера слегка расширим класс *EnemyEntity* из предыдущего примера:

```c#
public class EnemyEntity
        {
            public static readonly DateTime FirstEntitySpawnTime;
            public readonly DateTime EntitySpawnTime;

            public readonly string name;
            public int health;
            public int defence;

            static EnemyEntity()
            {
                //set spawn time of first ever entity
                FirstEntitySpawnTime = DateTime.Now; 
            }

            public EnemyEntity(string name, int health, int defence)
            {
                //set entity spawn time
                EntitySpawnTime = DateTime.Now;

                this.name = name;
                this.health = health;
                this.defence = defence;
            }

            //Entity actions
        }
```

Теперь при создании первого экземпляра класса *EnemyEntity* сработает статический конструктор, который запишет временную метку "Спавна" первой сущности.


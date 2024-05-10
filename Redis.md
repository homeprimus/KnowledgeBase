# [Redis](https://redis.io/)
#### Redis — резидентная система управления базами данных класса NoSQL с открытым исходным кодом, работающая со структурами данных типа «ключ — значение». Используется как для баз данных, так и для реализации кэшей, брокеров сообщений. Ориентирована на достижение максимальной производительности на атомарных операциях

Примеры можно найти на https://github.com/redis/NRedisStack/tree/master/Examples


```C#
    <PackageReference Include="NRedisStack" Version="0.12.0" />
```


## Hashes
#### Хэши Redis используются для хранения структурированных данных. Каждый хеш может содержать несколько пар поле-значени
```C#
using StackExchange.Redis;

var options = new ConfigurationOptions
{
    EndPoints = { { "localhost", 6379 } },
    //User = "default",
    //Password = "secret",
    //Ssl = true,
    //SslProtocols = SslProtocols.Tls12,
};

ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options);

IDatabase db = redis.GetDatabase();

var key = "user-session:123";
var hash = new HashEntry[] {
    new HashEntry("name", "John"),
    new HashEntry("surname", "Smith"),
    new HashEntry("company", "Redis"),
    new HashEntry("age", "29"),
};
db.HashSet(key, hash);
var hashFields123 = db.HashGetAll(key);
Console.WriteLine("HashGetAll = " + String.Join("; ", hashFields123));
Console.WriteLine("age = " + db.HashGet(key, "age"));

db.HashDelete(key, "age");
hashFields123 = db.HashGetAll(key);
Console.WriteLine("Delete hash field. HashGetAll =  " + String.Join("; ", hashFields123));


db.KeyDelete(key);
hashFields123 = db.HashGetAll(key);
Console.WriteLine("After delete key. HashGetAll =  " + String.Join("; ", hashFields123));

key = "user-session:456";
db.HashSet(key, "name", "Иван");
db.HashSet(key, "surname", "Иванов");
db.HashSet(key, "company", "Строитель");
db.HashSet(key, "age", "57");
var hashFields = db.HashGetAll(key);
Console.WriteLine(String.Join("; ", hashFields));
Console.WriteLine("age = " + db.HashGet(key, "age"));

```

## Lists
#### Списки Redis позволяют хранить упорядоченные списки элементов и манипулировать ими.

```C#
using StackExchange.Redis;

var options = new ConfigurationOptions
{
    EndPoints = { { "localhost", 6379 } },
};
ConnectionMultiplexer redis = ConnectionMultiplexer.Connect(options);
IDatabase db = redis.GetDatabase();

var key = "mylist";
// Add items to a Redis List
db.ListLeftPush(key, "item1");
db.ListLeftPush(key, "item2");

// Retrieve all items from the List
var listItems = db.ListRange(key);
Console.WriteLine(String.Join("; ", listItems));

```

## Puslish & Subscribe
```C#
/*
    Publisher
*/
using StackExchange.Redis;

var options = new ConfigurationOptions
{
    EndPoints = { { "localhost", 6379 } },
};
var chanelName = "bus.chanel";

var redis = ConnectionMultiplexer.Connect(options);

var pubsub = redis.GetSubscriber();
Enumerable
    .Range(1, 500)
    .ToList()
    .ForEach(i =>
    {
        pubsub.Publish(chanelName, $"message{i}");
        Console.WriteLine($"message{i}");
    });

Console.ReadKey();

/*
    Subscriber
*/
using StackExchange.Redis;

var options = new ConfigurationOptions
{
    EndPoints = { { "localhost", 6379 } },
};
var chanelName = "bus.chanel";

var redis = ConnectionMultiplexer.Connect(options);
var chanel =  redis.GetSubscriber().Subscribe(chanelName);
chanel.OnMessage((message) =>
{
    Console.WriteLine(message.Message);
    Task.Delay(200);
});
Console.ReadKey();

```
## Пакеты
[FusionCache](https://github.com/ZiggyCreatures/FusionCache)
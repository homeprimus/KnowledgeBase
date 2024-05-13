# MongoDB
Документно-ориентированая система управления бд. Вместо строк хрпнит документы, в которых есть первичный ключ **_id**
### Коллекции
Вместо таблиц используются **коллекции**, которые могут содержать самые разные объектв с различной структурой и различным набором свойств.

### Формат данных
BSON (БиСон) - binary JSON

### Компоненты
- mongosh консольный клиент
- mongod сервер баз данных
- mongos служба маршрутизации

```SQL
use userdb
db.users.insertOne( { name: "Tom" } )
db.users.find()
```

### Подключение 
```C#
var client = new MangoClient("mango://private:private@lovalhost");
```
### Документы
```C#
var client = new MangoClient("mango://private:private@lovalhost");
var doc = new BsonDocumrnt{
    { "name", "Andrey" },
    { "age", 57},
    { "company", new BsonDocunent { "name", "Quest" } },
    { "language", new BsonArray {"ru", "fr", "en" } }
};
Console.WriteLine(doc);

```

### Обращение к полям документа
```C#
var doc = new BsonDocument
{
    { "name", "Igor"},
    {"age", 57 },
    {"company", new BsonDocument{ { "name", "Quest" }  } },
    {"country", new BsonDocument { { "name", "RU" } } },
    {"languages", new BsonArray {"en", "ru", "fr", "jn"} }
};
Console.WriteLine(doc);
doc["age"] = 58;
doc["new property"] = "это новое поле";
Console.WriteLine(doc);

```
### Управление элементами

С помощью методов класса BsonDocument мы можем управлять элементами внутри документа:

- Add(): добавляет элемент в документ
- AddRange(): добавляет набор элементов в документ
- Clear(): удаляет все элементы из документа
- Contains(): возвращает true, если в документе есть элемент с определенным именем
- ContainsValue(): возвращает true, если в документе есть элемент с определенным значением
- GetValue(): возвращает значение определенного элемента по имени или по позиции
- Remove(): удаляет элемент из документа

### Установка Id
```c#
using MongoDB.Bson.Serialization.Attributes;
 
class Person
{
    [BsonId]
    public int PersonId { get; set; }
    public string Name { get; set; } = "";
}
```
### Исключение свойств BsonIgnore
```c#
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;
 
Person tom = new Person { Name = "Tom", Email = "tom@somemail.com" };
Console.WriteLine(tom.ToBsonDocument()); // { "Name" : "Tom" }
 
class Person
{
    public string Name { get; set; } = "";
    [BsonIgnore]
    public string Email { get; set; } = "";
}
```
### BsonElement
```c#
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;
 
Person tom = new Person { Name = "Tom", Email = "tom@somemail.com" };
Console.WriteLine(tom.ToBsonDocument()); // { "Name" : "Tom", "Login" : "tom@somemail.com" }
 
class Person
{
    public string Name { get; set; } = "";
    [BsonElement("Login")]
    public string Email { get; set; } = "";
}
```
### Игнорирование значений по умолчанию
- BsonIgnoreIfNull 
- BsonIgnoreIfDefault 

### BsonRepresentation

Еще один атрибут BsonRepresentation отвечает за представление свойства в базе данных. Например:
```c#	
class Person
{
    public string Name { get; set; } = "";
    [BsonRepresentation(BsonType.String)]
    public int Age { get; set; }
}
```
В этом случае целочисленному свойству Age в базе данных будет соответствовать строковое поле Age из-за применения атрибута ***[BsonRepresentation(BsonType.String)]***.
### BsonClassMap

Для настройки сопоставления классов C# с коллекциями MongoDB можно использовать класс BsonClassMap, который регистрирует принципы сопоставления. Например, возьмем тот же класс Person:
```c#
using MongoDB.Bson;
using MongoDB.Bson.Serialization;
 
BsonClassMap.RegisterClassMap<Person>(cm =>
{
    cm.AutoMap();
    cm.MapMember(p => p.Name).SetElementName("username");
});
Person tom = new Person { Name = "Tom", Age = 38};
Console.WriteLine(tom.ToBsonDocument()); // { "username" : "Tom", "Age" : 38 }
 
class Person
{
    public string Name { get; set; } = "";
    public int Age { get; set; }
}
```
### Конвенции

Конвенции наряду с атрибутами и BsonClassMap представляют еще один способ определения сопоставления классов и объектов BsonDocument. Конвенции определяются в виде набора - объекта ConventionPack. Этот объект может содержать набор конвенций. Каждая конвенция представляет объект класса, производного от ConventionBase. Например, переведем все имена элементов в BsonDocument в нижний регистр:
```C#
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Conventions;
 
var conventionPack = new ConventionPack
{
    new CamelCaseElementNameConvention()
};
ConventionRegistry.Register("camelCase", conventionPack, t => true);
Person tom = new Person { Name = "Tom", Age = 38};
Console.WriteLine(tom.ToBsonDocument()); // { "name" : "Tom", "age" : 38 }
 
class Person
{
    public string Name { get; set; } = "";
    public int Age { get; set; }
}
```
### Сохранение документов в базу данных InsertOneAsync InsertManyAsync
```C#
var db = client.GetDatabase("userdb");
var people = db.GetCollection<BsonDocument>("people");
var doc = new BsonDocument
{
    { "name", "Igor"},
    {"age", 57 },
    {"company", new BsonDocument{ { "name", "Quest" }  } },
    {"country", new BsonDocument { { "name", "RU" } } },
    {"languages", new BsonArray {"en", "ru", "fr", "jn"} }
};
people.InsertOne(doc);
Console.ReadKey();
```

### Фильтрация данных

```
{ "_id" : ObjectId("63596dc76348749cac779373"), "Name" : "Tom", "Age" : 38, "Languages" : ["english", "german", "spanish"] }
{ "_id" : ObjectId("63596dc76348749cac779374"), "Name" : "Bob", "Age" : 42, "Languages" : ["english"] }
{ "_id" : ObjectId("63596dc76348749cac779375"), "Name" : "Sam", "Age" : 25, "Languages" : ["english", "spanish"] }
{ "_id" : ObjectId("63596dc76348749cac779376"), "Name" : "Alice", "Age" : 33 }
{ "_id" : ObjectId("63596dc76348749cac779377"), "Name" : "Tom", "Age" : 33, "Languages" : ["english"] }
```
```c#
using MongoDB.Bson;
using MongoDB.Driver;
 
MongoClient client = new MongoClient("mongodb://localhost:27017");
 
var db = client.GetDatabase("test");
var collection = db.GetCollection<BsonDocument>("users");
 
// определяем фильтр - находим все документы, где Name = "Tom"
var filter = new BsonDocument { { "Name", "Tom" } };
 
List<bsondocument> users = await collection.Find(filter).ToListAsync();
foreach (var user in users)
{
    Console.WriteLine(user);
}
```
### Операторы выборки
-    **$eq** (равно)
-    **$ne** (не равно)
-    **$gt** (больше чем)
-    **$lt** (меньше чем)
-    **$gte** (больше или равно)
-    **$lte** (меньше или равно)
-    **$in** определяет массив значений, одно из которых должно иметь поле документа
-    **$nin** определяет массив значений, которые не должно иметь поле документа

найдем все документы, где Age больше 33:
```c#
var filter = new BsonDocument { { "Age", new BsonDocument("$gt", 33) } };
 
var users = await collection.Find(filter).ToListAsync();
```
### Логические операторы
Логические операторы выполняются над условиями выборки:
-    **$or**: соединяет два условия, и документ должен соответствовать одному из этих условий
-    **$and**: соединяет два условия, и документ должен соответствовать обоим условиям
-    **$not**: документ должен НЕ соответствовать условию
-    **$nor**: соединяет два условия, и документ должен НЕ соответстовать обоим условиям

фильтр для выбора документов, у которых поле "Age" имеет значение от 33 и выше, либо поле "Name" имеет значение "Tom":
```c#	
var filter = new BsonDocument("$or", new BsonArray{
 
    new BsonDocument("Age",new BsonDocument("$gte", 33)),
    new BsonDocument("Name", "Tom")
});
 
var users = await collection.Find(filter).ToListAsync();
```
### Проверка на отсутствие поля. Оператор $exists
Документы могут иметь разный набор полей, что, если нам надо получить те документы, где нет какого-то поля? В этом случае нам надо найти те документы, где данное поле имеет значение BsonNull. Например, найдем документы, где НЕ определен массив Languages:
```c#	
var filter = new BsonDocument("Languages", BsonNull.Value); // где Languages отсутствует
 
var users = await collection.Find(filter).ToListAsync();
foreach (var user in users)
{
    Console.WriteLine(user);
}
```
```c#
var filter = new BsonDocument("Languages", new BsonDocument { { "$exists", false } });
```
### Поиск по массивам

Ряд операторов предназначены для работы с массивами:
-    $all: определяет набор значений, которые должны иметься в массиве
-    $size: определяет количество элементов, которые должны быть в массиве
-    $elemMatch: определяет условие, которым должны соответствовать элементы в массиве

Например, найдем все документы, где в массиве Languages есть значения "english" и "spanish":
```c#	
var filter = new BsonDocument("Languages", new BsonDocument { {"$all", new BsonArray { "english", "spanish"} } });
 
var users = await collection.Find(filter).ToListAsync();
foreach (var user in users)
{
    Console.WriteLine(user);
}
```
### Соответствие регулярному выражению. Оператор $regex
Оператор $regex задает регулярное выражение, которому должно соответствовать значение поля. Например, пусть поле Name заканчивается на букву "m":
```c#	
var filter = new BsonDocument("Name", new BsonDocument { {"$regex", "m$" } });
 
var users = await collection.Find(filter).ToListAsync();
foreach (var user in users)
{
    Console.WriteLine(user);
}
```
### FilterDefinitionBuilder и определение фильтров
 Другим способом для создания фильтрации представляет применение класса FilterDefinitionBuilder - построителя фильтров. Он подразумевает подразуемвает применения одного или нескольких методов фильтрации:
```c#
// определяем построитель фильтров
var builder = Builders<BsonDocument>.Filter;
// создаем фильтр, который выбирает все документы, где Name = "Tom"
var filter = builder.Eq("Name", "Tom");
```
Операции сравнения
-    Eq: выбирает только те документы, у которых значение определенного поля равно некоторому значению
 -   Ne: выбирает только те документы, у которых значение определенного поля не равно некоторому значению
-    Gt: выбирает только те документы, у которых значение определенного поля больше некоторого значения
-    Gte: выбирает только те документы, у которых значение определенного поля больше или равно некоторому значению
-    Lt: выбирает только те документы, у которых значение определенного поля меньше некоторого значения
-    Lte: выбирает только те документы, у которых значение определенного поля меньше или равно некоторому значению
-    In: получает все документы, у которых значение поля может принимать одно из указанных значений
-    Nin: противоположность оператору In - выбирает все документы, у которых значение поля не принимает одно из указанных значений

Выберем все документы, где поле "Name" равно "Tom":

```c#
using MongoDB.Bson;
using MongoDB.Driver;
 
MongoClient client = new MongoClient("mongodb://localhost:27017");
 
var db = client.GetDatabase("test"); 
var collection = db.GetCollection<BsonDocument>("users");
// определяем построитель фильтров
var builder = Builders<BsonDocument>.Filter;
 
// определяем фильтр - находим все документы, где Name = "Tom"
var filter = builder.Eq("Name", "Tom");
 
var users = await collection.Find(filter).ToListAsync();
foreach (var user in users)
{
    Console.WriteLine(user);
}
```
### Логические операции
С помощью стандартных операций программирования конъюнкции, дизъюнкции и логического отрицания, а также соответствующих им методов можно комбинировать запросы:

    Операция ! и метод Not возвращают документы, которые НЕ соответствуют определенному условию.

    Операция | и метод Or возвращают документы, которые соответствуют как минимум одному из фильтров из набор.

    Операция & и метод And возвращают документы, которые соответствуют всем фильтрам из набора

Или, например, зададим фильтр для выбора документов, у которых поле "Age" имеет значение от 33 и выше, либо поле "Name" имеет значение "Tom":
```c#
using MongoDB.Bson;
using MongoDB.Driver;
 
MongoClient client = new MongoClient("mongodb://localhost:27017");
 
var db = client.GetDatabase("test"); 
var collection = db.GetCollection<BsonDocument>("users");
 
var builder = Builders<BsonDocument>.Filter;
var filter = builder.Or(builder.Eq("Name", "Tom"), builder.Gte("Age", 33));
 
var users = await collection.Find(filter).ToListAsync();
foreach (var user in users)
{
    Console.WriteLine(user);
}
```
Здесь применяется метод Or(). Он выбирает документы, которые соответствуют одному из подфильтров, которые передаются в метод в качестве параметров. Так, в данном случае передаем два фильтра: builder.Eq("Name", "Tom") и builder.Gte("Age", 33)

### Операции с элементами
-    Exists: выбирает из бд те документы, в которых присутствует определенное поле
-    NotExists: выбирает из бд те документы, в которых отсутствует определенное поле

Например, фильтр для выбора документов, где присутствует поле "Languages":
```c#	
var builder = Builders<bsondocument>.Filter;
var filter = builder.Exists("Languages");
```
### Метод Regex
Regex в качестве параметра получает имя поля и регулярное выражение, которому должно соответствовать значение этого поля. Например, найдем все документы, где поле Name заканчивается на букву "m":
```c#	
var builder = Builders<bsondocument>.Filter;
var filter = builder.Regex("Name", new BsonRegularExpression("m$"));
```
### Метод Where
Метод Where() принимает условие, которому должны соответствовать документы. 
```c#
var filter = builder.Where(d=>d["Age"]<30);
```
### Операции с массивами
-    All: выбирает все документы, в которые содержат все элементы массива
-    Size: выбирает все документы, которые содержат определенное число элементов
```c#
var builder = Builders<bsondocument>.Filter;
var filter = builder.All("Languages", new string[] { "english", "spanish" });
...
var builder = Builders<bsondocument>.Filter;
var filter = builder.Size("Languages", 1); 
```
### Сортировка
Для сортировки документов применяется метод Sort(), который в качестве параметра принимает определение сортировки. В качестве такого определения может выступать документ BsonDocument. 
```c#
var users = await collection.Find(new BsonDocument { }).Sort(new BsonDocument("Age", 1)).ToListAsync();
```
Значение 1 в new BsonDocument("Age", 1) указывает на направление сортировки - по возрастанию. Если мы хотим выполнить сортировку по убыванию, то передается значение -1
Также можно использовать сокращенное определение сортировки в виде строки:
```c#	
var users = await collection.Find("{}").Sort("{Age:1}").ToListAsync();
```
Дополнительно метод **SortBy()** сортирует по возрастанию. Он принимает делегат, который через параметр получает тип документа и возвращает поле/свойство документа, по которому идет сортировка
```c#
var users = await collection.Find("{}").SortBy(d => d["Name"]).ToListAsync();
```
Аналогично работает метод SortByDescending(), который сортирует по убыванию:
```c#	
var users = await collection.Find("{}").SortByDescending(d => d["Name"]).ToListAsync();
```
### Последовательное применение нескольких критериев сортировки
Класс SortDefinitionBuilder позволяет последовательно применить несколько критериев сортировки:
```c#
var sortDefinition = Builders<BsonDocument>.Sort.Ascending("Name").Descending("Age");
var users = await collection.Find("{}").Sort(sortDefinition).ToListAsync();
```
Также мы можем воспользоваться перегруженными версиями и задать критерии фильтрации через лямбда-выражения:
```c#
// сначала сортируем по Name во возврастанию, а затем по Age по убыванию
var sortDefinition = Builders<Person>.Sort.Ascending(p=>p.Name).Descending(p=>p.Age);
var users = await collection.Find("{}").Sort(sortDefinition).ToListAsync(); 
```
## Получение одного документа и части коллекции
### Получение одного документа
Для получения одного документа применяются методы 
- First()/FirstAsync()
- Single()/SingleAsync()
- FirstOrDefault()/FirstOrDefaultAsync()
- SingleOrDefault()/SingleOrDefaultAsync()

При этом, если выборка не содержит документов, то методы First()/FirstAsync()/Single()/SingleAsync() генерируют исключение, а FirstOrDefault()/FirstOrDefaultAsync()/SingleOrDefault()/SingleOrDefaultAsync() возвращают значение по умолчанию.

### Skip

Метод Skip() пропускает при выборке определенное количество документов. Например, пропустим 3 первых документа:
```c#
// пропускаем 3 первых документа
var users = await collection.Find("{}").Skip(3).ToListAsync();
```
### Limit
метод Limit() устанавливает максимальное количество документов, которое можно получить
```c#
// выбираем 3 первых документа
var users = await collection.Find("{}").Limit(3).ToListAsync();
```
### Постраничная навигация
Комбинация методов Skip() и Limit() позволяет извлечь определенную часть коллекции
```c#
var users = await collection.Find("{}")
    .Skip(2)        // пропускаем 2 документа
    .Limit(3)       // извлекаем следующие 3 документа
    .ToListAsync();
```

### Проекция
Если документы имеют сложную структуру, которая нам не очень подходит для вывода, то мы можем использовать проекции, то есть из изначальных документов коллекции получить совершенно другие данные. Для создания проекции применяется построитель проекции Builders.Projection. Этот класс предоставляет два метода для управления проекцией:
-    Include(): добавляет поле в выходной результат
-   Exclude(): исключает поле из выходного результата

Эти методы возвращают определение проекции в виде объекта ProjectionDefinition

Для выполнения самой проекции вызывается метод Projection(), в который передается объект ProjectionDefinition
```c#
var projection = Builders<BsonDocument>.Projection.Include("Name").Include("Age").Exclude("_id");
var users = await collection.Find(new BsonDocument()).Project(projection).ToListAsync();
...
// или
var users = await collection.Find("{}").Project("{Name:1, Age:1, _id:0}").ToListAsync();
```
```c#
var users = await collection.Find("{}")
    .Project(doc => new Person(doc["Name"].ToString()))
    .ToListAsync();

record Person(string? Name);    
```    
### Группировка
Группировка позволяет сгруппировать документы в выборке по определенному критерию. Для группировки у объекта IMongoCollection применяется метод **Aggregate**, а у его результата - объекта IAggregateFluent вызывается метод **Group()**.

Метод Group в качестве параметра принимает BsonDocument, который описывает, как должна проводиться группировка. 
```c#
var employees = await collection.Aggregate()
    .Group( new BsonDocument { 
        { "_id", "$Company.Title" },               // группируем по имени компании
        { "count", new BsonDocument("$sum", 1) }, // получаем количество документов в группе
    })
    .ToListAsync();
```
в метод Group() передается документ BsonDocument, который устанавливает параметры группировки. При этом нам надо установить параметр "_id" - ключ группировки. В данном случае для ключа выбрано поле Company.Title, при этом к названию поля добавляется знак доллара: "$Company.Title". То есть каждая группа объектов будет иметь одно и то же значение поля "Company.Title".
```c#	
{ "_id", "$Company.Title" }
```
То есть в данном записи формально мы говорим, что значением поля "count" в выходном документе будет результат, который задается объектом new BsonDocument("$sum", 1). А данный объект выполняет встроенную функцию mongodb - функцию "count", которая вычисляет количество документов в группе. Значение "1" указывает на множитель.
-    **$sum** вычисляет сумму
-    **$avg** вычисляет среднее значение
-    **$first** получает значение поля из первого документа группы
-    **$last** получает значение поля из последнего документа группы
-    **$max** вычисляет максимальное значение
-    **$min** вычисляет минимальное значение
-    **$push** добавляет значение в массив
-    **$addToSet** добавляет значение в набор

найдем среднее, максимальное и минимальное значение возраста (то есть значение поля "Age"):
```c#
var employees = await collection.Aggregate()
    .Group( new BsonDocument {
        { "_id", "$Company.Title" },
        { "minAge", new BsonDocument("$min", "$Age") },
        { "maxAge", new BsonDocument("$max", "$Age") },
        { "avgAge", new BsonDocument("$avg", "$Age") }
    })
    .ToListAsync();
```
для каждой группы значение "Name" первого и последнего документа группы:
```c#
var employees = await collection.Aggregate()
    .Group( new BsonDocument {
        { "_id", "$Company.Title" },
        { "first", new BsonDocument("$first", "$Name") },
        { "last", new BsonDocument("$last", "$Name") }
    })
    .ToListAsync();
```
добавм в каждую группу имена сотрудников соответствующей компании:
```c#
var employees = await collection.Aggregate()
    .Group(new BsonDocument {
        { "_id", "$Company.Title" },
        { "employees", new BsonDocument("$push", "$Name") } })
    .ToListAsync();
```
Консольный вывод:
```
{ "_id" : "Google", "employees" : ["Bob", "Alice"] }
{ "_id" : "Microsoft", "employees" : ["Tom", "Sam", "Dan"] }
```
### Использование классов C#
Вместо стандартного BsonDocument мы можем использовать свойства стандартных классов C#, которые описывают используемые данные
```c#
var companies = await collection.Aggregate()
    .Group(emp => emp.Company!.Title,   // группировка по свойству Company.Title
    g => new CompanyModel       // из группы создаем объект CompanyModel
    { 
        CompanyName = g.Key, 
        Employees = g.Select(e=>e.Name).ToList() // выбираем в список имена сотрудников
    })
    .ToListAsync();

class CompanyModel
{
    public string CompanyName { get; set; } = "";
    public List<string> Employees { get; set; } = new();
}
record Employee(ObjectId Id, string Name, int Age, Company? Company);
record Company(string Title);    
```
В данном случае коллекция типизируется типом Employee. В перегруженную версию метода Group передаем два делегата. Первый делегат выбирает из класса Employee свойство-признак, по которому будет идти группировка:
```c#	
.Group(emp => emp.Company!.Title,   // группировка по свойству Company.Title
```
Второй параметр-делегат в качестве параметра принимает созданную группу и возвращает сгенерированный на базе группы объект. В данном случае для представления объекта определен специальный класс CompanyModel (хотя можно было бы обойтись и анонимным объектом).
```c#	
g => new CompanyModel       // из группы создаем объект CompanyModel
    { 
        CompanyName = g.Key, 
        Employees = g.Select(e=>e.Name).ToList() // выбираем в список имена сотрудников
    })
```
Метод **g.Select()** здесь фактически выполняет функцию операции **$push** - выбирает все имена сотрудников и формирует из них список.

Подобным образом можно задействовать и другие операторы группировки:
```c#
var companies = await collection.Aggregate()
    .Group(emp => emp.Company!.Title,   // группировка по свойству Company.Title
    g =>  new     // из группы создаем анонимный объект
    { 
        CompanyName = g.Key, 
        Employees = g.Select(e=>e.Name).ToList(), // выбираем в список имена сотрудников
        SumAge = g.Sum(e=>e.Age),
        MinAge = g.Min(e=>e.Age),
        MaxAge = g.Max(e=>e.Age),
        AvgAge = g.Average(e=>e.Age),
        First = g.First().Name,
        Lass = g.Last().Name
    })
    .ToListAsync();
```    

## Редактирование документов
Для редактирования документов коллекции применяется ряд методов:
-    ReplaceOne()/ReplaceOneAsync(): полностью заменяет один документ другим
-    UpdateOne()/UpdateOneAsync(): обновляет один документ/p>
-    UpdateMany()/UpdateManyAsync(): обновляет набор документов
-    FindOneAndReplace()/FindOneAndReplaceAsync(): если находит документ, который соответствует фильтру, то заменяет его и возвращает заменный документ
-    ReplaceOne()/ReplaceOneAsync(): полностью заменяет один документ другим

Они имеют разные версии, принимают разное количество параметров. Но все они как минимум принимают два параметра:

-    Фильтр, который позволяет отфильтровать документы для обновления
-    новый документ, который заменяет старый или описывает принципы изменения старого

**При обновлении надо учитывать, что мы не можем изменить поле** ***_id***.
### ReplaceOneAsync
Метод ReplaceOneAsync() позволяет полностью заменить старый документ новым.
```c#
// определяем фильтр - документ, где Name = Tom и Age = 33
var filter = new BsonDocument { { "Name", "Tom" }, { "Age", 33 } };
// определяем документ, на который будет заменять
var newDocument = new BsonDocument { { "Name", "Tomas" }, { "Age", 34 } };
// выполняем замену
var result = await collection.ReplaceOneAsync(filter, newDocument);
```
Если документов, которые соответствуют фильтру, отсутствуют в выборке, то соответственно и замен не будет. Если фильтру соответствуют несколько документов, то будет заменен только первый из них.

Метод **ReplaceOneAsync()** возвращает объект **ReplaceOneResult**, с помощью которого мы можем получить информацию об обновлении, в частности, получить количество документов, которые соответствуют критерию, и количество обновленных документов.
## Вставка документа

С помощью дополнительного параметра ReplaceOptions метода ReplaceOneAsync() можно управлять параметрами обновления. **new ReplaceOptions { ***IsUpsert = true*** }** мы можем указать обязательность вставки документа в коллекцию, даже если не найдено соответствий по критерию:
```c#
// определяем фильтр - документ, где Name = Alex и Age = 33
var filter = new BsonDocument { { "Name", "Alex" }, { "Age", 33 } }; 
// определяем документ, на который будет заменять
var newDocument = new BsonDocument { { "Name", "Alexander" }, { "Age", 34 } };
// выполняем замену, если документ не найден, то вставку
var result = await collection.ReplaceOneAsync(filter, newDocument, new ReplaceOptions { IsUpsert=true });
```
## UpdateOneAsync

Если **ReplaceOne/ReplaceOneAsync** полностью заменяют весь документу, то методы **UpdateOne()/UpdateOneAsync()** позволяют обновить только отдельные поля документа:
```c#
// определяем фильтр - документ, где Name = Tom
var filter = new BsonDocument("Name", "Tom"); 
// определяем параметры обновления - обновляем только поле Age
var updateSettings = new BsonDocument("$set", new BsonDocument("Age", 39));
// выполняем обновление
var result = await collection.UpdateOneAsync(filter, updateSettings);
```

## Удаление поля

Для удаления отдельного поля используется оператор $unset. Например, удалим поле "Age":
```c#	
var result = await collection.UpdateOneAsync(
    new BsonDocument("Name", "Alexander"),
    new BsonDocument("$unset", new BsonDocument("Age", 1)));
```
Для удаляемого поля в качестве значения передается единица. Если вдруг подобного поля в документе не существует, то оператор не оказывает никакого влияния. Также можно удалять сразу несколько полей:
```c#	
var result = await collection.UpdateOneAsync(
    new BsonDocument("Name", "Sam"),
    new BsonDocument("$unset", new BsonDocument { { "Age", 1 }, { "Languages", 1 } }));
```
## Обновление массивов. Оператор $push
Оператор $push позволяет добавить еще одно значение в массив. Если поле-массив отсутствует, то оно автоматически создается. Например, добавим значение в поле Languages:
```c#	
var result = await collection.UpdateOneAsync(
    new BsonDocument("Name", "Sam"),
    new BsonDocument("$push", new BsonDocument { { "Languages", "english" }}));
```
Используя оператор $each, можно добавить сразу несколько значений:
```c#	
var result = await collection.UpdateOneAsync(
    new BsonDocument("Name", "Sam"),
    new BsonDocument("$push", 
            new BsonDocument("Languages", 
                new BsonDocument("$each", new BsonArray {"french", "german"}))));    
```                
## Оператор $addToSet
Оператор $addToSet подобно оператору $push добавляет объекты в массив. Отличие состоит в том, что $addToSet добавляет данные, если их еще нет в массиве ( при добавлении через $push данные дублируются, если добавляются элементы, которые уже есть в массиве):
```c#	
var result = await collection.UpdateOneAsync(
    new BsonDocument("Name", "Sam"),
    new BsonDocument("$addToSet", new BsonDocument("Languages", "italian")));
```
## Удаление элемента из массива
Оператор **$pop** позволяет удалять элемент из массива:
```c#
var result = await collection.UpdateOneAsync(
    new BsonDocument("Name", "Sam"),
    new BsonDocument("$pop", new BsonDocument("Languages", 1)));
```
Указывая для поля Languages значение 1, мы удаляем первый элемент с конца. Чтобы удалить первый элемент сначала массива, надо передать отрицательное значение:
```c#	
var result = await collection.UpdateOneAsync(
    new BsonDocument("Name", "Sam"),
    new BsonDocument("$pop", new BsonDocument("Languages", -1)));
```
Несколько иное действие предполагает оператор **$pull**. Он удаляет каждое вхождение элемента в массив. Например, через оператор **$push** мы можем добавить одно и то же значение в массив несколько раз. И теперь с помощью **$pull** удалим его:
```c#	
var result = await collection.UpdateOneAsync(
    new BsonDocument("Name", "Sam"),
    new BsonDocument("$pull", new BsonDocument("Languages", "german")));
```
А если мы хотим удалить не одно значение, а сразу несколько, тогда мы можем применить оператор **$pullAll**:
```c#	
var result = await collection.UpdateOneAsync(
    new BsonDocument("Name", "Sam"),
    new BsonDocument("$pullAll", new BsonDocument("Languages", new BsonArray { "english", "french" })));
```    
## UpdateManyAsync
Для обновления множества документов применяются методы **UpdateMany()/UpdateManyAsync()**, использование которых аналогично UpdateOne/UpdateOneAsync

## UpdateDefinitionBuilder
Для обновления документов вместо BsonDocument можно использовать объект UpdateDefinitionBuilder, у которого определяется ряд методов, соответствющих консольным операторам mongodb:
-    Set: изменяет значение поля
-    AddToSet: добавляет новые элементы в поле документа, которое представляет массив. Например, добавляем в массив Languages новую строку: ``Builders<BsonDocument>.Update.AddToSet("Languages", "spanish")``
-    Inc: инкрементирует значение числового свойства на указанное число единиц. Например, увеличим значение свойства Age на две единицы: ``Builders<BsonDocument>.Update.Inc("Age", 2)``
-    Mul: умножает значение числового свойства на указанное число
-    Push: добавляет новые элементы для ключа, который представляет массив. Например: ``Builders<BsonDocument>.Update.Push("Languages", "french")``
-    Pull: удаляет определенный элемент из массива. Например: ``Builders<BsonDocument>.Update.Pull("Languages", "french")``
-    Unset: удаляет ключ и его значение. Например: ``Builders<BsonDocument>.Update.Unset("Age")``
-    PopFirst: выбирает первый элемент для свойства, которое представляет массив
-    PopLast: выбирает последний элемент для свойства, которое представляет массив
-    Rename: переименовывает название ключа в документе. Например, переименуем поле Age в Year: ``Builders<BsonDocument>.Update.Rename("Age", "Year");.``

Например, применим метод Set(). Этот метод в качестве парамета принимает свойство, которое надо изменить, и его обновленное значение. 
```c#
// параметр фильтрации 
var filter = Builders<BsonDocument>.Filter.Eq("Name", "Tom");
// параметр обновления
var update = Builders<BsonDocument>.Update.Set("Age", 30);
var result = await collection.UpdateManyAsync(filter, update);
```
При работе с методом Set также надо учитывать, что если такого поля еще нет в документе в базе данных, например, оно просто ранее не было установлено, то данное поле добавляется в документ.

C помощью вспомогательного метода **CurrentDate()**, который имеется в UpdateDefinitionBuilder, можно установить у документа время последнего изменения:
```c#	
var filter = Builders<BsonDocument>.Filter.Eq("Name", "Tom");
var update = Builders<BsonDocument>.Update.Set("Age", 33).CurrentDate("LastModified");
var result = await collection.UpdateManyAsync(filter, update);
```
В данном случае к документу добавляется поле LastModified, которое хранит время обновления.
## FindOneAndReplaceAsync
Метод FindOneAndReplaceAsync() ищет документ, который соответствует критерию. И если документ найден, полностью заменяет его другим и возвращает его в качестве результата. Если документ не найден, ничего не делает и просто возвращает null:
```c#
// заменяем документ с Name="Tom" на {{"Name", "Tomas"}, { "Age", 22}}
var result = await collection.FindOneAndReplaceAsync(new BsonDocument("Name", "Tom"), new(){{"Name", "Tomas"}, { "Age", 22}});
```
В данном случае документ с "Name"="Tom" заменяется на документ {{"Name", "Tomas"}, { "Age", 22}}. В качестве результата возвращается старый документ, который был заменен. Если нам надо возвращать новый документ, на который произошла замена, то необходимо в метод в качестве третьего параметра передать объект FindOneAndReplaceOptions со свойством ReturnDocument = ReturnDocument.After.
```c#
var result = await collection.FindOneAndReplaceAsync(
    new BsonDocument("Name", "Tomas"),          // фильтр
    new BsonDocument { { "Name", "Tom" }, { "Age", 38 } },      // на какой документ заменяем
    new FindOneAndReplaceOptions<BsonDocument>() { ReturnDocument = ReturnDocument.After });
```
## FindOneAndUpdateAsync
Метод FindOneAndUpdateAsync() ищет документ, который соответствует критерию. И если документ найден, полностью обновляет его в соответствии с параметрами обновления. В качестве результата по умолчанию возвращаются поля документа до обновления другим. Если документ, который соответствует фильтру, не найден, то метод просто возвращает null:
```c#
var result = await collection.FindOneAndUpdateAsync(
    new BsonDocument("Name", "Tom"),          // фильтр
    new BsonDocument("$set", new BsonDocument("Age", 39)));
  ```
  ## Удаление документов
  Для удаления документов применяются методы 
  - DeleteOne()/DeleteOneAsync() 
  - DeleteMany()/DeleteManyAsync()
  
  определенные в интерфейсе IMongoCollection. В качестве параметра они принимают фильтр, который определяет документ для удаления.

Метод DeleteOne()/DeleteOneAsync() удаляет один документ
```c#
// удаляем документ с Name="Sam"
var result = await collection.DeleteOneAsync(new BsonDocument("Name", "Sam"));
```
Методы DeleteOne()/DeleteOneAsync()/DeleteMany()/DeleteManyAsync() возвращают объект DeleteResult, в котором с помощью свойства DeletedCount можно узнать количество удаленных документов.

Методы DeleteMany()/DeleteManyAsync() работают аналогично, только удаляют все документы, который соответствуют фильтру:
```c#
// удаляем все документы с Name="Tom"
var result = await collection.DeleteManyAsync(new BsonDocument("Name", "Tom"));
Console.WriteLine($"Удалено документов: {result.DeletedCount}");
```
## FindOneAndDeleteAsync
Дополнительно для удаления одного объекта можно применять методы **FindOneAndDelete()/FindOneAndDeleteAsync()**. Они принимают как минимум один параметр - фильтр документов для удаления. И, документ для удаления найден, удаляют его и возвращают в качестве результата. Если документ не найден, то возвращается null.
```c#
var result = await collection.FindOneAndDeleteAsync(new BsonDocument("Name", "Tom"));
```
## Метод BulkWriteAsync
В предыдущих темах было рассмотрено, как использовать различные методы для операций записи, то есть добавления, редактирования, удаления. Для каждой операции есть пара методов, которая работает с одним или несколькими документами: UpdateOneAsync() / UpdateManyAsync(), InsertOneAsync() / InsertManyAsync() и DeleteOneAsync() / DeleteManyAsync(). Однако для увеличения производительности для массовой записи данных мы также можем применять метод BulkWriteAsync()

Метод BulkWriteAsync() в качестве параметра принимает массив объектов WriteModel, которые описывают разные операции:
-    DeleteOneModel / DeleteManyModel: удаление
-    UpdateOneModel / UpdateManyModel: обновление
-    InsertOneModel: добавление
-    ReplaceOneModel: замена
```c#
await collection.BulkWriteAsync(new WriteModel<BsonDocument>[]
{
    new InsertOneModel<BsonDocument>(new BsonDocument{{"Name", "Tom"}, {"Age", 38 } })
});
```
### BulkWriteResult
Метод BulkWriteAsync возвращает объект BulkWriteResult, который содержит результат операций в своих свойствах:
-    Upserts: содержит коллекцию добавленных элементов
-    InsertedCount: количество добавленных документов
-    MatchedCount: количество документов, которые соответствуют фильтру
-    ModifiedCount: количество обновленных документов
-    DeletedCount: количество удаленных документов
## Хранение файлов в базе данных и GridFS
Для хранения больших объемов информации, в частности, файлов, в MongoDB используется система GridFS. Для работы с ней прежде всего необходимо дополнительно установить через Nuget пакет MongoDB.Driver.GridFS
Сохранение файлов в базу данных

Для добавления файлов в GridFS используется ряд методов, которые позволяют загружать либо данные из потока, либо массив байтов:
-    UploadFromBytes/UploadFromBytesAsync: загружает файл в GridFS в виде массива байтов.
-    UploadFromStream/UploadFromStreamAsync: получает данные из потока
```c#
using MongoDB.Bson;
using MongoDB.Driver;
using MongoDB.Driver.GridFS;
 
MongoClient client = new MongoClient("mongodb://localhost:27017");
 
var db = client.GetDatabase("test"); 
IGridFSBucket gridFS = new GridFSBucket(db);
 
// создаем поток из файла
using Stream fs = File.OpenRead("D:\\cats.jpg");
// сохраняем в бд
ObjectId id = await gridFS.UploadFromStreamAsync("cats.jpg", fs);
Console.WriteLine($"id файла: {id}");
```
### Чтение файлов из GridFS
Для загрузки файла из GridFS также применяется ряд методов в зависимости от того, во что мы загружаем файл - в массив байтов или в поток:
-    DownloadToStream/DownloadToStreamAsync: загрузка файла по id из GridFS в поток
-    DownloadAsBytes/DownloadAsBytesAsync: загрузка файла по id в виде массива байтов
-    DownloadToStreamByName/DownloadToStreamByNameAsync: загрузка файла по имени из GridFS в поток
-    DownloadAsBytesByName/DownloadAsBytesByNameAsync: загрузка файла по id в виде массива байтов
### Поиск файла
С помощью методов Find/FindAsync мы можем найти файл в GridFS, а точнее получить различные его свойства. В качестве параметра эти методы принимают объект FilterDefinition\<GridFSFileInfo\>, который определяет критерии поиска.

Результатом методов Find/FindAsync является объект IAsyncCursor\<GridFSFileInfo\> - список информации о файлах. Используя объект GridFSFileInfo, мы можем извлечь информацию о файле с помощью ряда его свойств:
-    Id: идентификатор файла
-    Filename: имя файла
-    UploadDateTime: дата и время загрузки файла (в виде объекта DateTime) файла
-    Length: длина файла в байтах
-    BackingDocument: документ BsonDocument, который представляет данный файл
-    ChunkSizeBytes: размер одного чанка файла в байтах
### Замена файла
Для замены файла надо повторно его загрузить с помощью одного из методов UploadXXX и при загрузке указать параметр замены
### Удаление файлов
Для удаления файла используются методы Delete/DeleteAsync, которые удаляют файл по id

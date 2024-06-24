# [Automapper](https://docs.automapper.org/en/latest/)

Сопоставитель объектов на основе соглашений.

AutoMapper использует гибкий API конфигурации для определения стратегии сопоставления объектов. AutoMapper использует основанный на соглашениях алгоритм сопоставления для сопоставления значений источника и назначения. AutoMapper ориентирован на сценарии проецирования моделей для сведения сложных объектных моделей к DTO и другим простым объектам, конструкция которых лучше подходит для сериализации, связи, обмена сообщениями или просто для защиты от коррупции уровня между уровнем домена и приложения.


## Конфигурация (Configuration)
Создайте экземпляр MapperConfiguration и инициализируйте конфигурацию с помощью конструктора:

```c#
var config = new MapperConfiguration(cfg => {
    cfg.CreateMap<Foo, Bar>();
    cfg.AddProfile<FooProfile>();
});
```
Экземпляр ``MapperConfiguration`` может храниться статически, в статическом поле или в контейнере внедрения зависимостей. ***После создания его нельзя изменить/модифицировать***.
```c#
var configuration = new MapperConfiguration(cfg => {
    cfg.CreateMap<Foo, Bar>();
    cfg.AddProfile<FooProfile>();
});
```
**Начиная с версии 9.0 статический API больше не доступен.**

## Экземпляры профиля (Profile Instances)
Хороший способ организовать конфигурации сопоставления — использовать профили. Создайте классы, наследуемые от ``Profile``, и поместите конфигурацию в конструктор:
```c#
// This is the approach starting with version 5
public class OrganizationProfile : Profile
{
	public OrganizationProfile()
	{
		CreateMap<Foo, FooDto>();
		// Use CreateMap... Etc.. here (Profile methods are the same as configuration methods)
	}
}

// How it was done in 4.x - as of 5.0 this is obsolete:
// public class OrganizationProfile : Profile
// {
//     protected override void Configure()
//     {
//         CreateMap<Foo, FooDto>();
//     }
// }
```
В более ранних версиях вместо конструктора использовался метод Configure. Начиная с версии 5, функция Configure() устарела. Он будет удален в версии 6.0.

Конфигурация внутри профиля применяется только к картам внутри профиля. Конфигурация, примененная к корневой конфигурации, применяется ко всем созданным картам.

## Сканирование сборки для автоматической настройки (Assembly Scanning for auto configuration)
Профили можно добавить в основную конфигурацию картографа несколькими способами: 

- напрямую:

```c#
cfg.AddProfile<OrganizationProfile>();
cfg.AddProfile(new OrganizationProfile());
```
- или путем автоматического сканирования профилей:
```c#
// Scan for all profiles in an assembly
// ... using instance approach:
var config = new MapperConfiguration(cfg => {
    cfg.AddMaps(myAssembly);
});
var configuration = new MapperConfiguration(cfg => cfg.AddMaps(myAssembly));

// Can also use assembly names:
var configuration = new MapperConfiguration(cfg =>
    cfg.AddMaps(new [] {
        "Foo.UI",
        "Foo.Core"
    });
);

// Or marker types for assemblies:
var configuration = new MapperConfiguration(cfg =>
    cfg.AddMaps(new [] {
        typeof(HomeController),
        typeof(Entity)
    });
);
```
AutoMapper просканирует назначенные сборки на наличие классов, унаследованных от ``Profile``, и добавит их в конфигурацию.
## Соглашения об именах (Naming Conventions)
Вы можете установить соглашения об именах источника и места назначения.
```c#
var configuration = new MapperConfiguration(cfg => {
  cfg.SourceMemberNamingConvention = LowerUnderscoreNamingConvention.Instance;
  cfg.DestinationMemberNamingConvention = PascalCaseNamingConvention.Instance;
});
```
Это сопоставит следующие свойства друг с другом:  ``property_name -> PropertyName``.
Вы также можете установить это для каждого профиля
```c#
public class OrganizationProfile : Profile
{
  public OrganizationProfile()
  {
    SourceMemberNamingConvention = LowerUnderscoreNamingConvention.Instance;
    DestinationMemberNamingConvention = PascalCaseNamingConvention.Instance;
    //Put your CreateMap... Etc.. here
  }
}
```
Если вам не нужно соглашение об именах, вы можете использовать ``ExactMatchNamingConvention``.
## Замена символов (Replacing characters)
Вы также можете заменить отдельные символы или целые слова в исходных элементах во время сопоставления имен элементов:
```c#
public class Source
{
    public int Value { get; set; }
    public int Ävíator { get; set; }
    public int SubAirlinaFlight { get; set; }
}
public class Destination
{
    public int Value { get; set; }
    public int Aviator { get; set; }
    public int SubAirlineFlight { get; set; }
}
```
Мы хотим заменить отдельные символы и, возможно, перевести слово:
```c#
var configuration = new MapperConfiguration(c =>
{
    c.ReplaceMemberName("Ä", "A");
    c.ReplaceMemberName("í", "i");
    c.ReplaceMemberName("Airlina", "Airline");
});
```
## Распознавание пре/постфиксов (Recognizing pre/postfixes)
Иногда ваши свойства источника/назначения будут иметь общие пре/постфиксы, из-за чего вам придется выполнять кучу пользовательских сопоставлений элементов, поскольку имена не совпадают. Чтобы решить эту проблему, вы можете распознать пре/постфиксы:
```c#
public class Source {
    public int frmValue { get; set; }
    public int frmValue2 { get; set; }
}
public class Dest {
    public int Value { get; set; }
    public int Value2 { get; set; }
}
var configuration = new MapperConfiguration(cfg => {
    cfg.RecognizePrefixes("frm");
    cfg.CreateMap<Source, Dest>();
});
configuration.AssertConfigurationIsValid();
```
По умолчанию AutoMapper распознает префикс «Get», если вам нужно очистить префикс:
```c#
var configuration = new MapperConfiguration(cfg => {
    cfg.ClearPrefixes();
    cfg.RecognizePrefixes("tmp");
});
```
## Глобальная фильтрация свойств/полей (Global property/field filtering)
По умолчанию AutoMapper пытается сопоставить каждое общедоступное свойство/поле. Вы можете отфильтровать свойства/поля с помощью фильтров свойств/полей:
```c#
var configuration = new MapperConfiguration(cfg =>
{
	// don't map any fields
	cfg.ShouldMapField = fi => false;

	// map properties with a public or private getter
	cfg.ShouldMapProperty = pi =>
		pi.GetMethod != null && (pi.GetMethod.IsPublic || pi.GetMethod.IsPrivate);
});
```
## Настройка видимости (Configuring visibility)
По умолчанию AutoMapper распознает только общедоступных участников. Он может сопоставляться с частными установщиками, но будет пропускать internal/private методы и свойства, если все свойство является internal/private. Чтобы дать указание AutoMapper распознавать элементы с другой видимостью, переопределите фильтры по умолчанию ``MustMapField`` и/или ``MustMapProperty``:
```c#
var configuration = new MapperConfiguration(cfg =>
{
    // map properties with public or internal getters
    cfg.ShouldMapProperty = p => p.GetMethod.IsPublic || p.GetMethod.IsAssembly;
    cfg.CreateMap<Source, Destination>();
});
```
Конфигурации карты теперь распознают внутренних/частных участников.
## Компиляция конфигурации (Configuration compilation)
Поскольку компиляция выражений может потребовать немного ресурсов, AutoMapper лениво компилирует планы карты типов на первой карте. Однако такое поведение не всегда желательно, поэтому вы можете указать AutoMapper напрямую скомпилировать сопоставления:
```c#
var configuration = new MapperConfiguration(cfg => {});
configuration.CompileMappings();
```
Для нескольких сотен сопоставлений это может занять пару секунд. Если это намного больше, у вас, вероятно, есть действительно большие планы выполнения.

## Длительное время компиляции (Long compilation times)
Время компиляции увеличивается с размером плана выполнения и зависит от количества свойств и их сложности. В идеале вы должны исправить свою модель так, чтобы у вас было много небольших DTO, каждый из которых предназначен для определенного варианта использования. Но вы также можете уменьшить размер плана выполнения, не меняя классы.

Вы можете установить ``MapAtRuntime`` для каждого члена или ``MaxExecutionPlanDepth`` глобально (по умолчанию установлено значение 1, установите его равным нулю).

Это уменьшит размер плана выполнения, заменив план выполнения дочернего объекта вызовом метода. Компиляция будет быстрее, но само отображение может быть медленнее. Найдите в репозитории более подробную информацию и используйте профилировщик, чтобы лучше понять эффект. Также помогает отказ от ``PreserveReferences`` и ``MaxDepth``.

## Проверка конфигурации(Configuration Validation)
Создаваемый вручную код сопоставления, хотя и утомителен, имеет то преимущество, что его можно тестировать. Одной из идей создания AutoMapper было устранение не только пользовательского кода сопоставления, но и необходимость ручного тестирования. Поскольку сопоставление источника и места назначения основано на соглашениях, вам все равно потребуется протестировать конфигурацию.

AutoMapper обеспечивает тестирование конфигурации в форме метода ``AssertConfigurationIsValid``. Предположим, мы немного неправильно настроили типы источника и назначения:
```c#
public class Source
{
	public int SomeValue { get; set; }
}

public class Destination
{
	public int SomeValuefff { get; set; }
}
```
В типе Destination мы, вероятно, перепутали свойство назначения. Другими типичными проблемами являются переименования исходных элементов. Чтобы протестировать нашу конфигурацию, мы просто создаем модульный тест, который настраивает конфигурацию и выполняет метод ``AssertConfigurationIsValid``:
```c#
var configuration = new MapperConfiguration(cfg =>
  cfg.CreateMap<Source, Destination>());

configuration.AssertConfigurationIsValid();
```
Выполнение этого кода создает исключение ``AutoMapperConfigurationException`` с описательным сообщением. AutoMapper проверяет, имеет ли каждый элемент типа назначения соответствующий элемент типа исходного типа.

## Переопределение ошибок конфигурации (Overriding configuration errors)
Чтобы исправить ошибку конфигурации (помимо переименования элементов источника/назначения), у вас есть три варианта предоставления альтернативной конфигурации:
- Пользовательские преобразователи значений
- Проекция
- Используйте опцию Ignore()

При третьем варианте у нас есть член типа назначения, который мы будем заполнять альтернативными способами, а не с помощью операции Map.
```c#
var configuration = new MapperConfiguration(cfg =>
  cfg.CreateMap<Source, Destination>()
	.ForMember(dest => dest.SomeValuefff, opt => opt.Ignore())
);
```
## Выбор участников для проверки (Selecting members to validate)
По умолчанию AutoMapper использует тип назначения для проверки элементов. Предполагается, что все элементы назначения должны быть сопоставлены. Чтобы изменить это поведение, используйте перегрузку ``CreateMap``, чтобы указать, какой список участников следует проверять:

```c#
var configuration = new MapperConfiguration(cfg =>
  cfg.CreateMap<Source, Destination>(MemberList.Source);
  cfg.CreateMap<Source2, Destination2>(MemberList.None);
);
```
Чтобы вообще пропустить проверку для этой карты, используйте ``MemberList.None``. Это значение по умолчанию для ``ReverseMap``.

## Пользовательские проверки (Custom validations)

Вы можете добавить пользовательские проверки через точку расширения. [Глянь сюда](https://github.com/AutoMapper/AutoMapper/blob/bdc0120497d192a2741183415543f6119f50a982/src/UnitTests/CustomValidations.cs#L42).

# Внедрение зависимости (Dependency Injection)
Существует пакет NuGet, который можно использовать с механизмом внедрения по умолчанию, описанным здесь и используемым в этом проекте.

Начиная с версии 13.0, AddAutoMapper является частью основного пакета, а поддержка пакета DI прекращена.

Вы определяете конфигурацию с помощью профилей. Затем вы сообщаете ``AutoMapper``, в каких сборках определены эти профили, вызывая метод расширения ``IServiceCollection`` ``AddAutoMapper`` при запуске:

```services.AddAutoMapper(profileAssembly1, profileAssembly2 /*, ...*/);```

или типы маркеров:

```services.AddAutoMapper(typeof(ProfileTypeFromAssembly1), typeof(ProfileTypeFromAssembly2) /*, ...*/);```

Теперь вы можете внедрить AutoMapper во время выполнения в свои службы/контроллеры:

```c#
public class EmployeesController {
	private readonly IMapper _mapper;

	public EmployeesController(IMapper mapper) => _mapper = mapper;

	// use _mapper.Map or _mapper.ProjectTo
}
```
## AutoFac
Существует сторонний пакет NuGet, который вы можете попробовать.

Также проверьте [этот блог](https://dotnetfalcon.com/autofac-support-for-automapper/).

## Другие DI (Other DI engines)
   
[примеры](https://github.com/AutoMapper/AutoMapper/wiki/DI-examples)


## API-интерфейсы низкого уровня (Low level API-s)

AutoMapper поддерживает возможность создания пользовательских преобразователей значений, пользовательских преобразователей типов и преобразователей значений с использованием статического местоположения службы:
```c#
var configuration = new MapperConfiguration(cfg =>
{
    cfg.ConstructServicesUsing(ObjectFactory.GetInstance);

    cfg.CreateMap<Source, Destination>();
});
```
Или динамическое расположение службы, которое будет использоваться в случае контейнеров на основе экземпляров (включая дочерние/вложенные контейнеры):
```c#
var mapper = new Mapper(configuration, childContainer.GetInstance);

var dest = mapper.Map<Source, Destination>(new Source { Value = 15 });
```
## Запрашиваемые расширения (Queryable Extensions)

Начиная с версии 8.0 вы можете использовать ``IMapper.ProjectTo``. Для более старых версий вам необходимо передать конфигурацию методу расширения ``IQueryable.ProjectToT(IConfigurationProvider)``.

Обратите внимание, что ``ProjectTo`` более ограничен, чем ``Map``, поскольку поддерживается только то, что разрешено базовым поставщиком ``LINQ``. Это означает, что вы не можете использовать DI с преобразователями и преобразователями значений, как с Map.

## Проекция (Projection)
Проекция преобразует источник в пункт назначения, выходя за рамки выравнивания объектной модели. Без дополнительной настройки ``AutoMapper`` требует, чтобы плоский пункт назначения соответствовал структуре именования исходного типа. Если вы хотите спроецировать исходные значения в место назначения, которое не совсем соответствует исходной структуре, необходимо указать пользовательские определения сопоставления элементов. Например, мы можем захотеть превратить эту исходную структуру:
```c#
public class CalendarEvent
{
	public DateTime Date { get; set; }
	public string Title { get; set; }
}
```
Into something that works better for an input form on a web page:
```c#
public class CalendarEventForm
{
	public DateTime EventDate { get; set; }
	public int EventHour { get; set; }
	public int EventMinute { get; set; }
	public string Title { get; set; }
}
```
Поскольку имена целевых свойств не совсем совпадают с исходным свойством (``CalendarEvent.Date`` должно быть ``CalendarEventForm.EventDate``), нам необходимо указать пользовательские сопоставления членов в нашей конфигурации карты типов:
```C#
// Model
var calendarEvent = new CalendarEvent
{
	Date = new DateTime(2008, 12, 15, 20, 30, 0),
	Title = "Company Holiday Party"
};

// Configure AutoMapper
var configuration = new MapperConfiguration(cfg =>
  cfg.CreateMap<CalendarEvent, CalendarEventForm>()
	.ForMember(dest => dest.EventDate, opt => opt.MapFrom(src => src.Date.Date))
	.ForMember(dest => dest.EventHour, opt => opt.MapFrom(src => src.Date.Hour))
	.ForMember(dest => dest.EventMinute, opt => opt.MapFrom(src => src.Date.Minute)));

// Perform mapping
CalendarEventForm form = mapper.Map<CalendarEvent, CalendarEventForm>(calendarEvent);

form.EventDate.ShouldEqual(new DateTime(2008, 12, 15));
form.EventHour.ShouldEqual(20);
form.EventMinute.ShouldEqual(30);
form.Title.ShouldEqual("Company Holiday Party");
```
В каждой конфигурации настраиваемого члена используется делегат действия для настройки каждого отдельного члена. В приведенном выше примере мы использовали параметр ``MapFrom`` для выполнения пользовательских сопоставлений элементов источника и назначения. Метод ``MapFrom`` принимает лямбда-выражение в качестве параметра, которое затем оценивается позже во время сопоставления. Выражение ``MapFrom`` может быть любым ``Func<TSource, object>``, лямбда-выражением объекта.

## Вложенные сопоставления (Nested Mappings)
Когда механизм сопоставления выполняет сопоставление, он может использовать один из множества методов для разрешения значения целевого элемента. Один из этих методов — использовать другую карту типов, где тип исходного элемента и тип целевого элемента также настраиваются в конфигурации сопоставления. Это позволяет нам не только сглаживать исходные типы, но и создавать сложные целевые типы. Например, наш исходный тип может содержать другой сложный тип:
```c#
public class OuterSource
{
	public int Value { get; set; }
	public InnerSource Inner { get; set; }
}

public class InnerSource
{
	public int OtherValue { get; set; }
}
```
Мы могли бы просто свести ``OuterSource.Inner.OtherValue`` к одному свойству ``InnerOtherValue``, но мы также можем захотеть создать соответствующий сложный тип для свойства ``Inner``:
```c#
public class OuterDest
{
	public int Value { get; set; }
	public InnerDest Inner { get; set; }
}

public class InnerDest
{
	public int OtherValue { get; set; }
}
```
В этом случае нам нужно будет настроить дополнительные сопоставления типов источника и места назначения:
```c#
var config = new MapperConfiguration(cfg => {
    cfg.CreateMap<OuterSource, OuterDest>();
    cfg.CreateMap<InnerSource, InnerDest>();
});
config.AssertConfigurationIsValid();

var source = new OuterSource
	{
		Value = 5,
		Inner = new InnerSource {OtherValue = 15}
	};
var mapper = config.CreateMapper();
var dest = mapper.Map<OuterSource, OuterDest>(source);

dest.Value.ShouldEqual(5);
dest.Inner.ShouldNotBeNull();
dest.Inner.OtherValue.ShouldEqual(15);
```
Здесь следует отметить несколько вещей:

- Порядок настройки типов не имеет значения
- При вызове Map не требуется указывать какие-либо сопоставления внутренних типов, а только сопоставление типов, которое будет использоваться для переданного исходного значения.

Используя как сглаживание, так и вложенные сопоставления, мы можем создавать различные формы назначения в соответствии с нашими потребностями.

# Списки и массивы (Lists and Arrays)
AutoMapper требует настройки только типов элементов, а не каких-либо типов массива или списка, которые могут использоваться. Например, у нас может быть простой тип источника и назначения:
```c#
public class Source
{
	public int Value { get; set; }
}

public class Destination
{
	public int Value { get; set; }
}
```
Поддерживаются все основные типы универсальных коллекций:
```c#
var configuration = new MapperConfiguration(cfg => cfg.CreateMap<Source, Destination>());

var sources = new[]
	{
		new Source { Value = 5 },
		new Source { Value = 6 },
		new Source { Value = 7 }
	};

IEnumerable<Destination> ienumerableDest = mapper.Map<Source[], Enumerable<Destination>>(sources);
ICollection<Destination> icollectionDest = mapper.Map<Source[], ICollection<Destination>>(sources);
IList<Destination> ilistDest = mapper.Map<Source[], IList<Destination>>(sources);
List<Destination> listDest = mapper.Map<Source[], List<Destination>>(sources);
Destination[] arrayDest = mapper.Map<Source[], Destination[]>(sources);
```
Если быть точным, поддерживаемые типы исходных коллекций включают:
-   IEnumerable
-   IEnumerable\<T\>
-   ICollection
-   ICollection\<T\>
-   IList
-   IList\<T\>
-   List\<T\>
-   Arrays
Для неуниверсальных перечислимых типов поддерживаются только несопоставленные, назначаемые типы, поскольку ``AutoMapper`` не сможет «угадать», какие типы вы пытаетесь сопоставить. Как показано в примере выше, нет необходимости явно настраивать типы списков, а только типы их членов.

При сопоставлении с существующей коллекцией сначала очищается целевая коллекция. Если это не то, что вам нужно, взгляните на ``AutoMapper.Collection``.

## Handling null collections
При сопоставлении свойства коллекции, если исходное значение равно нулю, ``AutoMapper`` сопоставляет поле назначения с пустой коллекцией, а не присваивает целевому значению значение NULL. Это согласуется с поведением ``Entity Framework`` и ``Framework Design Guidelines``, согласно которым ссылки, массивы, списки, коллекции, словари и IEnumerables C# НИКОГДА не должны быть нулевыми.

Это поведение можно изменить, установив для свойства ``AllowNullCollections`` значение true при настройке сопоставителя.
```c#
var configuration = new MapperConfiguration(cfg => {
    cfg.AllowNullCollections = true;
    cfg.CreateMap<Source, Destination>();
});
```
Этот параметр можно применить глобально и переопределить для каждого профиля и для каждого участника с помощью ``AllowNull`` и ``DoNotAllowNull``.


## Полиморфные типы элементов в коллекциях (Polymorphic element types in collections)
Часто у нас может быть иерархия типов как в исходном, так и в целевом типах. AutoMapper поддерживает полиморфные массивы и коллекции, поэтому, если они найдены, используются производные типы источника/назначения.
```c#
public class ParentSource
{
	public int Value1 { get; set; }
}

public class ChildSource : ParentSource
{
	public int Value2 { get; set; }
}

public class ParentDestination
{
	public int Value1 { get; set; }
}

public class ChildDestination : ParentDestination
{
	public int Value2 { get; set; }
}
```
AutoMapper по-прежнему требует явной настройки дочерних сопоставлений, поскольку AutoMapper не может «угадать», какое конкретное дочернее сопоставление назначения использовать. Вот пример вышеуказанных типов:
```c#
var configuration = new MapperConfiguration(c=> {
    c.CreateMap<ParentSource, ParentDestination>()
	     .Include<ChildSource, ChildDestination>();
    c.CreateMap<ChildSource, ChildDestination>();
});

var sources = new[]
	{
		new ParentSource(),
		new ChildSource(),
		new ParentSource()
	};

var destinations = mapper.Map<ParentSource[], ParentDestination[]>(sources);

destinations[0].ShouldBeInstanceOf<ParentDestination>();
destinations[1].ShouldBeInstanceOf<ChildDestination>();
destinations[2].ShouldBeInstanceOf<ParentDestination>();
```

## Конструирование (Construction)
AutoMapper может сопоставляться с конструкторами назначения на основе исходных членов:
```c#
public class Source {
    public int Value { get; set; }
}
public class SourceDto {
    
    public SourceDto(int valueParamSomeOtherName) {
        _value = valueParamSomeOtherName;
    }

    private int _value;

    public int Value {
        get { return _value; }
    }
}

var configuration = new MapperConfiguration(cfg =>
  cfg.CreateMap<Source, SourceDto>()
    .ForCtorParam("valueParamSomeOtherName", opt => opt.MapFrom(src => src.Value))
);
```
Это работает как для проекций LINQ, так и для сопоставлений в памяти.

Вы также можете отключить отображение конструктора:

``var configuration = new MapperConfiguration(cfg => cfg.DisableConstructorMapping());``

Вы можете настроить, какие конструкторы считаются целевым объектом:
```c#
// use only public constructors
var configuration = new MapperConfiguration(cfg => cfg.ShouldUseConstructor = constructor => constructor.IsPublic);
```
При сопоставлении с записями рассмотрите возможность использования только общедоступных конструкторов.

## Cглаживание (Flattening)
Одним из распространенных способов сопоставления объектов является преобразование сложной объектной модели в более простую. Можно взять сложную модель, например:

```c#
public class Order
{
	private readonly IList<OrderLineItem> _orderLineItems = new List<OrderLineItem>();

	public Customer Customer { get; set; }

	public OrderLineItem[] GetOrderLineItems()
	{
		return _orderLineItems.ToArray();
	}

	public void AddOrderLineItem(Product product, int quantity)
	{
		_orderLineItems.Add(new OrderLineItem(product, quantity));
	}

	public decimal GetTotal()
	{
		return _orderLineItems.Sum(li => li.GetTotal());
	}
}

public class Product
{
	public decimal Price { get; set; }
	public string Name { get; set; }
}

public class OrderLineItem
{
	public OrderLineItem(Product product, int quantity)
	{
		Product = product;
		Quantity = quantity;
	}

	public Product Product { get; private set; }
	public int Quantity { get; private set;}

	public decimal GetTotal()
	{
		return Quantity*Product.Price;
	}
}

public class Customer
{
	public string Name { get; set; }
}
```
Мы хотим преобразовать этот сложный объект Order в более простой OrderDto, содержащий только данные, необходимые для определенного сценария:
```c#
public class OrderDto
{
	public string CustomerName { get; set; }
	public decimal Total { get; set; }
}
```
Когда вы настраиваете пару типов источника и назначения в ``AutoMapper``, конфигуратор пытается сопоставить свойства и методы исходного типа со свойствами целевого типа. Если для какого-либо свойства целевого типа свойство, метод или метод с префиксом «Get» не существует в исходном типе, AutoMapper разбивает имя целевого элемента на отдельные слова (согласно соглашениям ``PascalCase``).
```c#
// Complex model

var customer = new Customer
	{
		Name = "George Costanza"
	};
var order = new Order
	{
		Customer = customer
	};
var bosco = new Product
	{
		Name = "Bosco",
		Price = 4.99m
	};
order.AddOrderLineItem(bosco, 15);

// Configure AutoMapper

var configuration = new MapperConfiguration(cfg => cfg.CreateMap<Order, OrderDto>());

// Perform mapping

OrderDto dto = mapper.Map<Order, OrderDto>(order);

dto.CustomerName.ShouldEqual("George Costanza");
dto.Total.ShouldEqual(74.85m);
```
Мы настроили карту типов в ``AutoMapper`` с помощью метода ``CreateMap``. ``AutoMapper`` может сопоставлять только те пары типов, о которых он знает, поэтому мы явно зарегистрировали пару типов источника/назначения с помощью ``CreateMap``. Для выполнения сопоставления мы используем метод Map.

В типе OrderDto свойство Total соответствует методу ``GetTotal()`` объекта Order. Свойство CustomerName соответствует свойству Customer.Name в заказе. Если мы правильно назовем наши целевые свойства, нам не нужно настраивать сопоставление отдельных свойств.

Если вы хотите отключить это поведение, вы можете использовать ``ExactMatchNamingConvention``:
``cfg.DestinationMemberNamingConvention = new ExactMatchNamingConvention();``


## IncludeMembers
Если вам нужно больше контроля при выравнивании, вы можете использовать IncludeMembers. Вы можете сопоставить элементы дочернего объекта с целевым объектом, если у вас уже есть сопоставление дочернего типа с целевым типом (в отличие от классического выравнивания, которое не требует сопоставления дочернего типа).
```c#
class Source
{
    public string Name { get; set; }
    public InnerSource InnerSource { get; set; }
    public OtherInnerSource OtherInnerSource { get; set; }
}
class InnerSource
{
    public string Name { get; set; }
    public string Description { get; set; }
}
class OtherInnerSource
{
    public string Name { get; set; }
    public string Description { get; set; }
    public string Title { get; set; }
}
class Destination
{
    public string Name { get; set; }
    public string Description { get; set; }
    public string Title { get; set; }
}

cfg.CreateMap<Source, Destination>().IncludeMembers(s=>s.InnerSource, s=>s.OtherInnerSource);
cfg.CreateMap<InnerSource, Destination>(MemberList.None);
cfg.CreateMap<OtherInnerSource, Destination>();

var source = new Source { Name = "name", InnerSource = new InnerSource{ Description = "description" }, 
                          OtherInnerSource = new OtherInnerSource{ Title = "title" } };
var destination = mapper.Map<Destination>(source);
destination.Name.ShouldBe("name");
destination.Description.ShouldBe("description");
destination.Title.ShouldBe("title");
```
Таким образом, это позволяет вам повторно использовать конфигурацию существующей карты для дочерних типов `InnerSource` и `OtherInnerSource` при сопоставлении родительских типов `Source` и `Destination`. Он работает аналогично отображению наследования, но использует композицию, а не наследование.

Порядок параметров в вызове `IncludeMembers` имеет значение. При сопоставлении целевого элемента выигрывает первое совпадение, начиная с самого исходного объекта, а затем с включенными дочерними объектами в указанном вами порядке. Итак, в приведенном выше примере Имя сопоставляется с самим исходным объектом, а Описание — с `InnerSource`, поскольку это первое совпадение.

Обратите внимание, что это сопоставление является статическим, оно происходит во время конфигурации, а не во время отображения, поэтому типы времени выполнения дочерних объектов не учитываются.

Обратите внимание, что это сопоставление является статическим, оно происходит во время конфигурации, а не во время отображения, поэтому типы времени выполнения дочерних объектов не учитываются.

``ForPath(destination => destination.IncludedMember, member => member.MapFrom(source => source))``

и наоборот. Если это не то, что вам нужно, вы можете избежать `ReverseMap` (явно создать обратную карту) или переопределить настройки по умолчанию (используя Ignore или `IncludeMembers` без параметров соответственно).

Подробности [смотрите в тестах](https://github.com/AutoMapper/AutoMapper/blob/master/src/UnitTests/IMappingExpression/IncludeMembers.cs).


## Обратное сопоставление и разглаживание (Reverse Mapping and Unflattening)
Начиная с версии 6.1.0, AutoMapper теперь поддерживает более широкую поддержку обратного сопоставления. Учитывая наши сущности:
```c#
public class Order {
  public decimal Total { get; set; }
  public Customer Customer { get; set; }
}

public class Customer {
  public string Name { get; set; }
}

// Мы можем свести это в DTO:

public class OrderDto {
  public decimal Total { get; set; }
  public string CustomerName { get; set; }
}

// Мы можем сопоставить оба направления, включая разглаживание:

var configuration = new MapperConfiguration(cfg => {
  cfg
    .CreateMap<Order, OrderDto>()
    .ReverseMap();
});

```
Вызывая `ReverseMap`, `AutoMapper` создает конфигурацию обратного сопоставления, включающую разглаживание:
```c#
var customer = new Customer {
  Name = "Bob"
};

var order = new Order {
  Customer = customer,
  Total = 15.8m
};

var orderDto = mapper.Map<Order, OrderDto>(order);

orderDto.CustomerName = "Joe";

mapper.Map(orderDto, order);

order.Customer.Name.ShouldEqual("Joe");
```
Разглаживание настраивается только для `ReverseMap`. Если вы хотите выполнить разведение, вы должны настроить `Entity -> Dto`, а затем вызвать `ReverseMap`, чтобы создать конфигурацию карты типов разведения из `Dto -> Entity`.

## Настройка обратного сопоставления (Customizing reverse mapping)
`AutoMapper` автоматически перевернет сопоставление «Customer.Name» с «CustomerName» на основе исходного выравнивания. Если вы используете `MapFrom`, `AutoMapper` попытается перевернуть карту:

```c#
cfg.CreateMap<Order, OrderDto>()
  .ForMember(d => d.CustomerName, opt => opt.MapFrom(src => src.Customer.Name))
  .ReverseMap();
```
Пока путь `MapFrom` является средством доступа к членам, `AutoMapper` будет выполнять развертывание по тому же пути (CustomerName = Customer.Name).
Если вам нужно настроить это, для обратной карты вы можете использовать `ForPath`:
```c#
cfg.CreateMap<Order, OrderDto>()
  .ForMember(d => d.CustomerName, opt => opt.MapFrom(src => src.Customer.Name))
  .ReverseMap()
  .ForPath(s => s.Customer.Name, opt => opt.MapFrom(src => src.CustomerName));
```
В большинстве случаев вам это не понадобится, поскольку исходный `MapFrom` будет перевернут. Используйте `ForPath`, если пути для получения и установки значений различаются.

Если вы не хотите, чтобы поведение было несглаживаемым, вы можете удалить вызов `ReverseMap` и создать две отдельные карты. Или вы можете использовать `Ignore`:

```c#
cfg.CreateMap<Order, OrderDto>()
  .ForMember(d => d.CustomerName, opt => opt.MapFrom(src => src.Customer.Name))
  .ReverseMap()
  .ForPath(s => s.Customer.Name, opt => opt.Ignore());
```

## IncludeMembers
`ReverseMap` также интегрируется с `IncludeMembers` и такими конфигурациями, как

``ForMember(destination => destination.IncludedMember, member => member.MapFrom(source => source))``

## Сопоставление наследования (Mapping Inheritance)
Наследование сопоставления выполняет две функции:

- Наследование конфигурации сопоставления из базового класса или конфигурации интерфейса.
- Полиморфное отображение во время выполнения

Наследование конфигурации базового класса является добровольным, и вы можете либо явно указать сопоставление для наследования от конфигурации базового типа с помощью `Include`, либо в конфигурации производного типа с помощью `IncludeBase`:
```c#
CreateMap<BaseEntity, BaseDto>()
   .Include<DerivedEntity, DerivedDto>()
   .ForMember(dest => dest.SomeMember, opt => opt.MapFrom(src => src.OtherMember));

CreateMap<DerivedEntity, DerivedDto>();
```
or
```c#
CreateMap<BaseEntity, BaseDto>()
   .ForMember(dest => dest.SomeMember, opt => opt.MapFrom(src => src.OtherMember));

CreateMap<DerivedEntity, DerivedDto>()
    .IncludeBase<BaseEntity, BaseDto>();
```
В каждом вышеописанном случае производное сопоставление наследует пользовательскую конфигурацию сопоставления из базовой карты.

`Include/IncludeBase` применяется рекурсивно, поэтому вам нужно включить только ближайший уровень иерархии.

Если для некоторого базового класса у вас есть много непосредственно производных классов, для удобства вы можете включить все производные карты из конфигурации карты базового типа:
```c#
CreateMap<BaseEntity, BaseDto>()
    .IncludeAllDerived();

CreateMap<DerivedEntity, DerivedDto>();
```
Обратите внимание, что при этом будут выполняться поиск производных типов во всех ваших сопоставлениях, и это будет медленнее, чем явное указание производных сопоставлений.

## Полиморфизм времени выполнения (Runtime polymorphism)

```c#
public class Order { }
public class OnlineOrder : Order { }
public class MailOrder : Order { }

public class OrderDto { }
public class OnlineOrderDto : OrderDto { }
public class MailOrderDto : OrderDto { }

var configuration = new MapperConfiguration(cfg => {
    cfg.CreateMap<Order, OrderDto>()
        .Include<OnlineOrder, OnlineOrderDto>()
        .Include<MailOrder, MailOrderDto>();
    cfg.CreateMap<OnlineOrder, OnlineOrderDto>();
    cfg.CreateMap<MailOrder, MailOrderDto>();
});

// Perform Mapping
var order = new OnlineOrder();
var mapped = mapper.Map(order, order.GetType(), typeof(OrderDto));
Assert.IsType<OnlineOrderDto>(mapped);
```
Вы заметите, что, поскольку сопоставленный объект является `OnlineOrder`, `AutoMapper` увидел, что у вас есть более конкретное сопоставление для `OnlineOrder`, чем `OrderDto`, и автоматически выбрал его.

## Указание наследования в производных классах (Specifying inheritance in derived classes)
Вместо настройки наследования от базового класса вы можете указать наследование от производных классов:
```c#
var configuration = new MapperConfiguration(cfg => {
  cfg.CreateMap<Order, OrderDto>()
    .ForMember(o => o.Id, m => m.MapFrom(s => s.OrderId));
  cfg.CreateMap<OnlineOrder, OnlineOrderDto>()
    .IncludeBase<Order, OrderDto>();
  cfg.CreateMap<MailOrder, MailOrderDto>()
    .IncludeBase<Order, OrderDto>();
});
```

## As
В простых случаях вы можете использовать `As` для перенаправления базовой карты на существующую производную карту:
```c#
    cfg.CreateMap<Order, OnlineOrderDto>();
    cfg.CreateMap<Order, OrderDto>().As<OnlineOrderDto>();
    
    mapper.Map<OrderDto>(new Order()).ShouldBeOfType<OnlineOrderDto>();
```

## Приоритеты сопоставления наследования (Inheritance Mapping Priorities)
Это вносит дополнительную сложность, поскольку существует несколько способов сопоставления свойства. Приоритет этих источников следующий:

- Явное сопоставление (с использованием `.MapFrom()`)
- Унаследованное явное сопоставление
- `Ignore` сопоставление свойств
- Сопоставление соглашений (свойства, сопоставленные с помощью соглашения)

Чтобы продемонстрировать это, давайте изменим наши классы, показанные выше.
```c#
//Domain Objects
public class Order { }
public class OnlineOrder : Order
{
    public string Referrer { get; set; }
}
public class MailOrder : Order { }

//Dtos
public class OrderDto
{
    public string Referrer { get; set; }
}

//Mappings
var configuration = new MapperConfiguration(cfg => {
    cfg.CreateMap<Order, OrderDto>()
        .Include<OnlineOrder, OrderDto>()
        .Include<MailOrder, OrderDto>()
        .ForMember(o=>o.Referrer, m=>m.Ignore());
    cfg.CreateMap<OnlineOrder, OrderDto>();
    cfg.CreateMap<MailOrder, OrderDto>();
});

// Perform Mapping
var order = new OnlineOrder { Referrer = "google" };
var mapped = mapper.Map(order, order.GetType(), typeof(OrderDto));
Assert.IsNull(mapped.Referrer);
```
Обратите внимание, что в нашей конфигурации сопоставления мы проигнорировали `Referrer` (поскольку он не существует в базовом классе заказа), и это имеет более высокий приоритет, чем сопоставление по соглашению, поэтому свойство не отображается.

Если вы хотите, чтобы свойство `Referrer` отображалось в сопоставлении `OnlineOrder` с `OrderDto`, вам следует включить явное сопоставление в сопоставление следующим образом:
```c#
    cfg.CreateMap<OnlineOrder, OrderDto>()
        .ForMember(o=>o.Referrer, m=>m.MapFrom(x=>x.Referrer));
```        
В целом эта функция должна сделать использование `AutoMapper` с классами, использующими наследование, более естественным.


## Сопоставление атрибутов (Attribute Mapping)
В дополнение к гибкой настройке есть возможность объявлять и настраивать карты с помощью атрибутов. Карты атрибутов могут дополнять или заменять конфигурацию плавного сопоставления.

## Тип конфигурации карты
Для поиска карт для настройки используйте метод `AddMaps`:
```c#
var configuration = new MapperConfiguration(cfg => cfg.AddMaps("MyAssembly"));
var mapper = new Mapper(configuration);
```
`AddMaps` ищет свободную конфигурацию карт (классы профиля) и сопоставления на основе атрибутов.

Чтобы объявить карту атрибутов, украсьте тип назначения `AutoMapAttribute`:
```c#
[AutoMap(typeof(Order))]
public class OrderDto {
    // destination members
```
Это эквивалентно конфигурации `CreateMap<Order, OrderDto>()`.    

## Настройка конфигурации карты типов (Customizing type map configuration)
Чтобы настроить общую конфигурацию карты типов, вы можете установить следующие свойства в `AutoMapAttribute`:
-    ReverseMap (bool)
-    ConstructUsingServiceLocator (bool)
-    MaxDepth (int)
-    PreserveReferences (bool)
-    DisableCtorValidation (bool)
-    IncludeAllDerived (bool)
-    TypeConverter (Type)
-    AsProxy (bool)
Все они соответствуют аналогичным параметрам конфигурации плавного сопоставления. Для сопоставления требуется только значение `sourceType`.

## Конфигурация участника (Member configuration)
Для карт на основе атрибутов вы можете украсить отдельные элементы дополнительной конфигурацией. Поскольку атрибуты в C# имеют ограничения (например, нет выражений), доступные параметры конфигурации немного ограничены.

Атрибуты на основе элементов объявляются в пространстве имен `AutoMapper.Configuration.Annotations`.

Если конфигурация на основе атрибутов недоступна или не работает, вы можете объединить карты на основе атрибутов и карт на основе профилей (хотя это может сбить с толку).

## Игнорирование участников (Ignoring members)
Используйте `IgnoreAttribute`, чтобы игнорировать отдельный элемент назначения при сопоставлении и/или проверке.
```c#
using AutoMapper.Configuration.Annotations;

[AutoMap(typeof(Order))]
public class OrderDto {
    [Ignore]
    public decimal Total { get; set; }
```

## Перенаправление на другой исходный элемент (Redirecting to a different source member)
Невозможно использовать `MapFrom` с выражением в атрибуте, но `SourceMemberAttribute` может перенаправить на отдельный именованный элемент:
```c#
using AutoMapper.Configuration.Annotations;

[AutoMap(typeof(Order))]
public class OrderDto {
   [SourceMember("OrderTotal")]
   public decimal Total { get; set; }
```
Или используйте оператор `nameof`:   
```c#
using AutoMapper.Configuration.Annotations;

[AutoMap(typeof(Order))]
public class OrderDto {
   [SourceMember(nameof(Order.OrderTotal))]
   public decimal Total { get; set; }
```
Вы не можете выполнить выравнивание с помощью этого атрибута, а только перенаправлять элементы исходного типа (т.е. в имени нет «Order.Customer.Office.Name»). Настройка сведения доступна только в конфигурации Fluent.

## Дополнительные возможности конфигурации (Additional configuration options)
Дополнительные параметры конфигурации на основе атрибутов включают в себя:
-    MapAtRuntimeAttribute
-    MappingOrderAttribute
-    NullSubstituteAttribute
-    UseExistingValueAttribute
-    ValueConverterAttribute
-    ValueResolverAttribute

Каждый соответствует одному и тому же варианту отображения конфигурации.

## Динамическое и расширенное сопоставление объектов (Dynamic and ExpandoObject Mapping)
`AutoMapper` может сопоставлять динамические объекты и обратно без какой-либо явной настройки:
```c#
public class Foo {
    public int Bar { get; set; }
    public int Baz { get; set; }
    public Foo InnerFoo { get; set; }
}
dynamic foo = new MyDynamicObject();
foo.Bar = 5;
foo.Baz = 6;

var configuration = new MapperConfiguration(cfg => {});

var result = mapper.Map<Foo>(foo);
result.Bar.ShouldEqual(5);
result.Baz.ShouldEqual(6);

dynamic foo2 = mapper.Map<MyDynamicObject>(result);
foo2.Bar.ShouldEqual(5);
foo2.Baz.ShouldEqual(6);
```
Точно так же вы можете сопоставлять объекты прямо из `Dictionary<string, object>`, `AutoMapper` выровняет ключи с именами свойств. Для сопоставления с дочерними объектами назначения вы можете использовать точечную запись.

```c#
var result = mapper.Map<Foo>(new Dictionary<string, object> { ["InnerFoo.Bar"] = 42 });
result.InnerFoo.Bar.ShouldEqual(42);
```


## Открытые дженерики (Open Generics)
AutoMapper может поддерживать карту открытого универсального типа. Создайте карту для открытых универсальных типов:
```c#
public class Source<T> {
    public T Value { get; set; }
}

public class Destination<T> {
    public T Value { get; set; }
}

// Create the mapping
var configuration = new MapperConfiguration(cfg => cfg.CreateMap(typeof(Source<>), typeof(Destination<>)));
```
Вам не нужно создавать карты для закрытых универсальных типов. `AutoMapper` применит любую конфигурацию от открытого универсального сопоставления к закрытому сопоставлению во время выполнения:
```c#
var source = new Source<int> { Value = 10 };

var dest = mapper.Map<Source<int>, Destination<int>>(source);

dest.Value.ShouldEqual(10);
```
Поскольку C# допускает только параметры закрытого универсального типа, вам необходимо использовать версию `CreateMap` `System.Type` для создания карт открытых универсальных типов. Отсюда вы можете использовать всю доступную конфигурацию сопоставления, и открытая универсальная конфигурация будет применена к карте закрытого типа во время выполнения. AutoMapper будет пропускать открытые карты универсальных типов во время проверки конфигурации, поскольку вы все равно можете создавать закрытые типы, которые не преобразуются, например `Source<Foo> -> Destination<Bar>`, где преобразование из Foo в Bar отсутствует.

Вы также можете создать открытый преобразователь универсальных типов:
```c#
var configuration = new MapperConfiguration(cfg =>
   cfg.CreateMap(typeof(Source<>), typeof(Destination<>)).ConvertUsing(typeof(Converter<>)));
```
AutoMapper также поддерживает преобразователи открытых универсальных типов с любым количеством универсальных аргументов:
```c#
var configuration = new MapperConfiguration(cfg =>
   cfg.CreateMap(typeof(Source<>), typeof(Destination<>)).ConvertUsing(typeof(Converter<,>)));
```
Закрытый тип `Source` будет первым универсальным аргументом, а закрытый тип `Destination` будет вторым аргументом закрытия `Converter<,>`.

Та же идея применима и к преобразователям значений. [Смотри тесты](https://github.com/AutoMapper/AutoMapper/blob/e8249d582d384ea3b72eec31408126a0b69619bc/src/UnitTests/OpenGenerics.cs#L11).


## Запрашиваемые расширения (Queryable Extensions)
При использовании ORM, такого как NHibernate или Entity Framework, со стандартными функциями `Mapper.Map` AutoMapper, вы можете заметить, что ORM будет запрашивать все поля всех объектов в графе, когда AutoMapper пытается сопоставить результаты с типом назначения.

Если ваш ORM предоставляет `IQueryables`, вы можете использовать вспомогательные методы `QueryableExtensions` `AutoMapper` для решения этой ключевой проблемы.

Используя Entity Framework в качестве примера, предположим, что у вас есть сущность `OrderLine`, связанная с сущностью `Item`. Если вы хотите сопоставить это с `OrderLineDTO` со свойством `Item’s` `Name`, стандартный вызов `Mapper.Map` приведет к тому, что Entity Framework запросит всю таблицу `OrderLine` и `Item`.

Вместо этого используйте этот подход.

Учитывая следующие сущности:
```c#
public class OrderLine
{
  public int Id { get; set; }
  public int OrderId { get; set; }
  public Item Item { get; set; }
  public decimal Quantity { get; set; }
}

public class Item
{
  public int Id { get; set; }
  public string Name { get; set; }
}

// И следующий DTO:

public class OrderLineDTO
{
  public int Id { get; set; }
  public int OrderId { get; set; }
  public string Item { get; set; }
  public decimal Quantity { get; set; }
}
```
Вы можете использовать запрашиваемые расширения следующим образом:
```c#
var configuration = new MapperConfiguration(cfg =>
    cfg.CreateProjection<OrderLine, OrderLineDTO>()
    .ForMember(dto => dto.Item, conf => conf.MapFrom(ol => ol.Item.Name)));

public List<OrderLineDTO> GetLinesForOrder(int orderId)
{
  using (var context = new orderEntities())
  {
    return context.OrderLines.Where(ol => ol.OrderId == orderId)
             .ProjectTo<OrderLineDTO>(configuration).ToList();
  }
}
```
`.ProjectTo<OrderLineDTO>()`() сообщит механизму сопоставления `AutoMapper` выдать предложение выбора в `IQueryable`, которое сообщит платформе сущности, что ему нужно только запросить столбец `Name` таблицы `Item`, так же, как если бы вы вручную проецировали свой `IQueryable` в `OrderLineDTO` с помощью `Select`.


## Ограничения поставщика запросов (Query Provider Limitations)
`ProjectTo` должен быть последним вызовом в цепочке методов `LINQ`. ORM работают с сущностями, а не с DTO. Примените любую фильтрацию и сортировку к сущностям и, в качестве последнего шага, проецируйте их на DTO. Поставщики запросов очень сложны, и выполнение вызова `ProjectTo` последним гарантирует, что поставщик запросов будет работать настолько точно, насколько это было задумано, для построения действительных запросов к базовой цели запроса (SQL, Mongo QL и т.д.).

Обратите внимание: чтобы эта функция работала, все преобразования типов должны быть явно обработаны в вашем сопоставлении. Например, вы не можете полагаться на переопределение `ToString()` класса `Item`, чтобы сообщить платформе сущностей о выборе только из столбца `Name`, и любые изменения типа данных, такие как `Double` на `Decimal`, также должны быть явно обработаны.


## API экземпляра (The instance API)

Начиная с версии 8.0 в IMapper есть аналогичные методы ProjectTo, которые кажутся более естественными при использовании IMapper с DI.


## Предотвращение проблем с ленивой загрузкой/SELECT N+1 (Preventing lazy loading/SELECT N+1 problems)
Поскольку проекция LINQ, созданная `AutoMapper`, преобразуется непосредственно в запрос SQL поставщиком запросов, сопоставление происходит на уровне SQL/ADO.NET, не затрагивая ваши сущности. Все данные оперативно извлекаются и загружаются в ваши DTO.

Вложенные коллекции используют `Select` для проецирования дочерних DTO:
```c#
from i in db.Instructors
orderby i.LastName
select new InstructorIndexData.InstructorModel
{
    ID = i.ID,
    FirstMidName = i.FirstMidName,
    LastName = i.LastName,
    HireDate = i.HireDate,
    OfficeAssignmentLocation = i.OfficeAssignment.Location,
    Courses = i.Courses.Select(c => new InstructorIndexData.InstructorCourseModel
    {
        CourseID = c.CourseID,
        CourseTitle = c.Title
    }).ToList()
};
```
Это сопоставление через `AutoMapper` приведет к проблеме SELECT N+1, поскольку каждый дочерний курс будет запрашиваться по одному, если в вашем ORM не указано, что требуется немедленная выборка. При проецировании LINQ для вашего ORM не требуется никакой специальной настройки или спецификации. ORM использует проекцию LINQ для построения именно того SQL-запроса, который необходим.

Это означает, что вам не нужно использовать явную активную загрузку (`Include`) с `ProjectTo`. Если вам нужно что-то вроде фильтрованного `Include`, добавьте фильтр на свою карту:

`` CreateProjection<Entity, Dto>().ForMember(d => d.Collection, o => o.MapFrom(s => s.Collection.Where(i => ...));``


## Пользовательская проекция (Custom projection)
В случае, если имена членов не совпадают или вы хотите создать вычисляемое свойство, вы можете использовать `MapFrom` (перегрузку на основе выражений), чтобы предоставить собственное выражение для целевого элемента:
```c#
var configuration = new MapperConfiguration(cfg => cfg.CreateProjection<Customer, CustomerDto>()
    .ForMember(d => d.FullName, opt => opt.MapFrom(c => c.FirstName + " " + c.LastName))
    .ForMember(d => d.TotalContacts, opt => opt.MapFrom(c => c.Contacts.Count()));
```
`AutoMapper` передает предоставленное выражение вместе с построенной проекцией. Пока ваш поставщик запросов может интерпретировать предоставленное выражение, все будет передаваться в базу данных.

Если выражение отклонено вашим поставщиком запросов (Entity Framework, NHibernate и т. д.), вам может потребоваться настроить выражение, пока не найдете то, которое будет принято.

## Пользовательское преобразование типов (Custom Type Conversion)
Иногда вам необходимо полностью заменить преобразование типа исходного типа в целевой. В обычном сопоставлении во время выполнения это достигается с помощью метода `ConvertUsing`. Чтобы выполнить аналог в проекции LINQ, используйте метод `ConvertUsing`:

`` cfg.CreateProjection<Source, Dest>().ConvertUsing(src => new Dest { Value = 10 }); ``

`ConvertUsing` на основе выражений немного более ограничен, чем перегрузки `ConvertUsing` на основе Func, поскольку будет работать только то, что разрешено в выражении, и базовый поставщик LINQ.


## Конструкторы пользовательских типов назначения (Custom destination type constructors)
Если у вашего целевого типа есть собственный конструктор, но вы не хотите переопределять все сопоставление, используйте перегрузку метода на основе выражения ConstructUsing:
```c#
cfg.CreateProjection<Source, Dest>()
    .ConstructUsing(src => new Dest(src.Value + 10));
```
`AutoMapper` автоматически сопоставляет параметры конструктора назначения с исходными элементами на основе совпадающих имен, поэтому используйте этот метод только в том случае, если `AutoMapper` не может правильно сопоставить конструктор назначения или если вам требуется дополнительная настройка во время построения.


## Преобразование строк (String conversion)
`AutoMapper` автоматически добавит `ToString()`, если тип целевого элемента является строкой, а тип исходного элемента — нет.
```c#
public class Order {
    public OrderTypeEnum OrderType { get; set; }
}
public class OrderDto {
    public string OrderType { get; set; }
}
var orders = dbContext.Orders.ProjectTo<OrderDto>(configuration).ToList();
orders[0].OrderType.ShouldEqual("Online");
```


## Явное расширение (Explicit expansion)
В некоторых сценариях, таких как OData, общий DTO возвращается через действие контроллера `IQueryable`. Без явных инструкций AutoMapper расширит все элементы результата. Чтобы контролировать, какие элементы расширяются во время проецирования, установите `ExplicitExpansion` в конфигурации, а затем передайте элементы, которые вы хотите явно расширить:
```c#
dbContext.Orders.ProjectTo<OrderDto>(configuration,
    dest => dest.Customer,
    dest => dest.LineItems);
// or string-based
dbContext.Orders.ProjectTo<OrderDto>(configuration,
    null,
    "Customer",
    "LineItems");
// for collections
dbContext.Orders.ProjectTo<OrderDto>(configuration,
    null,
    dest => dest.LineItems.Select(item => item.Product));
```
Более подробную информацию [смотрите в тестах](https://github.com/AutoMapper/AutoMapper/search?p=1&amp;q=ExplicitExpansion&amp;utf8=%E2%9C%93).    


## Агрегация (Aggregations)
LINQ может поддерживать агрегатные запросы, а `AutoMapper` поддерживает методы расширения LINQ. В примере пользовательской проекции, если мы переименовали свойство `TotalContacts` в `ContactsCount`, `AutoMapper` будет соответствовать методу расширения `Count()`, а поставщик LINQ преобразует счетчик в коррелированный подзапрос для агрегирования дочерних записей.

`AutoMapper` также может поддерживать сложные агрегаты и вложенные ограничения, если это поддерживает поставщик LINQ:
```c#
cfg.CreateProjection<Course, CourseModel>()
    .ForMember(m => m.EnrollmentsStartingWithA,
          opt => opt.MapFrom(c => c.Enrollments.Where(e => e.Student.LastName.StartsWith("A")).Count()));
```
Этот запрос возвращает общее количество студентов для каждого курса, чья фамилия начинается с буквы «А».


## Параметризация (Parameterization)
Иногда для своих значений прогнозам требуются параметры времени выполнения. Рассмотрим проекцию, которая должна получить текущее имя пользователя как часть своих данных. Вместо использования кода пост-мэппинга мы можем параметризовать нашу конфигурацию MapFrom:
```c#
string currentUserName = null;
cfg.CreateProjection<Course, CourseModel>()
    .ForMember(m => m.CurrentUserName, opt => opt.MapFrom(src => currentUserName));
```
Когда мы проектируем, мы заменим наш параметр во время выполнения:
```c#
dbContext.Courses.ProjectTo<CourseModel>(Config, new { currentUserName = Request.User.Name });
```
Это работает путем захвата имени  поля замыкания в исходном выражении, а затем использования анонимного объекта/словаря для применения значения к значению параметра перед отправкой запроса поставщику запросов.

Вы также можете использовать словарь для построения значений проекции:
```c#
dbContext.Courses.ProjectTo<CourseModel>(Config, new Dictionary<string, object> { {"currentUserName", Request.User.Name} });
```
Однако использование словаря приведет к тому, что в запросе будут жестко закодированные значения вместо параметризованного запроса, поэтому используйте его с осторожностью.


## Рекурсивные модели (Recursive models)
В идеале вам следует избегать моделей, которые ссылаются сами на себя (проведите небольшое исследование). Но если необходимо, вам нужно включить их:
```c#
configuration.Internal().RecursiveQueriesMaxDepth = someRandomNumber;
```


## Поддерживаемые параметры сопоставления (Supported mapping options)
Не все параметры сопоставления поддерживаются, поскольку созданное выражение должно интерпретироваться поставщиком LINQ. AutoMapper поддерживает только то, что поддерживается поставщиками LINQ:
-    MapFrom (Expression-based)
-    ConvertUsing (Expression-based)
-    Ignore
-    NullSubstitute
-    Value transformers
-    IncludeMembers
-    Runtime polymorphic mapping with Include/IncludeBase

Не поддерживается:
-    Condition
-    SetMappingOrder
-    UseDestinationValue
-    MapFrom (Func-based)
-    Before/AfterMap
-    Custom resolvers
-    Custom type converters
-    ForPath
-    Value converters
-    Any calculated property on your domain object

## Перевод выражений (UseAsDataSource) (Expression Translation)
Automapper поддерживает перевод выражений из одного объекта в другой в отдельном пакете. Это делается путем замены свойств исходного класса на то, чему они соответствуют в целевом классе.

Учитывая примеры классов:
```c#
public class OrderLine
{
  public int Id { get; set; }
  public int OrderId { get; set; }
  public Item Item { get; set; }
  public decimal Quantity { get; set; }
}

public class Item
{
  public int Id { get; set; }
  public string Name { get; set; }
}

public class OrderLineDTO
{
  public int Id { get; set; }
  public int OrderId { get; set; }
  public string Item { get; set; }
  public decimal Quantity { get; set; }
}

var configuration = new MapperConfiguration(cfg =>
{
  cfg.AddExpressionMapping();
  
  cfg.CreateMap<OrderLine, OrderLineDTO>()
    .ForMember(dto => dto.Item, conf => conf.MapFrom(ol => ol.Item.Name));
  cfg.CreateMap<OrderLineDTO, OrderLine>()
    .ForMember(ol => ol.Item, conf => conf.MapFrom(dto => dto));
  cfg.CreateMap<OrderLineDTO, Item>()
    .ForMember(i => i.Name, conf => conf.MapFrom(dto => dto.Item));
});
```
При сопоставлении из выражения DTO
```c#
Expression<Func<OrderLineDTO, bool>> dtoExpression = dto=> dto.Item.StartsWith("A");
var expression = mapper.Map<Expression<Func<OrderLine, bool>>>(dtoExpression);
```
Выражение будет переведено в ``ol => ol.Item.Name.StartsWith("A")``
Automapper знает, что `dto.Item` сопоставлен с `ol.Item.Name`, поэтому заменил его на выражение.

Перевод выражений также может работать с выражениями коллекций.

```c#
Expression<Func<IQueryable<OrderLineDTO>,IQueryable<OrderLineDTO>>> dtoExpression = dtos => dtos.Where(dto => dto.Quantity > 5).OrderBy(dto => dto.Quantity);
var expression = mapper.Map<Expression<Func<IQueryable<OrderLine>,IQueryable<OrderLine>>>(dtoExpression);
```
В результате чего ``ols => ols.Where(ol => ol.Quantity > 5).OrderBy(ol => ol.Quantity)``


## Сопоставление сведенных свойств со свойствами навигации (Mapping Flattened Properties to Navigation Properties)
AutoMapper также поддерживает сопоставление сглаженных свойств (TMode или DTO) в выражениях с соответствующими свойствами навигации (TData) (когда свойство навигации было удалено из модели представления или DTO), например CourseModel.DepartmentName из выражения модели становится Course.Department в выражении данных.

Возьмите следующий набор:
```c#
public class CourseModel
{
    public int CourseID { get; set; }

    public int DepartmentID { get; set; }
    public string DepartmentName { get; set; }
}
public class Course
{
    public int CourseID { get; set; }

    public int DepartmentID { get; set; }
    public Department Department { get; set; }
}

public class Department
{
    public int DepartmentID { get; set; }
    public string Name { get; set; }
}
```
Затем сопоставьте exp ниже с expMapped.
```c#
Expression<Func<IQueryable<CourseModel>, IIncludableQueryable<CourseModel, object>>> exp = i => i.Include(s => s.DepartmentName);
Expression<Func<IQueryable<Course>, IIncludableQueryable<Course, object>>> expMapped = mapper.MapExpressionAsInclude<Expression<Func<IQueryable<Course>, IIncludableQueryable<Course, object>>>>(exp);
```
Результирующее сопоставленное выражение (expMapped.ToString()) имеет вид `i => i.Include(s => s.Department); `. Эта функция позволяет определять свойства навигации для запроса только на основе модели представления.


## Поддерживаемые параметры сопоставления (Supported Mapping options)
Подобно тому, как расширения Queryable Extensions могут поддерживать только определенные вещи, которые поддерживают поставщики LINQ, перевод выражений следует тем же правилам, что и то, что он может и не может поддерживать.

## UseAsDataSource
Сопоставление выражений друг с другом является утомительным занятием и приводит к созданию длинного и некрасивого кода.

`UseAsDataSource().For<DTO>()` делает этот перевод чистым, поскольку не требуется явно отображать выражения. Он также вызывает для вас `ProjectTo<TDO>()`, где это применимо.

Использование EntityFramework в качестве примера
```c#
dataContext.OrderLines.UseAsDataSource().For<OrderLineDTO>().Where(dto => dto.Name.StartsWith("A"))
```
Имеет эквивалент
```c#
dataContext.OrderLines.Where(ol => ol.Item.Name.StartsWith("A")).ProjectTo<OrderLineDTO>()
```


## Когда ProjectTo() не вызывается (When ProjectTo() is not called)
Перевод выражений работает для всех видов функций, включая вызовы `Select`. Если `Select` используется после `UseAsDataSource()` и меняет тип возвращаемого значения, то `ProjectTo()` не будет вызываться, а вместо него будет использоваться `Mapper.Map`.

Пример:

`dataContext.OrderLines.UseAsDataSource().For<OrderLineDTO>().Select(dto => dto.Name)`

Имеет эквивалент

`dataContext.OrderLines.Select(ol => ol.Item.Name)`

## Зарегистрируйте обратный вызов при перечислении запроса UseAsDataSource(). (Register a callback, for when an UseAsDataSource() query is enumerated)
Иногда вам может потребоваться отредактировать коллекцию, возвращаемую сопоставленным запросом, прежде чем пересылать ее на следующий уровень приложения. С `.ProjectTo<TDto>` это довольно просто, поскольку нет смысла напрямую возвращать полученный `IQueryable<TDto>`, поскольку редактировать его все равно уже нельзя. Итак, вы, скорее всего, сделаете это:
```c#
var configuration = new MapperConfiguration(cfg =>
    cfg.CreateMap<OrderLine, OrderLineDTO>()
    .ForMember(dto => dto.Item, conf => conf.MapFrom(ol => ol.Item.Name)));

public List<OrderLineDTO> GetLinesForOrder(int orderId)
{
  using (var context = new orderEntities())
  {
    var dtos = context.OrderLines.Where(ol => ol.OrderId == orderId)
             .ProjectTo<OrderLineDTO>().ToList();
    foreach(var dto in dtos)
    {
        // редактируем какое-либо свойство или загружаем дополнительные данные из базы данных и дополняем dtos
    }
    return dtos;
  }
}
```
Однако если вы сделаете это с помощью подхода `.UseAsDataSource()`, вы потеряете всю его мощь, а именно возможность изменять внутреннее выражение до тех пор, пока оно не будет перечислено. Чтобы решить эту проблему, мы ввели обратный вызов `.OnEnumerated`. Используя его, вы можете сделать следующее:
```c#
var configuration = new MapperConfiguration(cfg =>
    cfg.CreateMap<OrderLine, OrderLineDTO>()
    .ForMember(dto => dto.Item, conf => conf.MapFrom(ol => ol.Item.Name)));

public IQueryable<OrderLineDTO> GetLinesForOrder(int orderId)
{
  using (var context = new orderEntities())
  {
    return context.OrderLines.Where(ol => ol.OrderId == orderId)
             .UseAsDataSource()
             .For<OrderLineDTO>()
             .OnEnumerated((dtos) =>
             {
                foreach(var dto in dtosCast<OrderLineDTO>())
                {
                     // отредактируйте какое-либо свойство или загрузите дополнительные данные из базы данных и дополните dtos
                }
             }
   }
}
```
этот обратный вызов `OnEnumerated(IEnumerable)` выполняется при перечислении самого `IQueryable<OrderLineDTO>`. Таким образом, это также работает с примерами OData, упомянутыми выше: выражения OData $filter и $orderby по-прежнему преобразуются в SQL, а обратный вызов `OnEnumerated()` предоставляется с отфильтрованным упорядоченным набором результатов из базы данных.


## (AutoMapper.Extensions.EnumMapping)
Встроенный преобразователь перечислений не настраивается, его можно только заменить. В качестве альтернативы AutoMapper поддерживает сопоставление значений перечисления на основе соглашений в отдельном пакете [AutoMapper.Extensions.EnumMapping](https://www.nuget.org/packages/AutoMapper.Extensions.EnumMapping/).

Для метода `CreateMap` эта библиотека предоставляет метод `ConvertUsingEnumMapping`. Этот метод добавляет все сопоставления по умолчанию от источника к значениям перечисления назначения.

Если вы хотите изменить некоторые сопоставления, вы можете использовать метод `MapValue`. Это цепной метод.

По умолчанию значения перечисления сопоставляются по значению (явно: `MapByValue()`), но можно сопоставлять по имени, вызывая `MapByName()`.
```c#
using AutoMapper.Extensions.EnumMapping;

public enum Source
{
    Default = 0,
    First = 1,
    Second = 2
}

public enum Destination
{
    Default = 0,
    Second = 2
}

internal class YourProfile : Profile
{
    public YourProfile()
    {
        CreateMap<Source, Destination>()
            .ConvertUsingEnumMapping(opt => opt
		        // optional: .MapByValue() or MapByName(), without configuration MapByValue is used
		        .MapValue(Source.First, Destination.Default)
            )
            .ReverseMap(); // to support Destination to Source mapping, including custom mappings of ConvertUsingEnumMapping
    }
}
    ...
```

## Соглашение по умолчанию (Default Convention)
Пакет `AutoMapper.Extensions.EnumMapping` сопоставит все значения из типа источника в тип назначения, если оба типа перечисления имеют одинаковое значение (или по имени, или по значению). Все значения перечисления Source, которые не имеют эквивалента Target, вызовут исключение, если включена `EnumMappingValidation`.


## Соглашение ReverseMap (ReverseMap Convention)
Для метода `ReverseMap` используется то же соглашение, что и для сопоставлений по умолчанию, но оно также учитывает переопределение сопоставлений значений перечисления, если это возможно.

Следующие шаги определяют обратные переопределения:

1. Создайте сопоставления Source и Destination (соглашение по умолчанию), включая пользовательские переопределения.

2. Создание сопоставлений для Destination и Source (соглашение по умолчанию) без пользовательских переопределений (должно быть определено)

3. Сопоставления из шага 1 будут использоваться для определения переопределений для ReverseMap. Поэтому сопоставления группируются по значению назначения.

    3a) если значение Source соответствует значению Destination, то это сопоставление является предпочтительным и переопределение не требуется.

Возможно, что значение Destination имеет несколько значений Source, заданных сопоставлениями переопределения.

Мы должны определить, какое значение источника будет новым назначением для текущего значения назначения (которое является новым значением источника).

Для каждого значения источника на сгруппированное значение назначения:

    3b) если значение перечисления Source не существует в типе перечисления Destination, то это сопоставление не может быть отменено.

    3c) если существует значение Source, которое не является частью Destination сопоставлений из шага 1, то это сопоставление не может быть отменено.

    3d) если значение Source не исключено опциями b и c, то это значение Source является новым значением Destination.

4. Все переопределения, определенные на шаге 3, будут применены к сопоставлениям, полученным на шаге 2.

5. Наконец, будут применены пользовательские сопоставления, предоставленные методу ReverseMap.

## Тестирование (Testing)
AutoMapper предоставляет хороший инструмент для проверки карт типов. Этот пакет добавляет дополнительный метод расширения `EnumMapperConfigurationExpressionExtensions.EnableEnumMappingValidation`, который расширяет существующий метод `AssertConfigurationIsValid()` и позволяет также проверять сопоставления перечислений.

Чтобы включить тестирование конфигурации сопоставления перечислений:
```c#
public class MappingConfigurationsTests
{
    [Fact]
    public void WhenProfilesAreConfigured_ItShouldNotThrowException()
    {
        // Arrange
        var config = new MapperConfiguration(configuration =>
        {
            configuration.EnableEnumMappingValidation();

            configuration.AddMaps(typeof(AssemblyInfo).GetTypeInfo().Assembly);
        });
		
        // Assert
        config.AssertConfigurationIsValid();
    }
}
```

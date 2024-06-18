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



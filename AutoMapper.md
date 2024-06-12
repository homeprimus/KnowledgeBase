# [Automapper](https://docs.automapper.org/en/latest/)

Сопоставитель объектов на основе соглашений.

AutoMapper использует гибкий API конфигурации для определения стратегии сопоставления объектов. AutoMapper использует основанный на соглашениях алгоритм сопоставления для сопоставления значений источника и назначения. AutoMapper ориентирован на сценарии проецирования моделей для сведения сложных объектных моделей к DTO и другим простым объектам, конструкция которых лучше подходит для сериализации, связи, обмена сообщениями или просто для защиты от коррупции уровня между уровнем домена и приложения.

```c#
var config = new MapperConfiguration(cfg => cfg.CreateMap<Order, OrderDto>());
...
config.AssertConfigurationIsValid();
...
var mapper = config.CreateMapper();
// or
var mapper = new Mapper(config);

OrderDto dto = mapper.Map<OrderDto>(order);
```

```c#
var config = new MapperConfiguration(
    cfg => 
    {
        cfg.CreateMap<MapperOrderProfile>();
    });
...
config.AssertConfigurationIsValid();

...

class MapperOrderProfile: Profile
{
    public MapperOrderProfile()
    {
        CreateMap<Order, OrderDto>();
    }
}
```
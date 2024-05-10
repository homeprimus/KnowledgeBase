# [OpenTelemetry](https://opentelemetry.io/ "Кроссплатформенный стандарт для сбора и создания данных телеметрии")

## [Александр Пугач "Метрики в .NET на примере OpenTelemetry и Prometheus"](https://www.youtube.com/watch?v=yAja9Q2KyDM&ab_channel=DotNetRu)

#### Метрики - количечтвенные данные, которые описывают производительность приложения или состояние системы
- RPS (request per seconts)
- Latency (задержка)
- Troughput (пропускная способность)
- Error rate (частота ошибок)
- CPU usage (имспользование ЦПУ)
- Memory usage (использование памяти)

#### Необходимиость мониторинга приложений
- Выявление и устранение деградаций
- Отслеживание и оптимизация производительности
- Выявленеи анвмалий и повышение надежности и безопасности

### .NET Metrics APIs
- PerfomanceCounter
- EventCounter
- System.Disgnostics.Metrics
- Third-party APIs (AppMetrics, Prometheus)


### Инструменты
- Синхронные
    Counter
    Histogram
    UpDownCounter
- Асинхронные
    ObservateCounter
    ObservateGauge
    ObserveteUpDownCounter

## Пример
### Пример использования в Console Application
#### Добавить в файл проекта
```C#
  <ItemGroup>
    <PackageReference Include="OpenTelemetry.Api" Version="1.8.1" />
    <PackageReference Include="OpenTelemetry.Exporter.Console" Version="1.8.1" />
    <PackageReference Include="OpenTelemetry.Exporter.Prometheus.HttpListener" Version="1.8.0-rc.1" />
  </ItemGroup>
```
```C#
using System.Diagnostics.Metrics;
using OpenTelemetry;
using OpenTelemetry.Metrics;

var meter = new Meter("Posokhov","v1");

// Синхронный метод
var counter = meter.CreateCounter<long>(
    "counter.name",
    unit: "things",
    description: "Пробуем в консоли OpenTelemetry");

// Асмнхронный метод
var obcerv = meter.CreateObservableGauge(
    name: "create.oservable.gauge",
    () =>
    { 
        return new Measurement<long>(
            Random.Shared.Next(),
            new KeyValuePair<string, object?>("tag2", "value2")
        );
    },
    unit: "gauge.unit",
    description: "Random.Shared.Next()"
    );

// Гистограмма
var histrogram = meter.CreateHistogram<long>(
    name: "create.histogram",
    unit: "histogram.unit",
    description: "Создали гистограмму"
);

using var provider = Sdk.CreateMeterProviderBuilder()
    .AddMeter(meter.Name)
    // вывод в консоль
    .AddConsoleExporter()
    // вывод в промитеус
    .AddPrometheusHttpListener(options =>
        {
            options.ScrapeEndpointPath = "/metrics";
            options.UriPrefixes = new string[] { "http://localhost:9464/" };
        }
    )
    .Build();

while (!Console.KeyAvailable)
{
    counter.Add(200, new KeyValuePair<string, object?>("tag1", "value1"));
    histrogram.Record(Random.Shared.Next(), new KeyValuePair<string, object?>("tag3", "value3"));
}

```
#### Пример использования в ASP .NET Application
```C#
  <ItemGroup>
    <PackageReference Include="OpenTelemetry.Exporter.Prometheus.AspNetCore" Version="1.8.0-rc.1" />
    <PackageReference Include="OpenTelemetry.Extensions.Hosting" Version="1.8.1" />
  </ItemGroup>
```
```C#
using System.Diagnostics.Metrics;
using OpenTelemetry.Metrics;

var builder = WebApplication.CreateBuilder(args);

var meter = new Meter("Asp .NET Meter", "v1");
builder.Services.AddOpenTelemetry()
    .WithMetrics(meterBuilder =>
    {
        meterBuilder.AddMeter(meter.Name)
            .AddPrometheusExporter();
    });

var app = builder.Build();

var counter = meter.CreateCounter<long>(
    name: "create.counter",
    unit: "counter.unit",
    description: "The number counter Hello world"
);
app.MapGet("/hello", async () =>
{
    counter.Add(1);
    await Task.Delay(300);
    return "Hello World";
});

app.UseOpenTelemetryPrometheusScrapingEndpoint("metrics");

app.MapGet("/", () => "Hello World!");

app.Run();

```

# Custom Instruments
- [opentelemetry-dotnet](https://github.com/open-telemetry/opentelemetry-dotnet/tree/main/src)
    + OpenTelemetry.Instrumentation.AspNetCore
    + OpenTelemetry.Instrumentation.GprcNetClient
    + OpenTelemetry.Instrumentation.Http
    + OpenTelemetry.Instrumentation.SqlClient
- [opentelemetry-dotnet-contrib](https://github.com/open-telemetry/opentelemetry-dotnet-contrib)
    + OpenTelemetry.Instrumentation.AspNet
    + OpenTelemetry.Instrumentation.Cassandra
    + OpenTelemetry.Instrumentation.Quartz
    + OpenTelemetry.Instrumentation.MassTransit
    + OpenTelemetry.Instrumentation.EntityFrameworkCore
    + ...
- DiagnosticSource and DiagnosticListener
- Third-party libraries
- custom instruments
    

# Альтернативы Opentelemetry в .NET
- [App.Metrics]()
- [Promitheus-net]()
- [Promitheus.Client]()

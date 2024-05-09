# [OpenTelemetry](https://opentelemetry.io/ "Кроссплатформенный стандарт для сбора и создания данных телеметрии")

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

    
# Паттерн Unit of Work

Паттерн Unit of Work - нередко используется паттерн репозиторий для инкапсулирования логики работы с источниками данных. И нередко мы оперируем множеством сущностей и моделей, для управления которыми создается также множество классов-репозиториев. Паттерн Unit of Work позволяет упростить работу с различными репозиториями и дает уверенность, что все репозитории будут использовать один и тот же контекст данных.
Позволяет решить две проблемы:
- Сложности обеспечения транзакционности базы данных

    При разработке сервисов, часто возникает неотъемлемая потребность в использовании транзакций базы данных для обеспечения целостности данных. Однако, при попытке интегрировать транзакционную логику в традиционные подходы, столкнулся с трудностями. Связывание транзакционной логики с логикой слоя базы данных оказалось нетривиальным и привело к нарушению принципов разделения ответственности. Это, в свою очередь, сказалось на тестировании и поддержке кода.
-  Нарушение изолированности слоя

    В попытке решить первую проблему, некоторые разработчики переносят работу с транзакциями на уровень слоя приложения, чтобы избежать прямой зависимости от базы данных. Однако, такой подход, несмотря на его обоснование, может нарушить изолированность слоев и противоречить принципам DDD и чистой архитектуры. Это, в конечном итоге, затрудняет поддержку приложения и усложняет его масштабирование.

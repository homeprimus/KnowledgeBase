# [Fluent Assertions ](https://fluentassertions.com/introduction)
Fluent Assertions — это набор методов расширения .NET, которые позволяют более естественно указать ожидаемый результат модульного теста в стиле TDD или BDD. Это обеспечивает простой интуитивный синтаксис, который начинается со следующего оператора using:
```c#
using FluentAssertions;
```
Это добавляет множество методов расширения в текущую область действия. Например, чтобы убедиться, что строка начинается, заканчивается и содержит определенную фразу.
```c#
tring actual = "ABCDEFGHI";
actual.Should().StartWith("AB").And.EndWith("HI").And.Contain("EF").And.HaveLength(9);
```
Чтобы проверить, что все элементы коллекции соответствуют предикату и содержат указанное количество элементов.
```c#
IEnumerable<int> numbers = new[] { 1, 2, 3 };

numbers.Should().OnlyContain(n => n > 0);
numbers.Should().HaveCount(4, "потому что мы думали, что поместили в коллекцию четыре предмета");
```
Во втором неудачном примере приятно то, что вместе с сообщением он выдаст исключение.

>«Ожидаемое число будет содержать 4 элемента(ов), потому что мы думали, что поместили в коллекцию четыре элемента, но нашли 3».

Чтобы убедиться, что определенное бизнес-правило применяется с помощью исключений.
```c#
var recipe = new RecipeBuilder()
                    .With(new IngredientBuilder().For("Milk").WithQuantity(200, Unit.Milliliters))
                    .Build();
Action action = () => recipe.AddIngredient("Milk", 100, Unit.Spoon);
    action
                    .Should().Throw<RuleViolationException>()
                    .WithMessage("*change the unit of an existing ingredient*")
                    .And.Violations.Should().Contain(BusinessRule.CannotChangeIngredientQuantity);
```
Одной из приятных особенностей является возможность связывать определенное утверждение поверх утверждения, которое действует на коллекцию или граф объектов.
```c#
dictionary.Should().ContainValue(myClass).Which.SomeProperty.Should().BeGreaterThan(0);
someObject.Should().BeOfType<Exception>().Which.Message.Should().Be("Other Message");
xDocument.Should().HaveElement("child").Which.Should().BeOfType<XElement>().And.HaveAttribute("attr", "1");
```
Такая цепочка может значительно облегчить чтение ваших модульных тестов.

## Обнаружение тестовых фреймворков
Fluent Assertions поддерживает множество различных платформ модульного тестирования. Просто добавьте ссылку на соответствующую сборку тестовой среды в проект модульного теста. Fluent Assertions автоматически найдет соответствующую сборку и будет использовать ее для создания исключений, специфичных для платформы.

Если по какой-то неизвестной причине Fluent Assertions не может найти сборку, а вы работаете под управлением .NET 4.7 или проекта .NET Core 3.0, попробуйте явно указать платформу, используя параметр конфигурации в `app.config` проекта. Если он не может найти ни одну из поддерживаемых платформ, он вернется к использованию специального класса исключений `AssertFailedException`.
```xml
<configuration>
  <appSettings>
    <!-- Supported values: nunit, xunit2, mstestv2, nspec3 and mspec -->
    <add key="FluentAssertions.TestFramework" value="nunit"/>
  </appSettings>
</configuration>
```
Просто добавьте пакет NuGet `FluentAssertions` в свой тестовый проект.

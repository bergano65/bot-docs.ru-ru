---
title: Модульное тестирование ботов | Документация Майкрософт
description: Из этой статьи вы узнаете, как выполнять модульное тестирование ботов с помощью платформ тестирования.
keywords: бот, тестирование ботов, платформа для тестирования ботов
author: gabog
ms.author: ggilaber
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7e85f7ebe5a39145314f10b3b0378c2c66dc361d
ms.sourcegitcommit: 23a1808e18176f1704f2f6f2763ace872b1388ae
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/25/2019
ms.locfileid: "68484401"
---
# <a name="how-to-unit-test-bots"></a>Как выполнять модульное тестирование ботов

[!INCLUDE[applies-to](../includes/applies-to.md)]

В этой статье описаны следующие задачи:

- создание модульных тестов для ботов;
- использование оператора assert для сопоставления действий, которые возвращает шаг диалога, с ожидаемыми значениями;
- использование оператора assert для проверки результатов, которые возвращает диалог;
- создание разных типов тестов, управляемых данными;
- создание макетов объектов для разных зависимостей диалогов (распознавателей LUIS и т. д.).

## <a name="prerequisites"></a>Предварительные требования

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Пример [CoreBot Tests](https://aka.ms/cs-core-test-sample) ссылается на пакет [Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/), [XUnit](https://xunit.net/) и [Moq](https://github.com/moq/moq) для создания модульных тестов.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Пример [CoreBot Tests](https://aka.ms/js-core-test-sample) ссылается на пакет [botbuilder-testing](https://www.npmjs.com/package/botbuilder-testing) и [Mocha](https://mochajs.org/) для создания модульных тестов, а также на [Mocha Test Explorer](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter) для визуализации результатов тестов в VS Code.

---

## <a name="testing-dialogs"></a>Тестирование диалогов

В примере CoreBot выполняется модульное тестирование диалогов с помощью класса `DialogTestClient` — средства изолированного тестирования за пределами бота, при использовании которого не нужно развертывать код в веб-службе.

С помощью этого класса можно создавать модульные тесты, которые проверяют ответы в диалогах на основе смены шагов. Модульные тесты с использованием класса `DialogTestClient` должны поддерживать другие диалоги, созданные с помощью библиотеки диалогов Bot Builder.

В следующем примере показаны тесты с использованием `DialogTestClient`:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut);

var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("Seattle");
Assert.Equal("Where are you traveling from?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("New York");
Assert.Equal("When would you like to travel?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("tomorrow");
Assert.Equal("OK, I will book a flight from Seattle to New York for tomorrow, Is this Correct?", reply.Text);

reply = await testClient.SendActivityAsync<IMessageActivity>("yes");
Assert.Equal("Sure thing, wait while I finalize your reservation...", reply.Text);

reply = testClient.GetNextReply<IMessageActivity>();
Assert.Equal("All set, I have booked your flight to Seattle for tomorrow", reply.Text);
```

Класс `DialogTestClient` определяется в пространстве имен `Microsoft.Bot.Builder.Testing`. Он включен в пакет NuGet [Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/).

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut);

let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');

reply = await testClient.sendActivity('Seattle');
assert.strictEqual(reply.text, 'Where are you traveling from?');

reply = await testClient.sendActivity('New York');
assert.strictEqual(reply.text, 'When would you like to travel?');

reply = await testClient.sendActivity('tomorrow');
assert.strictEqual(reply.text, 'OK, I will book a flight from Seattle to New York for tomorrow, Is this Correct?');

reply = await testClient.sendActivity('yes');
assert.strictEqual(reply.text, 'Sure thing, wait while I finalize your reservation...');

reply = testClient.getNextReply();
assert.strictEqual(reply.text, 'All set, I have booked your flight to Seattle for tomorrow');
```

Класс `DialogTestClient` включен в пакет npm [botbuilder-testing]().

---

### <a name="dialogtestclient"></a>DialogTestClient

Первый параметр `DialogTestClient` — это целевой канал. Он позволяет протестировать логику визуализации на основе целевого канала для бота (Teams, Slack, Кортана и т. д.). Если вы не уверены, какой целевой канал выбрать, вы можете использовать идентификаторы канала `Emulator` или `Test`. Но помните, что некоторые компоненты могут вести себя по-разному в зависимости от текущего канала, например `ConfirmPrompt` отображает варианты "Да/Нет" по-разному для каналов `Test` и `Emulator`. Этот параметр также можно использовать для проверки условной логики визуализации в диалоге на основе идентификатора канала.

Второй параметр — это экземпляр тестируемого диалога. (Примечание. Сокращение **sut**, используемое в фрагментах кода, означает "тестируемая система".)

Конструктор `DialogTestClient` предоставляет дополнительные параметры для дополнительной настройки поведения клиента или передачи параметров в тестируемый диалог, если это необходимо. Вы можете передать данные инициализации для диалога, добавить настраиваемое ПО промежуточного слоя, а также использовать TestAdapter и экземпляр `ConversationState`.

### <a name="sending-and-receiving-messages"></a>Отправка и получение сообщений

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Метод `SendActivityAsync<IActivity>` отправляет текстовый фрагмент или `IActivity` в диалог и возвращает первое полученное сообщение. Параметр `<T>` используется для возврата строго типизированного экземпляра ответа, чтобы вы могли утвердить его без приведения.

```csharp
var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);
```

В некоторых сценариях бот может отправить несколько сообщений в ответ на одно действие. В таких случаях `DialogTestClient` будет помещать ответы в очередь и использовать метод `GetNextReply<IActivity>` для извлечения следующего сообщения из очереди.

```csharp
reply = testClient.GetNextReply<IMessageActivity>();
Assert.Equal("All set, I have booked your flight to Seattle for tomorrow", reply.Text);
```

Если в очереди ответов отсутствуют сообщения, `GetNextReply<IActivity>` возвращает значение NULL.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Метод `sendActivity` отправляет текстовый фрагмент или `Activity` в диалог и возвращает первое полученное сообщение.

```javascript
let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');
```

В некоторых сценариях бот может отправить несколько сообщений в ответ на одно действие. В таких случаях `DialogTestClient` будет помещать ответы в очередь и использовать метод `getNextReply` для извлечения следующего сообщения из очереди.

```javascript
reply = testClient.getNextReply();
assert.strictEqual(reply.text, 'All set, I have booked your flight to Seattle for tomorrow');
```

Если в очереди ответов отсутствуют сообщения, `getNextReply` возвращает значение NULL.

---

### <a name="asserting-activities"></a>Утверждение действий

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Код в примере CoreBot утверждает только свойство `Text` полученных действий. Для более сложных ботов может потребоваться утверждать другие свойства, например `Speak`, `InputHint`, `ChannelData` и т. д.

```csharp
Assert.Equal("Sure thing, wait while I finalize your reservation...", reply.Text);
Assert.Equal("One moment please...", reply.Speak);
Assert.Equal(InputHints.IgnoringInput, reply.InputHint);
```

Это можно сделать, проверив каждое свойство, как показано выше. Вы можете написать собственные вспомогательные служебные программы для утверждения действий или использовать другие платформы, например [FluentAssertions](https://fluentassertions.com/), для написания пользовательских утверждений и упрощения кода теста.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Код в примере CoreBot утверждает только свойство `text` полученных действий. Для более сложных ботов может потребоваться утверждать другие свойства, например `speak`, `inputHint`, `channelData` и т. д.

```javascript
assert.strictEqual(reply.text, 'Sure thing, wait while I finalize your reservation...');
assert.strictEqual(reply.speak, 'One moment please...');
assert.strictEqual(reply.inputHint, InputHints.IgnoringInput);
```

Это можно сделать, проверив каждое свойство, как показано выше. Вы можете написать собственные вспомогательные служебные программы для утверждения действий или использовать другие библиотеки, например [Chai](https://www.chaijs.com/), для написания пользовательских утверждений и упрощения кода теста.

---

### <a name="passing-parameters-to-your-dialogs"></a>Передача параметров в диалоги

Конструктор `DialogTestClient` использует `initialDialogOptions` для передачи параметров в диалог. Например, `MainDialog` в этом примере инициализирует объект `BookingDetails` из результатов LUIS с сущностями, которые он разрешает из речевого фрагмента пользователя, а затем передает этот объект для вызова `BookingDialog`.

Это можно реализовать в тесте следующим образом:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var inputDialogParams = new BookingDetails()
{
    Destination = "Seattle",
    TravelDate = $"{DateTime.UtcNow.AddDays(1):yyyy-MM-dd}"
};

var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut, inputDialogParams);

```

`BookingDialog` получает этот параметр и обращается к нему в тесте так же, как при вызове из `MainDialog`.

```csharp
private async Task<DialogTurnResult> DestinationStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    var bookingDetails = (BookingDetails)stepContext.Options;
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const inputDialogParams = {
    destination: 'Seattle',
    travelDate: formatDate(new Date().setDate(now.getDate() + 1))
};

const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut, inputDialogParams);
```

`BookingDialog` получает этот параметр и обращается к нему в тесте так же, как при вызове из `MainDialog`.

```javascript
async destinationStep(stepContext) {
    const bookingDetails = stepContext.options;
    ...
}
```

---

### <a name="asserting-dialog-turn-results"></a>Результаты утверждения шага диалога

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Некоторые диалоги, например `BookingDialog` или `DateResolverDialog`, возвращают значение в вызывающий диалог. Объект `DialogTestClient` предоставляет свойство `DialogTurnResult` для анализа и утверждения результатов, возвращаемых диалогом.

Например:

```csharp
var sut = new BookingDialog();
var testClient = new DialogTestClient(Channels.Msteams, sut);

var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
Assert.Equal("Where would you like to travel to?", reply.Text);

...

var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
Assert.Equal("New York", bookingResults?.Origin);
Assert.Equal("Seattle", bookingResults?.Destination);
Assert.Equal("2019-06-21", bookingResults?.TravelDate);
```

Свойство `DialogTurnResult` также можно использовать для проверки и утверждения промежуточных результатов, возвращаемых шагами в каскаде.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Некоторые диалоги, например `BookingDialog` или `DateResolverDialog`, возвращают значение в вызывающий диалог. Объект `DialogTestClient` предоставляет свойство `dialogTurnResult` для анализа и утверждения результатов, возвращаемых диалогом.

Например:

```javascript
const sut = new BookingDialog();
const testClient = new DialogTestClient('msteams', sut);

let reply = await testClient.sendActivity('hi');
assert.strictEqual(reply.text, 'Where would you like to travel to?');

...

const bookingResults = client.dialogTurnResult.result;
assert.strictEqual('New York', bookingResults.destination);
assert.strictEqual('Seattle', bookingResults.origin);
assert.strictEqual('2019-06-21', bookingResults.travelDate);
```

Свойство `dialogTurnResult` также можно использовать для проверки и утверждения промежуточных результатов, возвращаемых шагами в каскаде.

---

### <a name="analyzing-test-output"></a>Анализ результатов теста

Иногда бывает нужно прочитать расшифровку модульного теста, чтобы проанализировать выполнение теста без его отладки.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Пакет [Microsoft.Bot.Builder.Testing](https://www.nuget.org/packages/Microsoft.Bot.Builder.Testing/) включает `XUnitDialogTestLogger` для записи в консоль сообщений, отправляемых и получаемых диалогом.

Чтобы использовать это ПО промежуточного слоя, тест должен использовать конструктор, который получает объект `ITestOutputHelper`, предоставляемый средством запуска тестов XUnit, и создать `XUnitDialogTestLogger` для передачи в `DialogTestClient` с использованием параметра `middlewares`.

```csharp
public class BookingDialogTests
{
    private readonly IMiddleware[] _middlewares;

    public BookingDialogTests(ITestOutputHelper output)
        : base(output)
    {
        _middlewares = new[] { new XUnitDialogTestLogger(output) };
    }

    [Fact]
    public async Task SomeBookingDialogTest()
    {
        // Arrange
        var sut = new BookingDialog();
        var testClient = new DialogTestClient(Channels.Msteams, sut, middlewares: _middlewares);

        ...
    }
}
```

Ниже показано, как после настройки `XUnitDialogTestLogger` выводит записанные данные:

![XUnitMiddlewareOutput](media/how-to-unit-test/cs/XUnitMiddlewareOutput.png)

Подробные сведения о записи и отправке выходных данных теста в консоль при использовании xUnit см. [в документации по xUnit](https://xunit.net/docs/capturing-output.html).

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Пакет [botbuilder-testing](https://www.npmjs.com/package/botbuilder-testing) включает `DialogTestLogger` для записи в консоль сообщений, отправляемых и получаемых диалогом.

Чтобы использовать это ПО промежуточного слоя, просто передайте его в `DialogTestClient` через параметр `middlewares`.

```javascript
const client = new DialogTestClient('msteams', sut, testData.initialData, [new DialogTestLogger()]);
```

Ниже показано, как после настройки `DialogTestLogger` выводит записанные данные:

![DialogTestLoggerOutput](media/how-to-unit-test/js/DialogTestLoggerOutput.png)

---

Эти выходные данные также регистрируются на сервере сборки во время сборок с непрерывной интеграцией. Они помогают анализировать сбои сборки.

## <a name="data-driven-tests"></a>Тесты, управляемые данными

В большинстве случаев логика диалога не меняется, и разные пути выполнения в диалоге основаны на речевых фрагментах пользователя. Чтобы не писать модульные тесты для каждого варианта в диалоге, проще использовать управляемые данными (параметризованные) тесты.

Так, пример теста в разделе с общими сведениями показывает, как проверить один поток выполнения. Но если пользователь не выполнит подтверждение, используется другая дата или существуют другие условия?

Управляемые данными тесты позволяют протестировать все эти изменения без необходимости переписывать тесты.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

В примере CoreBot мы используем тесты `Theory` из XUnit для параметризации тестов.

### <a name="theory-tests-using-inlinedata"></a>Тесты с использованием InlineData

Следующий тест проверяет, что диалог отменяется, когда пользователь вводит cancel (Отмена).

```csharp
[Fact]
public async Task ShouldBeAbleToCancel()
{
    var sut = new TestCancelAndHelpDialog();
    var testClient = new DialogTestClient(Channels.Test, sut);

    var reply = await testClient.SendActivityAsync<IMessageActivity>("Hi");
    Assert.Equal("Hi there", reply.Text);
    Assert.Equal(DialogTurnStatus.Waiting, testClient.DialogTurnResult.Status);

    reply = await testClient.SendActivityAsync<IMessageActivity>("cancel");
    Assert.Equal("Cancelling...", reply.Text);
}
```

Но для отмены диалога пользователь может ввести quit (Выйти), never mind (Не важно) или stop it (Остановить). Чтобы не писать новый тестовый случай для каждого возможного слова, создайте один метод теста `Theory`, который принимает параметры через список значений `InlineData` и определяет параметры для каждого тестового случая:

```csharp
[Theory]
[InlineData("cancel")]
[InlineData("quit")]
[InlineData("never mind")]
[InlineData("stop it")]
public async Task ShouldBeAbleToCancel(string cancelUtterance)
{
    var sut = new TestCancelAndHelpDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, middlewares: _middlewares);

    var reply = await testClient.SendActivityAsync<IMessageActivity>("Hi");
    Assert.Equal("Hi there", reply.Text);
    Assert.Equal(DialogTurnStatus.Waiting, testClient.DialogTurnResult.Status);

    reply = await testClient.SendActivityAsync<IMessageActivity>(cancelUtterance);
    Assert.Equal("Cancelling...", reply.Text);
}
```

Новый тест будет выполнен четыре раза с разными параметрами, и каждый вариант будет отображен как дочерний элемент в наборе тестов `ShouldBeAbleToCancel` в обозревателе тестов Visual Studio. Если один из тестов завершится с ошибкой, как показано ниже, не нужно перезапускать весь набор тестов. Просто щелкните правой кнопкой мыши и выполните отладку скрипта, который завершился сбоем.

![InlineDataTestResults](media/how-to-unit-test/cs/InlineDataTestResults.png)

### <a name="theory-tests-using-memberdata-and-complex-types"></a>Тестирование с использованием MemberData и сложных типов

`InlineData` удобно использовать для небольших тестов, управляемых данными, которые получают параметры со значениями простых типов (строка, целое число и т. д.).

`BookingDialog` получает объект `BookingDetails` и возвращает новый объект `BookingDetails`. Непараметризованная версия теста для этого диалогового окна будет выглядеть следующим образом:

```csharp
[Fact]
public async Task DialogFlow()
{
    // Initial parameters
    var initialBookingDetails = new BookingDetails
    {
        Origin = "Seattle",
        Destination = null,
        TravelDate = null,
    };

    // Expected booking details
    var expectedBookingDetails = new BookingDetails
    {
        Origin = "Seattle",
        Destination = "New York",
        TravelDate = "2019-06-25",
    };

    var sut = new BookingDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, initialBookingDetails);

    // Act/Assert
    var reply = await testClient.SendActivityAsync<IMessageActivity>("hi");
    ...

    var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
    Assert.Equal(expectedBookingDetails.Origin, bookingResults?.Origin);
    Assert.Equal(expectedBookingDetails.Destination, bookingResults?.Destination);
    Assert.Equal(expectedBookingDetails.TravelDate, bookingResults?.TravelDate);
}
```

Чтобы параметризировать этот тест, мы создали класс `BookingDialogTestCase`, содержащий данные наших тестовых случаев. Он содержит начальный объект `BookingDetails`, ожидаемый объект `BookingDetails` и массив строк с пользовательскими речевыми фрагментами и ожидаемыми ответами из диалога для каждого шага.

```csharp
public class BookingDialogTestCase
{
    public BookingDetails InitialBookingDetails { get; set; }

    public string[,] UtterancesAndReplies { get; set; }

    public BookingDetails ExpectedBookingDetails { get; set; }
}
```

Мы также создали вспомогательный класс `BookingDialogTestsDataGenerator`, который предоставляет метод `IEnumerable<object[]> BookingFlows()` для получения коллекции тестовых случаев, которые будут использоваться в тесте.

Чтобы каждый тестовый случай отображался в виде отдельного элемента в обозревателе тестов Visual Studio, средство запуска тестов XUnit требует, чтобы сложные типы, например `BookingDialogTestCase`, реализовали `IXunitSerializable`. Для упрощения Bot.Builder.Testing предоставляет класс `TestDataObject`, который реализует этот интерфейс и который может использоваться для упаковки данных тестовых случаев без реализации `IXunitSerializable`. 

Ниже приведен фрагмент кода `IEnumerable<object[]> BookingFlows()`, который показывает использование двух классов:

```csharp
public static class BookingDialogTestsDataGenerator
{
    public static IEnumerable<object[]> BookingFlows()
    {
        // Create the first test case object
        var testCaseData = new BookingDialogTestCase
        {
            InitialBookingDetails = new BookingDetails(),
            UtterancesAndReplies = new[,]
            {
                { "hi", "Where would you like to travel to?" },
                { "Seattle", "Where are you traveling from?" },
                { "New York", "When would you like to travel?" },
                { "tomorrow", $"Please confirm, I have you traveling to: Seattle from: New York on: {DateTime.Now.AddDays(1):yyyy-MM-dd}. Is this correct? (1) Yes or (2) No" },
                { "yes", null },
            },
            ExpectedBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = $"{DateTime.Now.AddDays(1):yyyy-MM-dd}",
            }, 
        };
        // wrap the test case object into TestDataObject and return it.
        yield return new object[] { new TestDataObject(testCaseData) };

        // Create the second test case object
        testCaseData = new BookingDialogTestCase
        {
            InitialBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = null,
            },
            UtterancesAndReplies = new[,]
            {
                { "hi", "When would you like to travel?" },
                { "tomorrow", $"Please confirm, I have you traveling to: Seattle from: New York on: {DateTime.Now.AddDays(1):yyyy-MM-dd}. Is this correct? (1) Yes or (2) No" },
                { "yes", null },
            },
            ExpectedBookingDetails = new BookingDetails
            {
                Destination = "Seattle",
                Origin = "New York",
                TravelDate = $"{DateTime.Now.AddDays(1):yyyy-MM-dd}",
            },
        };
        // wrap the test case object into TestDataObject and return it.
        yield return new object[] { new TestDataObject(testCaseData) };
    }
}
```

Создав объект для хранения тестовых данных и предоставляющий коллекцию тестовых случаев класс, мы воспользуемся атрибутом `MemberData` XUnit вместо `InlineData`, чтобы передать данные в тест. Первый параметр для `MemberData` — это имя статической функции, которая возвращает коллекцию тестовых случаев, а второй параметр — это тип класса, который предоставляет этот метод.

```csharp
[Theory]
[MemberData(nameof(BookingDialogTestsDataGenerator.BookingFlows), MemberType = typeof(BookingDialogTestsDataGenerator))]
public async Task DialogFlowUseCases(TestDataObject testData)
{
    // Get the test data instance from TestDataObject
    var bookingTestData = testData.GetObject<BookingDialogTestCase>();
    var sut = new BookingDialog();
    var testClient = new DialogTestClient(Channels.Test, sut, bookingTestData.InitialBookingDetails);

    // Iterate over the utterances and replies array.
    for (var i = 0; i < bookingTestData.UtterancesAndReplies.GetLength(0); i++)
    {
        var reply = await testClient.SendActivityAsync<IMessageActivity>(bookingTestData.UtterancesAndReplies[i, 0]);
        Assert.Equal(bookingTestData.UtterancesAndReplies[i, 1], reply?.Text);
    }

    // Assert the resulting BookingDetails object
    var bookingResults = (BookingDetails)testClient.DialogTurnResult.Result;
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.Origin, bookingResults?.Origin);
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.Destination, bookingResults?.Destination);
    Assert.Equal(bookingTestData.ExpectedBookingDetails?.TravelDate, bookingResults?.TravelDate);
}
```

Ниже приведен пример результатов для тестов `DialogFlowUseCases` в обозревателе тестов Visual Studio при их выполнении.

![BookingDialogTests](media/how-to-unit-test/cs/BookingDialogTestsResults.png)

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="simple-data-driven-tests"></a>Простые тесты, управляемые данными

Следующий тест проверяет, что диалог отменяется, когда пользователь вводит cancel (Отмена).

```javascript
describe('ShouldBeAbleToCancel', () => {
    it('Should cancel', async () => {
        const sut = new TestCancelAndHelpDialog();
        const client = new DialogTestClient('test', sut, null, [new DialogTestLogger()]);

        // Execute the test case
        let reply = await client.sendActivity('Hi');
        assert.strictEqual(reply.text, 'Hi there');
        assert.strictEqual(client.dialogTurnResult.status, 'waiting');

        reply = await client.sendActivity('cancel');
        assert.strictEqual(reply.text, 'Cancelling...');
    });
});
```

Учтите, что позже нам нужно обеспечить возможность обрабатывать и другие речевые фрагменты, например quit (Выйти), never mind (Не важно) и stop it (Остановить). Чтобы не писать три повторяющихся теста для каждого нового речевого фрагмента, можно выполнить рефакторинг теста, чтобы использовать список речевых фрагментов и определить параметры для каждого тестового случая:

```javascript
describe('ShouldBeAbleToCancel', () => {
    const testCases = ['cancel', 'quit', 'never mind', 'stop it'];

    testCases.map(testData => {
        it(testData, async () => {
            const sut = new TestCancelAndHelpDialog();
            const client = new DialogTestClient('test', sut, null, [new DialogTestLogger()]);

            // Execute the test case
            let reply = await client.sendActivity('Hi');
            assert.strictEqual(reply.text, 'Hi there');
            assert.strictEqual(client.dialogTurnResult.status, 'waiting');

            reply = await client.sendActivity(testData);
            assert.strictEqual(reply.text, 'Cancelling...');
        });
    });
});
```

Новый тест будет выполнен четыре раза с разными параметрами, и каждый вариант будет отображен как дочерний элемент в наборе тестов `ShouldBeAbleToCancel` в [обозревателе тестов Mocha](https://marketplace.visualstudio.com/items?itemName=hbenl.vscode-mocha-test-adapter). Если один из тестов завершится с ошибкой, как показано ниже, не нужно перезапускать весь набор тестов. Просто запустите скрипт, который завершился сбоем, и выполните его отладку.

![SimpleCancelTestResults](media/how-to-unit-test/js/SimpleCancelTestResults.png)

### <a name="data-driven-tests-with-complex-types"></a>Управляемые данными тесты со сложными типами

Небольшой список речевых фрагментов удобно использовать для небольших тестов, управляемых данными, которые получают параметры со значениями простых типов (строка, целое число и т. д.) или небольшие объекты.

`BookingDialog` получает объект `BookingDetails` и возвращает новый объект `BookingDetails`. Непараметризованная версия теста для этого диалогового окна будет выглядеть следующим образом:

```javascript
describe('BookingDialog', () => {
    it('Returns expected booking details', async () => {
        // Initial parameters
        const initialBookingDetails = {
            origin: 'Seattle',
            destination: undefined,
            travelDate: undefined
        };

        // Expected booking details
        const expectedBookingDetails = {
            origin: 'Seattle',
            destination: 'New York',
            travelDate: '2019-06-25'
        };

        const sut = new BookingDialog('bookingDialog');
        const client = new DialogTestClient('test', sut, initialBookingDetails, [new DialogTestLogger()]);

        // Execute the test case
        const reply = await client.sendActivity('Hi');
        ...

        // Check dialog results
        const result = client.dialogTurnResult.result;
        assert.strictEqual(result.destination, expectedBookingDetails.destination);
        assert.strictEqual(result.origin, expectedBookingDetails.origin);
        assert.strictEqual(result.travelDate, expectedBookingDetails.travelDate);
    });
});
```

Чтобы параметризировать этот тест, мы создали модуль `bookingDialogTestCases`, возвращающий данные тестовых случаев. Каждый элемент содержит имя тестового случая, начальный объект BookingDetails, ожидаемый объект BookingDetails и массив строк с пользовательскими речевыми фрагментами и ожидаемыми ответами из диалога для каждого шага.

```javascript
module.exports = [
    // Create the first test case object
    {
        name: 'Full flow',
        initialData: {},
        steps: [
            ['hi', 'To what city would you like to travel?'],
            ['Seattle', 'From what city will you be travelling?'],
            ['New York', 'On what date would you like to travel?'],
            ['tomorrow', `Please confirm, I have you traveling to: Seattle from: New York on: ${ tomorrow }. Is this correct? (1) Yes or (2) No`],
            ['yes', null]
        ],
        expectedResult: {
            destination: 'Seattle',
            origin: 'New York',
            travelDate: tomorrow
        }
    },
    // Create the second test case object
    {
        name: 'Destination and Origin provided',
        initialData: {
            destination: 'Seattle',
            origin: 'New York'
        },
        steps: [
            ['hi', 'On what date would you like to travel?'],
            ['tomorrow', `Please confirm, I have you traveling to: Seattle from: New York on: ${ tomorrow }. Is this correct? (1) Yes or (2) No`],
            ['yes', null]
        ],
        expectedStatus: 'complete',
        expectedResult: {
            destination: 'Seattle',
            origin: 'New York',
            travelDate: tomorrow
        }
    }
];
```

Создав список с тестовыми данными, мы можем выполнить рефакторинг теста, чтобы связать этот список с отдельными тестовыми случаями.

```javascript
describe('DialogFlowUseCases', () => {
    const testCases = require('./testData/bookingDialogTestCases.js');

    testCases.map(testData => {
        it(testData.name, async () => {
            const sut = new BookingDialog('bookingDialog');
            const client = new DialogTestClient('test', sut, testData.initialData, [new DialogTestLogger()]);

            // Execute the test case
            for (let i = 0; i < testData.steps.length; i++) {
                const reply = await client.sendActivity(testData.steps[i][0]);
                assert.strictEqual((reply ? reply.text : null), testData.steps[i][1]);
            }

            // Check dialog results
            const actualResult = client.dialogTurnResult.result;
            assert.strictEqual(actualResult.destination, testData.expectedResult.destination);
            assert.strictEqual(actualResult.origin, testData.expectedResult.origin);
            assert.strictEqual(actualResult.travelDate, testData.expectedResult.travelDate);
        });
    });
});
```

Ниже приведен пример результатов для набора тестов `DialogFlowUseCases` в обозревателе тестов Mocha при его выполнении:

![BookingDialogTests](media/how-to-unit-test/js/BookingDialogTestsResults.png)

---

## <a name="using-mocks"></a>Использование макетов

Вы можете использовать макеты для элементов, которые вы сейчас не тестируете. В справочных целях этот уровень можно рассматривать как модульное и интеграционное тестирование.

Макетирование максимального количества элементов обеспечивает лучшую изоляцию тестируемого объекта. Макетами элементов может быть хранилище, адаптер, ПО промежуточного слоя, конвейер действия, каналы и все, что напрямую не является частью бота. Использование макетов также может предусматривать временное удаление определенных элементов, например ПО промежуточного слоя, являющегося частью тестируемого бота. Это нужно для изоляции каждого элемента. Тем не менее, если вы тестируете ПО промежуточного слоя, возможно, вы захотите макетировать бот вместо этого.

Макетирование элементов может принимать разнообразные формы — от замены элемента другим известным объектом до реализации простой функции Hello World. Это может также принимать форму простого удаления элемента, если он не нужен, или его можно принудительно настроить, чтобы он не выполнял никаких действий.

Макеты позволяют настроить зависимости диалога и убедиться, что они находятся в известном состоянии во время выполнения теста. При этом вам не нужно полагаться на внешние ресурсы, включая базы данных, модели LUIS или другие объекты.

Чтобы облегчить тестирование диалогов и сократить зависимости от внешних объектов, вам может потребоваться внедрить внешние зависимости в конструктор диалогов.

Например, вместо создания экземпляра `BookingDialog` в `MainDialog` сделайте следующее:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public MainDialog()
    : base(nameof(MainDialog))
{
    ...
    AddDialog(new BookingDialog());
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor() {
    super('MainDialog');
    ...
    this.addDialog(new BookingDialog('bookingDialog'));
    ...
}
```

---

Мы передаем экземпляр `BookingDialog` в качестве параметра конструктора:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public MainDialog(BookingDialog bookingDialog)
    : base(nameof(MainDialog))
{
    ...
    AddDialog(bookingDialog);
    ...
}
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(bookingDialog) {
    super('MainDialog');
    ...
    this.addDialog(bookingDialog);
    ...
}
```

---

Так мы сможем заменить экземпляр `BookingDialog` макетом объекта и написать модульные тесты для `MainDialog`, не вызывая фактический класс `BookingDialog`.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Create the mock object
var mockDialog = new Mock<BookingDialog>();

// Use the mock object to instantiate MainDialog
var sut = new MainDialog(mockDialog.Object);

var testClient = new DialogTestClient(Channels.Test, sut);
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Create the mock object
const mockDialog = new MockBookingDialog();

// Use the mock object to instantiate MainDialog
const sut = new MainDialog(mockDialog);

const testClient = new DialogTestClient('test', sut);
```

---

### <a name="mocking-dialogs"></a>Макетирование диалогов

Как описано выше, `MainDialog` вызывает `BookingDialog` для получения объекта `BookingDetails`. Чтобы реализовать и настроить экземпляр макета `BookingDialog`, сделайте следующее:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Create the mock object for BookingDialog.
var mockDialog = new Mock<BookingDialog>();
mockDialog
    .Setup(x => x.BeginDialogAsync(It.IsAny<DialogContext>(), It.IsAny<object>(), It.IsAny<CancellationToken>()))
    .Returns(async (DialogContext dialogContext, object options, CancellationToken cancellationToken) =>
    {
        // Send a generic activity so we can assert that the dialog was invoked.
        await dialogContext.Context.SendActivityAsync($"{mockDialogNameTypeName} mock invoked", cancellationToken: cancellationToken);

        // Create the BookingDetails instance we want the mock object to return.
        var expectedBookingDialogResult = new BookingDetails()
        {
            Destination = "Seattle",
            Origin = "New York",
            TravelDate = $"{DateTime.UtcNow.AddDays(1):yyyy-MM-dd}"
        };

        // Return the BookingDetails we need without executing the dialog logic.
        return await dialogContext.EndDialogAsync(expectedBookingDialogResult, cancellationToken);
    });

// Create the sut (System Under Test) using the mock booking dialog.
var sut = new MainDialog(mockDialog.Object);
```

В этом примере мы использовали [Moq](https://github.com/moq/moq) для создания макета диалога, а также методы `Setup` и `Returns` для настройки его поведения.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
class MockBookingDialog extends BookingDialog {
    constructor() {
        super('bookingDialog');
    }

    async beginDialog(dc, options) {
        // Send a generic activity so we can assert that the dialog was invoked.
        await dc.context.sendActivity(`${ this.id } mock invoked`);

        // Create the BookingDetails instance we want the mock object to return.
        const bookingDetails = {
            origin: 'New York',
            destination: 'Seattle',
            travelDate: '2025-07-08'
        };

        // Return the BookingDetails we need without executing the dialog logic.
        return await dc.endDialog(bookingDetails);
    }
}
...
// Create the sut (System Under Test) using the mock booking dialog.
const sut = new MainDialog(new MockBookingDialog());
...

```

В этом примере мы реализуем макет диалога, сделав его производным от `BookingDialog`, и переопределим метод `beginDialog`, чтобы обойти базовую логику диалога.

---

### <a name="mocking-luis-results"></a>Макетирование результатов LUIS

В простых сценариях можно реализовать результаты макета LUIS с помощью кода. Для этого сделайте следующее:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var mockRecognizer = new Mock<IRecognizer>();
mockRecognizer
    .Setup(x => x.RecognizeAsync<FlightBooking>(It.IsAny<ITurnContext>(), It.IsAny<CancellationToken>()))
    .Returns(() =>
    {
        var luisResult = new FlightBooking
        {
            Intents = new Dictionary<FlightBooking.Intent, IntentScore>
            {
                { FlightBooking.Intent.BookFlight, new IntentScore() { Score = 1 } },
            },
            Entities = new FlightBooking._Entities(),
        };
        return Task.FromResult(luisResult);
    });
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript

// Create a mock class for the recognizer that overrides executeLuisQuery.
class MockFlightBookingRecognizer extends FlightBookingRecognizer {
    constructor(mockResult) {
        super();
        this.mockResult = mockResult;
    }

    async executeLuisQuery(context) {
        return this.mockResult;
    }
}
...
// Create a mock result from a string
const mockLuisResult = JSON.parse(`{"intents": {"BookFlight": {"score": 1}}, "entities": {"$instance": {}}}`);
// Use the mock result with the mock recognizer.
const mockRecognizer = new MockFlightBookingRecognizer(mockLuisResult);
...
```

---

Но результаты LUIS могут быть сложными. В таком случае проще записать желаемый результат в JSON-файл, добавить его в проект в качестве ресурса и десериализовать в результат LUIS. Вот пример:

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
var mockRecognizer = new Mock<IRecognizer>();
mockRecognizer
    .Setup(x => x.RecognizeAsync<FlightBooking>(It.IsAny<ITurnContext>(), It.IsAny<CancellationToken>()))
    .Returns(() =>
    {
        // Deserialize the LUIS result from embedded json file in the TestData folder.
        var bookingResult = GetEmbeddedTestData($"{GetType().Namespace}.TestData.FlightToMadrid.json");

        // Return the deserialized LUIS result.
        return Task.FromResult(bookingResult);
    });
```

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Create a mock result from a json file
const mockLuisResult = require(`./testData/FlightToMadrid.json`);
// Use the mock result with the mock recognizer.
const mockRecognizer = new MockFlightBookingRecognizer(mockLuisResult);
```

---

## <a name="additional-information"></a>Дополнительная информация

- [Пример теста CoreBot Test (C#)](https://aka.ms/cs-core-test-sample)
- [Пример теста CoreBot Test (JavaScript)](https://aka.ms/js-core-test-sample)
- [Тестирование ботов](https://github.com/microsoft/botframework-sdk/blob/master/specs/testing/testing.md)

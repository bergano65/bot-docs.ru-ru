---
title: Использование служб LUIS и QnA со средством Dispatch | Документация Майкрософт
description: Сведения об использовании служб LUIS и QnA Maker в боте.
keywords: Интеллектуальная служба распознавания речи (LUIS), служба QnA Maker, средство подготовки к отправке, несколько служб
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/31/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4d029dc7361ac8a7fadb61141faf60d8a62eab3c
ms.sourcegitcommit: a496714fb72550a743d738702f4f79e254c69d06
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/01/2018
ms.locfileid: "50736682"
---
# <a name="use-luis-and-qna-services-with-the-dispatch-tool"></a>Использование служб LUIS и QnA со средством Dispatch

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

<!--TODO: Add the JS sections back in and update them once the JS sample is working.-->

Руководство по использованию модели LUIS, созданной с помощью средства подготовки к отправке, для интеграции бота с несколькими приложениями распознавания речи (LUIS) и службами QnA Maker. Этот пример объединяет следующие службы.

| Service type (Тип службы) | ИМЯ | ОПИСАНИЕ |
|------|------|------|
| Приложение LUIS | HomeAutomation | Распознает намерение для службы автоматизации с данными связанной сущности.|
| Приложение LUIS | Weather | Распознает намерения Weather.GetForecast и Weather.GetCondition с данными о расположении.|
| Служба QnA Maker | Часто задаваемые вопросы  | Предоставляет ответы на несколько простых вопросов о боте. |

Код для этой статьи взят из примера **NLP с Dispatch** [[C#](https://aka.ms/dispatch-sample-cs)].

<!-- | [JS](https://aka.ms/dispatch-sample-js)-->

Общие сведения о языковых службах см. в статье о [распознавании речи](bot-builder-concept-luis.md). Изучите статьи с инструкциями для [LUIS](bot-builder-howto-v4-luis.md) и [QnA Maker](bot-builder-howto-qna.md) по реализации языковых служб в ботах.

Выполнив инструкции в файле README, вы можете настроить и проверить пример бота. Также можно сразу перейти к [примечаниям к коду](#notes-about-the-code).

## <a name="create-the-services-and-test-the-bot"></a>Создание служб и тестирование бота

Выполните инструкции из файла README для этого примера. Вы примените средства интерфейса командной строки для создания и публикации этих служб и обновления сведений о них в файле конфигурации (**.bot**).

1. Клонируйте репозиторий примеров или извлеките из него данные.
1. Установите средства CLI для Bot Builder.
1. Вручную настройте необходимые службы.

### <a name="test-your-bot"></a>Тестирование бота

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Запустите бот с помощью Visual Studio или Visual Studio Code.

<!--
# [JavaScript](#tab/javascript)
-->

---

Подключитесь к боту через Bot Framework Emulator.

Вот некоторые входные данные для включенных в пример служб.

* QnA Maker
  * `hi`, `good morning`
  * `what are you`, `what do you do`
* LUIS (автоматизация дома)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS (погода)
  * `whats the weather in chennai india`
  * `what's the forecast for bangalore`
  * `show me the forecast for nebraska`

## <a name="notes-about-the-code"></a>Примечания о коде

### <a name="packages"></a>Пакеты

В этом руководстве используются следующие пакеты.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Последняя версия v4 этих [пакетов NuGet](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package).

* `Microsoft.Bot.Builder`
* `Microsoft.Bot.Builder.AI.Luis`
* `Microsoft.Bot.Builder.AI.QnA`
* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Configuration`

<!--
# [JavaScript](#tab/javascript)

Download the [LUIS Dispatch sample][DispatchBotJs].  Install the required packages, including the `botbuilder-ai` package for LUIS and QnA Maker, using npm:

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`
-->

---

### <a name="botbuilder-cli-tools"></a>Средства CLI для Bot Builder

Этот пример использует следующие [средства CLI для BotBuilder](https://aka.ms/botbuilder-tools-readme) (их можно получить через npm) для создания, обучения и публикации служб LUIS, QnA Maker и диспетчеризации, а также для сохранения сведений об этих службах в файле конфигурации (**.bot**) вашего бота.

* [Диспетчеризации](https://aka.ms/botbuilder-tools-dispatch)
* [LUDown](https://aka.ms/botbuilder-tools-ludown-readme)
* [LUIS](https://aka.ms/botbuilder-tools-luis)
* [MSBot Command Line tool](https://aka.ms/botbuilder-tools-msbot-readme) (Программа командной строки MSBot)
* [QnAMaker](https://aka.ms/botbuilder-tools-qnaMaker)

> [!TIP]
> Чтобы убедиться в наличии последней версии npm и средств CLI, выполните следующую команду.
>
> ```shell
> npm i -g npm dispatch ludown luis-apis msbot qnamaker
> ```

После настройки служб с помощью этих средств файл **.bot** для нашего примера должен выглядеть примерно следующим образом. (Вы можете запустить `msbot secret -n`, чтобы зашифровать конфиденциальные значения, сохраненные в этом файле.)

```json
{
    "name": "NLP-With-Dispatch-Bot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "luis",
            "name": "Home Automation",
            "appId": "<your-home-automation-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "110"
        },
        {
            "type": "luis",
            "name": "Weather",
            "appId": "<your-weather-luis-app-id>",
            "version": "0.1",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "region": "westus",
            "id": "92"
        },
        {
            "type": "qna",
            "name": "Sample QnA",
            "kbId": "<your-qna-knowledge-base-id>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "endpointKey": "<your-qna-endpoint-key>",
            "hostname": "<your-qna-host-name>",
            "id": "184"
        },
        {
            "type": "dispatch",
            "name": "NLP-With-Dispatch-BotDispatch",
            "appId": "<your-dispatch-app-id>",
            "authoringKey": "<your-luis-authoring-key>",
            "subscriptionKey": "<your-cognitive-services-subscription-key>",
            "version": "Dispatch",
            "region": "westus",
            "serviceIds": [
                "110",
                "92",
                "184"
            ],
            "id": "27"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

### <a name="connecting-to-the-services-from-your-bot"></a>Подключение к службам из кода бота

Чтобы подключиться к службам диспетчеризации, LUIS и (или) QnA Maker, бот получает сведения из файла **.bot**.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В **Startup.cs** `ConfigureServices` считывает файл конфигурации, а `InitBotServices` применяет эту информацию для инициализации службы. При каждом создании бот инициализируется зарегистрированным объектом `BotServices`. Ниже приведены соответствующие фрагменты этих двух методов.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    var connectedServices = InitBotServices(botConfig);

    services.AddSingleton(sp => connectedServices);
    //...
}
```

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    var luisServices = new Dictionary<string, LuisRecognizer>();

    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.Luis:
                {
                    // ...
                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    luisServices.Add(luis.Name, recognizer);
                    break;
                }

            case ServiceTypes.Dispatch:
                // ...
                var dispatchApp = new LuisApplication(dispatch.AppId, dispatch.AuthoringKey, dispatch.GetEndpoint());

                // Since the Dispatch tool generates a LUIS model, we use the LuisRecognizer to resolve the
                // dispatching of the incoming utterance.
                var dispatchARecognizer = new LuisRecognizer(dispatchApp);
                luisServices.Add(dispatch.Name, dispatchARecognizer);
                break;

            case ServiceTypes.QnA:
                {
                    // ...
                    var qnaEndpoint = new QnAMakerEndpoint()
                    {
                        KnowledgeBaseId = qna.KbId,
                        EndpointKey = qna.EndpointKey,
                        Host = qna.Hostname,
                    };

                    var qnaMaker = new QnAMaker(qnaEndpoint);
                    qnaServices.Add(qna.Name, qnaMaker);
                    break;
                }
        }
    }

    return new BotServices(qnaServices, luisServices);
}
```

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

### <a name="calling-the-services-from-your-bot"></a>Вызов служб из кода бота

Логика бота проверяет входные данные пользователя по объединенной модели диспетчеризации.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В файле **NlpDispatchBot.cs** конструктор бота получает объект `BotServices`, который мы зарегистрировали при запуске.

```csharp
private readonly BotServices _services;

public NlpDispatchBot(BotServices services)
{
    _services = services ?? throw new System.ArgumentNullException(nameof(services));

    //...
}
```

В методе `OnTurnAsync` бота мы проверяем входящие сообщения от пользователя по модели диспетчеризации.

```csharp
// Get the intent recognition result
var recognizerResult = await _services.LuisServices[DispatchKey].RecognizeAsync(context, cancellationToken);
var topIntent = recognizerResult?.GetTopScoringIntent();

if (topIntent == null)
{
    await context.SendActivityAsync("Unable to get the top intent.");
}
else
{
    await DispatchToTopIntentAsync(context, topIntent, cancellationToken);
}
```

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

### <a name="working-with-the-recognition-results"></a>Применение результатов распознавания

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Когда модель возвращает результат, он позволяет понять, какие службы лучше всего подходят для обработки этого высказывания. Код нашего бота направляет запрос в соответствующую службу и обрабатывает полученный от нее ответ.

```csharp
/// <summary>
/// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
/// </summary>
private async Task DispatchToTopIntentAsync(
    ITurnContext context,
    (string intent, double score)? topIntent,
    CancellationToken cancellationToken = default(CancellationToken))
{
    const string homeAutomationDispatchKey = "l_Home_Automation";
    const string weatherDispatchKey = "l_Weather";
    const string noneDispatchKey = "None";
    const string qnaDispatchKey = "q_sample-qna";

    switch (topIntent.Value.intent)
    {
        case homeAutomationDispatchKey:
            await DispatchToLuisModelAsync(context, HomeAutomationLuisKey);

            // Here, you can add code for calling the hypothetical home automation service, passing in any entity
            // information that you need.
            break;
        case weatherDispatchKey:
            await DispatchToLuisModelAsync(context, WeatherLuisKey);

            // Here, you can add code for calling the hypothetical weather service,
            // passing in any entity information that you need
            break;
        case noneDispatchKey:
            // You can provide logic here to handle the known None intent (none of the above).
            // In this example we fall through to the QnA intent.
        case qnaDispatchKey:
            await DispatchToQnAMakerAsync(context, QnAMakerKey);
            break;

        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivityAsync($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
            break;
    }
}

/// <summary>
/// Dispatches the turn to the request QnAMaker app.
/// </summary>
private async Task DispatchToQnAMakerAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await _services.QnAServices[appName].GetAnswersAsync(context).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivityAsync(results.First().Answer, cancellationToken: cancellationToken);
        }
        else
        {
            await context.SendActivityAsync($"Couldn't find an answer in the {appName}.");
        }
    }
}

/// <summary>
/// Dispatches the turn to the requested LUIS model.
/// </summary>
private async Task DispatchToLuisModelAsync(
    ITurnContext context,
    string appName,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await context.SendActivityAsync($"Sending your request to the {appName} system ...");
    var result = await _services.LuisServices[appName].RecognizeAsync(context, cancellationToken);

    await context.SendActivityAsync($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", result.Intents)}");

    if (result.Entities.Count > 0)
    {
        await context.SendActivityAsync($"The following entities were found in the message:\n\n{string.Join("\n\n", result.Entities)}");
    }
}
```

<!--
# [JavaScript](#tab/javascript)

```javascript
```
-->

---

## <a name="evaluate-the-dispatchers-performance"></a>Оценка производительности диспетчера

Иногда возникают сообщения пользователя, представленные в качестве примеров, как в приложениях LUIS, так и в службах QnA Maker. В этом случае объединенное приложение LUIS, которое создает средство подготовки к отправке, не будет эффективным для входных данных. Производительность приложения можно проверить с помощью параметра `eval`.

```shell
dispatch eval
```

Выполнение команды `dispatch eval` создает файл **Summary.html**, который предоставляет статистику по прогнозируемой производительности языковой модели.

> [!TIP]
> Команду `dispatch eval` можно запустить в любом приложении LUIS, а не только в тех, которые созданы средством диспетчеризации.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>Изменение намерений для дубликатов и пересечений

Просмотрите примеры фраз, помеченные как дубликаты в **Summary.html** и удалите похожие или пересекающиеся примеры. Для примера предположим, что в приложении LUIS `Home Automation` для домашней автоматизации запросы типа "turn on the lights" (Включить свет) сопоставляются с намерением TurnOnLights, а запросы типа "Why won't my lights turn on?" (Почему не включается свет?) сопоставляются с намерением None. Во втором случае запрос передается в службу QnA Maker. При объединении приложения LUIS и службы QnA Maker с помощью диспетчеризации необходимо выполнить следующие действия.

* Удалить намерение "None" из исходного приложения LUIS `Home Automation`. Затем добавить фразы удаленного намерения в намерение "None" в приложение диспетчера.
* Если не удалить намерение "None" из исходного приложения LUIS, необходимо будет добавить в бот логику, чтобы передать сообщения в службу QnA Maker, которые соответствуют этому намерению.

> [!TIP]
> Просмотрите [Best practices for Language Understanding](./bot-builder-concept-luis.md#best-practices-for-language-understanding) (Рекомендации для распознавания речи), чтобы ознакомится с советами по улучшению производительности языковой модели.

## <a name="to-clean-up-resources-from-this-sample"></a>Очистка ресурсов, созданных для этого примера

Этот пример создает некоторое количество приложений и ресурсов. Чтобы удалить их, выполните следующие инструкции.

### <a name="luis-resources"></a>Ресурсы LUIS

1. Войдите на портал [luis.ai](https://www.luis.ai).
1. Перейдите к странице **My Apps** (Мои приложения).
1. Выберите приложения, созданные этим примером.
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. Щелкните **Delete** (Удалить), а затем **ОК** для подтверждения.

### <a name="qna-maker-resources"></a>Ресурсы QnA Maker

1. Войдите на портал [qnamaker.ai](https://www.qnamaker.ai/).
1. Перейдите к странице **Мои базы знаний**.
1. Нажмите кнопку "Удалить" для базы знаний `Sample QnA`, затем щелкните **Удалить** для подтверждения.

### <a name="azure-resources"></a>Ресурсы Azure

> [!WARNING]
> Не удаляйте такие ресурсы, которые могут использоваться другими приложениями и службами.

1. Войдите на [портале Azure](https://portal.azure.com/).
1. Перейдите к странице **Обзор** для ресурса Cognitive Services, который вы создали для этого примера.
1. Щелкните **Удалить**, а затем **Да** для подтверждения.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Распознавание речи](bot-builder-concept-luis.md)
* [Проектирование ботов базы знаний](../bot-service-design-pattern-knowledge-base.md)
* [Настройка подготовки речи](../bot-service-manage-speech-priming.md)
* [Использование LUIS для распознавания речи](bot-builder-howto-v4-luis.md)
* [Использование QnA Maker для ответов на вопросы](bot-builder-howto-qna.md)

---
title: Использование нескольких моделей LUIS и QnA | Документация Майкрософт
description: Сведения об использовании служб LUIS и QnA Maker в боте.
keywords: Luis, QnA, Dispatch tool, multiple services, route intents
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/26/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9b0ddf5cf8af61048ba78f10824c9573da82fc08
ms.sourcegitcommit: a722f960cd0a8513d46062439eb04de3a0275346
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/27/2018
ms.locfileid: "52336274"
---
# <a name="use-multiple-luis-and-qna-models"></a>Использование нескольких моделей LUIS и QnA

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

В этом руководстве мы покажем, как использовать службу Dispatch для передачи высказываний в системе с несколькими моделями LUIS и QnA Maker, чтобы реализовать в боте все требуемые сценарии. В этом примере мы настроим диспетчеризацию между несколькими моделями LUIS, чтобы поддерживать беседы об автоматизации дома и о погоде, и службой QnA Maker для ответов на вопросы по текстовому файлу вопросов и ответов. Этот пример объединяет следующие службы.

| ИМЯ | ОПИСАНИЕ |
|------|------|
| HomeAutomation | Приложение LUIS распознает намерение обращения к службе автоматизации и данные о сущностях.|
| Weather | Приложение LUIS распознает намерения `Weather.GetForecast` и `Weather.GetCondition`, а также данные о расположении.|
| Часто задаваемые вопросы  | База данных QnA Maker предоставляет ответы на несколько простых вопросов о боте. |

## <a name="prerequisites"></a>Предварительные требования

- Код в этой статье основан на примере **обработки естественного языка с помощью средства Dispatch**. Вам потребуется копия этого примера на языке [C#](https://aka.ms/dispatch-sample-cs) или [JS](https://aka.ms/dispatch-sample-js).
- Требуется понимание [основных концепций ботов](bot-builder-basics.md), [обработки естественного языка](bot-builder-howto-v4-luis.md), [QnA Maker](bot-builder-howto-qna.md) и файла [.bot](bot-file-basics.md).
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download) для целей тестирования.

## <a name="create-the-services-and-test-the-bot"></a>Создание служб и тестирование бота

Выполните инструкции из файла **README** для примеров на [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/14.nlp-with-dispatch/README.md) или [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/14.nlp-with-dispatch/README.md), чтобы собрать этот пример и запустить его в эмуляторе. 

Для справки ниже приведены некоторые вопросы и команды, поддерживаемые в добавленных службах.

* QnA Maker
  * `hi`, `good morning`
  * `what are you`, `what do you do`
* LUIS (автоматизация дома)
  * `turn on bedroom light`
  * `turn off bedroom light`
  * `make some coffee`
* LUIS (погода)
  * `whats the weather in redmond washington`
  * `what's the forecast for london`
  * `show me the forecast for nebraska`

### <a name="connecting-to-the-services-from-your-bot"></a>Подключение к службам из кода бота

Чтобы подключиться к службам диспетчеризации, LUIS и (или) QnA Maker, бот получает сведения из файла **.bot**.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В **Startup.cs** `ConfigureServices` считывает файл конфигурации, а `InitBotServices` применяет эту информацию для инициализации службы. При каждом создании бот инициализируется зарегистрированным объектом `BotServices`. Ниже приведены соответствующие фрагменты этих двух методов.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    //...
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-dispatch.bot", secretKey);
    services.AddSingleton(sp => botConfig
        ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // ...
    
    var connectedServices = InitBotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    
    services.AddBot<NlpDispatchBot>(options =>
    {
          
          // The Memory Storage used here is for local bot debugging only. 
          // When the bot is restarted, everything stored in memory will be gone.

          Storage dataStore = new MemoryStorage();

          // ...

          // Create Conversation State object.
          // The Conversation State object is where we persist anything at the conversation-scope.

          var conversationState = new ConversationState(dataStore);
          options.State.Add(conversationState);
     });
}

```
Следующий код инициализирует ссылки бота на внешние службы. Например, здесь создаются службы LUIS и QnaMaker. Эти внешние службы настраиваются в классе `BotConfiguration` по данным из файла .bot.

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
// Depending on the intent from Dispatch, routes to the right LUIS model or QnA service.
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

// Dispatches the turn to the request QnAMaker app.
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


// Dispatches the turn to the requested LUIS model.
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

Выполнение команды `dispatch eval` создает файл **Summary.html**, который предоставляет статистику по прогнозируемой производительности языковой модели. Команду `dispatch eval` можно запустить в любом приложении LUIS, а не только в тех, которые созданы средством диспетчеризации.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>Изменение намерений для дубликатов и пересечений

Просмотрите примеры фраз, помеченные как дубликаты в **Summary.html** и удалите похожие или пересекающиеся примеры. Для примера предположим, что в приложении LUIS `Home Automation` для домашней автоматизации запросы типа "turn on the lights" (Включить свет) сопоставляются с намерением TurnOnLights, а запросы типа "Why won't my lights turn on?" (Почему не включается свет?) сопоставляются с намерением None. Во втором случае запрос передается в службу QnA Maker. При объединении приложения LUIS и службы QnA Maker с помощью диспетчеризации необходимо выполнить следующие действия.

* Удалить намерение "None" из исходного приложения LUIS `Home Automation`. Затем добавить фразы удаленного намерения в намерение "None" в приложение диспетчера.
* Если не удалить намерение "None" из исходного приложения LUIS, необходимо будет добавить в бот логику, чтобы передать сообщения в службу QnA Maker, которые соответствуют этому намерению.


## <a name="additional-resources"></a>Дополнительные ресурсы 

**Удаление ресурсов.** Этот пример создает некоторое количество приложений и ресурсов, которые можно удалить с помощью описанных ниже действий. Следите при этом за тем, чтобы не удалить ресурсы, от которых зависят *другие приложения и (или) службы*. 

_Ресурсы LUIS_
1. Войдите на портал [luis.ai](https://www.luis.ai).
1. Перейдите к странице _My Apps_ (Мои приложения).
1. Выберите приложения, созданные этим примером.
   * `Home Automation`
   * `Weather`
   * `NLP-With-Dispatch-BotDispatch`
1. Щелкните _Delete_ (Удалить), а затем _ОК_ для подтверждения.

_Ресурсы QnA Maker_
1. Войдите на портал [qnamaker.ai](https://www.qnamaker.ai/).
1. Перейдите к странице _Мои базы знаний_.
1. Нажмите кнопку "Удалить" для базы знаний `Sample QnA`, затем щелкните _Удалить_ для подтверждения.

**Рекомендация.** Чтобы улучшить описанные здесь службы, изучите рекомендации по [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) и [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices).

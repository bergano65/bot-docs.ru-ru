---
title: Добавление возможности распознавания естественного языка в функционал бота | Документация Майкрософт
description: Сведения об использовании LUIS для распознавания естественного языка с пакетом SDK для Bot Builder.
keywords: Распознавание речи, LUIS, намерение, распознаватель, сущности, ПО промежуточного слоя
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: faf26b1c4ba87061631f217ee074283759f77c97
ms.sourcegitcommit: 392c581aa2f59cd1798ee2136b6cfee56aa3ee6d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/20/2018
ms.locfileid: "52156703"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>Добавление возможности распознавания естественного языка в функционал бота

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Возможность понимать, что пользователь хочет сказать и какой вкладывает контекст, может быть сложной задачей, но также может способствовать более естественной беседе с ботом. API распознавания речи, так же называемое LUIS, позволяет делать так, чтобы бот мог распознавать намерения пользовательских сообщений, использовать более естественный язык пользователя и лучше направлять последовательность общения. В этом разделе описывается настройка простого бота, который использует LUIS для распознавания нескольких разных намерений. 
## <a name="prerequisites"></a>Предварительные требования
- Учетная запись [luis.ai](https://www.luis.ai)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download).
- Код в этой статье основан на примере **обработки естественного языка с помощью LUIS**. Вам потребуется копия этого примера на языке [C#](https://aka.ms/cs-luis-sample) или [JS](https://aka.ms/js-luis-sample). 
- Понимание [основных концепций ботов](bot-builder-basics.md), [обработки естественного языка](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis) и файла [.bot](bot-file-basics.md).

## <a name="create-a-luis-app-in-the-luis-portal"></a>Создание приложения LUIS на портале LUIS
Войдите на портал LUIS, чтобы создать собственную версию примера приложения LUIS. Создавать и администрировать приложения можно на странице **Мои приложения**. 

1. Выберите **Import new app** (Импортировать новое приложение). 
1. Щелкните **Choose App file (JSON format)…** (Выберите файл приложения в формате JSON) 
1. Выберите файл `reminders.json`, расположенный в папке `CognitiveModels` примера. В поле **Optional Name** (Необязательное имя) введите значение **LuisBot**. Этот файл содержит три намерения: Calendar-Add, Calendar-Find и None. Мы будем использовать эти намерения для распознавания желаний пользователя в полученном от него сообщении. 
1. [Обучите](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train) приложение.
1. [Опубликуйте](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp) приложение в *рабочей* среде.

### <a name="obtain-values-to-connect-to-your-luis-app"></a>Получение значений для подключения к приложению LUIS

После публикации приложения LUIS ваш бот сможет обратиться к нему. Для доступа к приложению LUIS из кода бота потребуется записать несколько значений. Нужные сведения можно получить с помощью портала LUIS.

#### <a name="retrieve-application-information-from-the-luisai-portal"></a>Получение сведений о приложении на портале LUIS.ai
Файл .bot используется для того, чтобы собрать в одном месте ссылки на все службы. Полученные данные будут добавлены в файл .bot в следующем разделе. 
1. Выберите опубликованное приложение LUIS на сайте [luis.ai](https://www.luis.ai).
1. Открыв опубликованное приложение LUIS, выберите в нем вкладку **MANAGE** (Управление).
1. Выберите слева вкладку **Application Information** (Сведения о приложении) и сохраните значение из поля _Application ID_ (Идентификатор приложения) в параметр <YOUR_APP_ID>.
1. Выберите слева вкладку **Keys and Endpoints** (Ключи и конечные точки) и сохраните значение из поля _Authoring Key_ (Ключ разработки) в параметр <YOUR_AUTHORING_KEY>. Обратите внимание, что *ключ подписки* совпадает со значением *ключа разработки*. 
1. Прокрутите страницу вниз до конца и перенесите значение из поля _Region_ (Регион) в параметр <YOUR_REGION>.
1. Перенесите значение из поля _Endpoint точка_ (Конечная точка) в параметр <YOUR_ENDPOINT>.

#### <a name="update-the-bot-file"></a>Обновление файла бота
Добавьте в файл `nlp-with-luis.bot` необходимые сведения для доступа к приложению LUIS, включая идентификатор приложения, ключ разработки, ключ подписки, конечную точку и регион. Все эти данные вы ранее сохранили из опубликованного приложения LUIS.

```json
{
    "name": "LuisBot",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "166"
        },
        {
            "type": "luis",
            "name": "LuisBot",
            "appId": "<luis appid>",
            "version": "0.1",
            "authoringKey": "<luis authoring key>",
            "subscriptionKey": "<luis subscription key>",
            "region": "<luis region>",
            "id": "158"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```
# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="configure-your-bot-to-use-your-luis-app"></a>Настройка бота для работы с приложением LUIS

Затем мы инициализируем в `BotServices.cs` новый экземпляр класса BotService, который извлекает перечисленные выше сведения из файла `.bot`. Внешняя служба настраивается с помощью класса `BotConfiguration`.

```csharp
public class BotServices
{
    // Initializes a new instance of the BotServices class
    public BotServices(BotConfiguration botConfiguration)
    {
        foreach (var service in botConfiguration.Services)
        {
            switch (service.Type)
            {
                case ServiceTypes.Luis:
                {
                    var luis = (LuisService)service;
                    if (luis == null)
                    {
                        throw new InvalidOperationException("The LUIS service is not configured correctly in your '.bot' file.");
                    }

                    var app = new LuisApplication(luis.AppId, luis.AuthoringKey, luis.GetEndpoint());
                    var recognizer = new LuisRecognizer(app);
                    this.LuisServices.Add(luis.Name, recognizer);
                    break;
                    }
                }
            }
        }

    // Gets the set of LUIS Services used. LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

Теперь зарегистрируйте приложение LUIS как singleton в файле `Startup.cs`, применив следующий код в методе `ConfigureServices`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\nlp-with-luis.bot", secretKey);
    services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

    // Initialize Bot Connected Services clients.
    var connectedServices = new BotServices(botConfig);
    services.AddSingleton(sp => connectedServices);
    services.AddSingleton(sp => botConfig);

    services.AddBot<LuisBot>(options =>
    {
        // Retrieve current endpoint.
        var environment = _isProduction ? "production" : "development";
        var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
        if (!(service is EndpointService endpointService))
        {
            throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
        }

        options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);
        
        // ...
    });
}
```

Теперь в файле `Luis.cs` бот получает этот экземпляр LUIS.

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    // Services configured from the ".bot" file.
    private readonly BotServices _services;

    // Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration....");
        }
    }
    // ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В нашем примере код запуска находится в файле **index.js**, код с логикой бота — в файле **bot.js**, а дополнительные данные о конфигурации — в файле **nlp-with-luis.bot**.

В файле **bot.js** мы считываем сведения о конфигурации и используем их для создания службы LUIS и инициализации бота.
Замените значение `LUIS_CONFIGURATION` именем приложения LUIS, которое отображается в файле конфигурации.

```javascript
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the C# code.
const LUIS_CONFIGURATION = '<YOUR_LUIS_APP_NAME>';

// Get endpoint and LUIS configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const luisConfig = botConfig.findServiceByNameOrId(LUIS_CONFIGURATION);

// Map the contents to the required format for `LuisRecognizer`.
const luisApplication = {
    applicationId: luisConfig.appId,
    endpointKey: luisConfig.subscriptionKey || luisConfig.authoringKey,
    azureRegion: luisConfig.region
};

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
};

// Create adapter...

// Create the LuisBot.
let bot;
try {
    bot = new LuisBot(luisApplication, luisPredictionOptions);
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

Теперь мы создадим HTTP-сервер для прослушивания входящих запросов, которые будут создавать вызовы логики бота.

```javascript
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open nlp-with-luis.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async(turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

Теперь служба LUIS полностью настроена для вашего бота. Далее рассмотрим, как получить намерение из LUIS.

## <a name="get-the-intent-by-calling-luis"></a>Получение намерения путем вызова LUIS

Бот получает результаты из службы LUIS путем вызова API распознавания речи LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

Если нужно, чтобы бот просто отправлял ответ на основе намерения, распознанного приложением LUIS, вызовите `LuisRecognizer` для получения `RecognizerResult`. Это можно сделать в коде в любой момент, когда нужно получить намерение LUIS.

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))

{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Check LUIS model
        var recognizerResult = await _services.LuisServices[LuisKey].RecognizeAsync(turnContext, cancellationToken);
        var topIntent = recognizerResult?.GetTopScoringIntent();
        if (topIntent != null && topIntent.HasValue && topIntent.Value.intent != "None")
        {
            await turnContext.SendActivityAsync($"==>LUIS Top Scoring Intent: {topIntent.Value.intent}, Score: {topIntent.Value.score}\n");
        }
        else
        {
            var msg = @"No LUIS intents were found.
                        This sample is about identifying two user intents:
                        'Calendar.Add'
                        'Calendar.Find'
                        Try typing 'Add Event' or 'Show me tomorrow'.";
            await turnContext.SendActivityAsync(msg);
        }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            // Send a welcome message to the user and tell them what actions they may perform to use this bot
            await SendWelcomeMessageAsync(turnContext, cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected", cancellationToken: cancellationToken);
        }
}
```

Любые намерения, распознанные в высказывании, будут возвращены в виде сопоставления имен намерений для оценок и могут быть доступны из `recognizerResult.Intents`. Статический метод `recognizerResult?.GetTopScoringIntent()` предоставляется, чтобы упростить поиск главного намерения оценки для набора результатов.

Все распознанные сущности будут возвращены в виде сопоставления имен сущностей и значений, и их можно получить с помощью свойства `recognizerResults.entities`. Дополнительные метаданные сущности можно вернуть, передав параметр `verbose=true` при создании LuisRecognizer. Добавленные метаданные можно получить с помощью свойства `recognizerResults.entities.$instance`.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В файле **bot.js** мы передаем пользовательский ввод в метод `recognize` распознавателя LUIS, чтобы получать ответы из приложения LUIS.

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from the Language Understanding (LUIS) service.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class LuisBot {
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {LuisApplication} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {LuisPredictionOptions} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions, includeApiResults) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }

    /**
     * Every conversation turn calls this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls LUIS in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;

            if (topIntent.intent !== 'None') {
                await turnContext.sendActivity(`LUIS Top Scoring Intent: ${ topIntent.intent }, Score: ${ topIntent.score }`);
            } else {
                // If the top scoring intent was "None" tell the user no valid intents were found and provide help.
                await turnContext.sendActivity(`No LUIS intents were found.
                                                \nThis sample is about identifying two user intents:
                                                \n - 'Calendar.Add'
                                                \n - 'Calendar.Find'
                                                \nTry typing 'Add Event' or 'Show me tomorrow'.`);
            }
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
            turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            // If the Activity is a ConversationUpdate, send a greeting message to the user.
            await turnContext.sendActivity('Welcome to the NLP with LUIS sample! Send me a message and I will try to predict your intent.');
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            // Respond to all other Activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.LuisBot = LuisBot;
```

Распознаватель LUIS возвращает сведения о том, насколько точно высказывание соответствует существующим намерениям. Свойство `luisResult.intents` в объекте результата содержит массив оцененных намерений. Свойство `luisResult.topScoringIntent` в объекте результата содержит намерение с самой высокой оценкой и значение этой оценки.

---

<!--
## Extract entities

Besides recognizing intent, a LUIS app can also extract entities, which are important words for fulfilling a user's request. For example, for a weather bot, the LUIS app might be able to extract the location for the weather report from the user's message.

A common way to structure your conversation is to identify any entities in the user's message, and prompt for any of the required entities that are not found. Then, the subsequent steps handle the response to the prompt.


# [C#](#tab/cs)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult) with an [`Entities` property](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.core.extensions.recognizerresult#properties-) that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

The following helper function can be added to your bot to get entities out of the `RecognizerResult` from LUIS. It will require the use of the `Newtonsoft.Json.Linq` library, which you'll have to add to your **using** statements.

```cs
// Get entities from LUIS result
private T GetEntity<T>(RecognizerResult luisResult, string entityKey)
{
    var data = luisResult.Entities as IDictionary<string, JToken>;
    if (data.TryGetValue(entityKey, out JToken value))
    {
        return value.First.Value<T>();
    }
    return default(T);
}
```

When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

# [JavaScript](#tab/js)

Let's say the message from the user was "What's the weather in Seattle"? The [LuisRecognizer](https://docs.microsoft.com/en-us/javascript/api/botbuilder-ai/luisrecognizer) gives you a [RecognizerResult](https://docs.microsoft.com/en-us/javascript/api/botbuilder-core-extensions/recognizerresult) with an `entities` property that has this structure:

```json
{
"$instance": {
    "Weather_Location": [
        {
            "startIndex": 22,
            "endIndex": 29,
            "text": "seattle",
            "score": 0.8073087
        }
    ]
},
"Weather_Location": [
        "seattle"
    ]
}
```

This `findEntities` function looks for any entities recognized by the LUIS app that match the incoming `entityName`.

```javascript
// Helper function for finding a specified entity
// entityResults are the results from LuisRecognizer.get(context)
function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach(entity => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}


When gathering information like entities from multiple steps in a conversation, it can be helpful to save the information you need in your state. If an entity is found, it can be added to the appropriate state field. In your conversation if the current step already has the associated field completed, the step to prompt for that information can be skipped.

/Snip -->

## <a name="test-the-bot"></a>Тестирование бота

1. Выполните этот пример на локальном компьютере. Если потребуются дополнительные инструкции, изучите файл readme для примера на [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/12.nlp-with-luis/README.md) или [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/README.md).

1. В эмуляторе введите сообщение, как показано ниже. 

![Тестирование примера NLP](~/media/emulator-v4/nlp-luis-sample-testing.png)

В ответе бот возвратит намерение с самой высокой оценкой. В нашем примере это будет намерение `Calendar-Add`. Как вы помните, намерения определяются в файле `reminders.json`, который вы импортировали на портале luis.ai.

Оценка прогнозирования означает степень достоверности результатов прогнозирования LUIS. Оценка прогнозирования находится в диапазоне от нуля (0) до единицы (1). Пример оценки прогнозирования LUIS с высокой степенью достоверности — 0,99. Пример оценки прогнозирования LUIS с низкой достоверностью — 0,01. 

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Использование QnA Maker для ответов на вопросы](./bot-builder-howto-qna.md)

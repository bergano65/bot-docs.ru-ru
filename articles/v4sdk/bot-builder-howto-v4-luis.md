---
title: Использование LUIS для распознавания речи | Документация Майкрософт
description: Сведения об использовании LUIS для распознавания естественного языка с пакетом SDK для Bot Builder.
keywords: Распознавание речи, LUIS, намерение, распознаватель, сущности, ПО промежуточного слоя
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/12/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78654c78282c0a8e73dd17a093d27800f9fc8cb1
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326511"
---
# <a name="using-luis-for-language-understanding"></a>Использование LUIS для распознавания речи

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Возможность понимать, что пользователь хочет сказать и какой вкладывает контекст, может быть сложной задачей, но также может способствовать более естественной беседе с ботом. API распознавания речи, так же называемое LUIS, позволяет делать так, чтобы бот мог распознавать намерения пользовательских сообщений, использовать более естественный язык пользователя и лучше направлять последовательность общения. Дополнительные сведения об интеграции LUIS с ботом см. в разделе [Language Understanding for bots](./bot-builder-concept-LUIS.md) (API распознавания речи для ботов).

## <a name="prerequisites"></a>Предварительные требования
В этом разделе описывается настройка простого бота, который использует LUIS для распознавания нескольких разных намерений. Код в этой статье основан на NLP с примером LUIS для [C#](https://aka.ms/cs-luis-sample) и [JavaScript](https://aka.ms/js-luis-sample).

## <a name="create-a-luis-app-in-the-luis-portal"></a>Создание приложения LUIS на портале LUIS

Прежде всего зарегистрируйте учетную запись в [luis.ai](https://www.luis.ai) и создайте приложение LUIS на портале LUIS, выполнив представленные [здесь](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-start-new-app) инструкции. Если вы хотите создать собственную версию примера приложения LUIS, которое используется в этой статье, на портале LUIS [импортируйте](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) этот файл `LUIS.Reminders.json` ([C#](https://github.com/Microsoft/BotBuilder-Samples/blob/v4/samples/csharp_dotnetcore/12.nlp-with-luis/CognitiveModels/LUIS-Reminders.json) | [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/cognitiveModels/reminders.json)) для создания собственного приложения LUIS, а затем [обучите](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train) и [опубликуйте](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp) его.

### <a name="obtain-values-to-connect-to-your-luis-app"></a>Получение значений для подключения к приложению LUIS

После публикации приложения LUIS ваш бот сможет обратиться к нему. Для доступа к приложению LUIS из кода бота потребуется записать несколько значений. Нужную информацию можно получить с помощью портала LUIS или средств командной строки.

#### <a name="using-luis-portal"></a>Использование портала LUIS
- Выберите опубликованное приложение LUIS на сайте [luis.ai](https://www.luis.ai).
- Открыв опубликованное приложение LUIS, выберите в нем вкладку **MANAGE** (Управление).
- Выберите слева вкладку **Application Information** (Сведения о приложении) и сохраните значение из поля _Application ID_ (Идентификатор приложения) в параметр <YOUR_APP_ID>.
- Выберите слева вкладку **Keys and Endpoints** (Ключи и конечные точки) и сохраните значение из поля _Authoring Key_ (Ключ разработки) в параметр <YOUR_AUTHORING_KEY>. Обратите внимание на то, что значение <YOUR_SUBSCRIPTION_KEY> совпадает со значением <YOUR_AUTHORING_KEY>. Прокрутите вниз до конца страницы, сохраните значение из поля _Region_ (Регион) в параметр <YOUR_REGION> и значение из поля _Endpoint_ (Конечная точка) в параметр <YOUR_ENDPOINT>.

#### <a name="using-cli-tools"></a>Использование средств CLI

Вы можете использовать средства CLI для BotBuilder [luis](https://aka.ms/botbuilder-tools-luis) и [msbot](https://aka.ms/botbuilder-tools-msbot-readme), чтобы получить метаданные о приложении LUIS и добавить их в файл **.bot**.

1. Откройте терминал или командную строку и перейдите в корневой каталог проекта бота.
2. Убедитесь, что установлены средства `luis` и `msbot`.

    ```shell
    npm install luis msbot
    ```

3. Запустите `luis init`, чтобы создать файл ресурсов LUIS (**.luisrc**). Укажите ключ разработки и регион LUIS в ответ на соответствующие запросы. Идентификатор приложения пока вводить не требуется.
4. Выполните следующую команду, чтобы скачать метаданные и добавить их в файл конфигурации бота.
    Если вы используете шифрование файла конфигурации, для его обновления необходимо указать секретный ключ.

    ```shell
    luis get application --appId <your-app-id> --msbot | msbot connect luis --stdin [--secret <YOUR-SECRET>]
    ```

## <a name="configure-your-bot-to-use-your-luis-app"></a>Настройка бота для работы с приложением LUIS

Прежде всего следует добавить ссылку на приложение LUIS при инициализации бота. Затем вы сможете обращаться к нему из логики этого бота.

### <a name="prerequisite"></a>Предварительные требования

Прежде, чем писать код, проверьте наличие всех пакетов, который необходимы для приложения LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

Установите в бот следующий [пакет NuGet](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui).

* `Microsoft.Bot.Builder.AI.Luis`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Компоненты LUIS входят в состав пакета `botbuilder-ai`. Этот пакет можно добавить в проект с помощью npm.

```shell
npm install --save botbuilder-ai
```

---

# <a name="ctabcs"></a>[C#](#tab/cs)

Скачайте и откройте пример кода NLP LUIS, опубликованный [здесь](https://aka.ms/cs-luis-sample). Мы немного изменим этот код для наших задач. 

Во-первых, добавьте в файл `BotConfiguration.bot` необходимые сведения для доступа к приложению LUIS, включая идентификатор приложения, ключ разработки, ключ подписки, конечную точку и регион. Все эти данные вы ранее сохранили из опубликованного приложения LUIS.

```csharp
{
  "name": "LuisBot",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "1"
    },
    {
      "type": "luis",
      "name": "LuisBot",
      "AppId": "<YOUR_APP_ID>",
      "SubscriptionKey": "<YOUR_SUBSCRIPTION_KEY>",
      "AuthoringKey": "<YOUR_AUTHORING_KEY>",
      "GetEndpoint": "<YOUR_ENDPOINT>",
      "Region": "<YOUR_REGION>"
    }
  ],
  "padlock": "",
  "version": "2.0"
}
```

Затем мы инициализируем новый экземпляр класса BotService `BotServices.cs`, который извлекает перечисленные выше сведения из файла `.bot`. Добавьте в файл `BotServices.cs` указанный ниже код.

```csharp
public class BotServices
{
    /// Initializes a new instance of the BotServices class
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

    /// Gets the set of LUIS Services used.
    /// LuisServices is represented as a dictionary.  
    public Dictionary<string, LuisRecognizer> LuisServices { get; } = new Dictionary<string, LuisRecognizer>();
}
```

Теперь мы регистрируем приложение LUIS как отдельное приложение в файле `Startup.cs`, добавляя следующий код в `ConfigureServices`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
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

        // Creates a logger for the application to use.
        ILogger logger = _loggerFactory.CreateLogger<LuisBot>();

        // Catches any errors that occur during a conversation turn and logs them.
        options.OnTurnError = async (context, exception) =>
        {
            logger.LogError($"Exception caught : {exception}");
            await context.SendActivityAsync("Sorry, it looks like something went wrong.");
        };
        /// ...
    });
}
```

Затем нужно передать боту этот экземпляр LUIS. Откройте `LuisBot.cs` и добавьте следующий код в начало файла.

```csharp
public class LuisBot : IBot
{
    public static readonly string LuisKey = "LuisBot";
    private const string WelcomeText = "This bot will introduce you to natural language processing with LUIS. Type an utterance to get started";

    /// Services configured from the ".bot" file.
    private readonly BotServices _services;

    /// Initializes a new instance of the LuisBot class.
    public LuisBot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        if (!_services.LuisServices.ContainsKey(LuisKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a LUIS service named '{LuisKey}'.");
        }
    }
    /// ...
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В нашем примере код запуска находится в файле **index.js**, код с логикой бота — в файле **bot.js**, а дополнительные данные о конфигурации — в файле **nlp-with-luis.bot**.

Когда вы выполните все инструкции по созданию приложения LUIS и обновлению файла **.bot**, файл **nlp-with-luis.bot** будет содержать запись о службе приложения LUIS.

```json
{
    "name": "YOUR_LUIS_APP_NAME",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "35"
        },
        {
            "type": "luis",
            "name": "YOUR_LUIS_APP_NAME",
            "appId": "<YOUR_APP_ID>",
            "version": "0.1",
            "authoringKey": "<YOUR_AUTHORING_KEY>",
            "subscriptionKey": "<YOUR_SUBSCRIPTION_KEY>>",
            "region": "<YOUR_REGION>",
            "id": "83"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

В файле **index.js** мы считываем сведения о конфигурации и используем их для создания службы LUIS и инициализации бота.
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

## <a name="extract-entities"></a>Извлечение сущностей

Кроме распознавания намерений, приложение LUIS умеет извлекать сущности, то есть наиболее важные слова для выполнения запроса пользователя. Например, в случае бота для получения прогноза погоды приложение LUIS может извлечь расположение для прогноза погоды из сообщения пользователя.

Распространенный способ структурирования общения — выявление любых сущностей в сообщении пользователя и запрашивание требуемых сущностей, которые не были обнаружены. Затем в последующих шагах обрабатывается ответ на запрос.

<!--Snip
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
```
/Snip-->

Когда сведения, например сущности, собираются из нескольких шагов разговора, может оказаться полезным сохранить нужные сведения в состоянии. Если сущность найдена, ее можно добавить в соответствующее поле состояния. Если в ходе общения соответствующее поле текущего шага уже заполнено, шаг по запрашиванию этих сведений можно пропустить.

## <a name="additional-resources"></a>Дополнительные ресурсы

Примеры использования LUIS можно найти в проектах для [[C#](https://aka.ms/cs-luis-sample)] или [[JavaScript](https://aka.ms/js-luis-sample)].

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Combine LUIS apps and QnA services using the Dispatch tool](./bot-builder-tutorial-dispatch.md) (Объединение приложений LUIS и служб QnA с помощью средства Dispatch)

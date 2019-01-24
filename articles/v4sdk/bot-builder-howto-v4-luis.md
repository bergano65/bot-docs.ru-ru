---
title: Добавление возможности распознавания естественного языка в функционал бота | Документация Майкрософт
description: Сведения об использовании LUIS для распознавания естественного языка с пакетом SDK Bot Framework.
keywords: Распознавание речи, LUIS, намерение, распознаватель, сущности, ПО промежуточного слоя
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 11/28/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4c43426f508d629c325889da6a9f7b06cac7e846
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453898"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>Добавление возможности распознавания естественного языка в функционал бота

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Возможность понимать, что пользователь хочет сказать и какой вкладывает контекст, может быть сложной задачей, но также может способствовать более естественной беседе с ботом. API распознавания речи, так же называемое LUIS, позволяет делать так, чтобы бот мог распознавать намерения пользовательских сообщений, использовать более естественный язык пользователя и лучше направлять поток общения. В этом разделе описывается настройка простого бота, который использует LUIS для распознавания нескольких разных намерений. 
## <a name="prerequisites"></a>Предварительные требования
- Учетная запись [luis.ai](https://www.luis.ai)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download).
- Код в этой статье основан на примере **обработки естественного языка с помощью LUIS**. Вам потребуется копия этого примера на языке [C#](https://aka.ms/cs-luis-sample) или [JS](https://aka.ms/js-luis-sample). 
- Понимание [основных концепций ботов](bot-builder-basics.md), [обработки естественного языка](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/what-is-luis) и файла [.bot](bot-file-basics.md).

## <a name="create-a-luis-app-in-the-luis-portal"></a>Создание приложения LUIS на портале LUIS
Войдите на портал LUIS, чтобы создать собственную версию примера приложения LUIS. Создавать и администрировать приложения можно на странице **Мои приложения**. 

1. Выберите **Import new app** (Импортировать новое приложение). 
1. Щелкните **Choose App file (JSON format)…** (Выберите файл приложения в формате JSON) 
1. Выберите файл `reminders.json`, расположенный в папке `CognitiveModels` примера. В поле **Optional Name** (Необязательное имя) введите значение **LuisBot**. Этот файл содержит три намерения: Calendar_Add (Добавить в календарь), Calendar_Find (Найти в календаре) и None (Отсутствует). Мы будем использовать эти намерения для распознавания желаний пользователя в полученном от него сообщении. Если вы намерены добавить сущности, воспользуйтесь [дополнительным разделом](#optional---extract-entities) в конце этой статьи.
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

### <a name="configure-your-bot-to-use-your-luis-app"></a>Настройка бота для работы с приложением LUIS

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

Теперь в файле `LuisBot.cs` бот получает этот экземпляр LUIS.

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
// Language Understanding (LUIS) service name as defined in the .bot file.YOUR_LUIS_APP_NAME is "LuisBot" in the JavaScript code.
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

### <a name="get-the-intent-by-calling-luis"></a>Получение намерения путем вызова LUIS

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

### <a name="test-the-bot"></a>Тестирование бота

1. Выполните этот пример на локальном компьютере. Если потребуются дополнительные инструкции, изучите файл readme для примера на [C#](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/12.nlp-with-luis/README.md) или [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/12.nlp-with-luis/README.md).

1. В эмуляторе введите сообщение, как показано ниже. 

![Тестирование входных данных для примера NLP](./media/nlp-luis-sample-message.png)

В ответе бот возвратит намерение с самой высокой оценкой. В нашем примере это будет Calendar_Add. Как вы помните, в файле `reminders.json`, который вы импортировали на портале luis.ai, определены намерения Calendar-Add, Calendar-Find и None. 

![Тестирование ответа в примере NLP](./media/nlp-luis-sample-response.png) 

Оценка прогнозирования означает степень достоверности результатов прогнозирования LUIS. Оценка прогнозирования находится в диапазоне от нуля (0) до единицы (1). Пример оценки прогнозирования LUIS с высокой степенью достоверности — 0,99. Пример оценки прогнозирования LUIS с низкой достоверностью — 0,01. 

## <a name="optional---extract-entities"></a>Извлечение сущностей (необязательно)

Помимо распознавание намерений пользователя, приложения LUIS умеет возвращать сущности. Сущностями называются важные слова, которые имеют отношение к намерению. Иногда они помогают боту лучше выполнить запрос пользователя или реагировать более разумно. 

### <a name="why-use-entities"></a>Для чего нужны сущности

Сущности LUIS позволяют боту лучше понимать некоторые факты или события, которые отличаются от стандартных намерений. Это позволяет получить от пользователя дополнительную информацию, которая позволит боту точнее реагировать на действия пользователя или пропустить некоторые вопросы, в которых от пользователя запрашивается уже полученная информация. Например, в боте информации о погоде приложение LUIS может применить сущность _Location_ (Расположение) для извлечения из сообщения пользователя данных о местоположении, по которому запрошен отчет о погоде. Это позволит боту пропустить вопрос о местонахождении пользователя и сделать беседу с пользователем более осмысленной и краткой.

### <a name="prerequisites"></a>Предварительные требования

Чтобы использовать сущности в этом примере, следует создать приложение LUIS, которое включает в себя сущности. Выполните действия, описанные в разделе выше, чтобы [создать приложение LUIS](#create-a-luis-app-in-the-luis-portal), но вместо файла `reminders.json` используйте для сборки приложения LUIS файл [reminders-with-entities.json](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/nlp-with-luis). В этом файле описаны те же цели и три дополнительные сущности: Appointment (Встреча), Meeting (Собрание) и Schedule (Расписание). Эти сущности помогают LUIS определить намерение в сообщении пользователя. 

### <a name="extract-and-display-entities"></a>Извлечение и отображение сущностей
Приведенный ниже код можно (но не обязательно) добавить в этот пример приложения, чтобы извлекать и отображать сведения о сущностях, если LUIS использует их при определении намерений пользователя. 

# <a name="ctabcs"></a>[C#](#tab/cs)

Приведенную ниже вспомогательную функцию можно добавить в бот для получения сущностей из `RecognizerResult` от LUIS. Для этого потребуется библиотека `Newtonsoft.Json.Linq`, которую нужно добавить в операторы **using**. Если в ходе синтаксического анализа кода JSON, полученного от LUIS, обнаруживаются сведения о сущностях, функция Newtonsoft _DeserializeObject_ преобразует этот код JSON в динамический объект для доступа к этим сведениям о сущностях.

```cs
using Newtonsoft.Json.Linq;

private string ParseLuisForEntities(RecognizerResult recognizerResult)
{
   var result = string.Empty;

   // recognizerResult.Entities returns type JObject.
   foreach (var entity in recognizerResult.Entities)
   {
       // Parse JObject for a known entity types: Appointment, Meeting, and Schedule.
       var appointFound = JObject.Parse(entity.Value.ToString())["Appointment"];
       var meetingFound = JObject.Parse(entity.Value.ToString())["Meeting"];
       var schedFound = JObject.Parse(entity.Value.ToString())["Schedule"];

       // We will return info on the first entity found.
       if (appointFound != null)
       {
           // use JsonConvert to convert entity.Value to a dynamic object.
           dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
           if (o.Appointment[0] != null)
           {
              // Find and return the entity type and score.
              var entType = o.Appointment[0].type;
              var entScore = o.Appointment[0].score;
              result = "Entity: " + entType + ", Score: " + entScore + ".";
              
              return result;
            }
        }

        if (meetingFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Meeting[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Meeting[0].type;
                var entScore = o.Meeting[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }

        if (schedFound != null)
        {
            // use JsonConvert to convert entity.Value to a dynamic object.
            dynamic o = JsonConvert.DeserializeObject<dynamic>(entity.Value.ToString());
            if (o.Schedule[0] != null)
            {
                // Find and return the entity type and score.
                var entType = o.Schedule[0].type;
                var entScore = o.Schedule[0].score;
                result = "Entity: " + entType + ", Score: " + entScore + ".";
                
                return result;
            }
        }
    }

    // No entities were found, empty string returned.
    return result;
}
```

Обнаруженные сведения о сущностях можно отображать вместе с выявленным намерением пользователя. Чтобы отобразить эту информацию, добавьте следующие несколько строк кода в пример кода задачи _OnTurnAsync_ сразу после отображения сведений о намерении.

```cs
// See if LUIS found and used an entity to determine user intent.
var entityFound = ParseLuisForEntities(recognizerResult);

// Inform the user if LUIS used an entity.
if (entityFound.ToString() != string.Empty)
{
   await turnContext.SendActivityAsync($"==>LUIS Entity Found: {entityFound}\n");
}
else
{
   await turnContext.SendActivityAsync($"==>No LUIS Entities Found.\n");
}
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Можно добавить в бот следующий код, чтобы извлекать сведения о сущностях из результата `luisRecognizer`, полученного от LUIS. В код обработки `onTurn` в файле кода bot.js добавьте следующую строку сразу после объявления константы _topIntent_. Она собирает все полученные данные о сущностях: 

```javascript
// Since the LuisRecognizer was configured to include the raw results, get returned entity data.
var entityData = results.luisResult.entities;

```

Чтобы предоставить пользователю все полученные сведения о сущностях, добавьте следующие строки кода сразу после вызова _sendActivity_, который в этом примере кода информирует пользователей о наличии значения topIntent.

```javascript
// See if LUIS found and used an entity to determine user intent.
if (entityData.length > 0)
{
   if ((entityData[0].type == "Appointment") || (entityData[0].type == "Meeting") || (entityData[0].type == "Schedule") )
   {
      // inform user if LUIS used an entity.
      await turnContext.sendActivity(`LUIS Entity Found: Entity: ${entityData[0].entity}, Score: ${entityData[0].score}.`);
   }
}
else{
       await turnContext.sendActivity(`No LUIS Entities Found.`);
}
```

Этот код сначала проверяет, есть ли в результатах LUIS данные о сущностях, а при их наличии отображает сведения о первой обнаруженной сущности.

---

### <a name="test-bot-with-entities"></a>Тестирование бота с сущностями

1. Чтобы выполнить тестирование бота, который использует сущности, выполните этот пример локально, как описано [выше](#test-the-bot).

1. Введите в эмуляторе сообщение, предложенное ниже. 

![Тестирование входных данных для примера NLP](./media/nlp-luis-sample-message.png)

В следующем ответе бота содержится намерение Calendar_Add, получившее самую высокую оценку, а также сущность Meetings (Собрания), которую служба LUIS использовала для определения намерения пользователя.

![Тестирование ответа в примере NLP](./media/nlp-luis-sample-entity-response.png) 

Обнаружение сущностей может повысить общую производительность бота. Например, определение сущности Meetings (Собрания), как показано выше, позволяет приложению вызвать специализированную функцию для создания нового собрания в календаре пользователя.

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Использование QnA Maker для ответов на вопросы](./bot-builder-howto-qna.md)

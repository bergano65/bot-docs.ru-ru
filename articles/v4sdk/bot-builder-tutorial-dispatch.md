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
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c798c26f108458e1caeb16aa22c02c6e7c70fb61
ms.sourcegitcommit: 3cc768a8e676246d774a2b62fb9c688bbd677700
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/16/2019
ms.locfileid: "54323660"
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

Вы можете выполнить инструкции из файлов **README** для [C#](https://aka.ms/dispatch-sample-readme-cs) или [JS](https://aka.ms/dispatch-sample-readme-js), чтобы создать бот с помощью вызовов интерфейса командной строки, либо воспользоваться описанными ниже действиями для создания бота вручную через пользовательские интерфейсы Azure, LUIS и QnA Maker.

 ### <a name="create-your-bot-using-service-ui"></a>Создание бота с помощью пользовательского интерфейса службы
 
Чтобы приступить к созданию бота вручную, скачайте следующие 4 файла из репозитория GitHub [BotFramework-Samples](https://aka.ms/botdispatchgitsamples) в локальную папку: [home-automation.json](https://aka.ms/dispatch-home-automation-json), [weather.json](https://aka.ms/dispatch-weather-json), [nlp-with-dispatchDispatch.json](https://aka.ms/dispatch-dispatch-json), [QnAMaker.tsv](https://aka.ms/dispatch-qnamaker-tsv) Для этого можно, например, открыть приведенную выше ссылку на репозиторий GitHub, щелкнуть элемент **BotFramework-Samples** и выбрать действие Clone or download (Клонировать или скачать), чтобы переместить репозиторий на локальный компьютер. Обратите внимание, что эти файлы находятся не в том репозитории, где размещен указанный в предварительных требованиях пример.

### <a name="manually-create-luis-apps"></a>Создание приложений LUIS вручную

Войдите на [веб-портал LUIS](https://www.luis.ai/). В разделе _Мои приложения_ перейдите на вкладку _Import new app_ (Импорт нового приложения). Откроется следующее диалоговое окно:

![Импорт JSON-файла для LUIS](./media/tutorial-dispatch/import-new-luis-app.png)

Нажмите кнопку _Choose app file_ (Выбрать файл приложения) и выберите скачанный ранее файл home-automation.json. Оставьте пустым поле Optional Name (Имя (необязательно)). Нажмите кнопку _Готово_.

Когда служба LUIS откроет выбранное приложение для автоматизации дома, нажмите кнопку _Train_ (Обучение). В результате этого будет выполнено обучение приложения по импортированному ранее набору высказываний с помощью файла home-automation.json.

После завершения обучения нажмите кнопку _Publish_ (Публикация). Откроется следующее диалоговое окно:

![Публикация приложения LUIS](./media/tutorial-dispatch/publish-luis-app.png)

Выберите среду Production (Рабочая) и нажмите кнопку _Publish_ (Публикация).

Когда новое приложение LUIS будет опубликовано, выберите вкладку _MANAGE_ (Управление). На странице Application Information (Сведения о приложении) запишите значения `Application ID` и `Display name`. На странице Key and Endpoints (Ключ и конечные точки) запишите значения `Authoring Key` и `Region`. Эти значения вы позднее внесете в файл nlp-with-dispatch.bot.

Завершив эти действия, _обучите_ и _опубликуйте_ приложение LUIS с прогнозом погоды и приложение LUIS для диспетчеризации, повторив описанные выше шаги с локальными копиями файлов weather.json и nlp-with-dispatchDispatch.json.

### <a name="manually-create-qna-maker-app"></a>Создание приложения QnA Maker вручную

Первым шагом при настройке базы знаний службы QnA Maker является настройка службы QnA Maker в Azure. Чтобы сделать это, выполните пошаговые инструкции, приведенные на [этой странице](https://aka.ms/create-qna-maker). Теперь войдите на [веб-портал QnA Maker](https://qnamaker.ai). Перейдите к шагу 2.

![Создание QnA, шаг 2](./media/tutorial-dispatch/create-qna-step-2.png)

Выберите здесь следующие значения:
1. Учетную запись Azure AD.
1. Имя подписки Azure.
1. Имя, с которым вы создали службу QnA Maker. (Если нужная служба Azure QnA отсутствует в этом раскрывающемся списке, попробуйте обновить страницу.) 

Перейдите к шагу 3.

![Создание QnA, шаг 3](./media/tutorial-dispatch/create-qna-step-3.png)

Укажите имя для новой базы знаний QnA Maker. Для этого примера мы используем имя sample-qna.

Перейдите к шагу 4.

![Создание QnA, шаг 4](./media/tutorial-dispatch/create-qna-step-4.png)

Щелкните _+ Add File_ (+ Добавить файл) и выберите скачанный файл QnAMaker.tsv.

Здесь есть дополнительный параметр, позволяющий добавить в базу знаний личность _Chit-chat_ (Беседа), но в нашем примере он не используется.

Выберите действие _Save and train_ (Сохранить и обучить), а когда оно завершится, перейдите на вкладку _PUBLISH_ (Публикация) и опубликуйте приложение.

После публикации приложения QnA Maker откройте вкладку _SETTINGS_ (Параметры) и прокрутите эту страницу вниз до раздела Deployment Details (Сведения о развертывании). Запишите следующие значения из примера HTTP-запроса _Postman_.

```
POST /knowledgebases/<Your_Knowledgebase_Id>/generateAnswer
Host: <Your_Hostname>
Authorization: EndpointKey <Your_Endpoint_Key>
```
Эти значения вы позднее внесете в файл nlp-with-dispatch.bot.

### <a name="manually-update-your-bot-file"></a>Обновление файла .bot вручную

После создания всех приложений службы вам осталось только добавить сведения о них в файл nlp-with-dispatch.bot. Откройте этот файл из скачанного ранее примера для C# или JS. Добавьте следующие значения во все разделы с типом type: luis или type: dispatch.

```
"appId": "<Your_Recorded_App_Id>",
"authoringKey": "<Your_Recorded_Authoring_Key>",
"subscriptionKey": "<Your_Recorded_Authoring_Key>",
"version": "0.1",
"region": "<Your_Recorded_Region>",
```

В раздел с типом type: qna добавьте следующие значения:

```
"type": "qna",
"name": "sample-qna",
"id": "201",
"kbId": "<Your_Recorded_Knowledgebase_Id>",
"subscriptionKey": "<Your_Azure_Subscription_Key>", // Used when creating your QnA service.
"endpointKey": "<Your_Recorded_Endpoint_Key>",
"hostname": "<Your_Recorded_Hostname>"
```

Когда все изменения будут внесены, сохраните файл.

### <a name="test-your-bot"></a>Тестирование бота

Теперь запустите пример с помощью эмулятора. Открыв эмулятор, выберите файл nlp-with-dispatch.bot.

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Этот пример кода использует предопределенные константы именования для обозначения разделов в файле .bot. Если вы изменили имена разделов в файле _nlp-with-dispatch.bot_, найдите объявления соответствующих констант в файлах **bot.js**, **homeAutomation.js**, **qna.js** или **weather.js** и укажите в них новые имена разделов.  
```javascript
// In file bot.js
// this is the LUIS service type entry in the .bot file.
const DISPATCH_CONFIG = 'nlp-with-dispatchDispatch';

// In file homeAutomation.js
// this is the LUIS service type entry in the .bot file.
const LUIS_CONFIGURATION = 'Home Automation';

// In file qna.js
// Name of the QnA Maker service in the .bot file.
const QNA_CONFIGURATION = 'sample-qna';

// In file weather.js
// this is the LUIS service type entry in the .bot file.
const WEATHER_LUIS_CONFIGURATION = 'Weather';
```

В файле **bot.js** сведения из файла конфигурации _nlp-with-dispatch.bot_ используются для подключения бота диспетчеризации к разным службам. Каждый конструктор ищет и применяет соответствующие разделы из файла конфигурации, используя описанные выше имена разделов.

```javascript
class DispatchBot {
    constructor(conversationState, userState, botConfig) {
        //...
        this.homeAutomationDialog = new HomeAutomation(conversationState, userState, botConfig);
        this.weatherDialog = new Weather(botConfig);
        this.qnaDialog = new QnA(botConfig);

        this.conversationState = conversationState;
        this.userState = userState;

        // dispatch recognizer
        const dispatchConfig = botConfig.findServiceByNameOrId(DISPATCH_CONFIG);
        //...
```
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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В методе `onTurn` файла **bot.js** мы проверяем наличие входящих сообщений от пользователя. Если получен экземпляр типа _ActivityType.Message_, это сообщение отправляется с помощью метода бота _dispatchRecognizer_.

```javascript
if (turnContext.activity.type === ActivityTypes.Message) {
    // determine which dialog should fulfill this request
    // call the dispatch LUIS model to get results.
    const dispatchResults = await this.dispatchRecognizer.recognize(turnContext);
    const dispatchTopIntent = LuisRecognizer.topIntent(dispatchResults);
    //...
 }
```
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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Когда модель возвращает результат, он позволяет понять, какие службы лучше всего подходят для обработки этого высказывания. Код этого бота направляет запрос в соответствующую службу.

```javascript
switch (dispatchTopIntent) {
   case HOME_AUTOMATION_INTENT:
      await this.homeAutomationDialog.onTurn(turnContext);
      break;
   case WEATHER_INTENT:
      await this.weatherDialog.onTurn(turnContext);
      break;
   case QNA_INTENT:
      await this.qnaDialog.onTurn(turnContext);
      break;
   case NONE_INTENT:
      default:
      // Unknown request
       await turnContext.sendActivity(`I do not understand that.`);
       await turnContext.sendActivity(`I can help with weather forecast, turning devices on and off and answer general questions like 'hi', 'who are you' etc.`);
 }
 
 // In homeAutomation.js
 async onTurn(turnContext) {
    // make call to LUIS recognizer to get home automation intent + entities
    const homeAutoResults = await this.luisRecognizer.recognize(turnContext);
    const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
    // depending on intent, call turn on or turn off or return unknown
    switch (topHomeAutoIntent) {
       case HOME_AUTOMATION_INTENT:
          await this.handleDeviceUpdate(homeAutoResults, turnContext);
          break;
       case NONE_INTENT:
       default:
         await turnContext.sendActivity(`HomeAutomation dialog cannot fulfill this request.`);
    }
}
    
// In weather.js
async onTurn(turnContext) {
   // Call weather LUIS model.
   const weatherResults = await this.luisRecognizer.recognize(turnContext);
   const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
   // Get location entity if available.
   const locationEntity = (LOCATION_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_ENTITY][0] : undefined;
   const locationPatternAnyEntity = (LOCATION_PATTERNANY_ENTITY in weatherResults.entities) ? weatherResults.entities[LOCATION_PATTERNANY_ENTITY][0] : undefined;
   // Depending on intent, call "Turn On" or "Turn Off" or return unknown.
   switch (topWeatherIntent) {
      case GET_CONDITION_INTENT:
         await turnContext.sendActivity(`You asked for current weather condition in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case GET_FORECAST_INTENT:
         await turnContext.sendActivity(`You asked for weather forecast in Location = ` + (locationEntity || locationPatternAnyEntity));
         break;
      case NONE_INTENT:
      default:
         wait turnContext.sendActivity(`Weather dialog cannot fulfill this request.`);
   }
}
    
// In qna.js
async onTurn(turnContext) {
   // Call QnA Maker and get results.
   const qnaResult = await this.qnaRecognizer.generateAnswer(turnContext.activity.text, QNA_TOP_N, QNA_CONFIDENCE_THRESHOLD);
   if (!qnaResult || qnaResult.length === 0 || !qnaResult[0].answer) {
       await turnContext.sendActivity(`No answer found in QnA Maker KB.`);
       return;
    }
    // respond with qna result
    await turnContext.sendActivity(qnaResult[0].answer);
}
```
---

## <a name="edit-intents-to-improve-performance"></a>Изменение намерений для повышения производительности

Когда бот будет запущен, вы можете повысить его производительность, удаляя похожие или перекрывающиеся высказывания. Для примера предположим, что в приложении LUIS `Home Automation` для домашней автоматизации запросы типа "turn on the lights" (Включить свет) сопоставляются с намерением TurnOnLights, а запросы типа "Why won't my lights turn on?" (Почему не включается свет?) сопоставляются с намерением None. Во втором случае запрос передается в службу QnA Maker. При объединении приложения LUIS и службы QnA Maker с помощью диспетчеризации необходимо выполнить следующие действия.

* Удалите намерение None из исходного приложения LUIS `Home Automation` и добавьте высказывания из удаленного намерения в намерение None в приложении диспетчеризации.
* Если вы сохраните намерение "None" в исходном приложении LUIS, в бот придется добавить логику для передачи в службу QnA Maker сообщений, которые соответствуют этому намерению.

Любой из этих методов позволяет сократить число сообщений "Couldn't find an answer" (Не удалось найти ответ), которые бот будет возвращать пользователям. 

## <a name="additional-resources"></a>Дополнительные ресурсы

**Обновление или создание модели LUIS:** этот пример основан на предварительно настроенной модели LUIS. Сведения, которые вам потребуются для обновления этой модели или создания новой модели LUIS, вы найдете [здесь](https://aka.ms/create-luis-model#updating-your-cognitive-models
).

**Удаление ресурсов**. С помощью этого примера создается ряд приложений и ресурсов, которые можно удалить с помощью описанных ниже действий. Следите при этом за тем, чтобы не удалить ресурсы, от которых зависят *другие приложения или службы*. 

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

**Рекомендация**. Чтобы улучшить описанные здесь службы, изучите рекомендации по [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) и [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices).
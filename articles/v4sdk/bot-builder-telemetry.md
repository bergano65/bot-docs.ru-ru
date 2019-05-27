---
title: Добавление телеметрии в бот | Документация Майкрософт
description: Узнайте, как интегрировать новые функции телеметрии в бот.
keywords: telemetry, appinsights, monitor bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 95b56ec8e278c3d94430dc3c870803e8672fb053
ms.sourcegitcommit: 4086189a9c856fbdc832eb1a1d205e5f1b4e3acd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/16/2019
ms.locfileid: "65733329"
---
# <a name="add-telemetry-to-your-bot"></a>Добавление телеметрии в бот

[!INCLUDE[applies-to](../includes/applies-to.md)]

В версии 4.2 пакета SDK для Bot Framework теперь можно вести журнал телеметрии.  Это позволяет приложениям ботов отправлять данные о событиях во внешние службы, например Application Insights. В первом разделе рассматриваются эти методы и описываются расширенные функции телеметрии.

Этот документ описывает, как интегрировать новые функции телеметрии в бот. 

## <a name="basic-telemetry-options"></a>Основные варианты использования телеметрии

### <a name="basic-application-insights"></a>Application Insights

Для начала давайте добавим в бота основные данные телеметрии с помощью Application Insights. Дополнительные сведения о настройках см. в первых разделах руководства по [началу работы с Application Insights](https://github.com/Microsoft/ApplicationInsights-aspnetcore/wiki/Getting-Started-with-Application-Insights-for-ASP.NET-Core).   

Если вам нужна стандартная версия Application Insights без дополнительной настройки (например, без инициализаторов телеметрии), добавьте в метод `ConfigureServices()` следующий код.   Это самый простой метод инициализировать и настроить Application Insights, чтобы отслеживать запросы, внешние вызовы к другим службам и события взаимодействия между службами.

Необходимо добавить пакеты NuGet, включенные в следующий фрагмент кода.

**Startup.cs.**
```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.Bot.Builder.ApplicationInsights;
using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
 
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add Application Insights services into service collection
    services.AddApplicationInsightsTelemetry();

    // Add the standard telemetry client
    services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

    // Add ASP middleware to store the HTTP body, mapped with bot activity key, in the httpcontext.items
    // This will be picked by the TelemetryBotIdInitializer
    services.AddTransient<TelemetrySaveBodyASPMiddleware>();

    // Add telemetry initializer that will set the correlation context for all telemetry items
    services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

    // Add telemetry initializer that sets the user ID and session ID (in addition to other 
    // bot-specific properties, such as activity ID)
    services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();
    ...
}

// This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...
    app.UseBotApplicationInsights();
}
```

Затем вам нужно сохранить ключ инструментирования Application Insights в файле `appsettings.json` или переменной среды. Файл `appsettings.json` содержит метаданные о внешних службах, которые бот использует в своей работе.  Например, здесь хранятся сведения о подключении и метаданные для служб CosmosDB, Application Insights и LUIS (Распознавание речи). Ключ инструментирования можно найти на портале Azure в разделе **Обзор** (откройте на этой странице раскрывающийся список `Essentials` для службы, если он свернут). См. подробнее о [получении ключей](~/bot-service-resources-app-insights-keys.md).

Платформа найдет для вас ключ, если он правильно отформатирован. Форматирование записей `appsettings.json` должно выглядеть следующим образом:

```json
    "ApplicationInsights": {
        "InstrumentationKey": "putinstrumentationkeyhere"
    },
    "Logging": {
        "LogLevel": {
            "Default": "Warning"
        }
    }
```

См. подробнее о [добавлении Application Insights в приложение ASP.NET Core](https://docs.microsoft.com/en-us/azure/azure-monitor/app/asp-net-core-no-visualstudio). 

### <a name="customize-your-telemetry-client"></a>Настройка клиента телеметрии

Если вы хотите настроить клиент Application Insights или даже использовать другую службу, следует использовать другую конфигурацию. Скачайте пакет `Microsoft.Bot.Builder.ApplicationInsights` с помощью NuGet или используйте npm для установки `botbuilder-applicationinsights`. См. подробнее о [получении ключей Application Insights](~/bot-service-resources-app-insights-keys.md).

**Изменение конфигурации Application Insights**

Чтобы изменить конфигурацию, укажите `options` при добавлении Application Insights. Все остальное настраивается, как описано выше.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add Application Insights services into service collection
    services.AddApplicationInsightsTelemetry(options);
    ...
}
```

Объект `options` имеет тип `ApplicationInsightsServiceOptions`. См. подробнее о [доступных вариантах]().

**Применение пользовательской телеметрии**. Если вы хотите регистрировать созданные платформой Bot Framework события телеметрии в сторонней системе, создайте класс, производный от базового интерфейса `IBotTelemetryClient`, и настройте его. Затем при добавлении клиента телеметрии, как описано выше, внедрите пользовательский клиент. 

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry client.
    services.AddSingleton<IBotTelemetryClient, CustomTelemetryClient>();
    ...
}
```

### <a name="add-custom-logging-to-your-bot"></a>Добавление в бот функции настраиваемого ведения журнала

Настроив для бота поддержку новой системы ведения журнала, вы сможете добавлять в этот бот данные телеметрии.  Класс `BotTelemetryClient`(или `IBotTelemetryClient` в C#) предоставляет несколько методов для регистрации в журнале событий разных типов.  Правильный выбор типа события позволяет воспользоваться преимуществами существующих отчетов Application Insights, (если вы используете Application Insights).  В общем случае применяется тип `TraceEvent`.  Регистрируемые с помощью `TraceEvent` данные попадают в таблицу `CustomEvent` в Kusto.

Если вы используете в боте объект Dialog, каждый объект на его основе (включая запросы) теперь содержит новое свойство `TelemetryClient`.  Это экземпляр `BotTelemetryClient`, который используется для ведения журнала.  Он нужен не только для удобства. Далее в этой статье мы увидим, как `WaterfallDialogs` генерирует события при настройке этого свойства.

#### <a name="identifiers-and-custom-events"></a>Идентификаторы и пользовательские события

При регистрации событий в Application Insights создаваемые события уже содержат свойства по умолчанию, и вам не придется их заполнять.  Например, в каждом событии Custom (созданном с помощью API `TraceEvent`) содержатся свойства `user_id` и `session_id`.  Кроме них добавляются `activitiId`, `activityType` и `channelId`.

>Примечание. Эти значения не передаются в пользовательские клиенты телеметрии.

Свойство |type | Сведения
--- | --- | ---
`user_id`| `string` | [Идентификатор канала](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) + [Идентификатор источника](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [Идентификатор беседы](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [Идентификатор действия бота](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [Тип действия бота](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [Идентификатор канала действия бота](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)

## <a name="in-depth-telemetry"></a>Подробная телеметрия

В пакет SDK версии 4.4 добавлены четыре новых компонента.  Все компоненты выполняют запись в журнал, используя интерфейс `IBotTelemetryClient` (или `BotTelemetryClient` в Node.js), который можно переопределить с помощью пользовательской реализации.

- Компонент ПО промежуточного слоя Bot Framework (*TelemetryLoggerMiddleware*), который выполняет запись в журнал при получении, отправке, обновлении или удалении сообщений. Поддерживается переопределение для настройки пользовательского ведения журнала.
- Класс *LuisRecognizer*.  Поддерживается переопределение для настройки пользовательского ведения журнала двумя способами: с использованием вызова (свойства добавления или замены) или производных классов.
- Класс *QnAMaker*.  Поддерживается переопределение для настройки пользовательского ведения журнала двумя способами: с использованием вызова (свойства добавления или замены) или производных классов.

### <a name="telemetry-middleware"></a>ПО промежуточного слоя телеметрии

|C#  | JavaScript |
|:-----|:------------|
|**Microsoft.Bot.Builder.TelemetryLoggerMiddleware** | **botbuilder-core** |

#### <a name="out-of-box-usage"></a>Стандартное использование

TelemetryLoggerMiddleware — это компонент Bot Framework, который можно добавить без изменения. Это решение ведет запись в журнал для создания стандартных отчетов, поставляемых с пакетом SDK Bot Framework. 

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, TelemetryLoggerMiddleware>();

// Create the Bot Framework Adapter with error handling enabled.
services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();
```

#### <a name="adding-properties"></a>Добавление свойств
Если вы решили добавить дополнительные свойства, вы можете использовать производный класс TelemetryLoggerMiddleware.  Например, вы можете добавить свойство MyImportantProperty в событие `BotMessageReceived`.  Событие `BotMessageReceived` записывается в журнал, когда пользователь отправляет боту сообщение.  Добавить дополнительные свойства можно так:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task OnReceiveActivityAsync(
                  Activity activity,
                  CancellationToken cancellation)
    {
        // Fill in the "standard" properties for BotMessageReceived
        // and add our own property.
        var properties = FillReceiveEventProperties(activity, 
                    new Dictionary<string, string>
                    { {"MyImportantProperty", "myImportantValue" } } );
                    
        // Use TelemetryClient to log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgReceiveEvent,
                        properties);
    }
    ...
}
```

В файл Startup мы добавим новый класс:

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, MyTelemetryMiddleware>();
```

#### <a name="completely-replacing-properties--additional-events"></a>Полная замена свойств и дополнительные события

Если вы хотите полностью заменить записываемые в журнал свойства, можно использовать производный класс `TelemetryLoggerMiddleware` (например, как описано выше для расширения свойств).   Запись новых событий выполняется таким же образом.

Например, полностью заменить свойства `BotMessageSend` и отправить несколько событий можно так:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    public Task<RecognizerResult> OnLuisRecognizeAsync(
                  Activity activity,
                  string dialogId = null,
                  CancellationToken cancellation)
    {
        // Override properties for BotMsgSendEvent
        var botMsgSendProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        // Log event
        TelemetryClient.TrackEvent(
                        TelemetryLoggerConstants.BotMsgSendEvent,
                        botMsgSendProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("activityId",
                                   activity.Id);
        secondEventProperties.Add("MyImportantProperty",
                                   "myImportantValue");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
    }
    ...
}
```
Примечание. Если стандартные свойства не записываются в журнал, стандартные отчеты, поставляемые с продуктом, не будут создаваться.

#### <a name="events-logged-from-telemetry-middleware"></a>События, записанные в журнал из ПО промежуточного слоя телеметрии
[BotMessageSend](#customevent-botmessagesend)
[BotMessageReceived](#customevent-botmessagereceived)
[BotMessageUpdate](#customevent-botmessageupdate)
[BotMessageDelete](#customevent-botmessagedelete)

### <a name="telemetry-support-luis"></a>LUIS: поддержка телеметрии 

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.Luis** | **botbuilder-ai** |

#### <a name="out-of-box-usage"></a>Стандартное использование
LuisRecognizer — это существующий компонент Bot Framework. Телеметрию можно включить, передав интерфейс IBotTelemetryClient через `luisOptions`.  При необходимости можно переопределить свойства по умолчанию, записываемые в журнал, чтобы записывать новые события.

Для этого во время создания `luisOptions` нужно передать объект `IBotTelemetryClient`.

```csharp
var luisOptions = new LuisPredictionOptions(
      ...
      telemetryClient,
      false); // Log personal information flag. Defaults to false.

var client = new LuisRecognizer(luisApp, luisOptions);
```

#### <a name="adding-properties"></a>Добавление свойств
Если вы решили добавить дополнительные свойства, вы можете использовать производный класс `LuisRecognizer`.  Например, вы можете добавить свойство MyImportantProperty в событие `LuisResult`.  Событие `LuisResult` записывается в журнал, когда выполняется вызов прогнозирования LUIS.  Добавить дополнительные свойства можно так:

```csharp
class MyLuisRecognizer : LuisRecognizer 
{
   ...
   override protected Task OnRecognizerResultAsync(
           RecognizerResult recognizerResult,
           ITurnContext turnContext,
           Dictionary<string, string> properties = null,
           CancellationToken cancellationToken = default(CancellationToken))
   {
       var luisEventProperties = FillLuisEventProperties(result, 
               new Dictionary<string, string>
               { {"MyImportantProperty", "myImportantValue" } } );
        
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResultEvent,
                        luisEventProperties);
        ..
   }    
   ...
}
```

#### <a name="add-properties-per-invocation"></a>Добавление свойств с использованием вызова
Иногда нужно добавить дополнительные свойства во время вызова:
```csharp
var additionalProperties = new Dictionary<string, string>
{
   { "dialogId", "myDialogId" },
   { "conversationInfo", "myConversationInfo" },
};

var result = await recognizer.RecognizeAsync(turnContext,
     additionalProperties,
     CancellationToken.None).ConfigureAwait(false);
```

#### <a name="completely-replacing-properties--additional-events"></a>Полная замена свойств и дополнительные события
Если вы хотите полностью заменить записываемые в журнал свойства, можно использовать производный класс `LuisRecognizer` (например, как описано выше для расширения свойств).   Запись новых событий выполняется таким же образом.

Например, полностью заменить свойства `LuisResult` и отправить несколько событий можно так:

```csharp
class MyLuisRecognizer : LuisRecognizer
{
    ...
    override protected Task OnRecognizerResultAsync(
             RecognizerResult recognizerResult,
             ITurnContext turnContext,
             Dictionary<string, string> properties = null,
             CancellationToken cancellationToken = default(CancellationToken))
    {
        // Override properties for LuisResult event
        var luisProperties = new Dictionary<string, string>();
        properties.Add("MyImportantProperty", "myImportantValue");
        
        // Log event
        TelemetryClient.TrackEvent(
                        LuisTelemetryConstants.LuisResult,
                        luisProperties);
                        
        // Create second event.
        var secondEventProperties = new Dictionary<string, string>();
        secondEventProperties.Add("MyImportantProperty2",
                                   "myImportantValue2");
        TelemetryClient.TrackEvent(
                        "MySecondEvent",
                        secondEventProperties);
        ...
    }
    ...
}
```
Примечание. Если стандартные свойства не записываются в журнал, стандартные отчеты Application Insights, поставляемые с продуктом, не будут создаваться.

#### <a name="events-logged-from-telemetryluisrecognizer"></a>События, записанные в журнал из TelemetryLuisRecognizer
[LuisResult](#customevent-luisevent)

### <a name="telemetry-qna-recognizer"></a>Распознаватель телеметрии QnA

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.QnA** | **botbuilder-ai** |


#### <a name="out-of-box-usage"></a>Стандартное использование
Класс QnAMaker — это существующий компонент Bot Framework, который добавляет два дополнительных параметра конструктора. Это решение ведет запись в журнал для создания стандартных отчетов, поставляемые с пакетом SDK Bot Framework. Новые ссылки `telemetryClient` указывают на интерфейс `IBotTelemetryClient`, который выполняет запись в журнал.  

```csharp
var qna = new QnAMaker(endpoint, options, client, 
                       telemetryClient: telemetryClient,
                       logPersonalInformation: true);
```
#### <a name="adding-properties"></a>Добавление свойств 
Если вы хотите добавить дополнительные свойства, вы можете сделать это двумя способами: во время вызова QnA для получения ответов или путем наследования от класса `QnAMaker`.  

Наследование от класса `QnAMaker` можно выполнить, как показано ниже.  Например, вы можете добавить свойство MyImportantProperty в событие `QnAMessage`.  Событие `QnAMessage` записывается в журнал, когда выполняется вызов QnA `GetAnswers`.  Кроме того, мы записываем в журнал второе событие MySecondEvent.

```csharp
class MyQnAMaker : QnAMaker 
{
   ...
   protected override Task OnQnaResultsAsync(
                 QueryResult[] queryResults, 
                 ITurnContext turnContext, 
                 Dictionary<string, string> telemetryProperties = null, 
                 Dictionary<string, double> telemetryMetrics = null, 
                 CancellationToken cancellationToken = default(CancellationToken))
   {
            var eventData = await FillQnAEventAsync(queryResults, turnContext, telemetryProperties, telemetryMetrics, cancellationToken).ConfigureAwait(false);

            // Add my property
            eventData.Properties.Add("MyImportantProperty", "myImportantValue");

            // Log QnaMessage event
            TelemetryClient.TrackEvent(
                            QnATelemetryConstants.QnaMsgEvent,
                            eventData.Properties,
                            eventData.Metrics
                            );

            // Create second event.
            var secondEventProperties = new Dictionary<string, string>();
            secondEventProperties.Add("MyImportantProperty2",
                                       "myImportantValue2");
            TelemetryClient.TrackEvent(
                            "MySecondEvent",
                            secondEventProperties);       }    
    ...
}
```

#### <a name="adding-properties-during-getanswersasync"></a>Добавление свойств во время выполнения GetAnswersAsync
Если у вас есть свойства, которые вы хотите добавить во время выполнения, метод `GetAnswersAsync` может предоставить свойства и метрики для добавления в событие.

Например, добавить `dialogId` в событие можно так:
```csharp
var telemetryProperties = new Dictionary<string, string>
{
   { "dialogId", myDialogId },
};

var results = await qna.GetAnswersAsync(context, opts, telemetryProperties);
```
Класс `QnaMaker` позволяет переопределять свойства, включая свойства PersonalInfomation.

#### <a name="completely-replacing-properties--additional-events"></a>Полная замена свойств и дополнительные события
Если вы хотите полностью заменить записываемые в журнал свойства, можно использовать производный класс `TelemetryQnAMaker` (например, как описано выше для расширения свойств).   Запись новых событий выполняется таким же образом.

Например, полностью заменить свойства `QnAMessage` можно так:

```csharp
class MyLuisRecognizer : TelemetryQnAMaker
{
    ...
    protected override Task OnQnaResultsAsync(
         QueryResult[] queryResults, 
         ITurnContext turnContext, 
         Dictionary<string, string> telemetryProperties = null, 
         Dictionary<string, double> telemetryMetrics = null, 
         CancellationToken cancellationToken = default(CancellationToken))
    {
        // Add properties from GetAnswersAsync
        var properties = telemetryProperties ?? new Dictionary<string, string>();
        // GetAnswerAsync properties overrides - don't add if already present.
        properties.TryAdd("MyImportantProperty", "myImportantValue");

        // Log event
        TelemetryClient.TrackEvent(
                           QnATelemetryConstants.QnaMsgEvent,
                            properties);
    }
    ...
}
```
Примечание. Если стандартные свойства не записываются в журнал, стандартные отчеты, поставляемые с продуктом, не будут создаваться.

#### <a name="events-logged-from-telemetryluisrecognizer"></a>События, записанные в журнал из TelemetryLuisRecognizer
[QnAMessage](#customevent-qnamessage)


## <a name="waterfalldialog-events"></a>События WaterfallDialog

События можно не только добавлять, но и создавать с помощью объекта `WaterfallDialog` в пакете SDK. В следующем разделе описаны события, созданные на платформе Bot Framework. Чтобы сохранять эти события, установите свойству `TelemetryClient` значение `WaterfallDialog`.

Ниже показано, как изменить пример (CoreBot), который использует `WaterfallDialog`, для регистрации событий телеметрии.  CoreBot использует общий шаблон, в котором `WaterfallDialog` помещается в `ComponentDialog` (`GreetingDialog`).

```csharp
// IBotTelemetryClient is direct injected into our Bot
public CoreBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
...

// The IBotTelemetryClient passed to the GreetingDialog
...
Dialogs = new DialogSet(_dialogStateAccessor);
Dialogs.Add(new GreetingDialog(_greetingStateAccessor, telemetryClient));
...

// The IBotTelemetryClient to the WaterfallDialog
...
AddDialog(new WaterfallDialog(ProfileDialog, waterfallSteps) { TelemetryClient = telemetryClient });
...

```

Когда для `WaterfallDialog` будет настроено `IBotTelemetryClient`, выполняется процесс регистрации событий.

## <a name="events-generated-by-the-bot-framework-service"></a>События, создаваемые службой Bot Framework

В дополнение к `WaterfallDialog`, который создает события в коде бота, служба канала Bot Framework также заносит в журнал данные о событиях.  Это помогает диагностировать проблемы с работой каналов или общие сбои бота.

### <a name="customevent-activity"></a>CustomEvent: "Activity"
**Откуда регистрируется:** Служба канала. Регистрируется службой канала при получении сообщения.

### <a name="exception-bot-errors"></a>Исключение: "Bot Errors"
**Откуда регистрируется:** Служба канала. Регистрируется каналом, когда вызов к боту возвращает ответ с кодом HTTP, отличным от 2XX.

## <a name="all-events-generated"></a>Все созданные события

### <a name="customevent-waterfallstart"></a>CustomEvent: "WaterfallStart" 

В начале WaterfallDialog регистрируется событие `WaterfallStart`.

- `user_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `session_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Это идентификатор dialogId в строковом формате, переданный в каскад.  Его можно рассматривать как тип каскада)
- `customDimensions.InstanceID` (уникальное значение для каждого экземпляра диалога)

### <a name="customevent-waterfallstep"></a>CustomEvent: "WaterfallStep" 

Регистрирует отдельные шаги из каскадного диалога.

- `user_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `session_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Это идентификатор dialogId в строковом формате, переданный в каскад.  Его можно рассматривать как тип каскада)
- `customDimensions.StepName` (содержит имя метода или `StepXofY`, если это лямбда-выражение)
- `customDimensions.InstanceID` (уникальное значение для каждого экземпляра диалога)

### <a name="customevent-waterfalldialogcomplete"></a>CustomEvent: "WaterfallDialogComplete"

Регистрирует завершение каскадного диалога.

- `user_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `session_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Это идентификатор dialogId в строковом формате, переданный в каскад.  Его можно рассматривать как тип каскада)
- `customDimensions.InstanceID` (уникальное значение для каждого экземпляра диалога)

### <a name="customevent-waterfalldialogcancel"></a>CustomEvent: "WaterfallDialogCancel" 

Регистрирует отмену каскадного диалога

- `user_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `session_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Это идентификатор dialogId в строковом формате, переданный в каскад.  Его можно рассматривать как тип каскада)
- `customDimensions.StepName` (содержит имя метода или `StepXofY`, если это лямбда-выражение)
- `customDimensions.InstanceID` (уникальное значение для каждого экземпляра диалога)

### <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
Запись в журнал выполняется, когда бот получает новое сообщение от пользователя.

Если переопределение не выполняется, это событие записывается в журнал из `Microsoft.Bot.Builder.TelemetryLoggerMiddleware` с помощью метода `Microsoft.Bot.Builder.IBotTelemetry.TrackEvent()`.

- Идентификатор сеанса  
  - Если вы используете Application Insights, запись в журнал выполняется из `TelemetryBotIdInitializer` в качестве идентификатора **сеанса** (*Temeletry.Context.Session.Id*) для Application Insights.  
  - Соответствует [идентификатору беседы](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `session_id`.

- Идентификатор пользователя
  - Если вы используете Application Insights, запись в журнал выполняется из `TelemetryBotIdInitializer` в качестве идентификатора **пользователя** (*Temeletry.Context.User.Id*) для Application Insights.  
  - Значение этого свойства представляет собой объединенные значения [идентификатора канала](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) и [идентификатора пользователя](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `user_id`.

- Идентификатор действия 
  - Если вы используете Application Insights, запись в журнал выполняется из `TelemetryBotIdInitializer` в качестве свойства, добавляемого в событие.
  - Соответствует [идентификатору действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#Id), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `activityId`.

- Идентификатор канала
  - Если вы используете Application Insights, запись в журнал выполняется из `TelemetryBotIdInitializer` в качестве свойства, добавляемого в событие.  
  - Соответствует [идентификатору канала](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `channelId`.

- ActivityType 
  - Если вы используете Application Insights, запись в журнал выполняется из `TelemetryBotIdInitializer` в качестве свойства, добавляемого в событие.  
  - Соответствует [типу действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#type), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `activityType`.

- текст
  - Запись в журнал выполняется **при необходимости**, если свойству `logPersonalInformation` задано значение `true`.
  - Соответствует [способу описания действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#text), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `text`.

- Speak

  - Запись в журнал выполняется **при необходимости**, если свойству `logPersonalInformation` задано значение `true`.
  - Соответствует [способу озвучивания действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#speak), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `speak`.

  - 

- FromId
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromId`.

- FromName
  - Запись в журнал выполняется **при необходимости**, если свойству `logPersonalInformation` задано значение `true`.
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

- RecipientId
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

- RecipientName
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

- ConversationId
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

- ConversationName
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

- Языковой стандарт
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

### <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
**Откуда регистрируется:** TelemetryLoggerMiddleware 

Регистрируется, когда бот отправляет сообщение.

- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- SessionID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ReplyToID
- RecipientId
- ConversationName
- Языковой стандарт
- RecipientName (необязательно для персональных данных)
- Text (необязательно для персональных данных)
- Speak (необязательно для персональных данных)


### <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
**Откуда регистрируется:** TelemetryLoggerMiddleware. Регистрируется, когда бот обновляет сообщение (редкий случай).
- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- SessionID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName
- Языковой стандарт
- Text (необязательно для персональных данных)


### <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**Откуда регистрируется:** TelemetryLoggerMiddleware. Регистрируется, когда бот удаляет сообщение (редкий случай).
- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- SessionID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName

### <a name="customevent-luisevent"></a>CustomEvent: LuisEvent
**Откуда регистрируется:** LuisRecognizer

Регистрирует результаты, полученные от службы LUIS.

- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- SessionID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ApplicationId
- Намерение
- IntentScore
- Intent2 
- IntentScore2 
- FromId
- SentimentLabel
- SentimentScore
- Сущности (в виде JSON)
- Question (необязательно для персональных данных)

## <a name="customevent-qnamessage"></a>CustomEvent: QnAMessage
**Откуда регистрируется:** QnA Maker

Регистрирует результаты, полученные от службы QnA Maker.

- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- SessionID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Username (необязательно для персональных данных)
- Question (необязательно для персональных данных)
- MatchedQuestion
- QuestionId
- Ответ
- Оценка
- ArticleFound

## <a name="querying-the-data"></a>Запрос данных
При использовании Application Insights сопоставляются все данные (даже от разных служб).  Вы можете выполнить успешный запрос и просмотреть все связанные события для этого запроса.  
Следующие запросы возвращают последние запросы:
```sql
requests 
| where timestamp > ago(3d) 
| where resultCode == 200
| order by timestamp desc
| project timestamp, operation_Id, appName
| limit 10
```

Выберите несколько значений `operation_Id` из результатов первого запроса и получите дополнительные сведения.

```sql
let my_operation_id = "<OPERATION_ID>";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};
union_all
    | order by timestamp asc
    | project itemType, name, performanceBucket
```

Так вы получите хронологические сведения о выполнении одного запроса, где каждому вызову присвоен определенный контейнер.
![Пример вызова](media/performance_query.png)

> Примечание. Метка времени `customEvent` (действие) располагается не по порядку, так как эти события регистрируются асинхронно.

## <a name="create-a-dashboard"></a>Создание панели мониторинга

Для проверки проще всего создать панель мониторинга на [странице развертывания шаблона](https://portal.azure.com/#create/Microsoft.Template) на портале Azure.  
- Выберите ["Создать собственный шаблон в редакторе"].
- Скопируйте и вставьте любой из этих JSON-файлов, которые предоставлены специально для помощи при создании панели мониторинга:
  - [Панель мониторинга работоспособности системы](https://aka.ms/system-health-appinsights)
  - [Панель мониторинга работоспособности беседы](https://aka.ms/conversation-health-appinsights)
- Щелкните "Сохранить".
- Заполните `Basics`: 
   - Подписка: <your test subscription>
   - группу ресурсов <a test resource group>;
   - Расположение: <such as West US>
- Заполните `Settings`:
   - Имя компонента Insights <например, `core672so2hw`>
   - Группа ресурсов компонента Insights <например, `core67`>
   - Имя панели мониторинга <например, `'ConversationHealth'` или `SystemHealth`>
- Щелкните `I agree to the terms and conditions stated above`.
- Щелкните `Purchase`.
- Проверка
   - Щелкните [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups)
   - Выберите группу ресурсов (например, `core67`).
   - Если вы не видите здесь новый ресурс, проверьте раздел "Развертывания" и наличие ошибок в них.
   - Ошибки здесь обычно выглядят примерно так:
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template/#parameters for usage details.'\"\r\n }\r\n}"}]}
```

Чтобы просмотреть данные, перейдите на портал Azure. Щелкните слева **панели мониторинга**, затем выберите созданную ранее панель мониторинга из раскрывающегося списка.

## <a name="additional-resources"></a>Дополнительные ресурсы
Вы можете изучить эти примеры, в которых реализована телеметрия.
- C#
  - [LUIS с AppInsights](https://aka.ms/luis-with-appinsights-cs)
  - [QnA с AppInsights](https://aka.ms/qna-with-appinsights-cs)
- JS
  - [LUIS с AppInsights](https://aka.ms/luis-with-appinsights-js)
  - [QnA с AppInsights](https://aka.ms/qna-with-appinsights-js)


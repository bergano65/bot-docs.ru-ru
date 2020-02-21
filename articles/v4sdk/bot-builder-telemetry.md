---
title: Добавление телеметрии к возможностям бота (служба Azure Bot)
description: Узнайте, как интегрировать новые функции телеметрии в бот.
keywords: telemetry, appinsights, monitor bot
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e0064980a7b4cb1f5c37485550aa56a338209676
ms.sourcegitcommit: 9f8fe22e0ed192d4b929bfb7afa90b8e885672f1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/12/2020
ms.locfileid: "77173042"
---
# <a name="add-telemetry-to-your-bot"></a>Добавление телеметрии в бот

[!INCLUDE[applies-to](../includes/applies-to.md)]


В версии 4.2 пакета SDK для Bot Framework теперь можно вести журнал телеметрии.  Так приложения ботов смогут отправлять данные о событиях в службы телеметрии, например [Application Insights](https://aka.ms/appinsights-overview). Данные телеметрии включают полезные сведения о боте (например о том, какие функции используются чаще всего), помогают обнаруживать нежелательное поведение и предоставляют сведения о доступности, производительности и использовании.

***Примечание. В версии 4.6 стандартный метод реализации телеметрии в ботах был обновлен для обеспечения правильной записи телеметрии с пользовательским адаптером. Эта статья была обновлена с учетом этого обновления метода. Изменения сохраняют обратную совместимость, и боты с прежним методом будут и далее правильно вести журнал телеметрии.***


В этой статье приведены инструкции по реализации телеметрии в боте с помощью Application Insights:

* создание кода для включения телеметрии в боте и подключения к Application Insights;

* включение телеметрии в [диалогах](bot-builder-concept-dialog.md) бота;

* настройка телеметрии для сбора данных об использовании из других служб, например [LUIS](bot-builder-howto-v4-luis.md) и [QnA Maker.](bot-builder-howto-qna.md);

* Отображение данных телеметрии в Application Insights

## <a name="prerequisites"></a>Предварительные требования

* [Пример кода CoreBot](https://aka.ms/cs-core-sample).

* [Пример кода Application Insights](https://aka.ms/csharp-corebot-app-insights-sample).

* Подписка на [Microsoft Azure](https://portal.azure.com/).

* [Ключ Application Insights](../bot-service-resources-app-insights-keys.md).

* Опыт работы с [Application Insights](https://aka.ms/appinsights-overview).

> [!NOTE]
> [Пример кода Application Insights](https://aka.ms/csharp-corebot-app-insights-sample) основан на [примере кода CoreBot](https://aka.ms/cs-core-sample). В этой статье показано, как изменить пример кода CoreBot для включения телеметрии. Выполнив эти инструкции в Visual Studio, вы получите пример кода Application Insights.

## <a name="wiring-up-telemetry-in-your-bot"></a>Включение телеметрии в боте

Мы начнем с [примера приложения CoreBot](https://aka.ms/cs-core-sample), в который добавим код для интеграции телеметрии в любом боте. Так Application Insights сможет отслеживать запросы.

1. Откройте [пример кода CoreBot](https://aka.ms/cs-core-sample) в Visual Studio.

2. Добавление пакета NuGet `Microsoft.Bot.Builder.Integration.ApplicationInsights.Core `. Подробные сведения об использовании NuGet см. в руководстве по [установке пакетов и управлении ими в Visual Studio](https://aka.ms/install-manage-packages-vs).


3. Добавьте следующие инструкции в `Startup.cs`:
    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.Bot.Builder.ApplicationInsights;
    using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    ```

    Примечание. Выполняя эти инструкции, чтобы обновить пример кода CoreBot, вы заметите, что инструкция using для `Microsoft.Bot.Builder.Integration.AspNet.Core` уже используется в коде CoreBot.

4. Включите следующий код в метод `ConfigureServices()` в `Startup.cs`. Так службы телеметрии станут доступными для бота путем [внедрения зависимостей](https://aka.ms/asp.net-core-dependency-interjection):
    ```csharp
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        ...
            // Create the Bot Framework Adapter with error handling enabled.
            services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();

            // Add Application Insights services into service collection
            services.AddApplicationInsightsTelemetry();

            // Create the telemetry client.
            services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

            // Add telemetry initializer that will set the correlation context for all telemetry items.
            services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

            // Add telemetry initializer that sets the user ID and session ID (in addition to other bot-specific properties such as activity ID)
            services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();

            // Create the telemetry middleware to initialize telemetry gathering
            services.AddSingleton<TelemetryInitializerMiddleware>();

            // Create the telemetry middleware (used by the telemetry initializer) to track conversation events
            services.AddSingleton<TelemetryLoggerMiddleware>();
        ...
    }
    ```

    Примечание. Выполняя эти инструкции, чтобы обновить пример кода CoreBot, вы заметите, что `services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();` уже существует. 

5. Настройте адаптер на использование кода ПО промежуточного слоя, который был добавлен в метод `ConfigureServices()`. Это можно сделать в `AdapterWithErrorHandler.cs` с помощью параметра TelemetryInitializerMiddleware telemetryInitializerMiddleware в списке параметров конструкторов и инструкции `Use(telemetryInitializerMiddleware);` в конструкторе, как показано здесь:
    ```csharp
        public AdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger, TelemetryInitializerMiddleware telemetryInitializerMiddleware, ConversationState conversationState = null)
            : base(configuration, logger)
    {
        ...
        Use(telemetryInitializerMiddleware);
    }
    ```

6. Кроме того, необходимо добавить `Microsoft.Bot.Builder.Integration.ApplicationInsights.Core` в список инструкций using в `AdapterWithErrorHandler.cs`.

7. Добавьте ключ инструментирования Application Insights в файл `appsettings.json`. Файл `appsettings.json` содержит метаданные о внешних службах, которые использует бот во время работы. Например, здесь хранятся сведения о подключении и метаданные для служб CosmosDB, Application Insights и LUIS (Распознавание речи). Формат добавляемого в файл `appsettings.json` ключа должен быть таким:

    ```json
    {
        "MicrosoftAppId": "",
        "MicrosoftAppPassword": "",
        "ApplicationInsights": {
            "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        }
    }
    ```
    Примечание. Сведения о _ключе инструментирования Application_ см. в статье о [ключах Application Insights](../bot-service-resources-app-insights-keys.md).

На этом этапе вы уже включили телеметрию с помощью Application Insights.  Вы можете запустить бота локально с помощью эмулятора бота, а затем перейти к Application Insights, чтобы просмотреть журнал, в который регистрируются сведения о времени отклика, а также общей работоспособности приложения и выполнении. 

## <a name="enabling--disabling-activity-event-and-personal-information-logging"></a>Включение и отключение ведения журнала событий действий и персональных данных

### <a name="enabling-or-disabling-activity-logging"></a>Включение или отключение ведения журнала действий

По умолчанию `TelemetryInitializerMiddleware` будет использовать `TelemetryLoggerMiddleware` для регистрации данных телеметрии, когда бот отправляет или получает действия. Ведение журнала действий создает пользовательские журналы событий в ресурсе Application Insights.  При желании вы можете отключить ведение журнала событий действий, установив значение False для `logActivityTelemetry` в `TelemetryInitializerMiddleware` при его регистрации в **Startup.cs**.

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry initializer middleware
    services.AddSingleton<TelemetryInitializerMiddleware>(sp =>
            {
                var httpContextAccessor = sp.GetService<IHttpContextAccessor>();
                var loggerMiddleware = sp.GetService<TelemetryLoggerMiddleware>();
                return new TelemetryInitializerMiddleware(httpContextAccessor, loggerMiddleware, logActivityTelemetry: false);
            });
    ...
}
```

Кроме того, для добавления `IHttpContextAccessor` потребуется ссылка на библиотеку Microsoft HTTP ASPNetCore, которую можно добавить с помощью такой инструкции using:

```cs
using Microsoft.AspNetCore.Http; 
```

### <a name="enable-or-disable-logging-personal-information"></a>Включение или отключение сохранения персональных данных в журнал

По умолчанию, если включено ведение журнала действий, некоторые свойства входящих или исходящих действий исключаются из журнала, так как они могут содержать персональные данные, например имя пользователя и текст действия. Вы можете включить сохранение этих свойств в журнал, внеся следующие изменения в **Startup.cs** при регистрации `TelemetryLoggerMiddleware`.

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry initializer middleware
    services.AddSingleton<TelemetryLoggerMiddleware>(sp =>
            {
                var telemetryClient = sp.GetService<IBotTelemetryClient>();
                return new TelemetryLoggerMiddleware(telemetryClient, logPersonalInformation: true);
            });
    ...
}
```

Теперь нам нужно включить функцию телеметрии в диалоги. Так вы сможете получать дополнительные сведения о запущенных диалогах, а также собирать статистику по каждому из них.

## <a name="enabling-telemetry-in-your-bots-dialogs"></a>Включение телеметрии в диалогах бота

При добавлении нового диалога в любой класс ComponentDialog он наследует Microsoft.Bot.Builder.IBotTelemetryClient из родительского диалога.  Например, в примере приложения CoreBot все диалоги добавляются в класс MainDialog, который имеет тип ComponentDialog.  После установки свойства TelemetryClient в MainDialog все добавленные в него диалоги будут автоматически наследовать от него значение telemetryClient, и вам не придется явным образом указывать его при добавлении диалогов.
 
 Чтобы обновить код CoreBot, сделайте следующее:

1.  Обновите список параметров конструктора в `MainDialog.cs`, включив в него параметр `IBotTelemetryClient`, а затем в MainDialog установите для свойства TelemetryClient это же значение, как показано в следующем фрагменте кода:

    ```csharp
    public MainDialog(IConfiguration configuration, ILogger<MainDialog> logger, IBotTelemetryClient telemetryClient)
        : base(nameof(MainDialog))
    {
        // Set the telemetry client for this and all child dialogs.
        this.TelemetryClient = telemetryClient;
        ...
    }
    ```

> [!TIP] 
> Если у вас возникли проблемы при изменении примера кода CoreBot, см. [пример кода Application Insights](https://aka.ms/csharp-corebot-app-insights-sample).

Мы добавили телеметрию в диалоги бота. При его запуске вы увидите сведения, регистрируемые в журнале Application Insights. Но если вы используете такие интегрированные технологии, как LUIS и QnA Maker, вам также нужно добавить `TelemetryClient` в этот код.


## <a name="enabling-telemetry-to-capture-usage-data-from-other-services-like-luis-and-qna-maker"></a>Настройка телеметрии для сбора данных об использовании из других служб, например LUIS и QnA Maker

Теперь вы реализуете функции телеметрии в службе LUIS. Служба LUIS имеет встроенную функцию ведения журнала телеметрии, поэтому настроить получение данных телеметрии из LUIS очень просто.  Если вы хотите включить телеметрию в боте с поддержкой QnA Maker, выполните инструкции из [этой статьи](bot-builder-telemetry-QnAMaker.md).

1. Параметр _`IBotTelemetryClient telemetryClient`_ является обязательным в конструкторе `FlightBookingRecognizer` в `FlightBookingRecognizer.cs`:

    ```cs
    public FlightBookingRecognizer(IConfiguration configuration, IBotTelemetryClient telemetryClient)
    ```

2. Далее потребуется включить `telemetryClient` при создании `LuisRecognizer` в конструкторе `FlightBookingRecognizer`. Для этого добавьте `telemetryClient` в качестве нового класса _LuisPredictionOption_:

    ```cs
    if (luisIsConfigured)
    {
        var luisApplication = new LuisApplication(
            configuration["LuisAppId"],
            configuration["LuisAPIKey"],
            "https://" + configuration["LuisAPIHostName"]);

        // Set the recognizer options depending on which endpoint version you want to use.
        // More details can be found in https://docs.microsoft.com/en-gb/azure/cognitive-services/luis/luis-migration-api-v3
        var recognizerOptions = new LuisRecognizerOptionsV3(luisApplication)
        {
            TelemetryClient = telemetryClient,
        };
        _recognizer = new LuisRecognizer(recognizerOptions);
    }
    ```

3. Вам понадобится собственная служба LUIS. 

Теперь у вас есть функциональный бот, который записывает данные телеметрии в Application Insights. Запустить бота локально можно с помощью [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme). Вы не увидите изменений в поведении бота, но данные будут регистрироваться в Application Insights. Отправьте боту несколько сообщений. В следующем разделе описано, как просматривать результаты телеметрии в Application Insights.

Сведения о тестировании и отладке бота см. в таких статьях:

 * [Отладка бота](../bot-service-debug-bot.md)
 * [Рекомендации по тестированию и отладке](bot-builder-testing-debugging.md)
 * [Отладка с помощью эмулятора](../bot-service-debug-emulator.md)


## <a name="visualizing-your-telemetry-data-in-application-insights"></a>Отображение данных телеметрии в Application Insights
Application Insights отслеживает доступность, производительность и использование бота независимо от размещения (в облачной или локальной среде). Она использует мощную платформу анализа данных в Azure Monitor, чтобы предоставлять подробные аналитические сведения о работе приложения и диагностировать ошибки еще до того, как пользователи сообщат о них. Данные телеметрии, собираемые Application Insights, можно просматривать несколькими способами: с помощью запросов и панели мониторинга. 

### <a name="querying-your-telemetry-data-in-application-insights-using-kusto-queries"></a>Получение данных телеметрии в Application Insights с помощью запросов Kusto
В этом разделе описано, как использовать запросы журналов в Application Insights с помощью двух запросов, а также предоставлены ссылки на другую документацию с дополнительными сведениями.

Чтобы запросить данные:

1. Перейдите на [портал Azure](https://portal.azure.com).
2. Перейдите к Application Insights. Для этого щелкните **Монитор > Приложения** и выполните поиск. 
3. В Application Insights щелкните _Журналы (Analytics)_ на панели навигации.

    ![Журналы (Analytics)](media/AppInsights-LogView.png)

4. Откроется окно "Запрос".  Введите следующий запрос и нажмите _Запуск_:

    ```sql
    customEvents
    | where name=="WaterfallStart"
    | extend DialogId = customDimensions['DialogId']
    | extend InstanceId = tostring(customDimensions['InstanceId'])
    | join kind=leftouter (customEvents | where name=="WaterfallComplete" | extend InstanceId = tostring(customDimensions['InstanceId'])) on InstanceId    
    | summarize starts=countif(name=='WaterfallStart'), completes=countif(name1=='WaterfallComplete') by bin(timestamp, 1d), tostring(DialogId)
    | project Percentage=max_of(0.0, completes * 1.0 / starts), timestamp, tostring(DialogId) 
    | render timechart
    ```
5. Отобразится процент каскадных диалоговых окон, которые выполнялись до завершения.

    ![Журналы (Analytics)](media/AppInsights-Query-PercentCompleteDialog.png)


> [!TIP]
> Любой запрос можно закрепить на панели мониторинга Application Insights, нажав на кнопку в верхней правой части колонки **Журналы (Analytics)** . Просто выберите нужную панель мониторинга, и вы сможете выбрать этот запрос при ее следующем открытии.


## <a name="the-application-insights-dashboard"></a>Панель мониторинга Application Insights

Когда вы создаете ресурс Application Insights в Azure, автоматически создается и связанная панель мониторинга.  Эту панель мониторинга можно просмотреть, нажав кнопку **Панель мониторинга приложений** в верхней части колонки Application Insights. 

![Ссылка "Панель мониторинга приложений"](media/Application-Dashboard-Link.png)


Просмотреть данные можно также на портале Azure. Щелкните слева **Панель мониторинга** и выберите нужную панель мониторинга в раскрывающемся списке.

Здесь вы увидите некоторые стандартные сведения о производительности бота и дополнительные запросы, которые вы закрепили на панели мониторинга.



## <a name="additional-information"></a>Дополнительные сведения

* [Добавление данных телеметрии в бота QnAMaker](bot-builder-telemetry-qnamaker.md)

* [Что такое Azure Application Insights?](https://aka.ms/appinsights-overview)

* [Поиск в Application Insights](https://aka.ms/search-in-application-insights)

* [Создание настраиваемых панелей мониторинга ключевых показателей эффективности с помощью Azure Application Insights](https://aka.ms/custom-kpi-dashboards-application-insights)


<!--
The easiest way to test is by creating a dashboard using [Azure portal's template deployment page](https://portal.azure.com/#create/Microsoft.Template).
- Click ["Build your own template in the editor"]
- Copy and paste either one of these .json file that is provided to help you create the dashboard:
  - [System Health Dashboard](https://aka.ms/system-health-appinsights)
  - [Conversation Health Dashboard](https://aka.ms/conversation-health-appinsights)
- Click "Save"
- Populate `Basics`: 
   - Subscription: <your test subscription>
   - Resource group: <a test resource group>
   - Location: <such as West US>
- Populate `Settings`:
   - Insights Component Name: <like `core672so2hw`>
   - Insights Component Resource Group: <like `core67`>
   - Dashboard Name:  <like `'ConversationHealth'` or `SystemHealth`>
- Click `I agree to the terms and conditions stated above`
- Click `Purchase`
- Validate
   - Click on [`Resource Groups`](https://ms.portal.azure.com/#blade/HubsExtension/Resources/resourceType/Microsoft.Resources%2Fsubscriptions%2FresourceGroups)
   - Select your Resource Group from above (like `core67`).
   - If you don't see a new Resource, then look at "Deployments" and see if any have failed.
   - Here's what you typically see for failures:
     
```json
{"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-debug-deployment for usage details.","details":[{"code":"BadRequest","message":"{\r\n \"error\": {\r\n \"code\": \"InvalidTemplate\",\r\n \"message\": \"Unable to process template language expressions for resource '/subscriptions/45d8a30e-3363-4e0e-849a-4bb0bbf71a7b/resourceGroups/core67/providers/Microsoft.Portal/dashboards/Bot Analytics Dashboard' at line '34' and column '9'. 'The template parameter 'virtualMachineName' is not found. Please see https://aka.ms/arm-template-structure for usage details.'\"\r\n }\r\n}"}]}
```
-->

<!--
## Additional information

### Customize your telemetry client

If you want to customize your telemetry to log into a separate service, you have to configure the system differently. If using Application Insights to do so, download the package `Microsoft.Bot.Builder.ApplicationInsights` via NuGet, or use npm to install `botbuilder-applicationinsights`. Details on getting the Application Insights keys can be found [here](../bot-service-resources-app-insights-keys.md). Otherwise, include what is necessary for logging to that service, then follow the section below.

**Use Custom Telemetry**
If you want to log telemetry events generated by the Bot Framework into a completely separate system, create a new class derived from the base interface `IBotTelemetryClient` and configure. Then, when adding your telemetry client as above, just inject your custom client. 

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...
    // Add the telemetry client.
    services.AddSingleton<IBotTelemetryClient, CustomTelemetryClient>();
    ...
}
```

### Add custom logging to your bot

Once the Bot has the new telemetry logging support configured, you can begin adding telemetry to your bot.  The `BotTelemetryClient`(in C#, `IBotTelemetryClient`) has several methods to log distinct types of events.  Choosing the appropriate type of event enables you to take advantage of Application Insights existing reports (if you are using Application Insights).  For general scenarios `TraceEvent` is typically used.  The data logged using `TraceEvent` lands in the `CustomEvent` table in Kusto.

If using a Dialog within your Bot, every Dialog-based object (including Prompts) will contain a new `TelemetryClient` property.  This is the `BotTelemetryClient` that enables you to perform logging.  This is not just a convenience, we'll see later in this article if this property is set, `WaterfallDialogs` will generate events.

### Details of telemetry options

There are three main components available for your bot to log telemetry, and each component has customization available for logging your own events, which are discussed in this section. 

- A  [Bot Framework Middleware component](#telemetry-middleware) (*TelemetryLoggerMiddleware*) that will log when messages are received, sent, updated or deleted. You can override for custom logging.
- [*LuisRecognizer* class.](#telemetry-support-luis)  You can override for custom logging in two ways - per invocation (add/replace properties) or derived classes.
- [*QnAMaker*  class.](#telemetry-qnamaker)  You can override for custom logging in two ways - per invocation (add/replace properties) or derived classes.

All components log using the `IBotTelemetryClient`  (or `BotTelemetryClient` in node.js) interface which can be overridden with a custom implementation.

#### Telemetry Middleware

|C#  | JavaScript |
|:-----|:------------|
|**Microsoft.Bot.Builder.TelemetryLoggerMiddleware** | **botbuilder-core** |

##### Out of box usage

The TelemetryLoggerMiddleware is a Bot Framework component that can be added without modification, and it will perform logging that enables out of the box reports that ship with the Bot Framework SDK. 

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, TelemetryLoggerMiddleware>();

// Create the Bot Framework Adapter with error handling enabled.
services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();
```

And in our adapter, we would specify the use of middleware:

```csharp
public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, IMiddleware middleware, ConversationState conversationState = null)
           : base(credentialProvider)
{
    ...
    Use(middleware);
    ...
}
```

##### Adding properties
If you decide to add additional properties, the TelemetryLoggerMiddleware class can be derived.  For example, if you would like to add the property "MyImportantProperty" to the `BotMessageReceived` event.  `BotMessageReceived` is logged when the user sends a message to the bot.  Adding the additional property can be accomplished in the following way:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    protected override Task OnReceiveActivityAsync(
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

In Startup, we would add the new class:

```csharp
// Create the telemetry middleware to track conversation events
services.AddSingleton<IMiddleware, MyTelemetryMiddleware>();
```

##### Completely replacing properties / Additional event(s)

If you decide to completely replace properties being logged, the `TelemetryLoggerMiddleware` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`BotMessageSend` properties and send multiple events, the following demonstrates how this could be performed:

```csharp
class MyTelemetryMiddleware : TelemetryLoggerMiddleware
{
    ...
    protected override Task OnReceiveActivityAsync(
                  Activity activity,
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
Note: When the standard properties are not logged, it will cause the out of box reports shipped with the product to stop working.

##### Events Logged from Telemetry Middleware
[BotMessageSend](bot-builder-telemetry-reference.md#customevent-botmessagesend)
[BotMessageReceived](bot-builder-telemetry-reference.md#customevent-botmessagereceived)
[BotMessageUpdate](bot-builder-telemetry-reference.md#customevent-botmessageupdate)
[BotMessageDelete](bot-builder-telemetry-reference.md#customevent-botmessagedelete)

#### Telemetry support LUIS 

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.Luis** | **botbuilder-ai** |

##### Out of box usage
The LuisRecognizer is an existing Bot Framework component, and telemetry can be enabled by passing a IBotTelemetryClient interface through `luisOptions`.  You can override the default properties being logged and log new events as required.

During construction of `luisOptions`, an `IBotTelemetryClient` object must be provided for this to work.

```csharp
var luisPredictionOptions = new LuisPredictionOptions()
{
    TelemetryClient = telemetryClient
};
var recognizer = new LuisRecognizer(luisApplication, luisPredictionOptions);
```

##### Adding properties
If you decide to add additional properties, the `LuisRecognizer` class can be derived.  For example, if you would like to add the property "MyImportantProperty" to the `LuisResult` event.  `LuisResult` is logged when a LUIS prediction call is performed.  Adding the additional property can be accomplished in the following way:

```csharp
class MyLuisRecognizer : LuisRecognizer 
{
   ...
   protected override Task OnRecognizerResultAsync(
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

##### Add properties per invocation
Sometimes it's necessary to add additional properties during the invocation:
```csharp
var additionalProperties = new Dictionary<string, string>
{
   { "dialogId", "myDialogId" },
   { "conversationInfo", "myConversationInfo" },
};

var result = await recognizer.RecognizeAsync(turnContext,
     additionalProperties).ConfigureAwait(false);
```

##### Completely replacing properties / Additional event(s)
If you decide to completely replace properties being logged, the `LuisRecognizer` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`LuisResult` properties and send multiple events, the following demonstrates how this could be performed:

```csharp
class MyLuisRecognizer : LuisRecognizer
{
    ...
    protected override Task OnRecognizerResultAsync(
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
Note: When the standard properties are not logged, it will cause the Application Insights out of box reports shipped with the product to stop working.

##### Events Logged from TelemetryLuisRecognizer
[LuisResult](bot-builder-telemetry-reference.md#customevent-luisevent)

### Telemetry QnAMaker

|C#  | JavaScript |
|:-----|:------------|
| **Microsoft.Bot.Builder.AI.QnA** | **botbuilder-ai** |


##### Out of box usage
The QnAMaker class is an existing Bot Framework component that adds two additional constructor parameters which enable logging that enable out of the box reports that ship with the Bot Framework SDK. The new `telemetryClient` references a `IBotTelemetryClient` interface which performs the logging.  

```csharp
var qna = new QnAMaker(endpoint, options, client, 
                       telemetryClient: telemetryClient,
                       logPersonalInformation: true);
```
##### Adding properties 
If you decide to add additional properties, there are two methods of doing this - when properties need to be added during the QnA call to retrieve answers or deriving from the `QnAMaker` class.  

The following demonstrates deriving from the `QnAMaker` class.  The example shows adding the property "MyImportantProperty" to the `QnAMessage` event.  The`QnAMessage` event is logged when a QnA `GetAnswers`call is performed.  In addition, we log a second event "MySecondEvent".

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

##### Adding properties during GetAnswersAsync
If you have properties that need to be added during runtime, the `GetAnswersAsync` method can provide properties and/or metrics to add to the event.

For example, if you want to add a `dialogId` to the event, it can be done like the following:
```csharp
var telemetryProperties = new Dictionary<string, string>
{
   { "dialogId", myDialogId },
};

var results = await qna.GetAnswersAsync(context, opts, telemetryProperties);
```
The `QnaMaker` class provides the capability of overriding properties, including PersonalInfomation properties.

##### Completely replacing properties / Additional event(s)
If you decide to completely replace properties being logged, the `TelemetryQnAMaker` class can be derived (like above when extending properties).   Similarly, logging new events is performed in the same way.

For example, if you would like to completely replace the`QnAMessage` properties, the following demonstrates how this could be performed:

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
Note: When the standard properties are not logged, it will cause the out of box reports shipped with the product to stop working.

##### Events Logged from TelemetryLuisRecognizer
[QnAMessage](bot-builder-telemetry-reference.md#customevent-qnamessage)

### All other events

A full list of events logged for your bot's telemetry can be found on the [telemetry reference page](bot-builder-telemetry-reference.md).

#### Identifiers and Custom Events

When logging events into Application Insights, the events generated contain default properties that you won't have to fill.  For example, `user_id` and `session_id`properties are contained in each Custom Event (generated with the `TraceEvent` API).  In addition, `activitiId`, `activityType` and `channelId` are also added.

> [!NOTE]
> Custom telemetry clients will not be provided these values.


Property |Type | Details
--- | --- | ---
`user_id`| `string` | [ChannelID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) + [From.Id](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [ConversationID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [The bot activity ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [The bot activity type](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [The bot activity channel ID](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id)
-->

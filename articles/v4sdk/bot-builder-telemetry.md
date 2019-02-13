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
ms.openlocfilehash: 4c268bc40b7dc3315232d8f695bdb79343b15e21
ms.sourcegitcommit: c7d2e939ec71f46f48383c750fddaf6627b6489d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/07/2019
ms.locfileid: "55795595"
---
# <a name="add-telemetry-to-your-bot"></a>Добавление телеметрии в бот
В версии 4.2 пакета SDK для Bot Framework теперь можно вести журнал телеметрии.  Это позволяет приложениям ботов отправлять данные о событиях во внешние службы, например Application Insights.

Этот документ описывает, как интегрировать новые функции телеметрии в бот.  

## <a name="using-bot-configuration-option-1-of-2"></a>С помощью конфигурации бота (вариант 1 из 2)
У вас есть два способа настроить бот.  В первом предполагается интеграция с Application Insights.

Файл конфигурации бота содержит метаданные о внешних службах, которые бот использует в своей работе.  Например, здесь хранятся сведения о подключении и метаданные для служб CosmosDB, Application Insights и LUIS (Распознавание речи).   

Если вам нужен стандартная версия Application Insights без дополнительной настройки (включая, например, инициализаторы телеметрии), передайте объект конфигурации бота во время его инициализации.   Это самый простой метод инициализировать и настроить Application Insights, чтобы отслеживать запросы, внешние вызовы к другим службам и события взаимодействия между службами.

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Add Application Insights - pass in the bot configuration
     services.AddBotApplicationInsights(botConfig);
     ...
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
     app.UseBotApplicationInsights()
                 ...
                .UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
                ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**В конфигурации бота нет Application Insights**. Что делать, если конфигурация бота не содержит Application Insights?  Это не проблема. По умолчанию назначается "нулевой" клиент, который выполняет холостую команду на вызов любого метода.

**Несколько экземпляров Application Insights**. Что, если в конфигурации бота указано несколько разделов Application Insight?  В конфигурации вы можете указать, какой экземпляр службы Application Insights будет использоваться.

```csharp
// ASP.Net Core - Startup.cs

public void ConfigureServices(IServiceCollection services)
{
     // Add Application Insights
     services.AddBotApplicationInsights(botConfig, "myAppInsights");
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="overriding-the-telemetry-client-option-2-of-2"></a>Переопределение клиента телеметрии (вариант 2 из 2)

Если вы хотите настроить клиент Application Insights или даже использовать другую службу, следует использовать другую конфигурацию.

**Изменение конфигурации Application Insights**

```csharp

public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create Application Insight Telemetry Client
     // with custom configuration.
     var telemetryClient = TelemetryClient(myCustomConfiguration)
     
     // Add Application Insights
     services.AddBotApplicationInsights(new BotTelemetryClient(telemetryClient), "InstrumentationKey");
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

**Использование пользовательской телеметрии**. Если вы хотите регистрировать события телеметрии, созданные платформой Bot Framework, в сторонней системе, создайте класс, производный от базового интерфейса, и настройте его.  

```csharp
public void ConfigureServices(IServiceCollection services)
{
     ...
     // Create my IBotTelemetryClient-based logger
     var myTelemetryClient = MyTelemetryLogger();
     
     // Add Application Insights
     services.AddBotApplicationInsights(myTelemetryClient);
     ...
}
```

```JavaScript
const appInsightsClient = new ApplicationInsightsTelemetryClient(botConfig);
```

## <a name="add-custom-logging-to-your-bot"></a>Добавление в бот функции настраиваемого ведения журнала

Настроив для бота поддержку новой системы ведения журнала, вы сможете добавлять в этот бот данные телеметрии.  Класс `BotTelemetryClient`(или `IBotTelemetryClient` в C#) предоставляет несколько методов для регистрации в журнале событий разных типов.  Правильный выбор типа события позволяет воспользоваться преимуществами существующих отчетов Application Insights, (если вы используете Application Insights).  В общем случае применяется тип `TraceEvent`.  Регистрируемые с помощью `TraceEvent` данные попадают в таблицу `CustomEvent` в Kusto.

Если вы используете в боте объект Dialog, каждый объект на его основе (включая запросы) теперь содержит новое свойство `TelemetryClient`.  Это экземпляр `BotTelemetryClient`, который используется для ведения журнала.  Он нужен не только для удобства. Далее в этой статье мы увидим, как `WaterfallDialogs` генерирует события при настройке этого свойства.

### <a name="identifiers-and-custom-events"></a>Идентификаторы и пользовательские события

При регистрации событий в Application Insights создаваемые события уже содержат свойства по умолчанию, и вам не придется их заполнять.  Например, в каждом событии Custom (созданном с помощью API `TraceEvent`) содержатся свойства `user_id` и `session_id`.  Кроме них добавляются `activitiId`, `activityType` и `channelId`.

>Примечание. Эти значения не передаются в пользовательские клиенты телеметрии.

Свойство |type | Сведения
--- | --- | ---
`user_id`| `string` | [Идентификатор канала](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id) + [Идентификатор источника](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#from)
`session_id`| `string`|  [Идентификатор беседы](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#conversation)
`customDimensions.activityId`| `string` | [Идентификатор действия бота](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#id)
`customDimensions.activityType` | `string` | [Тип действия бота](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)
`customDimensions.channelId` | `string` |  [Идентификатор канала действия бота](https://github.com/Microsoft/botframework-obi/blob/master/botframework-activity/botframework-activity.md#channel-id)

## <a name="waterfalldialog-events"></a>События WaterfallDialog

Кроме собственных событий, теперь их создает объект `WaterfallDialog` в пакете SDK. В следующем разделе описаны события, созданные на платформе Bot Framework. Чтобы сохранять эти события, установите свойству `TelemetryClient` значение `WaterfallDialog`.

Ниже приведен пример изменения образца (BasicBot), который использует `WaterfallDialog`, для регистрации событий телеметрии.  BasicBot использует общий шаблон, в котором `WaterfallDialog` помещается в `ComponentDialog` (`GreetingDialog`).

```csharp
// IBotTelemetryClient is direct injected into our Bot
public BasicBot(BotServices services, UserState userState, ConversationState conversationState, IBotTelemetryClient telemetryClient)
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


## <a name="events-generated-by-the-bot-framework-service"></a>События, создаваемые службой Bot Framework

В дополнение к `WaterfallDialog`, который создает события в коде бота, служба канала Bot Framework также заносит в журнал данные о событиях.  Это помогает диагностировать проблемы с работой каналов или общие сбои бота.

### <a name="customevent-activity"></a>CustomEvent: "Activity"
**Откуда регистрируется:** Служба канала. Регистрируется службой канала при получении сообщения.

### <a name="exception-bot-errors"></a>Исключение: "Bot Errors"
**Откуда регистрируется:** Служба канала. Регистрируется каналом, когда вызов к боту возвращает ответ с кодом HTTP, отличным от 2XX.

## <a name="additional-events"></a>Дополнительные события

[Корпоративный шаблон](https://github.com/Microsoft/AI/tree/master/templates/Enterprise-Template) содержит открытый исходный кода, который вы можете свободно копировать.  Сюда включены несколько компонентов, которые можно повторно использовать и изменять в соответствии с потребностями в отчетах.

### <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
**Откуда регистрируется:** TelemetryLoggerMiddleware (**корпоративный пример**)

Регистрируется, когда бот получает новое сообщение.

- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ConversationID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Text (необязательно для персональных данных)
- FromId
- FromName
- RecipientId
- RecipientName
- ConversationId
- ConversationName
- Языковой стандарт

### <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
**Откуда регистрируется:** TelemetryLoggerMiddleware (**корпоративный пример**)

Регистрируется, когда бот отправляет сообщение.

- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ConversationID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ReplyToID
- Канал (канал источника, например Скайп, Cortana или Teams)
- RecipientId
- ConversationName
- Языковой стандарт
- Text (необязательно для персональных данных)
- RecipientName (необязательно для персональных данных)

### <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
**Откуда регистрируется:** TelemetryLoggerMiddleware. Регистрируется, когда бот обновляет сообщение (редкий случай).

### <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**Откуда регистрируется:** TelemetryLoggerMiddleware. Регистрируется, когда бот удаляет сообщение (редкий случай).

### <a name="customevent-luisintentinentname"></a>CustomEvent: LuisIntent.INENTName 
**Откуда регистрируется:** TelemetryLuisRecognizer (**корпоративный пример**)

Регистрирует результаты, полученные от службы LUIS.

- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ConversationID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Намерение
- IntentScore
- Вопрос
- ConversationId
- SentimentLabel
- SentimentScore
- *Сущности LUIS*
- **Новый** DialogId

### <a name="customevent-qnamessage"></a>CustomEvent: QnAMessage
**Откуда регистрируется:** TelemetryQnaMaker (**корпоративный пример**)

Регистрирует результаты, полученные от службы QnA Maker.

- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ConversationID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Имя пользователя
- ConversationId
- OriginalQuestion
- Вопрос
- Ответ
- Score (*Опционально*, если найден набор знаний)

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


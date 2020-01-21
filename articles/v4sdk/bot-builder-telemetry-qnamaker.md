---
title: Добавление телеметрии в бот QnA — Служба Azure Bot
description: Узнайте, как интегрировать новые функции телеметрии в бота с поддержкой QnA Maker.
keywords: телеметрия, AppInsights, Application Insights, мониторинг бота, QnA Maker
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 07/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d73bd8c26dde8826b145108268417a24840ed83e
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791246"
---
# <a name="add-telemetry-to-your-qnamaker-bot"></a>Добавление данных телеметрии в бота QnAMaker

[!INCLUDE[applies-to](../includes/applies-to.md)]


В версии 4.2 пакета SDK для Bot Framework теперь можно вести журнал телеметрии.  Так приложения ботов смогут отправлять данные о событиях в службы телеметрии, например [Application Insights](https://aka.ms/appinsights-overview). Данные телеметрии включают полезные сведения о боте (например о том, какие функции используются чаще всего), помогают обнаруживать нежелательное поведение и предоставляют сведения о доступности, производительности и использовании.

В пакет SDK для Bot Framework добавлены два новых компонента для ведения журнала телеметрии в ботах с поддержкой QnA Maker: `TelemetryLoggerMiddleware` и класс `QnAMaker`. `TelemetryLoggerMiddleware` представляет собой компонент ПО промежуточного слоя, который сохраняет в журнал данные о каждом полученном, отправленном, обновленном или удаленном сообщении, а класс QnAMaker поддерживает настраиваемое ведение журнала, расширяющее возможности телеметрии.

В этой статье вы изучите следующее:

* Код для подключения телеметрии к боту. 

* Код для включения встроенных функций ведения журнала и отчетности QnA, которые используют стандартные свойства событий. 

* Процедуру изменения или расширения стандартных свойств событий, предоставляемых пакетом SDK, для широкого спектра задач отчетности.


## <a name="prerequisites"></a>предварительные требования

* [Пример кода для QnA Maker](https://aka.ms/cs-qna).

* Подписка на [Microsoft Azure](https://portal.azure.com/).

* [Ключ Application Insights](../bot-service-resources-app-insights-keys.md).

* Вам будет проще, если вы уже знакомы со службой [QnA Maker](https://qnamaker.ai/).

* Учетная запись [QnA Maker](https://aka.ms/create-qna-maker).

* Опубликованная база знаний QnA Maker. Если у вас ее нет, выполните инструкции в статье [Руководство по созданию базы знаний на портале QnA Maker](https://aka.ms/create-publish-query-in-portal), чтобы создать базу знаний QnA Maker с вопросами и ответами.

> [!NOTE]
> В этой статье используется [пример кода для QnA Maker](https://aka.ms/cs-qna) и даны пошаговые инструкции по внедрению в него телеметрии. 

## <a name="wiring-up-telemetry-in-your-qna-maker-bot"></a>Подключение телеметрии к боту QnA Maker

Мы начнем с [примера приложения для QnA Maker](https://aka.ms/cs-qna) и добавим в него код, позволяющий интегрировать телеметрию в бота с помощью службы QnA. Так Application Insights сможет отслеживать запросы.

1. Откройте [пример кода для QnA Maker](https://aka.ms/cs-qna) в Visual Studio.

2. Добавление пакета NuGet `Microsoft.Bot.Builder.Integration.ApplicationInsights.Core `. Подробные сведения об использовании NuGet см. в руководстве по [установке пакетов и управлении ими в Visual Studio](https://aka.ms/install-manage-packages-vs).

3. Добавьте следующие инструкции в `Startup.cs`:
    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.Bot.Builder.ApplicationInsights;
    using Microsoft.Bot.Builder.Integration.ApplicationInsights.Core;
    ```

    > [!NOTE] 
    > Выполняя эти инструкции, чтобы обновить пример кода для QnA Maker, вы заметите, что инструкция using для `Microsoft.Bot.Builder.Integration.AspNet.Core` уже включена в код для QnA Maker.

4. Добавьте в метод `ConfigureServices()` файла `Startup.cs` следующий код. Теперь службы телеметрии станут доступными для бота через [внедрение зависимостей](https://aka.ms/asp.net-core-dependency-interjection):
    ```csharp
    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        // Create the Bot Framework Adapter with error handling enabled.
        services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();

        // Add Application Insights services into service collection
        services.AddApplicationInsightsTelemetry();

        // Add the standard telemetry client
        services.AddSingleton<IBotTelemetryClient, BotTelemetryClient>();

        // Create the telemetry middleware to track conversation events
        services.AddSingleton<TelemetryLoggerMiddleware>();

        // Add the telemetry initializer middleware
        services.AddSingleton<IMiddleware, TelemetryInitializerMiddleware>();

        // Add telemetry initializer that will set the correlation context for all telemetry items
        services.AddSingleton<ITelemetryInitializer, OperationCorrelationTelemetryInitializer>();

        // Add telemetry initializer that sets the user ID and session ID (in addition to other bot-specific properties, such as activity ID)
        services.AddSingleton<ITelemetryInitializer, TelemetryBotIdInitializer>();
        ...
    }
    ```
    
    > [!NOTE] 
    > Выполняя эти инструкции, чтобы обновить пример кода для QnA Maker, вы заметите, что `services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();` уже существует.

5. Настройте адаптер на использование кода ПО промежуточного слоя, который был добавлен в метод `ConfigureServices()`. Откройте `AdapterWithErrorHandler.cs` и добавьте `IMiddleware middleware` в список параметров конструктора. Добавьте инструкцию `Use(middleware);` последней строкой в конструктор:
    ```csharp
    public AdapterWithErrorHandler(ICredentialProvider credentialProvider, ILogger<BotFrameworkHttpAdapter> logger, IMiddleware middleware, ConversationState conversationState = null)
            : base(credentialProvider)
    {
        ...

        Use(middleware);
    }
    ```

6. Добавьте ключ инструментирования Application Insights в файл `appsettings.json`. Файл `appsettings.json` содержит метаданные о внешних службах, которые бот использует в своей работе. Например, здесь хранятся сведения о подключении и метаданные для служб CosmosDB, Application Insights и QnA Maker. Формат добавляемого в файл `appsettings.json` ключа должен быть таким:

    ```json
    {
        "MicrosoftAppId": "",
        "MicrosoftAppPassword": "",
        "QnAKnowledgebaseId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "QnAEndpointKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "QnAEndpointHostName": "https://xxxxxxxx.azurewebsites.net/qnamaker",
        "ApplicationInsights": {
            "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
        }
    }
    ```
   > [!Note] 
   > 
   > * Сведения о _ключе инструментирования Application_ см. в статье о [ключах Application Insights](../bot-service-resources-app-insights-keys.md).
    >
    > * У вас уже должна быть [учетная запись QnA Maker](https://aka.ms/create-qna-maker), а сведения о получении идентификатора базы знаний QnA, ключа конечной точки и имени узла вы можете найти [здесь](https://aka.ms/bot-framework-emulator-qna-keys).
    > 


На этом этапе вы уже включили телеметрию с помощью Application Insights.  Вы можете запустить бота локально с помощью эмулятора бота, а затем перейти к Application Insights, чтобы просмотреть журнал, в который регистрируются сведения о времени отклика, а также общей работоспособности приложения и выполнении. 

> [!TIP] 
> Включение и отключение ведения журнала событий действий и персональных данных описано в [этой статье](bot-builder-telemetry.md#enabling-or-disabling-activity-logging).

Теперь нам нужно включить функцию телеметрии в службу QnA Maker. 


## <a name="enabling-telemetry-to-capture-usage-data-from-the-qna-maker-service"></a>Настройка телеметрии для сбора данных о потреблении из службы QnA Maker

Служба QnA Maker имеет встроенную функцию ведения журнала телеметрии, поэтому настроить получение данных телеметрии из QnA Maker очень просто.  Для начала мы узнаем, как включить телеметрию в код QnA Maker, чтобы использовать встроенную функцию ведения журнала телеметрии, а затем попробуем изменить и добавить свойства существующих данных о событиях для поддержки более широкого спектра задач отчетности.

### <a name="enabling-default-qna-logging"></a>Включение стандартной функции ведения журнала QnA

1. Создайте закрытое поле только для чтения с типом `IBotTelemetryClient` в классе `QnABot` в `QnABot.cs`:

    ```cs
    public class QnABot : ActivityHandler
        {
            private readonly IBotTelemetryClient _telemetryClient;
            ...
   }
    ```

2. Добавьте параметр `IBotTelemetryClient` в конструктор класса `QnABot` в файле `QnABot.cs` и сохраните его значение в закрытое поле, созданное на предыдущем шаге:

    ```cs
    public QnABot(IConfiguration configuration, ILogger<QnABot> logger, IHttpClientFactory httpClientFactory, IBotTelemetryClient telemetryClient)
    {
        ...
        _telemetryClient = telemetryClient;
    }
    ```

3. Параметр _`telemetryClient`_ является обязательным при создании экземпляра объекта QnAMaker в `QnABot.cs`:

    ```cs
    var qnaMaker = new QnAMaker(new QnAMakerEndpoint
                {
                    KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
                    EndpointKey = _configuration["QnAEndpointKey"],
                    Host = _configuration["QnAEndpointHostName"]
                },
                null,
                httpClient,
                _telemetryClient);
    ```

    > [!TIP] 
    > Убедитесь, что имена свойств в записях `_configuration` соответствуют именам свойств, указанным в файле AppSettings.json. Значения этих свойств можно получить, нажав кнопку _Просмотреть код_ в https://www.qnamaker.ai/Home/MyServices:.

        
    ![Параметры приложения](media/AppSettings.json-QnAMaker.png)

#### <a name="viewing-telemetry-data-logged-from-the-qna-maker-default-entries"></a>Просмотр данных телеметрии, полученных из записей QnA Maker по умолчанию
Чтобы просмотреть в Application Insights сведения об использовании бота QnA Maker, выполните следующие действия после выполнения бота в эмуляторе.

1. Перейдите на [портал Azure](https://portal.azure.com/).

2. Перейдите к Application Insights, щелкнув __Мониторинг > Приложения__.

3. В Application Insights щелкните _Журналы (аналитика)_ на панели навигации, как показано ниже.

    ![Log Analytics](media/AppInsights-LogView-QnaBot.png)

4. Введите следующий запрос Kusto и щелкните _Запуск_.

    ```SQL
    customEvents
    | where name == 'QnaMessage'
    | extend answer = tostring(customDimensions.answer)
    | summarize count() by answer
    ```
5. Не закрывайте эту страницу в браузере. Мы вернемся к ней после добавления нового пользовательского свойства.

> [!TIP]
> Если вы не знакомы с языком запросов Kusto, который используется для написания запросов по журналам в Azure Monitor, но знаете язык запросов SQL, вам будет полезна [памятка по применению SQL для запросов по журналам Azure Monitor](https://aka.ms/azureMonitor-SQL-cheatsheet). 

### <a name="modifying-or-extending-the-default-event-properties"></a>Изменение или расширение стандартных свойств событий
Если вам нужны свойства, которые отсутствуют в классе `QnAMaker`, у вас есть два способа решить эту задачу. Оба они требуют создания собственного класса, производного от класса `QnAMaker`. Первый способ описан в разделе [Добавление свойств](#adding-properties) ниже, где свойства добавляются в существующее событие `QnAMessage`. Второй метод позволяет создавать новые события и добавлять в них свойства, как описано в разделе [Добавление новых событий с пользовательскими свойствами](#adding-new-events-with-custom-properties).  

> [!Note]
> Событие `QnAMessage` входит в состав пакета SDK для Bot Framework и предоставляет все свойства событий, которые сохраняются в Application Insights.



#### <a name="adding-properties"></a>Добавление свойств 

Наследование от класса `QnAMaker` демонстрируется в примере ниже.  Например, вы можете добавить свойство MyImportantProperty в событие `QnAMessage`.  Событие `QnAMessage` записывается в журнал при каждом вызове [GetAnswers](https://aka.ms/namespace-QnAMaker-GetAnswersAsync) службы в QnA.  

Изучив процесс добавления пользовательских свойств, мы создадим новое пользовательское событие и свяжем с ним свойства, а затем запустим бота в локальной среде с помощью эмулятора Bot Framework и посмотрим, что передается в Application Insights, с помощью языка запросов Kusto.

1. Создайте новый класс с именем `MyQnAMaker` в пространстве имен `Microsoft.BotBuilderSamples`, унаследовав его от класса `QnAMaker`, и сохраните как `MyQnAMaker.cs`. Для наследования от класса `QnAMaker` необходимо добавить инструкцию using `Microsoft.Bot.Builder.AI.QnA`. Теперь код должен выглядеть следующим образом:


    ```cs
    using Microsoft.Bot.Builder.AI.QnA;

    namespace Microsoft.BotBuilderSamples
    {
        public class MyQnAMaker : QnAMaker
        {

        }
    }
    ```
2. Добавьте конструктор класса в `MyQnAMaker`. Обратите внимание, что потребуются два дополнительных оператора using для параметров конструкторов `System.Net.Http` и `Microsoft.Bot.Builder`.

    ```cs
    ...
    using Microsoft.Bot.Builder.AI.QnA;
    using System.Net.Http;
    using Microsoft.Bot.Builder;

    namespace Microsoft.BotBuilderSamples
    {
        public class MyQnAMaker : QnAMaker
        {
            public MyQnAMaker(
                QnAMakerEndpoint endpoint,
                QnAMakerOptions options = null,
                HttpClient httpClient = null,
                IBotTelemetryClient telemetryClient = null,
                bool logPersonalInformation = false)
                : base(endpoint, options, httpClient, telemetryClient, logPersonalInformation)
            {

            }
        } 
    }  
    ```
3. Добавьте новое свойство в событие QnAMessage сразу после конструктора и включите инструкции `System.Collections.Generic`, `System.Threading` и `System.Threading.Tasks`:

    ```cs
    using Microsoft.Bot.Builder.AI.QnA;
    using System.Net.Http;
    using Microsoft.Bot.Builder;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
 
    namespace Microsoft.BotBuilderSamples
    {
            public class MyQnAMaker : QnAMaker
            {
            ...

            protected override async Task OnQnaResultsAsync(
                                QueryResult[] queryResults,
                                Microsoft.Bot.Builder.ITurnContext turnContext,
                                Dictionary<string, string> telemetryProperties = null,
                                Dictionary<string, double> telemetryMetrics = null,
                                CancellationToken cancellationToken = default(CancellationToken))
            {
                var eventData = await FillQnAEventAsync(
                                        queryResults,
                                        turnContext,
                                        telemetryProperties,
                                        telemetryMetrics,
                                        cancellationToken)
                                    .ConfigureAwait(false);

                // Add new property
                eventData.Properties.Add("MyImportantProperty", "myImportantValue");

                // Log QnAMessage event
                TelemetryClient.TrackEvent(
                                QnATelemetryConstants.QnaMsgEvent,
                                eventData.Properties,
                                eventData.Metrics
                                );
            }

        } 
    }    
    ```

4. Включите в бота код использования нового класса, и теперь вместо объекта `QnAMaker` вы будете создавать в `QnABot.cs` объект `MyQnAMaker`:

    ```cs
    var qnaMaker = new MyQnAMaker(new QnAMakerEndpoint
                {
                    KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
                    EndpointKey = _configuration["QnAEndpointKey"],
                    Host = _configuration["QnAEndpointHostName"]
                },
                null,
                httpClient,
                _telemetryClient);
    ```

##### <a name="viewing-telemetry-data-logged-from-the-new-property-_myimportantproperty_"></a>Просмотр данных телеметрии, сохраненных через новое свойство _MyImportantProperty_
После выполнения бота в эмуляторе вы можете просмотреть в Application Insights результаты этого, выполнив следующие действия:

1. Вернитесь в браузер, где открыто представление _Журналы (аналитика)_ .

2. Введите следующий запрос Kusto и щелкните _Запуск_.  Он возвращает количество выполнений для нового свойства:

    ```SQL
    customEvents
    | where name == 'QnaMessage'
    | extend MyImportantProperty = tostring(customDimensions.MyImportantProperty)
    | summarize count() by MyImportantProperty
    ```

3. Чтобы отобразить подробные сведения вместо счетчика, удалите последнюю строку и повторно выполните запрос:

    ```SQL
    customEvents
    | where name == 'QnaMessage'
    | extend MyImportantProperty = tostring(customDimensions.MyImportantProperty)
    ```
### <a name="adding-new-events-with-custom-properties"></a>Добавление новых событий с пользовательскими свойствами
Если вам нужно сохранять данные в другое событие, отличное от `QnaMessage`, вы можете создать пользовательское событие с нужными свойствами.  Для этого мы добавим в конец класса `MyQnAMaker` следующий код:

```CS
public class MyQnAMaker : QnAMaker
{
    ...

    // Create second event.
    var secondEventProperties = new Dictionary<string, string>();

    // Create new property for the second event.
    secondEventProperties.Add(
                        "MyImportantProperty2",
                        "myImportantValue2");

    // Log secondEventProperties event
    TelemetryClient.TrackEvent(
                    "MySecondEvent",
                    secondEventProperties);

} 
```                            
## <a name="the-application-insights-dashboard"></a>Панель мониторинга Application Insights

Когда вы создаете ресурс Application Insights в Azure, автоматически создается и связанная панель мониторинга.  Эту панель мониторинга можно просмотреть, нажав кнопку **Панель мониторинга приложений** в верхней части колонки Application Insights. 

![Ссылка "Панель мониторинга приложений"](media/Application-Dashboard-Link.png)


Просмотреть данные можно также на портале Azure. Щелкните слева **Панель мониторинга** и выберите нужную панель мониторинга в раскрывающемся списке.

Здесь вы увидите некоторые стандартные сведения о производительности бота и дополнительные запросы, которые вы закрепили на панели мониторинга.


## <a name="additional-information"></a>Дополнительные сведения

* [Добавление данных телеметрии в бот](bot-builder-telemetry.md)

* [Что такое Azure Application Insights?](https://aka.ms/appinsights-overview)

* [Поиск в Application Insights](https://aka.ms/search-in-application-insights)

* [Создание настраиваемых панелей мониторинга ключевых показателей эффективности с помощью Azure Application Insights](https://aka.ms/custom-kpi-dashboards-application-insights)

---
title: Использование средства подготовки к отправке для служб LUIS и QnA Maker | Документация Майкрософт
description: Сведения об использовании служб LUIS и QnA Maker в боте.
keywords: Интеллектуальная служба распознавания речи (LUIS), служба QnA Maker, средство подготовки к отправке, несколько служб
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d3c9355a0e87d31029b92614dc182f3d7010c736
ms.sourcegitcommit: 9a38d76afb0e82fdccc1f36f9b1a65042671e538
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/04/2018
ms.locfileid: "39514944"
---
## <a name="integrate-multiple-luis-apps-and-qna-services-with-the-dispatch-tool"></a>Интеграция нескольких приложений LUIS и служб QnA Maker с помощью средства подготовки к отправке

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Руководство по использованию модели LUIS, созданной с помощью средства подготовки к отправке, для интеграции бота с несколькими приложениями распознавания речи (LUIS) и службами QnA Maker. 

Представьте, что вы разработали следующие службы, а теперь хотите создать бот, который интегрируется с ними.

| Service type (Тип службы) | ИМЯ | ОПИСАНИЕ |
|------|------|------|
| Приложение LUIS | HomeAutomation | Распознает намерения HomeAutomation.TurnOn, HomeAutomation.TurnOff, и HomeAutomation.None.|
| Приложение LUIS | Weather | Распознает намерения Weather.GetForecast и Weather.GetCondition.|
| Служба QnA Maker | Часто задаваемые вопросы  | Содержит ответы на вопросы о системе автоматизации освещения дома. |

Давайте сначала создадим приложения и службы, а затем интегрируем их вместе.

> [!NOTE]
> Все три приложения должны быть созданы в одном расположении Azure для успешной отправки кода к ним. В этом примере для отправки кода используется расположение "Западная часть США".

## <a name="create-the-luis-apps"></a>Создание приложений LUIS

Самым быстрым способом создания приложений LUIS HomeAutomation и Weather считается загрузка файлов [homeautomation.json][HomeAutomationJSON] и [weather.json][WeatherJSON]. Затем перейдите на [веб-сайт LUIS](https://www.luis.ai/home) и войдите в систему. Щелкните **Мои приложения** > **Импортировать новое приложение** и выберите файл homeautomation.json. Придумайте имя для приложения `homeautomation`. Щелкните **Мои приложения** > **Импортировать новое приложение** и выберите файл weather.json. Придумайте имя для другого приложения `weather`.

## <a name="create-the-qna-cognitive-service-in-azure"></a>Создание Cognitive Service для службы QnA Maker в Azure

Служба QnA Maker состоит из двух частей: службы Cognitive Service в Azure и базы знаний для пар вопросов и ответов, которые публикуются с помощью Cognitive Service.

Чтобы создать службу Cognitive в Azure, войдите на портал Azure по ссылке https://portal.azure.com, и выполните следующие действия.

1. Щелкните **Все службы**.
1. Выполните поиск по `Cognitive` и выберите **Cognitive Services**.
1. Щелкните **Добавить**.
1. Выполните поиск по `QnA` и выберите **QnA Maker**.
1. В колонке QnA Maker щелкните **Создать**.
1. Заполните информацию и создайте службу QnA Maker.

    <!-- TODO: Add screenshot.-->

    * Введите имя службы. В этом руководстве используется `SmartLightQnA`.
    * Выберите подписку, которую необходимо использовать.
    * Выберите ценовую категорию. В этом руководстве используется уровень F0 ("Бесплатный").
    * Создайте или выберите группу ресурсов. В этом руководстве создается новая группа ресурсов `SmartLightQnA`.
    * Выберите поиск ценовой категории. В этом руководстве используется уровень B ("Базовый").
    * Выберите поиск расположения. В этом руководстве используется `West US`.
    * Введите имя приложения для использования. В этом руководстве используется `SmartLightQnA`.
    * Выберите расположение веб-сайта. В этом руководстве используется `West US`.
    * Включите по умолчанию App Insights.
    * Выберите расположение App Insights. В этом руководстве используется `West US 2`.
    * Щелкните **Создать** для создания службы QnA Maker.
    * Azure создает и начинает развертывать службу.

1. После развертывания службы просмотрите уведомление и щелкните **Перейти к ресурсу**, чтобы перейти к колонке для службы.
1. Щелкните **Ключи** и получите свои ключи.

    * Скопируйте имя службы и первый ключ. Они понадобятся на следующих шагах.
    * Поскольку имеется два ключа, можно повторно создать один из них без необходимости прерывания работы службы.

## <a name="create-and-publish-the-qna-maker-knowledge-base"></a>Создание и публикация базы знаний QnA Maker

Перейдите на [веб-сайт QnA Maker](https://qnamaker.ai) и войдите в систему. Выберите **Создать базу знаний** и назовите ее "Часто задаваемые вопросы". Нажмите кнопку **Выбор файла** и отправьте [пример TSV-файла][FAQ_TSV]. Щелкните **Создать** и, как только служба будет готова, щелкните **Опубликовать**.

## <a name="use-the-dispatch-tool-to-create-the-dispatcher-luis-app"></a>Использование средства подготовки к отправке для создания приложения диспетчера LUIS

Теперь создадим приложение LUIS, чтобы объединить созданные службы.

Установите [Средство подготовки к отправке][DispatchTool], выполнив следующую команду в командной строке Node.js.

```
npm install -g botdispatch
```

Выполните следующую команду, чтобы инициализировать средство подготовки к отправке с именем `CombineWeatherAndLights`. Замените [Ключ разработки LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-concept-keys) на `"YOUR-LUIS-AUTHORING-KEY"`.

```
dispatch init -name CombineWeatherAndLights -luisAuthoringKey "YOUR-LUIS-AUTHORING-KEY" -luisAuthoringRegion westus
```

Получите идентификатор для каждого созданного приложения LUIS. Они находятся в разделе **Мои приложения** на [сайте LUIS](https://www.luis.ai/home). Чтобы просмотреть идентификатор приложения, щелкните имя приложения и выберите **Параметры**. 

Затем выполните команду `dispatch add` для каждого созданного приложения LUIS.

```
dispatch add -type luis -id "HOMEAUTOMATION-APP-ID" -name homeautomation -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"
dispatch add -type luis -id "WEATHER-APP-ID" -name weather -version 0.1 -key "YOUR-LUIS-AUTHORING-KEY"

```

Выполните команду `dispatch add` для службы QnA Maker. Параметр `-key` — это ключ, который был сохранен, когда на портале Azure выполнялись шаги по [созданию Cognitive Service для службы QnA Maker в Azure](./bot-builder-tutorial-dispatch.md#create-the-qna-cognitive-service-in-azure).

```
dispatch add -type qna -id "QNA-KB-ID" -name faq -key "YOUR-QNA-SUBSCRIPTION-KEY"
```

Выполните команду `dispatch create`:

```
dispatch create
```

При этом создается приложение диспетчера LUIS с именем **CombineWeatherAndLights**. Новое приложение можно увидеть на сайте [https://www.luis.ai/home](https://www.luis.ai/home). 

![Приложение диспетчера в LUIS.ai](media/tutorial-dispatch/dispatch-app-in-luis.png)

Щелкните новое приложение. В разделе **Намерения** можно увидеть, что приложение имеет намерения `l_homeautomation`, `l_weather`, и `q_faq`.

![Намерения диспетчера в LUIS.ai](media/tutorial-dispatch/dispatch-intents-in-luis.png)

Для обучения приложения LUIS нажмите кнопку **Обучение** и используйте вкладку **Опубликовать**, чтобы [опубликовать](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp) приложение. Чтобы скопировать идентификатор нового приложения для использования в боте, щелкните **Параметры**.

## <a name="create-the-bot"></a>Создание бота

Теперь можно подключить намерения приложения диспетчера к логике бота, которое направляет сообщения в исходные приложения LUIS и службу QnA Maker.

Можно использовать пример, включенный в состав пакета SDK для Bot Builder, в качестве отправной точки.

# <a name="ctabcsaddref"></a>[C#](#tab/csaddref)

Начните с кода в [LUIS Dispatch sample][DispatchBotCS] (Пример LUIS Dispatch). В Visual Studio [обновите пакеты Nuget](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui#updating-a-package) до последней предварительной версии из следующих:

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai.QnA` (требуется для QnA Maker)
* `Microsoft.Bot.Builder.Ai.Luis` (требуется для LUIS)

# <a name="javascripttabjsaddref"></a>[JavaScript](#tab/jsaddref)

Скачайте [LUIS Dispatch sample][DispatchBotJs] (Пример LUIS Dispatch).  Установите необходимые пакеты, включая пакет `botbuilder-ai` для служб LUIS и QnA Maker, с помощью npm:

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


Настройте пример для использования приложения диспетчера.

# <a name="ctabcsbotconfig"></a>[C#](#tab/csbotconfig)

В файле **appsettings.json** во вкладке [Пример LUIS Dispatch][DispatchBotCS], измените следующие поля.

| ИМЯ | ОПИСАНИЕ |
|------|------|
| `Luis-SubscriptionKey` |  Ключ подписки LUIS. Это может быть ключ конечных точек или ключ разработки, описанные [здесь](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-keys). | 
| `Luis-ModelId-Dispatcher` | Идентификатор приложения LUIS, который создается средством подготовки к отправке. | 
| `Luis-ModelId-HomeAutomation` | Идентификатор приложения, созданный из файла homeautomation.json  | 
| `Luis-ModelId-Weather` | Идентификатор приложения, созданный из файла weather.json | 
| `QnAMaker-Endpoint-Url` | Необходимо установить https://westus.api.cognitive.microsoft.com/qnamaker/v2.0 для предварительной версии служб QnA Maker. <br/>Установите https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker для новых служб QnA Maker (общедоступная версия).|
| `QnAMaker-SubscriptionKey` | Ключ подписки QnA Maker. | 
| `QnAMaker-KnowledgeBaseId` | Идентификатор базы знаний, созданный на [портале QnA Maker](https://qnamaker.ai).| 



В **Startup.cs** взгляните на метод `ConfigureServices`. Он содержит код для инициализации `LuisRecognizerMiddleware`, с помощью только что созданного идентификатора приложения LUIS.

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(this.Configuration);
    services.AddBot<LuisDispatchBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var (luisModelId, luisSubscriptionKey, luisUri) = GetLuisConfiguration(this.Configuration, "Dispatcher");

        var luisModel = new LuisModel(luisModelId, luisSubscriptionKey, luisUri);

        // If you want to get all the intents and scores that LUIS recognized, set Verbose = true in luisOptions
        var luisOptions = new LuisRequest { Verbose = true };

        var middleware = options.Middleware;
        middleware.Add(new LuisRecognizerMiddleware(luisModel, luisOptions: luisOptions));
    });
}
```
Файлы с конфигурацией LUIS и QnA Maker можно получить с помощью методов `GetLuisConfiguration` и `GetQnAMakerConfiguration`.
```csharp
public static (string modelId, string subscriptionId, Uri uri) GetLuisConfiguration(IConfiguration configuration, string serviceName)
{
    var modelId = configuration.GetSection($"Luis-ModelId-{serviceName}")?.Value;
    var subscriptionId = configuration.GetSection("Luis-SubscriptionKey")?.Value;
    var uri = new Uri(configuration.GetSection("Luis-Url")?.Value);
    return (modelId, subscriptionId, uri);
}

public static (string knowledgeBaseId, string subscriptionKey, string uri) GetQnAMakerConfiguration(IConfiguration configuration)
{
    var knowledgeBaseId = configuration.GetSection("QnAMaker-KnowledgeBaseId")?.Value;
    var subscriptionKey = configuration.GetSection("QnAMaker-SubscriptionKey")?.Value;
    var uri = configuration.GetSection("QnAMaker-Endpoint-Url")?.Value;
    return (knowledgeBaseId, subscriptionKey, uri);
}
```

### <a name="dispatch-the-message"></a>Подготовка к отправке сообщения

Взгляните на файл **LuisDispatchBot.cs**, где бот отправляет сообщение на приложение LUIS или QnA Maker для внутреннего компонента. 

Если приложение диспетчера обнаруживает намерения `l_homeautomation` или `l_weather` в `DispatchToTopIntent`, он вызывает метод `DispatchToTopIntent`, который создает `LuisRecognizer` для вызова исходных приложений `homeautomation` и `weather`. Если бот обнаруживает намерение `q_faq` или `none`, которые используются в качестве резервного варианта, он вызывает метод, который запрашивает службу QnA Maker.

> [!NOTE] 
> Если имена намерений `l_homeautomation`, `l_weather` или `q_faq` не соответствуют приложению LUIS, созданному с помощью средства подготовки к отправке, измените их в соответствии со строчной версией имен намерений, с которой можно ознакомится на [портале LUIS](https://www.luis.ai).

```csharp
private async Task DispatchToTopIntent(ITurnContext context, (string intent, double score)? topIntent)
{
    switch (topIntent.Value.intent.ToLowerInvariant())
    {
        case "l_homeautomation":
            await DispatchToLuisModel(context, this.luisModelHomeAutomation, "home automation");
            break;
        case "l_weather":
            await DispatchToLuisModel(context, this.luisModelWeather, "weather");
            break;
        case "none":
        // You can provide logic here to handle the known None intent (none of the above).
        // In this example we fall through to the QnA intent.
        case "q_faq":
            await DispatchToQnAMaker(context, this.qnaEndpoint, "FAQ");
            break;
        default:
            // The intent didn't match any case, so just display the recognition results.
            await context.SendActivity($"Dispatch intent: {topIntent.Value.intent} ({topIntent.Value.score}).");

            break;
    }
}
```

Метод `DispatchToQnAMaker` отправляет сообщение пользователя в службу QnA Maker. Убедитесь перед запуском бота, что служба опубликована на [портале QnA Maker](https://qnamaker.ai).

```csharp
private static async Task DispatchToQnAMaker(ITurnContext context, QnAMakerEndpoint qnaOptions, string appName)
{
    QnAMaker qnaMaker = new QnAMaker(qnaOptions);
    if (!string.IsNullOrEmpty(context.Activity.Text))
    {
        var results = await qnaMaker.GetAnswers(context.Activity.Text.Trim()).ConfigureAwait(false);
        if (results.Any())
        {
            await context.SendActivity(results.First().Answer);
        }
        else
        {
            await context.SendActivity($"Couldn't find an answer in the {appName}.");
        }
    }
}
```

Метод `DispatchToLuisModel` отправляет сообщение пользователя в исходные приложения LUIS `homeautomation` и `weather`. Убедитесь перед запуском бота, что приложение LUIS опубликовано на [портале LUIS](https://www.luis.ai).

```csharp
private static async Task DispatchToLuisModel(ITurnContext context, LuisModel luisModel, string appName)
{
    await context.SendActivity($"Sending your request to the {appName} system ...");
    var (intents, entities) = await RecognizeAsync(luisModel, context.Activity.Text);

    await context.SendActivity($"Intents detected by the {appName} app:\n\n{string.Join("\n\n", intents)}");

    if (entities.Count() > 0)
    {
        await context.SendActivity($"The following entities were found in the message:\n\n{string.Join("\n\n", entities)}");
    }
    
    // Here, you can add code for calling the hypothetical home automation or weather service, 
    // passing in the appName and any intent or entity information that you need 
}
```

Метод `RecognizeAsync` вызывает `LuisRecognizer` для получения результатов из приложения LUIS.

```cs
private static async Task<(IEnumerable<string> intents, IEnumerable<string> entities)> RecognizeAsync(LuisModel luisModel, string text)
{
    var luisRecognizer = new LuisRecognizer(luisModel);
    var recognizerResult = await luisRecognizer.Recognize(text, System.Threading.CancellationToken.None);

    // list the intents
    var intents = new List<string>();
    foreach (var intent in recognizerResult.Intents)
    {
        intents.Add($"'{intent.Key}', score {intent.Value}");
    }

    // list the entities
    var entities = new List<string>();
    foreach (var entity in recognizerResult.Entities)
    {
        if (!entity.Key.ToString().Equals("$instance"))
        {
            entities.Add($"{entity.Key}: {entity.Value.First}");
        }
    }

    return (intents, entities);
}
```

# <a name="javascripttabjsbotconfig"></a>[JavaScript](#tab/jsbotconfig)

Начните с кода в [Dispatch bot sample][DispatchBotJs] (Пример бота диспетчера). Откройте файл **app.js** и при необходимости замените поля `appId` идентификаторами приложений LUIS. Если оставить поля `appId` в их изначальном виде, то будет использоваться общедоступная версия приложения LUIS, созданная для демонстрационных целей.

```javascript
// Create LuisRecognizers and QnAMaker
// The LUIS applications are public, meaning you can use your own subscription key to test the applications.
// For QnAMaker, users are required to create their own knowledge base.
// The exported LUIS applications and QnAMaker knowledge base can be found with the sample bot.

// The corresponding LUIS application JSON is `dispatchSample.json`
const dispatcher = new LuisRecognizer({
    appId: '0b18ab4f-5c3d-4724-8b0b-191015b48ea9',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `homeautomation.json`
const homeAutomation = new LuisRecognizer({
    appId: 'c6d161a5-e3e5-4982-8726-3ecec9b4ed8d',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});

// The corresponding LUIS application JSON is `weather.json`
const weather = new LuisRecognizer({
    appId: '9d0c9e9d-ce04-4257-a08a-a612955f2fb5',
    subscriptionKey: process.env.LUIS_SUBSCRIPTION_KEY,
    serviceEndpoint: 'https://westus.api.cognitive.microsoft.com/',
    verbose: true
});
```

В следующем коде замените `endpointKey` и `knowledgeBaseID` ключом и идентификатором базы знаний службы QnA Maker. `host` должно быть присвоено `https://westus.api.cognitive.microsoft.com/qnamaker/v2.0` для предварительной версии служб QnA Maker, а `https://YOUR-QNA-SERVICE-NAME.azurewebsites.net/qnamaker` — для новых служб QnA Maker (общедоступная версия).

```javascript
const faq = new QnAMaker(
    {
        knowledgeBaseId: '',
        endpointKey: '',
        host: ''
    },
    {
        answerBeforeNext: true
    }
);

```

Остальная часть кода в **app.js** обрабатывает намерения `l_homeautomation`, `l_weather` и `None`, запустив соответствующие диалоговые окна. В случае использования намерения `q_faq`, приложение возвращает ответ службы QnAMaker.

> [!NOTE] 
> Если имена намерений `l_homeautomation`, `l_weather` или `q_faq` не соответствуют приложению LUIS, созданного с помощью средства подготовки к отправке, измените их в соответствии с именами намерений, с которыми можно ознакомится на [портале LUIS](https://www.luis.ai).

```javascript
// create conversation state
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// register some dialogs for usage with the LUIS apps that are being dispatched to
const dialogs = new DialogSet();

function findEntities(entityName, entityResults) {
    let entities = []
    if (entityName in entityResults) {
        entityResults[entityName].forEach((entity, idx) => {
            entities.push(entity);
        });
    }
    return entities.length > 0 ? entities : undefined;
}

dialogs.add('HomeAutomation_TurnOff', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOff = state.homeAutomationTurnOff ? state.homeAutomationTurnOff + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOff}: You reached the "HomeAutomation.TurnOff" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('HomeAutomation_TurnOn', [
    async (dialogContext, args) => {
        const devices = findEntities('HomeAutomation_Device', args.entities);
        const operations = findEntities('HomeAutomation_Operation', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.homeAutomationTurnOn = state.homeAutomationTurnOn ? state.homeAutomationTurnOn + 1 : 1;
        await dialogContext.context.sendActivity(`${state.homeAutomationTurnOn}: You reached the "HomeAutomation.TurnOn" dialog.`);
        if (devices) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Device" entities:\n${devices.join(', ')}`);
        }
        if (operations) {
            await dialogContext.context.sendActivity(`Found these "HomeAutomation_Operation" entities:\n${operations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetForecast', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetForecast = state.weatherGetForecast ? state.weatherGetForecast + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetForecast}: You reached the "Weather.GetForecast" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('Weather_GetCondition', [
    async (dialogContext, args) => {
        const locations = findEntities('Weather_Location', args.entities);

        const state = conversationState.get(dialogContext.context);
        state.weatherGetCondition = state.weatherGetCondition ? state.weatherGetCondition + 1 : 1;
        await dialogContext.context.sendActivity(`${state.weatherGetCondition}: You reached the "Weather.GetCondition" dialog.`);
        if (locations) {
            await dialogContext.context.sendActivity(`Found these "Weather_Location" entities:\n${locations.join(', ')}`);
        }
        await dialogContext.end();
    }
]);

dialogs.add('None', [
    async (dialogContext) => {
        const state = conversationState.get(dialogContext.context);
        state.noneIntent = state.noneIntent ? state.noneIntent + 1 : 1;
        await dialogContext.context.sendActivity(`${state.noneIntent}: You reached the "None" dialog.`);
        await dialogContext.end();
    }
]);

adapter.use(dispatcher);

// Listen for incoming Activities
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            // Retrieve the LUIS results from our dispatcher LUIS application
            const luisResults = dispatcher.get(context);

            // Extract the top intent from LUIS and use it to select which LUIS application to dispatch to
            const topIntent = LuisRecognizer.topIntent(luisResults);

            const isMessage = context.activity.type === 'message';
            if (isMessage) {
                switch (topIntent) {
                    case 'l_homeautomation':
                        const homeAutoResults = await homeAutomation.recognize(context);
                        const topHomeAutoIntent = LuisRecognizer.topIntent(homeAutoResults);
                        await dc.begin(topHomeAutoIntent, homeAutoResults);
                        break;
                    case 'l_weather':
                        const weatherResults = await weather.recognize(context);
                        const topWeatherIntent = LuisRecognizer.topIntent(weatherResults);
                        await dc.begin(topWeatherIntent, weatherResults);
                        break;
                    case 'q_faq':
                        await faq.answer(context);
                        break;
                    default:
                        await dc.begin('None');
                }
            }

            if (!context.responded) {
                await dc.continue();
                if (!context.responded && isMessage) {
                    await dc.context.sendActivity(`Hi! I'm the LUIS dispatch bot. Say something and LUIS will decide how the message should be routed.`);
                }
            }
        }
    });
});

```

---
## <a name="run-the-bot"></a>Запуск бота

Проверьте бот с помощью [Bot Framework Emulator](../bot-service-debug-emulator.md). Напишите сообщение, например, "turn on the lights", чтобы отправить сообщение в приложение LUIS для домашней автоматизации, или напишите сообщение "get the weather in Seattle", чтобы отправить сообщение в приложение погоды LUIS.

> [!NOTE] 
> Перед запуском бота убедитесь, что опубликованы все приложения LUIS, созданные на [портале LUIS](https://www.luis.ai), и службы QnA Maker на [портале QnA Maker](https://qnamaker.ai).

![Отправка сообщений в бот подготовки к отправке](media/tutorial-dispatch/run-dispatch-bot.png)

## <a name="evaluate-the-dispatchers-performance"></a>Оценка производительности диспетчера

Иногда возникают сообщения пользователя, представленные в качестве примеров, как в приложениях LUIS, так и в службах QnA Maker. В этом случае объединенное приложение LUIS, которое создает средство подготовки к отправке, не будет эффективным для входных данных. Проверьте производительность приложения с помощью параметра `eval`. 

```
dispatch eval
```

Выполнение команды `dispatch eval` создает файл **Summary.html**, который предоставляет статистику о производительности новой языковой модели.

> [!TIP] 
> Фактически команду `dispatch eval` можно запустить в любом приложении LUIS, а не только в тех, что созданы с помощью средства подготовки к отправке.

### <a name="edit-intents-for-duplicates-and-overlaps"></a>Изменение намерений для дубликатов и пересечений

Просмотрите примеры фраз, помеченные как дубликаты в **Summary.html** и удалите похожие или пересекающиеся примеры. Например, предположим, что в приложение LUIS `homeautomation` для домашней автоматизации запросы типа "turn on the lights"сопоставляются с намерением TurnOnLights, а запросы типа "Why won't my lights turn on?" сопоставляются с намерением None. В этом случае они могут быть переданы в службу QnA Maker. При объединении приложения LUIS и службы QnA Maker с помощью отправки, необходимо выполнить следующие действия. 

* Удалить намерение "None" из исходного приложения LUIS `homeautomation`. Затем добавить фразы удаленного намерения в намерение "None" в приложение диспетчера.
* Если не удалить намерение "None" из исходного приложения LUIS, необходимо будет добавить в бот логику, чтобы передать сообщения в службу QnA Maker, которые соответствуют этому намерению.

> [!TIP] 
> Просмотрите [Best practices for Language Understanding](./bot-builder-concept-luis.md#best-practices-for-language-understanding) (Рекомендации для распознавания речи), чтобы ознакомится с советами по улучшению производительности языковой модели.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Using LUIS for Language Understanding][luis-v4-how-to] (Использование LUIS для распознавания речи)

[luis-v4-how-to]: bot-builder-howto-v4-luis.md
<!-- links -->

[HomeAutomationJSON]: https://aka.ms/dispatch-luis1
[WeatherJSON]: https://aka.ms/dispatch-luis2
[DispatchJSON]: https://aka.ms/dispatch-luis
[FAQ_TSV]: https://aka.ms/dispatch-qna-tsv

[DispatchTool]: https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch
[DispatchBotCS]: https://aka.ms/dispatch-sample-cs
[DispatchBotJs]: https://aka.ms/dispatch-sample-js




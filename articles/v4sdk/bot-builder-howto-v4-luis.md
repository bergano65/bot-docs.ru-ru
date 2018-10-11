---
title: Использование LUIS для распознавания речи | Документация Майкрософт
description: Сведения об использовании LUIS для распознавания естественного языка с пакетом SDK для Bot Builder.
keywords: Распознавание речи, LUIS, намерение, распознаватель, сущности, ПО промежуточного слоя
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/19/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c4a7cfba6f588c95dbebf4886ffd7e432d99c3ae
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389805"
---
# <a name="using-luis-for-language-understanding"></a>Использование LUIS для распознавания речи

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Возможность понимать, что пользователь имеет ввиду, в общении и в зависимости от контекста может быть сложной задачей, но может дать боту более естественно воспринимать общение. API распознавания речи, так же называемое LUIS, позволяет делать так, чтобы бот мог распознавать намерения пользовательских сообщений, использовать более естественный язык пользователя и лучше направлять последовательность общения. Дополнительные сведения об интеграции LUIS с ботом см. в разделе [Language Understanding for bots](./bot-builder-concept-LUIS.md) (API распознавания речи для ботов). 

В этом разделе описывается настройка простых ботов, которые используют LUIS для распознавания нескольких разных намерений.

## <a name="installing-packages"></a>Установка пакетов

Во-первых, убедитесь, что имеются пакеты, необходимые для LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

[Добавьте ссылку](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) на версию 4 следующих пакетов NuGet:


* `Microsoft.Bot.Builder.AI.LUIS`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Установите пакеты botbuilder и botbuilder-ai в свой проект с помощью npm:

* `npm install --save botbuilder`
* `npm install --save botbuilder-ai`

---

## <a name="set-up-your-luis-app"></a>Настройка приложения LUIS

Сначала настройте _приложение LUIS_, которое является службой, созданной на веб-сайте [luis.ai](https://www.luis.ai). Это приложение LUIS может быть подготовлено для определенных намерений, которые оно должно уметь распознавать. Сведения о создании приложения LUIS можно найти на веб-сайте LUIS.

В этом примере используется демонстрационное приложение LUIS, которое может распознавать намерения Help, Cancel и Weather. Идентификатор приложения уже находится в примере кода. Вам понадобится ключ Cognitive Services, который можно получить, зайдя на веб-сайт [luis.ai](https://www.luis.ai) и скопировав ключ из раздела **User settings** (Настройки пользователя) > **Authoring Key** (Ключ разработки).

> [!NOTE]
> Чтобы создать свою копию общедоступного приложения LUIS, использованного в этом примере, скопируйте файл [JSON](https://github.com/Microsoft/LUIS-Samples/blob/master/examples/simple-bot-example/FirstSimpleBotExample.json) LUIS. Затем [импортируйте](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/create-new-app#import-new-app) приложение LUIS, [ обучите](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/luis-how-to-train) и [опубликуйте](https://docs.microsoft.com/en-us/azure/cognitive-services/LUIS/publishapp) его. Замените идентификатор общедоступного приложения в примере кода на идентификатор приложения нового приложения LUIS.


### <a name="configure-your-bot-to-call-your-luis-app"></a>Настройка бота для вызова приложения LUIS

# <a name="ctabcs"></a>[C#](#tab/cs)

Хотя вы можете создать и вызывать приложение LUIS на каждом шаге, предпочтительным методом является регистрация службы LUIS в качестве singleton-класса и передача ее в качестве параметров в конструктор вашего бота. Этот метод чуть более сложный, поэтому мы продемонстрируем его здесь.

Начните с шаблона Echo бота и откройте **Startup.cs**. 

Добавьте инструкцию `using` для `Microsoft.Bot.Builder.AI.LUIS`

```csharp
// add this
using Microsoft.Bot.Builder.AI.LUIS;
```

Добавьте следующий код в конец `ConfigureServices` после инициализации состояния. В этом примере информацию предоставляет файл `appsettings.json`, но эти строки также можно получить из файла с расширением `.bot`, как это показано в примере, ссылка на который дана в конце статьи. Либо же информацию можно задать в коде для тестирования.

Singleton-класс возвращает новый объект `LuisRecognizer` в конструктор.

```csharp
    // Create and register a LUIS recognizer.
    services.AddSingleton(sp =>
    {
        // Get LUIS information from appsettings.json.
        var section = this.Configuration.GetSection("Luis");
        var luisApp = new LuisApplication(
            applicationId: section["AppId"],
            endpointKey: section["SubscriptionKey"],
            azureRegion: section["Region"]);

        // Specify LUIS options. These may vary for your bot.
        var luisPredictionOptions = new LuisPredictionOptions
        {
            IncludeAllIntents = true,
        };

        return new LuisRecognizer(
            application: luisApp,
            predictionOptions: luisPredictionOptions,
            includeApiResults: true);
    });
```

Вставьте свой ключ подписки с сайта [luis.ai](https://www.luis.ai) вместо `<subscriptionKey>`. Этот ключ можно найти, щелкнув имя своей учетной записи в правом верхнем углу и выбрав **Settings** (Параметры). Этот ключ называется **Authoring Key** (Ключ разработки).

> [!NOTE]
> Если вместо общедоступного используется собственное приложение LUIS, можно получить идентификатор, ключ подписки и URL-адрес для приложения LUIS на сайте [luis.ai](https://www.luis.ai). Эти сведения можно найти на вкладках **Publish** (Публикация) и **Settings** (Параметры) на странице вашего приложения.
>
>Базовый URL-адрес для использования в `LuisModel` можно найти, войдя на сайт [luis.ai](https://www.luis.ai), перейдя на вкладку **Publish** (Опубликовать) и просмотрев столбец **Endpoint** (Конечная точка) в разделе **Resources and Keys** (Ресурсы и ключи). Базовый URL-адрес — это часть **URL-адреса конечной точки** перед идентификатором подписки и другими параметрами.

Затем нужно передать боту этот экземпляр LUIS. Откройте файл **EchoBot.cs** и добавьте в конец файла следующий код. Для справки в пример также включены заголовок класса и элементы состояния, но мы не будем объяснять их в этой статье.

```csharp
public class EchoBot : IBot
{
    /// <summary>
    /// Gets the Echo Bot state.
    /// </summary>
    private IStatePropertyAccessor<EchoState> EchoStateAccessor { get; }

    /// <summary>
    /// Gets the LUIS recognizer.
    /// </summary>
    private LuisRecognizer Recognizer { get; } = null;

    public EchoBot(ConversationState state, LuisRecognizer luis)
    {
        EchoStateAccessor = state.CreateProperty<EchoState>("EchoBot.EchoState");

        // The incoming luis variable is the LUIS Recognizer we added above.
        this.Recognizer = luis ?? throw new ArgumentNullException(nameof(luis));
    }

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Сначала выполните шаги, приведенные в [кратком руководстве по JavaScript](../javascript/bot-builder-javascript-quickstart.md), чтобы создать бот. В этом примере информация службы LUIS задана в коде бота, но ее можно получить из файла `.bot`, как это показано в примере, ссылка на который дана в конце этой статьи.

В новом боте отредактируйте файл **app.js**, чтобы импортировать класс `LuisRecognizer` и создать экземпляр модели LUIS:

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

const luisApplication = {
    // This appID is for a public app that's made available for demo purposes
    // You can use it or use your own LUIS "Application ID" at https://www.luis.ai under "App Settings".
     applicationId: 'eb0bf5e0-b468-421b-9375-fdfb644c512e',
    // Replace endpointKey with your "Subscription Key"
    // your key is at https://www.luis.ai under Publish > Resources and Keys, look in the Endpoint column
    // The "subscription-key" is embeded in the Endpoint link. 
    endpointKey: '<your subscription key>',
    // You can find your app's region info embeded in the Endpoint link as well.
    // Some examples of regions are `westus`, `westcentralus`, `eastus2`, and `southeastasia`.
    azureRegion: 'westus'
}

// Create configuration for LuisRecognizer's runtime behavior.
const luisPredictionOptions = {
    includeAllIntents: true,
    log: true,
    staging: false
}

// Create the bot that handles incoming Activities.
const luisBot = new LuisBot(luisApplication, luisPredictionOptions);
```

Затем в конструкторе бота `LuisBot` получите приложение, чтобы создать экземпляр LuisRecognizer.

```javascript
    /**
     * The LuisBot constructor requires one argument (`application`) which is used to create an instance of `LuisRecognizer`.
     * @param {object} luisApplication The basic configuration needed to call LUIS. In this sample the configuration is retrieved from the .bot file.
     * @param {object} luisPredictionOptions (Optional) Contains additional settings for configuring calls to LUIS.
     */
    constructor(application, luisPredictionOptions) {
        this.luisRecognizer = new LuisRecognizer(application, luisPredictionOptions, true);
    }
```

> [!NOTE] 
> Если вместо общедоступного используется собственное приложение LUIS, можно получить идентификатор, ключ подписки и регион для приложения LUIS на сайте [https://www.luis.ai](https://www.luis.ai). Эти сведения можно найти на вкладках Publish (Публикация) и Settings (Параметры) на странице вашего приложения.

---

Теперь ваш бот настроен для использования Интеллектуальной службы распознавания речи (LUIS). Далее рассмотрим, как получить намерение из LUIS.

## <a name="get-the-intent-by-calling-luis"></a>Получение намерения путем вызова LUIS

Бот получает результаты из службы LUIS путем вызова API распознавания речи LUIS.

# <a name="ctabcs"></a>[C#](#tab/cs)

Если нужно, чтобы бот просто отправлял ответ на основе намерения, распознанного приложением LUIS, вызовите `LuisRecognizer` для получения `RecognizerResult`. Это можно сделать в коде в любой момент, когда нужно получить намерение LUIS.

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
// add this reference
using Microsoft.Bot.Builder.AI.LUIS;

namespace EchoBot
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Echo bot turn handler 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurnAsync(ITurnContext context, System.Threading.CancellationToken token)
        {            
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {
                // Call LUIS recognizer
                var result = this.Recognizer.RecognizeAsync(context, System.Threading.CancellationToken.None);
                
                var topIntent = result?.GetTopScoringIntent();

                switch ((topIntent != null) ? topIntent.Value.intent : null)
                {
                    case null:
                        await context.SendActivity("Failed to get results from LUIS.");
                        break;
                    case "None":
                        await context.SendActivity("Sorry, I don't understand.");
                        break;
                    case "Help":
                        await context.SendActivity("<here's some help>");
                        break;
                    case "Cancel":
                        // Cancel the process.
                        await context.SendActivity("<cancelling the process>");
                        break;
                    case "Weather":
                        // Report the weather.
                        await context.SendActivity("The weather today is sunny.");
                        break;
                    default:
                        // Received an intent we didn't expect, so send its name and score.
                        await context.SendActivity($"Intent: {topIntent.Value.intent} ({topIntent.Value.score}).");
                        break;
                }
            }
        }
    }    
}

```

Любые намерения, распознанные в высказывании, будут возвращены в виде сопоставления имен намерений для оценок и могут быть доступны из `result.Intents`. Статический метод `LuisRecognizer.topIntent()` предоставляется, чтобы упростить поиск главного намерения оценки для набора результатов.

Все распознанные сущности будут возвращены в виде сопоставления имен сущностей и значений, и их можно получить с помощью свойства `results.entities`. Дополнительные метаданные сущности можно вернуть, передав параметр `verbose=true` при создании LuisRecognizer. Добавленные метаданные можно получить с помощью свойства `results.entities.$instance`.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Отредактируйте код, который ожидает входящее действие, чтобы он вызывал `LuisRecognizer` для получения `RecognizerResult`.

```javascript
const { ActivityTypes } = require('botbuilder');
const { LuisRecognizer } = require('botbuilder-ai');

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Perform a call to LUIS to retrieve results for the user's message.
            const results = await this.luisRecognizer.recognize(turnContext);

            // Since the LuisRecognizer was configured to include the raw results, get the `topScoringIntent` as specified by LUIS.
            const topIntent = results.luisResult.topScoringIntent;
            
            switch (topIntent) {
                case 'None':
                    await context.sendActivity("Sorry, I don't understand.")
                    break;
                case 'Cancel':
                    await context.sendActivity("<cancelling the process>")
                    break;
                case 'Help':
                    await context.sendActivity("<here's some help>");
                    break;
                case 'Weather':
                    await context.sendActivity("The weather today is sunny.");
                    break;                        
                case 'null':
                    await context.sendActivity("Failed to get results from LUIS.")
                    break;
                default:
                    // Received an intent we didn't expect, so send its name and score.
                    await context.sendActivity(`The top intent was ${topIntent}`);
            }
        }
    });
});
```

Любые намерения, распознанные в высказывании, будут возвращены в виде сопоставления имен намерений для оценок и могут быть доступны из `results.intents`. Статический метод `LuisRecognizer.topIntent()` предоставляется, чтобы упростить поиск главного намерения оценки для набора результатов.


---

Попробуйте запустить бот в эмуляторе Bot Framework и произнесите слова weather, help и cancel.

![Запуск бота](./media/how-to-luis/run-luis-bot.png)

## <a name="extract-entities"></a>Извлечение сущностей

Кроме распознавания намерений, приложение LUIS умеет извлекать сущности, то есть наиболее важные слова для выполнения запроса пользователя. Например, в случае бота для получения прогноза погоды приложение LUIS может извлечь расположение для прогноза погоды из сообщения пользователя.

Распространенный способ структурирования общения — выявление любых сущностей в сообщении пользователя и запрашивание требуемых сущностей, которые не были обнаружены. Затем в последующих шагах обрабатывается ответ на запрос.

# <a name="ctabcs"></a>[C#](#tab/cs)

Предположим, что пользователь спросил: "What's the weather in Seattle?" `LuisRecognizer` выдает `RecognizerResult` со свойством `Entities`, которое имеет следующую структуру:

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

Приведенную ниже вспомогательную функцию можно добавить в бот для получения сущностей из `RecognizerResult` от LUIS. Для этого потребуется библиотека `Newtonsoft.Json.Linq`, которую нужно добавить в операторы **using**.

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

Когда сведения, например сущности, собираются из нескольких шагов разговора, может оказаться полезным сохранить нужные сведения в состоянии. Если сущность найдена, ее можно добавить в соответствующее поле состояния. Если в ходе общения соответствующее поле текущего шага уже заполнено, шаг по запрашиванию этих сведений можно пропустить.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Предположим, что пользователь спросил: "What's the weather in Seattle?" `LuisRecognizer` выдает `RecognizerResult` со свойством `entities`, которое имеет следующую структуру:

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

Эта функция `findEntities` ищет любые сущности, распознанные приложением LUIS, которые соответствуют аргументу `entityName`.


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

Когда сведения, например сущности, собираются из нескольких шагов разговора, может оказаться полезным сохранить нужные сведения в состоянии. Если сущность найдена, ее можно добавить в соответствующее поле состояния. Если в ходе общения соответствующее поле текущего шага уже заполнено, шаг по запрашиванию этих сведений можно пропустить.

## <a name="additional-resources"></a>Дополнительные ресурсы

Примеры использования LUIS можно найти в проектах для [[C#](https://aka.ms/cs-luis-sample)] или [[JavaScript](https://aka.ms/js-luis-sample)].

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Combine LUIS apps and QnA services using the Dispatch tool](./bot-builder-tutorial-dispatch.md) (Объединение приложений LUIS и служб QnA с помощью средства Dispatch)

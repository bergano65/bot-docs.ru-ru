---
title: Использование QnA Maker для ответов на вопросы | Документация Майкрософт
description: Узнайте, как использовать QnA Maker в боте.
keywords: вопрос и ответ, QnA, часто задаваемые вопросы, ПО промежуточного слоя
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 10/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3d488cc2bb61ef460ed45707596cb7db9e6c23e8
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999089"
---
# <a name="use-qna-maker-to-answer-questions"></a>Использование QnA Maker для ответов на вопросы

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Служба QnA Maker позволяет добавить в бот поддержку вопросов и ответов. Одна из основных сложностей при создании собственной службы QnA Maker — заполнить ее начальным набором вопросов и ответов. Во многих случаях вопросы и ответы уже существуют в таком содержимом как часто задаваемые вопросы или в другой документации. В других случаях следует настроить ответы на вопросы в более естественной манере общения.

## <a name="prerequisites"></a>Предварительные требования
- Создание учетной записи [QnA Maker](https://www.qnamaker.ai/)
- Скачайте образец QnA Maker [[C#](https://aka.ms/cs-qna) | [JavaScript](https://aka.ms/js-qna-sample)]

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>Создание службы QnA Maker и публикация базы знаний

После создания учетной записи службы QnA Maker выполните инструкции по созданию [службы QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) и [базы знаний](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base). 

Опубликовав свою базу знаний, запишите следующие значения для подключения бота к базе знаний программным способом.
- На сайте [QnA Maker](https://www.qnamaker.ai/) выберите свою базу знаний.
- При открытой базе знаний выберите **Параметры**. Сохраните значение из поля _имя службы_ как <имя_базы_знаний>
- Прокрутите вниз, чтобы найти **Сведения о развертывании** и запишите следующие значения:
   - POST /knowledgebases/<идентификатор_базы_знаний>/generateAnswer
   - Узел: https://<имя_узла>.azurewebsites.net/qnamaker
   - Авторизация: EndpointKey <ключ_конечной_точки>

## <a name="installing-packages"></a>Установка пакетов

Прежде чем приступить к кодированию, убедитесь, что имеются пакеты, необходимые для QnA Maker.

# <a name="ctabcs"></a>[C#](#tab/cs)

Добавьте в бот следующий [пакет NuGet](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui).

* `Microsoft.Bot.Builder.AI.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Компоненты QnA Maker входят в состав пакета `botbuilder-ai`. Этот пакет можно добавить в проект с помощью npm.

```shell
npm install --save botbuilder-ai
```

---

## <a name="using-cli-tools-to-update-your-bot-configuration"></a>Использование средств интерфейса командной строки для обновления конфигурации в файле .bot

Есть и другой способ получить доступ к значениям из базы знаний: с помощью средств CLI для BotBuilder [qnamaker](https://aka.ms/botbuilder-tools-qnaMaker) и [msbot](https://aka.ms/botbuilder-tools-msbot-readme) получите метаданные базы знаний и добавьте их в файл .bot.

1. Откройте окно терминала или командную строку и перейдите к корневому каталогу в проекте бота.
2. Запустите `qnamaker init`, чтобы создать файл ресурсов QnA Maker (**.qnamakerrc**). Он предложит вам ввести ключ подписки QnA Maker.
3. Выполните следующую команду, чтобы скачать метаданные и добавить их в файл конфигурации бота.

    ```shell
    qnamaker get kb --kbId <your-kb-id> --msbot | msbot connect qna --stdin [--secret <your-secret>]
    ```
Если вы используете шифрование файла конфигурации, для его обновления необходимо указать секретный ключ.

## <a name="using-qna-maker"></a>Использование QnA Maker
Прежде всего следует добавить ссылку на приложение QnA Maker при инициализации бота. Затем вы сможете обращаться к нему из логики этого бота.

# <a name="ctabcs"></a>[C#](#tab/cs)
Откройте пример QnA Maker, который вы скачали ранее. Мы немного изменим этот код для наших задач.
Во-первых, добавьте в файл `BotConfiguration.bot` необходимые сведения для доступа к базе знаний, в том числе имя узла, ключ конечной точки и идентификатор базы знаний (KbId). Это значения, сохраненные из раздела **Параметры** для базы знаний в QnA Maker.

```json
{
  "name": "QnABotSample",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "id": "1",
      "appPassword": ""
    },
    {
      "type": "qna",
      "name": "QnABot",
      "KbId": "<YOUR_KNOWLEDGE_BASE_ID>",
      "Hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
      "EndpointKey": "<YOUR_ENDPOINT_KEY>"
    }
  ],
  "version": "2.0",
  "padlock": ""
}
```

Далее мы создадим экземпляр QnA Maker в `Startup.cs`. Это действие извлекает перечисленные выше сведения из файла `BotConfiguration.bot`. Для целей тестирования эти значения можно просто включить в код.

```csharp
private static BotServices InitBotServices(BotConfiguration config)
{
    var qnaServices = new Dictionary<string, QnAMaker>();
    foreach (var service in config.Services)
    {
        switch (service.Type)
        {
            case ServiceTypes.QnA:
            {
                // Create a QnA Maker that is initialized and suitable for passing
                // into the IBot-derived class (QnABot).
                var qna = (QnAMakerService)service;
                if (qna == null)
                {
                    throw new InvalidOperationException("The QnA service is not configured correctly in your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.KbId))
                {
                    throw new InvalidOperationException("The QnA KnowledgeBaseId ('kbId') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.EndpointKey))
                {
                    throw new InvalidOperationException("The QnA EndpointKey ('endpointKey') is required to run this sample. Please update your '.bot' file.");
                }

                if (string.IsNullOrWhiteSpace(qna.Hostname))
                {
                    throw new InvalidOperationException("The QnA Host ('hostname') is required to run this sample. Please update your '.bot' file.");
                }

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
    var connectedServices = new BotServices(qnaServices);
    return connectedServices;
}
```

Теперь следует передать этот экземпляр QnAMaker боту. Откройте `QnABot.cs` и добавьте следующий код в начало файла. Если вы обращаетесь к собственной базе знаний, измените представленное ниже _приветственное сообщение_ на более полезное с инструкциями для пользователей.

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a quesiton to get started.";
    private readonly BotServices _services;
    public QnABot(BotServices services)
    {
        _services = services ?? throw new System.ArgumentNullException(nameof(services));
        Console.WriteLine($"{_services}");
        if (!_services.QnAServices.ContainsKey(QnAMakerKey))
        {
            throw new System.ArgumentException($"Invalid configuration. Please check your '.bot' file for a QnA service named '{QnAMakerKey}'.");
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
Откройте пример QnA Maker, который вы скачали ранее. Мы немного изменим этот код для наших задач.
В нашем примере код запуска находится в файле **index.js**, код с логикой бота — в файле **bot.js**, а дополнительные данные о конфигурации — в файле **qnamaker.bot**.

Когда вы выполните все инструкции по созданию базы знаний и обновлению файла **.bot**, файл **qnamaker.bot** будет содержать запись о базе знаний QnA Maker.

```json
{
    "name": "qnamaker",
    "description": "",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "1",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "qna",
            "name": "<YOUR_KB_NAME>",
            "kbId": "<YOUR_KNOWLEDGE_BASE_ID>",
            "endpointKey": "<YOUR_ENDPOINT_KEY>",
            "hostname": "https://<YOUR_HOSTNAME>.azurewebsites.net/qnamaker",
            "id": "221"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

В файле **index.js** мы считываем сведения о конфигурации и используем их для создания службы QnA Maker и инициализации бота.

Замените значение `QNA_CONFIGURATION` именем базы данных, которое отображается в файле конфигурации.

```js
// QnA Maker knowledge base name as specified in .bot file.
const QNA_CONFIGURATION = '<YOUR_KB_NAME>';

// Get endpoint and QnA Maker configurations by service name.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
const qnaConfig = botConfig.findServiceByNameOrId(QNA_CONFIGURATION);

// Map the contents to the required format for `QnAMaker`.
const qnaEndpointSettings = {
    knowledgeBaseId: qnaConfig.kbId,
    endpointKey: qnaConfig.endpointKey,
    host: qnaConfig.hostname
};

// Create adapter...

// Create the QnAMakerBot.
let bot;
try {
    bot = new QnAMakerBot(qnaEndpointSettings, {});
} catch (err) {
    console.error(`[botInitializationError]: ${ err }`);
    process.exit();
}
```

Теперь мы создадим HTTP-сервер для прослушивания входящих запросов, которые будут создавать вызовы логики бота.

```js
// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open qnamaker.bot file in the emulator.`);
});

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        await bot.onTurn(turnContext);
    });
});
```

---

## <a name="calling-qna-maker-from-your-bot"></a>Вызов QnA Maker из кода бота

# <a name="ctabcs"></a>[C#](#tab/cs)

Когда боту потребуется ответ от QnAMaker, вызовите из кода бота метод `GetAnswersAsync()`, чтобы получить ответ с учетом текущего контекста. Если вы обращаетесь к собственной базе знаний, измените представленное ниже _сообщение об отсутствии ответов_ на более полезное с инструкциями для пользователей.

```csharp
// Check QnA Maker model
var response = await _services.QnAServices[QnAMakerKey].GetAnswersAsync(turnContext);
if (response != null && response.Length > 0)
{
    await turnContext.SendActivityAsync(response[0].Answer, cancellationToken: cancellationToken);
}
else
{
    var msg = @"No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs.
                To see QnA Maker in action, ask the bot questions like 'Why won't it turn on?' or 'I need help'.";
    await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
}

    /// ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В файле **bot.js** мы передаем пользовательский ввод в метод `generateAnswer` службы QnA Maker, чтобы получать ответы из базы знаний. Если вы обращаетесь к собственной базе знаний, измените представленные ниже сообщения _об отсутствии ответов_ и _приветствия_ на более полезное с инструкциями для пользователей.

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');
const { QnAMaker, QnAMakerEndpoint, QnAMakerOptions } = require('botbuilder-ai');

/**
 * A simple bot that responds to utterances with answers from QnA Maker.
 * If an answer is not found for an utterance, the bot responds with help.
 */
class QnAMakerBot {
    /**
     * The QnAMakerBot constructor requires one argument (`endpoint`) which is used to create an instance of `QnAMaker`.
     * @param {QnAMakerEndpoint} endpoint The basic configuration needed to call QnA Maker. In this sample the configuration is retrieved from the .bot file.
     * @param {QnAMakerOptions} config An optional parameter that contains additional settings for configuring a `QnAMaker` when calling the service.
     */
    constructor(endpoint, qnaOptions) {
        this.qnaMaker = new QnAMaker(endpoint, qnaOptions);
    }

    /**
     * Every conversation turn for our QnAMakerBot will call this method.
     * @param {TurnContext} turnContext Contains all the data needed for processing the conversation turn.
     */
    async onTurn(turnContext) {
        // By checking the incoming Activity type, the bot only calls QnA Maker in appropriate cases.
        if (turnContext.activity.type === ActivityTypes.Message) {
            // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
            const qnaResults = await this.qnaMaker.generateAnswer(turnContext.activity.text);

            // If an answer was received from QnA Maker, send the answer back to the user.
            if (qnaResults[0]) {
                await turnContext.sendActivity(qnaResults[0].answer);

            // If no answers were returned from QnA Maker, reply with help.
            } else {
                await turnContext.sendActivity('No QnA Maker answers were found. This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. To see QnA Maker in action, ask the bot questions like "Why won\'t it turn on?" or "I need help."');
            }

        // If the Activity is a ConversationUpdate, send a greeting message to the user.
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate &&
                   turnContext.activity.recipient.id !== turnContext.activity.membersAdded[0].id) {
            await turnContext.sendActivity('Welcome to the QnA Maker sample! Ask me a question and I will try to answer it.');

        // Respond to all other Activity types.
        } else if (turnContext.activity.type !== ActivityTypes.ConversationUpdate) {
            await turnContext.sendActivity(`[${ turnContext.activity.type }]-type activity detected.`);
        }
    }
}

module.exports.QnAMakerBot = QnAMakerBot;
```

---

Задайте боту вопросы, чтобы увидеть ответы из службы QnA Maker. Дополнительные сведения о тестировании и публикации службы вопросов и ответов вы найдете в статье [о тестировании базы знаний](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/test-knowledge-base) в QnA Maker.

## <a name="next-steps"></a>Дополнительная информация

QnA Maker можно объединять с другими службами Cognitive Services, чтобы сделать бота еще более мощным. Средство подготовки к отправке предоставляет способ объединения QnA с распознаванием речи (LUIS) в боте.

> [!div class="nextstepaction"]
> [Combine LUIS apps and QnA services using the Dispatch tool](./bot-builder-tutorial-dispatch.md) (Объединение приложений LUIS и служб QnA с помощью средства Dispatch)

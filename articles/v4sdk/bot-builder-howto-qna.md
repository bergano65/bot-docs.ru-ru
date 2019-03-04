---
title: Использование QnA Maker для ответов на вопросы | Документация Майкрософт
description: Узнайте, как использовать QnA Maker в боте.
keywords: question and answer, QnA, FAQs, qna maker
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5a5aec71092503dad83827225f7c0adaf22c4d17
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/21/2019
ms.locfileid: "56590979"
---
# <a name="use-qna-maker-to-answer-questions"></a>Использование QnA Maker для ответов на вопросы

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Служба QnA Maker позволяет добавить в бот поддержку вопросов и ответов. Одна из основных сложностей при создании собственной службы QnA Maker — заполнить ее начальным набором вопросов и ответов. Во многих случаях вопросы и ответы уже существуют в таком содержимом как часто задаваемые вопросы или в другой документации. В других случаях следует настроить ответы на вопросы в более естественной манере общения.

В этой статье мы создадим базу знаний и используем ее в боте.

## <a name="prerequisites"></a>Предварительные требования
- Учетная запись [QnA Maker](https://www.qnamaker.ai/)
- Код в этой статье основан на примере **QnA Maker**. Вам потребуется копия примера [для C#](https://aka.ms/cs-qna) или [для Javascript](https://aka.ms/js-qna-sample).
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download).
- Понимание [основных принципов ботов](bot-builder-basics.md), [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/overview/overview) и файлов [.bot](bot-file-basics.md).

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>Создание службы QnA Maker и публикация базы знаний
1. Прежде всего необходимо создать [службу QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure).
1. Затем вы создадите базу знаний с помощью файла `smartLightFAQ.tsv`, который расположен в папке CognitiveModels проекта. Действия по созданию, обучению и публикации [базы знаний](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base) QnA Maker описаны в документации по QnA Maker. При выполнении этих действий присвойте базе знаний имя `qna` и выберите для ее заполнения файл `smartLightFAQ.tsv`.
> Примечание. Инструкции из этой статьи можно применить и для получения доступа к собственной базе знаний QnA Maker, разработанной пользователем.

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>Получение значений для подключения бота к базе знаний
1. На сайте [QnA Maker](https://www.qnamaker.ai/) выберите свою базу знаний.
1. При открытой базе знаний выберите **Параметры**. Сохраните значение из поля _Имя службы_. Это значение поможет вам найти нужную базу знаний в интерфейсе портала QnA Maker. Оно не используется для подключения к этой базе знаний из приложения бота. 
1. Прокрутите вниз, чтобы найти **Сведения о развертывании**, и запишите следующие значения:
   - POST /knowledgebases/<идентификатор_базы_знаний>/getAnswers
   - Host: <имя_узла>/qnamaker
   - Авторизация: EndpointKey <ключ_конечной_точки>
   
Эти три значения содержат сведения, необходимые приложению для подключения к базе знаний QnA Maker через службу Azure QnA.  

## <a name="update-the-bot-file"></a>Обновление файла .bot
Во-первых, добавьте в файл `qnamaker.bot` необходимые сведения для доступа к базе знаний, в том числе имя узла, ключ конечной точки и идентификатор базы знаний (kbId). Это значения, сохраненные из раздела **Параметры** для базы знаний в QnA Maker. 
> Примечание. Если вы добавляете доступ к базе знаний QnA Maker в существующее приложение бота, не забудьте добавить в файл .bot раздел "type": "qna", как показано ниже. Параметр name в этом разделе предоставляет ключ для доступа к этой информации из приложения.

```json
{
  "name": "qnamaker",
  "services": [
    {
      "type": "endpoint",
      "name": "development",
      "endpoint": "http://localhost:3978/api/messages",
      "appId": "",
      "appPassword": "",
      "id": "25"    
    },
    {
      "type": "qna",
      "name": "QnABot",
      "kbId": "<Your_Knowledge_Base_Id>",
      "subscriptionKey": "",
      "endpointKey": "<Your_Endpoint_Key>",
      "hostname": "<Your_Hostname>",
      "id": "117"
    }
  ],
  "padlock": "",
   "version": "2.0"
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)
Затем мы инициализируем в файле **BotServices.cs** новый экземпляр класса BotService, который извлекает перечисленные выше сведения из файла .bot. Внешняя служба настраивается с помощью класса BotConfiguration.

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

                if (string.IsNullOrWhiteSpace(qna.kbId))
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
                    KnowledgeBaseId = qna.kbId,
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

Затем в файле **QnABot.cs** мы передаем боту этот экземпляр QnAMaker. Если вы обращаетесь к собственной базе знаний, измените представленное ниже _приветственное сообщение_ на более полезное с инструкциями для пользователей. Именно в этом классе определяется и статическая переменная _QnAMakerKey_. Она указывает на раздел в файле .bot, который содержит сведения о соединении для доступа к базе знаний QnA Mkaer.

```csharp
public class QnABot : IBot
{
    public static readonly string QnAMakerKey = "QnABot";
    private const string WelcomeText = @"This bot will introduce you to QnA Maker.
                                         Ask a question to get started.";
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

В нашем примере код запуска находится в файле **index.js**, код с логикой бота — в файле **bot.js**, а дополнительные данные о конфигурации — в файле **qnamaker.bot**.

В файле **index.js** мы считываем сведения о конфигурации и используем их для создания службы QnA Maker и инициализации бота.

Обновите значение `QNA_CONFIGURATION` на значение параметра "name": в файле .bot. Это ключ, который следует разместить в разделе "type": "qna" файла .bot, где хранятся параметры для подключения к базе знаний QnA Maker.

```js
// Name of the QnA Maker service in the .bot file. 
const QNA_CONFIGURATION = '<BOT_FILE_NAME>';

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

В файле **bot.js** мы передаем пользовательский ввод в метод `getAnswers` службы QnA Maker, чтобы получать ответы из базы знаний. Если вы обращаетесь к собственной базе знаний, измените представленные ниже сообщения _об отсутствии ответов_ и _приветствия_ на более полезное с инструкциями для пользователей.

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
            const qnaResults = await this.qnaMaker.getAnswers(turnContext);

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

## <a name="test-the-bot"></a>Тестирование бота

Выполните этот пример на локальном компьютере. Если потребуются дополнительные инструкции, изучите файл readme для примера на [C#](https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/11.qnamaker) или [JS](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/javascript_nodejs/11.qnamaker/README.md).

В эмуляторе отправьте боту сообщение, как показано ниже.

![Тестирование примера QnA](~/media/emulator-v4/qna-test-bot.png)


## <a name="next-steps"></a>Дополнительная информация

QnA Maker можно объединять с другими службами Cognitive Services, чтобы сделать бота еще более мощным. Средство подготовки к отправке предоставляет способ объединения QnA с распознаванием речи (LUIS) в боте.

> [!div class="nextstepaction"]
> [Combine LUIS apps and QnA services using the Dispatch tool](./bot-builder-tutorial-dispatch.md) (Объединение приложений LUIS и служб QnA с помощью средства Dispatch)

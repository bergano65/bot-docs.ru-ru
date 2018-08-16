---
title: Использование QnA Maker | Документация Майкрософт
description: Узнайте, как использовать QnA Maker в боте.
keywords: вопрос и ответ, QnA, часто задаваемые вопросы, ПО промежуточного слоя
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78bc2c849a2c1900da33c7419693a7ff84c43cb0
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352953"
---
# <a name="how-to-use-qna-maker"></a>Использование QnA Maker

Чтобы добавить в бот поддержку простого вопроса и ответа, можно использовать службу [QnA Maker](https://qnamaker.ai/).

Одно из основных требований к написанию собственной службы QnA Maker — это задание начального значения с вопросами и ответами. Во многих случаях вопросы и ответы уже существуют в таком содержимом как часто задаваемые вопросы или в другой документации. В других случаях следует настроить ответы на вопросы в более естественной манере общения. 

## <a name="create-a-qna-maker-service"></a>Создание службы QnA Maker
Сначала создайте учетную запись и выполните вход в [QnA Maker](https://qnamaker.ai/). Затем перейдите на вкладку **Создание базы знаний**. Нажмите кнопку **Создать службу QnA** и следуйте инструкциям по созданию службы QnA Azure.

![Рис. 1 для QnA](media/QnA_1.png)

Вы будете перенаправлены на страницу [Создать QnA Maker](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesQnAMaker). Заполните форму и щелкните **Создать**.

![Рис. 2 для QnA](media/QnA_2.png)
 
После создания службы QnA на портале Azure вам будут предоставлены ключи в разделе управления ресурсами, которые можно проигнорировать. Перейдите к шагу 2, чтобы подключиться к службе QnA Azure. Обновите страницу, чтобы выбрать службу Azure, которая была только что создана, и введите имя базы знаний.

![Рис. 3 для QnA](media/QnA_3.png)

Щелкните **Создать базу знаний**. Введите свои собственные вопросы и ответы или скопируйте следующие примеры. 

![Рис. 4 для QnA](media/QnA_4.png)

Кроме того, можно выбрать **Заполнить базу знаний** и загрузить файл или URL-адрес. Пример исходного файла для создания простой службы QnA Maker можно найти [здесь](https://aka.ms/qna-tsv).

Добавив новые пары вопросов и ответов или заполнив базу знаний, щелкните **Сохранение и обучение**. После завершения на вкладке **Опубликовать** щелкните **Опубликовать**.

Чтобы подключить службу QnA к боту, потребуется строка HTTP-запроса, содержащая идентификатор базы знаний и ключ подписки QnA Maker. Скопируйте пример HTTP-запроса из результата публикации. 

![Рис. 5 для QnA](media/QnA_5.png)

## <a name="installing-packages"></a>Установка пакетов

Прежде чем приступить к кодированию, убедитесь, что имеются пакеты, необходимые для QnA Maker.

# <a name="ctabcs"></a>[C#](#tab/cs)

[Добавьте ссылку](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) на предварительную версию 4 следующих пакетов NuGet.

* `Microsoft.Bot.Builder.Ai.QnA`

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Любую из этих служб можно добавить к боту с помощью пакета botbuilder-ai. Этот пакет можно добавить в проект с помощью npm.

* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---


## <a name="using-qna-maker"></a>Использование QnA Maker

Сначала QnA Maker добавляется в качестве ПО промежуточного слоя. Затем можно использовать результаты в пределах логики бота.

# <a name="ctabcs"></a>[C#](#tab/cs)

Обновите метод `ConfigureServices` в файле `Startup.cs`, чтобы добавить объект `QnAMakerMiddleware`. Бот можно настроить для проверки базы знаний по каждому сообщению, получаемому от пользователя, просто добавив ее в стек ПО промежуточного слоя бота.


```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai.Qna;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using System;

public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        var endpoint = new QnAMakerEndpoint
        {
           KnowledgeBaseId = "YOUR-KB-ID",
           // Get the Host from the HTTP request example at https://www.qnamaker.ai
           // For GA services: https://<Service-Name>.azurewebsites.net/qnamaker
           // For Preview services: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0           
           Host = "YOUR-HTTP-REQUEST-HOST",
           EndpointKey = "YOUR-QNA-MAKER-SUBSCRIPTION-KEY"
        };
        options.Middleware.Add(new QnAMakerMiddleware(endpoint));
    });
}
```



Внесите правки в код в файле EchoBot.cs таким образом, чтобы параметр `OnTurn` отправил сообщение об откате в случае, если ПО промежуточного слоя QnA Maker не отправляет ответ на вопрос пользователя.

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

namespace Bot_Builder_Echo_Bot_QnA
{
    public class EchoBot : IBot
    {    
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {             
                if (!context.Responded)
                {
                    // QnA didn't send the user an answer
                    await context.SendActivity("Sorry, I couldn't find a good match in the KB.");

                }
            }
        }
    }
}
```


Пример бота см. [здесь](https://aka.ms/qna-cs-bot-sample).

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Сначала вставьте команду require или import в класс [QnAMaker](https://github.com/Microsoft/botbuilder-js/tree/master/doc/botbuilder-ai/classes/botbuilder_ai.qnamaker.md).

```js
const { QnAMaker } = require('botbuilder-ai');
```

Создайте `QnAMaker` путем инициализации его строкой HTTP-запроса для службы QnA. Пример запроса для службы можно скопировать из [портала QnA Maker](https://qnamaker.ai) в разделе Настройки > Сведения о развертывании.


Формат строки зависит от того, использует ли служба QnA Maker общедоступную или предварительную версию QnA Maker. Скопируйте пример HTTP-запроса и получите идентификатор базы знаний, ключ подписки и узел для использования при инициализации `QnAMaker`.

<!--
**Preview**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    "Host: https://westus.api.cognitive.microsoft.com/qnamaker/v2.0\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Ocp-Apim-Subscription-Key: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```

**GA**
```js
const qnaEndpointString = 
    // Replace xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx with your knowledge base ID
    "POST /knowledgebases/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/generateAnswer\r\n" + 
    // Replace <Service-Name> to match the Azure URL where your service is hosted
    "Host: https://<Service-Name>.azurewebsites.net/qnamaker\r\n" +
    // Replace xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx with your QnAMaker subscription key
    "Authorization: EndpointKey xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx\r\n"
const qna = new QnAMaker(qnaEndpointString);
```
-->
**Предварительный просмотр**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://westus.api.cognitive.microsoft.com/qnamaker/v2.0'
    },
    {
        // set this to true to send answers from QnA Maker
        answerBeforeNext: true
    }
);
```
**Общедоступная версия**
```js
const qna = new QnAMaker(
    {
        knowledgeBaseId: '<KNOWLEDGE-BASE-ID>',
        endpointKey: '<QNA-SUBSCRIPTION-KEY>',
        host: 'https://<Service-Name>.azurewebsites.net/qnamaker'
    },
    {
        answerBeforeNext: true
    }
);
```
Бот можно настроить на автоматический вызов службы QnA Maker, просто добавив ее в стек ПО промежуточного слоя бота.

```js
// Add QnA Maker as middleware
adapter.use(qna);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        // If `!context.responded`, that means an answer wasn't found for the user's utterance.
        // In this case, we send the user a fallback message.
        if (context.activity.type === 'message' && !context.responded) {
            await context.sendActivity('No QnA Maker answers were found.');
        } else if (context.activity.type !== 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

Инициализация `answerBeforeNext` как `true` при инициализации `QnAMaker` означает, что ПО промежуточного слоя QnA Maker автоматически реагирует при обнаружении ответа перед обращением к основной логике бота в `processActivity`. Если вместо этого задать параметру `answerBeforeNext` значение `false`, бот вызовет QnA Maker только после выполнения всей основной логики бота в `processActivity` и только в том случае, если бот не ответил пользователю. 

## <a name="calling-qna-maker-without-using-middleware"></a>Вызов QnA Maker без использования ПО промежуточного слоя

В предыдущем примере оператор `adapter.use(qna);` означает, что QnA выполняется как ПО промежуточного слоя, и поэтому отвечает на каждое сообщение, получаемое ботом. Для большего контроля над тем, как и когда вызывается QnA Maker, можно вызвать `qna.answer()` непосредственно из логики бота, а не устанавливать его как часть ПО промежуточного слоя.

Удалите оператор `adapter.use(qna);` и используйте следующий код для прямого вызова QnA Maker.

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            var handled = await qna.answer(context)
                if (!handled) {
                    await context.sendActivity(`I'm sorry. I didn't understand.`);
                }
        }
    });
});
```

Другой способ настройки QnA Maker — с помощью `qna.generateAnswer()`. Этот метод позволяет получить более подробные сведения об ответах из QnA Maker.


```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // Get all the answers QnA Maker finds
            var results = await qna.generateAnswer(context.activity.text);
                if (results && results.length > 0) {
                    await context.sendActivity(results[0].answer);
                } else {
                    await context.sendActivity(`I don't know.`);
                }
    
        }
    });
});
```
---

Задайте боту вопросы, чтобы увидеть ответы из службы QnA Maker.

![Рис. 6 для QnA](media/QnA_6.png)



## <a name="next-steps"></a>Дополнительная информация

QnA Maker можно объединять с другими службами Cognitive Services, чтобы сделать бота еще более мощным. Средство подготовки к отправке предоставляет способ объединения QnA с распознаванием речи (LUIS) в боте.

> [!div class="nextstepaction"]
> [Combine LUIS apps and QnA services using the Dispatch tool](./bot-builder-tutorial-dispatch.md) (Объединение приложений LUIS и служб QnA с помощью средства подготовки к отправке)

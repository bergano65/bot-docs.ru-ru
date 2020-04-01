---
title: Руководство по настройке бота, отвечающего на вопросы, в службе Azure Bot — Служба Azure Bot
description: Руководство по использованию QnA Maker в боте для ответов на вопросы.
keywords: QnA Maker, question and answer, knowledge base
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: tutorial
ms.service: bot-service
ms.date: 03/23/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 17d88db4e291458bc87d959c90759e0c44bcc283
ms.sourcegitcommit: 126c4f8f8c7a3581e7521dc3af9a937493e6b1df
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2020
ms.locfileid: "80499897"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>Руководство по Использование QnA Maker в боте для ответов на вопросы.

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

С помощью службы QnA Maker и базы знаний вы можете реализовать в боте поддержку вопросов и ответов. Созданная база знаний заполняется готовыми вопросами и ответами.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание службы QnA Maker и базы знаний
> * Добавление сведений о базе знаний в файл конфигурации
> * Обновление бота для запросов по базе знаний
> * Повторная публикация бота

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Бот, созданный при работе с [предыдущим руководством](bot-builder-tutorial-basic-deploy.md). Мы добавим в этот бот функцию вопросов и ответов.
* Вам будет проще, если вы уже знакомы со службой [QnA Maker](https://qnamaker.ai/). На портале QnA Maker мы создадим, обучим и опубликуем базу знаний для использования с этим ботом.
* Опыт [создания бота QnA](https://aka.ms/azure-create-qna) с помощью службы Azure Bot.

У вас также должны быть все обязательные компоненты для работы с предыдущим руководством.

## <a name="sign-in-to-qna-maker-portal"></a>Вход на портал QnA Maker

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

Войдите на [портал QnA Maker](https://qnamaker.ai/) с учетными данными Azure.

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>Создание службы QnA Maker и базы знаний

Мы импортируем существующее определение базы знаний из примера QnA Maker, размещенного в репозитории [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples).

1. Клонируйте или скопируйте на компьютер репозиторий примеров.
1. На портале QnA Maker выберите **Create a knowledge base** (Создать базу знаний).
   1. Если потребуется, создайте службу QnA. (Можно использовать существующую службу QnA Maker или создать новую специально для работы с этим руководством.) Более подробные инструкции по QnA Maker см. в руководствах по [созданию службы QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) и [созданию, подготовке и публикации базы знаний QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base).
   1. Подключите службу QnA к базе знаний.
   1. Присвойте имя базе знаний.
   1. Чтобы заполнить базу знаний, используйте файл `BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv` из репозитория примеров. Если вы уже скачали примеры, отправьте файл *smartLightFAQ.tsv* со своего компьютера.
   1. Щелкните **Create your KB** (Создать базу знаний).
1. Выполните для базы знаний действие **Save and train** (Сохранить и обучить).
1. Щелкните **Publish** (Опубликовать).

После публикации приложения QnA Maker откройте вкладку **SETTINGS** (Параметры) и прокрутите страницу вниз до раздела *Deployment Details* (Сведения о развертывании). Скопируйте следующие значения из примера HTTP-запроса *Postman*.

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL ending in /qnamaker.
Authorization: EndpointKey <qna-maker-resource-key>
```

Полная строка URL-адреса для имени узла будет выглядеть так: https://< >.azure.net/qnamaker.

Эти значения будут использоваться на следующем шаге в файле `appsettings.json` или `.env`.

Теперь база знаний готова для использования в боте.

## <a name="add-knowledge-base-information-to-your-bot"></a>Добавление сведений о базе знаний в бота

Начиная с бот-платформы версии 4.3, файлы с расширением .bot больше не предоставляются в Azure для скачивания вместе с исходным кодом бота. Следуйте дальнейшим инструкциям, чтобы подключить свой бот на C#, JavaScript или Python к базе знаний.

# <a name="c"></a>[C#](#tab/csharp)

Добавьте следующие значения в файл appsetting.json:

```json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "ScmType": "None",
  
  "QnAKnowledgebaseId": "knowledge-base-id",
  "QnAAuthKey": "qna-maker-resource-key",
  "QnAEndpointHostName": "your-hostname" // This is a URL ending in /qnamaker
}
```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

Добавьте следующие значения в файл с расширением .env:

```text
MicrosoftAppId=""
MicrosoftAppPassword=""
ScmType=None

QnAKnowledgebaseId="knowledge-base-id"
QnAAuthKey="qna-maker-resource-key"
QnAEndpointHostName="your-hostname" // This is a URL ending in /qnamaker
```

# <a name="python"></a>[Python](#tab/python)

Добавьте следующие значения в файл `config.py`:

```python
class DefaultConfig:
    """ Bot Configuration """
    PORT = 3978
    APP_ID = os.environ.get("MicrosoftAppId", "")
    APP_PASSWORD = os.environ.get("MicrosoftAppPassword", "")

    QNA_KNOWLEDGEBASE_ID = os.environ.get("QnAKnowledgebaseId", "")
    QNA_ENDPOINT_KEY = os.environ.get("QnAEndpointKey", "")
    QNA_ENDPOINT_HOST = os.environ.get("QnAEndpointHostName", "")

```

---

| Поле | Значение |
|:----|:----|
| QnAKnowledgebaseId | Идентификатор базы знаний, автоматически созданный на портале QnA Maker. |
| QnAAuthKey (QnAEndpointKey на Python)  | Ключ конечной точки, автоматически созданный на портале QnA Maker. |
| QnAEndpointHostName | URL-адрес узла, созданный на портале QnA Maker. Используйте полный формат URL-адрес, начиная с `https://` и заканчивая `/qnamaker`. Полная строка URL-адреса будет выглядеть так: https://< >.azure.net/qnamaker. |

Сохраните изменения.

## <a name="update-your-bot-to-query-the-knowledge-base"></a>Обновление бота для запросов по базе знаний

Обновите код инициализации, чтобы загрузить данные службы для вашей базы знаний.

# <a name="c"></a>[C#](#tab/csharp)

1. Добавьте в проект пакет NuGet **Microsoft.Bot.Builder.AI.QnA**.

   Это можно сделать с помощью диспетчера пакетов NuGet или из командной строки:

   ```cmd
   dotnet add package Microsoft.Bot.Builder.AI.QnA
   ```

   Дополнительные сведения о NuGet см. в [документации по NuGet](https://docs.microsoft.com/nuget/#pivot=start&panel=start-all).

1. Добавьте в проект пакет NuGet **Microsoft.Extensions.Configuration**.

1. В файл **Startup.cs** добавьте эти ссылки на пространства имен.

   **Startup.cs.**

   ```csharp
   using Microsoft.Bot.Builder.AI.QnA;
   using Microsoft.Extensions.Configuration;
   ```

1. Измените метод _ConfigureServices_. Он создает конечную точку QnAMakerEndpoint, подключающуюся к базе знаний, которая определена в файле **appsettings.json**.

   **Startup.cs.**

   ```csharp
   // Create QnAMaker endpoint as a singleton
   services.AddSingleton(new QnAMakerEndpoint
   {
      KnowledgeBaseId = Configuration.GetValue<string>($"QnAKnowledgebaseId"),
      EndpointKey = Configuration.GetValue<string>($"QnAAuthKey"),
      Host = Configuration.GetValue<string>($"QnAEndpointHostName")
    });

   ```

1. В файл **EchoBot.cs** добавьте эти ссылки на пространство имен.

   **EchoBot.cs**

   ```csharp
   using System.Linq;
   using Microsoft.Bot.Builder.AI.QnA;
   ```

1. Добавьте соединитель `EchoBotQnA` и инициализируйте его в конструкторе бота.

   **EchoBot.cs**

   ```csharp
   public QnAMaker EchoBotQnA { get; private set; }
   public EchoBot(QnAMakerEndpoint endpoint)
   {
      // connects to QnA Maker endpoint for each turn
      EchoBotQnA = new QnAMaker(endpoint);
   }
   ```

1. После метода _OnMembersAddedAsync( )_ создайте метод _AccessQnAMaker( )_ , добавив следующий код.

   **EchoBot.cs**

   ```csharp
   private async Task AccessQnAMaker(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      var results = await EchoBotQnA.GetAnswersAsync(turnContext);
      if (results.Any())
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("QnA Maker Returned: " + results.First().Answer), cancellationToken);
      }
      else
      {
         await turnContext.SendActivityAsync(MessageFactory.Text("Sorry, could not find an answer in the Q and A system."), cancellationToken);
      }
   }
   ```

1. Теперь в методе _OnMessageActivityAsync( )_ вызовите новый метод _AccessQnAMaker( )_ следующим образом:

   **EchoBot.cs**

   ```csharp
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
      await turnContext.SendActivityAsync(MessageFactory.Text($"Echo: {turnContext.Activity.Text}"), cancellationToken);

      await AccessQnAMaker(turnContext, cancellationToken);
   }
   ```

# <a name="javascript"></a>[JavaScript](#tab/javascript)

1. Откройте окно терминала или командную строку и перейдите к корневому каталогу проекта.
1. Добавьте в проект пакет npm **botbuilder-ai**.

   ```shell
   npm i botbuilder-ai
   ```

1. В файле **index.js**, после раздела // Create Adapter добавьте следующий код для считывания данных файла конфигурации с расширением .env, необходимых для создания служб QnA Maker.

   **index.js**

   ```javascript
   // Map knowledge base endpoint values from .env file into the required format for `QnAMaker`.
   const configuration = {
      knowledgeBaseId: process.env.QnAKnowledgebaseId,
      endpointKey: process.env.QnAAuthKey,
      host: process.env.QnAEndpointHostName
   };

   ```

1. Обновите код создания бота, чтобы передать сведения о конфигурации служб QnA.

   **index.js**

   ```javascript
   // Create the main dialog.
   const myBot = new MyBot(configuration, {});
   ```

1. В файл **bot.js** добавьте следующие сведения, необходимые для QnA Maker.

   **bot.js**

   ```javascript
   const { QnAMaker } = require('botbuilder-ai');
   ```

1. Измените конструктор, чтобы получить переданные параметры конфигурации, необходимые для создания соединителя QnA Maker и выдачи ошибки, если эти параметры не указаны.

   **bot.js**

   ```javascript
      class MyBot extends ActivityHandler {
         constructor(configuration, qnaOptions) {
            super();
            if (!configuration) throw new Error('[QnaMakerBot]: Missing parameter. configuration is required');
            // now create a qnaMaker connector.
            this.qnaMaker = new QnAMaker(configuration, qnaOptions);
   ```

1. Наконец обновите функцию `onMessage`, чтобы искать ответ в базе знаний. Передавайте каждый блок данных, введенных пользователем, в базу знаний QnA Maker и возвратите первый ответ QnA Maker пользователю.

    **bot.js**

    ```javascript
    this.onMessage(async (context, next) => {
        // send user input to QnA Maker.
        const qnaResults = await this.qnaMaker.getAnswers(context);

        // If an answer was received from QnA Maker, send the answer back to the user.
        if (qnaResults[0]) {
            await context.sendActivity(`QnAMaker returned response: ' ${ qnaResults[0].answer}`);
        }
        else {
            // If no answers were returned from QnA Maker, reply with help.
            await context.sendActivity('No QnA Maker response was returned.'
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        }
        await next();
    });
    ```

# <a name="python"></a>[Python](#tab/python)

1. Убедитесь, что вы установили пакеты, как описано в файле сведений репозитория примеров.
1. Добавьте ссылку на `botbuilder-ai` в файл `requirements.txt`, как показано ниже.

   **requirements.txt**
   <!-- Removed version numbers -->
   ```text
      botbuilder-core
      botbuilder-ai
      flask
   ```

   Обратите внимание, что версии могут отличаться.

1. В файле `app.py` измените создание экземпляра бота, как показано ниже.

   **app.py**

   ```python
   # Create the main dialog
   BOT = MyBot(APP.config)
   ```

1. В файле `bot.py` импортируйте `QnAMaker` и `QnAMakerEndpoint`; также импортируйте `Config`, как показано ниже.

   **bot.py**

   ```python
   from flask import Config

   from botbuilder.ai.qna import QnAMaker, QnAMakerEndpoint
   from botbuilder.core import ActivityHandler, MessageFactory, TurnContext
   from botbuilder.schema import ChannelAccount
   ```

1. Добавьте функцию __init__ для создания экземпляра объекта `qna-maker`. использование параметров конфигурации, предоставленных в файле `config.py`.  

   **bot.py**

   ```python
   def __init__(self, config: Config):
      self.qna_maker = QnAMaker(
         QnAMakerEndpoint(
            knowledge_base_id=config["QNA_KNOWLEDGEBASE_ID"],
            endpoint_key=config["QNA_ENDPOINT_KEY"],
            host=config["QNA_ENDPOINT_HOST"],
      )
   )

   ```

1. Обновите `on_message_activity` для создания запроса к базе знаний и получения ответа. Передавайте каждый блок данных, введенных пользователем, в базу знаний QnA Maker и возвратите первый ответ QnA Maker пользователю.

   **bot.py**

   ```python
   async def on_message_activity(self, turn_context: TurnContext):
      # The actual call to the QnA Maker service.
      response = await self.qna_maker.get_answers(turn_context)
      if response and len(response) > 0:
         await turn_context.send_activity(MessageFactory.text(response[0].answer))
      else:
         await turn_context.send_activity("No QnA Maker answers were found.")

   ```

1. При необходимости обновите приветственное сообщение в `on_members_added_activity`, например:

   **bot.py**

   ```python
   await turn_context.send_activity("Hello and welcome to QnA!")
   ```

---

### <a name="test-the-bot-locally"></a>Локальная проверка бота

На этом этапе бот уже может отвечать на некоторые вопросы. Запустите бот на локальном компьютере и откройте его в эмуляторе.

![Тестирование примера QnA](./media/qna-test-bot.png)

## <a name="republish-your-bot"></a>Повторная публикация бота

Теперь можно повторно опубликовать бота в Azure. Для этого необходимо заархивировать папку проекта и выполнить команду развертывания. Дополнительные сведения см. в статье [о развертывании бота](https://docs.microsoft.com/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=csharp).

[!INCLUDE [Work around for .NET Core 3.1 SDK](~/includes/deploy/samples-workaround-3-1.md)]

### <a name="zip-your-project-folder"></a>Архивация папки проекта

[!INCLUDE [zip up code](~/includes/deploy/snippet-zip-code.md)]

<!-- > [!IMPORTANT]
> Before creating a zip of your project files, make sure that you are _in_ the correct folder. 
> - For C# bots, it is the folder that has the .csproj file. 
> - For JS bots, it is the folder that has the app.js or index.js file. 
> - For Python bots, it is the folder that has the app.py file. 
>
> Select all the files and zip them up while in that folder, then run the command while still in that folder.
>
> If your root folder location is incorrect, the **bot will fail to run in the Azure portal**. -->

### <a name="deploy-your-code-to-azure"></a>Развертывание кода в Azure

[!INCLUDE [deploy code to Azure](~/includes/deploy/snippet-deploy-code-to-az.md)]

<!-- # [C#](#tab/csharp)
```cmd
az webapp deployment source config-zip --resource-group "resource-group-name" --name "bot-name-in-azure" --src "c:\bot\mybot.zip"
```

# [JavaScript](#tab/javascript)

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish-js.md)]

# [Python](#tab/python)

az webapp deployment source config-zip --resource-group "resource_group_name" --name "unique_bot_name" --src "zi

### Test the published bot

After you publish the bot, give Azure a minute or two to update and start the bot.

Use the Emulator to test the production endpoint for your bot, or use the Azure portal to test the bot in Web Chat.
In either case, you should see the same behavior as you did when you tested it locally.

## Clean up resources

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

Если вы не собираетесь использовать это приложение в дальнейшем, удалите все связанные ресурсы, выполнив следующие действия.

1. На портале Azure откройте группу ресурсов бота.
2. Щелкните **Удалить группу ресурсов**. Одновременно с группой ресурсов удаляются все содержащиеся в ней ресурсы.
3. На панели подтверждения введите имя группы ресурсов и щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

См. подробнее о сведения о том, как добавлять в бот новые функции, в инструкциях по **отправке и получению текстовых сообщений** и других руководствах по разработке.
> [!div class="nextstepaction"]
> [Отправка и получение текстовых сообщений](bot-builder-howto-send-messages.md)

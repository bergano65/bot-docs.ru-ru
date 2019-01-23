---
title: Руководство по настройке бота, отвечающего на вопросы, в службе Azure Bot | Документация Майкрософт
description: Руководство по использованию QnA Maker в боте для ответов на вопросы.
keywords: QnA Maker, question and answer, knowledge base
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: tutorial
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b1c34531ee60b2ce9037f42e4f5a7093501cf83a
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360969"
---
# <a name="tutorial-use-qna-maker-in-your-bot-to-answer-questions"></a>Руководство. Использование QnA Maker в боте для ответов на вопросы.

С помощью службы QnA Maker и базы знаний вы можете реализовать в боте поддержку вопросов и ответов. Созданная база знаний заполняется готовыми вопросами и ответами.

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание службы QnA Maker и базы знаний
> * Добавление сведений о базе знаний в файл .bot
> * Обновление бота для запросов по базе знаний
> * Повторная публикация бота

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="prerequisites"></a>Предварительные требования

* Бот, созданный при работе с [предыдущим руководством](bot-builder-tutorial-basic-deploy.md). Мы добавим в этот бот функцию вопросов и ответов.
* Вам будет проще, если вы уже знакомы со службой QnA Maker. На портале QnA Maker мы создадим, обучим и опубликуем базу знаний для использования с этим ботом.

Скорее всего, у вас уже готовы все обязательные компоненты для работы с предыдущим руководством.

[!INCLUDE [deployment prerequisites snippet](~/includes/deploy/snippet-prerequisite.md)]

## <a name="sign-in-to-qna-maker-portal"></a>Вход на портал QnA Maker

<!-- This and the next step are close duplicates of what's in the QnA How-To -->

Войдите на [портал QnA Maker](https://qnamaker.ai/) с учетными данными Azure.

## <a name="create-a-qna-maker-service-and-knowledge-base"></a>Создание службы QnA Maker и базы знаний

Мы импортируем существующее определение базы знаний из примера QnA Maker, размещенного в репозитории [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples).

1. Клонируйте или скопируйте на компьютер репозиторий примеров.
1. На портале QnA Maker выберите **Create a knowledge base** (Создать базу знаний).
   1. Если потребуется, создайте службу QnA. (Можно использовать существующую службу QnA Maker или создать новую специально для работы с этим руководством.) Более подробные инструкции по QnA Maker см. в руководствах по [созданию службы QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure) и [созданию, подготовке и публикации базы знаний QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base).
   1. Подключите службу QnA к базе знаний.
   1. Присвойте имя базе знаний.
   1. Чтобы заполнить базу знаний, используйте файл **BotBuilder-Samples\samples\csharp_dotnetcore\11.qnamaker\CognitiveModels\smartLightFAQ.tsv** из репозитория примеров.
   1. Щелкните **Create your KB** (Создать базу знаний).
1. Выполните для базы знаний действие **Save and train** (Сохранить и обучить).
1. Щелкните **Publish** (Опубликовать).

   Теперь база знаний готова для использования в боте. Запишите идентификатор базы знаний, ключ конечной точки и имя узла. Эти значения потребуются для следующего шага.

## <a name="add-knowledge-base-information-to-your-bot-file"></a>Добавление сведений о базе знаний в файл .bot

Добавьте в файл .bot сведения, необходимые для доступа к базе знаний.

1. Откройте файл .bot в редакторе.
1. Добавьте элемент `qna` в массив `services`.

    ```json
    {
        "type": "qna",
        "name": "<your-knowledge-base-name>",
        "kbId": "<your-knowledge-base-id>",
        "hostname": "<your-qna-service-hostname>",
        "endpointKey": "<your-knowledge-base-endpoint-key>",
        "subscriptionKey": "<your-azure-subscription-key>",
        "id": "<a-unique-id>"
    }
    ```

    | Поле | Значение |
    |:----|:----|
    | Тип | Этот параметр должен содержать значение `qna`. Оно означает, что эта запись службы описывает базу знаний QnA. |
    | name | Имя, которое вы назначили для базы знаний. |
    | kbId | Идентификатор базы знаний, автоматически созданный на портале QnA Maker. |
    | hostname | URL-адрес узла, созданный на портале QnA Maker. Используйте полный формат URL-адрес, начиная с `https://` и заканчивая `/qnamaker`. |
    | endpointKey | Ключ конечной точки, автоматически созданный на портале QnA Maker. |
    | subscriptionKey | Идентификатор подписки, которую вы использовали при создании службы QnA Maker в Azure. |
    | id | Уникальный идентификатор, который еще не используется службами, перечисленными в файле .bot (например, 201). |

1. Сохраните изменения.

## <a name="update-your-bot-to-query-the-knowledge-base"></a>Обновление бота для запросов по базе знаний

Обновите код инициализации, чтобы загрузить данные службы для вашей базы знаний.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Добавьте в проект пакет NuGet **Microsoft.Bot.Builder.AI.QnA**.
1. Присвойте классу, в котором реализован **IBot**, новое имя `QnaBot`.
1. Присвойте классу, который содержит методы доступа для бота, новое имя `QnaBotAccessors`.
1. В файл **Startup.cs** добавьте эти ссылки на пространства имен.
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Bot.Builder.AI.QnA;
    using Microsoft.Bot.Builder.Integration;
    ```
1. И, наконец, измените метод **ConfigureServices**, чтобы он инициализировал и регистрировал базы знаний, определенные в файле **.bot**. Обратите внимание, что эти первые несколько строк текста вызова `services.AddBot<QnaBot>(options =>` находятся перед ним.
    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        var secretKey = Configuration.GetSection("botFileSecret")?.Value;
        var botFilePath = Configuration.GetSection("botFilePath")?.Value;

        // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
        var botConfig = BotConfiguration.Load(botFilePath ?? @".\jfEchoBot.bot", secretKey);
        services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

        // Initialize the QnA knowledge bases for the bot.
        services.AddSingleton(sp => {
            var qnaServices = new List<QnAMaker>();
            foreach (var qnaService in botConfig.Services.OfType<QnAMakerService>())
            {
                qnaServices.Add(new QnAMaker(qnaService));
            }
            return qnaServices;
        });

        services.AddBot<QnaBot>(options =>
        {
            // Retrieve current endpoint.
            // ...
        });

        // Create and register state accessors.
        // ...
    }
    ```
1. В файл **QnaBot.cs** добавьте эти ссылки на пространства имен.
    ```csharp
    using System.Collections.Generic;
    using Microsoft.Bot.Builder.AI.QnA;
    ```
1. Добавьте свойство `_qnaServices` и инициализируйте его в конструкторе бота.
    ```csharp
    private readonly List<QnAMaker> _qnaServices;

    /// ...
    public QnaBot(QnaBotAccessors accessors, List<QnAMaker> qnaServices, ILoggerFactory loggerFactory)
    {
        // ...
        _qnaServices = qnaServices;
    }
    ```
1. Измените обработчик шагов, чтобы он проверял введенные пользователем данные во всех зарегистрированных базах знаний. Когда боту потребуется ответ от QnAMaker, вызовите из кода бота метод `GetAnswersAsync`, чтобы получить ответ с учетом текущего контекста. Если вы обращаетесь к собственной базе знаний, измените представленное ниже _сообщение об отсутствии ответов_ на более полезное с инструкциями для пользователей.
    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            foreach(var qnaService in _qnaServices)
            {
                var response = await qnaService.GetAnswersAsync(turnContext);
                if (response != null && response.Length > 0)
                {
                    await turnContext.SendActivityAsync(
                        response[0].Answer,
                        cancellationToken: cancellationToken);
                    return;
                }
            }

            var msg = "No QnA Maker answers were found. This example uses a QnA Maker knowledge base that " +
                "focuses on smart light bulbs. Ask the bot questions like 'Why won't it turn on?' or 'I need help'.";

            await turnContext.SendActivityAsync(msg, cancellationToken: cancellationToken);
        }
        else
        {
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
        }
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Откройте окно терминала или командную строку и перейдите к корневому каталогу проекта.
1. Добавьте в проект пакет npm **botbuilder-ai**.
    ```shell
    npm i botbuilder-ai
    ```
1. В файл **index.js** добавьте эту инструкцию require.
    ```javascript
    const { QnAMaker } = require('botbuilder-ai');
    ```
1. Получите сведения о конфигурации для создания служб QnA Maker.
    ```javascript
    // Read bot configuration from .bot file.
    // ...

    // Initialize the QnA knowledge bases for the bot.
    // Assume each QnA entry in the .bot file is well defined.
    const qnaServices = [];
    botConfig.services.forEach(s => {
        if (s.type == 'qna') {
            const endpoint = {
                knowledgeBaseId: s.kbId,
                endpointKey: s.endpointKey,
                host: s.hostname
            };
            const options = {};
            qnaServices.push(new QnAMaker(endpoint, options));
        }
    });

    // Get bot endpoint configuration by service name
    // ...
    ```
1. Обновите код создания бота, чтобы передавать эти службы QnA.
    ```javascript
    // Create the bot.
    const myBot = new MyBot(qnaServices);
    ```
1. В файл **bot.js** добавьте конструктор.
    ```javascript
    constructor(qnaServices) {
        this.qnaServices = qnaServices;
    }
    ```
1. Теперь обновите обработчик шагов, чтобы искать ответ в базе знаний.
    ```javascript
    async onTurn(turnContext) {
        if (turnContext.activity.type === ActivityTypes.Message) {
            for (let i = 0; i < this.qnaServices.length; i++) {
                // Perform a call to the QnA Maker service to retrieve matching Question and Answer pairs.
                const qnaResults = await this.qnaServices[i].getAnswers(turnContext);

                // If an answer was received from QnA Maker, send the answer back to the user and exit.
                if (qnaResults[0]) {
                    await turnContext.sendActivity(qnaResults[0].answer);
                    return;
                }
            }
            // If no answers were returned from QnA Maker, reply with help.
            await turnContext.sendActivity('No QnA Maker answers were found. '
                + 'This example uses a QnA Maker Knowledge Base that focuses on smart light bulbs. '
                + `Ask the bot questions like "Why won't it turn on?" or "I need help."`);
        } else {
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }
    ```

---

### <a name="test-the-bot-locally"></a>Локальная проверка бота

На этом этапе бот уже может отвечать на некоторые вопросы. Запустите бот на локальном компьютере и откройте его в эмуляторе.

![Тестирование примера QnA](~/media/emulator-v4/qna-test-bot.png)

## <a name="re-publish-your-bot"></a>Повторная публикация бота

Мы теперь готовы повторно опубликовать бот.

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

### <a name="test-the-published-bot"></a>Тестирование опубликованного бота

После публикации бота подождите пару минут, чтобы обновить и запустить бот в Azure.

1. С помощью эмулятора протестируйте рабочую конечную точку бота откройте портал Azure для тестирования бота через веб-чат.

   В обоих случаях поведение бота должно полностью совпадать с поведением при локальном тестировании.

## <a name="clean-up-resources"></a>Очистка ресурсов

<!-- In the first tutorial, we should tell them to use a new resource group, so that it is easy to clean up resources. We should also mention in this step in the first tutorial not to clean up resources if they are continuing with the sequence. -->

Если вы не собираетесь использовать это приложение в дальнейшем, удалите все связанные ресурсы, выполнив следующие действия.

1. На портале Azure откройте группу ресурсов бота.
1. Щелкните **Удалить группу ресурсов**. Одновременно с группой ресурсов удаляются все содержащиеся в ней ресурсы.
1. На панели подтверждения введите имя группы ресурсов и щелкните **Удалить**.

## <a name="next-steps"></a>Дополнительная информация

См. дополнительные сведения о том, как добавлять в бот новые функции, в руководствах по разработке.
> [!div class="nextstepaction"]
> [Кнопка дальнейших действий](bot-builder-howto-send-messages.md)

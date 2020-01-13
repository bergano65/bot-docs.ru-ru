---
title: Использование QnA Maker для ответов на вопросы | Документация Майкрософт
description: Узнайте, как использовать QnA Maker в боте.
keywords: question and answer, QnA, FAQs, qna maker
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fc22235e53307e5b1dde737930c6cd06c2e9df8a
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491817"
---
# <a name="use-qna-maker-to-answer-questions"></a>Использование QnA Maker для ответов на вопросы

[!INCLUDE[applies-to](../includes/applies-to.md)]

QnA Maker создает слой вопросов и ответов для диалога на основе ваших данных. Это позволяет вашему роботу отправить вопрос в QnA Maker и получить ответ без необходимости анализировать и интерпретировать цель вопроса.

Одна из основных сложностей при создании собственной службы QnA Maker — заполнить ее начальным набором вопросов и ответов. Во одних случаях вопросы и ответы уже существуют в таком содержимом, как разделы с вопросами и ответами или другая документация. В других случаях вы можете настроить ответы на вопросы в более естественном разговорном стиле.

## <a name="prerequisites"></a>предварительные требования

- Код в этой статье основан на примере QnA Maker. Вам потребуется копия примера на **[C#](https://aka.ms/cs-qna)** , **[JavaScript](https://aka.ms/js-qna-sample)** или **[Python](https://aka.ms/bot-qna-python-sample-code)** . 
- Учетная запись [QnA Maker](https://www.qnamaker.ai/)
- Знания о [работе ботов](bot-builder-basics.md), [QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/overview/overview) и [управлении ресурсами бота](bot-file-basics.md).

## <a name="about-this-sample"></a>Об этом примере

Чтобы использовать QnA Maker в боте, создайте базу знаний на портале [QnA Maker](https://www.qnamaker.ai/), как показано в следующем разделе. Затем бот может отправить вопросы пользователя в QnA Maker, предоставляющий лучшие ответы.

## <a name="ctabcs"></a>[C#](#tab/cs)

![Поток логики QnABot — C#](./media/qnabot-logic-flow.png)

`OnMessageActivityAsync` вызывается для каждого полученного блока данных, введенных пользователем. Этот метод обращается к информации `_configuration`, сохраненной в файле `appsetting.json` этого примера кода. При этом выполняется поиск значения для подключения к предварительно настроенной базе знаний QnA Maker.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![Поток логики QnABot — JavaScript](./media/qnabot-js-logic-flow.png)

`OnMessage` вызывается для каждого полученного блока данных, введенных пользователем. Этот метод обращается к соединителю `qnamaker`, который был предварительно настроен на основе значений, указанных в файле `.env` этого примера кода.  В QnA Maker метод `getAnswers` позволяет подключить бота к внешней базе знаний QnA Maker.

## <a name="pythontabpython"></a>[Python](#tab/python)

![Поток логики QnABot — JavaScript](./media/qnabot-python-logic-flow.png)

`on_message_activity` вызывается для каждого полученного блока данных, введенных пользователем. Этот метод обращается к соединителю `qna_maker`, который был предварительно настроен на основе значений, указанных в файле `config.py` этого примера кода.  Метод `qna_maker.getAnswers` позволяет подключить бот к внешней базе знаний QnA Maker.

---

Введенные пользователем данные передаются в эту базу знаний, а наиболее точный полученный ответ отображается для пользователя.

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>Создание службы QnA Maker и публикация базы знаний

Прежде всего создайте службу QnA Maker. Выполните действия, описанные в [документации по QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure), чтобы создать службу в Azure.

После этого создайте базу знаний на основе файла `smartLightFAQ.tsv`, который расположен в папке CognitiveModels этого примера проекта. Действия по созданию, обучению и публикации [базы знаний](https://docs.microsoft.com/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base) QnA Maker описаны в документации по QnA Maker. При выполнении этих действий присвойте базе знаний имя `qna` и выберите для ее заполнения файл `smartLightFAQ.tsv`.

> Примечание. Инструкции из этой статьи также помогут получить доступ к собственной базе знаний QnA Maker, разработанной пользователем.

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>Получение значений для подключения бота к базе знаний

1. На сайте [QnA Maker](https://www.qnamaker.ai/) выберите свою базу знаний.
1. При открытой базе знаний выберите **Параметры**. Сохраните значение из поля _Имя службы_. Это значение используется для поиска нужной базы знаний в интерфейсе портала QnA Maker. Оно не используется для подключения к этой базе знаний из приложения бота.
1. Прокрутите вниз, чтобы найти **сведения о развертывании**, и запишите следующие значения из примера HTTP-запроса Postman:
   - POST /knowledgebases/\<идентификатор_базы_знаний>/generateAnswer
   - Узел: \<имя_узла> // полный URL-адрес, заканчивающийся /qnamaker
   - Авторизация: EndpointKey \<ключ_конечной_точки>

Полная строка URL-адреса для имени узла будет выглядеть так: https://< >.azure.net/qnamaker. Эти три значения будут содержать сведения, необходимые приложению для подключения к базе знаний QnA Maker через службу Azure QnA.  

## <a name="update-the-settings-file"></a>Обновление файла параметров

Во-первых, добавьте в файл параметров сведения, необходимые для доступа к базе знаний: имя узла, ключ конечной точки и идентификатор базы знаний (kbId). Это значения, полученные на вкладке **Settings** (Параметры) базы знаний в QnA Maker.

Если это развертывание не предназначено для рабочей среды, поля идентификатора приложения и пароля можно оставить пустыми.

> [!NOTE]
> Если вы реализуете доступ к базе знаний QnA Maker в существующем приложении бота, обязательно создайте понятные описательные заголовки для объектов QnA. Параметр name в этом разделе предоставляет ключ для доступа к этой информации из приложения.

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="update-your-appsettingsjson-file"></a>Обновление файла appsettings.json

[!code-csharp[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/appsettings.json)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="update-your-env-file"></a>Обновление файла .env

[!code-javascript[.env file](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/.env)]

## <a name="pythontabpython"></a>[Python](#tab/python)

### <a name="update-your-configpy-file"></a>Обновление файла config.py

[!code-python[config.py](~/../botbuilder-python/samples/python/11.qnamaker/config.py?range=10-18)]

---

## <a name="set-up-the-qna-maker-instance"></a>Настройка экземпляра QnA Maker

Для начала мы создадим объект для доступа к базе знаний QnA Maker.

## <a name="ctabcs"></a>[C#](#tab/cs)

Убедитесь, что пакет NuGet **Microsoft.Bot.Builder.AI.QnA** установлен для вашего проекта.

В методе `OnMessageActivityAsync` в файле **QnABot.cs** мы создадим экземпляр QnAMaker. Класс `QnABot` извлекает имена для сведений о подключении, сохраненные выше в `appsettings.json`. Если вы выбрали другие имена для сведений о подключении к базе знаний в файле параметров, обязательно обновите имена в этом методе, чтобы они соответствовали выбранным.

**Bots/QnABot.cs**

[!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=32-39)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Убедитесь, что для проекта установлен пакет npm **botbuilder-ai**.

В нашем примере код для логики бота находится в файле **QnABot.js**.

В файле **QnABot.js** мы используем сведения о подключении, предоставленные в файле .env, чтобы подключиться к службе QnA Maker: _this.qnaMaker_.

**bots/QnABot.js**

[!code-javascript[QnAMaker](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=12-16)]

## <a name="pythontabpython"></a>[Python](#tab/python)

В файле **qna_bot.py** мы используем сведения о подключении, предоставленные в файле `config.py`, чтобы подключиться к службе QnA Maker: `self.qna_maker`.

**bots/qna_bot.py** [!code-python[QnAMaker](~/../botbuilder-python/samples/python/11.qnamaker/bots/qna_bot.py?range=13-19)]

---

## <a name="calling-qna-maker-from-your-bot"></a>Вызов QnA Maker из кода бота

## <a name="ctabcs"></a>[C#](#tab/cs)

Когда боту потребуется ответ от QnAMaker, вызовите из кода бота метод `GetAnswersAsync()`, чтобы получить ответ с учетом текущего контекста. Если вы обращаетесь к собственной базе знаний, измените представленное ниже _сообщение об отсутствии ответов_ на более информативное для пользователей.

**Bots/QnABot.cs**

[!code-csharp[qna get answers](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=43-52)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В файле **QnABot.js** мы передадим введенные пользователем данные в метод `getAnswers` службы QnA Maker, чтобы получить ответы из базы знаний. Если QnA Maker возвращает ответ, он отображается для пользователя. В противном случае пользователь получает сообщение No QnA Maker answers were found (Ответы в QnA Maker не найдены).

**bots/QnABot.js**

[!code-javascript[OnMessage](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=46-55)]

## <a name="pythontabpython"></a>[Python](#tab/python)

В файле **qna_bot.py** мы передадим введенные пользователем данные в метод `get_answers` службы QnA Maker, чтобы получить ответы из базы знаний. Если QnA Maker возвращает ответ, он отображается для пользователя. В противном случае пользователь получает сообщение *No QnA Maker answers were found* (Ответы в QnA Maker не найдены).

**bots/qna_bot.py** [!code-python[get_answers](~/../botbuilder-python/samples/python/11.qnamaker/bots/qna_bot.py?range=33-37)]

---

## <a name="test-the-bot"></a>Тестирование бота

Выполните этот пример на локальном компьютере. Установите [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download), если вы этого еще не сделали. См. подробнее в файле README для [C#](https://aka.ms/cs-qna) и [JavaScript](https://aka.ms/js-qna-sample).
или [пример на языке Python](https://aka.ms/bot-qna-python-sample-code). 

Запустите эмулятор, подключитесь к боту и отправьте сообщение, как показано ниже.

![Тестирование примера QnA](../media/emulator-v4/qna-test-bot.png)

## <a name="additional-information"></a>Дополнительные сведения

### <a name="multi-turn-prompts"></a>Многоэтапные подсказки

QnA Maker поддерживает дополнительные (многоэтапные) подсказки.
Если база знаний QnA Maker требует дополнительных сведений от пользователя, QnA Maker отправляет сведения о контексте, которые можно использовать при отправке запросов пользователю. Эти сведения затем используются для дополнительных обращений к службе QnA Maker.
Поддержка этой функции добавлена в пакет SDK для Bot Framework версии 4.6.

Чтобы создать такую базу знаний, воспользуйтесь статьей [Use follow-up prompts to create multiple turns of a conversation](https://aka.ms/qnamaker-multiturn-conversation) (Использование дальнейших подсказок для создания диалога с несколькими шагами) из документации по QnA Maker. <!--To learn how to incorporate multi-turn support in your bot, take a look at the QnA Maker Multi-turn [[**C#**](https://aka.ms/cs-qna-multiturn) | [**JS**](https://aka.ms/js-qna-multiturn)] sample.-->

<!--TODO: Update code based on final sample 
The following code snippets come from the proof-of-concept **multi-turn QnA Maker prompts** sample for
[**C#**](https://github.com/microsoft/BotBuilder-Samples/tree/master/experimental/qnamaker-prompting/csharp_dotnetcore) and
[**JavaScript**](https://github.com/microsoft/BotBuilder-Samples/tree/master/experimental/qnamaker-prompting/javascript_nodejs).
-->
<!--
#### Sample code

This sample uses a custom QnA dialog to track state for QnA Maker and handle the user's input and QnA Maker's response. When the user sends a message to the bot, the bot treats the input as either an initial query or a response to a follow-up question. The bot starts or continues its QnA dialog, which tracks the QnA Maker context information.

1. When the dialog **starts**, it makes an initial call to the QnA Maker service. QnA Maker context information **is not** included in the call.
1. When the dialog **continues**, it makes a follow-up call to the QnA Maker service. QnA Maker context information included **is** included in the call.
1. In either case, the dialog evaluates the query results.
   - Each QnA Maker result includes either an answer or a follow-up question to the user's initial query. (This sample uses only the first result.)
   - If the result includes follow-up prompts, the dialog prompts the user, saves the context information, and stays on the dialog stack, waiting for additional information from the user.
   - Otherwise, the result represents an answer, the dialog sends the answer and ends.

This sample implements this across two dialog classes:

- The base _functional dialog_ defines the begin, continue, and state logic for the dialog, notably, the _run state machine_ method.
- The derived _QnA dialog_ defines the logic to call QnA Maker, evaluate its response, and send an answer or follow-up question to the user.

#### QnA Maker context

You need to track context information for the QnA Maker service.
The dialog funnels incoming activities through its _run state machine_ method, which:

1. Gets previous QnA Maker context, if any, from state.
1. Calls its _process_ method to call QnA Maker and generates a response for the user.
1. If the result was a follow-up question, sends the question, saves new QnA Maker context to state, and waits for more input on the next turn.
1. If the result was an answer, sends the answer and ends the dialog.

##### [C#](#tab/csharp)

**Dialogs\FunctionDialogBase.cs** defines the **RunStateMachineAsync** method.

```csharp
private async Task<DialogTurnResult> RunStateMachineAsync(DialogContext dialogContext, CancellationToken cancellationToken)
{
     // Get the Process function's current state from the dialog state
     var oldState = GetPersistedState(dialogContext.ActiveDialog);

     // Run the Process function.
     var (newState, output, result) = await ProcessAsync(oldState, dialogContext.Context.Activity).ConfigureAwait(false);

     // If we have output to send then send it.
     foreach (var activity in output)
     {
          await dialogContext.Context.SendActivityAsync(activity).ConfigureAwait(false);
     }

     // If we have new state then we must still be running.
     if (newState != null)
     {
          // Add the state returned from the Process function to the dialog state.
          dialogContext.ActiveDialog.State[FunctionStateName] = newState;

          // Return Waiting indicating this dialog is still in progress.
          return new DialogTurnResult(DialogTurnStatus.Waiting);
     }
     else
     {
          // The Process function indicates it's completed by returning null for the state.
          return await dialogContext.EndDialogAsync(result).ConfigureAwait(false);
     }
}
```

##### [JavaScript](#tab/javascript)

**dialogs/functionDialogBase.js** defines the **runStateMachine** method.

```javascript
async runStateMachine(dc) {

     var oldState = this.getPersistedState(dc.activeDialog);

     var processResult = await this.processAsync(oldState, dc.context.activity);

     var newState = processResult[0];
     var output = processResult[1];
     var result = processResult[2];

     await dc.context.sendActivity(output);

     if(newState != null){
          dc.activeDialog.state[functionStateName] = newState;
          return { status: DialogTurnStatus.waiting };
     }
     else{
          return await dc.endDialog();
     }
}
```

---

#### QnA Maker input and response

You need to provide any previous context information when you call the QnA Maker service.
The dialog handles the call to QnA Maker in its _process_ method, which:

1. Makes the call to QnA Maker, passing in the previous context, if any.
   - This sample uses a _query QnA service_ helper method to format the parameters and make the call.
   - Importantly, if this is a follow-up call to QnA Maker, the _QnA Maker options_ should include values for the _QnA request context_ and the _QnA question ID_.
1. Gets QnA Maker's response and any follow-up prompts from the top result.
1. If the result was a follow-up question:
   - Generates a hero card that contains the question and options for the user's response.
   - Includes the hero card in a message activity.
   - Returns the message activity and the new QnA Maker context.
1. If the result was an answer, returns a message activity that contains the answer.

You can format a follow-up activity in many ways. This sample uses a hero card. However, using suggested actions would be another option.

##### [C#](#tab/csharp)

**Dialogs\QnADialog.cs** defines the **RunStateMachineAsync** method.

```csharp
protected override async Task<(object newState, IEnumerable<Activity> output, object result)> ProcessAsync(object oldState, Activity inputActivity)
{
     Activity outputActivity = null;
     QnABotState newState = null;

     var query = inputActivity.Text;
     var qnaResult = await _qnaService.QueryQnAServiceAsync(query, (QnABotState)oldState);
     var qnaAnswer = qnaResult[0].Answer;
     var prompts = qnaResult[0].Context?.Prompts;

     if (prompts == null || prompts.Length < 1)
     {
          outputActivity = MessageFactory.Text(qnaAnswer);
     }
     else
     {
          // Set bot state only if prompts are found in QnA result
          newState = new QnABotState
          {
          PreviousQnaId = qnaResult[0].Id,
          PreviousUserQuery = query
          };

          outputActivity = CardHelper.GetHeroCard(qnaAnswer, prompts);
     }

     return (newState, new Activity[] { outputActivity }, null);
}
```

##### [JavaScript](#tab/javascript)

**dialogs/qnaDialog.js** defines the **runStateMachine** method.

```javascript
async processAsync(oldState, activity){

     var newState = null;
     var query = activity.text;
     var qnaResult = await QnAServiceHelper.queryQnAService(query, oldState);
     var qnaAnswer = qnaResult[0].answer;

     var prompts = null;
     if(qnaResult[0].context != null){
          prompts = qnaResult[0].context.prompts;
     }

     var outputActivity = null;
     if(prompts == null || prompts.length < 1){
          outputActivity = MessageFactory.text(qnaAnswer);
     }
     else{
          var newState = {
               PreviousQnaId: qnaResult[0].id,
               PreviousUserQuery: query
          }

          outputActivity = CardHelper.GetHeroCard(qnaAnswer, prompts);
     }

     return [newState, outputActivity , null];
}  
```

---
-->
## <a name="next-steps"></a>Дальнейшие действия

QnA Maker можно объединять с другими службами Cognitive Services, чтобы сделать бота еще более мощным. Средство подготовки к отправке предоставляет способ объединения QnA с распознаванием речи (LUIS) в боте.

> [!div class="nextstepaction"]
> [Combine LUIS apps and QnA services using the Dispatch tool](./bot-builder-tutorial-dispatch.md) (Объединение приложений LUIS и служб QnA с помощью средства Dispatch)

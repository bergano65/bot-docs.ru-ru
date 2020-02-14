---
title: Отправка и получение текстовых сообщений — Служба Azure Bot
description: Узнайте, как отправлять и принимать текстовые сообщения с помощью пакета SDK Bot Framework.
keywords: sending message, message activities, simple text message, message, text message, receive message
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9dc5bfeab8bc56e81888be5e9463be167fcd2b18
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071812"
---
# <a name="send-and-receive-text-message"></a>Отправка и получение текстовых сообщений

[!INCLUDE[applies-to](../includes/applies-to.md)]

Основным способом взаимодействия бота с пользователями будут действия с **сообщениями**. Некоторые сообщения будут состоять из обычного текста, а другие — включать расширенное содержимое, такое как карточки или вложения. Обработчик этапа бота получает сообщения от пользователя, и вы можете отправлять ответы пользователю из него. Объект контекста этапа предоставляет методы для отправки сообщений обратно пользователю. В этой статье объясняется, как отправлять простые текстовые сообщения.

Markdown поддерживается для большинства текстовых полей, но особенности поддержки зависят от канала.

Для выполняющегося бота, отправляющего и получающего сообщения, выполните краткие инструкции в верхней части содержания или ознакомьтесь со [статьей о работе ботов](bot-builder-basics.md#bot-structure), в которой также содержатся ссылки на простые примеры, которые вы можете выполнить самостоятельно.

## <a name="send-a-text-message"></a>Отправка текстового сообщения

Для отправки простого текстового сообщения укажите строку, которую вы хотите отправить как действие:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В обработчиках действий бота используйте метод `SendActivityAsync` для объекта контекста шага, чтобы отправить ответ в виде одного сообщения. Кроме того, вы можете использовать метод `SendActivitiesAsync` объекта для отправки нескольких ответов за раз.

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В обработчиках действий бота используйте метод `sendActivity` для объекта контекста шага, чтобы отправить ответ в виде одного сообщения. Кроме того, вы можете использовать метод `sendActivities` объекта для отправки нескольких ответов за раз.

```javascript
await context.sendActivity("Welcome!");
```

# <a name="pythontabpython"></a>[Python](#tab/python)

В обработчиках действий бота используйте метод `send_activity` для объекта контекста шага, чтобы отправить ответ в виде одного сообщения.

```python
await turn_context.send_activity("Welcome!")
```

---
## <a name="receive-a-text-message"></a>Получение текстового сообщения

Чтобы получить простое текстовое сообщение, используйте свойство *text* объекта *activity*. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В обработчиках действий бота используйте следующий код для получения сообщения. 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В обработчиках действий бота используйте следующий код для получения сообщения.

```javascript
let text = turnContext.activity.text;
```

# <a name="pythontabpython"></a>[Python](#tab/python)

В обработчиках действий бота используйте следующий код для получения сообщения.

```python
response = context.activity.text
```

---

## <a name="send-a-typing-indicator"></a>Отправка индикатора ввода
Пользователи ожидают своевременный ответ на сообщения. Если бот выполняет какую-то длительную задачу, например, вызывает сервер или выполняет запрос, не давая пользователю никаких указаний на то, что его запрос принят во внимание, нетерпеливый пользователь может отправить дополнительные сообщения, предполагая, что бот сломан.

Боты в каналах Web Chat и Direct Line могут поддерживать отправку индикатора ввода, указывающего пользователю, что сообщение получено и обрабатывается. Имейте в виду, что бот должен завершить реплику в течение 15 секунд, иначе для службы Connector истечет времени ожидания. Чтобы реализовать длительную обработку, см. дополнительные сведения об [упреждающих сообщениях](bot-builder-howto-proactive-message.md). 

В следующем примере показана отправка индикатора ввода.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    if (string.Equals(turnContext.Activity.Text, "wait", System.StringComparison.InvariantCultureIgnoreCase))
    {
        await turnContext.SendActivitiesAsync(
            new Activity[] {
                new Activity { Type = ActivityTypes.Typing },
                new Activity { Type = "delay", Value= 3000 },
                MessageFactory.Text("Finished typing", "Finished typing"),
            },
            cancellationToken);
    }
    else
    {
        var replyText = $"Echo: {turnContext.Activity.Text}. Say 'wait' to watch me type.";
        await turnContext.SendActivityAsync(MessageFactory.Text(replyText, replyText), cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
this.onMessage(async (context, next) => {
    if (context.activity.text === 'wait') {
        await context.sendActivities([
            { type: ActivityTypes.Typing },
            { type: 'delay', value: 3000 },
            { type: ActivityTypes.Message, text: 'Finished typing' }
        ]);
    } else {
        await context.sendActivity(`You said '${ context.activity.text }'. Say "wait" to watch me type.`);
    }
    await next();
});
```

# <a name="pythontabpython"></a>[Python](#tab/python)

```python
async def on_message_activity(self, turn_context: TurnContext):
    if turn_context.activity.text == "wait":
        return await turn_context.send_activities([
            Activity(
                type=ActivityTypes.typing
            ),
            Activity(
                type="delay",
                value=3000
            ),
            Activity(
                type=ActivityTypes.message,
                text="Finished Typing"
            )
        ])
    else:
        return await turn_context.send_activity(
            f"You said {turn_context.activity.text}.  Say 'wait' to watch me type."
        )
```

---

## <a name="additional-resources"></a>Дополнительные ресурсы

- Дополнительные сведения об обработке действий в целом см. в разделе [Обработка действий](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack).
- См. дополнительные сведения о схеме действий Bot Framework и [действиях в беседах](https://aka.ms/botSpecs-activitySchema#message-activity).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавление мультимедиа в сообщения](./bot-builder-howto-add-media-attachments.md)

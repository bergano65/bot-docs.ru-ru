---
title: Отправка упреждающих сообщений | Документация Майкрософт
description: Узнайте, как прервать текущий ход общения упреждающим сообщением, используя пакет SDK Bot Builder для Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f432a570f5a8393a2aef3e4ec97c5e7e8cbaa43f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305803"
---
# <a name="send-proactive-messages"></a>Отправка упреждающих сообщений
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.js](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>Типы упреждающих сообщений

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>Отправка нерегламентированного упреждающего сообщения

В следующих примерах кода показано, как отправить нерегламентированное упреждающее сообщение с помощью пакета SDK Bot Builder для Node.js.

Чтобы иметь возможность отправить нерегламентированное сообщение пользователю, боту сначала необходимо собрать и сохранить сведения о пользователе из текущего сеанса общения. Свойство **address** сообщения сдержит все сведения, которые понадобятся боту, чтобы позже отправить нерегламентированное сообщение пользователю. 

```javascript
bot.dialog('adhocDialog', function(session, args) {
    var savedAddress = session.message.address;

    // (Save this information somewhere that it can be accessed later, such as in a database, or session.userData)
    session.userData.savedAddress = savedAddress;

    var message = 'Hello user, good to meet you! I now know your address and can send you notifications in the future.';
    session.send(message);
})
```

> [!NOTE]
> Бот может хранить данные пользователя любым способом, обеспечивающим ему последующий доступ к этим данным.

Собрав сведения о пользователе, бот может отправить пользователю нерегламентированное упреждающее сообщение в любое время. Для этого он просто получает сохраненные ранее данные пользователя, создает сообщение и отправляет его.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

var bot = new builder.UniversalBot(connector)
                .set('storage', inMemoryStorage); // Register in-memory storage 

function sendProactiveMessage(address) {
    var msg = new builder.Message().address(address);
    msg.text('Hello, this is a notification');
    msg.textLocale('en-US');
    bot.send(msg);
}
```

> [!TIP]
> Отправку нерегламентированного упреждающего сообщения могут инициировать асинхронные триггеры, например HTTP-запросы, таймеры, очереди или другие элементы кода на выбор разработчика.

## <a name="send-a-dialog-based-proactive-message"></a>Отправка упреждающего сообщения на основе диалога

В следующем примере кода показано, как отправить упреждающее сообщение на основе диалога с помощью пакета SDK Bot Builder для Node.js. Полный рабочий пример доступен в папке [Microsoft/BotBuilder-примеры/узел/core-proactiveMessages/startNewDialog](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog).

Чтобы иметь возможность отправить сообщение на основе диалога пользователю, боту сначала необходимо собрать (и сохранить) сведения из текущего сеанса общения. Объект `session.message.address` содержит все сведения, которые понадобятся боту, чтобы отправить пользователю упреждающее сообщение на основе диалога. 

```javascript
// proactiveDialog dialog
bot.dialog('proactiveDialog', function (session, args) {

    savedAddress = session.message.address;

    var message = 'Hey there, I\'m going to interrupt our conversation and start a survey in five seconds...';
    session.send(message);

    message = `You can also make me send a message by accessing: http://localhost:${server.address().port}/api/CustomWebApi`;
    session.send(message);

    setTimeout(() => {
        startProactiveDialog(savedAddress);
    }, 5000);
});
```

Когда необходимо отправить сообщение, бот создает диалог и добавляет его в верхнюю часть стека диалогов. Новый диалог берет на себя управление общением, передает упреждающее сообщение, закрывается и возвращает управление предыдущему диалогу в стеке. 

```javascript
// initiate a dialog proactively 
function startProactiveDialog(address) {
    bot.beginDialog(address, "*:survey");
}
```

> [!NOTE]
> В приведенном выше примере требуется пользовательский файл **botadapter.js**, который можно [скачать из репозитория GitHub](https://github.com/Microsoft/BotBuilder-Samples/blob/master/Node/core-proactiveMessages/startNewDialog/botadapter.js).

Диалог опроса управляет общением, пока не завершается. Затем он закрывается (путем вызова `session.endDialog()`), тем самым возвращая управление предыдущему диалогу. 


```javascript
// handle the proactive initiated dialog
bot.dialog('survey', function (session, args, next) {
  if (session.message.text === "done") {
    session.send("Great, back to the original conversation");
    session.endDialog();
  } else {
    session.send('Hello, I\'m the survey dialog. I\'m interrupting your conversation to ask you a question. Type "done" to resume');
  }
});
```

## <a name="sample-code"></a>Пример кода

Полный пример, в котором показано, как отправлять упреждающие сообщения с помощью пакета SDK Bot Builder для Node.js, приведен в <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages" target="_blank">примере proactiveMessages</a> на сайте GitHub. В примере proactiveMessages <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/simpleSendMessage" target="_blank">simpleSendMessage</a> показывает, как отправить нерегламентированное упреждающее сообщение, а <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages/startNewDialog" target="_blank">startNewDialog</a> показывает, как отправить упреждающее сообщение на основе диалога.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Проектирование хода общения](../bot-service-design-conversation-flow.md)

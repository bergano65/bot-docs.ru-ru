---
title: Отправка упреждающих сообщений | Документация Майкрософт
description: Узнайте, как прервать текущий поток общения упреждающим сообщением, используя пакет SDK Bot Framework для Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e5f8ec76e5711371653e75e11ac6fcc447b4f2e1
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/21/2019
ms.locfileid: "56590940"
---
# <a name="send-proactive-messages"></a>Отправка упреждающих сообщений
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.js](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>Типы упреждающих сообщений

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>Отправка нерегламентированного упреждающего сообщения

В следующих примерах кода показано, как отправить нерегламентированное упреждающее сообщение с помощью пакета SDK Bot Framework для Node.js.

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

В следующих примерах кода показано, как отправить упреждающее сообщение на основе диалога с помощью пакета SDK Bot Framework для Node.js. Полный рабочий пример доступен в папке [startNewDialog](https://aka.ms/js-startnewdialog-sample-v3).

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
> В приведенном выше примере требуется пользовательский файл **botadapter.js**, который можно [скачать из репозитория GitHub](https://aka.ms/js-botadaptor-file-v3).

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

Полный пример, в котором показано, как отправлять упреждающие сообщения с помощью пакета SDK Bot Framework для Node.js, приведен в <a href="https://aka.ms/js-proactivemessages-sample-v3" target="_blank">примере Proactive Messages</a> на сайте GitHub. В примере proactiveMessages <a href="https://aka.ms/js-simplesendmessage-sample-v3" target="_blank">simpleSendMessage</a> показывает, как отправить нерегламентированное упреждающее сообщение, а <a href="https://aka.ms/js-startnewdialog-sample-v3" target="_blank">startNewDialog</a> показывает, как отправить упреждающее сообщение на основе диалога.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Проектирование потока общения](../bot-service-design-conversation-flow.md)

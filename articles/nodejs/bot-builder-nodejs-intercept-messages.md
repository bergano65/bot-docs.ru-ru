---
title: Перехват сообщений | Документы Майкрософт
description: Сведения о том, как создавать журналы и другие записи путем перехвата и обработки передаваемой информации с помощью пакета SDK Bot Builder.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: f7470fa3b3702b9231bc03566aa85b23ee883a05
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904217"
---
# <a name="intercept-messages"></a>Перехват сообщений

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introduction to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="example"></a>Пример

В следующем примере кода показано, как перехватывать сообщения, которые передаются между пользователем и ботом, с помощью **ПО промежуточного слоя** в пакете SDK Bot Builder для Node.js. 

Во-первых, настройте обработчики для входящих сообщений (`botbuilder`) и для исходящих сообщений (`send`).

```javascript
server.post('/api/messages', connector.listen());
var bot = new builder.UniversalBot(connector);
bot.use({
    botbuilder: function (session, next) {
        myMiddleware.logIncomingMessage(session, next);
    },
    send: function (event, next) {
        myMiddleware.logOutgoingMessage(event, next);
    }
})
```

Затем реализуйте каждый из обработчиков, определив действие, которое будет выполнено над каждым перехваченным сообщением.

```javascript
module.exports = {
    logIncomingMessage: function (session, next) {
        console.log(session.message.text);
        next();
    },
    logOutgoingMessage: function (event, next) {
        console.log(event.text);
        next();
    }
}
```

Теперь при поступлении каждого входящего сообщения (от пользователя к боту) будет запущено `logIncomingMessage`, а при поступлении каждого исходящего сообщения (от бота к пользователю) будет запущено `logOutgoingMessage`.
В этом примере бот просто выводит некоторые сведения о каждом сообщении. При необходимости вы можете изменить `logIncomingMessage` и `logOutgoingMessage`, чтобы определить действия, которые должны быть выполнены с каждым сообщением. 

## <a name="sample-code"></a>Пример кода

Полный пример, в котором показано, как перехватывать сообщения и записывать их в журнал с помощью пакета SDK Bot Builder для Node.js, см. в разделе <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/capability-middlewareLogging" target="_blank">Пример использования ПО промежуточного слоя и записи в журнал</a> GitHub.

---
title: Перехват сообщений | Документы Майкрософт
description: Сведения о том, как создавать журналы и другие записи путем перехвата и обработки передаваемой информации с помощью пакета SDK Bot Builder.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/02/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 31380961f117a2b4a3ffaae3c82d682a63001c0c
ms.sourcegitcommit: 984705927561cc8d6a84f811ff24c8c71b71c76b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/02/2018
ms.locfileid: "50965682"
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

Полный пример, в котором показано, как перехватывать сообщения и записывать их в журнал с помощью пакета SDK Bot Builder для Node.js, см. в разделе <a href="https://aka.ms/v3-js-capability-middlewareLogging" target="_blank">Пример использования ПО промежуточного слоя и записи в журнал</a> GitHub.

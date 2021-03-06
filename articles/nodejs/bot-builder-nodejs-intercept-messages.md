---
title: Перехват сообщений (JS версии 3) — Служба Azure Bot
description: Сведения о том, как создавать журналы и другие записи путем перехвата и обработки передаваемой информации с помощью пакета SDK Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/02/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 32934a8cb857b9ff3f2b4f2058a58fcce12cfc3d
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790886"
---
# <a name="intercept-messages"></a>Перехват сообщений

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introduction to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="example"></a>Пример

В следующем примере кода показано, как перехватывать сообщения, которые передаются между пользователем и ботом, с помощью **ПО промежуточного слоя** в пакете SDK Bot Framework для Node.js. 

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

## <a name="sample-code"></a>Образец кода

Полный пример, в котором показано, как перехватывать сообщения и записывать их в журнал с помощью пакета SDK Bot Framework для Node.js, см. в <a href="https://aka.ms/v3-js-capability-middlewareLogging" target="_blank">примере использования ПО промежуточного слоя и записи в журнал</a> на сайте GitHub.

---
title: Отправка индикатора ввода | Документация Майкрософт
description: Узнайте, как добавить индикатор "please wait", чтобы сообщить пользователю, что бот обрабатывает запрос, используя пакет SDK Bot Builder для Node.js
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 1c6b01b1cab9165b1513bfda5af182f27635290e
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904386"
---
# <a name="send-a-typing-indicator"></a>Отправка индикатора ввода 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Пользователи ожидают своевременный ответ на сообщения. Если бот выполняет какую-то длительную задачу, например, вызывает сервер или выполняет запрос, не давая пользователю никаких указаний на то, что его запрос принят во внимание, нетерпеливый пользователь может отправить дополнительные сообщения, предполагая, что бот сломан.
Многие каналы поддерживают отправку индикации ввода, указывающие пользователю, что сообщение было получено и обрабатывается.


## <a name="typing-indicator-example"></a>Пример индикатора ввода

Следующий пример демонстрирует отправку индикатора ввода, используя [session.sendTyping()][SendTyping].  Вы можете протестировать его в Bot Framework Emulator.


```javascript

// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.sendTyping();
    setTimeout(function () {
        session.send("Hello there...");
    }, 3000);
});
```

Индикаторы ввода также полезны при вставке задержки сообщений для предотвращения отправки сообщений, содержащих изображения.

Дополнительные сведения см. в разделе [Добавление вложений в виде форматированных карточек в сообщения](bot-builder-nodejs-send-rich-cards.md).


## <a name="additional-resources"></a>Дополнительные ресурсы

* [sendTyping][SendTyping]


[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

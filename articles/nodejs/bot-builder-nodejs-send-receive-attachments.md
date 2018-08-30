---
title: Отправка и получение вложений | Документация Майкрософт
description: Узнайте, как отправлять и получать сообщения, содержащие вложения, с помощью пакета SDK Bot Builder для Node.js.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4203e9d7a9c5c8e6ab068def879747a4c6158367
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904048"
---
# <a name="send-and-receive-attachments"></a>Отправка и получение вложений

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

Обмен сообщениями между пользователем и ботом может включать вложения мультимедиа, такие как изображения, видео, аудио и файлы. Типы вложений, которые могут быть отправлены, зависят от канала, но это базовые типы.

* **Мультимедиа и файлы**. Можно отправлять такие файлы, как изображения, аудио- и видеофайлы, задав для **contentType** тип MIME [объекта IAttachment][IAttachment] и затем передав ссылку на файл в **contentUrl**.
* **Карточки**. Можно отправлять обширный набор визуальных карточек <!-- and custom keyboards -->, задав для **contentType** нужный тип карточки и затем передав JSON для этой карточки. При использовании одного из таких классов построителя форматированных карточек, как **HeroCard**, вложение добавляется автоматически. С соответствующими примерами можно ознакомиться в разделе [Добавление вложений в виде форматированных карточек в сообщения](bot-builder-nodejs-send-rich-cards.md).

## <a name="add-a-media-attachment"></a>Добавление мультимедийного вложения
Объект message должен быть экземпляром [IMessage][IMessage]. Он особенно удобен для отправки пользователю сообщения в виде объекта, если требуется что-то вложить в сообщение (например, изображение). Используйте метод [session.send()][SessionSend] для отправки сообщений в виде объекта JSON. 

## <a name="example"></a>Пример

В следующем примере проверяется, отправил ли пользователь вложение, и если это так, то отображаются все изображения, содержащиеся во вложении. Вы можете протестировать этот пример в Bot Framework Emulator, отправив боту изображение.

```javascript
// Create your bot with a function to receive messages from the user
var bot = new builder.UniversalBot(connector, function (session) {
    var msg = session.message;
    if (msg.attachments && msg.attachments.length > 0) {
     // Echo back attachment
     var attachment = msg.attachments[0];
        session.send({
            text: "You sent:",
            attachments: [
                {
                    contentType: attachment.contentType,
                    contentUrl: attachment.contentUrl,
                    name: attachment.name
                }
            ]
        });
    } else {
        // Echo back users text
        session.send("You said: %s", session.message.text);
    }
});
```
## <a name="additional-resources"></a>Дополнительные ресурсы

* [Предварительный просмотр компонентов с помощью Channel Inspector][inspector]
* [IMessage][IMessage]
* [Отправка форматированной карточки][SendRichCard]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[SendRichCard]: bot-builder-nodejs-send-rich-cards.md
[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send
[IAttachment]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html
[inspector]: ../bot-service-channel-inspector.md

---
title: Отправка и получение вложений — Служба Azure Bot
description: Узнайте, как отправлять и получать сообщения, содержащие вложения, с помощью пакета SDK Bot Framework для Node.js.
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 182fb3ab327b55d8976a607871bb2380a88f8b30
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790562"
---
# <a name="send-and-receive-attachments"></a>Отправка и получение вложений

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

Обмен сообщениями между пользователем и ботом может включать вложения мультимедиа, такие как изображения, видео, аудио и файлы. Типы вложений, которые могут быть отправлены, зависят от канала, но это базовые типы.

* **Мультимедиа и файлы**. Вы можете отправлять файлы разных форматов, например изображения, аудио- и видеофайлы, указав в параметре **contentType** тип MIME для [объекта IAttachment][IAttachment] и передав ссылку на файл в параметре **contentUrl**.
* **Карточки**. Вы можете отправлять обширный набор визуальных карточек, <!-- and custom keyboards --> задав в параметре **contentType** нужный тип карточки и передав содержимое этой карточки в формате JSON. При использовании одного из таких классов построителя форматированных карточек, как **HeroCard**, вложение добавляется автоматически. С соответствующими примерами можно ознакомиться в разделе [Добавление вложений в виде форматированных карточек в сообщения](bot-builder-nodejs-send-rich-cards.md).

## <a name="add-a-media-attachment"></a>Добавление мультимедийного вложения
Объект message должен быть экземпляром [IMessage][IMessage]. Его особенно удобно использовать для отправки пользователю сообщения в виде объекта, если в это сообщение требуется что-то вложить (например, изображение). Используйте метод [session.send()][SessionSend] для отправки сообщений в виде объекта JSON. 

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

* [Справочник по каналам][inspector]
* [IMessage][IMessage]
* [Отправка форматированной карточки][SendRichCard]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[SendRichCard]: bot-builder-nodejs-send-rich-cards.md
[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send
[IAttachment]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iattachment.html
[inspector]: ../bot-service-channels-reference.md

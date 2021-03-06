---
title: Добавление предлагаемых действий в сообщения (JS версии 3) — Служба Azure Bot
description: Сведения о добавлении предлагаемых действий в сообщения с помощью пакета SDK Bot Framework для Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/19/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ebad14abfdef2e274562b17ca1945d709a2c8d54
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790389"
---
# <a name="add-suggested-actions-to-messages"></a>Добавление предлагаемых действий к сообщениям

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

## <a name="suggested-actions-example"></a>Пример предлагаемых действий

Чтобы добавить предлагаемые действия в сообщения, задайте свойство `suggestedActions` сообщения списку объектов [CardAction][ICardAction], представляющих кнопки, которые будут отображаться для пользователя.

В примере кода ниже показана отправка сообщения, которое представляет три предлагаемых действия для пользователя:

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

Когда пользователь выбирает одно из предложенных действий, бот получает сообщение от пользователя, которое содержит значение `value` соответствующего действия.

Учтите, что метод `imBack` отправит `value` в окно чата используемого канала. Если это нежелательное поведение, можно использовать метод `postBack`, который отправит выбор в бот, не отображая этот выбор в окне чата. Некоторые каналы не поддерживают `postBack`, но и в этих случаях метод будет работать как `imBack`.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Примеры][samples]
- [IMessage][IMessage]
- [ICardAction][ICardAction]
- [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

<!-- The inspector is no longer supported: we're redirecting to the samples for now. -->
[samples]: https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples

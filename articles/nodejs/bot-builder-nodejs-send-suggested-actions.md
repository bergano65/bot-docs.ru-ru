---
title: Добавление предлагаемых действий в сообщения | Документация Майкрософт
description: Сведения о добавлении предлагаемых действий в сообщения с помощью пакета SDK Bot Builder для Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/06/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 510a340506416e61f906228ccd26c0bdd6e5d4d3
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305034"
---
# <a name="add-suggested-actions-to-messages"></a>Добавление предлагаемых действий в сообщения
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> Используйте [инспектор каналов][channelInspector], чтобы узнать, как предлагаемые действия отображаются и используются с разными каналами.

## <a name="suggested-actions-example"></a>Пример предлагаемых действий

Чтобы добавить предлагаемые действия в сообщения, задайте свойство `suggestedActions` сообщения списку объектов [CardAction][ICardAction], представляющих кнопки, которые будут отображаться для пользователя.

В примере кода ниже показана отправка сообщения, которое представляет три предлагаемых действия для пользователя:

[!code-javascript[Send suggested actions](../includes/code/node-send-suggested-actions.js#sendSuggestedActions)]

Когда пользователь выбирает одно из предложенных действий, бот получает сообщение от пользователя, которое содержит значение `value` соответствующих действий.

Учтите, что метод `imBack` отправит `value` в окно чата используемого канала. Если это нежелательное поведение, можно использовать метод `postBack`, который отправит выбор в бот, не отображая этот выбор в окне чата. Некоторые каналы не поддерживают `postBack`, но и в этих случаях метод будет работать как `imBack`.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Предварительный просмотр компонентов с помощью Channel Inspector][inspector]
* [IMessage][IMessage]
* [ICardAction][ICardAction]
* [session.send][SessionSend]

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[SessionSend]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#send

[ICardAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icardaction.html

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md

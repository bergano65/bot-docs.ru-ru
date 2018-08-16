---
title: Добавление предлагаемых действий в сообщения | Документация Майкрософт
description: Сведения о добавлении предлагаемых действий в сообщения с помощью пакета SDK Bot Builder для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/13/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 63e5f4b8ac86b6b0e29d09796dbe74295bf3e213
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301484"
---
# <a name="add-suggested-actions-to-messages"></a>Добавление предлагаемых действий в сообщения
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

> [!TIP]
> Используйте [инспектор каналов][channelInspector], чтобы узнать, как предлагаемые действия отображаются и используются с разными каналами.

## <a name="send-suggested-actions"></a>Отправка предлагаемых действий

Чтобы добавить предложенные действия в сообщения, задайте свойство `SuggestedActions` действия списку объектов [CardAction][cardAction], представляющих кнопки, которые будут отображаться для пользователя. 

В примере кода ниже показано создание сообщения, которое представляет три предлагаемых действия для пользователя:

[!code-csharp[Add suggested actions](../includes/code/dotnet-add-suggested-actions.cs#addSuggestedActions)]

Когда пользователь выбирает одно из предложенных действий, бот получает сообщение от пользователя, которое содержит значение `Value` соответствующих действий.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Предварительный просмотр компонентов с помощью Channel Inspector][inspector]
- [Общие сведения о действиях](bot-builder-dotnet-activities.md)
- [Создание сообщений](bot-builder-dotnet-create-messages.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">Интерфейс IMessageActivity</a>
- <a href="/dotnet/api/microsoft.bot.connector.cardaction" target="_blank">Класс CardAction</a>
- <a href="/dotnet/api/microsoft.bot.connector.suggestedactions" target="_blank">Класс SuggestedActions</a>

[cardAction]: /dotnet/api/microsoft.bot.connector.cardaction

[inspector]: ../bot-service-channel-inspector.md

[channelInspector]: ../bot-service-channel-inspector.md



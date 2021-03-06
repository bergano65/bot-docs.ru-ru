---
title: Добавление предлагаемых действий в сообщения (C# версии 3) — Служба Azure Bot
description: Сведения о добавлении предлагаемых действий в сообщения с помощью пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/13/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 61aaff340a235f7b9b552d79205fab8e772216f9
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75796687"
---
# <a name="add-suggested-actions-to-messages"></a>Добавление предлагаемых действий к сообщениям

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-suggested-actions.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-suggested-actions.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-suggested-actions.md)

[!INCLUDE [Introduction to suggested actions](../includes/snippet-suggested-actions-intro.md)]

## <a name="send-suggested-actions"></a>Отправка предлагаемых действий

Чтобы добавить предложенные действия в сообщения, задайте свойство `SuggestedActions` действия списку объектов [CardAction][cardAction], представляющих кнопки, которые будут отображаться для пользователя. 

В примере кода ниже показано создание сообщения, которое представляет три предлагаемых действия для пользователя:

[!code-csharp[Add suggested actions](../includes/code/dotnet-add-suggested-actions.cs#addSuggestedActions)]

Когда пользователь выбирает одно из предложенных действий, бот получает сообщение от пользователя, которое содержит значение `Value` соответствующего действия.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Общие сведения о действиях](bot-builder-dotnet-activities.md)
- [Создание сообщений](bot-builder-dotnet-create-messages.md)
- [Класс Activity](https://aka.ms/ActivityClass-dotnet-API)
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">Интерфейс IMessageActivity</a>
- <a href="/dotnet/api/microsoft.bot.connector.cardaction" target="_blank">Класс CardAction</a>
- <a href="/dotnet/api/microsoft.bot.connector.suggestedactions" target="_blank">Класс SuggestedActions</a>

[cardAction]: /dotnet/api/microsoft.bot.connector.cardaction

[inspector]: ../bot-service-channel-inspector.md



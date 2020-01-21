---
title: Обзор действий — Служба Azure Bot
description: В этой статье представлены разные типы действий, доступные в пакете SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9aa9edf7d0bd20d1fa615ccbba5fc655a8dc40f1
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75797270"
---
# <a name="activities-overview"></a>Общие сведения о действиях

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

## <a name="activity-types-in-the-bot-framework-sdk-for-net"></a>Типы действий в пакете SDK Bot Framework для .NET

В пакете SDK Bot Framework для .NET поддерживаются типы действий, приведенные ниже.

| Тип действия | Интерфейс | Description |
|------|------|------|
| [message](#message) | IMessageActivity | Представляет связь между ботом и пользователем. |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity | Указывает, что бот был добавлен в общение, другие участники были добавлены или удалены из него или метаданные общения изменились. |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity | Указывает, что бот был добавлен или удален из списка контактов пользователя. |
| [typing](#typing) | ITypingActivity | Указывает, что пользователь или бот на другом конце общения составляет ответ. | 
| [deleteUserData](#deleteuserdata) | Недоступно | Указывает боту, что пользователь выполнил запрос на удаление любых пользовательских данных, которые бот мог сохранить. |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity | Указывает на конец общения. |
| [event](#event) | IEventActivity | Представляет сообщение, отправляемое боту, который не отображается для пользователя. |
| [invoke](#invoke) | IInvokeActivity | Представляет сообщение, отправляемое боту для запроса определенной операции. Этот тип действия зарезервирован для внутреннего использования в Microsoft Bot Framework. |
| [messageReaction](#messagereaction) | IMessageReactionActivity | Указывает, что пользователь отреагировал на существующее действие. Например, в сообщение, пользователь нажимает кнопку "Like". |

## <a name="message"></a>message

Бот отправляет действия **message** для передачи сведений пользователям и получения действий **message** от пользователей. Некоторые сообщения могут состоять из простого текста, в то время как другие могут содержать более богатое содержимое, такое как [произносимый текст](bot-builder-dotnet-text-to-speech.md), [предлагаемые действия](bot-builder-dotnet-add-suggested-actions.md), [мультимедийные вложения](bot-builder-dotnet-add-media-attachments.md), [форматированные карточки](bot-builder-dotnet-add-rich-card-attachments.md) и [данные по каналу](bot-builder-dotnet-channeldata.md). Дополнительные сведения о часто используемых свойствах сообщения см. в статье [Создание сообщений](bot-builder-dotnet-create-messages.md).

## <a name="conversationupdate"></a>conversationUpdate

Бот получает действие **conversationUpdate** всякий раз, когда его добавляют в общение, добавляют или удаляют других участников из общения или изменяются метаданные общения. 

Если элементы были добавлены к общению, то свойство `MembersAdded` действия будет содержать массив объектов `ChannelAccount` для идентификации новых элементов. 

Чтобы определить, был ли добавлен бот к общению (т. е. является одним из новых элементов), оцените, соответствует ли значение `Recipient.Id` действия (идентификатор бота) свойству `Id` каких-либо учетных записей в массиве `MembersAdded`.

Если элементы были удалены из общения, свойство `MembersRemoved` будет содержать массив объектов `ChannelAccount` для идентификации удаленных элементов. 

> [!TIP]
> Если бот получает действие **conversationUpdate**, указывающее, что пользователь вступает в общение, можно ответить на него, отправив приветственное сообщение. 

## <a name="contactrelationupdate"></a>contactRelationUpdate

Бот получает действие **contactRelationUpdate** при каждом добавлении или удалении из списка контактов пользователя. Значение свойства `Action` действия (add | remove) указывает, был ли бот добавлен или удален из списка контактов пользователя.

## <a name="typing"></a>typing

Бот получает действие **typing**, чтобы указать, что пользователь вводит ответ. Бот может отправлять действие **typing**, чтобы указать пользователю, что выполняется запрос или составление ответа. 

## <a name="deleteuserdata"></a>deleteUserData

Бот получает действие **deleteUserData**, когда пользователь запрашивает удаление любых данных, которые были ранее сохранены ботом. Если бот получает действия такого типа, следует удалить все личные сведения, который ранее хранились для пользователя, выполнившего запрос.

## <a name="endofconversation"></a>endOfConversation 

Бот получает действие **endOfConversation**, чтобы указать, что пользователь закончил общение. Бот может отправить действие **endOfConversation**, чтобы уведомить пользователя, что общение окончено. 

## <a name="event"></a>event

Бот может получить действие **event** от внешнего процесса или службы, которые передают информацию боту, чтобы эта информация не была видна пользователям. Отправитель действия **event** обычно не ожидает, что бот каким-либо способом подтвердит получение.

## <a name="invoke"></a>invoke

Бот может использовать действие **invoke**, которое представляет запрос для выполнения конкретной операции. Отправитель действия **invoke** обычно предполагает, что бот подтвердит получение с помощью HTTP-ответа. Этот тип действия зарезервирован для внутреннего использования в Microsoft Bot Framework.

## <a name="messagereaction"></a>messageReaction

Некоторые каналы отправляют действия **messageReaction** боту, когда пользователь отреагировал на существующее действие. Например, в сообщение, пользователь нажимает кнопку "Like". Свойство **ReplyToId** будет указывать, на которое действие отреагировал пользователь.

Действию **messageReaction** может соответствовать любое количество типов **messageReactionType**, которые определены в канале. Например, канал может отправлять тип реакции "Like" или "PlusOne". 

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Действия отправки и получения](bot-builder-dotnet-connector.md)
- [Создание сообщений](bot-builder-dotnet-create-messages.md)
- [Класс Activity](https://aka.ms/ActivityClass-dotnet-API)

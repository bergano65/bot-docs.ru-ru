---
title: Действия отправки и получения — Служба Azure Bot
description: Сведения об обмене данными с пользователем через различные каналы связи с использованием службы соединителя через пакет SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c361067c7c51a4230d6b3da127a932fcb93cd2fe
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75796669"
---
# <a name="send-and-receive-activities"></a>Действия отправки и получения

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Соединитель Bot Framework Connector предоставляет единый интерфейс REST API, позволяющий боту взаимодействовать через различные каналы связи, такие как Skype, электронная почта, Slack и т. д. Он обеспечивает обмен данными между ботом и пользователем, ретранслируя сообщения из бота в канал и из канала в бот. 

В этой статье описывается, как использовать соединитель с помощью пакета SDK Bot Framework для .NET для обмена данными между ботом и пользователем через выбранный канал. 

> [!NOTE]
> Для создания бота вполне достаточно методов, которые описаны в этой статье, но пакет SDK Bot Framework содержит и дополнительные функции, например [диалоги](bot-builder-dotnet-dialogs.md) и [FormFlow](bot-builder-dotnet-formflow.md), которые позволяют оптимизировать управление потоком беседы и состоянием, а также упростить внедрение распознавания речи и других служб Cognitive Services.

## <a name="create-a-connector-client"></a>Создание клиента соединителя

Класс [ConnectorClient][ConnectorClient] содержит методы, которые бот использует для взаимодействия с пользователем по каналу. При получении ботом объекта <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> из этого соединителя он должен использовать указанный для этого действия `ServiceUrl` для создания клиента соединителя, который впоследствии будет использоваться для создания ответа. 

[!code-csharp[Create connector client](../includes/code/dotnet-send-and-receive.cs#createConnectorClient)]

> [!TIP]
> Так как конечная точка канала может быть нестабильной, бот, по возможности, должен направлять обмен данными на конечную точку, указанную соединителем в объекте `Activity` (а не полагаться на кэшированную конечную точку). 
>
> Если боту требуется инициировать диалог, он может использовать кэшированную конечную точку для указанного канала (поскольку в этом случае входящий объект `Activity` отсутствует), однако кэшированные конечные точки необходимо часто обновлять. 

## <a id="create-reply"></a> Создание ответа

Соединитель использует объект [Activity](bot-builder-dotnet-activities.md) для передачи данных между ботом и каналом (пользователем). Каждое действие содержит сведения, используемые для маршрутизации сообщения в соответствующее место назначения, а также сведения о создателе сообщения (свойство `From`), контексте сообщения и получателе сообщения (свойство `Recipient`).

Когда бот получает действие из соединителя, свойство `Recipient` входящего действия задает удостоверение бота в этой беседе. Так как некоторые каналы (например, Slack) назначают боту новое удостоверение при его добавлении в беседу, бот должен всегда использовать значение свойства `Recipient` входящего действия в качестве значения свойства `From` в своем ответе.

Вы можете самостоятельно создать с нуля и инициализировать исходящий объект `Activity`, но пакет SDK Bot Framework позволяет создать ответ намного проще. С помощью метода `CreateReply` входящего действия вы просто указываете текст сообщения для ответа, а исходящее действие создается с автоматически заполненными свойствами `Recipient`, `From` и `Conversation`.

[!code-csharp[Create reply](../includes/code/dotnet-send-and-receive.cs#createReply)]

## <a name="send-a-reply"></a>Отправка ответа

Создав ответ, вы можете отправить его путем вызова метода `ReplyToActivity` клиента соединителя. Соединитель будет доставлять ответ, используя семантику соответствующего канала. 

[!code-csharp[Send reply](../includes/code/dotnet-send-and-receive.cs#sendReply)]

> [!TIP]
> Если бот отвечает на сообщение пользователя, всегда используйте метод `ReplyToActivity`.

## <a name="send-a-non-reply-message"></a>Отправка сообщения (не ответа) 

Если бот является частью беседы, он может отправлять сообщение, которое не является прямым ответом на любые сообщения пользователя, вызвав метод `SendToConversation`. 

[!code-csharp[Send non-reply message](../includes/code/dotnet-send-and-receive.cs#sendNonReplyMessage)]

Вы можете использовать метод `CreateReply` для инициализации нового сообщения (который будет автоматически задавать свойства `Recipient`, `From` и `Conversation` для сообщения). Кроме того, можно использовать метод `CreateMessageActivity`, чтобы создать новое сообщение и самостоятельного задать все значения свойств.

> [!NOTE]
> Bot Framework не накладывает каких-либо ограничений на число сообщений, которые может отправлять бот. Тем не менее большинство каналов принудительно применяет ограничения регулирования, запрещая ботам отправлять большое число сообщений за короткий период времени. Кроме того, если бот отправляет несколько сообщений за короткий промежуток времени, канал не всегда может обрабатывать сообщения в правильной последовательности.

## <a name="start-a-conversation"></a>Начало общения

Возможны ситуации, когда боту требуется инициировать беседу с одним или несколькими пользователями. Беседу можно начать, вызвав метод `CreateDirectConversation` (для частной беседы с одним пользователем) или метод `CreateConversation` (для групповой беседы с несколькими пользователями) для получения объекта `ConversationAccount`. Затем вы создаете сообщение и отправляете его, вызвав метод `SendToConversation`. Для использования метода `CreateDirectConversation` или `CreateConversation` необходимо сначала [создать клиент соединителя](#create-a-connector-client) с помощью URL-адреса службы целевого канала (который можно получить из кэша, если он сохранялся из предыдущих сообщений). 

> [!NOTE]
> Не все каналы поддерживают групповые беседы. Сведения о том, поддерживает ли канал групповые беседы, см. в документации канала.

Этот пример кода использует метод `CreateDirectConversation` для создания частной беседы с одним пользователем.

[!code-csharp[Start private conversation](../includes/code/dotnet-send-and-receive.cs#startPrivateConversation)]

Этот пример кода использует метод `CreateConversation` для создания групповой беседы с несколькими пользователями.

[!code-csharp[Start group conversation](../includes/code/dotnet-send-and-receive.cs#startGroupConversation)]

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Общие сведения о действиях](bot-builder-dotnet-activities.md)
- [Создание сообщений](bot-builder-dotnet-create-messages.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочная информация по пакету SDK Bot Framework для .NET</a>
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
- <a href="/dotnet/api/microsoft.bot.connector.connectorclient" target="_blank">Класс ConnectorClient</a>

[ConnectorClient]: /dotnet/api/microsoft.bot.connector.connectorclient

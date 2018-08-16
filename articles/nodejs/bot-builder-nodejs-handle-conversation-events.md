---
title: Обработка событий пользователя и диалога | Документация Майкрософт
description: Узнайте, как обрабатывать такие события, как присоединение пользователя к диалогу, с помощью пакета SDK Bot Builder для Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 23cb3189de10a67a524114d0c7d9f1f9d2b0441d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305131"
---
# <a name="handle-user-and-conversation-events"></a>Обработка событий пользователя и диалога

В этой статье объясняется, как бот может обрабатывать события, например присоединение пользователя к диалогу, добавление бота в список контактов или отправка сообщения **Goodbye** (До свидания), когда бот покидает диалог.


## <a name="greet-a-user-on-conversation-join"></a>Приветствие пользователя при присоединении к диалогу
Bot Framework предоставляет событие [conversationUpdate][conversationUpdate] для уведомления бота каждый раз, когда участник присоединяется к диалогу или покидает его. Участником диалога может быть пользователь или бот.

Приведенный ниже фрагмент кода позволяет боту приветствовать новых участников диалога или **прощаться** с ними, когда бот покидает диалог.

[!INCLUDE [conversationUpdate sample Node.js](../includes/snippet-code-node-conversationupdate-1.md)]

## <a name="acknowledge-add-to-contacts-list"></a>Выражение признательности при добавлении в список контактов

Событие [ContactRelationUpdate][contactRelationUpdate] позволяет сообщить боту, что пользователь добавил его в свой список контактов.

[!INCLUDE [contactRelationUpdate sample Node.js](../includes/snippet-code-node-contactrelationupdate-1.md)]

## <a name="add-a-first-run-dialog"></a>Добавление диалога первого запуска

Так как не все каналы поддерживают события **conversationUpdate** и **contactRelationUpdate**, универсальный способ поприветствовать пользователя, который присоединяется к диалогу, — добавить диалог первого запуска.

В приведенном ниже примере мы добавили функцию, которая активирует диалог каждый раз, когда добавляется новый пользователь. Вы можете настроить способ активации действия, указав для него обработчик [onFindAction][onFindAction]. 

[!INCLUDE [first-run sample Node.js](../includes/snippet-code-node-first-run-dialog-1.md)]

Вы также можете указать, что происходит после активации действия, предоставив обработчик [onSelectAction][onSelectAction]. Для активируемых действий вы можете указать обработчик [onInterrupted][onInterrupted], чтобы перехватывать прерывание, прежде чем оно произойдет. Дополнительные сведения см. в статье об [обработке событий пользователя](bot-builder-nodejs-dialog-actions.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [conversationUpdate][conversationUpdate]
* [contactRelationUpdate][contactRelationUpdate]

[conversationUpdate]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iconversationupdate.html
[contactRelationUpdate]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.icontactrelationupdate.html

[onFindAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onfindaction
[onSelectAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#onselectaction
[onInterrupted]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#oninterrupted

[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping
[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata

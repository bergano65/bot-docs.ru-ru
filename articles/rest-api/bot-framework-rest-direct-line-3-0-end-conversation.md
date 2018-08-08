---
title: Завершение диалога | Документы Майкрософт
description: Узнайте, как завершить диалог с помощью API Direct Line версии 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: ac984609acfdd8f85088bd47ccded1f45e953b2c
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305574"
---
# <a name="end-a-conversation"></a>Завершение диалога

Клиент или бот может дать сигнал об окончании диалога Direct Line, отправив [действие](bot-framework-rest-connector-activities.md) **endOfConversation**. 

## <a name="send-an-endofconversation-activity"></a>Отправка действия endOfConversation

Действие **endOfConversation** прекращает взаимодействие между ботом и клиентом. После отправки действия **endOfConversation** клиент по-прежнему может [получать сообщения](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get) с помощью `HTTP GET`, но ни клиент, ни бот не могут отправлять в диалог дополнительные сообщения. 

Чтобы завершить диалог, просто отправьте запрос POST, чтобы отправить действие **endOfConversation**.

### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
    "type": "endOfConversation",
    "from": {
        "id": "user1"
    }
}
```

### <a name="response"></a>Ответ

Если запрос выполнен успешно, ответ будет содержать идентификатор отправленного действия.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0004"
}
```

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-3-0-concepts.md)
- [Аутентификация](bot-framework-rest-direct-line-3-0-authentication.md)
- [Отправка действий боту](bot-framework-rest-direct-line-3-0-send-activity.md)

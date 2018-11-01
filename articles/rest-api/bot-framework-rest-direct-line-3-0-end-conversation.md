---
title: Завершение диалога | Документы Майкрософт
description: Узнайте, как завершить диалог с помощью API Direct Line версии 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: f0985f28fd1744bcfb6bf5cea1c2230254670e01
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000211"
---
# <a name="end-a-conversation"></a>Завершение диалога

Клиент или бот может дать сигнал об окончании диалога Direct Line, отправив [действие](bot-framework-rest-connector-activities.md) **endOfConversation**. 

> [!NOTE] 
> Событие endOfConversation поддерживается только в канале Cortana. В других каналах эта функциональность не реализована. Каждый канал определяет, как реагировать на действие endOfConversation. Если вы разрабатываете клиент DirectLine, нужно реализовать правильное поведение клиента, например выдачу сообщения об ошибке, если бот отправил действие в диалог, который уже завершен.

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

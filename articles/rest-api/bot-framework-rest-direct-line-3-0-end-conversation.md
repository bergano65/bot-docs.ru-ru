---
title: Завершение диалога в службе Bot
description: Узнайте, как завершить диалог с помощью API Direct Line версии 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 4b8a3046ee53d90fe2abdc97a3c931a4f67c051f
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789436"
---
# <a name="end-a-conversation"></a>Завершение диалога

**EndOfConversation** — это [действие](https://aka.ms/botSpecs-activitySchema), означающее что канал или бот завершил диалог. 

> [!NOTE] 
> Хотя событие **endOfConversation** отправляется несколькими каналами, канал Cortana — это единственный канал, который его принимает. Другие каналы, включая Direct Line, не реализуют эту функцию, отклоняя или пересылая вместо этого действие — каждый канал определяет, как реагировать на действие endOfConversation. Если вы разрабатываете клиент DirectLine, нужно реализовать правильное поведение клиента, например выдачу сообщения об ошибке, если бот отправил действие в диалог, который уже завершен.

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

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
ms.openlocfilehash: 438558995f83ade38404856d61ba66ee77480a27
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711938"
---
# <a name="end-a-conversation"></a>Завершение диалога

**EndOfConversation** — это [действие](bot-framework-rest-connector-activities.md) означает, что канал или бот завершили диалог. 

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
- [Проверка подлинности](bot-framework-rest-direct-line-3-0-authentication.md)
- [Отправка действий боту](bot-framework-rest-direct-line-3-0-send-activity.md)

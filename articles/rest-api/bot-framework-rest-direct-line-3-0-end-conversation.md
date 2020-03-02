---
title: Завершение диалога в службе Bot
description: Узнайте, как завершить диалог с помощью API Direct Line версии 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 67c74bc3f51328370e4d6ba207855b20e4d79497
ms.sourcegitcommit: 308e6df385b9bac9c8d60f8b75eabc813b823c38
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/20/2020
ms.locfileid: "77520003"
---
# <a name="end-a-conversation"></a>Завершение диалога

**EndOfConversation** — это [действие](https://aka.ms/botSpecs-activitySchema), означающее что канал или бот завершил диалог. 

> [!NOTE] 
> Хотя событие **endOfConversation** отправляется несколькими каналами, канал Cortana — это единственный канал, который его принимает. Другие каналы, включая Direct Line, не реализуют эту функцию, отклоняя или пересылая вместо этого действие — каждый канал определяет, как реагировать на действие endOfConversation.

## <a name="send-an-endofconversation-activity"></a>Отправка действия endOfConversation

Чтобы запросить окончание диалога с помощью канала Кортаны, отправьте запрос POST действия End of Conversation (Завершение беседы) в конечную точку обмена сообщениями канала.

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

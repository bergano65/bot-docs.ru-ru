---
title: Получение сообщений от бота — Служба Azure Bot
description: Узнайте, как получать сообщение от бота с помощью Direct Line API 1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: c88d3f363bf4bcc40fa7a21aa1fcdd0b764abe1e
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789700"
---
# <a name="receive-messages-from-the-bot"></a>Получение сообщений от бота

> [!IMPORTANT]
> В этой статье описывается получения сообщения ботом с помощью Direct Line API 1.1. При создании подключения между клиентским приложением и ботом используйте [API версии 3.0 для Direct Line](bot-framework-rest-direct-line-3-0-receive-activities.md).

Чтобы использовать протокол Direct Line 1.1 для получения сообщений, клиенты должны взаимодействовать через интерфейс `HTTP GET`. 

## <a name="retrieve-messages-with-http-get"></a>Получение сообщения с помощью HTTP-запроса GET

Для получения сообщений из конкретного общения отправьте запрос `GET` в конечную точку `api/conversations/{conversationId}/messages`, при необходимости задавая параметр `watermark`, чтобы указать последнее сообщение, отображаемое клиенту. Даже если в ответе JSON не было включено сообщение, он будет возвращен с обновленным значением свойства `watermark`.

Ниже приведены примеры фрагментов кода для запроса и ответа "Get Messages". Ответ Get Messages содержит свойство `watermark` в качестве [MessageSet](bot-framework-rest-direct-line-1-1-api-reference.md#messageset-object). Клиенты должны просматривать доступные сообщения, перемещая значение `watermark` до тех пор, пока сообщения не перестанут возвращаться. 

### <a name="request"></a>Запрос

```http
GET https://directline.botframework.com/api/conversations/abc123/messages?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Ответ

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "messages": [
        {
            "conversation": "abc123",
            "id": "abc123|0000",
            "text": "hello",
            "from": "user1"
        }, 
        {
            "conversation": "abc123",
            "id": "abc123|0001",
            "text": "Nice to see you, user1!",
            "from": "bot1"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>Рекомендации относительно расписания

Несмотря на то что Direct Line — это составной протокол с потенциальными пробелами в расписании, протокол и служба предназначены для упрощения создания надежных клиентов. Свойство `watermark`, которое отправляется в ответе Get Messages, является надежным. Клиент не пропустит ни одного сообщения, пока воспроизводит водяной знак дословно.

Клиенты должны выбрать интервал опроса, который соответствует их предполагаемому использованию.

- Приложения типа "служба — служба" часто используют интервал опроса, равный 5 или 10 с.

- Приложения, взаимодействующие с клиентами, часто используют интервал опроса, равный 1 с, и выдают дополнительный запрос приблизительно через 300 мс после каждого сообщения, отправленного клиентом (для быстрого получения ответа от бота). Эта задержка в 300 мс настраивается в зависимости от времени передачи и скорости бота.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-1-1-concepts.md)
- [Аутентификация](bot-framework-rest-direct-line-1-1-authentication.md)
- [Начало общения](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [Отправка сообщения боту](bot-framework-rest-direct-line-1-1-send-message.md)
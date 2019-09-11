---
title: Отправка сообщения боту | Документация Майкрософт
description: Узнайте, как отправлять сообщения боту, используя Direct Line API 1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 461ea04e0865991c3a6d643db7511e58d516ec41
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299608"
---
# <a name="send-a-message-to-the-bot"></a>Отправка сообщения боту

> [!IMPORTANT]
> В этой статье описывается отправка сообщения боту с помощью Direct Line API 1.1. При создании подключения между клиентским приложением и ботом используйте вместо этого [Direct Line API 3.0](bot-framework-rest-direct-line-3-0-send-activity.md).

С помощью протокола Direct Line 1.1 клиенты могут обмениваться сообщениями с ботами. Эти сообщения преобразовываются в схему, поддерживаемую ботом (Bot Framework версии 1 или 3). В каждом запросе клиент может отправить одно сообщение. 

## <a name="send-a-message"></a>Отправка сообщения

Для отправки сообщения боту клиент должен создать объект [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) для определения сообщения, а затем отправить запрос `POST` на адрес `https://directline.botframework.com/api/conversations/{conversationId}/messages`, указав объект Message в теле запроса.

Ниже приведены примеры фрагментов кода для запроса отправки сообщения и соответствующего ответа.

### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/api/conversations/abc123/messages
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
[other headers]
```

```json
{
  "text": "hello",
  "from": "user1"
}
```

### <a name="response"></a>Ответ

После доставки сообщения боту служба отвечает кодом состояния HTTP, который отражает код состояния бота. Если бот выдает ошибку, клиенту в ответ на запрос отправки сообщения возвращается ответ HTTP 500 ("Внутренняя ошибка сервера"). Если вызов POST завершен успешно, служба возвращает код состояния HTTP 204. В тексте ответа данные не возвращаются. Сообщение клиента и любые сообщения от бота можно получить через [опрос](bot-framework-rest-direct-line-1-1-receive-messages.md). 

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a name="total-time-for-the-send-message-requestresponse"></a>Общее время отправки сообщения (отправка запроса и получение ответа)

Общее время для отправки запроса POST в общение Direct Line является суммой следующих значений:

- транзитное время передачи HTTP-запроса с клиента в службу Direct Line;
- время внутренней обработки в Direct Line (обычно меньше 120 мс);
- транзитное время передачи из службы Direct Line в бот;
- время обработки в боте;
- транзитное время обратной передачи ответа HTTP клиенту.

## <a name="send-attachments-to-the-bot"></a>Отправка вложений боту

В некоторых случаях клиенту может потребоваться отправить боту вложения, такие как изображения или документы. Отправка вложений осуществляется либо путем [указания URL-адресов](#send-by-url) вложений в объекте [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object), отправляемом клиентом с помощью `POST /api/conversations/{conversationId}/messages`, либо путем [передачи вложений](#upload-attachments) с помощью `POST /api/conversations/{conversationId}/upload`.

## <a id="send-by-url"></a> Отправка вложений по URL-адресу

Для отправки одного или нескольких вложений как часть объекта [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object) с помощью `POST /api/conversations/{conversationId}/messages`, укажите URL-адреса вложений в массиве `images` и/или массиве `attachments` сообщения.

## <a id="upload-attachments"></a> Отправка вложений путем передачи

Иногда на устройстве клиента могут находиться изображения или документы, которые нужно отправить боту, но URL-адреса, соответствующие этим файлам, отсутствуют. В этом случае клиент может выполнить запрос `POST /api/conversations/{conversationId}/upload`, чтобы отправить вложения боту путем передачи. Формат и содержимое запроса зависят от того, сколько вложений отправляет клиент, — [одно](#upload-one-attachment) или [несколько](#upload-multiple-attachments).

### <a id="upload-one-attachment"></a> Отправка одного вложения путем передачи

Чтобы отправить одно вложение путем передачи, выполните следующий запрос. 

```http
POST https://directline.botframework.com/api/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

В этом URI запроса замените **{conversationId}** на идентификатор общения, а **{userId}** — на идентификатор пользователя, который отправляет сообщение. В заголовках запроса задайте для `Content-Type` тип вложения, а для `Content-Disposition` задайте имя файла вложения.

Ниже приведены примеры фрагментов кода для запроса отправки одного вложения и соответствующего ответа.

#### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>Ответ

Если запрос выполнен успешно, после завершения передачи сообщение отправляется боту, и служба возвращает код состояния HTTP 204.

```http
HTTP/1.1 204 No Content
[other headers]
```

### <a id="upload-multiple-attachments"></a> Отправка нескольких вложений путем передачи

Чтобы отправить несколько вложений путем передачи, отправьте составной запрос `POST` к конечной точке `/api/conversations/{conversationId}/upload`. Задайте `multipart/form-data` в качестве заголовка `Content-Type` запроса и включите заголовок `Content-Type` и заголовок `Content-Disposition` для каждой части, чтобы указать тип и имя файла каждого вложения. В URI запроса задайте параметру `userId` значение идентификатора пользователя, который отправляет сообщение. 

В запрос можно включить объект [Message](bot-framework-rest-direct-line-1-1-api-reference.md#message-object), добавив часть, которая указывает значение `application/vnd.microsoft.bot.message` заголовка `Content-Type`. Это позволяет клиенту настроить сообщение, содержащее вложения. Если запрос содержит объект Message, то перед отправкой этого объекта в него добавляются вложения, заданные другими частями полезных данных. 

Ниже приведены примеры фрагментов кода для запроса отправки нескольких вложений и соответствующего ответа. В этом примере запрос отправляет сообщение, содержащее текст и одно вложение с изображением. Чтобы включить несколько вложений в это сообщение, в запрос можно добавить дополнительные части.

#### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/api/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.bot.message
[other headers]

{
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe",
  "from": "user1"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>Ответ

Если запрос выполнен успешно, после завершения передачи сообщение отправляется боту, и служба возвращает код состояния HTTP 204.

```http
HTTP/1.1 204 No Content
[other headers]
```

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-1-1-concepts.md)
- [Аутентификация](bot-framework-rest-direct-line-1-1-authentication.md)
- [Начало общения](bot-framework-rest-direct-line-1-1-start-conversation.md)
- [Получение сообщений от бота](bot-framework-rest-direct-line-1-1-receive-messages.md)
---
title: Отправка действия боту | Документы Майкрософт
description: Сведения об отправке действия боту с помощью API Direct Line версии 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 290a2733b96a458eb3529b0b0854703631e05f22
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000041"
---
# <a name="send-an-activity-to-the-bot"></a>Отправка действия боту

С помощью протокола Direct Line 3.0 клиенты и боты могут обмениваться различными типами [действий](bot-framework-rest-connector-activities.md), в том числе действиями **сообщений**, **ввода** и настраиваемыми действиями, которые поддерживает бот. В каждом запросе клиент может отправить одно действие. 

## <a name="send-an-activity"></a>Отправка действия

Чтобы отправить действие боту, клиент должен создать объект [Activity](bot-framework-rest-connector-api-reference.md#activity-object) для определения действия, а затем выполнить запрос `POST` к `https://directline.botframework.com/v3/directline/conversations/{conversationId}/activities`, указав объект Activity в теле запроса.

Ниже приведены примеры фрагментов кода для запроса отправки действия и соответствующего ответа.

### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: application/json
[other headers]
```

```json
{
    "type": "message",
    "from": {
        "id": "user1"
    },
    "text": "hello"
}
```

### <a name="response"></a>Ответ

После того как действие доставлено боту, служба отвечает с кодом состояния HTTP, который отражает код состояния бота. Если в боте возникает ошибка, в ответ на запрос отправки действия клиент получает ответ HTTP 502 ("Недопустимый шлюз"). Если запрос POST выполнен успешно, ответ содержит полезные данные JSON, указывающие идентификатор действия, которое было отправлено боту.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "id": "0001"
}
```

### <a name="total-time-for-the-send-activity-requestresponse"></a>Общее время запроса и ответа отправки действия

Общее время для отправки запроса POST в диалог Direct Line является суммой следующих значений:

- транзитное время передачи HTTP-запроса с клиента в службу Direct Line;
- время внутренней обработки в Direct Line (обычно меньше 120 мс);
- транзитное время передачи из службы Direct Line в бот;
- время обработки в боте;
- транзитное время обратной передачи ответа HTTP клиенту.

## <a name="send-attachments-to-the-bot"></a>Отправка вложений боту

В некоторых случаях клиенту может потребоваться отправить боту вложения, такие как изображения или документы. Отправка вложений осуществляется либо путем [указания URL-адресов](#send-by-url) вложений в объекте [Activity](bot-framework-rest-connector-api-reference.md#activity-object), отправляемом клиентом с помощью `POST /v3/directline/conversations/{conversationId}/activities`, либо путем [передачи вложений](#upload-attachments) с помощью `POST /v3/directline/conversations/{conversationId}/upload`.

## <a id="send-by-url"></a> Отправка вложений по URL-адресу

Чтобы отправить одно или несколько вложений в составе объекта [Activity](bot-framework-rest-connector-api-reference.md#activity-object) с помощью `POST /v3/directline/conversations/{conversationId}/activities`, нужно просто включить один или несколько объектов [Attachment](bot-framework-rest-connector-api-reference.md#attachment-object) в объект Activity и задать свойству `contentUrl` каждого объекта Attachment значения HTTP, HTTPS или URI `data` вложения.

## <a id="upload-attachments"></a> Отправка вложений путем передачи

Иногда на устройстве клиента могут находиться изображения или документы, которые нужно отправить боту, но URL-адреса, соответствующие этим файлам, отсутствуют. В этом случае клиент может выполнить запрос `POST /v3/directline/conversations/{conversationId}/upload`, чтобы отправить вложения боту путем передачи. Формат и содержимое запроса зависят от того, сколько вложений отправляет клиент, — [одно](#upload-one-attachment) или [несколько](#upload-multiple-attachments).

### <a id="upload-one-attachment"></a> Отправка одного вложения путем передачи

Чтобы отправить одно вложение путем передачи, выполните следующий запрос. 

```http
POST https://directline.botframework.com/v3/directline/conversations/{conversationId}/upload?userId={userId}
Authorization: Bearer SECRET_OR_TOKEN
Content-Type: TYPE_OF_ATTACHMENT
Content-Disposition: ATTACHMENT_INFO
[other headers]

[file content]
```

В этом URI запроса замените **{conversationId}** на идентификатор общения, а **{userId}** — на идентификатор пользователя, который отправляет сообщение. Параметр `userId` является обязательным. В заголовках запроса задайте для `Content-Type` тип вложения, а для `Content-Disposition` задайте имя файла вложения.

Ниже приведены примеры фрагментов кода для запроса отправки одного вложения и соответствующего ответа.

#### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: image/jpeg
Content-Disposition: name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]
```

#### <a name="response"></a>Ответ

Если запрос выполнен успешно, после завершения передачи боту отправляется **сообщение** с действием. А ответ, полученный клиентом, будет содержать идентификатор отправленного действия.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "id": "0003"
}
```

### <a id="upload-multiple-attachments"></a> Отправка нескольких вложений путем передачи

Чтобы отправить несколько вложений путем передачи, отправьте составной запрос `POST` к конечной точке `/v3/directline/conversations/{conversationId}/upload`. Задайте `multipart/form-data` в качестве заголовка `Content-Type` запроса и включите заголовок `Content-Type` и заголовок `Content-Disposition` для каждой части, чтобы указать тип и имя файла каждого вложения. В URI запроса задайте параметру `userId` значение идентификатора пользователя, который отправляет сообщение. 

В запрос можно включить объект [Activity](bot-framework-rest-connector-api-reference.md#activity-object), добавив часть, которая указывает значение `application/vnd.microsoft.activity` заголовка `Content-Type`. Если запрос содержит объект Activity, то перед отправкой этого объекта в него добавляются вложения, заданные другими частями полезных данных. Если в запросе отсутствует объект Activity, создается пустой объект Activity, который будет контейнером для сбора отправляемых указанных вложений.

Ниже приведены примеры фрагментов кода для запроса отправки нескольких вложений и соответствующего ответа. В этом примере запрос отправляет сообщение, содержащее текст и одно вложение с изображением. Чтобы включить несколько вложений в это сообщение, в запрос можно добавить дополнительные части.

#### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/upload?userId=user1
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
Content-Type: multipart/form-data; boundary=----DD4E5147-E865-4652-B662-F223701A8A89
[other headers]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: image/jpeg
Content-Disposition: form-data; name="file"; filename="badjokeeel.jpg"
[other headers]

[JPEG content]

----DD4E5147-E865-4652-B662-F223701A8A89
Content-Type: application/vnd.microsoft.activity
[other headers]

{
  "type": "message",
  "from": {
    "id": "user1"
  },
  "text": "Hey I just IM'd you\n\nand this is crazy\n\nbut here's my webhook\n\nso POST me maybe"
}

----DD4E5147-E865-4652-B662-F223701A8A89
```

#### <a name="response"></a>Ответ

Если запрос выполнен успешно, после завершения передачи боту отправляется сообщение с действием. А ответ, полученный клиентом, будет содержать идентификатор отправленного действия.

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
- [Начало общения](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [Повторное подключение к диалогу](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md)
- [Получение действий от бота](bot-framework-rest-direct-line-3-0-receive-activities.md)
- [Завершение диалога](bot-framework-rest-direct-line-3-0-end-conversation.md)

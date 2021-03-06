---
title: Справочник по программному интерфейсу Direct Line API 1.1 — Служба Azure Bot
description: Узнайте о заголовках, кодах состояния HTTP, схеме, операциях и объектах в Direct Line API 1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 6cb2d9cd933952e363631d64f527b4c12d5f3b40
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791394"
---
# <a name="api-reference---direct-line-api-11"></a>Справочник по программному интерфейсу Direct Line API 1.1

> [!IMPORTANT]
> Эта статья содержит справочные сведения для Direct Line API 1.1. При создании подключения между клиентским приложением и ботом используйте [API версии 3.0 для Direct Line](bot-framework-rest-direct-line-3-0-api-reference.md).

C помощью Direct Line API 1.1 можно взаимодействовать с ботом в клиентском приложении. Direct Line API 1.1 использует стандартные отраслевые REST и JSON по протоколу HTTPS.

## <a name="base-uri"></a>Базовый универсальный код ресурса

Для доступа к Direct Line API 1.1 используйте этот базовый URI для всех запросов API.

`https://directline.botframework.com`

## <a name="headers"></a>Заголовки

Кроме стандартных заголовков HTTP-запроса запрос Direct Line API должен включать заголовок `Authorization`, определяющий секрет или токен для аутентификации клиента, который выполняет запрос. Можно указать заголовок `Authorization` с помощью схем "Bearer" или "BotConnector". 

**Схема Bearer**
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**Схема BotConnector**
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

Дополнительные сведения о том, как получить секрет или токен, который клиент может использовать для аутентификации запросов Direct Line API, см. в руководстве по [аутентификации](bot-framework-rest-direct-line-1-1-authentication.md).

## <a name="http-status-codes"></a>Коды состояния HTTP

<a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">Код состояния HTTP</a>, который возвращается с каждым ответом, указывает результат соответствующего запроса. 

| HTTP status code (Код состояния HTTP) | Значение |
|----|----|
| 200 | Запрос выполнен успешно. |
| 204 | Запрос успешно выполнен, но содержимое не было возвращено. |
| 400 | Запрос неправильный или имеет недопустимый формат. |
| 401 | Клиент не авторизован для выполнения запроса. Часто этот код состояния возникает, когда заголовок `Authorization` отсутствует или имеет недопустимый формат. |
| 403 | Клиенту запрещено выполнять запрошенную операцию. Часто этот код состояния возникает, потому что заголовок `Authorization` указывает недопустимый токен или секрет. |
| 404 | Запрошенный ресурс не найден. Обычно этот код состояния обозначает, что в запросе указан недопустимый URI. |
| 500 | Внутренняя ошибка сервера в службе Direct Line |
| 502 | В боте произошла ошибка. Бот недоступен или вернул ошибку.  **Это обобщенный код ошибки.** |

## <a name="token-operations"></a>Операции с токенами 
Используйте эти операции для создания или обновления токена, который клиент может использовать для доступа к одному диалогу.

| Операция | Description |
|----|----|
| [Создание токена](#generate-token) | Создает токен для нового диалога. | 
| [Обновление токена](#refresh-token) | Обновляет токен. | 

### <a name="generate-token"></a>Generate Token
Создает токен, действующий для одного диалога. 
```http 
POST /api/tokens/conversation
```

| | |
|----|----|
| **Текст запроса** | Недоступно |
| **Возвращает** | Строка, представляющая токен | 

### <a name="refresh-token"></a>Refresh Token
Обновляет токен. 
```http 
GET /api/tokens/{conversationId}/renew
```

| | |
|----|----|
| **Текст запроса** | Недоступно |
| **Возвращает** | Строка, представляющая новый токен | 

## <a name="conversation-operations"></a>Операции диалога 
Используйте эти операции, чтобы открыть общение с ботом и обменяться сообщениями между клиентом и ботом.

| Операция | Description |
|----|----|
| [Начало диалога](#start-conversation) | Открывает новый диалог с ботом. | 
| [Получение сообщений](#get-messages) | Получает сообщения от бота. |
| [Send a Message](#send-a-message) | Отправка сообщения боту. | 
| [Загрузка и отправка файлов](#upload-send-files) | Загружает и отправляет файлы в виде вложений. |

### <a name="start-conversation"></a>Начало диалога
Открывает новый диалог с ботом. 
```http 
POST /api/conversations
```

| | |
|----|----|
| **Текст запроса** | Недоступно |
| **Возвращает** | Объект [Conversation](#conversation-object). | 

### <a name="get-messages"></a>Get Messages
Получает сообщения от бота для указанного общения. Задайте параметр `watermark` в URI запроса, чтобы указать последнее сообщение, увиденное клиентом. 

```http
GET /api/conversations/{conversationId}/messages?watermark={watermark_value}
```

| | |
|----|----|
| **Текст запроса** | Недоступно |
| **Возвращает** | Объект [MessageSet](#messageset-object). Ответ содержит `watermark` как свойство объекта `MessageSet`. Клиенты должны просматривать доступные сообщения, перемещая значение `watermark` до тех пор, пока сообщения не перестанут возвращаться. | 

### <a name="send-a-message"></a>Send a Message
Отправка сообщения боту. 
```http 
POST /api/conversations/{conversationId}/messages
```

| | |
|----|----|
| **Текст запроса** | Объект [Message](#message-object) |
| **Возвращает** | В тексте ответа данные не возвращаются. Служба отвечает кодом состояния HTTP 204, если сообщение было отправлено успешно. Клиент может получить отправленное сообщение (вместе с любыми сообщениями, которые бот отправил клиенту) с помощью операции [Получить сообщения](#get-messages). |

### <a id="upload-send-files"></a> Загрузка и отправка файлов
Загружает и отправляет файлы в виде вложений. Задайте параметр `userId` в URI запроса, чтобы указать идентификатор пользователя, отправляющего вложения.
```http 
POST /api/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **Текст запроса** | Для одного вложения заполните текст запроса содержимым файла. Для нескольких вложений создайте текст составного запроса, содержащего одну часть для каждого вложения, а также (необязательно) одну часть для объекта [Message](#message-object), который должен служить контейнером для указанного(ых) вложения(й). Дополнительные сведения см. в разделе [Отправка сообщения боту](bot-framework-rest-direct-line-1-1-send-message.md). |
| **Возвращает** | В тексте ответа данные не возвращаются. Служба отвечает кодом состояния HTTP 204, если сообщение было отправлено успешно. Клиент может получить отправленное сообщение (вместе с любыми сообщениями, которые бот отправил клиенту) с помощью операции [Получить сообщения](#get-messages). | 

> [!NOTE]
> Загруженные файлы удаляются через 24 часа.

## <a name="schema"></a>схема

Схема Direct Line 1.1 представляет собой упрощенную копию схемы Bot Framework версии 1, которая включает следующие объекты.

### <a name="message-object"></a>Объект "Message"

Определяет сообщение, которое клиент отправляет боту или получает от бота.

| Свойство | Тип | Description |
|----|----|----|
| **идентификатор** | строка | Идентификатор, который уникально идентифицирует сообщение (назначается службой Direct Line). | 
| **conversationId** | строка | Идентификатор, определяющий общение.  | 
| **created** | строка | Дата и время создания сообщения, выраженные в формате <a href="https://en.wikipedia.org/wiki/ISO_8601" target="_blank">ISO-8601</a>. | 
| **from** | строка | Идентификатор, определяющий пользователя, который является отправителем сообщения. При создании сообщения клиенты должны установить это свойство в стабильный идентификатор пользователя. Несмотря на то что Direct Line назначит идентификатор пользователя, если он не указан, это обычно приводит к непредвиденному поведению. | 
| **text** | строка | Текст сообщения, отправляемого от пользователя к боту или от бота к пользователю. | 
| **channelData** | объект | Объект, содержащий определяемое каналом содержимое. Некоторые каналы предоставляют функции, требующие дополнительных сведений, которые не могут быть представлены с помощью схемы вложения. В таких случаях для этого свойства задается определяемое каналом содержимое, как указано в документации канала. Эти данные отправляются без изменений между клиентом и ботом. Это свойство необходимо присвоить составному объекту, либо оставить поле пустым. Не устанавливайте его на строку, номер или другой простой тип. | 
| **images** | string[] | Массив строк, содержащий URL-адреса для образов, которые содержат сообщение. В некоторых случаях строки в этом массиве могут быть относительными URL-адресами. Если какая-либо строка в этом массиве не начинается с "http" или "https", добавьте `https://directline.botframework.com` к строке, чтобы сформировать полный URL-адрес. | 
| **attachments** | [Attachment](#attachment-object)[] | Массив объектов **Attachment**, которые представляют собой вложения без изображений, содержащиеся в сообщении. Каждый объект в массиве содержит свойство `url` и свойство `contentType`. В сообщениях, которые клиент получает от бота, свойство `url` иногда может указывать относительный URL-адрес. Для любого значения свойства `url`, которое не начинается с "http" или "https", добавьте `https://directline.botframework.com` к строке, чтобы сформировать полный URL-адрес. | 

В следующем примере показан объект "Message", который содержит все возможные свойства. В большинстве случаев при создании сообщения клиенту нужно только предоставить свойство `from` и по меньшей мере одно свойство содержимого (например, `text`, `images`, `attachments` или `channelData`).

```json
{
    "id": "CuvLPID4kDb|000000000000000004",
    "conversationId": "CuvLPID4kDb",
    "created": "2016-10-28T21:19:51.0357965Z",
    "from": "examplebot",
    "text": "Hello!",
    "channelData": {
        "examplefield": "abc123"
    },
    "images": [
        "/attachments/CuvLPID4kDb/0.jpg?..."
    ],
    "attachments": [
        {
            "url": "https://example.com/example.docx",
            "contentType": "application/vnd.openxmlformats-officedocument.wordprocessingml.document"
        }, 
        {
            "url": "https://example.com/example.doc",
            "contentType": "application/msword"
        }
    ]
}
```

### <a name="messageset-object"></a>Объект MessageSet 
Определяет набор сообщений.<br/><br/>

| Свойство | Тип | Description |
|----|----|----|
| **messages** | [Message](#message-object)[] | Массив объектов **Message**. |
| **watermark** | строка | Максимальный водяной знак сообщений в наборе. Клиент может использовать значение `watermark`, чтобы указать последнее сообщение, которое он видел при [получении сообщений от бота](bot-framework-rest-direct-line-1-1-receive-messages.md). |

### <a name="attachment-object"></a>Объект Attachment
Определяет вложение без изображения.<br/><br/> 

| Свойство | Тип | Description |
|----|----|----|
| **contentType** | строка | Тип мультимедиа содержимого во вложении. |
| **url** | строка | URL-адрес для содержимого вложения. |

### <a name="conversation-object"></a>Объект Conversation
Определяет диалог Direct Line.<br/><br/>

| Свойство | Тип | Description |
|----|----|----|
| **conversationId** | строка | Идентификатор, который уникально идентифицирует диалог, для которого действует указанный токен. |
| **token** | строка | Токен, действующий для указанного диалога. |
| **expires_in** | number | Число секунд до истечения срока действия токена. |

### <a name="error-object"></a>Объект ошибки
Определяет ошибку.<br/><br/> 

| Свойство | Тип | Description |
|----|----|----|
| **code** | строка | Код ошибки. Одно из следующих значений: **MissingProperty**, **MalformedData**, **NotFound**, **ServiceError**, **Internal**, **InvalidRange**, **NotSupported**, **NotAllowed**, **BadCertificate**. |
| **message** | строка | Текстовое описание ошибки. |
| **statusCode** | number | Код состояния. |

### <a name="errormessage-object"></a>Объект ErrorMessage
Полезные данные стандартизированного сообщения об ошибке.<br/><br/> 


|        Свойство        |          Тип          |                                 Description                                 |
|------------------------|------------------------|-----------------------------------------------------------------------------|
| <strong>error</strong> | [Ошибка](#error-object) | Объект <strong>Error</strong>, содержащий сведения об ошибке. |


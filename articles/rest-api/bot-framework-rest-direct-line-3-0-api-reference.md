---
title: Справочник по программному интерфейсу Direct Line API 3.0 | Документация Майкрософт
description: Узнайте о заголовках, кодах состояния HTTP, схеме, операциях и объектах в Direct Line API 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 618b2ffe99114679aa5592b816adf6e1b82be83e
ms.sourcegitcommit: eacf1522d648338eebefe2cc5686c1f7866ec6a2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/30/2019
ms.locfileid: "70167184"
---
# <a name="api-reference---direct-line-api-30"></a>Справочник по программному интерфейсу Direct Line API 3.0

C помощью API Direct Line 3.0 можно взаимодействовать с ботом в клиентском приложении. Direct Line API 3.0 использует отраслевые стандарты REST и JSON по протоколу HTTPS.

## <a name="base-uri"></a>Базовый универсальный код ресурса

Для доступа к Direct Line API 3.0 используйте этот базовый URI во всех запросах к API:

`https://directline.botframework.com`

## <a name="headers"></a>Заголовки

Кроме стандартных заголовков HTTP-запроса запрос Direct Line API должен включать заголовок `Authorization`, определяющий секрет или токен для аутентификации клиента, который выполняет запрос. Используйте для заголовка `Authorization` следующий формат:

```http
Authorization: Bearer SECRET_OR_TOKEN
```

Дополнительные сведения о том, как получить секрет или токен, который клиент может использовать для аутентификации запросов Direct Line API, см. в руководстве по [аутентификации](bot-framework-rest-direct-line-3-0-authentication.md).

## <a name="http-status-codes"></a>Коды состояния HTTP

<a href="http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html" target="_blank">Код состояния HTTP</a>, который возвращается с каждым ответом, указывает результат соответствующего запроса.

| HTTP status code (Код состояния HTTP) | Значение |
|----|----|
| 200 | Запрос выполнен успешно. |
| 201 | Запрос выполнен успешно. |
| 202 | Запрос принят для обработки. |
| 204 | Запрос успешно выполнен, но содержимое не было возвращено. |
| 400 | Запрос неправильный или имеет недопустимый формат. |
| 401 | Клиент не авторизован для выполнения запроса. Часто этот код состояния возникает, когда заголовок `Authorization` отсутствует или имеет недопустимый формат. |
| 403 | Клиенту запрещено выполнять запрошенную операцию. Если в запросе указан действительный маркер с истекшим сроком действия, свойство `code` в объекте [Ошибка][], который возвращается в [ErrorResponse][], будет иметь значение `TokenExpired`. |
| 404 | Запрошенный ресурс не найден. Обычно этот код состояния обозначает, что в запросе указан недопустимый URI. |
| 500 | Внутренняя ошибка сервера в службе Direct Line |
| 502 | Бот недоступен или вернул ошибку. **Это обобщенный код ошибки.** |

> [!NOTE]
> Также в пути подключения WebSocket используется код состояния HTTP 101, но, вероятно, его уже обрабатывает ваш клиент WebSocket.

### <a name="errors"></a>Errors

Любой ответ, указывающий код состояния HTTP в диапазоне 4xx или 5xx, будет содержать в тексте ответа объект [ErrorResponse][], который предоставляет сведения об ошибке. Если вы получите сообщение об ошибке в диапазоне 4xx, проверьте объект **ErrorResponse**, чтобы определить причину ошибки и устранить проблему, прежде чем повторно отправлять запрос.

> [!NOTE]
> Коды состояния HTTP и значения в свойстве `code` в объекте **ErrorResponse** являются неизменными. Значения, указанные в свойстве `message` в объекте **ErrorResponse**, могут изменяться со временем.

В следующих фрагментах кода представлены пример запроса и ответ на него, содержащий ошибку.

#### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/v3/directline/conversations/abc123/activities
[detail omitted]
```

#### <a name="response"></a>Ответ

```http
HTTP/1.1 502 Bad Gateway
[other headers]
```

```json
{
    "error": {
        "code": "BotRejectedActivity",
        "message": "Failed to send activity: bot returned an error"
    }
}
```

## <a name="token-operations"></a>Операции с токенами

Используйте эти операции для создания или обновления токена, который клиент может использовать для доступа к одному диалогу.

| Операция | ОПИСАНИЕ |
|----|----|
| [Создание токена](#generate-token) | Создает токен для нового диалога. |
| [Обновление токена](#refresh-token) | Обновляет токен. |

### <a name="generate-token"></a>Generate Token

Создает токен, действующий для одного диалога.

```http
POST /v3/directline/tokens/generate
```

| | |
|----|----|
| **Текст запроса** | Объект [TokenParameters](#tokenparameters-object) |
| **Возвращает** | Объект [Conversation](#conversation-object). |

### <a name="refresh-token"></a>Обновление токена

Обновляет токен.

```http
POST /v3/directline/tokens/refresh
```

| | |
|----|----|
| **Текст запроса** | Недоступно |
| **Возвращает** | Объект [Conversation](#conversation-object). |

## <a name="conversation-operations"></a>Операции диалога

Используйте эти операции, чтобы открыть диалог с ботом и обменяться сообщениями между клиентом и ботом.

| Операция | ОПИСАНИЕ |
|----|----|
| [Начало диалога](#start-conversation) | Открывает новый диалог с ботом. |
| [Получение сведений о диалоге](#get-conversation-information) | Получает сведения о существующем диалоге. Эта операция создает URL-адрес потока WebSocket, который позволяет клиенту [повторно подключиться](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) к диалогу. |
| [Получение действий](#get-activities) | Получает действия от бота. |
| [Отправка действия](#send-an-activity) | Отправляет действие боту. |
| [Загрузка и отправка файлов](#upload-and-send-files) | Загружает и отправляет файлы в виде вложений. |

### <a name="start-conversation"></a>Начало диалога

Открывает новый диалог с ботом.

```http
POST /v3/directline/conversations
```

| | |
|----|----|
| **Текст запроса** | Объект [TokenParameters](#tokenparameters-object) |
| **Возвращает** | Объект [Conversation](#conversation-object). |

### <a name="get-conversation-information"></a>Получение сведений о диалоге

Получает информацию о существующем диалоге и создает URL-адрес потока WebSocket, который позволяет клиенту [повторно подключиться](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) к диалогу. Вы можете включить необязательный параметр `watermark` в URI запроса, чтобы указать последнее сообщение, увиденное клиентом.

```http
GET /v3/directline/conversations/{conversationId}?watermark={watermark_value}
```

| | |
|----|----|
| **Текст запроса** | Недоступно |
| **Возвращает** | Объект [Conversation](#conversation-object). |

### <a name="get-activities"></a>Получение действий

Получает действия от бота для указанного диалога. Вы можете включить необязательный параметр `watermark` в URI запроса, чтобы указать последнее сообщение, увиденное клиентом.

```http
GET /v3/directline/conversations/{conversationId}/activities?watermark={watermark_value}
```

| | |
|----|----|
| **Текст запроса** | Недоступно |
| **Возвращает** | Объект [ActivitySet](#activityset-object). Ответ содержит `watermark` как свойство объекта `ActivitySet`. Клиенты должны постранично просмотреть доступные действия, переходя по значению `watermark`, пока действия не перестанут возвращаться. |

### <a name="send-an-activity"></a>Отправка действия

Отправляет действие боту.

```http
POST /v3/directline/conversations/{conversationId}/activities
```

| | |
|----|----|
| **Текст запроса** | Объект [Действие][] |
| **Возвращает** | Объект [ResourceResponse][], который содержит свойство `id` с идентификатором действия, отправленного боту. |

### <a name="upload-and-send-files"></a>Загрузка и отправка файлов

Загружает и отправляет файлы в виде вложений. Задайте параметр `userId` в URI запроса, чтобы указать идентификатор пользователя, отправляющего вложения.

```http
POST /v3/directline/conversations/{conversationId}/upload?userId={userId}
```

| | |
|----|----|
| **Текст запроса** | Для одного вложения заполните текст запроса содержимым файла. Для нескольких вложений создайте текст запроса из нескольких частей, по одной для каждого вложения, а также (необязательно) одну часть для объекта [Действие][], которая будет служить контейнером для всех вложений. Дополнительные сведения см. в руководстве по [отправке действия боту](bot-framework-rest-direct-line-3-0-send-activity.md). |
| **Возвращает** | Объект [ResourceResponse][], который содержит свойство `id` с идентификатором действия, отправленного боту. |

> [!NOTE]
> Загруженные файлы удаляются через 24 часа.

## <a name="schema"></a>Схема

Схема Direct Line 3.0 включает все объекты, которые определены в [схеме Bot Framework](bot-framework-rest-connector-api-reference.md#schema), а также объекты, которые связаны с Direct Line.

### <a name="activityset-object"></a>Объект ActivitySet

Определяет набор действий.

| Свойство | type | ОПИСАНИЕ |
|----|----|----|
| **действия** | [Действие][][] | Массив объектов **Activity**. |
| **watermark** | строка | Максимальное значение для водяного знака действий в наборе. Клиент может использовать значение `watermark`, чтобы указать последнее полученное им сообщение, при [получении новых действий от бота](bot-framework-rest-direct-line-3-0-receive-activities.md#http-get) или при [создании нового URL-адреса потока WebSocket](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md). |

### <a name="conversation-object"></a>Объект Conversation

Определяет диалог Direct Line.

| Свойство | type | ОПИСАНИЕ |
|----|----|----|
| **conversationId** | строка | Идентификатор, который уникально идентифицирует диалог, для которого действует указанный токен. |
| **eTag** | строка | ETag HTTP (тег сущности). |
| **expires_in** | number | Число секунд до истечения срока действия токена. |
| **referenceGrammarId** | строка | Идентификатор справочной грамматики для этого бота. |
| **streamUrl** | строка | URL-адрес для потока сообщений диалога. |
| **token** | строка | Токен, действующий для указанного диалога. |

### <a name="tokenparameters-object"></a>Объект TokenParameters

Параметры для создания токена.

| Свойство | type | ОПИСАНИЕ |
|----|----|----|
| **eTag** | строка | ETag HTTP (тег сущности). |
| **trustedOrigins** | string[] | Доверенные источники для внедрения в токен. |
| **user** | [ChannelAccount][] | Учетная запись пользователя для внедрения в маркер. |

## <a name="activities"></a>Действия

Для каждого объекта [Действие][], полученного клиентом от бота через Direct Line, выполняется следующее:

- сохраняются карточки, включенные как вложения;
- URL-адреса отправленных вложений подменяются частной ссылкой;
- свойство `channelData` сохраняется без изменений.

Клиенты могут [получать](bot-framework-rest-direct-line-3-0-receive-activities.md) несколько действий от бота как набор действий ([ActivitySet](#activityset-object)).

Когда клиент отправляет боту объект `Activity` через Direct Line, происходит следующее:

- свойство `type` принимает тип отправленного действия (обычно это **message**);
- свойство `from` заполняется идентификатором пользователя, выбранным клиентом;
- вложения могут содержать URL-адреса, ведущие к существующим ресурсам или ресурсам, отправленным через конечную точку вложений Direct Line;
- свойство `channelData` сохраняется без изменений.
- Общий размер действия, сериализованного и зашифрованного в формате JSON, не должен превышать 256 тысяч символов. Поэтому рекомендуется, чтобы количество действий не превышало 150 тысяч. Если требуется больше данных, разбейте действие на несколько составных частей или рассмотрите возможность использования вложений.

В каждом запросе клиент может [отправить](bot-framework-rest-direct-line-3-0-send-activity.md) только одно действие.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Спецификация по действиям Bot Framework](https://aka.ms/botSpecs-activitySchema)

[Действие]: bot-framework-rest-connector-api-reference.md#activity-object
[ChannelAccount]: bot-framework-rest-connector-api-reference.md#channelaccount-object
[Ошибка]: bot-framework-rest-connector-api-reference.md#error-object
[ErrorResponse]: bot-framework-rest-connector-api-reference.md#errorresponse-object
[ResourceResponse]: bot-framework-rest-connector-api-reference.md#resourceresponse-object

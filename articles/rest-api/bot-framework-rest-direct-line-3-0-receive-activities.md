---
title: Получение действий от бота | Документация Майкрософт
description: Узнайте, как получать действия от бота с помощью Direct Line API 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 06/13/2019
ms.openlocfilehash: c99e7ce86415ee1291a92e2684b975fd03c822f7
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252694"
---
# <a name="receive-activities-from-the-bot"></a>Получение действий от бота

С помощью протокола Direct Line 3.0 клиенты могут получать действия через поток `WebSocket` или извлекать действия путем выполнения запросов `HTTP GET`.

## <a name="websocket-vs-http-get"></a>WebSocket и запрос GET HTTP

Потоковая передача WebSocket эффективно передает сообщения клиентам, тогда как интерфейс GET позволяет клиентам точно запрашивать сообщения. Несмотря на то что механизм WebSocket часто является предпочтительным благодаря своей эффективности, механизм GET может быть полезен клиентам, которые не могут использовать WebSocket.

Служба позволяет только 1 подключение WebSocket для диалога. Direct Line может закрыть дополнительные подключения WebSocket со значением причины `collision`.

Не все [типы действий](bot-framework-rest-connector-activities.md) доступны как через WebSocket, так и через запрос GET HTTP. В следующей таблице описана доступность различных типов действий для клиентов, использующих протокол Direct Line.

| тип действия; | Доступность | 
|----|----|
| message | HTTP GET и WebSocket |
| typing | Только WebSocket |
| conversationUpdate | Недоступно для отправки и получения через клиент |
| contactRelationUpdate | Не поддерживается в Direct Line |
| endOfConversation | HTTP GET и WebSocket |
| Все другие типы действий | HTTP GET и WebSocket |

## <a id="connect-via-websocket"></a> Получение действий через потоковую передачу WebSocket

Когда клиент отправляет запрос [Начало общения](bot-framework-rest-direct-line-3-0-start-conversation.md) для открытия общения с помощью бота, ответ службы включает в себя свойство `streamUrl`, которое клиент может впоследствии использовать для подключения через WebSocket. URL-адрес потока предварительно одобрен, поэтому запрос клиента на подключение через WebSocket не требует заголовка `Authorization`.

В следующем примере показан запрос, который использует `streamUrl` для подключения через WebSocket.

```http
-- connect to wss://directline.botframework.com --
GET /v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
Upgrade: websocket
Connection: upgrade
[other headers]
```

Служба возвращает код состояния HTTP 101 ("Переключение протоколов").

```http
HTTP/1.1 101 Switching Protocols
[other headers]
```

### <a name="receive-messages"></a>Получение сообщений

После подключения через WebSocket клиент может получать эти типы сообщений из службы Direct Line.

- Сообщение, содержащее объект [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object), который включает в себя одно или несколько действий и водяной знак (описание приведено ниже).
- Пустое сообщение, которое используется службой Direct Line для подтверждения того, что подключение по-прежнему действительно.
- Дополнительные типы будут определены позже. Эти типы идентифицируются свойствами в корне JSON.

`ActivitySet` содержит сообщения, отправленные ботом и всеми пользователями в общении. В следующем примере показан объект `ActivitySet`, содержащий одно сообщение.

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }
    ],
    "watermark": "0000a-42"
}
```

### <a name="process-messages"></a>Обработка сообщений

Клиент должен отслеживать значение свойства `watermark`, которое оно получает в каждом [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object), чтобы иметь возможность использовать водяной знак для гарантии того, что ни одно сообщение не будет утрачено, если потеряется подключение и потребуется [повторно подключиться к общению](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md). Если клиент получает `ActivitySet`, где свойство `watermark` равняется `null` или отсутствует, он должен игнорировать это значение и не перезаписывать полученный ранее водяной знак.

Клиент должен игнорировать пустые сообщения, которые получает из службы Direct Line.

Клиент может отправить пустое сообщение в службу Direct Line, чтобы проверить подключение. Служба Direct Line проигнорирует пустые сообщения, получаемые от клиента.

Служба Direct Line может принудительно закрыть подключение к WebSocket при определенных условиях. Если клиент не получил действие `endOfConversation`, он может [создать новый URL-адрес потока WebSocket](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md), который можно использовать для повторного подключения к общению. 

Поток WebSocket содержит обновления в реальном времени и недавние сообщения (с момента вызова на подключение через WebSocket), но не включает сообщения, отправленные ранее самого последнего `POST` в `/v3/directline/conversations/{id}`. Для получения более ранних сообщений из общения используйте `HTTP GET`, как описано ниже.

## <a id="http-get"></a> Извлечение действий с помощью запроса GET HTTP

Клиенты, которые не могут использовать WebSocket, могут извлечь действия с помощью запроса `HTTP GET`.

Для получения сообщений из конкретного общения отправьте запрос `GET` в конечную точку `/v3/directline/conversations/{conversationId}/activities`, при необходимости задавая параметр `watermark`, чтобы указать последнее сообщение, отображаемое клиенту. 

В следующих фрагментах кода приведен пример запроса и ответа на получение действий общения. Ответ на получение действий общения содержит `watermark` как свойство [ActivitySet](bot-framework-rest-direct-line-3-0-api-reference.md#activityset-object). Клиенты должны постранично просмотреть доступные действия, переходя по значению `watermark`, пока действия не перестанут возвращаться.

### <a name="request"></a>Запрос

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123/activities?watermark=0001a-94
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Ответ

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
    "activities": [
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0000",
            "from": {
                "id": "user1"
            },
            "text": "hello"
        }, 
        {
            "type": "message",
            "channelId": "directline",
            "conversation": {
                "id": "abc123"
            },
            "id": "abc123|0001",
            "from": {
                "id": "bot1"
            },
            "text": "Nice to see you, user1!"
        }
    ],
    "watermark": "0001a-95"
}
```

## <a name="timing-considerations"></a>Рекомендации относительно расписания

Большинство клиентов хотят сохранить полный журнал сообщений. Несмотря на то что Direct Line — это составной протокол с потенциальными пробелами в расписании, протокол и служба предназначены для упрощения создания надежных клиентов.

- Свойство `watermark`, которое отправляется в поток WebSocket, и ответ на получение действий общения являются надежными. Клиент не пропустит ни одного сообщения, пока воспроизводит водяной знак дословно.

- Когда клиент начинает общение и подключается к потоку WebSocket, все действия, которые отправляются после команды POST, но перед открытием сокета, воспроизводятся перед новыми действиями.

- Когда клиент издает запрос "Получить активности общения" (для обновления истории), в то время как он подключен к потоку WebSocket, действия могут дублироваться по обоим каналам. Клиенты должны отслеживать все известные идентификаторы действий, чтобы иметь возможность отклонить повторяющиеся действия, если они встречаются.

Клиентам, проводящим опрос с помощью `HTTP GET`, следует выбирать интервал опроса, который соответствует их предполагаемому использованию.

- Приложения типа "служба — служба" часто используют интервал опроса, равный 5 или 10 с.

- Приложения, взаимодействующие с клиентами, часто используют интервал опроса, равный 1 с, и выдают один дополнительный запрос вскоре после каждого сообщения, отправленного клиентом (для быстрого получения ответа от бота). Минимальное значение этой задержки составляет 300 мс, но ее настоятельно рекомендуется настроить с учетом скорости бота и транзитного времени. Для любого продолжительного периода времени опрос не следует проводить чаще, чем один раз в секунду.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-3-0-concepts.md)
- [Аутентификация](bot-framework-rest-direct-line-3-0-authentication.md)
- [Начало общения](bot-framework-rest-direct-line-3-0-start-conversation.md)
- [Reconnect to a conversation](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) (Повторное подключение к общению)
- [Отправка действий боту](bot-framework-rest-direct-line-3-0-send-activity.md)
- [Конец общения](bot-framework-rest-direct-line-3-0-end-conversation.md)

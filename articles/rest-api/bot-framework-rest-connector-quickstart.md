---
title: Создание ботов с помощью службы Bot Connector — Служба Azure Bot
description: Создание ботов с помощью службы Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: f5ac5397d2ef13eb08d92d9cf560fddf582165c4
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071882"
---
# <a name="create-a-bot-with-the-bot-connector-service"></a>Создание ботов с помощью службы Bot Connector
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Служба Azure Bot](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Служба Bot Connector позволяет боту обмениваться сообщениями с каналами, настроенными на [портале Azure](https://portal.azure.com), используя стандартные отраслевые форматы REST и JSON по протоколу HTTPS. В этом руководстве подробно объясняется, как получить маркер доступа от Bot Framework и использовать службу Bot Connector для обмена сообщениями с пользователем.

## <a id="get-token"></a> Получение маркера доступа

> [!IMPORTANT]
> [Зарегистрируйте бот](../bot-service-quickstart-registration.md) в Bot Framework (если еще не сделали этого), чтобы получить идентификатор приложения и пароль. Они потребуются для получения маркера доступа.

Для взаимодействия со службой Bot Connector необходимо указать маркер доступа в заголовке `Authorization` каждого запроса API, используя следующий формат: 

```http
Authorization: Bearer ACCESS_TOKEN
```

Вы можете получить маркер доступа для вашего бота, выполнив запрос API.

### <a name="request"></a>Запрос

Чтобы запросить маркер доступа для аутентификации запросов к службе Bot Connector, выполните приведенный ниже запрос, заменив **MICROSOFT-APP-ID** и **MICROSOFT-APP-PASSWORD** идентификатором приложения и паролем, полученными при [регистрации](../bot-service-quickstart-registration.md) бота в Bot Framework.

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="response"></a>Ответ

Если запрос выполнен успешно, вы получите ответ HTTP 200, в котором указан маркер доступа и сведения о его сроке действия. 

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

> [!TIP]
> Дополнительные сведения об аутентификации в службе Bot Connector см. в [этой статье](bot-framework-rest-connector-authentication.md).

## <a name="exchange-messages-with-the-user"></a>Обмен сообщениями с пользователем

Диалог — это последовательность сообщений, передаваемых между пользователем и ботом. 

### <a name="receive-a-message-from-the-user"></a>Получение сообщения от пользователя

Когда пользователь отправляет сообщение, Bot Framework Connector отсылает запрос POST к конечной точке, указанной вами при [регистрации](../bot-service-quickstart-registration.md) бота. Текст запроса — это объект [Действие][]. В представленном ниже примере приведен текст запроса, который бот получает, когда пользователь отправляет ему простое сообщение. 

```json
{
    "type": "message",
    "id": "bf3cc9a2f5de...",
    "timestamp": "2016-10-19T20:17:52.2891902Z",
    "serviceUrl": "https://smba.trafficmanager.net/apis",
    "channelId": "channel's name/id",
    "from": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
    },
    "recipient": {
        "id": "12345678",
        "name": "bot's name"
    },
    "text": "Haircut on Saturday"
}
```

### <a name="reply-to-the-users-message"></a>Ответ на сообщение пользователя

Когда конечная точка бота получит запрос `POST`, который представляет собой сообщение от пользователя (т. е. `type` = **message**), используйте сведения из этого запроса, чтобы создать объект [Действие][] для ответа.

1. Задайте свойство **conversation** с учетом содержимого свойства **conversation** в сообщении пользователя.
2. Укажите свойство **from** с учетом содержимого свойства **recipient** в сообщении пользователя.
3. Задайте свойство **recipient** с учетом содержимого свойства **from** в сообщении пользователя.
4. Укажите свойства **text** и **attachments** соответствующим образом.

Используйте свойство `serviceUrl` во входящем запросе, чтобы [определить базовый URI](bot-framework-rest-connector-api-reference.md#base-uri), который бот будет применять для отправки ответа. 

Чтобы отправить ответ, отправьте запрос `POST` с объектом `Activity` по адресу `/v3/conversations/{conversationId}/activities/{activityId}`, как показано в примере ниже. Текст этого запроса — это объект `Activity`, в котором пользователю предлагается выбрать доступное время встречи.

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "text": "I have these times available:",
    "replyToId": "bf3cc9a2f5de..."
}
```

В этом примере запрос `https://smba.trafficmanager.net/apis` представляет базовый URI. Базовый URI для запросов, отправляемых вашим ботом, может отличаться. Дополнительные сведения о настройке базового URI см. в статье [Справочник по API](bot-framework-rest-connector-api-reference.md#base-uri). 

> [!IMPORTANT]
> Как показано в этом примере, заголовок `Authorization` каждого отправляемого вами запроса API, должен содержать слово **Bearer**, после которого указывается маркер доступа, [полученный от Bot Framework](#get-token).

Чтобы отправить другое сообщение, в котором пользователь может нажатием кнопки выбрать доступное время встречи, отправьте другой запрос `POST` к той же конечной точке:

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/bf3cc9a2f5de... 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "bot's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "user's name"
    },
    "attachmentLayout": "list",
    "attachments": [
      {
        "contentType": "application/vnd.microsoft.card.thumbnail",
        "content": {
          "buttons": [
            {
              "type": "imBack",
              "title": "10:30",
              "value": "10:30"
            },
            {
              "type": "imBack",
              "title": "11:30",
              "value": "11:30"
            },
            {
              "type": "openUrl",
              "title": "See more",
              "value": "http://www.contososalon.com/scheduling"
            }
          ]
        }
      }
    ],
    "replyToId": "bf3cc9a2f5de..."
}
```   

## <a name="next-steps"></a>Дальнейшие действия

При работе с этим руководством вы получили маркер доступа от Bot Framework и использовали службу Bot Connector для обмена сообщениями с пользователем. Для тестирования и отладки бота можно использовать [Bot Framework Emulator](../bot-service-debug-emulator.md). Чтобы предоставить доступ к боту другим пользователям, нужно [настроить](../bot-service-manage-channels.md) его для работы с одним или несколькими каналами.

[Действие]: bot-framework-rest-connector-api-reference.md#activity-object

---
title: Добавление подсказок ввода к сообщениям | Документация Майкрософт
description: Сведения о добавлении подсказок ввода к сообщениям с помощью службы Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: cee7e79190d967590296ccbcfec7a112f2ae8588
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998597"
---
# <a name="add-input-hints-to-messages"></a>Добавление подсказок для ввода в сообщения
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

Задав подсказку для ввода, можно указать, какое действие бот выполнит после того, как сообщение будет доставлено клиенту: примет, будет ожидать или пропустит ввод данных пользователем. Для многих каналов это позволяет клиентам соответствующим образом задавать состояние элементов управления ввода данных пользователем. Например, если подсказка для ввода указывает, что бот пропускает ввод данных пользователем, клиент может закрыть микрофон и отключить поле ввода, чтобы предотвратить ввод данных пользователем.

## <a name="accepting-input"></a>Принятие ввода данных

Чтобы указать, что бот является пассивным наблюдателем ввода входных данных, но не ожидает от пользователя ответа, установите свойству `inputHint` значение **acceptingInput**, где объект [Действие][Activity] будет представлять сообщение. В результате во многих каналах поле ввода будет включено, а микрофон закрыт, но доступен пользователю. Например, Кортана откроет микрофон, чтобы принять входные данные пользователя, если пользователь будет удерживать кнопку микрофона. 

В следующем примере показан запрос, который отправляет сообщение, и указывает, что бот готов принять входные данные. В этом примере запроса `https://smba.trafficmanager.net/apis` представляет базовый URI. Базовый URI для запросов, отправляемых вашим ботом, может отличаться. Дополнительные сведения о настройке базового URI см. в статье [Справочник по API](bot-framework-rest-connector-api-reference.md#base-uri).

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Here's a picture of the house I was telling you about.",
    "inputHint": "acceptingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="expecting-input"></a>Ожидание ввода данных

Чтобы указать, что бот ожидает от пользователя ответа, установите свойству `inputHint` значение **expectingInput**, где объект [Действие][Activity] будет представлять сообщение. В результате во многих каналах поле ввода будет включено, а микрофон открыт. 

В следующем примере показан запрос, который отправляет сообщение и указывает, что бот ожидает ввода. В этом примере запроса `https://smba.trafficmanager.net/apis` представляет базовый URI. Базовый URI для запросов, отправляемых вашим ботом, может отличаться. Дополнительные сведения о настройке базового URI см. в статье [Справочник по API](bot-framework-rest-connector-api-reference.md#base-uri).

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "What is your favorite color?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="ignoring-input"></a>Игнорирование ввода данных
 
Чтобы указать, что бот не готов принимать входные данные от пользователя, установите свойству `inputHint` значение **ignoringInput**, где объект [Действие][Activity] будет представлять сообщение. В результате во многих каналах поле ввода будет отключено, а микрофон закрыт. 

В следующем примере показан запрос, который отправляет сообщение, и указывает, что бот игнорирует входные данные. В этом примере запроса `https://smba.trafficmanager.net/apis` представляет базовый URI. Базовый URI для запросов, отправляемых вашим ботом, может отличаться. Дополнительные сведения о настройке базового URI см. в статье [Справочник по API](bot-framework-rest-connector-api-reference.md#base-uri).

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/abcd1234/activities/5d5cdc723
Authorization: Bearer ACCESS_TOKEN
Content-Type: application/json
```

```json
{
    "type": "message",
    "from": {
        "id": "12345678",
        "name": "sender's name"
    },
    "conversation": {
        "id": "abcd1234",
        "name": "conversation's name"
   },
   "recipient": {
        "id": "1234abcd",
        "name": "recipient's name"
    },
    "text": "Please hold while I perform the calculation.",
    "inputHint": "ignoringInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Создание сообщений](bot-framework-rest-connector-create-messages.md)
- [Отправка и получение сообщений](bot-framework-rest-connector-send-and-receive-messages.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
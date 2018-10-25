---
title: Добавление речи в сообщения | Документация Майкрософт
description: Сведения о добавлении речи в сообщения с помощью службы Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 2aac000b7e8dd52b00659ffecde5184df6c29991
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998691"
---
# <a name="add-speech-to-messages"></a>Добавление речи в сообщения
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

При создании бота для канала с поддержкой речевых функций, такого как Cortana, можно сформировать сообщения, в которых указывается произносимый ботом текст. Можно также попытаться повлиять на состояние микрофона клиента, задав [подсказку для ввода](bot-framework-rest-connector-add-input-hints.md), чтобы указать, что бот принимает, ожидает или игнорирует ввод данных пользователем.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>Указание текста, произносимого ботом

Чтобы указать текст, который бот будет произносить по каналу с поддержкой речи, задайте свойство `speak` в объекте [Activity][Activity], который представляет ваше сообщение. Можно установить свойство `speak` в текстовую строку или строку, которая отформатирована как <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Speech Synthesis Markup Language (SSML)</a> (Язык разметки синтеза речи) — язык разметки на основе XML, который позволяет управлять различными характеристиками речи бота, такими как голос, скорость, громкость, произношение, тон и другое. 

В следующем примере запроса показана отправка сообщения, которое задает текст для отображения и текст для произнесения и указывает, что бот [принимает входные данные пользователя](bot-framework-rest-connector-add-input-hints.md). Он определяет свойство `speak`, используя формат <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a>, чтобы указать, что слово "sure" должно быть произнесено с умеренным количеством акцентов. В этом примере запроса `https://smba.trafficmanager.net/apis` представляет базовый URI. Базовый URI для запросов, отправляемых вашим ботом, может отличаться. Дополнительные сведения о настройке базового URI см. в статье [Справочник по API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "text": "Are you sure that you want to cancel this transaction?",
    "speak": "Are you <emphasis level='moderate'>sure</emphasis> that you want to cancel this transaction?",
    "inputHint": "expectingInput",
    "replyToId": "5d5cdc723"
}
```

## <a name="input-hints"></a>Подсказки для ввода

При отправке сообщения по каналу с поддержкой речевых функций можно попытаться оказать влияние на состояние микрофона клиента, включив подсказку для ввода, указывающую, как бот реагирует на ввод данных пользователем — принимает, ожидает или игнорирует. Дополнительные сведения см. в статье [Добавление подсказок для ввода в сообщения](bot-framework-rest-connector-add-input-hints.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Создание сообщений](bot-framework-rest-connector-create-messages.md)
- [Отправка и получение сообщений](bot-framework-rest-connector-send-and-receive-messages.md)
- [Добавление подсказок для ввода в сообщения](bot-framework-rest-connector-add-input-hints.md)
- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Язык разметки синтеза речи (Speech Synthesis Markup Language, SSML)</a>

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

---
title: Добавление речи в сообщения | Документация Майкрософт
description: Сведения о добавлении речи в сообщения с помощью службы Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: e5d0ee3ffd97de190c4e009fe96568b1463c07ab
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037464"
---
# <a name="add-speech-to-messages"></a>Добавление речи в сообщения
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

При создании бота для канала с поддержкой речевых функций, такого как Cortana, можно сформировать сообщения, в которых указывается произносимый ботом текст. Можно также попытаться повлиять на состояние микрофона клиента, задав [подсказку для ввода](bot-framework-rest-connector-add-input-hints.md), чтобы указать, что бот принимает, ожидает или игнорирует ввод данных пользователем.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>Указание текста, произносимого ботом

Чтобы указать текст, который бот будет произносить по каналу с поддержкой речи, задайте свойство `speak` в объекте [Действие][], который представляет ваше сообщение. Можно установить свойство `speak` в текстовую строку или строку, которая отформатирована как <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-synthesis-markup" target="_blank">Speech Synthesis Markup Language (SSML)</a> (Язык разметки синтеза речи) — язык разметки на основе XML, который позволяет управлять различными характеристиками речи бота, такими как голос, скорость, громкость, произношение, тон и другое. 

В следующем примере запроса показана отправка сообщения, которое задает текст для отображения и текст для произнесения и указывает, что бот [принимает входные данные пользователя](bot-framework-rest-connector-add-input-hints.md). Он определяет свойство `speak`, используя формат <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-synthesis-markup" target="_blank">SSML</a>, чтобы указать, что слово "sure" должно быть произнесено с умеренным количеством акцентов. В этом примере запрос `https://smba.trafficmanager.net/apis` представляет базовый URI. Базовый URI для запросов, отправляемых вашим ботом, может отличаться. Дополнительные сведения о настройке базового URI см. в статье [Справочник по API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "speak": "<speak version=\"1.0\" xmlns=\"http://www.w3.org/2001/10/synthesis\" xml:lang=\"en-US\">Are you <emphasis level=\"moderate\">sure</emphasis> that you want to cancel this transaction?</speak>",
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
- [Принципы использования действий в Bot Framework](https://aka.ms/botSpecs-activitySchema)
- <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-synthesis-markup" target="_blank">Язык разметки синтеза речи (Speech Synthesis Markup Language, SSML)</a>

[Действие]: bot-framework-rest-connector-api-reference.md#activity-object

---
title: Отправка и получение текстовых сообщений | Документация Майкрософт
description: Узнайте, как отправлять и принимать текстовые сообщения с помощью пакета SDK Bot Framework.
keywords: sending message, message activities, simple text message, message, text message, receive message
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4b2cd49d84ea90f0ac6449ce4da61495100d45c4
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215366"
---
# <a name="send-and-receive-text-message"></a>Отправка и получение текстовых сообщений

[!INCLUDE[applies-to](../includes/applies-to.md)]

Основным способом взаимодействия бота с пользователями будут действия с **сообщениями**. Некоторые сообщения будут состоять из обычного текста, а другие — включать расширенное содержимое, такое как карточки или вложения. Обработчик этапа бота получает сообщения от пользователя, и вы можете отправлять ответы пользователю из него. Объект контекста этапа предоставляет методы для отправки сообщений обратно пользователю. В этой статье объясняется, как отправлять простые текстовые сообщения.

Markdown поддерживается для большинства текстовых полей, но особенности поддержки зависят от канала.

## <a name="send-a-text-message"></a>Отправка текстового сообщения

Для отправки простого текстового сообщения укажите строку, которую вы хотите отправить как действие:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В обработчиках действий бота используйте метод `SendActivityAsync` для объекта контекста шага, чтобы отправить ответ в виде одного сообщения. Кроме того, вы можете использовать метод `SendActivitiesAsync` объекта для отправки нескольких ответов за раз.

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В обработчиках действий бота используйте метод `sendActivity` для объекта контекста шага, чтобы отправить ответ в виде одного сообщения. Кроме того, вы можете использовать метод `sendActivities` объекта для отправки нескольких ответов за раз.

```javascript
await context.sendActivity("Welcome!");
```
---
## <a name="receive-a-text-message"></a>Получение текстового сообщения

Чтобы получить простое текстовое сообщение, используйте свойство *text* объекта *activity*. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В обработчиках действий бота используйте следующий код для получения сообщения. 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В обработчиках действий бота используйте следующий код для получения сообщения.

```javascript
let text = turnContext.activity.text;
```

---

## <a name="additional-resources"></a>Дополнительные ресурсы

- Дополнительные сведения об обработке действий в целом см. в разделе [Обработка действий](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack).
- См. дополнительные сведения о схеме действий Bot Framework и [действиях в беседах](https://aka.ms/botSpecs-activitySchema#message-activity).

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Добавление мультимедиа в сообщения](./bot-builder-howto-add-media-attachments.md)

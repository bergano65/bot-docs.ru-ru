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
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9cfe077c8d8573145625b211c3c1ca05a6a21e19
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224819"
---
# <a name="send-and-receive-text-message"></a>Отправка и получение текстовых сообщений 

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Основным способом взаимодействия бота с пользователями будут действия с **сообщениями**. Некоторые сообщения будут состоять из обычного текста, а другие — включать расширенное содержимое, такое как карточки или вложения. Обработчик этапа бота получает сообщения от пользователя, и вы можете отправлять ответы пользователю из него. Объект контекста этапа предоставляет методы для отправки сообщений обратно пользователю. В этой статье объясняется, как отправлять простые текстовые сообщения.

## <a name="send-a-text-message"></a>Отправка текстового сообщения

Для отправки простого текстового сообщения укажите строку, которую вы хотите отправить как действие:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В методе `OnTurnAsync` бота используйте метод `SendActivityAsync` для объекта контекста шага, чтобы отправить ответ в виде одного сообщения. Кроме того, вы можете использовать метод `SendActivitiesAsync` объекта для отправки нескольких ответов за раз.

```cs
await turnContext.SendActivityAsync($"Welcome!");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В обработчике `onTurn` бота используйте метод `sendActivity` для объекта контекста шага, чтобы отправить ответ в виде одного сообщения. Кроме того, вы можете использовать метод `sendActivities` объекта для отправки нескольких ответов за раз.

```javascript
await context.sendActivity("Welcome!");
```
---
## <a name="receive-a-text-message"></a>Получение текстового сообщения

Чтобы получить простое текстовое сообщение, используйте свойство *text* объекта *activity*. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В методе `OnTurnAsync` бота используйте приведенный ниже код для получения сообщения. 

```cs
var responseMessage = turnContext.Activity.Text;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В методе `OnTurnAsync` бота используйте приведенный ниже код для получения сообщения. 
```javascript
let text = turnContext.activity.text;
```
---


## <a name="additional-resources"></a>Дополнительные ресурсы
Дополнительные сведения об обработке действий в целом см. в разделе [Обработка действий](~/v4sdk/bot-builder-basics.md#the-activity-processing-stack). Сведения об отправке сообщений с расширенным содержимым см. в статье о [добавлении мультимедийных вложений](bot-builder-howto-add-media-attachments.md).

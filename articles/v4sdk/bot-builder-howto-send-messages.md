---
title: Отправка сообщений с помощью пакета SDK построителя ботов | Документы Майкрософт
description: Сведения об отправке сообщений с помощью пакета SDK построителя ботов.
keywords: отправка сообщения, действия с сообщениями, простое текстовое сообщение, речь, голосовое сообщение
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/04/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b854eeb4ecb45a875ab3be5d867333cd63babffd
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305559"
---
# <a name="sending-messages"></a>Отправка сообщений

Основным способом взаимодействия бота с пользователями будут действия с **сообщениями**. Некоторые сообщения будут состоять из обычного текста, а другие — включать расширенное содержимое, такое как карточки или вложения. Обработчик этапа бота получает сообщения от пользователя, и вы можете отправлять ответы пользователю из него. Объект контекста этапа предоставляет методы для отправки сообщений обратно пользователю. Дополнительные сведения об обработке действий в целом см. в разделе [Обработка действий](bot-builder-concept-activity-processing.md).

В этой статье описывается, как отправить простые текстовые и голосовые сообщения. Сведения об отправке сообщений с расширенным содержимым см. в статье о [добавлении мультимедийных вложений](bot-builder-howto-add-media-attachments.md). Сведения о работе с объектами запроса см. в статье о том, как [запрашивать ввод у пользователей](bot-builder-prompts.md).

## <a name="send-a-simple-text-message"></a>Отправка простого текстового сообщения

Для отправки простого текстового сообщения укажите строку, которую вы хотите отправить как действие:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В методе **OnTurn** бота используйте объект контекста этапа **SendActivity** для отправки ответа в виде одного сообщения. Можно также использовать метод **SendActivities** объекта для отправки нескольких ответов за раз.

```cs
await context.SendActivity("Greetings from sample message.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В обработчике этапа бота используйте метод **sendActivity** объекта контекста этапа для отправки ответа. Можно также использовать метод **SendActivities** объекта для отправки нескольких ответов за раз.

```javascript
await context.sendActivity("Greetings from sample message.");
```

---

## <a name="send-a-spoken-message"></a>Отправка голосовых сообщений

Некоторые каналы могут использовать боты с поддержкой речевых функций, что позволяет им "общаться" с пользователем. Такое общение может включать как письменное, так и голосовое содержимое.

> [!NOTE]
> Для каналов, которые не могут использовать ботов с речевой функцией, голосовое содержимое игнорируется.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Используйте необязательный параметр **speak** для ввода произнесенного текста как части ответа.

```cs
await context.SendActivity(
    "This is the text to be displayed.",
    "This is the text to be spoken.");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы добавить речь, вам потребуется `Microsoft.Bot.Builder.MessageFactory` для создания сообщений. `MessageFactory` чаще используется с [мультимедийными вложениями](bot-builder-howto-add-media-attachments.md) (в описании которых работа с ним объясняется более подробно), однако мы используем его здесь.

```javascript
// Require MessageFactory from botbuilder
const {MessageFactory} = require('botbuilder');

const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.');
await context.sendActivity(basicMessage);
```

---
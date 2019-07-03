---
title: Создание сообщений | Документация Майкрософт
description: Узнайте, как создавать сообщения с помощью пакета SDK Bot Framework для Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/7/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 36df5898f4a5c02253aae04b3b85dbe70fc21ada
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404876"
---
# <a name="create-messages"></a>Создание сообщений

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Обмен данными между ботом и пользователем осуществляется посредством сообщений. Бот будет отправлять действия Message для передачи информации пользователям, а также будет получать действия Message от пользователей. Некоторые сообщения могут состоять из простого текста, в то время как другие могут содержать более богатое содержимое, такое как произносимый текст, предлагаемые действия, мультимедийные вложения, форматированные карточки и данные по каналу.

В этой статье описаны некоторые распространенные методы обмена сообщениями, которые можно использовать, чтобы улучшить взаимодействие с пользователями.

## <a name="default-message-handler"></a>Обработчик сообщений по умолчанию

Пакет SDK Bot Framework для Node.js содержит обработчик сообщений по умолчанию, встроенный в объект [`session`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html). Этот обработчик сообщений позволяет отправлять и получать текстовые сообщения, передаваемые между ботом и пользователем.

### <a name="send-a-text-message"></a>Отправка текстового сообщения

Отправить текстовое сообщение с помощью обработчика сообщений по умолчанию не сложно, просто вызовите метод `session.send` и передайте в него **строку**.

В этом примере показано, как отправить текстовое сообщение, чтобы приветствовать пользователя.
```javascript
session.send("Good morning.");
```

В этом примере показано, как отправить текстовое сообщение с помощью шаблона строки JavaScript.
```javascript
var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
session.send(msg); //msg: "You ordered: Potato Salad for a total of $5.99."
```

### <a name="receive-a-text-message"></a>Получение текстового сообщения

Когда пользователь отправляет боту сообщение, бот получает его с помощью свойства `session.message`.

В этом примере показано, как получить доступ к сообщению пользователя.
```javascript
var userMessage = session.message.text;
```

## <a name="customizing-a-message"></a>Настройка сообщения

Чтобы обеспечить больший контроль над форматированием текста сообщений, можно создать пользовательский объект [`message`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html) и задать необходимые свойства перед его отправкой пользователю.

В этом примере показано, как создать пользовательский объект `message` и задать свойства `text`, `textFormat` и `textLocale`.

```javascript
var customMessage = new builder.Message(session)
    .text("Hello!")
    .textFormat("plain")
    .textLocale("en-us");
session.send(customMessage);
```

В случаях, когда объект `session` не используется, можно применить метод `bot.send`, чтобы отправить форматированное сообщение пользователю.

Свойство `textFormat` сообщения может использоваться для указания формата текста. Свойству `textFormat` может быть присвоено значение **plain**, **markdown** или **xml**. Значение `textFormat` по умолчанию — **markdown**. 

## <a name="message-property"></a>Свойство Message

Объект [`Message`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html) содержит внутреннее свойство **data**, которое используется для управления отправляемым сообщением. Другие свойства можно задать, используя различные предоставляемые этим объектом методы. 

## <a name="message-methods"></a>Методы Message

Для установки и получения свойств сообщения используются методы объекта. В следующей таблице приведен список методов, которые можно вызывать для установки и получения различных свойств **Message**.

| Метод | ОПИСАНИЕ |
| ---- | ---- | 
| [`addAttachment(attachment:AttachmentType)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addattachment) | Добавляет вложение в сообщение.|
| [`addEntity(obj:Object)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#addentity) | Добавляет сущность в сообщение. |
| [`address(adr:IAddress)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#address) | Адрес для отправки сообщения. Чтобы отправить упреждающее сообщение пользователю, сохраните адрес сообщения в контейнере userData. |
| [`attachmentLayout(style:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachmentlayout) | Указание по размещению нескольких вложений для клиента. По умолчанию используется значение "list". |
| [`attachments(list:AttachmentType)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#attachments) | Список карточек или изображений для отправки пользователю. |
| [`compose(prompts:string[], ...args:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#compose) | Выполняет составление сложного и случайного ответа пользователю. |
| [`entities(list:Object[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#entities) | Структурированные объекты, передаваемые боту или пользователю. |
| [`inputHint(hint:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#inputhint) | Указание, отправляемое пользователю и сообщающее о том, требует ли бот дальнейшего ввода данных. Это значение будет автоматически заполнено с помощью встроенных запросов для исходящих сообщений. |
| [`localTimeStamp((optional)time:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#localtimestamp) | Местное время отправки сообщения клиентом или ботом (устанавливается клиентом), например "2016-09-23T13:07:49.4714686-07:00". |
| [`originalEvent(event:any)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#originalevent) | Сообщение в исходном или собственном формате канала для входящих сообщений. |
| [`sourceEvent(map:ISourceEventMap)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#sourceevent) | Может использоваться для исходящих сообщений для передачи исходных данных события, например пользовательских вложений. |
| [`speak(ssml:TextType, ...args:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#speak) | Задает поле speak сообщения как поле *SSML*. Его содержимое будет произнесено для пользователя на поддерживаемых устройствах. |
| [`suggestedActions(suggestions:ISuggestedActions `&#124;` IIsSuggestedActions)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#suggestedactions) | Необязательные предлагаемые действия для отправки пользователю. Предлагаемые действия отображается только в каналах, которые их поддерживают. |
| [`summary(text:TextType, ...argus:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#summary) | Текст, который отображается как альтернативный текст и краткое описание содержимого сообщения (например: "List of recent conversations" (Список последних бесед)). |
| [`text(text:TextType, ...args:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#text) | Задает текст сообщения. |
| [`textFormat(style:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textformat) | Задает формат текста. Формат по умолчанию — **markdown**. |
| [`textLocale(locale:string)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#textlocale) | Задает целевой язык сообщения. |
| [`toMessage()`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#tomessage) | Получает JSON сообщения. |
| [`composePrompt(session:Session, prompts:string[], args?:any[])`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#composeprompt-1) | Объединяет массив запросов в единый локализованный запрос и затем, при необходимости, заполняет слоты шаблона запросов переданными аргументами. |
| [`randomPrompt(prompts:TextType)`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message.html#randomprompt) | Получает случайный запрос из переданного массива **prompts*. |

## <a name="next-step"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Отправка и получение вложений](bot-builder-nodejs-send-receive-attachments.md)


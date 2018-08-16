---
title: Реализации функций, связанных с каналами | Документация Майкрософт
description: Сведения о том, как реализовать функции, связанные с каналами, с помощью API Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 3cb6f552bee4857d3562e637b2a5728b30ac48a5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304955"
---
# <a name="implement-channel-specific-functionality"></a>Реализация функций, связанных с каналами

Некоторые каналы предоставляют функции, которые невозможно реализовать, используя только [текст сообщений и вложения](bot-framework-rest-connector-create-messages.md). Чтобы реализовать функции, связанные с каналами, вы можете передать в канал собственные метаданные через свойство `channelData` объекта [Activity][Activity]. Например, с помощью свойства `channelData` бот может передать в Telegram команду отправки наклейки или потребовать, чтобы Office 365 отправил сообщение электронной почты.

В этой статье описано, как реализовать функции, связанные с каналами, на основе свойства `channelData` в действии сообщения.

| Канал | Функции |
|----|----|
| Email | Отправка и получение сообщений электронной почты, которые содержат текст, тему и метаданные о важности |
| Slack | Отправка сообщений Slack с полным контролем |
| Facebook | Отправка уведомлений Facebook из кода приложения |
| Telegram | Выполните действия, реализованные в Telegram, такие как предоставление в совместный доступ голосового напоминания или наклейки |
| Kik | Отправка и получение сообщений Kik | 

> [!NOTE]
> Значение свойства `channelData` объекта [Activity][Activity] — это объект JSON. Структура этого объекта JSON будет разной в зависимости от канала и реализованных функций, как описано ниже. 

## <a name="create-a-custom-email-message"></a>Создание пользовательского сообщения электронной почты

Чтобы создать сообщение электронной почты, присвойте свойству `channelData` объекта [Action][Activity] объект JSON, содержащий следующие свойства:

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

В этом фрагменте кода демонстрируется свойство `channelData` для сообщения электронной почты.

```json
"channelData":
{
    "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
    "subject": "Super awesome message subject",
    "importance": "high",
    "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
}
```

## <a name="create-a-full-fidelity-slack-message"></a>Создание сообщение Slack с полным контролем

Чтобы создать сообщение Slack с полным контролем, присвойте свойству `channelData` объекта [действия][Activity] объект JSON, который определяет <a href="https://api.slack.com/docs/messages" target="_blank">сообщения</a>, <a href="https://api.slack.com/docs/message-attachments" target="_blank">вложения</a> и (или) <a href="https://api.slack.com/docs/message-buttons" target="_blank">кнопки</a> Slack. 

> [!NOTE]
> Для включения поддержки кнопок в сообщениях Slack необходимо включить **интерактивные сообщения** при [подключении бота](../bot-service-manage-channels.md) к каналу Slack.

В этом фрагменте кода демонстрируется свойство `channelData` для пользовательского сообщения Slack.

```json
"channelData": {
   "text": "Now back in stock! :tada:",
   "attachments": [
        {
            "title": "The Further Adventures of Slackbot",
            "author_name": "Stanford S. Strickland",
            "author_icon": "https://api.slack.com/img/api/homepage_custom_integrations-2x.png",
            "image_url": "http://i.imgur.com/OJkaVOI.jpg?1"
        },
        {
            "fields": [
                {
                    "title": "Volume",
                    "value": "1",
                    "short": true
                },
                {
                    "title": "Issue",
                    "value": "3",
                    "short": true
                }
            ]
        },
        {
            "title": "Synopsis",
            "text": "After @episod pushed exciting changes to a devious new branch back in Issue 1, Slackbot notifies @don about an unexpected deploy..."
        },
        {
            "fallback": "Would you recommend it to customers?",
            "title": "Would you recommend it to customers?",
            "callback_id": "comic_1234_xyz",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "recommend",
                    "text": "Recommend",
                    "type": "button",
                    "value": "recommend"
                },
                {
                    "name": "no",
                    "text": "No",
                    "type": "button",
                    "value": "bad"
                }
            ]
        }
    ]
}
```

Когда пользователь нажмет кнопку в сообщении Slack, бот получит ответное сообщение, в котором свойство `channelData` содержит объект JSON `payload`. Этот объект `payload` определяет содержимое исходного сообщения, нажатую кнопку и идентификатор пользователя, который нажал эту кнопку. 

В этом фрагменте кода показан пример свойства `channelData` в сообщении, которое бот получает при нажатии кнопки в сообщении Slack.

```json
"channelData": {
    "payload": {
        "actions": [
            {
                "name": "recommend",
                "value": "yes"
            }
        ],
        . . .
        "original_message": "{…}",
        "response_url": "https://hooks.slack.com/actions/..."
    }
}
```

Бот может ответить на это сообщение [обычным способом](bot-framework-rest-connector-send-and-receive-messages.md#create-reply) или отправить ответ напрямую на конечную точку, которая определена свойством `response_url` объекта `payload`. Сведения том, как и в каких случаях следует отправлять ответ для `response_url`, см. в документации по <a href="https://api.slack.com/docs/message-buttons" target="_blank">кнопкам Slack</a>. 

## <a name="create-a-facebook-notification"></a>Создание оповещения Facebook

Чтобы создать оповещение Facebook, присвойте свойству `channelData` объекта [Action][Activity] объект JSON, содержащий следующие свойства: 

| Свойство | ОПИСАНИЕ |
|----|----|
| notification_type | Тип уведомления (например, **REGULAR**, **SILENT_PUSH**, **NO_PUSH**).
| attachment | Вложение, которое содержит изображение, видео или другой тип мультимедиа, уведомление или другое шаблонное вложение, например квитанция. |

> [!NOTE]
> Дополнительные сведения о формате и содержании свойств `notification_type` и `attachment` см. в документации по <a href="https://developers.facebook.com/docs/messenger-platform/send-api-reference#guidelines" target="_blank">API Facebook</a>. 

В этом фрагменте кода демонстрируется свойство `channelData` для вложения Facebook.

```json
"channelData": {
    "notification_type": "NO_PUSH",
    "attachment": {
        "type": "template"
        "payload": {
            "template_type": "receipt",
            . . .
        }
    }
}
```

## <a name="create-a-telegram-message"></a>Создание сообщения Telegram

Чтобы создать сообщение, которое реализует специальные действия Telegram, например предоставление в совместный доступ голосового напоминания или наклейки, присвойте свойству `channelData` объекта [Activity][Activity] объект JSON, который определяет следующие свойства: 

| Свойство | ОПИСАНИЕ |
|----|----|
| метод | Вызываемый метод API Telegram Bot. |
| parameters | Параметры указанного метода. |

Поддерживаются следующие методы Telegram: 

- answerInlineQuery
- editMessageCaption
- editMessageReplyMarkup
- editMessageText
- forwardMessage
- kickChatMember
- sendAudio
- sendChatAction
- sendContact
- sendDocument
- sendLocation
- sendMessage
- sendPhoto
- sendSticker
- sendVenue
- sendVideo
- sendVoice
- unbanChateMember

Дополнительные сведения об этих методах Telegram и их параметрах см. в документации по <a href="https://core.telegram.org/bots/api#available-methods" target="_blank">API Telegram Bot</a>.

> [!NOTE]
> <ul><li>Параметр <code>chat_id</code> применяется во всех методах Telegram. Если вы не укажете <code>chat_id</code> в числе параметров, платформа автоматически создаст идентификатор.</li>
> <li>Чтобы не передавать содержимое файла прямо в запросе, включите ссылку на файл через URL-адрес и тип носителя, как показано в следующем примере.</li>
> <li>В каждом сообщении, которое бот получает из канала Telegram, свойство <code>channelData</code> содержит отправленное ранее сообщение.</li></ul>

В этом фрагменте кода показан пример свойства `channelData`, которое определяет один метод Telegram.

```json
"channelData": {
    "method": "sendSticker",
    "parameters": {
        "sticker": {
            "url": "https://domain.com/path/gif",
            "mediaType": "image/gif",
        }
    }
}
```

В этом фрагменте кода показан пример свойства `channelData`, которое определяет массив методов Telegram.

```json
"channelData": [
    {
        "method": "sendSticker",
        "parameters": {
            "sticker": {
                "url": "https://domain.com/path/gif",
                "mediaType": "image/gif",
            }
        }
    },
    {
        "method": "sendMessage",
        "parameters": {
            "text": "<b>This message is HTML formatted.</b>",
            "parse_mode": "HTML"
        }
    }
]
```

## <a name="create-a-native-kik-message"></a>Создание собственного сообщения Kik

Чтобы создать собственное сообщение Kik, присвойте свойству `channelData` объекта [Action][Activity] объект JSON, содержащий следующие свойства: 

| Свойство | ОПИСАНИЕ |
|----|----|
| отправляемых из облака на устройство | Массив сообщений Kik. См. дополнительные сведения о <a href="https://dev.kik.com/#/docs/messaging#message-formats" target="_blank">формате сообщений Kik</a>. |

В этом фрагменте кода демонстрируется свойство `channelData` для собственного сообщения Kik.

```json
"channelData": {
    "messages": [
        {
            "chatId": "c6dd8165…",
            "type": "link",
            "to": "kikhandle",
            "title": "My Webpage",
            "text": "Some text to display",
            "url": "http://botframework.com",
            "picUrl": "http://lorempixel.com/400/200/",
            "attribution": {
                "name": "My App",
                "iconUrl": "http://lorempixel.com/50/50/"
            },
            "noForward": true,
            "kikJsData": {
                    "key": "value"
                }
        }
    ]
}
```

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Общие сведения о действиях](bot-framework-rest-connector-activities.md)
- [Создание сообщений](bot-framework-rest-connector-create-messages.md)
- [Отправка и получение сообщений](bot-framework-rest-connector-send-and-receive-messages.md)
- [Предварительный просмотр компонентов с помощью Channel Inspector](../bot-service-channel-inspector.md)

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

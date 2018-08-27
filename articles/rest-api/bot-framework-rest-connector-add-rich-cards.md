---
title: Добавление форматированных карточек как вложений в сообщения | Документация Майкрософт
description: Сведения о добавлении форматированных карточек в сообщения с помощью службы Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 04f70777003ef5298de264f5ee8685b3a5005395
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305826"
---
# <a name="add-rich-card-attachments-to-messages"></a>Добавление вложений в виде форматированных карточек в сообщения
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Боты и каналы обычно обмениваются текстовыми строками, но некоторые каналы также поддерживают обмен вложениями, что позволяет боту отправлять пользователям сообщения с более широким набором возможностей. Например, бот может отправлять форматированные карточки и вложения мультимедиа (такие как изображения, видео, аудио и файлы). В этой статье описывается добавление форматированных карточек как вложений в сообщения с помощью службы Bot Connector.

> [!NOTE]
> Сведения о добавлении вложений мультимедиа в сообщения см. в статье [Добавление мультимедийных вложений в сообщения](bot-framework-rest-connector-add-media-attachments.md).

## <a id="types-of-cards"></a> Типы форматированных карточек

Форматированная карточка содержит название, описание, ссылку и изображения. Сообщение может содержать несколько форматированных карточек, которые отображаются в виде списка или карусели.
Сейчас Bot Framework поддерживает восемь типов форматированных карточек. 

| Тип карточки | ОПИСАНИЕ |
|----|----|
| <a href="/adaptive-cards/get-started/bots">AdaptiveCard</a> | Настраиваемая карточка, которая содержит любое сочетание текста, речи, изображений, кнопок и полей ввода. См. раздел [о поддержке каналов](/adaptive-cards/get-started/bots#channel-status).  |
| [AnimationCard][animationCard] | Карточка, которая воспроизводит файлы GIF с анимацией или короткие видео. |
| [AudioCard][audioCard] | Карточка, которая воспроизводит звуковой файл. |
| [HeroCard][heroCard] | Карточка, которая обычно содержит одно большое изображение, одну или несколько кнопок и текст. |
| [ThumbnailCard][thumbnailCard] | Карточка, которая обычно содержит один эскиз, одну или несколько кнопок и текст. |
| [ReceiptCard][receiptCard] | Карточка, с помощью которой бот выдает пользователю квитанцию. Обычно она содержит список элементов, включаемых в квитанцию, налог и общую сумму, а также другую информацию. |
| [SignInCard][signinCard] | Карточка, позволяющая боту запрашивать вход пользователя. Обычно она содержит текст и одну или несколько кнопок, которые пользователь нажимает для инициации процесса входа. |
| [VideoCard][videoCard] | Карточка, которая воспроизводит видео. |

> [!TIP]
> Сведения о том, как определить тип форматированных карточек, поддерживаемых каналом, и способ их обработки каналом, см. в разделе [Channel Inspector][ChannelInspector]. Сведения об ограничениях содержимого карточек (например, максимальное количество кнопок или максимальная длина названия) см. в документации канала.

## <a name="process-events-within-rich-cards"></a>Обработка событий в форматированных карточках

Для обработки событий в форматированных карточках используйте объекты [CardAction][CardAction], чтобы указать, что должно происходить, когда пользователь нажимает кнопку или щелкает сегмент карточки. Каждый объект [CardAction][CardAction] содержит следующие свойства.

| Свойство | type | ОПИСАНИЕ | 
|----|----|----|
| Тип | строка | тип действия (одно из значений, указанных в таблице ниже) |
| title | строка | название кнопки |
| изображение | строка | URL-адрес изображения для кнопки |
| value | строка | значение, необходимое для выполнения указанного типа действия |

> [!NOTE]
> Для создания кнопок в адаптивных карточках используются не объекты `CardAction`, а схема, которая определяется <a href="http://adaptivecards.io" target="_blank">адаптивными карточками</a>. О том, как добавить кнопку в адаптивную карточку, см. в разделе [Добавление адаптивной карточки в сообщение](#adaptive-card).

В этой таблице перечислены допустимые значения для свойства `type` объекта [CardAction][CardAction] и описано ожидаемое содержимое свойства `value` для каждого типа.

| Тип | value | 
|----|----|
| openUrl | URL-адрес, который будет открыт во встроенном браузере |
| imBack | Текст сообщения, которое будет отправлено боту (от пользователя, который нажал кнопку или щелкнул карточку). Это сообщение (от пользователя к боту) увидят все участники общения через клиентское приложение, в котором ведется общение. |
| postBack | Текст сообщения, которое будет отправлено боту (от пользователя, который нажал кнопку или щелкнул карточку). Некоторые клиентские приложения могут отображать этот текст на канале сообщений, где он будет виден всем участникам общения. |
| вызывает | Место назначения телефонного звонка в следующем виде: **tel:123123123123** |
| playAudio | URL-адрес аудио для воспроизведения |
| playVideo | URL-адрес видео для воспроизведения |
| showImage | URL-адрес изображения для отображения |
| downloadFile | URL-адрес файла для загрузки |
| signin | URL-адрес потока OAuth для инициализации |

## <a name="add-a-hero-card-to-a-message"></a>Добавление имиджевой карточки в сообщение

Чтобы добавить вложение в виде форматированной карточки в сообщение, создайте объект, который соответствует [типу карточки](#types-of-cards), которую необходимо добавить. Затем создайте объект [Вложение][Attachment], задайте свойство `contentType` для типа мультимедиа карты и свойство `content` для объекта, который был создан для представления карты. Укажите объект [Вложение][Attachment] в пределах массива сообщения `attachments`.

> [!TIP]
> Сообщения, содержащие вложения в виде форматированных карточек, обычно не указывают `text`.

На некоторых каналах можно добавлять несколько форматированных карточек в массив `attachments` сообщения. Это может быть полезно в сценариях, где необходимо предоставить пользователю несколько параметров. Например, если бот позволяет пользователям забронировать номера в гостинице, то он может предоставить список форматированных карточек, в которых будут показаны типы доступных номеров. Каждая карточка может содержать изображение и список удобств, соответствующих типу комнаты, а пользователь может выбрать тип комнаты, нажав на кнопку или щелкнув карточку.

> [!TIP]
> Чтобы отобразить несколько форматированных карточек в виде списка, задайте свойству `attachmentLayout` объекта [Действие][Activity] значение list. Чтобы отобразить несколько форматированных карточек в виде карусели, задайте свойству `attachmentLayout` объекта [Действие][Activity] значение carousel. Если канал не поддерживает формат карусели, форматированные карточки будут отображены в виде списка, даже если свойству `attachmentLayout` задано значение carousel.

Следующий пример демонстрирует запрос, который отправляет сообщение, содержащее одно вложение имиджевой карточки. В этом примере запроса `https://smba.trafficmanager.net/apis` представляет базовый URI. Базовый URI для запросов, отправляемых вашим ботом, может отличаться. Дополнительные сведения о настройке базового URI см. в статье [Справочник по API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.hero",
            "content": {
                "title": "title goes here",
                "subtitle": "subtitle goes here",
                "text": "descriptive text goes here",
                "images": [
                    {
                        "url": "http://aka.ms/Fo983c",
                        "alt": "picture of a duck",
                        "tap": {
                            "type": "playAudio",
                            "value": "url to an audio track of a duck call goes here"
                        }
                    }
                ],
                "buttons": [
                    {
                        "type": "playAudio",
                        "title": "Duck Call",
                        "value": "url to an audio track of a duck call goes here"
                    },
                    {
                        "type": "openUrl",
                        "title": "Watch Video",
                        "image": "http://aka.ms/Fo983c",
                        "value": "url goes here of the duck in flight"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

## <a id="adaptive-card"></a> Добавление адаптивной карточки в сообщение

Адаптивная карточка может содержать любое сочетание текста, речи, изображений, кнопок и полей ввода. Адаптивные карточки создаются в формате JSON, указанном на веб-сайте <a href="http://adaptivecards.io" target="_blank">адаптивных карточек</a>, что позволяет получить полный контроль над содержимым и форматом карточки. 

Ознакомьтесь со сведениями на веб-сайте <a href="http://adaptivecards.io" target="_blank">адаптивных карточек</a>, чтобы получить представление о схеме адаптивных карточек, изучить элементы адаптивных карточек и просмотреть примеры JSON, которые можно использовать для создания карточек различного состава и уровня сложности. Кроме того, воспользовавшись интерактивным визуализатором, можно разрабатывать полезные нагрузки для адаптивных карточек и просматривать выходные данные карточек.

Следующий пример демонстрирует запрос, который отправляет сообщение, содержащее одну адаптивную карточку для напоминания календаря. В этом примере запроса `https://smba.trafficmanager.net/apis` представляет базовый URI. Базовый URI для запросов, отправляемых вашим ботом, может отличаться. Дополнительные сведения о настройке базового URI см. в статье [Справочник по API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "attachments": [
        {
            "contentType": "application/vnd.microsoft.card.adaptive",
            "content": {
                "type": "AdaptiveCard",
                "body": [
                    {
                        "type": "TextBlock",
                        "text": "Adaptive Card design session",
                        "size": "large",
                        "weight": "bolder"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Conf Room 112/3377 (10)"
                    },
                    {
                        "type": "TextBlock",
                        "text": "12:30 PM - 1:30 PM"
                    },
                    {
                        "type": "TextBlock",
                        "text": "Snooze for"
                    },
                    {
                        "type": "Input.ChoiceSet",
                        "id": "snooze",
                        "style": "compact",
                        "choices": [
                            {
                                "title": "5 minutes",
                                "value": "5",
                                "isSelected": true
                            },
                            {
                                "title": "15 minutes",
                                "value": "15"
                            },
                            {
                                "title": "30 minutes",
                                "value": "30"
                            }
                        ]
                    }
                ],
                "actions": [
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Snooze"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "I'll be late"
                    },
                    {
                        "type": "Action.Http",
                        "method": "POST",
                        "url": "http://foo.com",
                        "title": "Dismiss"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

Итоговая карточка содержит три блока текста, поле ввода (список значений) и три кнопки.

![Адаптивная карточка с напоминанием календаря](../media/adaptive-card-reminder.png)


## <a name="additional-resources"></a>Дополнительные ресурсы

- [Создание сообщений](bot-framework-rest-connector-create-messages.md)
- [Отправка и получение сообщений](bot-framework-rest-connector-send-and-receive-messages.md)
- [Добавление мультимедийных вложений в сообщения](bot-framework-rest-connector-add-media-attachments.md)
- [Channel Inspector][ChannelInspector]
- <a href="http://adaptivecards.io" target="_blank">Adaptive Cards</a> (Адаптивные карточки)

[ChannelInspector]: ../bot-service-channel-inspector.md

[animationCard]: bot-framework-rest-connector-api-reference.md#animationcard-object
[audioCard]: bot-framework-rest-connector-api-reference.md#audiocard-object
[heroCard]: bot-framework-rest-connector-api-reference.md#herocard-object
[thumbnailCard]: bot-framework-rest-connector-api-reference.md#thumbnailcard-object
[receiptCard]: bot-framework-rest-connector-api-reference.md#receiptcard-object
[signinCard]: bot-framework-rest-connector-api-reference.md#signincard-object
[videoCard]: bot-framework-rest-connector-api-reference.md#videocard-object

[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[Attachment]: bot-framework-rest-connector-api-reference.md#attachment-object
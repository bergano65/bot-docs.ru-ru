---
title: Добавление вложений в виде форматированных карточек в сообщения в службе Bot
description: Сведения о добавлении форматированных карточек в сообщения с помощью службы Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: e51d4bcc7059e1130932ca6a8b956dd3b097ef8c
ms.sourcegitcommit: 4e1af50bd46debfdf9dcbab9a5d1b1633b541e27
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/25/2020
ms.locfileid: "76752906"
---
# <a name="add-rich-card-attachments-to-messages"></a>Добавление вложений в виде форматированных карточек в сообщения

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Боты и каналы обычно обмениваются текстовыми строками, но некоторые каналы также поддерживают обмен вложениями, что позволяет боту отправлять пользователям сообщения с более широким набором возможностей. Например, бот может отправлять форматированные карточки и вложения мультимедиа (такие как изображения, видео, аудио и файлы). В этой статье описывается добавление форматированных карточек как вложений в сообщения с помощью службы Bot Connector.

> [!NOTE]
> Сведения о добавлении вложений мультимедиа в сообщения см. в статье [Добавление мультимедийных вложений в сообщения](bot-framework-rest-connector-add-media-attachments.md).

## <a name="types-of-rich-cards"></a>Типы функциональных карточек

Форматированная карточка содержит название, описание, ссылку и изображения.
Сообщение может содержать несколько форматированных карточек, которые отображаются в виде списка или карусели.
Сейчас Bot Framework поддерживает восемь типов форматированных карточек.

| Тип карточки | Описание |
|----|----|
| [AdaptiveCard](/adaptive-cards/get-started/bots) | Настраиваемая карточка, которая может содержать любое сочетание текста, речи, изображений, кнопок и полей для ввода. См. описание [поддержки для каждого канала](/adaptive-cards/get-started/bots#channel-status). |
| [AnimationCard][] | Карточка, которая может воспроизводить GIF-файлы с анимацией или короткие видеоролики. |
| [AudioCard][] | Карточка, которая может воспроизводить звуковой файл. |
| [HeroCard][]; | Карточка, которая обычно содержит одно большое изображение, одну или несколько кнопок и текст. |
| [ThumbnailCard][]. | Карточка, которая обычно содержит один эскиз, одну или несколько кнопок и текст. |
| [ReceiptCard][]; | Карточка, с помощью которой бот выдает квитанцию пользователю. Обычно она содержит список элементов, включаемых в квитанцию, налог, а также общую информацию и другой текст. |
| [SignInCard][] | Карточка, в которой бот запрашивает вход пользователя. Обычно она содержит текст и одну или несколько кнопок, которые можно нажать, чтобы начать процесс входа. |
| [VideoCard][] | Карточка, которая может воспроизводить видео. |

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="process-events-within-rich-cards"></a>Обработка событий в форматированных карточках

Для обработки событий в форматированных карточках используйте объекты [CardAction][], чтобы указать, что должно происходить, когда пользователь нажимает кнопку или касается сегмента карточки. Каждый объект `CardAction` содержит следующие свойства.

| Свойство | Тип | Описание |
|----|----|----|
| channelData | строка | Относящиеся к каналу данные, связанные с этим действием. |
| displayText | строка | Текст, отображаемый в канале чата при нажатии кнопки. |
| text | строка | Текст для действия. |
| type | строка | тип действия (одно из значений, указанных в таблице ниже) |
| title | строка | название кнопки |
| Изображение | строка | URL-адрес изображения для кнопки |
| value | строка | значение, необходимое для выполнения указанного типа действия |

> [!NOTE]
> Для создания кнопок в адаптивных карточках используются не объекты `CardAction`, а схема, которая определяется адаптивными карточками.
> О том, как добавить кнопку в адаптивную карточку, см. в разделе [Добавление адаптивной карточки в сообщение](#add-an-adaptive-card-to-a-message).

В этой таблице перечислены допустимые значения для свойства `type` объекта `CardAction` и описано ожидаемое содержимое свойства `value` для каждого типа.

| type | value |
|----|----|
| openUrl | URL-адрес, который будет открыт во встроенном браузере |
| imBack | Текст сообщения для отправки боту (от пользователя, который нажал кнопку или коснулся карты). Это сообщение (от пользователя к боту) увидят все участники общения через клиентское приложение, в котором ведется общение. |
| postBack | Текст сообщения для отправки боту (от пользователя, который нажал кнопку или коснулся карты). Некоторые клиентские приложения могут отображать этот текст на канале сообщений, где он будет виден всем участникам общения. |
| вызывает | Место назначения телефонного звонка в следующем виде: **tel:123123123123** |
| playAudio | URL-адрес аудио для воспроизведения |
| playVideo | URL-адрес видео для воспроизведения |
| showImage | URL-адрес изображения для отображения |
| downloadFile | URL-адрес файла для загрузки |
| signin | URL-адрес потока OAuth для инициализации |

## <a name="add-a-hero-card-to-a-message"></a>Добавление имиджевой карточки в сообщение

Чтобы добавить вложение в виде форматированной карточки в сообщение, создайте объект, который соответствует [типу карточки](#types-of-rich-cards), которую необходимо добавить. Затем создайте объект [Вложение][], задайте свойству `contentType` тип мультимедиа карты, а свойству `content` — объект, который был создан для представления карты. Укажите объект `Attachment` в массиве `attachments` сообщения.

> [!TIP]
> Сообщения, содержащие вложения в виде форматированных карточек, обычно не указывают `text`.

На некоторых каналах можно добавлять несколько форматированных карточек в массив `attachments` сообщения. Это может быть полезно в сценариях, где необходимо предоставить пользователю несколько параметров. Например, если бот позволяет пользователям забронировать номера в гостинице, то он может предоставить список форматированных карточек, в которых будут показаны типы доступных номеров. Каждая карточка может содержать изображение и список удобств, соответствующих типу комнаты, а пользователь может выбрать тип комнаты, нажав на кнопку или щелкнув карточку.

> [!TIP]
> Чтобы отобразить несколько форматированных карточек в виде списка, задайте свойству `attachmentLayout` объекта [Действие][] значение list.
> Чтобы отобразить несколько форматированных карточек в виде карусели, задайте свойству `attachmentLayout` объекта `Activity` значение carousel.
> Если канал не поддерживает формат карусели, форматированные карточки будут отображены в виде списка, даже если свойству `attachmentLayout` задано значение carousel.

Следующий пример демонстрирует запрос, который отправляет сообщение, содержащее одно вложение имиджевой карточки. В этом примере запрос `https://smba.trafficmanager.net/apis` представляет базовый URI. Базовый URI для запросов, отправляемых вашим ботом, может отличаться. Дополнительные сведения о настройке базового URI см. в статье [Справочник по API](bot-framework-rest-connector-api-reference.md#base-uri).

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
                        "url": "https://aka.ms/DuckOnARock",
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
                        "image": "https://aka.ms/DuckOnARock",
                        "value": "url goes here of the duck in flight"
                    }
                ]
            }
        }
    ],
    "replyToId": "5d5cdc723"
}
```

## <a name="add-an-adaptive-card-to-a-message"></a>Добавление адаптивной карточки в сообщение

Адаптивная карточка может содержать любое сочетание текста, речи, изображений, кнопок и полей для ввода.
Адаптивные карточки создаются в формате JSON (см. [здесь](http://adaptivecards.io)), что позволяет получить больший контроль над содержимым и форматом карточек.

Ознакомьтесь со сведениями на веб-сайте [адаптивных карточек](http://adaptivecards.io), чтобы получить представление о схеме адаптивных карточек, изучить элементы адаптивных карточек и просмотреть примеры JSON, которые можно использовать для создания карточек различного состава и уровня сложности. А воспользовавшись интерактивным визуализатором, вы сможете разрабатывать соответствующие полезные нагрузки и просматривать выходные данные карточки. Ниже приведен пример одной адаптивной карточки для назначения задания.

```json
{
  "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
  "type": "AdaptiveCard",
  "version": "1.0",
  "body": [
    {
      "type": "Container",
      "items": [
        {
          "type": "TextBlock",
          "text": "Publish Adaptive Card schema",
          "weight": "bolder",
          "size": "medium"
        },
        {
          "type": "ColumnSet",
          "columns": [
            {
              "type": "Column",
              "width": "auto",
              "items": [
                {
                  "type": "Image",
                  "url": "https://pbs.twimg.com/profile_images/3647943215/d7f12830b3c17a5a9e4afcc370e3a37e_400x400.jpeg",
                  "size": "small",
                  "style": "person"
                }
              ]
            },
            {
              "type": "Column",
              "width": "stretch",
              "items": [
                {
                  "type": "TextBlock",
                  "text": "Matt Hidinger",
                  "weight": "bolder",
                  "wrap": true
                },
                {
                  "type": "TextBlock",
                  "spacing": "none",
                  "text": "Created {{DATE(2017-02-14T06:08:39Z, SHORT)}}",
                  "isSubtle": true,
                  "wrap": true
                }
              ]
            }
          ]
        }
      ]
    },
    {
      "type": "Container",
      "items": [
        {
          "type": "TextBlock",
          "text": "Now that we have defined the main rules and features of the format, we need to produce a schema and publish it to GitHub. The schema will be the starting point of our reference documentation.",
          "wrap": true
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "Board:",
              "value": "Adaptive Card"
            },
            {
              "title": "List:",
              "value": "Backlog"
            },
            {
              "title": "Assigned to:",
              "value": "Matt Hidinger"
            },
            {
              "title": "Due date:",
              "value": "Not set"
            }
          ]
        }
      ]
    }
  ],
  "actions": [
    {
      "type": "Action.ShowCard",
      "title": "Comment",
      "card": {
        "type": "AdaptiveCard",
        "body": [
          {
            "type": "Input.Text",
            "id": "comment",
            "isMultiline": true,
            "placeholder": "Enter your comment"
          }
        ],
        "actions": [
          {
            "type": "Action.Submit",
            "title": "OK"
          }
        ]
      }
    },
    {
      "type": "Action.OpenUrl",
      "title": "View",
      "url": "https://adaptivecards.io"
    }
  ]
}

```

Результирующая карточка содержит заголовок, сведения о пользователе, создавшем карточку (имя и аватар), время создания карточки, описание назначения задания и сведения, связанные с назначением. Кроме того, доступны две кнопки, позволяющие просмотреть назначение задания или добавить комментарий.

![Адаптивная карточка с напоминанием календаря](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Создание сообщений](bot-framework-rest-connector-create-messages.md)
- [Отправка и получение сообщений](bot-framework-rest-connector-send-and-receive-messages.md)
- [Добавление мультимедийных вложений в сообщения](bot-framework-rest-connector-add-media-attachments.md)
- [Принципы использования действий в Bot Framework](https://aka.ms/botSpecs-activitySchema)
- [Channel Inspector][ChannelInspector]

[ChannelInspector]: ../bot-service-channels-reference.md
[Действие]: bot-framework-rest-connector-api-reference.md#activity-object
[Вложение]: bot-framework-rest-connector-api-reference.md#attachment-object
[CardAction]: bot-framework-rest-connector-api-reference.md#cardaction-object
[AnimationCard]: bot-framework-rest-connector-api-reference.md#animationcard-object
[AudioCard]: bot-framework-rest-connector-api-reference.md#audiocard-object
[HeroCard]: bot-framework-rest-connector-api-reference.md#herocard-object;
[ThumbnailCard]: bot-framework-rest-connector-api-reference.md#thumbnailcard-object.
[ReceiptCard]: bot-framework-rest-connector-api-reference.md#receiptcard-object;
[SigninCard]: bot-framework-rest-connector-api-reference.md#signincard-object
[VideoCard]: bot-framework-rest-connector-api-reference.md#videocard-object

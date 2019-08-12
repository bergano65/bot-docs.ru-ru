---
title: Добавление мультимедийных вложений в сообщения | Документы Майкрософт
description: Сведения о добавлении мультимедийных вложений в сообщения с помощью службы соединителя ботов.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/25/2018
ms.openlocfilehash: 03facdff733787e95ca3bc68dfee15d747340453
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757091"
---
# <a name="add-media-attachments-to-messages"></a>Добавление мультимедийных вложений в сообщения
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

Боты и каналы обычно обмениваются текстовыми строками, но некоторые каналы также поддерживают обмен вложениями, что позволяет боту отправлять пользователям сообщения с более широким набором возможностей. Например, бот может отправлять вложения мультимедиа (такие как изображения, видео, звук и файлы) и [форматированные карточки](bot-framework-rest-connector-add-rich-cards.md). В этой статье описывается добавление вложений мультимедиа в сообщения с помощью службы соединителя ботов.

> [!TIP]
> Сведения о том, как определить тип и число вложений, поддерживаемые каналом, и способ их обработки каналом, см. в статье [Channel Inspector][ChannelInspector].

## <a name="add-a-media-attachment"></a>Добавление мультимедийного вложения  

Чтобы добавить мультимедийное вложение в сообщение, создайте объект `Attachment`, задайте свойство `name`, задайте для свойства `contentUrl` URL-адрес файла мультимедиа, а для свойства `contentType` — соответствующий тип мультимедиа (например, **image/png**, **audio/wav**, **video/mp4**). Затем в объекте `Activity`, представляющем сообщение, укажите объект `Attachment` в массиве `attachments`. 

Следующий пример демонстрирует запрос, который отправляет сообщение, содержащее текст и одно вложенное изображение. В этом примере запрос `https://smba.trafficmanager.net/apis` представляет базовый URI. Базовый URI для запросов, отправляемых вашим ботом, может отличаться. Дополнительные сведения о настройке базового URI см. в [справочнике по API](bot-framework-rest-connector-api-reference.md#base-uri).

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "https://aka.ms/DuckOnARock",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

Для каналов, которые поддерживают встроенные двоичные файлы изображений, можно задать свойство `contentUrl` объекта `Attachment` как двоичный файл base64 изображения (например, **data:image/png;base64,iVBORw0KGgo…** ). Канал будет отображать изображение или URL-адрес изображения рядом со строкой текста сообщения.

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
    "text": "Here's a picture of the duck I was telling you about.",
    "attachments": [
        {
            "contentType": "image/png",
            "contentUrl": "data:image/png;base64,iVBORw0KGgo…",
            "name": "duck-on-a-rock.jpg"
        }
    ],
    "replyToId": "5d5cdc723"
}
```

В сообщение можно вложить файл видео или звуковой файл, используя ту же процедуру, которая описана выше для файла изображения. В зависимости от канала аудио- и видеоданные могут воспроизводится внутри сообщения или отображаться в виде ссылки.

> [!NOTE] 
> Бот также может получать сообщения, содержащие мультимедийные вложения.
> Например, сообщение, полученное ботом, может содержать вложение, если канал позволяет пользователю отправлять фотографию для анализа или документ для сохранения.

## <a name="add-an-audiocard-attachment"></a>Добавление вложения AudioCard

Добавление вложения `AudioCard` или `VideoCard` ничем не отличается от добавления вложения мультимедиа. Например, следующий пример кода JSON демонстрирует добавление карточки аудиофайла в мультимедийное вложение.

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
      "contentType": "application/vnd.microsoft.card.audio",
      "content": {
        "title": "Allegro in C Major",
        "subtitle": "Allegro Duet",
        "text": "No Image, No Buttons, Autoloop, Autostart, Sharable",
        "duration": "PT2M55S",
        "media": [
          {
            "url": "https://contoso.com/media/AllegrofromDuetinCMajor.mp3"
          }
        ],
        "shareable": true,
        "autoloop": true,
        "autostart": true,
        "value": {
            // Supplementary parameter for this card
        }
      }
    }],
    "replyToId": "5d5cdc723"
}
```

Как только канал получит это вложение, он начнет воспроизведение звукового файла. Если пользователь взаимодействует со звуковым файлом, например нажимая кнопку **Пауза**, канал отправит обратный вызов боту с примерно следующим запросом JSON:

```json
{
    ...
    "type": "event",
    "name": "media/pause",
    "value": {
        "url": // URL for media
        "cardValue": {
            // Supplementary parameter for this card
        }
    }
}
```

Имя события мультимедиа **media/pause** появится в поле `activity.name`. Список всех имен событий мультимедиа см. в таблице ниже.

| Событие | ОПИСАНИЕ |
| ---- | ---- |
| **media/next** | Клиент перешел к следующему файлу мультимедиа. |
| **media/pause** | Клиент приостановил воспроизведение мультимедиа. |
| **media/play** | Клиент запустил воспроизведение мультимедиа. |
| **media/previous** | Клиент перешел к предыдущему файлу мультимедиа. |
| **media/resume** | Клиент возобновил воспроизведение мультимедиа. |
| **media/stop** | Клиент остановил воспроизведение мультимедиа. |

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Создание сообщений](bot-framework-rest-connector-create-messages.md)
- [Отправка и получение сообщений](bot-framework-rest-connector-send-and-receive-messages.md)
- [Добавление вложений в виде форматированных карточек в сообщения](bot-framework-rest-connector-add-rich-cards.md)
- [Принципы использования действий в Bot Framework](https://aka.ms/botSpecs-activitySchema)
- [Принципы использования карточек в Bot Framework](https://aka.ms/botSpecs-cardSchema)

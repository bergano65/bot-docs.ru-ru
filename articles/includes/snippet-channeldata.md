---
ms.openlocfilehash: 0eab36869a986d15905cdfde5c317613276c844d
ms.sourcegitcommit: e573c586472c5328ce875114308d9d1b73651e62
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/03/2019
ms.locfileid: "70242747"
---
Некоторые каналы предоставляют функции, которые невозможно реализовать, используя только текст сообщений и вложения. Чтобы реализовать функции, связанные с каналами, вы можете передать в канал собственные метаданные через свойство _данных канала_ для объекта действия. Например, используя свойство данных канала, бот может передать в Telegram команду отправки наклейки или потребовать, чтобы из Office 365 было отправлено сообщение электронной почты.

В этой статье объясняется, как реализовать функции, связанные с каналами, на основе свойства данных канала в действии сообщения.

| Канал | Функции |
|----|----|
| Email | Отправка и получение сообщений электронной почты, которые содержат текст, тему и метаданные о важности. |
| Slack | Отправка сообщений Slack с полным контролем. |
| Facebook | Отправка уведомлений Facebook из кода приложения. |
| Telegram | Выполнение действий, реализованных в Telegram, таких как публикация голосового напоминания или наклейки. |
| Kik | Отправка и получение сообщений Kik. |

> [!NOTE]
> Значение свойства данных канала для объекта действия — это объект JSON.
> Поэтому в примерах в этой статье показан ожидаемый формат свойства JSON `channelData` в различных сценариях.
> Чтобы создать объект JSON с помощью .NET, используйте класс `JObject` (.NET).

## <a name="create-a-custom-email-message"></a>Создание пользовательского сообщения электронной почты

Чтобы создать сообщение электронной почты, присвойте свойству данных канала для объекта действия объект JSON, содержащий следующие свойства:

| Свойство | ОПИСАНИЕ |
|----|----|
| bccRecipients | Строка адреса электронной почты, разделенная точками с запятой (;) для добавления к полю "Скрытая копия" сообщения. |
| ccRecipients | Строка адреса электронной почты, разделенная точками с запятой (;) для добавления к полю "Копия" сообщения. |
| htmlBody | Документ HTML, который задает текст сообщения электронной почты. Сведения о поддерживаемых элементах и атрибутах HTML приведены в документации по каналу. |
| importance | Уровень важности сообщения электронной почты. Допустимые значения: **high**, **normal** и **low**. Значение по умолчанию — **normal**. |
| subject | Тема сообщения электронной почты. Сведения о требованиях к полю приведены в документации по каналу. |
| toRecipients | Строка адреса электронной почты, разделенная точками с запятой (;) для добавления к полю "Кому" сообщения. |

> [!NOTE]
> Сообщения, получаемые ботом от пользователей по каналу электронной почты, могут содержать свойство данных канала, которое содержит объект JSON, как описано выше.

В этом фрагменте кода демонстрируется свойство `channelData` для сообщения электронной почты.

```json
"channelData": {
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

## <a name="create-a-full-fidelity-slack-message"></a>Создание сообщение Slack с полным контролем

Чтобы создать сообщение Slack с полным контролем, присвойте свойство данных канала для объекта действия объекту JSON, который определяет <a href="https://api.slack.com/docs/messages" target="_blank">сообщения</a>, <a href="https://api.slack.com/docs/message-attachments" target="_blank">вложения</a> и (или) <a href="https://api.slack.com/docs/message-buttons" target="_blank">кнопки</a> Slack.

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

Когда пользователь нажмет кнопку в сообщении Slack, бот получит ответное сообщение, в котором свойство данных канала содержит объект JSON `payload`.
Этот объект `payload` определяет содержимое исходного сообщения, нажатую кнопку и идентификатор пользователя, который нажал эту кнопку.

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

Бот может ответить на это сообщение обычным способом или отправить ответ напрямую на конечную точку, которая определена свойством `response_url` объекта `payload`.
Сведения том, как и в каких случаях следует отправлять ответ для `response_url`, см. в документации по <a href="https://api.slack.com/docs/message-buttons" target="_blank">кнопкам Slack</a>.

Чтобы создать динамические кнопки, можно использовать следующий код JSON:

```json
{
    "text": "Would you like to play a game ? ",
    "attachments": [
        {
            "text": "Choose a game to play!",
            "fallback": "You are unable to choose a game",
            "callback_id": "wopr_game",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "actions": [
                {
                    "name": "game",
                    "text": "Chess",
                    "type": "button",
                    "value": "chess"
                },
                {
                    "name": "game",
                    "text": "Falken's Maze",
                    "type": "button",
                    "value": "maze"
                },
                {
                    "name": "game",
                    "text": "Thermonuclear War",
                    "style": "danger",
                    "type": "button",
                    "value": "war",
                    "confirm": {
                        "title": "Are you sure?",
                        "text": "Wouldn't you prefer a good game of chess?",
                        "ok_text": "Yes",
                        "dismiss_text": "No"
                    }
                }
            ]
        }
    ]
}
```

Чтобы создать интерактивные меню, используйте такой код JSON:

```json
{
    "text": "Would you like to play a game ? ",
    "response_type": "in_channel",
    "attachments": [
        {
            "text": "Choose a game to play",
            "fallback": "If you could read this message, you'd be choosing something fun to do right now.",
            "color": "#3AA3E3",
            "attachment_type": "default",
            "callback_id": "game_selection",
            "actions": [
                {
                    "name": "games_list",
                    "text": "Pick a game...",
                    "type": "select",
                    "options": [
                        {
                            "text": "Hearts",
                            "value": "menu_id_hearts"
                        },
                        {
                            "text": "Bridge",
                            "value": "menu_id_bridge"
                        },
                        {
                            "text": "Checkers",
                            "value": "menu_id_checkers"
                        },
                        {
                            "text": "Chess",
                            "value": "menu_id_chess"
                        },
                        {
                            "text": "Poker",
                            "value": "menu_id_poker"
                        },
                        {
                            "text": "Falken's Maze",
                            "value": "menu_id_maze"
                        },
                        {
                            "text": "Global Thermonuclear War",
                            "value": "menu_id_war"
                        }
                    ]
                }
            ]
        }
    ]
}
```

## <a name="create-a-facebook-notification"></a>Создание оповещения Facebook

Чтобы создать оповещение Facebook, присвойте свойству данных канала для объекта действия объект JSON, содержащий следующие свойства:

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

Чтобы создать сообщение, которое реализует специальные действия Telegram, например предоставление в совместный доступ голосового напоминания или наклейки, присвойте свойству данных канала для объекта действия объект JSON, который определяет следующие свойства: 

| Свойство | ОПИСАНИЕ |
|----|----|
| method | Вызываемый метод API Telegram Bot. |
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
> <li>В каждом сообщении, которое бот получает из канала Telegram, свойство <code>ChannelData</code> содержит отправленное ранее сообщение.</li></ul>

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

Чтобы создать собственное сообщение Kik, присвойте свойству данных канала для объекта действия объект JSON, содержащий следующие свойства:

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

## <a name="create-a-line-message"></a>Создание сообщения LINE

Чтобы создать сообщение, которое реализует особые типы сообщений (например, наклейки, шаблоны или связанные с сообщениями LINE действия, включая открытие камеры на телефоне), настройте для свойства данных канала объекта действия объект JSON, который определяет типы сообщений LINE и типы действий. 

| Свойство | ОПИСАНИЕ |
|----|----|
| Тип | Имя типа сообщения или действия LINE |

Поддерживаются следующие типы сообщений LINE:
* наклейка;
* гиперкарта; 
* шаблон (кнопка, подтверждение, карусель); 
* общая панель. 

Эти действия LINE можно указать в поле действия объекта JSON типа сообщения: 
* обратная передача; 
* Сообщение 
* URI 
* средство выбора даты и времени; 
* Camera 
* галерея камеры; 
* Location 

Дополнительные сведения об этих методах LINE и их параметрах см. в документации по [API LINE Bot](https://developers.line.biz/en/docs/messaging-api/). 

В этом фрагменте кода показан пример свойства `channelData`, которое определяет тип сообщения канала `ButtonTemplate` и три типа действий: камера, галерея камеры и средство выбора даты и времени. 

```json
"channelData": { 
    "type": "ButtonsTemplate", 
    "altText": "This is a buttons template", 
    "template": { 
        "type": "buttons", 
        "thumbnailImageUrl": "https://example.com/bot/images/image.jpg", 
        "imageAspectRatio": "rectangle", 
        "imageSize": "cover", 
        "imageBackgroundColor": "#FFFFFF", 
        "title": "Menu", 
        "text": "Please select", 
        "defaultAction": { 
            "type": "uri", 
            "label": "View detail", 
            "uri": "http://example.com/page/123" 
        }, 
        "actions": [{ 
                "type": "cameraRoll", 
                "label": "Camera roll" 
            }, 
            { 
                "type": "camera", 
                "label": "Camera" 
            }, 
            { 
                "type": "datetimepicker", 
                "label": "Select date", 
                "data": "storeId=12345", 
                "mode": "datetime", 
                "initial": "2017-12-25t00:00", 
                "max": "2018-01-24t23:59", 
                "min": "2017-12-25t00:00" 
            } 
        ] 
    } 
}
```

## <a name="adding-a-bot-to-teams"></a>Подключение бота к Teams

Добавленный в группу бот становится участником группы, которого можно упоминать (`@mentioned`) в ходе диалога. Фактически, боты получают сообщения, только если они упоминаются (`@mentioned`), следовательно, другие диалоги в канале им не отправляются.
См. сведения о [ведении бесед в групповых чатах и каналах с помощью бота Microsoft Teams](https://aka.ms/bots-con-channel).

Так как боты в группе или канале отвечают, только если они упоминаются (`@botname`) в сообщении, каждое сообщение, полученное ботом в канале группы, содержит собственное имя, которое должно распознаваться при анализе. Кроме того, боты могут распознавать имена других упомянутых пользователей и в свою очередь упоминать их в своих сообщениях.

### <a name="check-for-and-strip-bot-mention"></a>Проверка и удаление упоминания @bot

```csharp

Mention[] m = sourceMessage.GetMentions();
var messageText = sourceMessage.Text;

for (int i = 0;i < m.Length;i++)
{
    if (m[i].Mentioned.Id == sourceMessage.Recipient.Id)
    {
        //Bot is in the @mention list.
        //The below example will strip the bot name out of the message, so you can parse it as if it wasn't included. Note that the Text object will contain the full bot name, if applicable.
        if (m[i].Text != null)
            messageText = messageText.Replace(m[i].Text, "");
    }
}
```

```javascript
var text = message.text;
if (message.entities) {
    message.entities
        .filter(entity => ((entity.type === "mention") && (entity.mentioned.id.toLowerCase() === botId)))
        .forEach(entity => {
            text = text.replace(entity.text, "");
        });
    text = text.trim();
}

```

> [!IMPORTANT] 
> Добавлять бота по GUID рекомендуется только для тестирования. В противном случае функциональность бота будет существенно ограничена. Боты, используемые в рабочей среде, следует добавлять в Teams как часть приложения. См. сведения о [создании бота](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-create) и [тестировании и отладке бота Microsoft Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-test).


## <a name="additional-resources"></a>Дополнительные ресурсы

- [Сущности и типы действий](../bot-service-activities-entities.md)
- [Принципы использования действий в Bot Framework](https://aka.ms/botSpecs-activitySchema)

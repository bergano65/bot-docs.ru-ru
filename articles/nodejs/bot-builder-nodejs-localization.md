---
title: Поддержка локализации — Служба Azure Bot
description: Узнайте, как определить расположение пользователя и включить функциональные возможности локализации с помощью пакета SDK Bot Framework для Node.js.
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: eb9d2c484a76c0db5720f274b3c592d2d648a75b
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790876"
---
# <a name="support-localization"></a>Поддержка локализации

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

В Bot Builder входит система с широкими возможностями локализации для построения ботов, которая может взаимодействовать с пользователем на нескольких языках. Все запросы к боту можно локализовать с помощью файлов JSON, хранящихся в структуре каталогов бота. Если для обработки естественного языка используется такая система как LUIS, в [LuisRecognizer][LUISRecognizer] можно настроить отдельную модель для каждого языка, поддерживаемого ботом, и пакет SDK будет автоматически выбирать модель, которая совпадает с предпочитаемым языковым стандартом пользователя.

## <a name="determine-the-locale-by-prompting-the-user"></a>Определение языкового стандарта путем отправки запроса пользователю
Первым шагом к локализации бота является добавление возможности определять предпочитаемый язык пользователя. Пакет SDK предоставляет метод [session.preferredLocale()][preferredLocal] для сохранения и получения этого параметра на уровне пользователя. Следующий пример является диалогом для запроса у пользователя предпочитаемого языка и последующего сохранения его выбора.

``` javascript
bot.dialog('/localePicker', [
    function (session) {
        // Prompt the user to select their preferred locale
        builder.Prompts.choice(session, "What's your preferred language?", 'English|Español|Italiano');
    },
    function (session, results) {
        // Update preferred locale
        var locale;
        switch (results.response.entity) {
            case 'English':
                locale = 'en';
                break;
            case 'Español':
                locale = 'es';
                break;
            case 'Italiano':
                locale = 'it';
                break;
        }
        session.preferredLocale(locale, function (err) {
            if (!err) {
                // Locale files loaded
                session.endDialog(`Your preferred language is now ${results.response.entity}`);
            } else {
                // Problem loading the selected locale
                session.error(err);
            }
        });
    }
]);
```

## <a name="determine-the-locale-by-using-analytics"></a>Определение языкового стандарта с помощью аналитики
Другой способ определения языка пользователя — использовать такую службу как [API анализа текста](/azure/cognitive-services/cognitive-services-text-analytics-quick-start) для автоматического обнаружения языка пользователя на основе текста сообщения, который они отправили.

Приведенный ниже фрагмент кода иллюстрирует, как можно включить эту службу в бот.
``` javascript
var request = require('request');

bot.use({
    receive: function (event, next) {
        if (event.text && !event.textLocale) {
            var options = {
                method: 'POST',
                url: 'https://westus.api.cognitive.microsoft.com/text/analytics/v2.0/languages?numberOfLanguagesToDetect=1',
                body: { documents: [{ id: 'message', text: event.text }]},
                json: true,
                headers: {
                    'Ocp-Apim-Subscription-Key': '<YOUR API KEY>'
                }
            };
            request(options, function (error, response, body) {
                if (!error && body) {
                    if (body.documents && body.documents.length > 0) {
                        var languages = body.documents[0].detectedLanguages;
                        if (languages && languages.length > 0) {
                            event.textLocale = languages[0].iso6391Name;
                        }
                    }
                }
                next();
            });
        } else {
            next();
        }
    }
});
```

После добавления приведенного выше фрагмента кода в бот вызов [session.preferredLocale()][preferredLocal] автоматически возвратит определенный язык. Порядок поиска `preferredLocale()` выглядит следующим образом.
1. Языковой стандарт сохраняется вызовом `session.preferredLocale()`. Это значение сохраняется в `session.userData['BotBuilder.Data.PreferredLocale']`.
2. Обнаруженный языковой стандарт назначается `session.message.textLocale`.
3. Настроенный по умолчанию языковой стандарт бота, например английский (en).

Можно настроить языковой стандарт бота по умолчанию с помощью конструктора.

```javascript
var bot = new builder.UniversalBot(connector, {
    localizerSettings: { 
        defaultLocale: "es" 
    }
});
```

## <a name="localize-prompts"></a>Локализация запросов
Система локализации по умолчанию для пакета SDK Bot Framework основывается на файлах и позволяет боту поддерживать несколько языков, используя файлы JSON, хранящиеся на диске. По умолчанию система локализации будет искать запросы бота в файле **./locale/<IETF TAG>/index.json**, в котором <IETF TAG> является допустимым [языковым тегом IETF][IEFT], представляющим предпочтительный язык, для которого нужно найти запросы. 

На следующем снимке экрана показана структура каталогов для бота с поддержкой трех языков: английского, итальянского и испанского.

![Структура каталогов для трех языков](../media/locale-dir.png)

Структура файла представляет собой простую карту JSON идентификаторов сообщений в локализованных текстовых строках. Если значение является массивом, а не строкой, из массива в произвольном порядке выбирается один запрос при получении этого значения с помощью [session.localizer.gettext()][GetText]. 

Бот автоматически извлекает локализованную версию сообщения, если передать идентификатор сообщения в вызове [session.send()](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#send) вместо языкового текста.

```javascript
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("greeting");
        session.send("instructions");
        session.beginDialog('/localePicker');
    },
    function (session) {
        builder.Prompts.text(session, "text_prompt");
    }
]);
```

На внутреннем уровне пакет SDK вызывает [`session.preferredLocale()`][preferredLocale], чтобы получить предпочтительный языковой стандарт пользователя, а затем использует этот параметр в вызове [`session.localizer.gettext()`][GetText], чтобы сопоставить идентификатор сообщения с локализованной текстовой строкой.  Бывают случаи, когда необходимо вручную вызвать локализатор. Например, значения перечисления, переданные в [`Prompts.choice()`][promptsChoice], автоматически не локализуются. Поэтому перед вызовом запроса может потребоваться вручную извлечь локализованный список.

```javascript
var options = session.localizer.gettext(session.preferredLocale(), "choice_options");
builder.Prompts.choice(session, "choice_prompt", options);
```

Локализатор по умолчанию выполняет поиск идентификатора сообщения в нескольких файлах, и если найти идентификатор не удается (или если отсутствуют файлы локализации), он просто возвращает текст идентификатора, делая использование файлов локализации прозрачным и необязательным.  Поиск файлов осуществляется в следующем порядке:

1. производится поиск файла **index.json** с учетом языкового стандарта, возвращенного [`session.preferredLocale()`][preferredLocale];
2. если в языковой стандарт включен необязательный подтег **en-US**, то производится поиск корневого тега **en**;
3. производится поиск настроенного по умолчанию языкового стандарта бота.

## <a name="use-namespaces-to-customize-and-localize-prompts"></a>Использование пространств имен для настройки и локализации запросов
Локализатор по умолчанию поддерживает организацию пространства имен запросов для избежания конфликтов между идентификаторами сообщений.  Бот может переопределить организованные в пространстве имен запросы, чтобы настроить или перефразировать запросы из других пространств имен.  Вы можете использовать эту возможность для настройки встроенных сообщений в пакете SDK, позволяя либо добавлять поддержку дополнительных языков, либо просто перефразировать текущие сообщения в пакете SDK.  Например, можно изменить стандартное сообщение об ошибке пакета SDK, просто добавив файл с именем **BotBuilder.json** в каталог языкового стандарта и затем добавив запись для идентификатора сообщения `default_error`.

![BotBuilder.json для организации пространства имен языкового стандарта](../media/locale-namespacing.png)


## <a name="additional-resources"></a>Дополнительные ресурсы

Дополнительные сведения о локализации распознавателя см. в статье [Recognize user intent from message content](bot-builder-nodejs-recognize-intent-messages.md) (Распознавание намерения пользователя из содержимого сообщения).


[LUIS]: https://www.luis.ai/
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
[IntentRecognizerSetOptions]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html
[LUISRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISSample]: https://aka.ms/v3-js-luisSample
[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute
[preferredLocal]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[preferredLocale]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#preferredlocale
[promptsChoice]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice
[GetText]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ilocalizer.html#gettext
[IEFT]: https://en.wikipedia.org/wiki/IETF_language_tag


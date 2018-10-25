---
title: Создание бота с поддержкой речевых функций с навыками Кортаны | Документация Майкрософт
description: Узнайте, как создать бота с поддержкой речевых функций с навыками Кортаны и пакетом SDK Bot Builder для Node.js.
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e728a3999c484d19a78f03bd8eb7b8bd8833c39f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998041"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>Создание бота с поддержкой речи с навыками Кортаны

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.js](../nodejs/bot-builder-nodejs-cortana-skill.md)

Пакет SDK Bot Builder для Node.js позволяет создать бота с поддержкой речевых функций, подключая его к каналу Кортаны в качестве навыка Кортаны. Навыки Кортаны позволяют предоставлять функциональность через Кортану в ответ на ввод речи пользователя.

> [!TIP]
> Дополнительные сведения о том, что такое навык и каковы их возможности, см. в разделе [The Cortana Skills Kit][CortanaGetStarted] (Набор навыков Кортаны).

Создание навыка Кортаны с использованием Bot Framework требует очень мало знаний о Кортане и прежде всего состоит из создания бота. Одним из ключевых отличий от других ботов, которые можно создать, является то, что Кортана имеет как визуальные, так и звуковые компоненты. Визуальный компонент Кортана предоставляет область холста для отрисовки содержимого, например карты. Для звукового компонента необходимо предоставить текст или SSML в сообщениях бота, которые Кортана зачитывает пользователю, предоставляя боту голос. 

> [!NOTE]
> Помощник Кортана доступен на различных устройствах. У одних есть экран, в то время как другие, например автономный динамик, могут и не иметь его. Следует убедиться, что бот способен работать в обоих сценариях. Чтобы узнать, как проверить сведения об устройстве, см. статью [Get Cortana's Channel Data][CortanaSpecificEntities] (Получение данных о канале Кортаны).

## <a name="adding-speech-to-your-bot"></a>Добавление речевых функций для бота

Речевые сообщения бота представлены в виде SSML (Speech Synthesis Markup Language — язык разметки синтеза речи). Пакет SDK для Bot Builder позволяет включать SSML в ответы бота, чтобы контролировать то, что он говорит, в дополнение к тому, что он показывает.

### <a name="sessionsay"></a>Метод session.say

Чтобы говорить с пользователем, бот использует метод **session.say** вместо **session.send**. Он включает в себя необязательные параметры для отправки выходных данных SSML, а также вложения, такие как карты. 

Этот метод имеет следующий формат.

```session.say(displayText: string, speechText: string, options?: object)```

| Параметр | ОПИСАНИЕ |
|------|------|
| **displayText** | Текстовое сообщение для отображения в пользовательском интерфейсе Кортаны.|
| **speechText** | Текст или SSML, которые Кортана читает пользователю. |
| **options** | Объект [IMessage][IMessage], который может содержать вложение или подсказку для ввода. Подсказки для ввода указывают, что делает бот: принимает, ожидает или игнорирует входные данные. Вложения карты отображаются на панели холста Кортаны ниже информации **displayText**.   |

Свойство **inputHint** помогает указать Кортане, ожидает ли бот входные данные. Если используется встроенный запрос, для этого значения автоматически устанавливается значение по умолчанию **expectingInput**.


| Значение | ОПИСАНИЕ |
|------|------|
| **acceptingInput** | Бот пассивно готов к вводу, но не ожидает ответа. Кортана принимает входные данные от пользователя, если пользователь удерживает кнопку микрофона.|
| **expectingInput** | Указывает, что бот активно ожидает ответа от пользователя. Кортана слушает, что пользователь говорит в микрофон.  |
| **ignoringInput** | Кортана игнорирует входные данные. Бот может отправить эту подсказку, если он активно обрабатывает запрос и будет игнорировать входные данные от пользователей до тех пор, пока запрос не будет завершен.  |


В следующем примере показано, как Кортана читает простой текст или SSML.

```javascript
// Have Cortana read plain text
session.say('This is the text that Cortana displays', 'This is the text that is spoken by Cortana.');

// Have Cortana read SSML
session.say('This is the text that Cortana displays', '<speak version="1.0" xmlns="http://www.w3.org/2001/10/synthesis" xml:lang="en-US">This is the text that is spoken by Cortana.</speak>');
```

В этом примере показано, как дать знать Кортане, что ожидается пользовательский ввод. Микрофон остается открытым.
```javascript
// Add an InputHint to let Cortana know to expect user input
session.say('Hi there', 'Hi, what’s your name?', {
    inputHint: builder.InputHint.expectingInput
});
```
<!-- TODO: tip about time limit and batching -->


### <a name="prompts"></a>Запросы

Помимо использования метода **session.say ()** также можно передавать текст или SSML во встроенные запросы, используя параметры **speak** и **retrySpeak**.  

```javascript
builder.Prompts.text(session, 'text based prompt', {                                    
    speak: 'Cortana reads this out initially',                                               
    retrySpeak: 'This message is repeated by Cortana after waiting a while for user input',  
    inputHint: builder.InputHint.expectingInput                                              
});
```



<!-- TODO: Link to SSML library -->

Чтобы представить пользователю список вариантов, используйте **Prompts.choice**. Параметр **synonyms** позволяет более гибко распознавать высказывания пользователя. Параметр **value** возвращается в **results.response.entity**. Параметр **action** указывает метку, отображаемую ботом для выбора.

**Prompts.choice** поддерживает порядковый выбор. Это означает, что пользователь может выбрать элемент в списке, сказав "the first", "the second" или "the third". Например, учитывая следующие запросы, если пользователь запрашивает Кортану "the second option", запрос вернет значение 8.

```javascript
        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|ate|8 sided|8 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') // use helper function to format SSML
        });
```

В предыдущем примере SSML свойство **speak** запросы форматируется с использованием строк, которые хранятся в файле локализованных запросов в следующем формате. 

```json
{
    "choose_sides": "__Number of Sides__",
    "choose_sides_ssml": [
        "How many sides on the dice?",
        "Pick your poison.",
        "All the standard sizes are supported."
    ]
}
```


Затем вспомогательная функция создает необходимый корневой элемент Speech Synthesis Markup Language (SSML). 

```javascript

module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}
```

> [!TIP]
> Можно найти небольшой служебный модуль (ssml.js) для создания ответов бота на основе SSML в [Roller Skill Bot Sample](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill) (Пример бота навыка броска).
> Существует также несколько полезных библиотек SSML, доступных через [npm](https://www.npmjs.com/search?q=ssml), которые упрощают создание хорошо отформатированного SSML.

## <a name="display-cards-in-cortana"></a>Отображение карт в Кортане

Помимо произношения ответов, Кортана также может отображать вложения карточек. Кортана поддерживает следующие форматированные карточки:
* [HeroCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.herocard.html);
* [ReceiptCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.receiptcard.html);
* [ThumbnailCard](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.thumbnailcard.html).

Сведения о том, как эти карты выглядят в Кортане, см. в разделе [Principles of Cortana Skills design][CardDesign] (Принципы проектирования навыков Кортаны). Пример того, как добавить форматированную карточку в бот, см. в разделе [Добавление вложений в виде форматированных карточек в сообщения](bot-builder-nodejs-send-rich-cards.md). 

Следующий код демонстрирует, как добавить свойства **speak** и **inputHint** в сообщение, содержащее карту Hero.

```javascript 

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play yahtzee', 'Play Yahtzee')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'I\'m roller, the dice rolling bot. You can say \'roll some dice\''))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput); // Tell Cortana to accept input
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });


/** This helper function builds the required root element of a Speech Synthesis Markup Language (SSML) document. */
module.exports.speak = function (template, params, options) {
    options = options || {};
    var output = '<speak xmlns="http://www.w3.org/2001/10/synthesis" ' +
        'version="' + (options.version || '1.0') + '" ' +
        'xml:lang="' + (options.lang || 'en-US') + '">';
    output += module.exports.vsprintf(template, params);
    output += '</speak>';
    return output;
}

```
## <a name="sample-rollerskill"></a>Пример RollerSkill
Код в следующих разделах относится к примеру навыка Кортаны, который предназначен для того, чтобы бросать кости. Скачайте полный код для бота из [репозитория BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill).

Можно вызвать навык, назвав [имя вызова][InvocationNameGuidelines] помощнику Кортане. После [подключения бота к каналу Кортаны][CortanaChannel] и регистрации его в качестве навыка Кортана можно вызвать навык броска, сказав Кортане: "Ask Roller" (Вызвать навык броска) или "Ask Roller to Roll Dice" (Вызвать навык броска и бросить кости).

### <a name="explore-the-code"></a>Обзор кода

Пример RollerSkill начинается с открытия карты несколькими кнопками, чтобы сообщить пользователю, какие параметры для него доступны.

```javascript
/**
 *   Create your bot with a default message handler that receive messages from the user.
 * - This function is be called anytime the user's utterance isn't
 *   recognized by the other handlers in the bot.
 */
var bot = new builder.UniversalBot(connector, function (session) {
    // Just redirect to our 'HelpDialog'.
    session.replaceDialog('HelpDialog');
});

//...

bot.dialog('HelpDialog', function (session) {
    var card = new builder.HeroCard(session)
        .title('help_title')
        .buttons([
            builder.CardAction.imBack(session, 'roll some dice', 'Roll Dice'),
            builder.CardAction.imBack(session, 'play craps', 'Play Craps')
        ]);
    var msg = new builder.Message(session)
        .speak(speak(session, 'help_ssml'))
        .addAttachment(card)
        .inputHint(builder.InputHint.acceptingInput);
    session.send(msg).endDialog();
}).triggerAction({ matches: /help/i });
```

### <a name="prompt-the-user-for-input"></a>Запрос пользователю на ввод данных

В следующем диалоговом окне настраивается пользовательская игра для воспроизведения ботом.  Он спрашивает у пользователя, сколько сторон должны иметь кости, а затем сколько костей нужно бросить. После того как он построит структуру игры, он перейдет к отдельному диалогу 'PlayGameDialog'.

Запуская диалоговое окно, обработчик **triggerAction()** в этом диалоговом окне позволяет пользователю сказать что-то вроде "I'd like to roll some dice". Он использует регулярное выражение в соответствии с вводом пользователя, но можно так же легко использовать [намерение LUIS](./bot-builder-nodejs-recognize-intent-luis.md). 


```javascript


bot.dialog('CreateGameDialog', [
    function (session) {
        // Initialize game structure.
        // - dialogData gives us temporary storage of this data in between
        //   turns with the user.
        var game = session.dialogData.game = { 
            type: 'custom', 
            sides: null, 
            count: null,
            turns: 0
        };

        var choices = [
            { value: '4', action: { title: '4 Sides' }, synonyms: 'four|for|4 sided|4 sides' },
            { value: '6', action: { title: '6 Sides' }, synonyms: 'six|sex|6 sided|6 sides' },
            { value: '8', action: { title: '8 Sides' }, synonyms: 'eight|8 sided|8 sides' },
            { value: '10', action: { title: '10 Sides' }, synonyms: 'ten|10 sided|10 sides' },
            { value: '12', action: { title: '12 Sides' }, synonyms: 'twelve|12 sided|12 sides' },
            { value: '20', action: { title: '20 Sides' }, synonyms: 'twenty|20 sided|20 sides' },
        ];
        builder.Prompts.choice(session, 'choose_sides', choices, { 
            speak: speak(session, 'choose_sides_ssml') 
        });
    },
    function (session, results) {
        // Store users input
        // - The response comes back as a find result with index & entity value matched.
        var game = session.dialogData.game;
        game.sides = Number(results.response.entity);

        /**
         * Ask for number of dice.
         */
        var prompt = session.gettext('choose_count', game.sides);
        builder.Prompts.number(session, prompt, {
            speak: speak(session, 'choose_count_ssml'),
            minValue: 1,
            maxValue: 100,
            integerOnly: true
        });
    },
    function (session, results) {
        // Store users input
        // - The response is already a number.
        var game = session.dialogData.game;
        game.count = results.response;

        /**
         * Play the game we just created.
         * 
         * replaceDialog() ends the current dialog and start a new
         * one in its place. We can pass arguments to dialogs so we'll pass the
         * 'PlayGameDialog' the game we created.
         */
        session.replaceDialog('PlayGameDialog', { game: game });
    }
]).triggerAction({ matches: [
    /(roll|role|throw|shoot).*(dice|die|dye|bones)/i,
    /new game/i
 ]});
```

### <a name="render-results"></a>Результаты отрисовки

 Этот диалог является основным циклом игры. Бот хранит игровую структуру в **session.conversationData**, поэтому, если пользователь скажет "roll again", можно просто снова бросить тот же набор костей.

```javascript

bot.dialog('PlayGameDialog', function (session, args) {
    // Get current or new game structure.
    var game = args.game || session.conversationData.game;
    if (game) {
        // Generate rolls
        var total = 0;
        var rolls = [];
        for (var i = 0; i < game.count; i++) {
            var roll = Math.floor(Math.random() * game.sides) + 1;
            if (roll > game.sides) {
                // Accounts for 1 in a million chance random() generated a 1.0
                roll = game.sides;
            }
            total += roll;
            rolls.push(roll);
        }

        // Format roll results
        var results = '';
        var multiLine = rolls.length > 5;
        for (var i = 0; i < rolls.length; i++) {
            if (i > 0) {
                results += ' . ';
            }
            results += rolls[i];
        }

        // Render results using a card
        var card = new builder.HeroCard(session)
            .subtitle(game.count > 1 ? 'card_subtitle_plural' : 'card_subtitle_singular', game)
            .buttons([
                builder.CardAction.imBack(session, 'roll again', 'Roll Again'),
                builder.CardAction.imBack(session, 'new game', 'New Game')
            ]);
        if (multiLine) {
            //card.title('card_title').text('\n\n' + results + '\n\n');
            card.text(results);
        } else {
            card.title(results);
        }
        var msg = new builder.Message(session).addAttachment(card);

        // Determine bots reaction for speech purposes
        var reaction = 'normal';
        var min = game.count;
        var max = game.count * game.sides;
        var score = total/max;
        if (score == 1.0) {
            reaction = 'best';
        } else if (score == 0) {
            reaction = 'worst';
        } else if (score <= 0.3) {
            reaction = 'bad';
        } else if (score >= 0.8) {
            reaction = 'good';
        }

        // Check for special craps rolls
        if (game.type == 'craps') {
            switch (total) {
                case 2:
                case 3:
                case 12:
                    reaction = 'craps_lose';
                    break;
                case 7:
                    reaction = 'craps_seven';
                    break;
                case 11:
                    reaction = 'craps_eleven';
                    break;
                default:
                    reaction = 'craps_retry';
                    break;
            }
        }

        // Build up spoken response
        var spoken = '';
        if (game.turn == 0) {
            spoken += session.gettext('start_' + game.type + '_game_ssml') + ' ';
        } 
        spoken += session.gettext(reaction + '_roll_reaction_ssml');
        msg.speak(ssml.speak(spoken));

        // Increment number of turns and store game to roll again
        game.turn++;
        session.conversationData.game = game;

        /**
         * Send card and bot's reaction to user. 
         */

        msg.inputHint(builder.InputHint.acceptingInput);
        session.send(msg).endDialog();
    } else {
        // User started session with "roll again" so let's just send them to
        // the 'CreateGameDialog'
        session.replaceDialog('CreateGameDialog');
    }
}).triggerAction({ matches: /(roll|role|throw|shoot) again/i });
```

## <a name="next-steps"></a>Дополнительная информация
Если бот работает локально или он развернут в облаке, его можно вызвать из Кортаны. Дополнительные сведения о действиях, необходимых для тестирования навыка Кортаны, см. в разделе [Test a Cortana skill](../bot-service-debug-cortana-skill.md) (Тестирование навыка Кортаны).


## <a name="additional-resources"></a>Дополнительные ресурсы
* [Cortana Skills Kit][CortanaGetStarted] (Набор навыков Кортаны)
* [Добавление речи в сообщения](bot-builder-nodejs-text-to-speech.md)
* [Speech Synthesis Markup Language (SSML) reference][SSMLRef] (Справочник по SSML)
* [Voice design best practices for Cortana][VoiceDesign] (Рекомендации по проектированию речевых функций для Кортаны)
* [Card design best practices for Cortana][CardDesign] (Рекомендации по проектированию карточек для Кортаны)
* [Cortana Dev Center][CortanaDevCenter] (Центр разработки Кортаны)
* [Testing and debugging Cortana Skills][Cortana-TestBestPractice] (Тестирование и отладка навыков Кортаны)


[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/


[SSMLRef]: https://aka.ms/cortana-ssml
[IMessage]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage.html
[Send]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#send
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines
[VoiceDesign]: https://aka.ms/cortana-design-voice
[CardDesign]: https://aka.ms/cortana-design-card
[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-Publish]: https://aka.ms/cortana-publish


[CortanaTry]: https://aka.ms/try-cortana-bot
[CortanaChannel]: https://aka.ms/bot-cortana-channel
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice

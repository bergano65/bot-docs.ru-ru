---
title: Создание навыка Кортаны с помощью Node.js | Документация Майкрософт
description: Изучите основные понятия для создания навыка Кортаны, представленные в пакете SDK Bot Builder для Node.js.
keywords: Bot Framework, навык Кортаны, речь, .NET, Bot Builder, пакет SDK, ключевые понятия, основные понятия
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 99254c5f2de8b524212d8f6a268dbc37c128ab5e
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996479"
---
# <a name="key-concepts-for-building-a-bot-for-cortana-skills-using-nodejs"></a>Ключевые понятия для создания бота для навыков Кортаны с помощью Node.js
 
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!NOTE]
> Содержимое этой статьи является предварительным и будет обновляться.

В ней представлены ключевые понятия для создания навыка Кортаны, представленные в пакете SDK Bot Builder для Node.js. 

## <a name="what-is-a-cortana-skill"></a>Что такое навык Кортаны?
Навык Кортаны — это бот, который можно вызвать с помощью клиента Кортаны, аналогичный боту, встроенному в Windows 10. Пользователь запускает бот, произнося ключевые слова или фразы, связанные с этим ботом. Ключевые слова, которые используются для запуска бота, можно настроить на портале Bot Framework. 

Кортану можно рассматривать как канал с поддержкой речи, который может отправлять и принимать голосовые сообщения в дополнение к текстовому общению. Бот, публикуемый навык Кортаны, должен поддерживать как текстовое, так и устное общение. Платформа Bot Framework предоставляет методы для языка SSML, позволяющие определить сообщения, произносимые ботом.

## <a name="acknowledge-user-utterances"></a>Подтверждение высказываний пользователя 

<!-- Establishing conversational understanding -->
<!-- Placeholder: In this section, describe how you have to write your speech to sound natural -->


При создании бота с поддержкой речевых функций попробуйте найти в сеансе общения общий язык и взаимопонимание. Боту следует подтверждать высказывания пользователя, показывая, что пользователь услышан и понят.

Пользователь запутается, если система не сможет подтверждать его высказывания. Например, ниже приведен сеанс общения, в котором не вполне понятно, что делать, когда бот спрашивает "What's next?" (Что же дальше?)

```
Agent: Did you want to review some more of your profile?

User: No.

Agent: What's next?
```

Если бот будет добавлять "Okay" (ОК) в качестве подтверждения, это будет понятнее для пользователя.

```
Agent: Did you want to review some more of your profile?

User: No.

Agent: **Okay**, what's next?
```


Степени подтверждения в порядке усиления:
1. Непрерывное внимание.
2. Последующее соответствующее вовлечение.
3. Подтверждение: минимальный ответ или продолжение ("yeah" (Ага), "uh-huh" (Ух), "okay" (ОК), "great" (Отлично)).
4. Демонстрация: бот показывает, что понял пользователя, перефразируя или завершая его фразу.
5. Отображение: повторение всей фразы или ее части.

#### <a name="acknowledgement-and-next-relevant-contribution"></a>Подтверждение и последующее вовлечение
Пользователь: "..." "I need to travel in May" (Мне нужно слетать кое-куда в мае).
Агент: "**And**, what day in May did you want to travel?" (И в какой именно день мая вы хотите кое-куда слетать?)
Пользователь: "OK I need to be there from the 12th to the 15th?" (Хорошо, мне нужно быть там с 12-го по 15-е?)
Агент: "**And**, you're flying into what city?" (И в какой город вы летите?)

#### <a name="grounding-by-demonstration"></a>Подтверждение с помощью демонстрации
Пользователь: "..." "I need to travel in May" (Мне нужно слетать кое-куда в мае).
Агент: "**And**, what day in May did you want to travel?" (И в какой именно день мая вы хотите кое-куда слетать?)
Пользователь: "OK I need to be there from the 12th to the 15th?" (Хорошо, мне нужно быть там с 12-го по 15-е?)
Агент: "**And**, you're flying into what city?" (И в какой город вы летите?)


### <a name="closure"></a>Завершение

Бот, выполняющий действие, должен подтвердить его успешное завершение.
Важно также указать ошибку или понимание фразы. 
* Безмолвное завершение: когда вы нажимаете кнопку лифта, она подсвечивается.
Двухэтапный процесс:
* Уровень представления 
* принятие.


### <a name="differences-in-content-presentation"></a>Различия в представлении содержимого
При разработке бота с поддержкой речевых функций помните, что устный диалог часто отличается от текстовых сообщений, отправляемых ботом.
<!-- If there are differences in what the bot will say, in the text vs the speak fields of a prompt or in a waterfall, for example, discuss them here.

## Speech

You bot uses the **session.say** method to speak to the user. The speak method has three overloads:
* If you pass only one parameter to **session.say**, it can be a text parameter.
* If you pass two parameters to **session.say**, it can take text and SSML.
* If you pass three parameters, the third parameter takes an options structure that specifies all the options you can pass to build an **IMessage** object.

```javascript
var bot = new builder.UniversalBot(connector, function (session) {
    session.say("Hello... I'm a decision making bot.'.", 
        ssml.speak("Hello. I can help you answer all of life's tough questions."));
    session.replaceDialog('rootMenu');
});

```
## Speech in messages

The **IMessage** object provides a **speak** property for SSML. It can be used to play a .wav file.

The **inputHint** property helps indicate to Cortana whether your bot is expecting input. If you're using a built-in prompt, this value is automatically set to the default of **expectingInput**.

The **inputHint** property can take the following values: 
* **expectingInput**: Indicates that the bot is actively expecting a response from the user. Cortana listens for the user to speak into the microphone.
* **acceptingInput**: Indicates that the bot is passively ready for input but is not waiting on a response. Cortana accepts input from the user if the user holds down the microphone button.
* **ignoringInput**: Cortana is ignoring input. Your bot may send this hint if it is actively processing a request and will ignore input from users until the request is complete.

Prompts can take a `speak:` or `retrySpeak` option.

```javascript
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```

Prompts.number has *ordinal support*, meaning that you can say "the last", "the first", "the next-to-last" to choose an item in a list.




## Using synonyms

<!-- Axl Rose example -->     
```javascript   
         var choices = [
            { 
                value: 'flipCoinDialog',
                action: { title: "Flip A Coin" },
                synonyms: 'toss coin|flip quarter|toss quarter'
            },
            {
                value: 'rollDiceDialog',
                action: { title: "Roll Dice" },
                synonyms: 'roll die|shoot dice|shoot die'
            },
            {
                value: 'magicBallDialog',
                action: { title: "Magic 8-Ball" },
                synonyms: 'shake ball'
            },
            {
                value: 'quit',
                action: { title: "Quit" },
                synonyms: 'exit|stop|end'
            }
        ];
        builder.Prompts.choice(session, "Decision Options", choices, {
            listStyle: builder.ListStyle.button,
            speak: ssml.speak("How would you like me to decide?")
        });
```


## <a name="configuring-your-bot"></a>Настройка бота

## <a name="prompts"></a>Запросы


## <a name="additional-resources"></a>Дополнительные ресурсы

[CortanaGetstarted]: /cortana/getstarted
[SSMLRef]: https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx

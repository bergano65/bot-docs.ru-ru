---
title: Создание навыка Кортаны с помощью Node.js | Документация Майкрософт
description: Изучите основные понятия для создания навыка Кортаны, представленные в пакете SDK Bot Framework для Node.js.
keywords: Bot Framework, навык Кортаны, речь, .NET, Bot Builder, пакет SDK, ключевые понятия, основные понятия
author: DeniseMak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/10/2019
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5de773f6f8f4d46c0c1fe880588f2530c3c68f56
ms.sourcegitcommit: cacd381d185b2b8b7fb99082baf83d9f65dde341
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/11/2019
ms.locfileid: "59508211"
---
# <a name="key-concepts-for-building-a-bot-for-cortana-skills-using-nodejs"></a>Ключевые понятия для создания бота для навыков Кортаны с помощью Node.js
 
[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!NOTE]
> Содержимое этой статьи является предварительным и будет обновляться.

В ней представлены ключевые понятия для создания навыка Кортаны, представленные в пакете SDK Bot Framework для Node.js. 

## <a name="what-is-a-cortana-skill"></a>Что такое навык Кортаны?
Навык Кортаны — это бот, который можно вызвать с помощью клиента Кортаны, аналогичный боту, встроенному в Windows 10. Пользователь запускает бот, произнося ключевые слова или фразы, связанные с этим ботом. Ключевые слова, которые используются для запуска бота, можно настроить на портале Bot Framework. 

Кортану можно рассматривать как канал с поддержкой речи, который может отправлять и принимать голосовые сообщения в дополнение к текстовому общению. Бот, публикуемый навык Кортаны, должен поддерживать как текстовое, так и устное общение. Платформа Bot Framework предоставляет методы для языка SSML, позволяющие определить сообщения, произносимые ботом.

## <a name="acknowledge-user-utterances"></a>Подтверждение высказываний пользователя 

<!-- Establishing conversational understanding -->
<!-- Placeholder: In this section, describe how you have to write your speech to sound natural -->


При создании бота с поддержкой речевых функций попробуйте найти в сеансе общения общий язык и взаимопонимание. Боту следует подтверждать высказывания пользователя, показывая, что пользователь услышан и понят.

Пользователь запутается, если система не сможет подтверждать его высказывания. Например, ниже приведен сеанс общения, в котором не вполне понятно, что делать, когда бот спрашивает "What's next?" (Что же дальше?)

> **Кортана**: Did you want to review some more of your profile? (Хотите просмотреть другие данные профиля?)  
> **User**: №  
> **Кортана**: Что дальше?

Если бот будет добавлять "Okay" (ОК) в качестве подтверждения, это будет понятнее для пользователя.

> **Кортана**: Did you want to review some more of your profile? (Хотите просмотреть другие данные профиля?)  
> **User**: №  
> **Кортана**: **Okay**, what's next? (Хорошо. Что дальше?)

Степени подтверждения в порядке усиления:

1. Непрерывное внимание.
2. Последующее соответствующее вовлечение.
3. Подтверждение. Минимальный ответ или фраза для поддержания разговора: "yeah" (Ну да), "uh-huh" (Ага), "okay" (ОК), "great" (Отлично).
4. Демонстрация. Подтверждение понимания путем перефразирования или завершения фразы.
5. Отображение. Повторение всей фразы или ее части.

### <a name="acknowledgement-and-next-relevant-contribution"></a>Подтверждение и последующее вовлечение

> **User**: "I need to travel in May" (Мне нужно слетать кое-куда в мае).  
> **Кортана**: **Okay**. (Хорошо.) What day in May did you want to travel? (В какой день мая вы хотите кое-куда слетать?)  
> **User**: Well, I need to be there from the 12th to the 15th? (Мне нужно быть там с 12-го по 15-е.)  
> **Кортана**: **Okay**. (Хорошо.) What city are you flying into ? (В какой город вы летите?)  

### <a name="grounding-by-demonstration"></a>Подтверждение с помощью демонстрации

> **User**: "I need to travel in May" (Мне нужно слетать кое-куда в мае).  
> **Кортана**: **And**, what day in May did you want to travel?" (А в какой день мая вы хотите кое-куда слетать?)  
> **User**: Okay, I need to be there from the 12th to the 15th? (Мне нужно быть там с 12-го по 15-е.)  
> **Кортана**: **And** you're flying into what city? (А в какой город вы летите?)  
    
### <a name="closure"></a>Завершение

Бот, выполняющий действие, должен подтвердить его успешное завершение. Важно также указать ошибку или понимание фразы. 

* Безмолвное завершение. Когда вы нажимаете кнопку лифта, она подсвечивается.  
Этот процесс включает два этапа:
    * представление (при нажатии кнопки);
    * принятие (когда кнопка загорается).

## <a name="differences-in-content-presentation"></a>Различия в представлении содержимого
Учтите, что Кортана поддерживается на различных устройствах, но не все из них оборудованы экранами. Один из факторов, которые вам надо учитывать при проектировании бота с поддержкой речевых функций, заключается в том, что разговорный диалог часто будет не совпадать с текстовыми сообщениями, отображаемыми ботом.
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

Prompts must use the `speak:` option.

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

Документация по Кортане: [документация по навыкам Кортаны](/cortana/skills/).

Справочник по SSML для Кортаны: [справочник по языку разметки синтеза речи (SSML)](/cortana/skills/speech-synthesis-markup-language).

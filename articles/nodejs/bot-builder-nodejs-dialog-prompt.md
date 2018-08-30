---
title: Запрос пользователю на ввод данных | Документация Майкрософт
description: Сведения об использовании запросов для сбора данных, введенных пользователем с помощью пакета SDK Bot Builder для Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: aa20dc396b68ede3271d12a8deab2e673a79d1d1
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904486"
---
# <a name="prompt-for-user-input"></a>Запрос пользователю на ввод данных

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Пакет SDK Bot Builder для Node.js предоставляет набор встроенных запросов для упрощения сбора данных, вводимых от пользователя. 

*Запрос* используется всякий раз, когда боту нужно, чтобы пользователь ввел данные. Запросы можно использовать, чтобы попросить пользователя ввести ряд данных, объединив запросы в каскад. Запросы можно использовать в сочетании с [каскадом](bot-builder-nodejs-dialog-waterfall.md), что может помочь в [управлении ходом общения](bot-builder-nodejs-manage-conversation-flow.md) в боте. 

Из этой статьи можно узнать, как работают запросы и как их можно использовать для сбора сведений от пользователей.

## <a name="prompts-and-responses"></a>Запросы и ответы

Всякий раз, когда необходимо получить данные от пользователя, можно отправить запрос, дождаться ответа от пользователя, а затем обработать полученную информацию и отправить пользователю ответ.

Приведенный ниже пример кода запрашивает у пользователя его имя и отвечает приветственным сообщением.

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

Используя эту базовую конструкцию, можно моделировать ход общения, добавляя столько запросов и ответов, сколько требуется боту.

## <a name="prompt-results"></a>Результаты запроса 

Встроенные запросы выполняются как [диалоги](bot-builder-nodejs-dialog-overview.md), которые возвращают ответ пользователя в поле `results.response`. Ответы, полученные объектами JSON, отображаются в поле `results.response.entity`. Результаты запроса может получать любой тип [обработчика диалога](bot-builder-nodejs-dialog-overview.md#dialog-handlers). Как только бот получает ответ, он может использовать его или с помощью метода [`session.endDialogWithResult`][EndDialogWithResult] передать его диалогу вызова.

В приведенном ниже примере кода показано, как с помощью метода `session.endDialogWithResult` вернуть результат запроса в диалог вызова. В этом примере диалог `greetings` использует результаты запроса, полученные диалогом `askName`, чтобы приветствовать пользователя по имени.

```javascript
// Ask the user for their name and greet them by name.
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="prompt-types"></a>Типы запросов
В пакет SDK Bot Builder для Node.js включено несколько различных типов встроенных запросов. 

|**Тип запроса**     | **Описание** |     
| ------------------ | --------------- |
|[Prompts.text](#promptstext) | Запрашивает у пользователя ввод строки текста. |     
|[Prompts.confirm](#promptsconfirm) | Запрашивает у пользователя подтверждение действия.| 
|[Prompts.number](#promptsnumber) | Запрашивает у пользователя ввод числа.     |
|[Prompts.time](#promptstime) | Запрашивает у пользователя время или дату и время.      |
|[Prompts.choice](#promptschoice) | Запрос пользователю на выбор варианта из списка.    |
|[Prompts.attachment](#promptsattachment) | Запрос пользователю на загрузку изображения или видео.|       

В следующих разделах приведены дополнительные сведения о каждом типе запроса.

### <a name="promptstext"></a>Prompts.text

Используйте метод [Prompts.text()][PromptsText], чтобы запросить у пользователя ввод **строки текста**. Запрос возвращает ответ пользователя как значение параметра [IPromptTextResult][IPromptTextResult].

```javascript
builder.Prompts.text(session, "What is your name?");
```

### <a name="promptsconfirm"></a>Prompts.confirm

Используйте метод [Prompts.confirm()][PromptsConfirm], чтобы запросить у пользователя подтверждение действия, содержащего ответ **Да или Нет**. Запрос возвращает ответ пользователя как значение параметра [IPromptConfirmResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html).

```javascript
builder.Prompts.confirm(session, "Are you sure you wish to cancel your order?");
```

### <a name="promptsnumber"></a>Prompts.number

Используйте метод [Prompts.number()][PromptsNumber], чтобы запросить у пользователя ввод **числа**. Запрос возвращает ответ пользователя как значение параметра [IPromptNumberResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html).

```javascript
builder.Prompts.number(session, "How many would you like to order?");
```

### <a name="promptstime"></a>Prompts.time

Используйте метод [Prompts.text()][PromptsTime], чтобы запросить у пользователя ввод **времени** или **даты и времени**. Запрос возвращает ответ пользователя как значение параметра [IPromptTimeResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html). Для анализа ответа пользователя и поддержки как относительных (например, "in 5 minutes"), так и не относительных ответов (например, "June 6th at 2pm"), в платформе используется библиотека [Chrono](https://github.com/wanasit/chrono).

Поле [results.response](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html#response), в котором отображается ответ пользователя, содержит объект [сущности](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html), указывающий дату и время. Чтобы сопоставить дату и время в объекте JavaScript `Date`, используйте метод [EntityRecognizer.resolveTime()](http://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime).

> [!TIP] 
> Время, которое пользователь вводит, преобразуется в формат UTC и зависит от часового пояса сервера, на котором размещен бот. Необходимо учитывать часовые пояса, так как сервер может находиться в другом часовом поясе, отличном от часового пояса пользователя. Для преобразования даты и времени в локальное время попросите пользователя указать, в каком часовом поясе он находится.

```javascript
bot.dialog('createAlarm', [
    function (session) {
        session.dialogData.alarm = {};
        builder.Prompts.text(session, "What would you like to name this alarm?");
    },
    function (session, results, next) {
        if (results.response) {
            session.dialogData.name = results.response;
            builder.Prompts.time(session, "What time would you like to set an alarm for?");
        } else {
            next();
        }
    },
    function (session, results) {
        if (results.response) {
            session.dialogData.time = builder.EntityRecognizer.resolveTime([results.response]);
        }

        // Return alarm to caller  
        if (session.dialogData.name && session.dialogData.time) {
            session.endDialogWithResult({ 
                response: { name: session.dialogData.name, time: session.dialogData.time } 
            }); 
        } else {
            session.endDialogWithResult({
                resumed: builder.ResumeReason.notCompleted
            });
        }
    }
]);
```

### <a name="promptschoice"></a>Prompts.choice

Используйте метод [Prompts.choice()][PromptsChoice], чтобы запросить у пользователя  **выбор варианта из списка**. Пользователь может передать свой выбор путем ввода номера, связанного с выбранным вариантом или путем ввода имени выбранного варианта. Поддерживаются как полные, так и частичные совпадения имен параметров. Запрос возвращает ответ пользователя как значение параметра [IPromptChoiceResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html). 

Чтобы указать стиль списка, который представлен пользователю, задайте свойство [IPromptOptions.listStyle](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptoptions.html#liststyle). В следующей таблице приведены значения перечисления `ListStyle` для этого свойства.


Далее приведены значения перечисления `ListStyle`.

| Индекс | ИМЯ | ОПИСАНИЕ |
| ---- | ---- | ---- |
| 0 | Нет | Список не отображается. Параметр используется, когда список является частью запроса. |
| 1 | строкой | Варианты отображаются как список в одну строку следующего вида: "1. Красный, 2. Зеленый, или 3. Синий". |
| 2 | list | Варианты отображаются в виде нумерованного списка. |
| 3 | кнопка | Варианты отображаются как кнопки для каналов, поддерживающих кнопки. Для остальных каналов они будут отображены в виде текста. |
| 4. | авто | Выбор стиля осуществляется автоматически и основывается на канале и количестве вариантов для выбора. | 

Доступ к этому перечислению можно получить из объекта `builder` или можно указать индекс, чтобы выбрать `ListStyle`. Например, обе строки кода, приведенные ниже, при выполнении имеют одинаковый результат.

```javascript
// ListStyle passed in as Enum
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: builder.ListStyle.button });

// ListStyle passed in as index
builder.Prompts.choice(session, "Which color?", "red|green|blue", { listStyle: 3 });
```

Для того чтобы указать список параметров, можно использовать строку, разделенную символами вертикальной черты (`|`), массив строк или карту объектов.

Строка, разделенная символами вертикальной черты. 

```javascript
builder.Prompts.choice(session, "Which color?", "red|green|blue");
```

Массив строк.

```javascript
builder.Prompts.choice(session, "Which color?", ["red","green","blue"]);
```

Карта объекта. 

```javascript
var salesData = {
    "west": {
        units: 200,
        total: "$6,000"
    },
    "central": {
        units: 100,
        total: "$3,000"
    },
    "east": {
        units: 300,
        total: "$9,000"
    }
};

bot.dialog('getSalesData', [
    function (session) {
        builder.Prompts.choice(session, "Which region would you like sales for?", salesData); 
    },
    function (session, results) {
        if (results.response) {
            var region = salesData[results.response.entity];
            session.send(`We sold ${region.units} units for a total of ${region.total}.`); 
        } else {
            session.send("OK");
        }
    }
]);
```

### <a name="promptsattachment"></a>Prompts.attachment

Используйте метод [Prompts.attachment()][PromptsAttachment], чтобы запросить у пользователя загрузку файла, например изображения или видео. Запрос возвращает ответ пользователя как значение параметра [IPromptAttachmentResult](http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html).

```javascript
builder.Prompts.attachment(session, "Upload a picture for me to transform.");
```

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знаете, как провести пользователей через каскад и запросить у них информацию, рассмотрим способы, которые помогут лучше управлять последовательностью общения.

> [!div class="nextstepaction"]
> [Manage conversation flow with dialogs](bot-builder-nodejs-dialog-manage-conversation-flow.md) (Управление последовательностью общения с помощью диалогов)


[SendAttachments]: bot-builder-nodejs-send-receive-attachments.md
[SendCardWithButtons]: bot-builder-nodejs-send-rich-cards.md
[RecognizeUserIntent]: bot-builder-nodejs-recognize-intent-messages.md
[SaveUserData]: bot-builder-nodejs-save-user-data.md

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[Session]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session


[SendTyping]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#sendtyping

[EndDialogWithResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#enddialogwithresult

[IPromptResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html

[Result_Response]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#reponse

[ResumeReason]: https://docs.botframework.com/en-us/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html

[Result_Resumed]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed

[entity]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ientity.html

[ResolveTime]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#resolvetime

[PromptsRef]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html

[PromptsText]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#text

[IPromptTextResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttextresult.html

[PromptsConfirm]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#confirm

[IPromptConfirmResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptconfirmresult.html

[PromptsNumber]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#number

[IPromptNumberResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptnumberresult.html

[PromptsTime]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#time

[IPromptTimeResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iprompttimeresult.html

[PromptsChoice]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#choice

[IPromptChoiceResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptchoiceresult.html

[PromptsAttachment]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.__global.iprompts.html#attachment

[IPromptAttachmentResult]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptattachmentresult.html

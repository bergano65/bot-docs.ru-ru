---
title: Извлечение типизированных результатов LUIS | Документация Майкрософт
description: Сведения об использовании LUIS для извлечения сущностей с помощью пакета SDK Bot Builder.
keywords: intents, entities, LUISGen, extract
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/16/17
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6a88b0a7f44f43d0676ba88314fbba7c486e6be4
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39304963"
---
# <a name="extract-intents-and-entities-using-luisgen"></a>Извлечение намерений и сущностей с помощью LUISGen

Кроме распознавания намерений, приложение LUIS умеет извлекать сущности, то есть наиболее важные слова для выполнения запроса пользователя. Например, в сценарии предварительного заказа столика в ресторане приложение LUIS может извлечь из сообщения пользователя такие данные, как размер группы, дата резервирования или расположение ресторана. 


Вы можете заранее создать классы с помощью [средства LUISGen](https://github.com/Microsoft/botbuilder-tools/tree/master/LUISGen), чтобы упростить извлечение сущностей из LUIS в коде бота.

В командной строке Node.js установите `luisgen` в каталог, доступный через переменную глобального пути.
```
npm install -g luisgen
```

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="generate-a-luis-results-class"></a>Создание класса результатов LUIS

Скачайте [пример бота LUIS CafeBot](https://aka.ms/contosocafebot-luis) и запустите LUISGen из его корневой папки:

```
luisgen Assets\LU\models\LUIS\cafeLUISModel.json -cs ContosoCafeBot.CafeLUISModel
```

## <a name="examine-the-generated-code"></a>Изучение созданного кода
Будет создан файл **cafeLUISModel.cs**, который можно сразу добавить в проект. Он содержит класс `cafeLuisModel` для получения от LUIS результатов со строгой типизацией.

Этот класс содержит перечисление для получения намерений, определенных в приложении LUIS.
```cs
public enum Intent {
    Book_Table, 
    Greeting, 
    None, 
    Who_are_you_intent
};
```
Также он имеет свойство `Entities`. Так как любая сущность может встречаться в сообщении пользователя несколько раз, класс `_Entities` определяет массивы для сущностей каждого типа. 
```cs
public class _Entities
{
    // Simple entities
    public string[] partySize;

    // Built-in entities
    public Microsoft.Bot.Builder.Ai.LUIS.DateTimeSpec[] datetime;
    public double[] number;

    // Lists
    public string[][] cafeLocation;

    // Instance
    public class _Instance
    {
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] partySize;
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] datetime;
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] number;
        public Microsoft.Bot.Builder.Ai.LUIS.InstanceData[] cafeLocation;
    }
    [JsonProperty("$instance")]
    public _Instance _instance;
}
public _Entities Entities;
```

> [!NOTE]
> Все типы сущностей должны быть массивами, так как LUIS может обнаружить в высказывании пользователя несколько сущностей соответствующего типа. Например, если пользователь произнесет фразу "make reservations for 5pm tomorrow and 9pm next Saturday" (Зарезервировать места на 5 вечера завтра и на 9 вечера в следующую субботу), в результатах `datetime` вернутся "5pm tomorrow" и "9pm next Saturday".
>

|Сущность | type | Пример | Примечания |
|-------|-----|------|---|
|partySize| string[]| Группа людей (`four`)| Простой объект распознает строки. В нашем примере Entities.partySize[0] имеет тип `"four"`.
|Datetime| [DateTimeSpec](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.datetimespec?view=botbuilder-4.0.0-alpha)[]| Резервирование на время `9pm tomorrow`| Каждый объект **DateTimeSpec** содержит поле timex, а котором содержатся возможные значения времени в формате **timex**. Дополнительные сведения о формате timex см. здесь: http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3 Дополнительные сведения о библиотеке распознавания см. здесь: https://github.com/Microsoft/Recognizers-Text
|number| double[]| Группа людей (`four`), в том числе дети (`2`) | `number` определяет все числа, а не только размер группы. <br/> Например, для высказывания "Party of four which includes 2 children" (Группа из четырех человек, в том числе двое детей) параметр `Entities.number[0]` будет иметь значение 4, а `Entities.number[1]` — 2.
|cafelocation| string[][]; | Резервирование в расположении `Seattle`.| cafeLocation — это списочный объект, который содержит все распознанные элементы некоторых списков. Если хотя бы одна распознанная сущность входит в несколько списков, этот параметр содержит массив массивов. Например, фраза "reservation in Washington" (Зарезервировать ресторан в Вашингтоне) даст совпадения со штатом Вашингтон и городом Вашингтон (округ Колумбия).

Свойство `_Instance` содержит [InstanceData](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.instancedata?view=botbuilder-4.0.0-alpha) с дополнительными подробностями о каждой распознанной сущности.

## <a name="check-intents-in-your-bot"></a>Проверка намерений в коде бота
В файле **CafeBot.cs** обратите внимание на код, заключенный в блок `OnTurn`. Здесь вы видите, что бот вызывает LUIS и проверяет намерения, чтобы выбрать подходящий диалог. Результаты LUIS после вызова [`Recognize`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.ai.luis.luisrecognizer?view=botbuilder-4.0.0-alpha#methods) передаются в качестве аргумента в диалог `BookTable`.



```cs
if(!context.Responded)
{
    // call LUIS and get results
    LuisRecognizer lRecognizer = createLUISRecognizer();
    // Use the generated class as the type parameter to Recognize()
    cafeLUISModel lResult = await lRecognizer.Recognize<cafeLUISModel>(utterance, ct);
    Dictionary<string,object> lD = new Dictionary<string,object>();
    if(lResult != null) {
        lD.Add("luisResult", lResult);
    }
    
    // top level dispatch
    switch (lResult.TopIntent().intent)
    {
        case cafeLUISModel.Intent.Greeting:
            await context.SendActivity("Hello!");
            if (userState.sendCards) await context.SendActivity(CreateCardResponse(context.Activity, createWelcomeCardAttachment()));
            break;

        case cafeLUISModel.Intent.Book_Table:
            await dc.Begin("BookTable", lD);
            break;

        case cafeLUISModel.Intent.Who_are_you_intent:
            await context.sendActivity("I'm the Contoso Cafe bot.");
            break;

        case cafeLUISModel.Intent.None:
        default:
            await getQnAResult(context);
            break;
    }
}
```

## <a name="extract-entities-in-a-dialog"></a>Извлечение сущностей в диалоге

Теперь перейдите к элементу `Dialogs/BookTable.cs`. Диалог `BookTable` содержит каскадную последовательность шагов, каждый из которых проверяет наличие некоторой сущности в результатах LUIS, переданных в `args`. Если нужная сущность не найдена, диалог пропускает соответствующий ей запрос путем вызова `next()`. Если сущность найдена, диалог выполняет этот запрос и передает полученный ответ пользователя на следующий шаг каскадной последовательности.

```cs
    Dialogs.Add("BookTable",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                dc.ActiveDialog.State = new Dictionary<string, object>();
                IDictionary<string,object> state = dc.ActiveDialog.State;

                // add any LUIS entities to active dialog state.
                if(args.ContainsKey("luisResult")) {
                    cafeLUISModel lResult = (cafeLUISModel)args["luisResult"];
                    updateContextWithLUIS(lResult, ref state);
                }
                
                // prompt if we do not already have cafelocation
                if(state.ContainsKey("cafeLocation")) {
                    state["bookingLocation"] = state["cafeLocation"];
                    await next();
                } else {
                    await dc.Prompt("choicePrompt", "Which of our locations would you like?", promptOptions);
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("cafeLocation")) {
                    var choiceResult = (FoundChoice)args["Value"];
                    state["bookingLocation"] = choiceResult.Value;
                }
                bool promptForDateTime = true;
                if(state.ContainsKey("datetime")) {
                    // validate timex
                    var inputdatetime = new string[] {(string)state["datetime"]};
                    var results = evaluateTimeX((string[])inputdatetime);
                    if(results.Count != 0) {
                        var timexResolution = results.First().TimexValue;
                        var timexProperty = new TimexProperty(timexResolution.ToString());
                        var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                        state["bookingDateTime"] = bookingDateTime;
                        promptForDateTime = false;
                    }
                }
                // prompt if we do not already have date and time
                if(promptForDateTime) {
                    await dc.Prompt("timexPrompt", "When would you like to arrive? (We open at 4PM.)",
                                    new PromptOptions { RetryPromptString = "We only accept reservations for the next 2 weeks and in the evenings between 4PM - 8PM" });
                } else {
                    await next();
                }                       
                
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("datetime")) { 
                    var timexResult = (TimexResult)args;
                    var timexResolution = timexResult.Resolutions.First();
                    var timexProperty = new TimexProperty(timexResolution.ToString());
                    var bookingDateTime = $"{timexProperty.ToNaturalLanguage(DateTime.Now)}";
                    state["bookingDateTime"] = bookingDateTime;
                }
                // prompt if we already do not have party size
                if(state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = state["partySize"];
                    await next();
                } else {
                    await dc.Prompt("numberPrompt", "How many in your party?");
                }
            },
            async (dc, args, next) =>
            {
                var state = dc.ActiveDialog.State;
                if(!state.ContainsKey("partySize")) {
                    state["bookingGuestCount"] = args["Value"];
                }

                await dc.Prompt("confirmationPrompt", $"Thanks, Should I go ahead and book a table for {state["bookingGuestCount"].ToString()} guests at our {state["bookingLocation"].ToString()} location for {state["bookingDateTime"].ToString()} ?");
            },
            async (dc, args, next) =>
            {
                var dialogState = dc.ActiveDialog.State;

                // TODO: Verify user said yes to confirmation prompt

                // TODO: book the table! 

                await dc.Context.SendActivity($"Thanks, I have {dialogState["bookingGuestCount"].ToString()} guests booked for our {dialogState["bookingLocation"].ToString()} location for {dialogState["bookingDateTime"].ToString()}.");
            }
        }
    );
}

// This helper method updates dialog state with any LUIS results
private void updateContextWithLUIS(cafeLUISModel lResult, ref IDictionary<string,object> dialogContext) {
    if(lResult.Entities.cafeLocation != null && lResult.Entities.cafeLocation.GetLength(0) > 0) {
        dialogContext.Add("cafeLocation", lResult.Entities.cafeLocation[0][0]);
    }
    if(lResult.Entities.partySize != null && lResult.Entities.partySize.GetLength(0) > 0) {
        dialogContext.Add("partySize", lResult.Entities.partySize[0][0]);
    } else {
        if(lResult.Entities.number != null && lResult.Entities.number.GetLength(0) > 0) {
            dialogContext.Add("partySize", lResult.Entities.number[0]);
        }
    }
    if(lResult.Entities.datetime != null && lResult.Entities.datetime.GetLength(0) > 0) {
        dialogContext.Add("datetime", lResult.Entities.datetime[0].Expressions[0]);
    }
}
```
## <a name="run-the-sample"></a>Запуск примера

Откройте `ContosoCafeBot.sln` в Visual Studio 2017 и запустите бот. Подключитесь к запущенному примеру с помощью [эмулятора Bot Framework](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator).

Произнесите команду `reserve a table`, чтобы начать диалог для резервирования.

![Запуск бота](media/how-to-luisgen/run-bot.png)

# <a name="typescripttabjs"></a>[TypeScript](#tab/js)

Скачайте [пример бота CafeBot_LUIS](https://aka.ms/contosocafebot-typescript-luis-dialogs) и запустите LUISGen из его корневой папки:

```
luisgen cafeLUISModel.json -ts CafeLUISModel
```

Будет создан файл **CafeLUISModel.ts**, который можно сразу добавить в проект. Чтобы получить от распознавателя LUIS типизированный результат, укажите в созданном файле нужные типы.


```typescript
// call LUIS and get typed results
await luisRec.recognize(context).then(async (res : any) => 
{    
    // get a typed result
    var typedresult = res as CafeLUISModel;   
    
```

## <a name="pass-the-typed-result-to-a-dialog"></a>Передача типизированного результата в диалог

Изучите код в файле **luisbot.ts**. В обработчике `processActivity` бот передает в диалог типизированный результат.

```typescript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';

        // Create dialog context 
        const state = conversationState.get(context);
        const dc = dialogs.createContext(context, state);
            
        if (!isMessage) {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }

        // Check to see if anyone replied. 
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {

                
                await luisRec.recognize(context).then(async (res : any) => 
                {    
                    var typedresult = res as CafeLUISModel;                
                    let topIntent = LuisRecognizer.topIntent(res);    
                    switch (topIntent)
                    {
                        case Intents.Book_Table: {
                            await context.sendActivity("Top intent is Book_Table ");                          
                            await dc.begin('reserveTable', typedresult);
                            break;
                        }
                        
                        case Intents.Greeting: {
                            await context.sendActivity("Top intent is Greeting");
                            break;
                        }
    
                        case Intents.Who_are_you_intent: {
                            await context.sendActivity("Top intent is Who_are_you_intent");
                            break;
                        }
                        default: {
                            await dc.begin('default', topIntent);
                            break;
                        }
                    }
    
                }, (err) => {
                    // there was some error
                    console.log(err);
                }
                );                                
            }
        }
    });
});
```

## <a name="check-for-existing-entities-in-a-dialog"></a>Поиск существующих сущностей в диалоге

В файле **luisbot.ts** диалог `reserveTable` вызывает вспомогательную функцию `SaveEntities` для проверки сущностей, обнаруженных приложением LUIS. Каждая обнаруженная сущность сохраняется в состоянии диалога. Каждый шаг каскадной последовательности проверяет, была ли сохранена некоторая сущность в состояние диалога. Если сущность не обнаружена, выдается запрос для ее ввода.

```typescript
dialogs.add('reserveTable', [
    async function(dc, args, next){
        var typedresult = args as CafeLUISModel;

        // Call a helper function to save the entities in the LUIS result
        // to dialog state
        await SaveEntities(dc, typedresult);

        await dc.context.sendActivity("Welcome to the reservation service.");
        
        if (dc.activeDialog.state.dateTime) {
            await next();     
        }
        else {
            await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.dateTime) {
            // Save the dateTimePrompt result to dialog state
            dc.activeDialog.state.dateTime = result[0].value;
        }

        // If we don't have party size, ask for it next
        if (!dc.activeDialog.state.partySize) {
            await dc.prompt('textPrompt', "How many people are in your party?");
        } else {
            await next();
        }
    },
    async function(dc, result, next){
        if (!dc.activeDialog.state.partySize) {
            dc.activeDialog.state.partySize = result;
        }
        // Ask for the reservation name next
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.Name = result;

        // Save data to conversation state
        var state = conversationState.get(dc.context);

        // Copy the dialog state to the conversation state
        state = dc.activeDialog.state;

        // TODO: Add in <br/>Location: ${state.cafeLocation}
        var msg = `Reservation confirmed. Reservation details:             
            <br/>Date/Time: ${state.dateTime} 
            <br/>Party size: ${state.partySize} 
            <br/>Reservation name: ${state.Name}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

Вспомогательная функция `SaveEntities` проверяет наличие сущностей `datetime` и `partysize`. Сущность `datetime` является [предварительно созданной сущностью](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2).

```typescript
// Helper function that saves any entities found in the LUIS result
// to the dialog state
async function SaveEntities( dc: DialogContext<TurnContext>, typedresult) {
    // Resolve entities returned from LUIS, and save these to state
    if (typedresult.entities)
    {
        console.log(`Entities found.`);
        let datetime = typedresult.entities.datetime;

        if (datetime) {
            // Use the first date or time found in the utterance
            if (datetime[0].timex) {
                timexValues = datetime[0].timex;
                // Datetime results from LUIS are represented in timex format:
                // http://www.timeml.org/publications/timeMLdocs/timeml_1.2.1.html#timex3                                
                // More information on the library which does the recognition can be found here: 
                // https://github.com/Microsoft/Recognizers-Text

                if (datetime[0].type === "datetime") {
                    // To parse timex, here you use the resolve and creator from
                    // @microsoft/recognizers-text-data-types-timex-expression
                    // The second parameter is an array of constraints the results must satisfy
                    var resolution = Resolver.evaluate(
                        // array of timex values to evaluate. There may be more than one: "today at 6" can be 6AM or 6PM.
                        timexValues,
                        // constrain results to times between 4pm and 8pm                        
                        [Creator.evening]);
                    if (resolution[0]) {
                        // toNaturalLanguage takes the current date into account to create a friendly string
                        dc.activeDialog.state.dateTime = resolution[0].toNaturalLanguage(new Date());
                        // You can also use resolution.toString() to format the date/time.
                    } else {
                        // time didn't satisfy constraint.
                        dc.activeDialog.state.dateTime = null;
                    }
                } 
                else  {
                    console.log(`Type ${datetime[0].type} is not yet supported. Provide both the date and the time.`);
                }
            }                                                
        }
        let partysize = typedresult.entities.partySize;
        if (partysize) {
            console.log(`partysize entity detected.${partysize}`);
            // use first partySize entity that was found in utterance
            dc.activeDialog.state.partySize = partysize[0];
        }
        let cafelocation = typedresult.entities.cafeLocation;

        if (cafelocation) {
            console.log(`location entity detected.${cafelocation}`);
            // use first cafeLocation entity that was found in utterance
            dc.activeDialog.state.cafeLocation = cafelocation[0][0];
        }
    } 
}
```

## <a name="run-the-sample"></a>Запуск примера

1. Если у вас не установлен компилятор TypeScript, установите его сейчас с помощью следующей команды:

```
npm install --global typescript
```

2. Перед запуском бота установите зависимости, выполнив команду `npm install` в корневом каталоге нашего примера:

```
npm install
```

3. В корневом каталоге запустите сборку примера с помощью `tsc`. Это действие создаст файл `luisbot.js`.

4. Запустите `luisbot.js` в каталоге `lib`.

5. Выполните собранный пример через [эмулятор Bot Framework](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator).

6. Произнесите команду `reserve a table`, чтобы начать диалог для резервирования.

![Запуск бота](media/how-to-luisgen/run-bot.png)

---


## <a name="additional-resources"></a>Дополнительные ресурсы

См. дополнительные сведения об [Интеллектуальной службе распознавания речи](./bot-builder-concept-luis.md).


## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Combine LUIS apps and QnA services using the Dispatch tool](./bot-builder-tutorial-dispatch.md) (Объединение приложений LUIS и служб QnA с помощью средства подготовки к отправке)



---
title: Управление ходом разговора с помощью диалогов | Документы Майкрософт
description: Сведения об управлении ходом разговора с помощью диалогов в пакете SDK Bot Builder для Node.js.
keywords: ход разговора, диалоги, запросы, каскады, набор диалогов
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/8/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 99184ba71072c159c598c7f68289c42a51926795
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305263"
---
# <a name="manage-conversation-flow-with-dialogs"></a>Управление ходом разговора с помощью диалогов
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]


Управление ходом разговора является важнейшей задачей при разработке ботов. При использовании пакета SDK Bot Builder вы можете управлять ходом разговора с помощью **диалогов**.

Диалог напоминает функцию в программе. Обычно он предназначен для выполнения определенной операции и может вызываться так часто, как это необходимо. Вы можете объединить несколько диалогов в цепочку, чтобы поддержать практически любой разговор. Библиотека **диалогов** в пакете SDK Bot Builder включает встроенные компоненты, такие как **запросы** и **каскады**, которые помогают вам управлять ходом разговора с помощью диалогов. Библиотека запросов предоставляет различные запросы, с помощью которых можно запрашивать у пользователей различную информацию. Каскады позволяют объединить несколько шагов в последовательности.

В этой статье показано, как создать объект диалога и как добавить шаги для запросов и каскада в набор диалогов, чтобы управлять как простыми, так и сложными разговорами. 

## <a name="install-the-dialogs-library"></a>Установка библиотеки диалогов

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
Для использования диалогов установите пакет NuGet `Microsoft.Bot.Builder.Dialogs` для своего проекта или решения.
Затем добавьте ссылку на библиотеку диалогов в файлы кода с помощью инструкций using. Например: 

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
Библиотеку `botbuilder-dialogs` можно скачать из NPM. Чтобы установить библиотеку `botbuilder-dialogs` выполните следующую команду NPM:

```cmd
npm install --save botbuilder-dialogs
```

Чтобы использовать **диалоги** в своем боте, включите эту библиотеку в код бота. Например: 

**app.js**

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```
---

## <a name="create-a-dialog-stack"></a>Создание стека диалога

Чтобы использовать диалоги, необходимо создать *набор диалогов*.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Библиотека `Microsoft.Bot.Builder.Dialogs` предоставляет класс `DialogSet`.
В набор диалогов можно добавить именованные диалоги и наборы диалогов и затем обращаться к ним по имени.

```csharp
IDialog dialog = null;
// Initialize dialog.

DialogSet dialogs = new DialogSet();
dialogs.Add("dialog name", dialog);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Библиотека `botbuilder-dialogs` предоставляет класс `DialogSet`.
Класс **DialogSet** определяет **стек диалогов** и предоставляет простой интерфейс для управления стеком.
Пакет SDK Bot Builder реализует стек в качестве массива.

Чтобы создать объект **DialogSet**, выполните следующие действия:

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

Приведенный выше вызов создает объект **DialogSet** со **стеком диалога** по умолчанию с именем `dialogStack`.
Если вы хотите указать имя стека, можно передать его в качестве параметра метода **DialogSet()**. Например: 

```javascript
const dialogs = new botbuilder_dialogs.DialogSet("myStack");
```
---

При создании диалога происходит только добавление определения диалога в набор диалогов. Диалог не запускается, пока вы не отправите его в стек диалога, вызвав метод _begin_ или _replace_. 

Имя диалога (например, `addTwoNumbers`) должно быть уникальным в пределах каждого набора диалогов. В каждом наборе диалогов можно определить любое количество диалогов.

В библиотеке диалогов определены следующие диалоги:
-   Диалог **запроса**, в котором используются по крайней мере две функции: одна функция запрашивает входные данные у пользователя и вторая обрабатывает входные данные.
    Эти две функции можно объединить, воспользовавшись моделью **каскада**.
-   Диалог **каскада** определяет последовательность _действий каскада_, которые выполняются по порядку.
    Диалог каскада может содержать одно действие, в этом случае его можно считать простым диалогом с одним действием.

## <a name="create-a-single-step-dialog"></a>Создание диалога с одним действием

Диалоги с одним действием могут использоваться для разговоров с одним оборотом. В этом примере создается бот, который может обнаружить, что пользователь ввел строку вида "1 + 2", и запустить диалог `addTwoNumbers` для ответа "1 + 2 = 3". 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Значения передаются в диалоги и возвращаются из диалогов в форме контейнеров свойств `IDictionary<string,object>`.

Чтобы создать простой диалог в наборе диалогов, используйте метод `Add`. В следующем коде добавляется каскад с одним действием с именем `addTwoNumbers`.

Это действие предполагает, что передаваемые аргументы диалога содержат свойства `first` и `second`, которые представляют числа, которые нужно сложить.

Начните с шаблона EchoBot. Затем добавьте соответствующий код в класс бота, чтобы добавить диалог в конструктор.
```csharp
public class EchoBot : IBot
{
    private DialogSet _dialogs;

    public EchoBot()
    {
        _dialogs = new DialogSet();
        _dialogs.Add("addTwoNumbers", new WaterfallStep[]
        {              
            async (dc, args, next) =>
            {
                double sum = (double)args["first"] + (double)args["second"];
                await dc.Context.SendActivity($"{args["first"]} + {args["second"]} = {sum}");
                await dc.End();
            }
        });
    }

    // The rest of the class definition is omitted here but would include OnTurn()
}

```

### <a name="pass-arguments-to-the-dialog"></a>Передача аргументов для диалога

Для вызова диалога из метода `OnTurn` бота, измените метод `OnTurn` следующим образом:
```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // create a dialog context
        var dialogCtx = _dialogs.CreateContext(context, state);

        // Bump the turn count. 
        state.TurnCount++;

        await dialogCtx.Continue();
        if (!context.Responded)
        {
            // Call a helper function that identifies if the user says something 
            // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add            
            if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
            { 
                var dialogArgs = new Dictionary<string, object>
                {
                    ["first"] = first,
                    ["second"] = second
                };                        
                await dialogCtx.Begin("addTwoNumbers", dialogArgs);
            }
            else
            {
                // Echo back to the user whatever they typed.
                await context.SendActivity($"Turn: {state.TurnCount}. You said '{context.Activity.Text}'");
            }
        }
    }
}
```

Добавьте вспомогательную функцию в класс бота. Вспомогательная функция использует простое регулярное выражение, чтобы определить, является ли сообщение пользователя запросом на сложение двух чисел.

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number, 
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7. 
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";
    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";
    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;
    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);
    var succeeded = false;
    first = 0;
    second = 0;
    if (matches.Count == 0)
    {
        succeeded = false;
    }
    else
    {
        var matched = matches[0];
        if ( System.Double.TryParse(matched.Groups[1].Value, out first) 
            && System.Double.TryParse(matched.Groups[2].Value, out second))
        {
            succeeded = true;
        } 
    }
    return succeeded;
}
```

Если вы используете шаблон EchoBot, измените класс `EchoState` в файле **EchoState.cs** следующим образом:

```cs
/// <summary>
/// Class for storing conversation state.
/// This bot only stores the turn count in order to echo it to the user
/// </summary>
public class EchoState: Dictionary<string, object>
{
    private const string TurnCountKey = "TurnCount";
    public EchoState()
    {
        this[TurnCountKey] = 0;            
    }

    public int TurnCount
    {
        get { return (int)this[TurnCountKey]; }
        set { this[TurnCountKey] = value; }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Начните с шаблона JS, описанного в разделе [Создание бота с помощью пакета SDK Bot Builder версии 4](../javascript/bot-builder-javascript-quickstart.md). В **app.js** добавьте инструкцию для включения `botbuilder-dialogs`.
```js
const {DialogSet} = require('botbuilder-dialogs');
```

В **app.js** добавьте следующий код, который определяет простой диалог с именем `addTwoNumbers`, относящийся к набору `dialogs`:

```javascript
const dialogs = new DialogSet("myDialogStack");

// Show the sum of two numbers.
dialogs.add('addTwoNumbers', [async function (dc, numbers){
        var sum = Number.parseFloat(numbers[0]) + Number.parseFloat(numbers[1]);
        await dc.context.sendActivity(`${numbers[0]} + ${numbers[1]} = ${sum}`);
        await dc.end();
    }]
);
```

Замените код для обработки входящих действий в файле **app.js** на следующий. Бот вызывает вспомогательную функцию, чтобы проверить, является ли входящее сообщение запросом на сложение двух чисел. Если это так, он передает числа в аргументах в метод `DialogContext.begin`.

```js
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = context.activity.type === 'message';
        if (isMessage) {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // create a dialog context
            const dc = dialogs.createContext(context, state);

            // MatchesAdd2Numbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await MatchesAdd2Numbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {    
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`); 
            }           
        } else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "what's 1+2?"`);
            }
        }
    });
});
```

Добавьте вспомогательную функцию в файл **app.js**. Вспомогательная функция использует регулярное выражение, чтобы определить, является ли сообщение пользователя запросом на сложение двух чисел. Если сообщение соответствует регулярному выражению, возвращается массив, содержащий числа, которые нужно сложить.

```javascript
async function MatchesAdd2Numbers(message) {
    const ADD_NUMBERS_REGEXP = /([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))(?:\s*)\+(?:\s*)([-+]?(?:[0-9]+(?:\.[0-9]+)?|\.[0-9]+))/i;
    let matched = ADD_NUMBERS_REGEXP.exec(message);
    if (!matched) {
        // message wasn't a request to add 2 numbers
        return null;
    }
    else {
        var numbers = [matched[1], matched[2]];
        return numbers;
    }
}
```

---

### <a name="run-the-bot"></a>Запуск бота

Попробуйте запустить бот в эмуляторе Bot Framework и произнесите что-нибудь вроде: "Сколько будет 1+1?".

![Запуск бота](./media/how-to-dialogs/bot-output-add-numbers.png)



## <a name="using-dialogs-to-guide-the-user-through-steps"></a>Использование диалогов для продвижения пользователя по действиям

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="create-a-composite-dialog"></a>Создание составного диалога

Следующие фрагменты кода взяты из примера кода [Microsoft.Bot.Samples.Dialog.Prompts](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/MIcrosoft.Bot.Samples.Dialog.Prompts) в репозитории botbuilder-dotnet.

В файле Startup.cs:
1.  Переименуйте бот в `DialogContainerBot`.
1.  Используйте простой словарь как контейнер свойств для состояния разговора с ботом.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<DialogContainerBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new ConversationState<Dictionary<string, object>>(new MemoryStorage()));
    });
}
```

Переименуйте `EchoBot` в `DialogContainerBot`.

В `DialogContainerBot.cs` определите класс для диалога профиля.

```csharp
public class ProfileControl : DialogContainer
{
    public ProfileControl()
        : base("fillProfile")
    {
        Dialogs.Add("fillProfile", 
            new WaterfallStep[]
            {
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State = new Dictionary<string, object>();
                    await dc.Prompt("textPrompt", "What's your name?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["name"] = args["Value"];
                    await dc.Prompt("textPrompt", "What's your phone number?");
                },
                async (dc, args, next) =>
                {
                    dc.ActiveDialog.State["phone"] = args["Value"];
                    await dc.End(dc.ActiveDialog.State);
                }
            }
        );
        Dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
    }
}
```

Затем в определении бота объявите поле для главного диалога бота и инициализируйте это поле в конструкторе бота.
Главный диалог бота включает диалог профиля.

```csharp
private DialogSet _dialogs;

public DialogContainerBot()
{
    _dialogs = new DialogSet();

    _dialogs.Add("getProfile", new ProfileControl());
    _dialogs.Add("firstRun",
        new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                    await dc.Context.SendActivity("Welcome! We need to ask a few questions to get started.");
                    await dc.Begin("getProfile");
            },
            async (dc, args, next) =>
            {
                await dc.Context.SendActivity($"Thanks {args["name"]} I have your phone number as {args["phone"]}!");
                await dc.End();
            }
        }
    );
}
```

В методе `OnTurn` бота:
-   Поприветствуйте пользователя в начале разговора.
-   Инициализируйте и _продолжайте_ главный диалог каждый раз при получении сообщения от пользователя.

    Если диалог не привел к получению ответа, предположите, что он завершился раньше или еще не начался, и _начните_ его, указав имя диалога в наборе, с которого должен начаться разговор.

```csharp
public async Task OnTurn(ITurnContext turnContext)
{
    try
    {
        switch (turnContext.Activity.Type)
        {
            case ActivityTypes.ConversationUpdate:
                foreach (var newMember in turnContext.Activity.MembersAdded)
                {
                    if (newMember.Id != turnContext.Activity.Recipient.Id)
                    {
                        await turnContext.SendActivity("Hello and welcome to the Composite Control bot.");
                    }
                }
                break;

            case ActivityTypes.Message:
                var state = ConversationState<Dictionary<string, object>>.Get(turnContext);
                var dc = _dialogs.CreateContext(turnContext, state);

                await dc.Continue();

                if (!turnContext.Responded)
                {
                    await dc.Begin("firstRun");
                }

                break;
        }
    }
    catch (Exception e)
    {
        await turnContext.SendActivity($"Exception: {e.Message}");
    }
}

```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="create-a-dialog-with-waterfall-steps"></a>Создание диалога с действиями каскада

Диалог — это последовательность сообщений, передаваемых между пользователем и ботом. Если цель бота состоит в том, чтобы провести пользователя через последовательность действий, вы можете использовать **каскад**, чтобы определить шаги разговора.

**Каскад** является конкретной реализацией диалога, которая обычно используется для сбора информации от пользователя или для предоставления пользователю инструкций по выполнению ряда задач. Задачи реализуются как массив функций, в котором результаты первой функции передаются в качестве входных данных в следующую функцию и так далее. Каждая функция обычно представляет один шаг в общем процессе. На каждом шаге бот [запрашивает у пользователя входные данные](bot-builder-prompts.md), ожидает ответа, а затем передает результат в следующий шаг.

Например, в следующем коде определяются три функции в массиве, которые соответствуют трем действиям **каскада**:

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, results){
        var workPlace = results;
        await dc.context.sendActivity(`${workPlace} is a fun place.`);
        await dc.end(); // Ends the dialog
    }
]);

dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```

Сигнатура для действия **каскада** выглядит следующим образом:

| Параметр | ОПИСАНИЕ |
| ---- | ----- |
| `context` | Контекст диалога. |
| `args` | Необязателен. Содержит аргументы, передаваемые в действие. |
| `next` | Необязателен. Метод, который позволяет перейти к следующему действию каскада. При вызове этого метода можно указать аргумент *args*, что позволяет передать аргументы в следующее действие каскада. |

В каждом из действий перед возвратом необходимо вызывать один из следующих методов: *next()*, *dialogs.prompt()*, *dialogs.end()*, *dialogs.begin()* или *Promise.resolve()*, в противном случае бот остановится на этом шаге. То есть если функция не возвращает один из этих методов, то при любых данных, введенных пользователем, действие будет выполняться повторно каждый раз, когда пользователь отправляет сообщение боту.

При достижении конца каскада рекомендуется выполнить возврат с помощью метода `end()`, чтобы можно было извлечь диалог из стека. Дополнительные сведения см. в разделе [Завершение диалога](#end-a-dialog). Аналогично, чтобы перейти от одного действия к другому, в конце действия каскада необходимо использовать запрос или явный вызов функции `next()`. 

### <a name="start-a-dialog"></a>Начало диалога

Чтобы начать диалог, передайте идентификатор диалога *dialogId*, который вы хотите запустить, в метод `begin()`, `prompt()` или `replace()`. Метод **begin** отправит диалог на вершину стека, а метод **replace** извлечет текущий диалог из стека и поместит новый диалог в стек.

Чтобы запустить диалог без аргументов, сделайте следующее:

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

Чтобы запустить диалог с аргументами, сделайте следующее:

```javascript
// Start the 'greetings' dialog with the 'userName' passed in. 
await dc.begin('greetings', userName);
```

Чтобы запустить диалог **запроса**, сделайте следующее:

```javascript
// Start a 'choicePrompt' dialog with choices passed in as an array of colors to choose from.
await dc.prompt('choicePrompt', `choice: select a color`, ['red', 'green', 'blue']);
```

В зависимости от типа запроса, который вы запускаете, сигнатура аргумента запроса может различаться. Метод **DialogSet.prompt** является вспомогательным. Этот метод принимает аргументы и создает соответствующие параметры запроса; затем он вызывает метод **begin**, чтобы запустить диалог запроса.

Чтобы заменить диалог в стеке, сделайте следующее:

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); // Can optionally passed in an 'args' as the second argument.
```

Дополнительные сведения об использовании метода **replace()** см. в разделах [Повтор диалога](#repeat-a-dialog) и [Циклы диалогов](#dialog-loops) ниже.

## <a name="end-a-dialog"></a>Завершение диалога

Для завершения диалога необходимо извлечь его из стека и при желании возвратить результат в родительский диалог. Для каждого возвращенного результата будет вызван метод **Dialog.resume()** родительского диалога.

Лучше всего явно вызывать метод `end()` в конце диалога; однако это необязательно, так как диалог будет автоматически извлечен из стека при достижении конца каскада.

Чтобы завершить диалог, сделайте следующее:

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

Чтобы завершить диалог с передачей необязательных аргументов в родительский диалог, сделайте следующее:

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end(result);
```

Вы также можете завершить диалог, возвратив разрешенное обещание:

```javascript
await Promise.resolve();
```

Вызов метода `Promise.resolve()` приведет к тому, что диалог завершится и будет извлечен из стека. Однако этот метод не вызывает родительское диалоговое окно, чтобы возобновить выполнение. После вызова метода `Promise.resolve()` выполнение останавливается, и бот возобновляет работу с той точки, где остановил работу родительский диалог, когда пользователь отправил сообщение боту. Эта схема может быть не самой удобной для завершения диалога с точки зрения пользователя. Старайтесь завершать диалог с помощью вызова метода `end()` или `replace()`, чтобы бот мог продолжить взаимодействие с пользователем.

### <a name="clear-the-dialog-stack"></a>Очистка стека диалогов

Если вы хотите извлечь все диалоги из стека, можно очистить стек диалогов, вызвав метод `dc.endAll()`.

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

### <a name="repeat-a-dialog"></a>Повтор диалога

Чтобы повторить диалог, используйте метод `dialogs.replace()`.

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu'); 
```

Если вы хотите отображать главное меню по умолчанию, вы можете создать диалог `mainMenu`, выполнив следующие действия:

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected.
dialogs.add('mainMenu', [
    async function(dc){
        await dc.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function(dc, result){
        if(result.value.match(/order dinner/ig)){
            await dc.begin('orderDinner');
        }
        else if(result.value.match(/reserve a table/ig)){
            await dc.begin('reserveTable');
        }
        else {
            // Repeat the menu
            await dc.replace('mainMenu');
        }
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);

dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
```

В этом диалоге используется метод `ChoicePrompt` для отображения меню, затем пользователь должен выбрать пункт меню. Когда пользователь выбирает `Order Dinner` или `Reserve a table`, запускается диалог для соответствующего варианта, и после завершения задачи диалог не завершается на последнем шаге, а запускается повторно.

### <a name="dialog-loops"></a>Циклы диалогов

Еще один способ использования метода `replace()` — имитация цикла. Рассмотрим следующий сценарий в качестве примера. Если вы хотите разрешить пользователю добавить несколько элементов меню в корзину, можно использовать цикл для элементов меню, который будет действовать до тех пор, пока пользователь не завершит заказ.

```javascript
// Order dinner:
// Help user order dinner from a menu

var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
        "More info", "Process order", "Cancel", "Help"],
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50": {
        Description: "Clam Chowder",
        Price: 4.50
    }

}

// The order cart
var orderCart = {
    orders: [],
    total: 0,
    clear: function(dc) {
        this.orders = [];
        this.total = 0;
        dc.context.activity.conversation.orderCart = null;
    }
};

dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        orderCart.clear(dc); // Clears the cart.

        await dc.begin('orderPrompt'); // Prompt for orders
    },
    async function (dc, result) {
        if(result == "Cancel"){
            await dc.end();
        }
        else { 
            await dc.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function(dc, result){
        await dc.context.sendActivity(`Thank you. Your order will be delivered to room ${result} within 45 minutes.`);
        await dc.end();
    }
]);

// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                // ...
                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            orderCart.clear(context);
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else if(choice.value.match(/more info/ig)){
            var msg = "More info: <br/>Potato Salad: contains 330 calaries per serving. <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. <br/>" 
                + "Clam Chowder: contains 650 calaries per serving."
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else if(choice.value.match(/help/ig)){
            var msg = `Help: <br/>To make an order, add as many items to your cart as you like then choose the "Process order" option to check out.`
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else {
            var choice = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!choice){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                orderCart.orders.push(choice);
                orderCart.total += dinnerMenu[choice.value].Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt');
            }
        }
    }
]);

// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());

```

В приведенном выше примере кода основной диалог `orderDinner` использует вспомогательный диалог с именем `orderPrompt` для обработки выбираемых пользователем вариантов. Диалог `orderPrompt` отображает меню, просит пользователя выбрать элемент, добавляет элемент в корзину и просит выбрать элемент еще раз. Это позволяет пользователю добавить несколько элементов в заказ. Цикл в диалоге продолжает работать, пока пользователь не выберет `Process order` или `Cancel`. В этот момент выполнение возвращается в родительский диалог (например: `orderDinner`). Диалог `orderDinner` выполняет последние необходимые действия в случае, если пользователь хочет обработать заказ; в противном случае он завершается и возвращает выполнение в родительский диалог (например: `mainMenu`). Диалог `mainMenu`, в свою очередь, выполняет последнее действие, которое представляет собой повторное отображение элементов главного меню.

---

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знаете, как использовать **диалоги**, **запросы** и **каскады** для управления ходом разговора, посмотрим, как разбить диалоги на модульные задачи вместо того, чтобы собирать их вместе в объекте `dialogs` в логике главного бота.

> [!div class="nextstepaction"]
> [Создание модульной логики бота с помощью составных элементов управления](bot-builder-compositcontrol.md)
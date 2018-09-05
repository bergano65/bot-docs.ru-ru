---
title: Управление простым процессом общения с помощью диалогов | Документация Майкрософт
description: Сведения о том, как управлять простым процессом общения с помощью диалогов в пакете SDK Bot Builder для Node.js.
keywords: simple conversation flow, dialogs, prompts, waterfalls, dialog set
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 8/2/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 77162601f542e6faa8908bc71abc971eb99fcc93
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/21/2018
ms.locfileid: "42756471"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>Управление простым процессом общения с помощью диалогов

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

С помощью библиотеки диалогов вы можете управлять простыми и сложными процессами общения. В рамках простого процесса общения пользователь выполняет все действия в *каскадной последовательности*, после чего обмен сообщениями прекращается. Диалоги также могут представлять собой [сложный процесс общения](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md), части которого могут разветвляться и образовывать цикл.

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

<!-- TODO: This paragraph belongs in a conceptual topic. -->Диалог напоминает функцию в программе. Обычно он предназначен для выполнения определенной операции в определенном порядке и может вызываться так часто, как это необходимо. Используя диалоги, разработчик бота может управлять процессом общения. Чтобы поддержать практически любое общение можно объединить несколько диалогов в цепочку. Библиотека **диалогов** в пакете SDK Bot Builder включает встроенные компоненты, такие как _запросы_ и _диалоги с каскадной последовательностью действий_, которые помогают управлять процессом общения. С помощью запросов можно запрашивать у пользователей различные типы данных. Вы можете использовать каскад, чтобы объединить несколько шагов в последовательность.

В рамках этой статьи мы используем _наборы диалогов_ для создания процесса общения, содержащего запросы и каскадную последовательность действий. У нас есть два примера диалогов. Первый — это диалог с одним действием, в котором выполняется операция, не требующая ввода данных пользователем. Второй — диалог с несколькими действиями, в котором пользователю предлагается ввести некоторые сведения.

## <a name="install-the-dialogs-library"></a>Установка библиотеки диалогов

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Мы начнем с базового шаблона EchoBot. Инструкции см. в [кратком руководстве по .NET](~/dotnet/bot-builder-dotnet-quickstart.md).

Для использования диалогов установите пакет NuGet `Microsoft.Bot.Builder.Dialogs` для своего проекта или решения.
Затем при необходимости добавьте ссылку на библиотеку диалогов в файлы кода с помощью инструкций using.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Библиотеку `botbuilder-dialogs` можно скачать из NPM. Чтобы установить библиотеку `botbuilder-dialogs` выполните следующую команду NPM:

```cmd
npm install --save botbuilder-dialogs@preview
```

Чтобы использовать этот **диалог** в боте, вставьте следующий код в файл **app.js**.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

## <a name="create-a-dialog-stack"></a>Создание стека диалога

В первом примере мы создадим диалог с одним действием, в котором можно сложить два числа и вывести результат.

Чтобы использовать диалоги, необходимо создать *набор диалогов*.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Библиотека `Microsoft.Bot.Builder.Dialogs` предоставляет класс `DialogSet`.
Создайте класс **AdditionDialog** и добавьте инструкции using, которые нам потребуются.
В набор диалогов можно добавить именованные диалоги и наборы диалогов и затем обращаться к ним по имени.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

Создайте производный от **DialogSet** класс и определите идентификаторы и ключи, которые потребуются нам для идентификации диалогов и входных данных для этого набора диалогов.

```csharp
/// <summary>Defines a simple dialog for adding two numbers together.</summary>
public class AdditionDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Main = "additionDialog";

    /// <summary>Defines the IDs of the input arguments.</summary>
    public struct Inputs
    {
        public const string First = "first";
        public const string Second = "second";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

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

Имя диалога (например, `addTwoNumbers`) должно быть уникальным в пределах каждого набора диалогов. В каждом наборе диалогов можно определить любое количество диалогов. Сведения о том, как создать несколько наборов диалогов и обеспечить их эффективную совместную работу, см. в руководстве по [созданию логики модульного бота](bot-builder-compositcontrol.md).

В библиотеке диалогов определены следующие диалоги:

* Диалог **запроса**, в котором используются по крайней мере две функции: одна функция запрашивает входные данные у пользователя и вторая обрабатывает входные данные. Эти две функции можно объединить, воспользовавшись моделью **каскада**.
* Диалог **каскада** определяет последовательность _действий каскада_, которые выполняются по порядку. Диалог каскада может содержать одно действие, в этом случае его можно считать простым диалогом с одним действием.

## <a name="create-a-single-step-dialog"></a>Создание диалога с одним действием

Диалоги с одним действием могут использоваться для разговоров с одним оборотом. В этом примере создается бот, который может обнаружить, что пользователь ввел строку вида "1 + 2", и запустить диалог `addTwoNumbers` для ответа "1 + 2 = 3".

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Значения передаются в диалоги и возвращаются из диалогов в форме контейнеров свойств `IDictionary<string,object>`.

Чтобы создать простой диалог в наборе диалогов, используйте метод `Add`. В следующем коде добавляется каскад с одним действием с именем `addTwoNumbers`.

Это действие предполагает, что передаваемые аргументы диалога содержат свойства `first` и `second`, которые представляют числа, которые нужно сложить.

Добавьте следующий конструктор в класс **AdditionDialog**.

```csharp
/// <summary>Defines the steps of the dialog.</summary>
public AdditionDialog()
{
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Get the input from the arguments to the dialog and add them.
            var x =(double)args[Inputs.First];
            var y =(double)args[Inputs.Second];
            var sum = x + y;

            // Display the result to the user.
            await dc.Context.SendActivity($"{x} + {y} = {sum}");

            // End the dialog.
            await dc.End();
        }
    });
}
```

### <a name="pass-arguments-to-the-dialog"></a>Передача аргументов для диалога

В коде бота обновите инструкции using.

```cs
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Text.RegularExpressions;
using System.Threading.Tasks;
```

Добавьте статическое свойство в класс для дополнительного диалога.

```cs
private static AdditionDialog AddTwoNumbers { get; } = new AdditionDialog();
```

Для вызова диалога из метода `OnTurn` бота, измените метод `OnTurn` следующим образом:

```cs
public async Task OnTurn(ITurnContext context)
{
    // Handle any message activity from the user.
    if (context.Activity.Type is ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var conversationState = context.GetConversationState<ConversationData>();

        // Generate a dialog context for the addition dialog.
        var dc = AddTwoNumbers.CreateContext(context, conversationState.DialogState);

        // Call a helper function that identifies if the user says something
        // like "2 + 3" or "1.25 + 3.28" and extract the numbers to add.
        if (TryParseAddingTwoNumbers(context.Activity.Text, out double first, out double second))
        {
            // Start the dialog, passing in the numbers to add.
            var args = new Dictionary<string, object>
            {
                [AdditionDialog.Inputs.First] = first,
                [AdditionDialog.Inputs.Second] = second
            };
            await dc.Begin(AdditionDialog.Main, args);
        }
        else
        {
            // Echo back to the user whatever they typed.
            await context.SendActivity($"You said '{context.Activity.Text}'");
        }
    }
}
```

Добавьте вспомогательную функцию **TryParseAddingTwoNumbers** в класс бота. Вспомогательная функция использует простое регулярное выражение, чтобы определить, является ли сообщение пользователя запросом на сложение двух чисел.

```cs
// Recognizes if the message is a request to add 2 numbers, in the form: number + number,
// where number may have optionally have a decimal point.: 1 + 1, 123.99 + 45, 0.4+7.
// For the sake of simplicity it doesn't handle negative numbers or numbers like 1,000 that contain a comma.
// If you need more robust number recognition, try System.Recognizers.Text
public static bool TryParseAddingTwoNumbers(string message, out double first, out double second)
{
    // captures a number with optional -/+ and optional decimal portion
    const string NUMBER_REGEXP = "([-+]?(?:[0-9]+(?:\\.[0-9]+)?|\\.[0-9]+))";

    // matches the plus sign with optional spaces before and after it
    const string PLUSSIGN_REGEXP = "(?:\\s*)\\+(?:\\s*)";

    const string ADD_TWO_NUMBERS_REGEXP = NUMBER_REGEXP + PLUSSIGN_REGEXP + NUMBER_REGEXP;

    var regex = new Regex(ADD_TWO_NUMBERS_REGEXP);
    var matches = regex.Matches(message);

    first = 0;
    second = 0;
    if (matches.Count > 0)
    {
        var matched = matches[0];
        if (double.TryParse(matched.Groups[1].Value, out first)
            && double.TryParse(matched.Groups[2].Value, out second))
        {
            return true;
        }
    }
    return false;
}
```

Если вы используете шаблон EchoBot, измените имя класса **EchoState** на **ConversationData** и его содержимое, чтобы включить следующее:

```cs
using System.Collections.Generic;

/// <summary>
/// Class for storing conversation state.
/// </summary>
public class ConversationData
{
    /// <summary>Property for storing dialog state.</summary>
    public Dictionary<string, object> DialogState { get; set; } = new Dictionary<string, object>();
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

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
        // State will store all of your information
        const convoState = conversationState.get(context);
        const dc = dialogs.createContext(context, convoState);

        if (isMessage) {
            // TryParseAddingTwoNumbers checks if the message matches a regular expression
            // and if it does, returns an array of the numbers to add
            var numbers = await TryParseAddingTwoNumbers(context.activity.text); 
            if (numbers != null && numbers.length >=2 )
            {
                await dc.begin('addTwoNumbers', numbers);
            }
            else {
                // Just echo back the user's message if they're not adding numbers
                const count = (convoState.count === undefined ? convoState.count = 0 : ++convoState.count);
                return context.sendActivity(`Turn ${count}: You said "${context.activity.text}"`);
            }
        }
        else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }

        if (!context.responded) {
            await dc.continue();
            // if the dialog didn't send a response
            if (!context.responded && isMessage) {
                await dc.context.sendActivity(`Hi! I'm the add 2 numbers bot. Say something like "What's 2+3?"`);
            }
        }
    });
});

```

Добавьте вспомогательную функцию в файл **app.js**. Вспомогательная функция использует регулярное выражение, чтобы определить, является ли сообщение пользователя запросом на сложение двух чисел. Если сообщение соответствует регулярному выражению, возвращается массив, содержащий числа, которые нужно сложить.

```javascript
async function TryParseAddingTwoNumbers(message) {
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

В этом примере мы создадим диалог с несколькими действиями, чтобы запрашивать сведения у пользователя.

### <a name="create-a-dialog-with-waterfall-steps"></a>Создание диалога с действиями каскада

**Каскад** является конкретной реализацией диалога, которая обычно используется для сбора информации от пользователя или для предоставления пользователю инструкций по выполнению ряда задач. Задачи реализуются как массив функций, в котором результаты первой функции передаются в качестве входных данных в следующую функцию и так далее. Каждая функция обычно представляет один шаг в общем процессе. На каждом шаге бот [запрашивает у пользователя входные данные](bot-builder-prompts.md), ожидает ответа, а затем передает результат в следующий шаг.

Например, в следующем примере кода определяются три функции в массиве, которые соответствуют трем действиям **каскада**. После каждого запроса бот подтверждает введенные пользователем данные, но не сохраняет их. Если нужно сохранить введенные пользователем данные, ознакомьтесь со статьей [Хранение данных пользователя](bot-builder-tutorial-persist-user-inputs.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В примере ниже показан конструктор для диалога приветствия, где **GreetingDialog** является производным от **DialogSet**, **Inputs.Text** содержит идентификатор, используемый для объекта **TextPrompt**, а **Main** содержит идентификатор для самого диалога приветствия.

```csharp
public GreetingDialog()
{
    // Include a text prompt.
    Add(Inputs.Text, new TextPrompt());

    // Define the dialog logic for greeting the user.
    Add(Main, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Ask for their name.
            await dc.Prompt(Inputs.Text, "What is your name?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var name = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"Hi, {name}!");

            // Ask where they work.
            await dc.Prompt(Inputs.Text, "Where do you work?");
        },
        async (dc, args, next) =>
        {
            // Get the prompt result.
            var work = args["Text"] as string;

            // Acknowledge their input.
            await dc.Context.SendActivity($"{work} is a fun place.");

            // End the dialog.
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

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

---

Сигнатура для действия **каскада** выглядит следующим образом:

| Параметр | ОПИСАНИЕ |
| :---- | :----- |
| `dc` | Контекст диалога. |
| `args` | Необязательный. Содержит аргументы, передаваемые в действие. |
| `next` | Необязательный. Метод, который позволяет перейти к следующему действию каскада без запроса. При вызове этого метода можно указать аргумент параметра *args*. Таквы можете передавать аргументы в следующее действие каскада. |

В каждом действии перед возвратом необходимо вызывать делегат *next()* или один из методов контекста диалога: *begin*, *end*, *prompt* либо *replace*. В противном случае бот остановится на этом шаге. То есть, если в конце функции не указан ни один из этих методов, при любых данных, введенных пользователем, действие будет выполняться повторно каждый раз, когда пользователь отправляет сообщение боту.

При достижении конца каскада рекомендуется выполнить возврат с помощью метода _end_, чтобы можно было извлечь диалог из стека. Дополнительные сведения см. в разделе [Завершение диалога](#end-a-dialog) ниже. Аналогично, чтобы перейти от одного действия к другому, в конце действия каскада необходимо использовать запрос или явный вызов делегата _next_.

## <a name="start-a-dialog"></a>Начало диалога

Чтобы начать диалог, передайте *идентификатор нужного диалога* в метод контекста диалога _begin_, _prompt_ или _replace_. Метод _begin_ отправит диалог на вершину стека, а метод _replace_ извлечет текущий диалог из стека и поместит новый диалог в стек.

Чтобы запустить диалог без аргументов, сделайте следующее:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog.
await dc.Begin("greetings");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog.
await dc.begin('greetings');
```

---

Чтобы запустить диалог с аргументами, сделайте следующее:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Start the greetings dialog, passing in a property bag.
await dc.Begin("greetings", args);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Start the 'greetings' dialog with the 'userName' passed in.
await dc.begin('greetings', userName);
```

---

Чтобы запустить диалог **запроса**, сделайте следующее:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В этом примере **Inputs.Text** содержит идентификатор **TextPrompt** из того же набора диалогов.

```csharp
// Ask a user for their name.
await dc.Prompt(Inputs.Text, "What is your name?");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Ask a user for their name.
await dc.prompt('textPrompt', "What is your name?");
```

---

В зависимости от типа запроса, который вы запускаете, сигнатура аргумента запроса может различаться. Метод **DialogSet.prompt** является вспомогательным. Этот метод принимает аргументы и создает соответствующие параметры запроса; затем он вызывает метод **begin**, чтобы запустить диалог запроса. Дополнительные сведения о запросах см. в статье [Запрос пользователям на ввод данных](bot-builder-prompts.md).

## <a name="end-a-dialog"></a>Завершение диалога

Метод _end_ завершает диалог, извлекая его из стека и возвращает результат в родительский диалог, если необходимо.

Лучше всего явно вызвать метод _end_ в конце диалога. Но это необязательно, так как диалог будет автоматически извлечен из стека при достижении конца каскада.

Чтобы завершить диалог, сделайте следующее:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog by popping it off the stack.
await dc.End();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog by popping it off the stack
await dc.end();
```

---

Чтобы завершить диалог и возвратить сведения о родительском диалоге, добавьте аргумент контейнера свойств.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and return information to the parent dialog.
await dc.end(new Dictionary<string, object>
    {
        ["property1"] = value1,
        ["property2"] = value2
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and pass a result to the parent dialog
await dc.end({
    "property1": value1,
    "property2": value2
});
```

---

## <a name="clear-the-dialog-stack"></a>Очистка стека диалогов

Чтобы извлечь все диалоги из стека, можно очистить стек диалогов, вызвав метод _end all_.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Pop all dialogs from the current stack.
await dc.EndAll();
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Pop all dialogs from the current stack.
await dc.endAll();
```

---

## <a name="repeat-a-dialog"></a>Повтор диалога

Чтобы повторить диалог, используйте метод _replace_. Метод контекста диалога *replace* извлечет текущий диалог из стека, поместит новый диалог на вершину стека и начнет его. Это отличный способ для обработки [сложных диалогов](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md) и хороший метод для управления меню.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// End the current dialog and start the main menu dialog.
await dc.Replace("mainMenu");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// End the current dialog and start the 'mainMenu' dialog.
await dc.replace('mainMenu');
```

---

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы узнали, как управлять простым процессом общения, давайте рассмотрим, как использовать метод _replace_ для сложных процессов общения.

> [!div class="nextstepaction"]
> [Управление сложным процессом общения](bot-builder-dialog-manage-complex-conversation-flow.md)

---
title: Создание запросов на ввод данных пользователем с помощью библиотеки диалогов | Документация Майкрософт
description: Узнайте, как запросить у пользователя входные данные, используя библиотеку диалогов из пакета SDK Bot Builder для Node.js.
keywords: запросы, диалоговые окна, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, повторный запрос, проверка
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0b238ed510fd1d6fda82734af373f344b0dc28e3
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905368"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>Создание запросов на ввод данных пользователем с помощью библиотеки диалогов

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Часто боты собирают информацию с помощью заданных пользователю вопросов. Можно просто отправить пользователю стандартное сообщение, используя метод **send activity** для [контекста реплик](bot-builder-concept-activity-processing.md#turn-context), чтобы запросить ввод строки. Но пакет SDK для Bot Builder предоставляет библиотеку _диалогов_, которую можно использовать для запроса различных типов информации. В этом разделе описывается использование **запросов** на ввод данных пользователем.

В этой статье описывается использование запросов в диалоговом окне. Общие сведения об использовании диалогов см. в разделе [Управление ходом разговора с помощью диалогов](bot-builder-dialog-manage-conversation-flow.md).

## <a name="prompt-types"></a>Типы запросов

Библиотека диалоговых окон предлагает несколько различных типов запросов, каждый из которых запрашивает другой тип ответа.

| prompt | ОПИСАНИЕ |
|:----|:----|
| **AttachmentPrompt** | Запрос на добавление вложения пользователем, например документа или изображения. |
| **ChoicePrompt** | Запрос пользователю на выбор одного из нескольких вариантов. |
| **ConfirmPrompt** | Запрос на подтверждение действий пользователем. |
| **DatetimePrompt** | Запрос на ввод даты и времени пользователем. Пользователи могут отвечать, используя естественный язык, например "Tomorrow at 8pm" или "Friday at 10am". Пакет SDK для Bot Framework использует предварительно созданную сущность LUIS `builtin.datetimeV2`. Дополнительные сведения см. в [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2). |
| **NumberPrompt** | Запрос на ввод номера пользователем. Пользователь может ответить либо "10", либо "ten". Например, если ответ "ten", то запрос преобразует ответ в число и возвращает `10` в результате. |
| **TextPrompt** | Запрос на ввод текстовой строки пользователем. |

## <a name="add-references-to-prompt-library"></a>Добавление ссылок в библиотеку запросов

Получить библиотеку **диалоговых окон** можно, добавив пакет **диалоговых окон** к боту. Диалоги рассматриваются в статье об [использовании диалогов для управления простым процессом общения](bot-builder-dialog-manage-conversation-flow.md). Мы же будем использовать их для создания запросов.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Установите пакет **Microsoft.Bot.Builder.Dialogs** из NuGet.

Затем включите ссылку на библиотеку из кода бота.

```cs
using Microsoft.Bot.Builder.Dialogs;
```

Диалоговое окно можно определить как класс или строку в качестве свойства в файле кода бота.

Код в этой статье написан для диалогового окна, определенного как класс.
В следующих примерах предполагается, что код добавляется в конструктор диалогового окна.

Основной последовательностью диалогового окна является коллекция шагов и необходимость получения идентификатора. Бот использует этот идентификатор для извлечения диалогового окна, поэтому рекомендуется задать его как константу.

```cs
public class MyDialog : DialogSet
{
    public const string Name = "mainDialog";

    public MyDialog()
    {
        // Define your dialog's prompts and steps here.
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Установите пакет диалогов из NPM.

```cmd
npm install --save botbuilder-dialogs@preview
```

Чтобы использовать **диалоги** в боте, включите эту библиотеку в код бота.

В файле app.js добавьте следующее.

```javascript
const {DialogSet} = require("botbuilder-dialogs");
const dialogs = new DialogSet();
```

---

## <a name="prompt-the-user"></a>Запрос пользователю

Чтобы запросить пользователя ввести данные, можно добавить запрос в диалоговое окно. Например, можно определить запрос типа **TextPrompt** и присвоить ему идентификатор диалога **textPrompt**.

После добавления диалогового окна запроса можно использовать его в простом двухэтапном каскадном диалоге или использовать несколько запросов вместе в многоэтапном каскаде. *Каскадный* диалог — это просто способ определения последовательности шагов. Дополнительные сведения см. в разделе об [использовании диалогов](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps) статьи [Управление простым процессом общения с помощью диалогов](bot-builder-dialog-manage-conversation-flow.md).

На первом этапе диалоговое окно запрашивает у пользователя имя, а на втором обрабатывает введенные пользователем данные как ответ на запрос.

Например, в следующем диалоговом окне запрашивается имя пользователя, а затем пользователь приветствуется по имени.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Каждому запросу, используемому в диалоговом окне, также присваивается имя, которое используется диалогом или ботом для доступа к запросу. Во всех этих примерах мы предоставляем идентификаторы запросов в качестве констант.

Вызов метода **Prompt** или **End** контекста диалога сигнализирует о завершении шага в диалоговом окне. Без этих инструкций диалоговое окно не будет работать должным образом.

```csharp
/// <summary>Defines a simple greeting dialog that asks for the user's name.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            // Each step takes in a dialog context, arguments, and the next delegate.
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];
                await dc.Context.SendActivity($"Hi {user}!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {TextPrompt} = require("botbuilder-dialogs");
```

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('textPrompt', new TextPrompt());
dialogs.add('greetings', [
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);
        await dc.end();
    }
]);
```

---

> [!NOTE]
> Чтобы запустить диалоговое окно, получите контекст диалога и используйте метод _begin_. Дополнительные сведения см. в статье об [использовании диалогов для управления простым процессом общения](./bot-builder-dialog-manage-conversation-flow.md).

## <a name="reusable-prompts"></a>Многоразовые запросы

Запрос можно повторно использовать для получения различной информации, используя запрос того же типа. Например, приведенный выше пример кода определяет текстовый запрос и использует его для запроса имени пользователя. Если требуется, можно также использовать этот же запрос, чтобы запросить у пользователя другую текстовую строку, например "Where do you work?".

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>Defines a simple greeting dialog that asks for the user's name and place of work.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the text prompt.</summary>
        public const string Text = "textPrompt";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Text, new TextPrompt());
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the user's name.
                await dc.Prompt(Inputs.Text, "What is your name?");
            },
            async(dc, args, next) =>
            {
                var user = (string)args["Text"];

                // Ask them where they work.
                await dc.Prompt(Inputs.Text, $"Hi {user}! Where do you work?");
            },
            async(dc, args, next) =>
            {
                var workplace = (string)args["Text"];

                await dc.Context.SendActivity($"{workplace} is a cool place!");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, userName){
        await dc.context.sendActivity(`Hi ${userName}!`);

        // Ask them where they work.
        await dc.prompt('textPrompt', 'Where do you work?');
    },
    async function(dc, workPlace){
        await dc.context.sendActivity(`${workPlace} is a cool place!`);

        await dc.end();
    }
]);
```

---

Однако если вы хотите связать запрос с ожидаемым значением, которое он запрашивает, можно присвоить каждому запросу уникальный *dialogId*. Диалоговое окно добавляется с уникальным идентификатором. Используя разные идентификаторы, можно также создать несколько диалоговых окон **запроса** того же типа. Например, можно создать два диалоговых окна **TextPrompt** для приведенного выше примера.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
/// <summary>The ID of the main dialog in the set.</summary>
public const string Name = "mainDialog";

/// <summary>Defines the IDs of the prompts in the set.</summary>
public struct Inputs
{
    /// <summary>The ID of the name prompt.</summary>
    public const string Name = "namePrompt";

    /// <summary>The ID of the work prompt.</summary>
    public const string Work = "workPrompt";
}

/// <summary>Defines the prompts and steps of the dialog.</summary>
public MyDialog()
{
    Add(Inputs.Name, new TextPrompt());
    Add(Inputs.Work, new TextPrompt());
    Add(Name, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the user's name.
            await dc.Prompt(Inputs.Name, "What is your name?");
        },
        async(dc, args, next) =>
        {
            var user = (string)args["Text"];

            // Ask them where they work.
            await dc.Prompt(Inputs.Work, $"Hi {user}! Where do you work?");
        },
        async(dc, args, next) =>
        {
            var workplace = (string)args["Text"];

            await dc.Context.SendActivity($"{workplace} is a cool place!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
dialogs.add('namePrompt', new TextPrompt());
dialogs.add('workPlacePrompt', new TextPrompt());
```

---

Для повторного использования кода определение одного `textPrompt` будет работать для всех этих запросов, поскольку они запрашивают текстовую строку в качестве ответа. Однако при необходимости проверить входные данные запроса удобнее указывать диалоговые окна. В этом случае запросы могут использовать **TextPrompt**, но каждый ищет разный набор значений. Давайте рассмотрим, как можно проверять ответы на запрос с помощью `NumberPrompt`.

## <a name="specify-prompt-options"></a>Указание параметров запроса

При использовании запроса в шаге диалогового окна можно также предоставить параметры запросов, такие как строка повторного запроса.

Указание строки повторного запроса рекомендуется, когда есть вероятность, что входные данные пользователя не соответствуют запросу либо потому, что находятся в формате, который запрос не может проанализировать, например "tomorrow" для числового запроса, либо потому, что входные данные не соответствуют критериям проверки.

> [!TIP]
> При создании числового запроса необходимо указать используемые языки и региональные параметры ввода. Числовой запрос может интерпретировать широкий диапазон входных данных, таких как "twelve" или "one quarter", а также "12" и "0,25". Язык и региональные параметры ввода помогают запросу корректно интерпретировать данные, которые вводит пользователь.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Язык и региональные параметры ввода определяются в дополнительной библиотеке.

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
```

Следующий код добавит числовой запрос к существующему набору диалоговых окон **dialogs**.

```csharp
dialogs.Add("numberPrompt", new NumberPrompt<int>(Culture.English));
```

В шаге диалогового окна следующий код запросит у пользователя входные данные и предоставит строку повторного запроса на случай, если входные данные не могут быть интерпретированы как число.

```csharp
await dc.Prompt("numberPrompt", "How many people are in your party?", new PromptOptions()
{
    RetryPromptString = "Sorry, please specify the number of people in your party."
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {NumberPrompt} = require("botbuilder-dialogs");
```

```javascript
await dc.prompt('numberPrompt', 'How many people in your party?', { retryPrompt: `Sorry, please specify the number of people in your party.` })
```

```javascript
dialogs.add('numberPrompt', new NumberPrompt());
```

---

В частности, запрос на выбор требует некоторую дополнительную информацию, список доступных пользователю вариантов.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В этом примере используются типы из следующих пространств имен.

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System.Collections.Generic;
```


Когда мы используем **ChoicePrompt**, чтобы запросить пользователя выбрать один из нескольких вариантов, нужно предоставить запрос с набором опций, которые предоставляются внутри объекта **ChoicePromptOptions**. Здесь используется **ChoiceFactory** для преобразования списка вариантов в соответствующий формат.

Мы также используем действие **SuggestedActions** в качестве повторного запроса как способ повторного предоставления пользователю вариантов ввода.


```csharp
/// <summary>Defines a dialog that asks for a choice of color.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the color prompt.</summary>
        public const string Color = "colorPrompt";
    }

    /// <summary>The available colors to choose from.</summary>
    public List<string> Colors = new List<string> { "Green", "Blue" };

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        Add(Inputs.Color, new ChoicePrompt(Culture.English));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for a color. A choice prompt requires that you specify choice options.
                await dc.Prompt(Inputs.Color, "Please make a choice.", new ChoicePromptOptions()
                {
                    Choices = ChoiceFactory.ToChoices(Colors),
                    RetryPromptActivity =
                        MessageFactory.SuggestedActions(Colors, "Please choose a color.") as Activity
                });
            },
            async(dc, args, next) =>
            {
                var color = (FoundChoice)args["Value"];

                await dc.Context.SendActivity($"You chose {color.Value}.");
                await dc.End();
            }
        });
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ChoicePrompt} = require("botbuilder-dialogs");
```

```javascript
dialogs.add('choicePrompt', new ChoicePrompt());
```

```javascript
// A choice prompt requires that you specify choice options.
const list = ['green', 'blue'];
await dc.prompt('choicePrompt', 'Please make a choice', list, {retryPrompt: 'Please choose a color.'});
```

---

## <a name="validate-a-prompt-response"></a>Подтверждение ответа на запрос

Можно проверить ответ на запрос перед возвратом допустимого значения на следующий шаг **каскада**. Например, чтобы проверить **NumberPrompt** в диапазоне чисел между **6** и **20**, нужно иметь логику проверки, подобную этой.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Defines a dialog that asks for the number of people in a party.</summary>
public class MyDialog : DialogSet
{
    /// <summary>The ID of the main dialog in the set.</summary>
    public const string Name = "mainDialog";

    /// <summary>Defines the IDs of the prompts in the set.</summary>
    public struct Inputs
    {
        /// <summary>The ID of the party size prompt.</summary>
        public const string Size = "parytySize";
    }

    /// <summary>Defines the prompts and steps of the dialog.</summary>
    public MyDialog()
    {
        // Include a validation function for the party size prompt.
        Add(Inputs.Size, new NumberPrompt<int>(Culture.English, async (context, result) =>
        {
            if (result.Value < 6 || result.Value > 20)
            {
                result.Status = PromptStatus.OutOfRange;
            }
        }));
        Add(Name, new WaterfallStep[]
        {
            async (dc, args, next) =>
            {
                // Prompt for the party size.
                await dc.Prompt(Inputs.Size, "How many people are in your party?", new PromptOptions()
                {
                    RetryPromptString = "Please specify party size between 6 and 20."
                });
            },
            async(dc, args, next) =>
            {
                var size = (int)args["Value"];

                await dc.Context.SendActivity($"Okay, {size} people!");
                await dc.End();
            }
        });
    }
}
```

Проверка также может быть инкапсулирована внутри собственного частного метода и добавлена ​​таким образом.

```cs
/// <summary>Validates input for the partySize prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task PartySizeValidator(ITurnContext context, Int32Result result)
{
    if (result.Value < 6 || result.Value > 20)
    {
        result.Status = PromptStatus.OutOfRange;
    }
}
```

В диалоговом окне укажите метод, используемый для проверки ввода.

```cs
// Include a validation function for the party size prompt.
Add(Inputs.Size, new NumberPrompt<int>(Culture.English, PartySizeValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Customized prompts with validations
// A number prompt with validation for valid party size within a range.
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt( async (context, value) => {
    try {
        if(value < 6 ){
            throw new Error('Party size too small.');
        }
        else if(value > 20){
            throw new Error('Party size too big.')
        }
        else {
            return value; // Return the valid value
        }
    } catch (err) {
        await context.sendActivity(`${err.message} <br/>Please provide a valid number between 6 and 20.`);
        return undefined;
    }
}));
```

---

Аналогично, если нужно проверить ответ **DatetimePrompt** для даты и времени в будущем, нужно иметь логику проверки, подобную этой.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using DateTimeResult = Microsoft.Bot.Builder.Prompts.DateTimeResult;
using PromptStatus = Microsoft.Bot.Builder.Prompts.PromptStatus;
```

```cs
/// <summary>Validates input for the reservationTime prompt.</summary>
/// <param name="context">The context object for the current turn of the bot.</param>
/// <param name="result">The recognition result from the prompt.</param>
/// <returns>An updated recognition result.</returns>
private static async Task TimeValidator(ITurnContext context, DateTimeResult result)
{
    if (result.Resolution.Count == 0)
    {
        await context.SendActivity("Sorry, I did not recognize the time that you entered.");
        result.Status = PromptStatus.NotRecognized;
    }

    // Find any recognized time that is not in the past.
    var now = DateTime.Now;
    DateTime time = default(DateTime);
    var resolution = result.Resolution.FirstOrDefault(
        res => DateTime.TryParse(res.Value, out time) && time > now);

    if (resolution != null)
    {
        // If found, keep only that result.
        result.Resolution.Clear();
        result.Resolution.Add(resolution);
    }
    else
    {
        // Otherwise, flag the input as out of range.
        await context.SendActivity("Please enter a time in the future, such as \"tomorrow at 9am\"");
        result.Status = PromptStatus.OutOfRange;
    }
}
```

```csharp
Add(Inputs.Time, new DateTimePrompt(Culture.English, TimeValidator));
```

Дополнительные примеры можно найти в [примерах репозитория](https://github.com/Microsoft/botbuilder-dotnet).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```JavaScript
// A date and time prompt with validation for date/time in the future.
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt( async (context, values) => {
    try {
        if (values.length < 0) { throw new Error('missing time') }
        if (values[0].type !== 'datetime') { throw new Error('unsupported type') }
        const value = new Date(values[0].value);
        if (value.getTime() < new Date().getTime()) { throw new Error('in the past') }
        return value;
    } catch (err) {
        await context.sendActivity(`Please enter a valid time in the future like "tomorrow at 9am".`);
        return undefined;
    }
}));
```

Дополнительные примеры можно найти в [примерах репозитория](https://github.com/Microsoft/botbuilder-js).

---

> [!TIP]
> Если пользователь дает неоднозначный ответ на запрос даты и времени, можно разрешить несколько разных дат. В зависимости от цели использования можно проверить все разрешения, предоставленные результатом запроса, а не только первый.

Можно использовать аналогичные методы для проверки ответов на запросы любого типа.

## <a name="save-user-data"></a>Сохранение данных пользователя

Если запрашиваются входные данные пользователя, есть несколько вариантов управления ими. Например, можно использовать или отклонить входные данные, сохранить их в глобальной переменной, в энергозависимом контейнере или в памяти хранилища, в файле или во внешней базе данных. Дополнительные сведения о сохранении данных пользователя см. в разделе [Управление данными пользователя](bot-builder-howto-v4-state.md).

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знаете, как запрашивать у пользователя входные данные, улучшите код бота и пользовательский интерфейс, управляя разнообразными последовательностями общения с помощью диалоговых окон.



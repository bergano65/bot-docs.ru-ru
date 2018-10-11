---
title: Создание запросов на ввод данных пользователем с помощью библиотеки диалогов | Документация Майкрософт
description: Узнайте, как запросить у пользователя входные данные, используя библиотеку диалогов из пакета SDK Bot Builder для Node.js.
keywords: запросы, диалоговые окна, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, повторный запрос, проверка
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 27066f76db29a82b4ab9dd75bf5eee01dcce3116
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389713"
---
# <a name="prompt-users-for-input-using-the-dialogs-library"></a>Создание запросов на ввод данных пользователем с помощью библиотеки диалогов

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Сбор данных путем размещения вопросов — это один из основных способов взаимодействия между ботом и пользователями. Для его непосредственного выполнения можно использовать метод _отправки действия_ объекта в [контексте реплики](bot-builder-concept-activity-processing.md#turn-context), а затем обработать следующее входящее сообщение как ответ. Однако пакет SDK Bot Builder предоставляет библиотеку **диалогов**, в которой содержатся методы, предназначенные для облегчения постановки вопросов и призванные гарантировать, что ответы соответствуют определенному типу данных или правилам настраиваемой проверки. В этом разделе описано, как достигнуть этого с помощью **запроса** входных данных у пользователя.

В этой статье описывается использование запросов в диалоговом окне. Общие сведения об использовании диалогов см. в разделе [Управление ходом разговора с помощью диалогов](bot-builder-dialog-manage-conversation-flow.md).

## <a name="prompt-types"></a>Типы запросов

Библиотека диалогов предлагает много различных типов запросов, каждый из которых возвращает разные типы ответов.

| prompt | ОПИСАНИЕ |
|:----|:----|
| **AttachmentPrompt** | Запрос на добавление вложения пользователем, например документа или изображения. |
| **ChoicePrompt** | Запрос пользователю на выбор одного из нескольких вариантов. |
| **ConfirmPrompt** | Запрос на подтверждение действий пользователем. |
| **DatetimePrompt** | Запрос на ввод даты и времени пользователем. Пользователи могут отвечать, используя естественный язык, например "Tomorrow at 8pm" или "Friday at 10am". Пакет SDK для Bot Framework использует предварительно созданную сущность LUIS `builtin.datetimeV2`. Дополнительные сведения см. в [builtin.datetimev2](https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-prebuilt-entities#builtindatetimev2). |
| **NumberPrompt** | Запрос на ввод номера пользователем. Пользователь может ответить либо "10", либо "ten". Например, если ответ "ten", то запрос преобразует ответ в число и возвращает `10` в результате. |
| **TextPrompt** | Запрос на ввод текстовой строки пользователем. |

## <a name="add-references-to-prompt-library"></a>Добавление ссылок в библиотеку запросов

Получить библиотеку **диалогов** можно, добавив пакет **botbuilder-dialogs** в бот. Диалоги рассматриваются в статье об [использовании диалогов для управления простым процессом общения](bot-builder-dialog-manage-conversation-flow.md). Мы же будем использовать их для создания запросов.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Установите пакет **Microsoft.Bot.Builder.Dialogs** из NuGet.

Затем включите ссылку на библиотеку из кода бота.

```cs
using Microsoft.Bot.Builder.Dialogs;
```

Нужно настроить состояние диалогов беседы с помощью методов доступа. Здесь мы не будем углубляться в этот код. Подробные сведения о нем можно найти в статье о [состоянии](bot-builder-howto-v4-state.md).

В параметрах бота в файле **Startup.cs** сначала определите объекты состояния, затем добавьте одноэлементный шаблон, чтобы указать класс метода доступа в конструкторе ботов. В классе, указанном для `BotAccessor`, просто хранится состояние беседы и пользователя, а также методы доступа для каждого из этих элементов. Полное определение класса содержится в примере, ссылка на который доступна в конце этой статьи. 

```cs
    services.AddBot<MultiTurnPromptsBot>(options =>
    {
        InitCredentialProvider(options);

        // Create and add conversation state.
        var convoState = new ConversationState(dataStore);
        options.State.Add(convoState);

        // Create and add user state.
        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    services.AddSingleton(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        if (options == null)
        {
            throw new InvalidOperationException("BotFrameworkOptions must be configured prior to setting up the State Accessors");
        }

        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        if (conversationState == null)
        {
            throw new InvalidOperationException("ConversationState must be defined and added before adding conversation-scoped state accessors.");
        }

        var userState = options.State.OfType<UserState>().FirstOrDefault();
        if (userState == null)
        {
            throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
        }

        // The dialogs will need a state store accessor. Creating it here once (on-demand) allows the dependency injection
        // to hand it to our IBot class that is create per-request.
        var accessors = new BotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
```

Затем в коде бота определите следующие объекты для набора диалогов.

```cs
    private readonly BotAccessors _accessors;

    /// <summary>
    /// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
    /// </summary>
    private DialogSet _dialogs;

    /// <summary>
    /// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    public MultiTurnPromptsBot(BotAccessors accessors)
    {
        _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

        // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
        _dialogs = new DialogSet(accessors.ConversationDialogState);

        // ...
        // other constructor items
        // ...
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Установите пакет диалогов из NPM.

```cmd
npm install --save botbuilder-dialogs
```

Чтобы использовать **диалоги** в боте, включите эту библиотеку в код бота.

В файле app.js добавьте следующее.

```javascript
// Import components from the dialogs library.
const { DialogSet } = require("botbuilder-dialogs");
// Import components from the main Bot Builder library.
const { ConversationState, MemoryStorage } = require('botbuilder');

// Set up a memory storage system to store information.
const storage = new MemoryStorage();
// We'll use ConversationState to track the state of the dialogs.
const conversationState = new ConversationState(storage);
// Create a property used to track state.
const dialogState = conversationState.createProperty('dialogState');

// Create a dialog set to control our prompts, store the state in dialogState
const dialogs = new DialogSet(dialogState);
```

---

## <a name="prompt-the-user"></a>Запрос пользователю

Чтобы запросить ввод данных пользователем, определите запрос с помощью одного из встроенных классов, таких как **TextPrompt**, а затем добавьте его в набор диалогов и присвойте ему идентификатор диалога.

Добавив запрос, используйте его в двухшаговом диалоге *каскадного* типа, который является способом определения последовательности действий. Несколько запросов можно соединить друг с другом для создания многошаговых бесед. Дополнительные сведения см. в разделе об [использовании диалогов](bot-builder-dialog-manage-conversation-flow.md#using-dialogs-to-guide-the-user-through-steps) статьи [Управление простым процессом общения с помощью диалогов](bot-builder-dialog-manage-conversation-flow.md).

Например, в следующем диалоге запрашивается имя пользователя, а полученный ответ затем используется в приветствии. В первой реплике диалога запрашивается имя пользователя. Ответ пользователя передается в качестве параметра в функцию второго шага, которая обрабатывает входные данные и отправляет индивидуальное приветствие.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Каждому запросу, используемому в диалоговом окне, также присваивается имя, которое используется диалогом или ботом для доступа к запросу. Во всех этих примерах мы предоставляем идентификаторы запросов в качестве констант.

В своем конструкторе ботов добавьте определения для двухшагового каскадного диалога и запрос, который необходимо использовать в нем. Здесь мы добавляем их как независимые функции, но при желании их можно определить как встроенные лямбда-выражения.

```csharp
 public MultiTurnPromptsBot(BotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        SayHiAsync,
    };

    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
}
```

Затем определите два шага каскадного диалога в рамках бота. Для текстового запроса указывается идентификатор *имени* класса `TextPrompt`, определенного выше. Обратите внимание, что имена методов совпадают с атрибутом `WaterfallStep[]` выше. Будущие примеры в этой статье не будут включать этот код, но вы должны знать, что для выполнения дополнительных шагов необходимо добавить имя метода в правильном порядке в этом атрибуте `WaterfallStep[]`.

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Импортируйте класс TextPrompt в свое приложение.

```javascript
const { TextPrompt } = require("botbuilder-dialogs");
```

Создайте новый запрос и добавьте его в набор диалогов.

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add(new TextPrompt('textPrompt'));
dialogs.add('greetings', [
    async function (step){
        // the results of this prompt will be passed to the next step
        return await step.prompt('textPrompt', 'What is your name?');
    },
    async function(step) {
        // step.result is the result of the prompt defined above
        const userName = step.result;
        await step.context.sendActivity(`Hi ${userName}!`);
        return await step.endDialog();
    }
]);
```

---

> [!NOTE]
> Чтобы запустить диалоговое окно, получите контекст диалога и используйте метод _begin_. Дополнительные сведения см. в статье об [использовании диалогов для управления простым процессом общения](./bot-builder-dialog-manage-conversation-flow.md).

## <a name="reusable-prompts"></a>Многоразовые запросы

Запрос может использоваться повторно для постановки различных вопросов до тех пор, пока ответы относятся к одному типу. Например, приведенный выше пример кода определяет текстовый запрос и использует его для запроса имени пользователя. Можно также использовать этот же запрос, чтобы запросить у пользователя еще одну текстовую строку, например "Where do you work?" (Где вы работаете?).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В примере идентификатор нашего текстового запроса, *name*, не придает коду ясности. Однако это хороший пример того, что идентификатор запроса может быть любым.

Теперь наши методы включают третий шаг, позволяющий спросить пользователя о том, где он работает.

```cs
    private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
        // Running a prompt here means the next WaterfallStep will be run when the users response is received.
        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> WorkAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

        return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Where do you work?") }, cancellationToken);
    }

    private static async Task<DialogTurnResult> SayHiAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync($"{stepContext.Result} is a cool place!");

        return await stepContext.EndDialogAsync(cancellationToken);
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
// Ask them where they work.
dialogs.add(new TextPrompt('textPrompt'));
dialogs.add('greetings',[
    async function (step){
        // Use the textPrompt to ask for a name.
        return await step.prompt('textPrompt', 'What is your name?');
    },
    async function (step){
        const userName = step.result;
        await step.context.sendActivity(`Hi ${ userName }!`);

        // Now, reuse the same prompt to ask them where they work.
        return await step.prompt('textPrompt', 'Where do you work?');
    },
    async function(step) {
        const workPlace = step.result;
        await step.context.sendActivity(`${ workPlace } is a cool place!`);

        return await step.endDialog();
    }
]);
```

---

Если необходимо использовать несколько разных запросов, присвойте каждому запросу уникальный идентификатор *dialogId*. Каждому диалогу и запросу, которые добавляются в набор диалогов, требуется уникальный идентификатор. Кроме того, можно создать несколько диалогов с **запросами** одного типа. Например, можно создать два диалоговых окна **TextPrompt** для приведенного выше примера.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
_dialogs.Add(new WaterfallDialog("details", waterfallSteps));
_dialogs.Add(new TextPrompt("name"));
_dialogs.Add(new TextPrompt("workplace"));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
dialogs.add(new TextPrompt('namePrompt'));
dialogs.add(new TextPrompt('workPlacePrompt'));
```

---

Для обеспечения повторного использования кода определение одного атрибута `TextPrompt` будет работать для всех этих запросов, так как они ожидают ответ в виде текста. Возможность присваивать диалогам названия удобно использовать, когда для данных, которые вводятся в ответ на запрос, необходимо применять разные правила проверки. Давайте рассмотрим способ проверки ответа на запрос с помощью `NumberPrompt`.

## <a name="specify-prompt-options"></a>Указание параметров запроса

При использовании запроса в шаге диалогового окна можно также предоставить параметры запросов, такие как строка повторного запроса.

Указание строки повторного запроса рекомендуется, когда есть вероятность, что входные данные пользователя не соответствуют запросу либо потому, что находятся в формате, который запрос не может проанализировать, например "tomorrow" для числового запроса, либо потому, что входные данные не соответствуют критериям проверки. Числовой запрос может интерпретировать широкий диапазон входных данных, таких как "twelve" или "one quarter", а также "12" и "0,25".

Local является необязательным параметром определенных запросов, таких как **NumberPrompt**. Он может помочь более точно проанализировать входные данные по запросу, но не является обязательным.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Следующий код добавит числовой запрос к существующему набору диалогов **_dialogs**.

```csharp
_dialogs.Add(new NumberPrompt<int>("age"));
```

В шаге диалогового окна следующий код запросит у пользователя входные данные и предоставит строку повторного запроса на случай, если входные данные не могут быть интерпретированы как число.

```csharp
return await stepContext.PromptAsync(
    "age",
    new PromptOptions {
        Prompt = MessageFactory.Text("Please enter your age."),
        RetryPrompt = MessageFactory.Text("I didn't get that. Please enter a valid age."),
    },
    cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Import the NumberPrompt class from the dialog library.
const { NumberPrompt } = require("botbuilder-dialogs");

// Add a NumberPrompt to our dialog set and give it the ID numberPrompt.
dialogs.add(new NumberPrompt('numberPrompt'));

// Call the numberPrompt dialog with the (optional) retryPrompt parameter.
await dc.prompt('numberPrompt', 'How many people in your party?', { retryPrompt: `Sorry, please specify the number of people in your party.` })
```

---

Запрос выбора имеет дополнительный обязательный параметр — список доступных пользователю вариантов.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Когда используется **ChoicePrompt**, чтобы попросить пользователя выбрать один из нескольких вариантов, нужно предоставить запрос с набором вариантов, предусмотренных внутри объекта **ChoicePromptOptions**. Здесь используется **ChoiceFactory** для преобразования списка вариантов в соответствующий формат.

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    await stepContext.Context.SendActivityAsync($"Hi {stepContext.Result}!");

    return await stepContext.PromptAsync(
        "color",
        new PromptOptions {
            Prompt = MessageFactory.Text("What's your favorite color?"),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Import the ChoicePrompt class into your app from the dialogs library.
const { ChoicePrompt } = require("botbuilder-dialogs");
```

```javascript
// Add a ChoicePrompt to the dialog set and give it an ID of choicePrompt.
dialogs.add(new ChoicePrompt('choicePrompt'));
```

```javascript
// Call the choicePrompt into action, passing in an array of options.
const list = ['green', 'blue', 'red', 'yellow'];
await dc.prompt('choicePrompt', 'Please make a choice', list, { retryPrompt: 'Please choose a color.' });
```

---

## <a name="validate-a-prompt-response"></a>Подтверждение ответа на запрос

Можно проверить ответ на запрос, прежде чем значение будет возвращено на следующем этапе **каскада**. Например, чтобы проверить ответ на запрос **NumberPrompt** в диапазоне чисел между **6** и **20**, нужно включить функцию проверки, подобную этой:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Изменение при добавлении запроса в набор диалогов для включения функции проверки

```cs
_dialogs.Add(new NumberPrompt<int>("partySize", PartySizeValidatorAsync));
```

После этого проверка определяется как свой собственный метод, и в зависимости от результата проверки указывается значение true или false. Если возвращается значение false, пользователь получит повторный запрос.

```cs
private Task<bool> PartySizeValidatorAsync(PromptValidatorContext<int> promptContext, CancellationToken cancellationToken)
{
    var result = promptContext.Recognized.Value;

    if (result < 6 || result > 20)
    {
        return Task.FromResult(false);
    }

    return Task.FromResult(true);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Customized prompts with validations
// A number prompt with validation for valid party size within a range.
dialogs.add(new NumberPrompt('partySizePrompt', async (promptContext) => {
    // Check to make sure a value was recognized.
    if (promptContext.recognized.succeeded) {
        const value = promptContext.recognized.value;
        try {
            if (value < 6 ) {
                throw new Error('Party size too small.');
            } else if (value > 20) {
                throw new Error('Party size too big.')
            } else {
                return true; // Indicate that this is a valid value.
            }
        } catch (err) {
            await promptContext.context.sendActivity(`${ err.message } <br/>Please provide a valid number between 6 and 20.`);
            return false; // Indicate that this is invalid.
        }
    } else {
        return false;
    }
}));
```

---

Аналогично, если нужно проверить ответ **DatetimePrompt** для даты и времени в будущем, нужно иметь логику проверки, подобную этой.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
    private Task<bool> DateTimeValidatorAsync(PromptValidatorContext<IList<DateTimeResolution>> prompt, CancellationToken cancellationToken)
    {
        if (prompt.Recognized.Succeeded)
        {
            var resolution = prompt.Recognized.Value.First();

            // Verify that the Timex received is within the desired bounds, compared to today.
            var now = DateTime.Now;
            DateTime.TryParse(resolution.Value, out var time);

            if (time < now)
            {
                return Task.FromResult(false);
            }

            return Task.FromResult(true);
        }

        return Task.FromResult(false);
    }
```

```csharp
_dialogs.Add(new DateTimePrompt("date", DateTimeValidatorAsync));
```

Дополнительные примеры можно найти в [примерах репозитория](https://aka.ms/bot-samples-readme).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```JavaScript
// A date and time prompt with validation for date/time in the future.
dialogs.add(new atetimePrompt('dateTimePrompt', async (promptContext) => {
    if (promptContext.recognized.succeeded) {
        const values = promptContext.recognized.value;
        try {
            if (values.length < 0) { throw new Error('missing time') }
            if (values[0].type !== 'date') { throw new Error('unsupported type') }
            const value = new Date(values[0].value);
            if (value.getTime() < new Date().getTime()) { throw new Error('in the past') }

            // update the return value of the prompt to be a real date object
            promptContext.recognized.value = value;
            return true; // indicate valid 
        } catch (err) {
            await promptContext.context.sendActivity(`Please enter a valid time in the future like "tomorrow at 9am".`);
            return false; // indicate invalid
        }
    } else {
        return false;
    }
}));
```

Дополнительные примеры можно найти в [примерах репозитория](https://aka.ms/bot-samples-readme).

---

> [!TIP]
> Если пользователь дает неоднозначный ответ на запрос даты и времени, можно разрешить несколько разных дат. В зависимости от цели использования можно проверить все разрешения, предоставленные результатом запроса, а не только первый.

Можно использовать аналогичные методы для проверки ответов на запросы любого типа.

## <a name="save-user-data"></a>Сохранение данных пользователя

Если запрашиваются входные данные пользователя, есть несколько вариантов управления ими. Например, можно использовать или отклонить входные данные, сохранить их в глобальной переменной, в энергозависимом контейнере или в памяти хранилища, в файле или во внешней базе данных. Дополнительные сведения о сохранении данных пользователя см. в разделе [Управление данными пользователя](bot-builder-howto-v4-state.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

Полный пример использования некоторых из этих запросов см. в разделе Multi Turn Prompt Bot для [C#](https://aka.ms/cs-multi-prompts-sample) или [JavaScript](https://aka.ms/js-multi-prompts-sample).

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знаете, как запрашивать у пользователя входные данные, улучшите код бота и пользовательский интерфейс, управляя разнообразными последовательностями общения с помощью диалоговых окон.

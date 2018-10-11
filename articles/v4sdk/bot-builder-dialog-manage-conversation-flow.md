---
title: Управление простым процессом общения с помощью диалогов | Документация Майкрософт
description: Сведения о том, как управлять простым процессом общения с помощью диалогов в пакете SDK Bot Builder для Node.js.
keywords: simple conversation flow, dialogs, prompts, waterfalls, dialog set
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 9/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c70711d747e9646acf63b6ee206d0b8db25ef202
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389692"
---
# <a name="manage-simple-conversation-flow-with-dialogs"></a>Управление простым процессом общения с помощью диалогов

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

С помощью библиотеки диалогов можно управлять простыми и сложными процессами общения. В рамках простого процесса общения пользователь последовательно выполняет все шаги *каскада*, после чего обмен сообщениями прекращается. [Сложный процесс общения](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md) включает ветви и циклы.

<!-- TODO: This paragraph belongs in a conceptual topic. -->

Объекты Dialogs — это структуры бота, которые выполняются подобно функциям программы бота. Они создают сообщения, которые отправляет бот, и выполняют необходимые вычислительные задачи. Они предназначены для выполнения определенных операций в указанном порядке. Эти объекты можно вызывать начинать различными способами: в ответ на запрос пользователя, внешний стимул, или же диалог может быть активирован из других диалогов.

С их помощью разработчик бота может управлять процессом общения. Можно создать несколько таких объектов и связать их вместе, чтобы создать процесс общения, которым должен управлять бот. Библиотека объектов **Dialogs** пакета SDK Bot Builder включает встроенные функции, например _запросы_, _каскадные диалоги_ и _диалоги-компоненты_, которые помогают управлять процессом общения. С помощью запросов можно запрашивать у пользователей различные типы данных. Вы можете использовать каскад, чтобы объединить несколько шагов в последовательность. Диалоги-компоненты можно использовать для создания модульных систем диалогов, содержащих несколько поддиалогов.

В рамках этой статьи мы используем _наборы диалогов_ для создания последовательности общения с запросами и каскадами. В статье будут использованы примеры кода из примера **диалогового запроса** [[C#](https://aka.ms/cs-multi-prompts-sample)|[JS](https://aka.ms/js-multi-prompts-sample)].

Общие сведения о диалогах см. в статьях о [библиотеке диалогов](bot-builder-concept-dialog.md) и [состоянии диалогов](bot-builder-dialog-state.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы использовать общую функциональность диалогов, вам необходимо установить пакет NuGet `Microsoft.Bot.Builder.Dialogs` для проекта или решения.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Для использования базовых функций диалогов вам необходима библиотека `botbuilder-dialogs`, которую можно установить с помощью npm.

---

## <a name="using-dialogs-to-guide-the-user-through-steps"></a>Использование диалогов для продвижения пользователя по действиям

В это примере мы создадим многошаговый диалог для запрашивания сведений у пользователя с использованием набора диалогов.

### <a name="create-a-dialog-with-waterfall-steps"></a>Создание диалога с действиями каскада

**WaterfallDialog** — это реализация диалога, которая обычно используется для сбора информации от пользователя или предоставления пользователю инструкций по выполнению ряда задач. Каждый шаг общения реализован в виде функции. На каждом шаге бот [запрашивает у пользователя входные данные](bot-builder-prompts.md), ожидает ответа, а затем передает результат в следующий шаг. Результат выполнения первой функции передается в виде аргумента следующую функцию и т. д.

Например, в следующем примере кода определен массив делегатов, которые представляют шаги **каскада**. После каждого запроса бот подтверждает получение данных от пользователя. Существует множество способов сохранения сведений, получаемых в процессе диалога. Некоторые варианты описаны в статье [Хранение данных пользователя](bot-builder-tutorial-persist-user-inputs.md).

В этом примере сведения, получаемые в процессе диалога, записываются непосредственно в профиль пользователя.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В этом примере каскадный диалог задается в файле с расширением .bot.

Укажите пространства имен, используемые в этом файле.

```csharp
using System;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
```

Укажите свойство экземпляра для набора диалогов.

```csharp
/// <summary>
/// The <see cref="DialogSet"/> that contains all the Dialogs that can be used at runtime.
/// </summary>
private DialogSet _dialogs;
```

Внутри конструктора бота создайте набор диалогов, добавив в него запросы и каскадный диалог.

```csharp
/// <summary>
/// Initializes a new instance of the <see cref="MultiTurnPromptsBot"/> class.
/// </summary>
/// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
public MultiTurnPromptsBot(MultiTurnPromptsBotAccessors accessors)
{
    _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

    // The DialogSet needs a DialogState accessor, it will call it when it has a turn context.
    _dialogs = new DialogSet(accessors.ConversationDialogState);

    // This array defines how the Waterfall will execute.
    var waterfallSteps = new WaterfallStep[]
    {
        NameStepAsync,
        NameConfirmStepAsync,
        AgeStepAsync,
        ConfirmStepAsync,
        SummaryStepAsync,
    };

    // Add named dialogs to the DialogSet. These names are saved in the dialog state.
    _dialogs.Add(new WaterfallDialog("details", waterfallSteps));
    _dialogs.Add(new TextPrompt("name"));
    _dialogs.Add(new NumberPrompt<int>("age"));
    _dialogs.Add(new ConfirmPrompt("confirm"));
}
```

Далее определите каждый шаг в виде отдельного метода. Вы также можете определить шаги, используя лямбда-выражения.

```csharp
/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> NameConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Name = (string)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    await stepContext.Context.SendActivityAsync(MessageFactory.Text($"Thanks {stepContext.Result}."), cancellationToken);

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Would you like to give your age?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> AgeStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // User said "yes" so we will be prompting for the age.

        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
        return await stepContext.PromptAsync("age", new PromptOptions { Prompt = MessageFactory.Text("Please enter your age.") }, cancellationToken);
    }
    else
    {
        // User said "no" so we will skip the next step. Give -1 as the age.
        return await stepContext.NextAsync(-1, cancellationToken);
    }
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> ConfirmStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the current profile object from user state.
    var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

    // Update the profile.
    userProfile.Age = (int)stepContext.Result;

    // We can send messages to the user at any point in the WaterfallStep.
    if (userProfile.Age == -1)
    {
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"No age given."), cancellationToken);
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your age as {userProfile.Age}."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is a Prompt Dialog.
    return await stepContext.PromptAsync("confirm", new PromptOptions { Prompt = MessageFactory.Text("Is this ok?") }, cancellationToken);
}

/// <summary>
/// One of the functions that make up the <see cref="WaterfallDialog"/>.
/// </summary>
/// <param name="stepContext">The <see cref="WaterfallStepContext"/> gives access to the executing dialog runtime.</param>
/// <param name="cancellationToken">A <see cref="CancellationToken"/>.</param>
/// <returns>A <see cref="DialogTurnResult"/> to communicate some flow control back to the containing WaterfallDialog.</returns>
private async Task<DialogTurnResult> SummaryStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    if ((bool)stepContext.Result)
    {
        // Get the current profile object from user state.
        var userProfile = await _accessors.UserProfile.GetAsync(stepContext.Context, () => new UserProfile(), cancellationToken);

        // We can send messages to the user at any point in the WaterfallStep.
        if (userProfile.Age == -1)
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name}."), cancellationToken);
        }
        else
        {
            await stepContext.Context.SendActivityAsync(MessageFactory.Text($"I have your name as {userProfile.Name} and age as {userProfile.Age}."), cancellationToken);
        }
    }
    else
    {
        // We can send messages to the user at any point in the WaterfallStep.
        await stepContext.Context.SendActivityAsync(MessageFactory.Text("Thanks. Your profile will not be kept."), cancellationToken);
    }

    // WaterfallStep always finishes with the end of the Waterfall or with another dialog, here it is the end.
    return await stepContext.EndDialogAsync(cancellationToken: cancellationToken);
}
```

Диалог запускается поочередным обработчиком бота, который сначала создает контекст диалога, а затем соответственно продолжает или начинает диалог, после чего сохраняет состояние общения и пользователя в конце шага.

```csharp
// Run the DialogSet - let the framework identify the current state of the dialog from
// the dialog stack and figure out what (if any) is the active dialog.
var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
var results = await dialogContext.ContinueDialogAsync(cancellationToken);

// If the DialogTurnStatus is Empty we should start a new dialog.
if (results.Status == DialogTurnStatus.Empty)
{
    await dialogContext.BeginDialogAsync("details", null, cancellationToken);
}
```

```csharp
// Save the dialog state into the conversation state.
await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

// Save the user profile updates into the user state.
await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В этом примере каскадный диалог задается в файле **bot.js**.

Импортируйте объекты, необходимые в коде.

```javascript
const { ActivityTypes } = require('botbuilder');
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

В конструкторе бота определите и создайте набор диалогов, добавив в него запросы и каскадный диалог.

```javascript
/**
*
* @param {ConversationState} conversationState A ConversationState object used to store the dialog state.
* @param {UserState} userState A UserState object used to store values specific to the user.
*/
constructor(conversationState, userState) {
    // Create a new state accessor property. See https://aka.ms/about-bot-state-accessors to learn more about bot state and state accessors.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty(DIALOG_STATE_PROPERTY);

    this.userProfile = this.userState.createProperty(USER_PROFILE_PROPERTY);

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts that will be used by the main dialogs.
    this.dialogs.add(new TextPrompt(NAME_PROMPT));
    this.dialogs.add(new ChoicePrompt(CONFIRM_PROMPT));
    this.dialogs.add(new NumberPrompt(AGE_PROMPT, async (prompt) => {
        if (prompt.recognized.succeeded) {
            if (prompt.recognized.value <= 0) {
                await prompt.context.sendActivity(`Your age can't be less than zero.`);
                return false;
            } else {
                return true;
            }
        }

        return false;
    }));

    // Create a dialog that asks the user for their name.
    this.dialogs.add(new WaterfallDialog(WHO_ARE_YOU, [
        this.promptForName.bind(this),
        this.confirmAgePrompt.bind(this),
        this.promptForAge.bind(this),
        this.captureAge.bind(this)
    ]));

    // Create a dialog that displays a user name after it has been collected.
    this.dialogs.add(new WaterfallDialog(HELLO_USER, [
        this.displayProfile.bind(this)
    ]));
}
```

Далее определите каждый шаг в виде отдельного метода. Вы также можете определить шаги, используя лямбда-выражения.

```javascript
// This step in the dialog prompts the user for their name.
async promptForName(step) {
    return await step.prompt(NAME_PROMPT, `What is your name, human?`);
}

// This step captures the user's name, then prompts whether or not to collect an age.
async confirmAgePrompt(step) {
    const user = await this.userProfile.get(step.context, {});
    user.name = step.result;
    await this.userProfile.set(step.context, user);
    await step.prompt(CONFIRM_PROMPT, 'Do you want to give your age?', ['yes', 'no']);
}

// This step checks the user's response - if yes, the bot will proceed to prompt for age.
// Otherwise, the bot will skip the age step.
async promptForAge(step) {
    if (step.result && step.result.value === 'yes') {
        return await step.prompt(AGE_PROMPT, `What is your age?`,
            {
                retryPrompt: 'Sorry, please specify your age as a positive number or say cancel.'
            }
        );
    } else {
        return await step.next(-1);
    }
}

// This step captures the user's age.
async captureAge(step) {
    const user = await this.userProfile.get(step.context, {});
    if (step.result !== -1) {
        user.age = step.result;
        await this.userProfile.set(step.context, user);
        await step.context.sendActivity(`I will remember that you are ${ step.result } years old.`);
    } else {
        await step.context.sendActivity(`No age given.`);
    }
    return await step.endDialog();
}

// This step displays the captured information back to the user.
async displayProfile(step) {
    const user = await this.userProfile.get(step.context, {});
    if (user.age) {
        await step.context.sendActivity(`Your name is ${ user.name } and you are ${ user.age } years old.`);
    } else {
        await step.context.sendActivity(`Your name is ${ user.name } and you did not share your age.`);
    }
    return await step.endDialog();
}
```

Диалог запускается поочередным обработчиком бота, который сначала создает контекст диалога, а затем соответственно продолжает или начинает диалог, после чего сохраняет состояние общения и пользователя в конце шага.

```javascript
// Create a dialog context object.
const dc = await this.dialogs.createContext(turnContext);
```

```javascript
// If the bot has not yet responded, continue processing the current dialog.
await dc.continueDialog();
```

```javascript
// Start the sample dialog in response to any other input.
if (!turnContext.responded) {
    const user = await this.userProfile.get(dc.context, {});
    if (user.name) {
        await dc.beginDialog(HELLO_USER);
    } else {
        await dc.beginDialog(WHO_ARE_YOU);
    }
}
```

```javascript
// Save changes to the user state.
await this.userState.saveChanges(turnContext);

// End this turn by saving changes to the conversation state.
await this.conversationState.saveChanges(turnContext);
```

---

## <a name="dialog-context-and-waterfall-step-context-objects"></a>Контекстные объекты диалога и шага каскада

Контекстный объект диалога используется для взаимодействия с набором диалогов из поочередного обработчика бота.
Контекстный объект шага каскада используется для взаимодействия с набором диалогов из шага каскада.

## <a name="to-start-a-dialog"></a>Начало диалога

Чтобы начать диалог, передайте идентификатор *dialogId* нужного диалога в метод контекста диалога _beginDialog_, _prompt_ или _replaceDialog_. Метод _beginDialog_ отправит диалог на вершину стека, а метод _replaceDialog_ извлечет текущий диалог из стека и поместит новый диалог в стек.

Метод _prompt_ контекста — это вспомогательный метод, который принимает аргументы и создает соответствующие параметры запроса, а затем начинает диалог запроса. Дополнительные сведения о запросах см. в статье [Запрос пользователям на ввод данных](bot-builder-prompts.md).

## <a name="to-end-a-dialog"></a>Завершение диалога

Метод _end dialog_ завершает диалог, извлекая его из стека и, если необходимо, возвращает результат в родительский диалог.

Рекомендуется вызывать метод _endDialog_  явным образом в конце диалога.

## <a name="to-clear-the-dialog-stack"></a>Очистка стека диалогов

Если нужно извлечь все диалоги из стека, можно очистись стек диалога, вызвав метод _cancel all dialogs_ для контекста диалога.

## <a name="to-repeat-a-dialog"></a>Повтор диалога

Чтобы повторить диалог, используйте метод _replace dialog_, который извлечет текущий диалог из стека и поместит другой диалог на вершину стека, после чего начнет этот диалог. Это отличный способ для обработки [сложных диалогов](~/v4sdk/bot-builder-dialog-manage-complex-conversation-flow.md) и хороший метод для управления меню.

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы узнали, как управлять простым процессом общения, давайте рассмотрим, как использовать метод _replace dialog_ для управления сложными процессами общения.

> [!div class="nextstepaction"]
> [Управление сложным процессом общения](bot-builder-dialog-manage-complex-conversation-flow.md)

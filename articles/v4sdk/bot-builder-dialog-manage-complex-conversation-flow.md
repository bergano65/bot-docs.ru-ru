---
title: Создание сложного потока беседы с использованием ветвления и циклов | Документация Майкрософт
description: Сведения о том, как управлять сложным потоком беседы с помощью диалогов из пакета SDK Bot Framework.
keywords: complex conversation flow, repeat, loop, menu, dialogs, prompts, waterfalls, dialog set
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a5f3fe4fbec5a44a68bd6dcb7a2d6e2770052923
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905027"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>Создание сложного потока беседы с использованием ветвления и циклов

[!INCLUDE[applies-to](../includes/applies-to.md)]

В этой статье мы покажем, как управлять сложными беседами с ветвлениями и циклами. Мы также продемонстрируем передачу аргументов между разными частями диалога.

## <a name="prerequisites"></a>Предварительные требования

- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download).
- Код в этой статье основан на примере **complex dialog** (сложный диалог). Вам потребуется копия этого примера на языке [C#](https://aka.ms/cs-complex-dialog-sample) или [JS](https://aka.ms/js-complex-dialog-sample).
- Понимание [основных принципов работы ботов](bot-builder-basics.md), [библиотеки диалогов](bot-builder-concept-dialog.md), [состояния диалогов](bot-builder-dialog-state.md) и [файлов .bot](bot-file-basics.md).

## <a name="about-the-sample"></a>Сведения о примере

Этот пример содержит бот, который может выполнять регистрацию пользователей для оценки одной или двух компаний из заданного списка.

- Он запрашивает имя и возраст пользователя, и в зависимости от возраста пользователя выполняет _ветвление_.
  - Если пользователь слишком молод, бот не предлагает ему оценивать компании.
  - Если возраст пользователя достаточен, бот начинает сбор предпочтений пользователя.
    - Он предлагает пользователю выбрать компании для оценки.
    - Если пользователь выберет компанию, повторяется _цикл_ для выбора второй компании.
- И в завершение бот благодарит пользователя за участие.

Для управления сложной беседой здесь применяются два каскадных диалога и несколько запросов.

## <a name="configure-state-for-your-bot"></a>Настройка состояния для бота

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Определите, какие сведения о пользователе мы будем собирать.

```csharp
public class UserProfile
{
    public string Name { get; set; }
    public int Age { get; set; }

    //The list of companies the user wants to review.
    public List<string> CompaniesToReview { get; set; } = new List<string>();
}
```

Определите класс для хранения объектов управления состоянием и методов доступа к свойствам состояния бота.

```csharp
public class ComplexDialogBotAccessors
{
    public ComplexDialogBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public ConversationState ConversationState { get; }
    public UserState UserState { get; }
}
```

Создайте объект управления состоянием и зарегистрируйте класс методов доступа в методе `ConfigureServices` класса `Statup`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Register the bot.

    // Create conversation and user state management objects, using memory storage.
    IStorage dataStore = new MemoryStorage();
    var conversationState = new ConversationState(dataStore);
    var userState = new UserState(dataStore);

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<ComplexDialogBotAccessors>(sp =>
    {
        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new ComplexDialogBotAccessors(conversationState, userState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfileAccessor = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В файле **index.js** мы определяем объекты управления состоянием.

```javascript
const { BotFrameworkAdapter, MemoryStorage, UserState, ConversationState } = require('botbuilder');

// ...

// Define state store for your bot.
const memoryStorage = new MemoryStorage();

// Create user and conversation state with in-memory storage provider.
const userState = new UserState(memoryStorage);
const conversationState = new ConversationState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

Конструктор бота создаст методы доступа к свойству состояния бота.

---

## <a name="initialize-your-bot"></a>Инициализация бота

Создайте для бота _набор диалогов_, в который мы добавим все диалоги для этого примера.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Внутри конструктора бота создайте набор диалогов, добавив в него запросы и два каскадных диалога.

Для этого примера каждый шаг мы определяем в виде отдельного метода. Этот подход мы применим в следующем разделе.

```csharp
public class ComplexDialogBot : IBot
{
    // Define constants for the bot...

    // Define properties for the bot's accessors and dialog set.
    private readonly ComplexDialogBotAccessors _accessors;
    private readonly DialogSet _dialogs;

    // Initialize the bot and add dialogs and prompts to the dialog set.
    public ComplexDialogBot(ComplexDialogBotAccessors accessors)
    {
        _accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));

        // Create a dialog set for the bot. It requires a DialogState accessor, with which
        // to retrieve the dialog state from the turn context.
        _dialogs = new DialogSet(accessors.DialogStateAccessor);

        // Add the prompts we need to the dialog set.
        _dialogs
            .Add(new TextPrompt(NamePrompt))
            .Add(new NumberPrompt<int>(AgePrompt))
            .Add(new ChoicePrompt(SelectionPrompt));

        // Add the dialogs we need to the dialog set.
        _dialogs.Add(new WaterfallDialog(TopLevelDialog)
            .AddStep(NameStepAsync)
            .AddStep(AgeStepAsync)
            .AddStep(StartSelectionStepAsync)
            .AddStep(AcknowledgementStepAsync));

        _dialogs.Add(new WaterfallDialog(ReviewSelectionDialog)
            .AddStep(SelectionStepAsync)
            .AddStep(LoopStepAsync));
    }

    // Turn handler and other supporting methods...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В файле **bot.js** в конструкторе бота определите и создайте набор диалогов, добавив в него запросы и каскадные диалоги.

Для этого примера каждый шаг мы определяем в виде отдельного метода. Этот подход мы применим в следующем разделе.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, TextPrompt, NumberPrompt, ChoicePrompt, DialogTurnStatus } = require('botbuilder-dialogs');

// Define constants for the bot...

class MyBot {
    constructor(conversationState, userState) {
        // Create the state property accessors and save the state management objects.
        this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
        this.userProfileAccessor = userState.createProperty(USER_PROFILE_PROPERTY);
        this.conversationState = conversationState;
        this.userState = userState;

        // Create a dialog set for the bot. It requires a DialogState accessor, with which
        // to retrieve the dialog state from the turn context.
        this.dialogs = new DialogSet(this.dialogStateAccessor);

        // Add the prompts we need to the dialog set.
        this.dialogs
            .add(new TextPrompt(NAME_PROMPT))
            .add(new NumberPrompt(AGE_PROMPT))
            .add(new ChoicePrompt(SELECTION_PROMPT));

        // Add the dialogs we need to the dialog set.
        this.dialogs.add(new WaterfallDialog(TOP_LEVEL_DIALOG)
            .addStep(this.nameStep.bind(this))
            .addStep(this.ageStep.bind(this))
            .addStep(this.startSelectionStep.bind(this))
            .addStep(this.acknowledgementStep.bind(this)));

        this.dialogs.add(new WaterfallDialog(REVIEW_SELECTION_DIALOG)
            .addStep(this.selectionStep.bind(this))
            .addStep(this.loopStep.bind(this)));
    }

    // Turn handler and other supporting methods...
}
```

---

## <a name="implement-the-steps-for-the-waterfall-dialogs"></a>Реализация действий для каскадных диалогов

Теперь мы реализуем действия для двух диалогов.

### <a name="the-top-level-dialog"></a>Диалог верхнего уровня

Исходный диалог верхнего уровня состоит из четырех шагов:

1. запрос имени пользователя;
1. запрос возраста пользователя;
1. ветвление в зависимости от возраста пользователя;
1. отправка пользователю благодарности за участие и возвращение собранных сведений.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The first step of the top-level dialog.
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Create an object in which to collect the user's information within the dialog.
    stepContext.Values[UserInfo] = new UserProfile();

    // Ask the user to enter their name.
    return await stepContext.PromptAsync(
        NamePrompt,
        new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") },
        cancellationToken);
}

// The second step of the top-level dialog.
private async Task<DialogTurnResult> AgeStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Set the user's name to what they entered in response to the name prompt.
    ((UserProfile)stepContext.Values[UserInfo]).Name = (string)stepContext.Result;

    // Ask the user to enter their age.
    return await stepContext.PromptAsync(
        AgePrompt,
        new PromptOptions { Prompt = MessageFactory.Text("Please enter your age.") },
        cancellationToken);
}

// The third step of the top-level dialog.
private async Task<DialogTurnResult> StartSelectionStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Set the user's age to what they entered in response to the age prompt.
    int age = (int)stepContext.Result;
    ((UserProfile)stepContext.Values[UserInfo]).Age = age;

    if (age < 25)
    {
        // If they are too young, skip the review-selection dialog, and pass an empty list to the next step.
        await stepContext.Context.SendActivityAsync(
            MessageFactory.Text("You must be 25 or older to participate."),
            cancellationToken);
        return await stepContext.NextAsync(new List<string>(), cancellationToken);
    }
    else
    {
        // Otherwise, start the review-selection dialog.
        return await stepContext.BeginDialogAsync(ReviewSelectionDialog, null, cancellationToken);
    }
}

// The final step of the top-level dialog.
private async Task<DialogTurnResult> AcknowledgementStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Set the user's company selection to what they entered in the review-selection dialog.
    List<string> list = stepContext.Result as List<string>;
    ((UserProfile)stepContext.Values[UserInfo]).CompaniesToReview = list ?? new List<string>();

    // Thank them for participating.
    await stepContext.Context.SendActivityAsync(
        MessageFactory.Text($"Thanks for participating, {((UserProfile)stepContext.Values[UserInfo]).Name}."),
        cancellationToken);

    // Exit the dialog, returning the collected user information.
    return await stepContext.EndDialogAsync(stepContext.Values[UserInfo], cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async nameStep(stepContext) {
    // Create an object in which to collect the user's information within the dialog.
    stepContext.values[USER_INFO] = {};

    // Ask the user to enter their name.
    return await stepContext.prompt(NAME_PROMPT, 'Please enter your name.');
}

async ageStep(stepContext) {
    // Set the user's name to what they entered in response to the name prompt.
    stepContext.values[USER_INFO].name = stepContext.result;

    // Ask the user to enter their age.
    return await stepContext.prompt(AGE_PROMPT, 'Please enter your age.');
}

async startSelectionStep(stepContext) {
    // Set the user's age to what they entered in response to the age prompt.
    stepContext.values[USER_INFO].age = stepContext.result;

    if (stepContext.result < 25) {
        // If they are too young, skip the review-selection dialog, and pass an empty list to the next step.
        await stepContext.context.sendActivity('You must be 25 or older to participate.');
        return await stepContext.next([]);
    } else {
        // Otherwise, start the review-selection dialog.
        return await stepContext.beginDialog(REVIEW_SELECTION_DIALOG);
    }
}

async acknowledgementStep(stepContext) {
    // Set the user's company selection to what they entered in the review-selection dialog.
    const list = stepContext.result || [];
    stepContext.values[USER_INFO].companiesToReview = list;

    // Thank them for participating.
    await stepContext.context.sendActivity(`Thanks for participating, ${stepContext.values[USER_INFO].name}.`);

    // Exit the dialog, returning the collected user information.
    return await stepContext.endDialog(stepContext.values[USER_INFO]);
}
```

---

### <a name="the-review-selection-dialog"></a>Диалог выбора элементов для обзора

Диалог выбора элементов для обзора состоит из двух шагов:

1. предложение пользователю выбрать компанию для оценки или ввести `done`, чтобы завершить процесс;
1. повтор того же диалога или выход, в зависимости от некоторых условий.

В такой системе диалог верхнего уровня всегда находится в стеке выше, чем диалог выбора элементов для оценки, а окно выбора можно считать дочерним элементом диалога верхнего уровня.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The first step of the review-selection dialog.
private async Task<DialogTurnResult> SelectionStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Continue using the same selection list, if any, from the previous iteration of this dialog.
    List<string> list = stepContext.Options as List<string> ?? new List<string>();
    stepContext.Values[CompaniesSelected] = list;

    // Create a prompt message.
    string message;
    if (list.Count is 0)
    {
        message = $"Please choose a company to review, or `{DoneOption}` to finish.";
    }
    else
    {
        message = $"You have selected **{list[0]}**. You can review an additional company, " +
            $"or choose `{DoneOption}` to finish.";
    }

    // Create the list of options to choose from.
    List<string> options = _companyOptions.ToList();
    options.Add(DoneOption);
    if (list.Count > 0)
    {
        options.Remove(list[0]);
    }

    // Prompt the user for a choice.
    return await stepContext.PromptAsync(
        SelectionPrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text(message),
            RetryPrompt = MessageFactory.Text("Please choose an option from the list."),
            Choices = ChoiceFactory.ToChoices(options),
        },
        cancellationToken);
}

// The final step of the review-selection dialog.
private async Task<DialogTurnResult> LoopStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Retrieve their selection list, the choice they made, and whether they chose to finish.
    List<string> list = stepContext.Values[CompaniesSelected] as List<string>;
    FoundChoice choice = (FoundChoice)stepContext.Result;
    bool done = choice.Value == DoneOption;

    if (!done)
    {
        // If they chose a company, add it to the list.
        list.Add(choice.Value);
    }

    if (done || list.Count is 2)
    {
        // If they're done, exit and return their list.
        return await stepContext.EndDialogAsync(list, cancellationToken);
    }
    else
    {
        // Otherwise, repeat this dialog, passing in the list from this iteration.
        return await stepContext.ReplaceDialogAsync(ReviewSelectionDialog, list, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async selectionStep(stepContext) {
    // Continue using the same selection list, if any, from the previous iteration of this dialog.
    const list = Array.isArray(stepContext.options) ? stepContext.options : [];
    stepContext.values[COMPANIES_SELECTED] = list;

    // Create a prompt message.
    let message;
    if (list.length === 0) {
        message = 'Please choose a company to review, or `' + DONE_OPTION + '` to finish.';
    } else {
        message = `You have selected **${list[0]}**. You can review an addition company, ` +
            'or choose `' + DONE_OPTION + '` to finish.';
    }

    // Create the list of options to choose from.
    const options = list.length > 0
        ? COMPANY_OPTIONS.filter(function (item) { return item !== list[0] })
        : COMPANY_OPTIONS.slice();
    options.push(DONE_OPTION);

    // Prompt the user for a choice.
    return await stepContext.prompt(SELECTION_PROMPT, {
        prompt: message,
        retryPrompt: 'Please choose an option from the list.',
        choices: options
    });
}

async loopStep(stepContext) {
    // Retrieve their selection list, the choice they made, and whether they chose to finish.
    const list = stepContext.values[COMPANIES_SELECTED];
    const choice = stepContext.result;
    const done = choice.value === DONE_OPTION;

    if (!done) {
        // If they chose a company, add it to the list.
        list.push(choice.value);
    }

    if (done || list.length > 1) {
        // If they're done, exit and return their list.
        return await stepContext.endDialog(list);
    } else {
        // Otherwise, repeat this dialog, passing in the list from this iteration.
        return await stepContext.replaceDialog(REVIEW_SELECTION_DIALOG, list);
    }
}
```

---

## <a name="update-the-bots-turn-handler"></a>Обновление обработчика шагов бота

Обработчик шагов бота повторяет один и тот же поток общения, который определен в этих диалогах.
При получении сообщения от пользователя применяется следующий алгоритм.

1. Продолжить активный диалог, если таковой имеется.
   - Если активного диалога нет, очистить профиль пользователя и запустить диалог верхнего уровня.
   - Если активный диалог завершается, собрать и сохранить полученные данные и отобразить информационное сообщение.
   - Все остальные ситуации означают, что мы находимся в середине процесса активного диалога, и дополнительные действия в данный момент не требуются.
1. Сохранить состояние диалога, чтобы не потерять обновленные сведения о состоянии диалога.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext == null)
    {
        throw new ArgumentNullException(nameof(turnContext));
    }

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        DialogContext dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
        DialogTurnResult results = await dialogContext.ContinueDialogAsync(cancellationToken);
        switch (results.Status)
        {
            case DialogTurnStatus.Cancelled:
            case DialogTurnStatus.Empty:
                // If there is no active dialog, we should clear the user info and start a new dialog.
                await _accessors.UserProfileAccessor.SetAsync(turnContext, new UserProfile(), cancellationToken);
                await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
                await dialogContext.BeginDialogAsync(TopLevelDialog, null, cancellationToken);
                break;

            case DialogTurnStatus.Complete:
                // If we just finished the dialog, capture and display the results.
                UserProfile userInfo = results.Result as UserProfile;
                string status = "You are signed up to review "
                    + (userInfo.CompaniesToReview.Count is 0
                        ? "no companies"
                        : string.Join(" and ", userInfo.CompaniesToReview))
                    + ".";
                await turnContext.SendActivityAsync(status);
                await _accessors.UserProfileAccessor.SetAsync(turnContext, userInfo, cancellationToken);
                await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
                break;

            case DialogTurnStatus.Waiting:
                // If there is an active dialog, we don't need to do anything here.
                break;
        }

        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    }

    // Processes ConversationUpdate Activities to welcome the user.
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
    {
        // Welcome new users...
    }
    else
    {
        // Give a default reply for all other activity types...
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        const dialogContext = await this.dialogs.createContext(turnContext);
        const results = await dialogContext.continueDialog();
        switch (results.status) {
            case DialogTurnStatus.cancelled:
            case DialogTurnStatus.empty:
                // If there is no active dialog, we should clear the user info and start a new dialog.
                await this.userProfileAccessor.set(turnContext, {});
                await this.userState.saveChanges(turnContext);
                await dialogContext.beginDialog(TOP_LEVEL_DIALOG);
                break;
            case DialogTurnStatus.complete:
                // If we just finished the dialog, capture and display the results.
                const userInfo = results.result;
                const status = 'You are signed up to review '
                    + (userInfo.companiesToReview.length === 0 ? 'no companies' : userInfo.companiesToReview.join(' and '))
                    + '.';
                await turnContext.sendActivity(status);
                await this.userProfileAccessor.set(turnContext, userInfo);
                await this.userState.saveChanges(turnContext);
                break;
            case DialogTurnStatus.waiting:
                // If there is an active dialog, we don't need to do anything here.
                break;
        }
        await this.conversationState.saveChanges(turnContext);
    } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
        // Welcome new users...
    } else {
        // Give a default reply for all other activity types...
    }
}
```

---

## <a name="test-your-dialog"></a>Проверка диалога

1. Выполните этот пример на локальном компьютере. Если потребуются дополнительные инструкции, изучите файл README для примера на [C#](https://aka.ms/cs-complex-dialog-sample) или [JS](https://aka.ms/js-complex-dialog-sample).
1. Примените эмулятор для тестирования бота, как показано ниже.

![Пример тестирования сложного диалога](~/media/emulator-v4/test-complex-dialog.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

Общие сведения о создании диалогов можно получить в статье [о реализации последовательного потока диалога](bot-builder-dialog-manage-conversation-flow.md), в котором на основе одного каскадного диалога и нескольких запросов создается простой пример взаимодействия, который задает пользователю набор вопросов.

Библиотека диалогов выполняет простую проверку запросов. Вы также можете добавить любую собственную проверку. Дополнительные сведения о сборе данных от пользователя с помощью запросов диалога см. в [этой статье](bot-builder-prompts.md).

Чтобы упростить код диалога и использовать его повторно в нескольких ботах, следует определить компоненты набора диалогов в отдельном классе.
Подробнее об этом см. в статье [о повторном использовании диалогов](bot-builder-compositcontrol.md).

## <a name="next-steps"></a>Дополнительная информация

Вы можете добавить в бот реакцию на другие входящие запросы, которая будет прерывать обычный поток беседы, например для получения справки или отмены действия.

> [!div class="nextstepaction"]
> [Обработка прерываний диалога пользователем](bot-builder-howto-handle-user-interrupt.md)

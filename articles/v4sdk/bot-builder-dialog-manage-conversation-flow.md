---
title: Реализация последовательной беседы | Документация Майкрософт
description: Сведения о том, как управлять простым процессом общения с помощью диалогов в пакете SDK Bot Framework для Node.js.
keywords: simple conversation flow, sequential conversation flow, dialogs, prompts, waterfalls, dialog set
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 554d040dd4c9d126fa70c24f1af5a1ac97a39204
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317624"
---
# <a name="implement-sequential-conversation-flow"></a>Реализация последовательной беседы

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

С помощью библиотеки диалогов можно управлять простыми и сложными процессами общения. В самом простом сценарии взаимодействия бот выполняет фиксированную последовательность действий и завершает диалог. В этой статье мы применим _каскадный диалог_, несколько _запросов_ и _набор диалогов_ для создания простого взаимодействия, в котором пользователю предлагаются несколько вопросов.

## <a name="prerequisites"></a>Предварительные требования
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download).
- Код в этой статье основан на примере **multi-turn-prompt**. Вам потребуется копия этого примера на языке [C#](https://aka.ms/cs-multi-prompts-sample) или [JS](https://aka.ms/js-multi-prompts-sample).
- Понимание [основных принципов работы ботов](bot-builder-basics.md), [библиотеки диалогов](bot-builder-concept-dialog.md), [состояния диалогов](bot-builder-dialog-state.md), и [файла .bot](bot-file-basics.md).


В следующих разделах описаны действия, которые вам потребуются для реализации простых диалогов для большинства ботов:

## <a name="configure-your-bot"></a>Настройка бота

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Мы будет инициализировать метод доступа к свойству состояния бота в коде конфигурации, размещенном в файле **Startup.cs**.

Мы определяем класс `MultiTurnPromptsBotAccessors` для хранения объектов управления состоянием и методов доступа к свойствам состояния бота.
Здесь мы публикуем только часть этого кода.

```csharp
public class MultiTurnPromptsBotAccessors
{
    // Initializes a new instance of the class.
    public MultiTurnPromptsBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<DialogState> ConversationDialogState { get; set; }
    public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }

    public ConversationState ConversationState { get; }
    public UserState UserState { get; }
}
```

Класс с методами доступа мы регистрируем в методе `ConfigureServices` класса `Startup`.
Здесь мы снова публикуем только часть кода.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<MultiTurnPromptsBotAccessors>(sp =>
    {
        // We need to grab the conversationState we added on the options in the previous step
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new MultiTurnPromptsBotAccessors(conversationState, userState)
        {
            ConversationDialogState = conversationState.CreateProperty<DialogState>("DialogState"),
            UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
        };

        return accessors;
    });
}
```

Внедрение зависимостей обеспечит доступ к методам доступа из кода конструктора бота.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В файле **index.js** мы определяем объекты управления состоянием.
Здесь мы публикуем только часть этого кода.

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage, UserState } = require('botbuilder');

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the main dialog, which serves as the bot's main handler.
const bot = new MultiTurnBot(conversationState, userState);
```

Конструктор бота создаст методы доступа к свойству состояния бота: `this.dialogState` и `this.userProfile`.

---

## <a name="update-the-bot-turn-handler-to-call-the-dialog"></a>Обновление обработчика шагов бота для вызова диалога

Чтобы запустить диалоговое окно, обработчик шагов бота должен создать контекст диалога для набора диалогов для бота. Бот может определить несколько наборов диалогов, но обычно для каждого бота создается только один набор. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Диалог вызывается из обработчика шагов бота. Сначала этот обработчик создает `DialogContext`, а затем продолжает активный диалог или создает новый в зависимости от ситуации. В конце шага обработчик сохраняет состояния диалога и пользователя.

В классе `MultiTurnPromptsBot` мы определили свойство `_dialogs`, содержащее набор диалогов, на основе которого мы создаем контекст диалога. Здесь мы также отображаем только часть кода из обработчика шагов.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // ...
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Run the DialogSet - let the framework identify the current state of the dialog from
        // the dialog stack and figure out what (if any) is the active dialog.
        var dialogContext = await _dialogs.CreateContextAsync(turnContext, cancellationToken);
        var results = await dialogContext.ContinueDialogAsync(cancellationToken);

        // If the DialogTurnStatus is Empty we should start a new dialog.
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync("details", null, cancellationToken);
        }
    }

    // ...
    // Save the dialog state into the conversation state.
    await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);

    // Save the user profile updates into the user state.
    await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В коде бота используются несколько классов из библиотеки диалогов.

```javascript
const { ChoicePrompt, DialogSet, NumberPrompt, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

Диалог вызывается из обработчика шагов бота. Сначала этот обработчик создает `DialogContext` (`dc`), а затем продолжает активный диалог или создает новый в зависимости от ситуации. В конце шага обработчик сохраняет состояния диалога и пользователя.

Класс `MultiTurnBot` определен в файле **bot.js**. Конструктор для этого класса добавляет свойство `dialogs` для набора диалогов, на основе которого мы создаем контекст диалога. Этот бот однократно собирает данные пользователя с помощью диалога `WHO_ARE_YOU`. Заполнив профиль пользователя, бот применяет диалог `HELLO_USER` для ответа. Здесь мы также отображаем только часть кода из обработчика шагов.

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Create a dialog context object.
        const dc = await this.dialogs.createContext(turnContext);

        const utterance = (turnContext.activity.text || '').trim().toLowerCase();

        // ...
        // If the bot has not yet responded, continue processing the current dialog.
        await dc.continueDialog();

        // Start the sample dialog in response to any other input.
        if (!turnContext.responded) {
            const user = await this.userProfile.get(dc.context, {});
            if (user.name) {
                await dc.beginDialog(HELLO_USER);
            } else {
                await dc.beginDialog(WHO_ARE_YOU);
            }
        }
    }

    // ...
    // Save changes to the user state.
    await this.userState.saveChanges(turnContext);

    // End this turn by saving changes to the conversation state.
    await this.conversationState.saveChanges(turnContext);
}
```

---

В обработчике шагов бота мы создаем контекст диалога для набора диалогов. Контекст диалога обращается к кэшу состояний для бота, "запоминая" место в беседе, на котором был остановлен последний шаг.

При активном диалоге метод _continue dialog_ контекста диалога продолжает этот диалог с использованием входных данных от пользователя, по которым был запущен очередной шаг. В противном случае бот вызывает метод _begin dialog_ контекста диалога, чтобы начать новый диалог.

Наконец, мы вызываем метод _save changes_ для объектов управления состоянием, чтобы сохранить внесенные на этом шаге изменения.

### <a name="about-dialog-and-bot-state"></a>Сведения о диалоге и состоянии бота

В этом боте мы определили два метода доступа к свойству состояния.

* Один из них создается в состоянии беседы для свойства состояния диалога. Состояние диалога отслеживает позицию пользователя в наборе диалогов. Это свойство обновляется контекстом диалога, например при вызове методов begin dialog или continue dialog.
* Второй создается в состоянии пользователя для свойства профиля пользователя. Бот использует этот метод для отслеживания сведений о пользователе. Мы управляем этим состоянием прямо в коде бота.

Методы доступа _get_ и _set_ для свойства состояния позволяют получить и сохранить значение этого свойства в кэше объекта управления состоянием. Кэш заполняется автоматически при первом обращении к значению свойства состояния в течение шага, но сохранять его нужно явным образом. Чтобы сохранить изменения в обоих свойствах состояния, мы вызываем метод _save changes_ для соответствующего объекта управления состоянием.

## <a name="initialize-your-bot-and-define-your-dialog"></a>Инициализация бота и определение диалога

Наша простая беседа составлена в виде серии вопросов, обращенных к пользователю. В версиях для C# и для JavaScript есть небольшие различия в наборе этапов:

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Запрос имени пользователя.
1. Запрос согласия сообщить свой возраст.
1. Если ответ положительный, запрашивается возраст. В противном случае этот этап пропускается.
1. Запрос подтверждения собранных сведений.
1. Отправка сообщения о состоянии и завершение работы.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Алгоритм диалога `who_are_you`:

1. Запрос имени пользователя.
1. Запрос согласия сообщить свой возраст.
1. Если ответ положительный, запрашивается возраст. В противном случае этот этап пропускается.
1. Отправка сообщения о состоянии и завершение работы.

Алгоритм диалога `hello_user`:

1. Отображение собранных ботом сведений о пользователе.

---

При определении каскадных шагов нужно помнить о нескольких важных аспектах.

* На каждом шаге бота отображаются введенные пользователем данные, а затем ответ бота. Это означает, что бот запрашивает данные у пользователя в конце этапа каскадного диалога, а ответ на этот запрос получает в начале следующего этапа каскадного диалога.
* Каждый запрос, по сути, является диалогом из двух реплик, для которого в цикле выводится своя строка запроса, пока не будут получены "допустимые" входные данные. 

В этом примере диалог определен в файле бота и инициализируется в конструкторе бота.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Укажите свойство экземпляра для набора диалогов.

```csharp
// The DialogSet that contains all the Dialogs that can be used at runtime.
private DialogSet _dialogs;
```

Внутри конструктора бота создайте набор диалогов, добавив в него запросы и каскадный диалог.

```csharp
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

В этом примере мы определяем каждый этап в виде отдельного метода. Этапы также можно определить в конструкторе с помощью лямбда-выражений.

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // WaterfallStep always finishes with the end of the Waterfall or with another dialog; here it is a Prompt Dialog.
    // Running a prompt here means the next WaterfallStep will be run when the users response is received.
    return await stepContext.PromptAsync("name", new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") }, cancellationToken);
}

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

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В этом примере каскадный диалог задается в файле **bot.js**.

Определите идентификаторы, которые будут использоваться для методов доступа к свойствам состояния, запросов и диалогов.

```javascript
const DIALOG_STATE_PROPERTY = 'dialogState';
const USER_PROFILE_PROPERTY = 'user';

const WHO_ARE_YOU = 'who_are_you';
const HELLO_USER = 'hello_user';

const NAME_PROMPT = 'name_prompt';
const CONFIRM_PROMPT = 'confirm_prompt';
const AGE_PROMPT = 'age_prompt';
```

В конструкторе бота определите и создайте набор диалогов, добавив в него запросы и каскадные диалоги.
`NumberPrompt` содержит пользовательскую проверку возраста, значение которого не может быть нулевым или отрицательным.

```javascript
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

Методы этапов в нашем диалоге используют ссылки на свойства экземпляра. Поэтому нам нужен метод `bind` для правильного разрешения объекта `this` в каждом методе шага.

В этом примере мы определяем каждый этап в виде отдельного метода. Этапы также можно определить в конструкторе с помощью лямбда-выражений.

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

---

В этом примере обновляется состояние профиля пользователя из диалога. Такой подход допустим для простого бота, но не позволит повторно использовать этот диалог в другом боте.

Есть несколько подходов, позволяющих отделить этапы диалога от состояний бота. Например, после сбора информации вы можете сделать следующее:

* Выполните метод _end dialog_, чтобы передать собранные данные в возвращаемом значении обратно в контекст родительского элемента. Это может быть обработчик шагов бота или предыдущий активный диалог из стека диалогов. Именно так разрабатываются классы запросов.
* Создайте запрос к соответствующей службе. Это может быть хорошим вариантом, если бот выступает в роли интерфейса для более крупной службы.

## <a name="test-your-dialog"></a>Проверка диалога

Скомпилируйте и запустите бот на локальном компьютере и попробуйте взаимодействовать с ним с помощью эмулятора.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

1. Бот отправляет начальное приветственное сообщение в ответ на действие обновления диалога, в котором в беседу добавляется пользователь.
1. Введите `hi` или другое значение. На этом шаге еще не существует активного диалога, поэтому бот запускает диалог `details`.
   * Бот отправляет первый запрос из этого диалога и ожидает следующих входных данных.
1. Отвечайте на вопросы, которые вам задает бот, постепенно проходя через весь диалог.
1. На последнем шаге диалог отправляет сообщение `Thanks`, основанное на введенных вами данных.
   * Завершенный диалог удаляется из стека диалогов, и у бота снова нет никаких активных диалогов.
1. Введите `hi` или любую другую строку, чтобы снова начать тот же диалог.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Бот отправляет начальное приветственное сообщение в ответ на действие обновления диалога, в котором в беседу добавляется пользователь.
1. Введите `hi` или другое значение. На этом шаге еще не существует активного диалога и профиля пользователя, поэтому бот запускает диалог `who_are_you`.
   * Бот отправляет первый запрос из этого диалога и ожидает следующих входных данных.
1. Отвечайте на вопросы, которые вам задает бот, постепенно проходя через весь диалог.
1. На последнем шаге диалог отправляет краткое подтверждающее сообщение.
1. Введите `hi` или другое значение.
   * Бот запускает одноэтапный диалог `hello_user`, в котором отображаются собранные сведения, и немедленно завершает работу.

---

## <a name="additional-resources"></a>Дополнительные ресурсы
Проверку типа для каждого запроса вы можете доверить встроенным механизмам, как показано здесь, или добавить собственные методы проверки. Дополнительные сведения о сборе данных от пользователя с помощью запросов диалога см. в [этой статье](bot-builder-prompts.md).

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Создание сложной последовательной беседы с использованием ветвей и циклов](bot-builder-dialog-manage-complex-conversation-flow.md)

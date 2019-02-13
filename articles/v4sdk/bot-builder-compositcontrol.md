---
title: Повторное использование диалогов | Документация Майкрософт
description: Узнайте, как модулировать логику бота с помощью контейнера диалогового окна в пакете SDK Bot Framework для Node.js и C#.
keywords: составной элемент управления, модульная логика бота
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 01/16/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b72ffa951e176a174dd8b00e69229b27bf28a360
ms.sourcegitcommit: 32615b88e4758004c8c99e9d564658a700c7d61f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/04/2019
ms.locfileid: "55711998"
---
# <a name="reuse-dialogs"></a>Повторное использование диалогов

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Представьте себе, что вы создаете бот для отеля, который обрабатывает множество задач, таких как приветствие пользователя, резервирование обеденного стола, заказ еды, установка сигнала будильника, отображение текущей погоды и много других. Каждую из этих задач в боте можно выполнять в одном объекте диалога, но это сделает код диалога слишком громоздким и запутанным.

Можно разделить фрагменты потока беседы на компонентные диалоги, чтобы вместо большого набора диалогов иметь дело с более удобными компонентами. Это позволит упростить код и его отладку, а также позволит нескольким командам одновременно работать над разработкой бота.
Также можно создать библиотеку компонентных диалогов и применять их в нескольких наборах диалогов и (или) в разных ботах.

В нашем примере создается бот для отеля, который содержит компонентные диалоги для сценариев регистрации, побудки и бронирования столиков.

Эта статья основана на сведениях об управлении [простыми](bot-builder-dialog-manage-conversation-flow.md) и [сложными](bot-builder-dialog-manage-complex-conversation-flow.md) потоками беседы.

## <a name="create-your-project"></a>Создание проекта

Мы начнем работу с шаблона проверки связи и добавим в него библиотеку диалогов.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В качестве основы мы применим шаблон **EchoBot**. Инструкции см. в [кратком руководстве по .NET](../dotnet/bot-builder-dotnet-sdk-quickstart.md).

Для использования диалогов установите пакет NuGet `Microsoft.Bot.Builder.Dialogs` для своего проекта или решения.
Затем при необходимости добавьте ссылку на библиотеку диалогов в файлы кода с помощью инструкций using.

Переименуйте файл **EchoBotWithCounter.cs** в **HotelBot.cs** и присвойте классу в нем новое имя **HotelBot**.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В качестве основы мы применим шаблон **Echo**. Инструкции вы найдете в [кратком руководстве по JavaScript](../javascript/bot-builder-javascript-quickstart.md).

Библиотеку `botbuilder-dialogs` можно скачать из npm. Чтобы установить библиотеку `botbuilder-dialogs`, выполните следующую команду npm:

```cmd
npm install --save botbuilder-dialogs
```

---

## <a name="managing-state"></a>Управление состоянием

Существует множество способов настройки управления состоянием для бота, который использует составные диалоговые окна. В этом боте каждый компонентный диалог возвращает в качестве результата объект. Контекст вызова определяет, какие значения будут возвращаться. Дополнительные сведения об управлении состоянием см. в разделе [Сохранение состояния с помощью свойств общения и пользователя](bot-builder-howto-v4-state.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Каждый диалог будет собирать некоторые сведения, которые сохраняются в сведениях о состоянии пользователя с помощью обработчика шагов или главного меню. Мы определим компонентные диалоги для регистрации, резервирования столиков и выбора времени для побудки. Каждый из них возвращает объект соответствующего класса. Добавьте в проект каждый из следующих классов в виде нового модуля класса C#.

```csharp
/// <summary>
/// User state information.
/// </summary>
public class UserInfo
{
    public GuestInfo Guest { get; set; }
    public TableInfo Table { get; set; }
    public WakeUpInfo WakeUp { get; set; }
}

/// <summary>
/// State information associated with the check-in dialog.
/// </summary>
public class GuestInfo
{
    public string Name { get; set; }
    public string Room { get; set; }
}

/// <summary>
/// State information associated with the reserve-table dialog.
/// </summary>
public class TableInfo
{
    public string Number { get; set; }
}

/// <summary>
/// State information associated with the wake-up call dialog.
/// </summary>
public class WakeUpInfo
{
    public string Time { get; set; }
}
```

Переименуйте **EchoBotAccessors.cs** в **BotAccessors.cs** и присвойте классу новое имя **BotAccessors**.

Затем обновите файл, разместив в нем следующий код. Нам потребуются методы доступа к состоянию диалога и к собранным сведениям о пользователе.

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>
/// Contains the state objects and the state property accessors for the bot.
/// </summary>
public class BotAccessors
{
    // The property accessor keys to use.
    public const string UserInfoAccessorName = "HotelBot.UserInfo";
    public const string DialogStateAccessorName = "HotelBot.DialogState";

    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    /// <summary>Gets or sets the state property accessor for the user information we're tracking.</summary>
    /// <value>Accessor for user information.</value>
    public IStatePropertyAccessor<UserInfo> UserInfoAccessor { get; set; }

    /// <summary>Gets or sets the state property accessor for the dialog state.</summary>
    /// <value>Accessor for the dialog state.</value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>Gets the conversation state for the bot.</summary>
    /// <value>The conversation state for the bot.</value>
    public ConversationState ConversationState { get; }

    /// <summary>Gets the user state for the bot.</summary>
    /// <value>The user state for the bot.</value>
    public UserState UserState { get; }
}
```

В файле **Startup.cs** измените код метода `ConfigureServices`, который устанавливает состояние бота, и методы доступа к свойству состояния бота.

```csharp
using Microsoft.Bot.Builder.Dialogs;

public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<HotelBot>(options =>
    {
        //...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create conversation and user state objects.
        options.State.Add(new ConversationState(dataStore));
        options.State.Add(new UserState(dataStore));
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            UserInfoAccessor = userState.CreateProperty<UserInfo>(BotAccessors.UserInfoAccessorName),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Каждый диалог будет собирать некоторые сведения, которые сохраняются в сведениях о состоянии пользователя с помощью обработчика шагов или главного меню. Мы определим компонентные диалоги для регистрации, резервирования столиков и выбора времени для побудки. Каждый из них возвращает объект с соответствующими свойствами. Эти свойства мы соберем для бота в пользовательском состоянии.

В файле **bot.js** обновите конструктор бота, чтобы он создавал методы доступа к свойствам состояния для отслеживания состояний пользователя и диалога.

```javascript
// Define the identifiers for our state property accessors.
const DIALOG_STATE_PROPERTY = 'dialogStatePropertyAccessor';
const USER_INFO_PROPERTY = 'userInfoPropertyAccessor';

constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);
}
```

В файле **index.js** измените классы, импортированные из библиотеки `botbuilder`, и код создания объектов состояния и самого бота:

```javascript
// Import required bot services.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();

// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

---

## <a name="about-component-dialogs"></a>Сведения о компонентных диалогах

Компонентный диалог позволяет создавать независимые диалоги для обработки определенных сценариев, разбивая большие наборы диалогов на более управляемые компоненты. Каждый из этих компонентов имеет отдельный набор диалогов, что позволяет избежать конфликтов имен с родительским набором диалогов.

Используйте метод _add dialog_, чтобы добавлять диалоги и запросы в компонентный диалог.
Первый элемент, добавленный с помощью этого метода, настраивается как начальный диалог. Но это можно изменить, явно задав свойство `InitialDialogId` в конструкторе компонентного диалога.
При запуске компонентного диалога будет запущен _начальный диалог_.

## <a name="define-the-check-in-component-dialog"></a>Определение компонентного диалога для регистрации

Начнем с простого диалога регистрации, который будет запрашивать у пользователей имя и комнату, в которой они хотят остановиться. Мы создаем класс `ComponentDialog`, расширяющий класс `CheckInDialog`. Этот класс содержит конструктор, который определяет имя корневого диалога, и мы определим его как каскадный диалог с тремя шагами.

Ниже приведены шаги для диалога регистрации.

1. Спросить имя гостя.
1. Спросить номер комнаты, в которой они хотели бы остановиться.
1. Отправить сообщение с подтверждением и закончить диалог.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Добавьте в проект класс `CheckInDialog` с приведенным ниже кодом.

В этом диалоге мы можем сохранять данные в локальный объект состояния, доступный через свойство `Values` в контексте шага. Когда диалог завершается, локальный объект состояния удаляется. Таким образом, мы возвращаем из диалога значение с данными о госте.

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class CheckInDialog : ComponentDialog
{
    private const string InitialId = "mainDialog";
    private const string GuestKey = nameof(CheckInDialog);
    private const string TextPrompt = "textPrompt";

    // You can start this from the parent using the ID assigned in the parent.
    public CheckInDialog(string id) : base(id)
    {
        InitialDialogId = InitialId;

        // Define the prompts used in this conversation flow.
        AddDialog(new TextPrompt(TextPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
            NameStepAsync,
            RoomStepAsync,
            FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(InitialId, waterfallSteps));
    }

    private static async Task<DialogTurnResult> NameStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Clear the guest information and prompt for the guest's name.
        step.Values[GuestKey] = new GuestInfo();
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What is your name?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> RoomStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the name and prompt for the room number.
        string name = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Name = name;
        return await step.PromptAsync(
            TextPrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text($"Hi {name}. What room will you be staying in?"),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Save the room number and "sign off".
        string room = step.Result as string;
        ((GuestInfo)step.Values[GuestKey]).Room = room;

        await step.Context.SendActivityAsync(
            "Great, enjoy your stay!",
            cancellationToken: cancellationToken);

        // End the dialog, returning the guest info.
        return await step.EndDialogAsync(
            (GuestInfo)step.Values[GuestKey],
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Создайте файл **checkInDialog.js** и добавьте в него класс `CheckInDialog`, расширяющий класс `ComponentDialog`.

В этом диалоге мы можем сохранять данные в локальный объект состояния, доступный через свойство `values` в контексте шага. Когда диалог завершается, локальный объект состояния удаляется. Таким образом, мы возвращаем из диалога значение с данными о госте.

```JavaScript
const { ComponentDialog, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'mainDialog';

class CheckInDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new TextPrompt('textPrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(initialId, [
            async function (step) {
                // Clear the guest information and prompt for the guest's name.
                step.values.guestInfo = {};
                return await step.prompt('textPrompt', "What is your name?");
            },
            async function (step) {
                // Save the name and prompt for the room number.
                step.values.guestInfo.userName = step.result;
                return await step.prompt('textPrompt', `Hi ${step.result}. What room will you be staying in?`);
            },
            async function (step) {
                // Save the room number and "sign off".
                step.values.guestInfo.roomNumber = step.result;
                await step.context.sendActivity(`Great! Enjoy your stay in room ${step.result}!`);

                // End the dialog, returning the guest info.
                return await step.endDialog(step.values.guestInfo);
            }
        ]));
    }
}

exports.CheckInDialog = CheckInDialog;
```

---

## <a name="define-the-reserve-table-and-wake-up-component-dialogs"></a>Определение компонентных диалогов резервирования столиков и побудки

Одним из преимуществ использования компонентного диалога является возможность совместно использовать несколько диалогов. Поскольку каждое `DialogSet` поддерживает уникальный набор `dialogs`, совместно использовать или создать перекрестную ссылку `dialogs` не так уж и просто. В таких случаях следует использовать компонентные диалоги. Вы можете распределить сложные и многогранные аспекты потоков беседы по нескольким компонентным диалогам, что упростит управление диалогами и их обслуживание. Мы создадим еще два компонентных диалога: один для выбора столика, бронируемого на обед, и другой для создания сигнала побудки. Мы снова будем использовать отдельный класс для каждого диалога, которые будут расширениями главного класса `ComponentDialog`.

При запуске они будут принимать информацию от гостя в виде параметров для выбора.

Ниже приведены шаги для диалога бронировании столика.

1. Спросить, какой столик забронировать.
1. Отправить сообщение с подтверждением и закончить диалог, сообщив номер столика.

Ниже приведены шаги для диалога побудки.

1. Спросить время, на которое нужно назначить уведомление.
1. Отправить сообщение с подтверждением и закончить диалог, сообщив время побудки.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Добавьте в проект класс `ReserveTableDialog` с приведенным ниже кодом.

Мы получим имя гостя из объекта параметров, переданного при запуске диалога. Затем мы вернем из диалога одно значение, которое обозначает номер столика.

```csharp
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class ReserveTableDialog : ComponentDialog
{
    private const string InitialId = "mainDialog";
    private const string TablePrompt = "choicePrompt";

    public ReserveTableDialog(string id) : base(id)
    {
        InitialDialogId = InitialId;

        // Define the prompts used in this conversation flow.
        AddDialog(new ChoicePrompt(TablePrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                TableStepAsync,
                FinalStepAsync,
        };
        AddDialog(new WaterfallDialog(InitialId, waterfallSteps));
    }

    private static async Task<DialogTurnResult> TableStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Welcome {guest.Name}" : "Welcome";

        string prompt = $"{greeting}, How many diners will be at your table?";
        string[] choices = new string[] { "1", "2", "3", "4", "5", "6" };
        return await step.PromptAsync(
            TablePrompt,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(prompt),
                Choices = ChoiceFactory.ToChoices(choices),
            },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // ChoicePrompt returns a FoundChoice object.
        string table = (step.Result as FoundChoice).Value;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Sounds great;  we will reserve a table for you for {table} diners.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the table info.
        return await step.EndDialogAsync(
            new TableInfo { Number = table },
            cancellationToken);
    }
}
```

Добавьте в проект класс `SetAlarmDialog` с приведенным ниже кодом.

Мы получим номер комнаты из объекта параметров, переданного при запуске диалога. Затем мы вернем из диалога одно значение, которое обозначает время побудки этого гостя.

```csharp
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

public class SetAlarmDialog : ComponentDialog
{
    private const string InitialId = "mainDialog";
    private const string AlarmPrompt = "dateTimePrompt";

    public SetAlarmDialog(string id) : base(id)
    {
        InitialDialogId = InitialId;

        // Define the prompts used in this conversation flow.
        // Ideally, we'd add validation to this prompt.
        AddDialog(new DateTimePrompt(AlarmPrompt));

        // Define the conversation flow using a waterfall model.
        WaterfallStep[] waterfallSteps = new WaterfallStep[]
        {
                AlarmStepAsync,
                FinalStepAsync,
        };

        AddDialog(new WaterfallDialog(InitialId, waterfallSteps));
    }

    private static async Task<DialogTurnResult> AlarmStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        string greeting = step.Options is GuestInfo guest
                && !string.IsNullOrWhiteSpace(guest?.Name)
                ? $"Hi {guest.Name}" : "Hi";

        string prompt = $"{greeting}. When would you like your alarm set for?";
        return await step.PromptAsync(
            AlarmPrompt,
            new PromptOptions { Prompt = MessageFactory.Text(prompt) },
            cancellationToken);
    }

    private static async Task<DialogTurnResult> FinalStepAsync(
        WaterfallStepContext step,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Ambiguous responses can generate multiple results.
        var resolution = (step.Result as IList<DateTimeResolution>)?.FirstOrDefault();

        // Time ranges have a start and no value.
        var alarm = resolution.Value ?? resolution.Start;
        string roomNumber = (step.Options as GuestInfo)?.Room;

        // Send a confirmation message.
        await step.Context.SendActivityAsync(
            $"Your alarm is set to {alarm} for room {roomNumber}.",
            cancellationToken: cancellationToken);

        // End the dialog, returning the alarm info.
        return await step.EndDialogAsync(
            new WakeUpInfo { Time = alarm },
            cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Создайте файл **reserveTableDialog.js** и добавьте в него класс `ReserveTableDialog`, расширяющий класс `ComponentDialog`.

Мы получим имя гостя из объекта параметров, переданного при запуске диалога. Затем мы вернем из диалога одно значение, которое обозначает номер столика.

```JavaScript
const { ComponentDialog, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'mainDialog';

class ReserveTableDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new ChoicePrompt('choicePrompt'));

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(initialId, [
            async function (step) {
                // Welcome the user and ask for their table preference.
                const greeting = step.options && step.options.userName ? `Welcome ${step.options.userName}` : `Welcome`;

                const promptOptions = {
                    prompt: `${greeting}, How many diners will be at your table?`,
                    reprompt: 'That was not a valid choice, please select a table size between 1 and 6 guests.',
                    choices: ['1', '2', '3', '4', '5', '6']
                };
                return await step.prompt('choicePrompt', promptOptions);
            },
            async function (step) {
                const choice = step.result;

                // Send a confirmation message.
                const tableNumber = choice.value;
                await step.context.sendActivity(`Sounds great, we will reserve a table for you for ${tableNumber} diners.`);

                // End the dialog, returning the table info.
                return await step.endDialog({ table: tableNumber });
            }
        ]));
    }
}

exports.ReserveTableDialog = ReserveTableDialog;
```

Создайте файл **setAlarmDialog.js** и добавьте в него класс `SetAlarmDialog`, расширяющий класс `ComponentDialog`.

Мы получим номер комнаты из объекта параметров, переданного при запуске диалога. Затем мы вернем из диалога одно значение, которое обозначает время побудки этого гостя.

```JavaScript
const { ComponentDialog, DateTimePrompt, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'mainDialog';

class SetAlarmDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the prompts used in this conversation flow.
        this.addDialog(new DateTimePrompt('datePrompt'));

        this.addDialog(new WaterfallDialog(initialId, [
            async function (step) {
                step.values.wakeUp = {};
                if (step.options && step.options.roomNumber) {
                    step.values.roomNumber = step.options.roomNumber;
                }

                const greeting = step.options && step.options.userName ? `Hi ${step.options.userName}` : `Hi`;
                return await step.prompt('datePrompt', `${greeting}. What time would you like your alarm to be set?`);
            },
            async function (step) {
                // Ambiguous responses can generate multiple results.
                const resolution = step.result[0];

                // Time ranges have a start and no value.
                const alarm = resolution.value ? resolution.value : resolution.start;
                const roomNumber = step.values.roomNumber;

                // Send a confirmation message.
                await step.context.sendActivity(`Your alarm is set to ${alarm} for room ${roomNumber}.`);

                // End the dialog, returning the alarm info.
                return await step.endDialog({ alarm: alarm });
            }]));

        // Defining the prompt used in this conversation flow
    }
}

exports.SetAlarmDialog = SetAlarmDialog;
```

---

## <a name="add-the-component-dialogs-to-the-bot"></a>Добавление компонентных диалогов в бот

Теперь, когда мы определили три компонентных диалога, их можно применить в нашем боте.

- Все три диалога добавляются в основной набор диалогов бота.
- Когда начинается новое общение, у нас нет активного диалогового окна, и начинает работать логика запуска бота.
- Если у нас нет информации об этом госте, запускается диалог регистрации.
- После получения сведений о госте управление передается главному диалогу, который в цикле предлагает пользователю начать диалог резервирования столика или диалог побудки.

Мы обновим логику обработчика шагов в боте.

1. Получить состояние пользователя.
1. Продолжить активный диалог.
1. Если диалог завершился на этом шаге, значит это был диалог регистрации.
   1. Сохранить сведения о госте.
   1. Запустить главный диалог.
1. Если бот еще не отправил пользователю сообщение, значит сейчас нет активного диалога.
    1. Если у нас нет информации об этом госте, запустить диалог регистрации.
    1. В остальных случаях запустить главный диалог.
1. Сохранить все произошедшие изменения состояния.

Ниже приведены шаги для главного диалога.

1. Спросить гостей, что они хотели бы сделать: забронировать столик или установить сигнал уведомления.
1. Запустить соответствующий дочерний диалог или отправить сообщение _введенные данные не удалось распознать_ и повторить процесс с первого шага.
1. Обработать возвращаемое из дочернего диалога значение и снова запустить главный диалог.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В файле **HotelBot.cs** обновите инструкции using.

```csharp
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;
using Microsoft.Extensions.Logging;
```

Обновите код инициализации и определите набор диалогов и главный диалог.

```csharp
// Define the IDs for the dialogs in the bot's dialog set.
private const string MainDialogId = "mainDialog";
private const string CheckInDialogId = "checkInDialog";
private const string TableDialogId = "tableDialog";
private const string AlarmDialogId = "alarmDialog";

// Define the dialog set for the bot.
private readonly DialogSet _dialogs;

// Define the state accessors and the logger for the bot.
private readonly BotAccessors _accessors;
private readonly ILogger _logger;

/// <summary>
/// Initializes a new instance of the <see cref="HotelBot"/> class.
/// </summary>
/// <param name="accessors">Contains the objects to use to manage state.</param>
/// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    if (loggerFactory == null)
    {
        throw new System.ArgumentNullException(nameof(loggerFactory));
    }

    _logger = loggerFactory.CreateLogger<HotelBot>();
    _logger.LogTrace($"{nameof(HotelBot)} turn start.");
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Define the steps of the main dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        MenuStepAsync,
        HandleChoiceAsync,
        LoopBackAsync,
    };

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    _dialogs = new DialogSet(_accessors.DialogStateAccessor)
        .Add(new WaterfallDialog(MainDialogId, steps))
        .Add(new CheckInDialog(CheckInDialogId))
        .Add(new ReserveTableDialog(TableDialogId))
        .Add(new SetAlarmDialog(AlarmDialogId));
}

private static async Task<DialogTurnResult> MenuStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Present the user with a set of "suggested actions".
    List<string> menu = new List<string> { "Reserve Table", "Wake Up" };
    await stepContext.Context.SendActivityAsync(
        MessageFactory.SuggestedActions(menu, "How can I help you?"),
        cancellationToken: cancellationToken);
    return Dialog.EndOfTurn;
}

private async Task<DialogTurnResult> HandleChoiceAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Since the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    string choice = (stepContext.Result as string)?.Trim()?.ToLowerInvariant();
    switch (choice)
    {
        case "reserve table":
            return await stepContext.BeginDialogAsync(TableDialogId, userInfo.Guest, cancellationToken);

        case "wake up":
            return await stepContext.BeginDialogAsync(AlarmDialogId, userInfo.Guest, cancellationToken);

        default:
            // If we don't recognize the user's intent, start again from the beginning.
            await stepContext.Context.SendActivityAsync(
                "Sorry, I don't understand that command. Please choose an option from the list.");
            return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
    }
}

private async Task<DialogTurnResult> LoopBackAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Get the user's info. (Because the type factory is null, this will throw if state does not yet have a value for user info.)
    UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(stepContext.Context, null, cancellationToken);

    // Process the return value from the child dialog.
    switch (stepContext.Result)
    {
        case TableInfo table:
            // Store the results of the reserve-table dialog.
            userInfo.Table = table;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        case WakeUpInfo alarm:
            // Store the results of the set-wake-up-call dialog.
            userInfo.WakeUp = alarm;
            await _accessors.UserInfoAccessor.SetAsync(stepContext.Context, userInfo, cancellationToken);
            break;
        default:
            // We shouldn't get here, since these are no other branches that get this far.
            break;
    }

    // Restart the main menu dialog.
    return await stepContext.ReplaceDialogAsync(MainDialogId, null, cancellationToken);
}
```

Также обновите обработчик шагов в боте, чтобы использовать новый набор диалогов.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Establish dialog state from the conversation state.
        DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        // Get the user's info.
        UserInfo userInfo = await _accessors.UserInfoAccessor.GetAsync(turnContext, () => new UserInfo(), cancellationToken);

        // Continue any current dialog.
        DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync();

        // Process the result of any complete dialog.
        if (dialogTurnResult.Status is DialogTurnStatus.Complete)
        {
            switch (dialogTurnResult.Result)
            {
                case GuestInfo guestInfo:
                    // Store the results of the check-in dialog.
                    userInfo.Guest = guestInfo;
                    await _accessors.UserInfoAccessor.SetAsync(turnContext, userInfo, cancellationToken);
                    break;
                default:
                    // We shouldn't get here, since the main dialog is designed to loop.
                    break;
            }
        }

        // Every dialog step sends a response, so if no response was sent,
        // then no dialog is currently active.
        else if (!turnContext.Responded)
        {
            if (string.IsNullOrEmpty(userInfo.Guest?.Name))
            {
                // If we don't yet have the guest's info, start the check-in dialog.
                await dc.BeginDialogAsync(CheckInDialogId, null, cancellationToken);
            }
            else
            {
                // Otherwise, start our bot's main dialog.
                await dc.BeginDialogAsync(MainDialogId, null, cancellationToken);
            }
        }

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
        await _accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В файл бота **bot.js** для нашего примера нужно импортировать не только используемые классы из пакета SDK, но и те классы, которые мы определили для компонентных диалогов.

```JavaScript
const { ActivityTypes, MessageFactory } = require('botbuilder');
const { DialogSet, WaterfallDialog, Dialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Import our component dialogs.
const { CheckInDialog } = require("./checkInDialog");
const { ReserveTableDialog } = require("./reserveTableDialog");
const { SetAlarmDialog } = require("./setAlarmDialog");
```

Кроме того, нам понадобится создать набор диалогов и добавить в него все используемые диалоги.

Мы определяем каскадные действия главного диалога в виде функций класса, а не в блоке кода. Мы применяем для этих функций `bind()`, чтобы внутри них корректно разрешался `this`.

Так будет выглядеть обновленный конструктор бота.

```JavaScript
constructor(conversationState, userState) {
    // Record the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;

    // Create our state property accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.userInfoAccessor = userState.createProperty(USER_INFO_PROPERTY);

    // Create our bot's dialog set, adding a main dialog and the three component dialogs.
    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new CheckInDialog('checkInDialog'))
        .add(new ReserveTableDialog('reserveTableDialog'))
        .add(new SetAlarmDialog('setAlarmDialog'))
        .add(new WaterfallDialog('mainDialog', [
            this.promptForChoice.bind(this),
            this.startChildDialog.bind(this),
            this.saveResult.bind(this)
    ]));
}
```

Ниже кода конструктора бота добавьте следующий код, который реализует действия главного диалога.

```JavaScript
async promptForChoice(step) {
    const menu = ["Reserve Table", "Wake Up"];
    await step.context.sendActivity(MessageFactory.suggestedActions(menu, 'How can I help you?'));
    return Dialog.EndOfTurn;
}

async startChildDialog(step) {
    // Get the user's info.
    const user = await this.userInfoAccessor.get(step.context);
    // Check the user's input and decide which dialog to start.
    // Pass in the guest info when starting either of the child dialogs.
    switch (step.result) {
        case "Reserve Table":
            return await step.beginDialog('reserveTableDialog', user.guestInfo);
            break;
        case "Wake Up":
            return await step.beginDialog('setAlarmDialog', user.guestInfo);
            break;
        default:
            await step.context.sendActivity("Sorry, I don't understand that command. Please choose an option from the list.");
            return await step.replaceDialog('mainDialog');
            break;
    }
}

async saveResult(step) {
    // Process the return value from the child dialog.
    if (step.result) {
        const user = await this.userInfoAccessor.get(step.context);
        if (step.result.table) {
            // Store the results of the reserve-table dialog.
            user.table = step.result.table;
        } else if (step.result.alarm) {
            // Store the results of the set-wake-up-call dialog.
            user.alarm = step.result.alarm;
        }
        await this.userInfoAccessor.set(step.context, user);
    }
    // Restart the main menu dialog.
    return await step.replaceDialog('mainDialog'); // Show the menu again
}
```

Теперь обновите обработчик шагов бота:

```JavaScript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        const user = await this.userInfoAccessor.get(turnContext, {});
        const dc = await this.dialogs.createContext(turnContext);
        const dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            user.guestInfo = dialogTurnResult.result;
            await this.userInfoAccessor.set(turnContext, user);
            await dc.beginDialog('mainDialog');
        } else if (!turnContext.responded) {
            if (!user.guestInfo) {
                await dc.beginDialog('checkInDialog');
            } else {
                await dc.beginDialog('mainDialog');
            }
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

Как вы видите, компонентные диалоги добавляются в главный диалог бота так же, как и [запросы](bot-builder-prompts.md) в диалог. В основной диалог можно добавить сколько угодно дочерних диалогов. Каждый модуль добавит дополнительные возможности и услуги, которые бот может предложить пользователям.

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знаете, как использовать компонентные диалоги, и готовы рассмотреть способы использования Интеллектуальной службы распознавания речи (LUIS), которые помогут боту правильно выбрать момент для начала диалогов.

> [!div class="nextstepaction"]
> [Use LUIS for Language Understanding](./bot-builder-howto-v4-luis.md) (Использование LUIS для Распознавания речи)

---
title: Управление сложным процессом общения с помощью диалогов | Документация Майкрософт
description: Сведения о том, как управлять сложным процессом общения с помощью диалогов в пакете SDK Bot Builder для Node.js.
keywords: complex conversation flow, repeat, loop, menu, dialogs, prompts, waterfalls, dialog set
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/03/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 871bfd9f8d693c5082fe1ccf38349f4d3d46ece2
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326571"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>Управление сложными процессами общения с помощью диалогов

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

В последней статье мы продемонстрировали применение библиотеки диалогов для управления простыми сеансами беседы. В рамках [простого процесса общения](bot-builder-dialog-manage-conversation-flow.md) пользователь выполняет все действия в *каскадной последовательности*, после чего обмен сообщениями прекращается. В этой статье мы будем использовать диалоги для управления более сложными диалогами, части которого могут ответвляться и образовывать цикл. Для этого мы будем применять разные методы, определенные для контекста диалогов и контекста каскадных шагов, и передавать аргументы между разными частями диалога.

Дополнительные сведения о диалогах см. в [описании библиотеки диалогов](bot-builder-concept-dialog.md).

Чтобы обеспечить больше возможностей для управления *стеком диалогов*, в библиотеке **диалогов** предоставляется метод _замены диалогов_. Этот метод позволяет переключить активный диалог на другой, сохраняя состояние и поток беседы. Методы _начала диалога_ и _замены диалога_ позволяют организовать ветвление и циклы для создания таких сложных взаимодействий при необходимости. Если сложность беседы достигнет такого уровня, на котором каскадные диалоги с трудом поддаются управлению, попробуйте применить [компонентные диалоги](bot-builder-compositcontrol.md) или создайте пользовательский класс для управления диалогами на основе базового класса `Dialog`.

В этой статье мы создадим пример диалогов для бота, выполняющего роль консьержа в гостинице. Он предоставит гостю доступ к нескольким популярным возможностям: забронировать столик в ресторане гостиницы и заказать еду в номер.  Каждая из этих функций, а также связывающее их меню, будут созданы в форме диалогов, собранных в набор диалогов.

Эти два варианта предоставляются гостю в диалоге верхнего уровня. Если гость хочет забронировать столик, диалог верхнего уровня выполняет асинхронный метод _begin dialog_, чтобы запустить диалог бронирования столиков. Если же гость хочет заказать обслуживание в номер, диалог верхнего уровня начнет диалог заказа ужина.

В рамках этого диалога сначала пользователю предлагается меню для выбора блюд, а затем запрашивается номер комнаты. Выбор блюд _также_ является диалогом — он подключается к процессу многократно, когда гость выбирает из меню блюда для отправки заказа.

На схеме ниже показаны диалоги, которые мы создадим в рамках этой статьи, и связь между ними.

![Схема с диалогами, используемыми в рамках этой статьи](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>Создание ветвей

В контексте диалога содержится _стек диалогов_, и для каждого диалога из стека отслеживается, какой шаг будет следующим. Чтобы поместить диалог на вершину стека, используется метод _begin dialog_, а для извлечения диалога с вершины стека используется метод _end dialog_.

Чтобы организовать ветвление, выберите один стартовый набор дочерних диалогов. Дополнительные сведения о концепции ветвления беседы см. [здесь](bot-builder-concept-dialog.md#branch-a-conversation).

## <a name="how-to-loop"></a>Создание цикла

С помощью метода _replace dialog_ из контекста диалога можно заменить диалог, который находится на вершине стека. Состояние предыдущего диалога будет отклонено, и новый диалог начнется с начала. Этот метод можно использовать для создания цикла, в котором диалог будет заменяться собой. Но чтобы [сохранить собранную ботом информацию](bot-builder-howto-v4-state.md), вам потребуется передать эти сведения новому экземпляру диалога при вызове метода _replace dialog_.

В примере ниже вы увидите, что заказ для обслуживания в номере хранится в данных о состоянии диалога, и новой версии диалога `orderPrompt` передается текущее состояние диалога, чтобы он мог просто добавлять элементы в заказ. Если вы предпочитаете хранить эти сведения в состоянии бота за пределами диалога, изучите метод [сохранения данных пользователя](bot-builder-tutorial-persist-user-inputs.md).

## <a name="create-the-dialogs-for-the-hotel-bot"></a>Создание диалогов для бота гостиницы

В этом разделе объясняется, как создавать диалоги, чтобы управлять процессом общения с описанным выше ботом гостиницы.

### <a name="install-the-dialogs-library"></a>Установка библиотеки диалогов

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Мы начнем с базового шаблона EchoBot. Инструкции см. в [кратком руководстве по .NET](../dotnet/bot-builder-dotnet-sdk-quickstart.md).

Для использования диалогов установите пакет NuGet `Microsoft.Bot.Builder.Dialogs` для своего проекта или решения.
Затем при необходимости добавьте ссылку на библиотеку диалогов в файлы кода с помощью инструкций using.

```csharp
using Microsoft.Bot.Builder.Dialogs;
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В качестве основы мы применим шаблон EchoBot. Инструкции вы найдете в [кратком руководстве по JavaScript](../javascript/bot-builder-javascript-quickstart.md).



Библиотеку `botbuilder-dialogs` можно скачать из npm. Чтобы установить библиотеку `botbuilder-dialogs`, выполните следующую команду npm:

```cmd
npm install --save botbuilder-dialogs
```

Чтобы использовать **диалоги** в боте, включите эту библиотеку в код бота. Например, добавьте в файл **index.js** следующее:

```javascript
const { DialogSet } = require('botbuilder-dialogs');
```

А в файл **bot.js** вот это:

```javascript
const { DialogSet, NumberPrompt, ChoicePrompt, WaterfallDialog } = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>Создание набора диалогов

Создайте _набор диалогов_, в который мы добавим все диалоги для этого примера.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Создайте класс **HotelDialogs** и добавьте инструкции using, которые нам потребуются.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;
using Microsoft.Bot.Schema;
```

Наследуйте этот класс от **DialogSet**. Добавьте конструктор, который принимает параметр `IStatePropertyAccessor<DialogState>`, чтобы управлять внутренним состоянием экземпляра набора диалогов. Также определите идентификаторы и ключи, которые потребуются нам для идентификации диалогов, запросов и сведений о состоянии для этого набора диалогов.

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialogs : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

    public HotelDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }

    /// <summary>Contains the IDs for the other dialogs in the set.</summary>
    private static class Dialogs
    {
        public const string OrderDinner = "orderDinner";
        public const string OrderPrompt = "orderPrompt";
        public const string ReserveTable = "reserveTable";
    }

    /// <summary>Contains the IDs for the prompts used by the dialogs.</summary>
    private static class Inputs
    {
        public const string Choice = "choicePrompt";
        public const string Number = "numberPrompt";
    }

    /// <summary>Contains the keys used to manage dialog state.</summary>
    private static class Outputs
    {
        public const string OrderCart = "orderCart";
        public const string OrderTotal = "orderTotal";
        public const string RoomNumber = "roomNumber";
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В файл **index.js** добавьте код, который создает метод доступа к свойству состояния, чтобы управлять состоянием диалога, и с его помощью создайте набор диалогов для работы с ботом.

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const dialogStateAccessor = conversationState.createProperty('dialogState');

// Create a dialog set for the bot.
const dialogSet = new DialogSet(dialogStateAccessor);

// Create the bot.
const bot = new MyBot(conversationState, dialogSet)
```

Затем обновите вызов метода обработки действий, чтобы использовать в нем объект бота.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to the bot's turn handler.
        await bot.onTurn(context);
    });
});
```

---

### <a name="add-the-prompts-to-the-set"></a>Добавление запросов в набор

Мы будем использовать класс **ChoicePrompt**, чтобы спросить у гостя, будет ли он заказывать ужин или резервировать столик, а также чтобы предложить меню для выбора блюд на ужин. А класс **NumberPrompt** мы будем использовать, чтобы узнать у гостя номер комнаты, если он решил заказать ужин.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В конструктор **HotelDialogs** добавьте два запроса.

```csharp
// Add the prompts.
Add(new ChoicePrompt(Inputs.Choice));
Add(new NumberPrompt<int>(Inputs.Number));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Измените конструктор бота, добавив к набору диалогов два запроса.

```javascript
constructor(conversationState, dialogSet) {
    // Creates a new state accessor property.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
    this.conversationState = conversationState;
    this.dialogSet = dialogSet;

    this.dialogSet.add(new ChoicePrompt('choicePrompt'));
    this.dialogSet.add(new NumberPrompt('numberPrompt'));
}
```

---

### <a name="define-some-of-the-supporting-information"></a>Определение некоторых вспомогательных сведений

Так как нам потребуются сведения о каждом варианте в меню ужина, давайте укажем их сейчас.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Создайте внутренний статический класс **Lists**, который будет содержать эти сведения. Также давайте создадим внутренние классы **WelcomeChoice** и **MenuChoice**, которые будут содержать сведения о каждом варианте.

Пока мы здесь, добавим варианты для списка в приветственном диалоге верхнего уровня, а также создадим вспомогательные списки, которые будем использовать позже, запрашивая у гостей эти сведения. Мы немного усложнили начальную часть, но это поможет упростить код диалога.

```csharp
/// <summary>Describes an option for the top-level dialog.</summary>
private class WelcomeChoice
{
    /// <summary>Gets or sets the text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>Gets or sets the ID of the associated dialog for this option.</summary>
    public string DialogName { get; set; }
}

/// <summary>Describes an option for the food-selection dialog.</summary>
/// <remarks>We have two types of options. One represents meal items that the guest
/// can add to their order. The other represents a request to process or cancel the
/// order.</remarks>
private class MenuChoice
{
    /// <summary>The request text for cancelling the meal order.</summary>
    public const string Cancel = "Cancel order";

    /// <summary>The request text for processing the meal order.</summary>
    public const string Process = "Process order";

    /// <summary>Gets or sets the name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>Gets or sets the price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>Gets the text to show the guest for this option.</summary>
    public string Description => double.IsNaN(Price) ? Name : $"{Name} - ${Price:0.00}";
}
```

```csharp
/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>Gets the options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static readonly List<string> _welcomeList = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the top-level dialog.</summary>
    public static IList<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(_welcomeList);

    /// <summary>Gets the reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_welcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>Gets the options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static readonly List<string> _menuList = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>Gets the choices to present in the choice prompt for the food-selection dialog.</summary>
    public static IList<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(_menuList);

    /// <summary>Gets the reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(_menuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В файле **bot.js** создайте константу **dinnerMenu**, которая будет содержать эти сведения.

```javascript
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel"],
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
```

---

### <a name="create-the-welcome-dialog"></a>Создание приветственного диалога

Этот диалог использует метод `ChoicePrompt` для отображения меню, а затем ожидает выбор пользователя. Когда пользователь выбирает `Order dinner` или `Reserve a table`, он запускает соответствующий диалог. Когда дочерний диалог завершается, запускается новый цикл диалога `mainMenu` и начинается диалог `mainMenu`, чтобы не озадачить пользователя внезапным и молчаливым завершением на последнем шаге диалога. Бот с такой простой структурой всегда будет выводить меню, и пользователь всегда будете знать, что он выбрал.

Диалог `mainMenu` содержит такую каскадную последовательность шагов:

* На первом шаге каскадной последовательности мы инициализируем или удаляем состояние диалога, приветствуем гостя и просим его выбрать один из доступных вариантов: `Order dinner` или `Reserve a table`.
* На втором шаге мы получаем выбор пользователя и запускаем дочерний диалог, связанный с выбранным вариантом. Когда дочерний диалог завершается, возобновляется диалог со следующим действием.
* На последнем шаге мы используем метод _replace dialog_, чтобы заменить этот диалог его же новым экземпляром, что фактически превращает приветственный диалог в цикл вывода меню.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Добавьте в конструктор каскадный диалог. Затем определите каскадные действия. Здесь мы разместили их во вложенном классе.

В конструкторе **HotelDialogs**.

```csharp
// Define the steps for and add the main welcome dialog.
WaterfallStep[] welcomeDialogSteps = new WaterfallStep[]
{
    MainDialogSteps.PresentMenuAsync,
    MainDialogSteps.ProcessInputAsync,
    MainDialogSteps.RepeatMenuAsync,
};

Add(new WaterfallDialog(MainMenu, welcomeDialogSteps));
```

В классе **HotelDialogs** добавьте определения шагов.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class MainDialogSteps
{
    public static async Task<DialogTurnResult> PresentMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Greet the guest and ask them to choose an option.
        await stepContext.Context.SendActivityAsync(
            "Welcome to Contoso Hotel and Resort.",
            cancellationToken: cancellationToken);
        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("How may we serve you today?"),
                RetryPrompt = Lists.WelcomeReprompt,
                Choices = Lists.WelcomeChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)stepContext.Result;
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        return await stepContext.BeginDialogAsync(dialogId, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> RepeatMenuAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Start this dialog over again.
        return await stepContext.ReplaceDialogAsync(MainMenu, null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте в конструктор бота каскадный диалог `mainMenu`.

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
this.dialogSet.add(new WaterfallDialog('mainMenu', [
    async function (step) {
        // Welcome the user and send a prompt.
        await step.context.sendActivity("Welcome to Contoso Hotel and Resort.");
        return await step.prompt('choicePrompt', "How may we serve you today?", ['Order Dinner', 'Reserve a table']);
    },
    async function (step) {
        // Handle the user's response to the previous prompt and branch the dialog.
        if (step.result.value.match(/order dinner/ig)) {
            return await step.beginDialog('orderDinner');
        } else if (step.result.value.match(/reserve a table/ig)) {
            return await step.beginDialog('reserveTable');
        }
    },
    async function (step) {
        // Calling replaceDialog will loop the main menu
        return await step.replaceDialog('mainMenu');
    }
]));
```

---

### <a name="create-the-order-dinner-dialog"></a>Создание диалога для заказа ужина

В диалоге для заказа ужина мы приветствуем гостя в службе заказа, после чего сразу запускается диалог для выбора блюд, который мы рассмотрим в следующем разделе. Важно отметить, что если гость просит службу обработать заказ, в диалоге возвращается список заказанных блюд. Чтобы завершить процесс, диалог запрашивает номер комнаты, в которую нужно доставить заказ, а затем отправляет сообщение с подтверждением. Если гость отменяет заказ, диалог не запрашивает номер комнаты, а сразу же завершается.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В класс **HotelDialog** добавьте структуру данных, которая будет использоваться для отслеживания заказа гостя. Так как она будет сохраняться в состоянии диалога, этому классу нужен конструктор по умолчанию для сериализации.

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice>
{
    public OrderCart() : base() { }

    public OrderCart(OrderCart other) : base(other) { }
}
```

Добавьте в конструктор каскадный диалог. Затем определите каскадные действия. Здесь мы разместили их во вложенном классе.

В конструкторе **HotelDialogs**.

```csharp
// Define the steps for and add the order-dinner dialog.
WaterfallStep[] orderDinnerDialogSteps = new WaterfallStep[]
{
    OrderDinnerSteps.StartFoodSelectionAsync,
    OrderDinnerSteps.GetRoomNumberAsync,
    OrderDinnerSteps.ProcessOrderAsync,
};

Add(new WaterfallDialog(Dialogs.OrderDinner, orderDinnerDialogSteps));
```

В классе **HotelDialogs** добавьте определения шагов.

* На первом шаге мы отправляем приветственное сообщение и запускаем диалог для выбора блюд.
* На втором шаге мы проверяем, возвращает ли диалог для выбора блюд корзину заказов.
  * Если возвращает, мы запрашиваем у гостя номер комнаты и сохраняем данные корзины.
  * Если не возвращает, следует считать, что гость отменил заказ, и вызвать метод завершения диалога _end dialog_.
* На последнем шаге мы записываем номер комнаты гостя и отправляем ему сообщение с подтверждением, чтобы завершить диалог. На этом этапе бот отправляет заказ в службу на обработку.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-dinner dialog.
/// </summary>
private static class OrderDinnerSteps
{
    public static async Task<DialogTurnResult> StartFoodSelectionAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Welcome to our Dinner order service.",
            cancellationToken: cancellationToken);

        // Start the food selection dialog.
        return await stepContext.BeginDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
    }

    public static async Task<DialogTurnResult> GetRoomNumberAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        if (stepContext.Result != null && stepContext.Result is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            stepContext.Values[Outputs.OrderCart] = cart;
            return await stepContext.PromptAsync(
                Inputs.Number,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your room number?"),
                    RetryPrompt = MessageFactory.Text("Please enter your room number."),
                },
                cancellationToken);
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }

    public static async Task<DialogTurnResult> ProcessOrderAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get and save the guest's answer.
        var roomNumber = (int)stepContext.Result;
        stepContext.Values[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await stepContext.Context.SendActivityAsync(
            $"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.",
            cancellationToken: cancellationToken);
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте в конструктор бота каскадный диалог `orderDinner`.

```javascript
// Order dinner:
// Help user order dinner from a menu
this.dialogSet.add(new WaterfallDialog('orderDinner', [
    async function (step) {
        await step.context.sendActivity("Welcome to our dinner order service.");

        return await step.beginDialog('orderPrompt', step.values.orderCart = {
            orders: [],
            total: 0
        }); // Prompt for orders
    },
    async function (step) {
        if (step.result == "Cancel") {
            return await step.endDialog();
        } else {
            return await step.prompt('numberPrompt', "What is your room number?");
        }
    },
    async function (step) {
        await step.context.sendActivity(`Thank you. Your order will be delivered to room ${step.result} within 45 minutes.`);
        return await step.endDialog();
    }
]));
```

---

### <a name="create-the-order-prompt-dialog"></a>Создание диалога с запросом на оформление заказа

В диалоге выбора блюд гостю предоставляется список вариантов, содержащий варианты блюд, которые гости могут заказать на ужин, и два запроса для обработки заказа. Этот цикл диалога выполнятся до тех пор, пока гость не выберет обработку заказа или не отменит его.

* Когда гость выбирает блюдо на ужин, оно добавляется в _корзину_.
* Если гость выбирает обработку заказа, мы сначала проверяем, не пуста ли корзина.
  * Если она пуста, отправляется сообщение об ошибке и цикл повторяется.
  * В противном случае этот диалог завершается, возвращая сведения о покупках в родительском диалоге.
* Если гость решил отменить заказ, диалог завершается, не возвращая сведения о покупках.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Добавьте в конструктор каскадный диалог. Затем определите каскадные действия. Здесь мы разместили их во вложенном классе.

В конструкторе **HotelDialogs**.

```csharp
// Define the steps for and add the order-prompt dialog.
WaterfallStep[] orderPromptDialogSteps = new WaterfallStep[]
{
    OrderPromptSteps.PromptForItemAsync,
    OrderPromptSteps.ProcessInputAsync,
};

Add(new WaterfallDialog(Dialogs.OrderPrompt, orderPromptDialogSteps));
```

* На первом шаге мы инициализируем состояние диалога. Если входные аргументы для диалога содержат сведения о корзине, мы сохраняем их в состоянии о диалоге. В противном случае мы создаем и добавляем пустую корзину. Затем создаем запрос для гостя на выбор блюд из меню ужина.
* На следующем шаге изучаем вариант, выбранный гостем.
  * Если это запрос на обработку заказа, проверяем содержит ли корзина какие-либо элементы.
    * Если да, диалог завершается, возвращая содержимое корзины.
    * В противном случае гостю отправляется сообщения об ошибке и диалог начинается сначала.
  * Если это запрос на отмену заказа, диалог завершается, возвращая пустой словарь.
  * Если это выбранное на ужин блюдо, оно добавляется в корзину, отправляется сообщение о состоянии и запускается диалог, передавая текущее состояние заказа.

В классе **HotelDialogs** добавьте определения шагов.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the order-prompt dialog.
/// </summary>
private static class OrderPromptSteps
{
    public static async Task<DialogTurnResult> PromptForItemAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // First time through, options will be null.
        var cart = (stepContext.Options is OrderCart oldCart && oldCart != null)
            ? new OrderCart(oldCart) : new OrderCart();

        stepContext.Values[Outputs.OrderCart] = cart;
        stepContext.Values[Outputs.OrderTotal] = cart.Sum(item => item.Price);

        return await stepContext.PromptAsync(
            Inputs.Choice,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("What would you like?"),
                RetryPrompt = Lists.MenuReprompt,
                Choices = Lists.MenuChoices,
            },
            cancellationToken);
    }

    public static async Task<DialogTurnResult> ProcessInputAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        // Get the guest's choice.
        var choice = (FoundChoice)stepContext.Result;
        var menuOption = Lists.MenuOptions[choice.Index];

        // Get the current order from dialog state.
        var cart = (OrderCart)stepContext.Values[Outputs.OrderCart];

        if (menuOption.Name is MenuChoice.Process)
        {
            if (cart.Count > 0)
            {
                // If there are any items in the order, then exit this dialog,
                // and return the list of selected food items.
                return await stepContext.EndDialogAsync(cart, cancellationToken);
            }
            else
            {
                // Otherwise, send an error message and restart from
                // the beginning of this dialog.
                await stepContext.Context.SendActivityAsync(
                    "Your cart is empty. Please add at least one item to the cart.",
                    cancellationToken: cancellationToken);
                return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, null, cancellationToken);
            }
        }
        else if (menuOption.Name is MenuChoice.Cancel)
        {
            await stepContext.Context.SendActivityAsync(
                "Your order has been cancelled.",
                cancellationToken: cancellationToken);

            // Exit this dialog, returning null.
            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
        else
        {
            // Add the selected food item to the order and update the order total.
            cart.Add(menuOption);
            var total = (double)stepContext.Values[Outputs.OrderTotal] + menuOption.Price;
            stepContext.Values[Outputs.OrderTotal] = total;

            await stepContext.Context.SendActivityAsync(
                $"Added {menuOption.Name} (${menuOption.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.",
                cancellationToken: cancellationToken);

            // Present the order options again, passing in the current order state.
            return await stepContext.ReplaceDialogAsync(Dialogs.OrderPrompt, cart);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте в конструктор бота каскадный диалог `orderPrompt`.

```javascript
// Helper dialog to repeatedly prompt user for orders
this.dialogSet.add(new WaterfallDialog('orderPrompt', [
    async function (step) {
        // Define a new cart of one does not exists
        step.values.orderCart = step.options;

        return await step.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function (step) {
        const choice = step.result;
        if (choice.value.match(/process order/ig)) {
            if (step.values.orderCart.orders.length > 0) {
                // Process the order
                await step.context.sendActivity("Processing your order.");
                // ...
                step.values.orderCart = undefined; // Reset cart
                return await step.endDialog();
            } else {
                await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        } else if (choice.value.match(/cancel/ig)) {
            await step.context.sendActivity("Your order has been canceled.");
            return await step.endDialog(choice.value);
        } else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if (!item) {
                await step.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            } else {
                // Add the item to cart
                step.values.orderCart.orders.push(item);
                step.values.orderCart.total += item.Price;

                await step.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${step.values.orderCart.total}`);

                // Ask again
                return await step.replaceDialog('orderPrompt', step.values.orderCart);
            }
        }
    }
]));
```

В приведенном выше примере кода основной диалог `orderDinner` использует вспомогательный диалог с именем `orderPrompt` для обработки выбираемых пользователем вариантов. Диалог `orderPrompt` отображает меню с вариантами блюд, предлагает пользователю выбрать элемент, добавляет его в корзину и предлагает выбрать элемент еще раз в рамках цикла. Это позволяет пользователю добавить несколько элементов в заказ. Цикл в диалоге продолжает работать, пока пользователь не выберет `Process order` или `Cancel`. В этот момент выполнение возвращается в родительский диалог (в диалог `orderDinner`). Диалог `orderDinner` выполняет последние необходимые действия в случае, если пользователь хочет обработать заказ; в противном случае он завершается и возвращает выполнение в родительский диалог (например: `mainMenu`). Диалог `mainMenu`, в свою очередь, выполняет последнее действие, которое представляет собой повторное отображение элементов главного меню.

---

### <a name="create-the-reserve-table-dialog"></a>Создание диалога для резервирования столика

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

Для простоты в этом примере мы покажем только минимальную реализацию диалога `reserveTable`.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Добавьте в конструктор каскадный диалог. Затем определите каскадные действия. Здесь мы разместили их во вложенном классе.

В конструкторе **HotelDialogs**.

```csharp
// Define the steps for and add the reserve-table dialog.
WaterfallStep[] reserveTableDialogSteps = new WaterfallStep[]
{
    ReserveTableSteps.StubAsync,
};

Add(new WaterfallDialog(Dialogs.ReserveTable, reserveTableDialogSteps));
```

В классе **HotelDialogs** добавьте определения шагов.

```csharp
/// <summary>
/// Contains the waterfall dialog steps for the reserve-table dialog.
/// </summary>
private static class ReserveTableSteps
{
    public static async Task<DialogTurnResult> StubAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken)
    {
        await stepContext.Context.SendActivityAsync(
            "Your table has been reserved.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте в конструктор бота макет каскадного диалога `reserveTable`.

```javascript
// Reserve a table:
// Help the user to reserve a table
this.dialogSet.add(new WaterfallDialog('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(step){
        await step.context.sendActivity("Your table has been reserved");
        await step.endDialog();
    }
]));
```

---

### <a name="update-the-bot-code-to-call-the-dialogs"></a>Обновление кода бота для вызова диалогов

Обновите код обработчика шагов в боте, добавив вызов диалога.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Переименуйте файл **EchoBotAccessors.cs** в **BotAccessors.cs** и смените имя класса с `EchoBotAccessors` на `BotAccessors`. Обновите инструкции using и определение класса, чтобы предоставить метод доступа к свойству состояния, который нужен нам для работы этого бота.

```csharp
using System;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
```

```csharp
/// <summary>
/// This class is created as a Singleton and passed into the IBot-derived constructor.
///  - See <see cref="EchoWithCounterBot"/> constructor for how that is injected.
///  - See the Startup.cs file for more details on creating the Singleton that gets
///    injected into the constructor.
/// </summary>
public class BotAccessors
{
    /// <summary>
    /// Initializes a new instance of the <see cref="BotAccessors"/> class.
    /// Contains the <see cref="ConversationState"/> and associated <see cref="IStatePropertyAccessor{T}"/>.
    /// </summary>
    /// <param name="conversationState">The state object that stores the counter.</param>
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    /// <summary>
    /// Gets the <see cref="IStatePropertyAccessor{T}"/> name used for the <see cref="DialogState"/> accessor.
    /// </summary>
    /// <remarks>Accessors require a unique name.</remarks>
    /// <value>The accessor name for the dialog state accessor.</value>
    public static string DialogStateAccessorName { get; } = $"{nameof(BotAccessors)}.DialogState";

    /// <summary>
    /// Gets or sets the DialogState property accessor.
    /// </summary>
    /// <value>
    /// The DialogState property accessor.
    /// </value>
    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    /// <summary>
    /// Gets the <see cref="ConversationState"/> object for the conversation.
    /// </summary>
    /// <value>The <see cref="ConversationState"/> object.</value>
    public ConversationState ConversationState { get; }
}
```

Обновите файл **Startup.cs**, настроив в нем singleton `BotAccessors`.

1. Обновите инструкции using.

    ```csharp
    using System;
    using System.Linq;
    using Microsoft.AspNetCore.Builder;
    using Microsoft.AspNetCore.Hosting;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Integration;
    using Microsoft.Bot.Builder.Integration.AspNet.Core;
    using Microsoft.Bot.Configuration;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Extensions.Configuration;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using Microsoft.Extensions.Options;
    ```

1. Обновите ту часть метода `ConfigureServices`, которая регистрирует методы доступа к свойству состояния бота.

    ```csharp
    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorName),
        };

        return accessors;
    });
    ```

Переименуйте файл EchoWithCounterBot.cs в HotelBot.cs и переименуйте класс EchoWithCounterBot в HotelBot.

1. Обновите код инициализации бота.

    ```csharp
    private readonly BotAccessors _accessors;
    private readonly HotelDialogs _dialogs;
    private readonly ILogger _logger;

    /// <summary>
    /// Initializes a new instance of the <see cref="HotelBot"/> class.
    /// </summary>
    /// <param name="accessors">A class containing <see cref="IStatePropertyAccessor{T}"/> used to manage state.</param>
    /// <param name="loggerFactory">A <see cref="ILoggerFactory"/> that is hooked to the Azure App Service provider.</param>
    public HotelBot(BotAccessors accessors, ILoggerFactory loggerFactory)
    {
        _logger = loggerFactory.CreateLogger<HotelBot>();
        _logger.LogTrace("EchoBot turn start.");
        _accessors = accessors;
        _dialogs = new HotelDialogs(_accessors.DialogStateAccessor);
    }
    ```

1. Обновите обработчик шагов в боте, чтобы он запускал этот диалог.

    ```csharp
    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        var dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            await dc.ContinueDialogAsync(cancellationToken);
            if (!turnContext.Responded)
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }
        else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate)
        {
            var activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                await dc.BeginDialogAsync(HotelDialogs.MainMenu, null, cancellationToken);
            }
        }

        await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    let dc = await this.dialogSet.createContext(turnContext);

    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {

        await dc.continueDialog();

        if (!turnContext.responded) {
            await dc.beginDialog('mainMenu');
        }
    } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
        // Do we have any new members added to the conversation?
        if (turnContext.activity.membersAdded.length !== 0) {
            // Iterate over all new members added to the conversation
            for (var idx in turnContext.activity.membersAdded) {
                // Greet anyone that was not the target (recipient) of this message.
                // Since the bot is the recipient for events from the channel,
                // context.activity.membersAdded === context.activity.recipient.Id indicates the
                // bot was added to the conversation, and the opposite indicates this is a user.
                if (turnContext.activity.membersAdded[idx].id !== turnContext.activity.recipient.id) {
                    // Start the dialog.
                    await dc.beginDialog('mainMenu');
                }
            }
        }
    }

    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
```

---

## <a name="next-steps"></a>Дополнительная информация

Вы можете расширить этот бот, добавив в меню другие варианты, например "Дополнительные сведения" или "Справка". Дополнительные сведения о реализации таких прерываний см. в статье [Обработка прерываний со стороны пользователя](bot-builder-howto-handle-user-interrupt.md).

Теперь, когда вы знаете, как использовать диалоги, запросы и каскады для управления сложным процессом общения, рассмотрим, как разбить наши диалоги (`orderDinner` и`reserveTable`) на отдельные объекты вместо того, чтобы собирать их вместе в одном большом наборе диалогов.

> [!div class="nextstepaction"]
> [Создание модульной логики бота с помощью составных элементов управления](bot-builder-compositcontrol.md)

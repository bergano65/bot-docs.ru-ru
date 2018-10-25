---
title: Обработка прерываний со стороны пользователя | Документация Майкрософт
description: Узнайте, как обрабатывать прерывания со стороны пользователя и прямой поток беседы.
keywords: interrupt, interruptions, switching topic, break
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/20/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 15c3ed60dacf1751e82c2bb0804cdd938f286f40
ms.sourcegitcommit: 6c2426c43cd2212bdea1ecbbf8ed245145b3c30d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/08/2018
ms.locfileid: "48852329"
---
# <a name="handle-user-interruptions"></a>Обработка прерываний со стороны пользователя

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Обработка прерываний является важным аспектом для создания надежного бота.

При этом вы можете подумать, что ваши пользователи будут следить за потоком беседы шаг за шагом, но скорее всего они изменят свое мнение или вместо того, чтобы отвечать, зададут вопрос в середине процесса. В таких ситуациях как вы хотите, чтобы бот обрабатывал входные данные пользователя? На что должно быть похоже взаимодействие с пользователем? Как поддерживать пользовательские данные о состоянии? Обработка прерываний позволяет подготовить бота к таким ситуациям.

Нет единственного правильного ответа на эти вопросы, так как каждая ситуация является уникальной для сценария, который обрабатывает ваш бот. В этом разделе рассматривается несколько распространенных способов обработки прерываний со стороны пользователя и предлагаются некоторые способы их реализации в боте.

## <a name="handle-expected-interruptions"></a>Обработка ожидаемых прерываний

Поток процедурной беседы имеет базовый набор шагов, которым должен следовать пользователь, и любые его действия, которые отличаются от этих шагов, являются потенциальными прерываниями. В обычном потоке существуют ожидаемые прерывания.

**Резервирование столов.** Основные шаги бота, который резервирует стол, могут заключаться в том, чтобы попросить пользователя задать дату и время, количество людей и имя для резервирования. Некоторые ожидаемые прерывания в этом процессе, которые можно спрогнозировать:

* `cancel`. Выход из процесса.
* `help`. Предоставление дополнительных сведений об этом процессе.
* `more info`. Предоставление указаний и предложений или альтернативных способов резервирования столов (например, адрес электронной почты или номер телефона для связи).
* `show list of available tables`. Отображение списка доступных столов на дату и время, которое нужно пользователю (если возможно).

**Заказ ужина.** В боте заказа ужина основные действия могут включать предоставление меню и добавление пользователем блюд в "корзину". Некоторые ожидаемые прерывания в этом процессе, которые можно спрогнозировать:

* `cancel`. Выход из процесса заказа.
* `more info`. Предоставление диетических сведений о каждом пункте меню.
* `help`. Предоставление справки по использованию системы.
* `process order`. Обработка заказа.

Можно предоставить их пользователю как список **предлагаемых действий** или как указание, чтобы он, по крайней мере, знал, какие команды нужно отправить боту, чтобы он понял.

Например, в потоке заказа ужина ожидаемые прерывания можно указать вместе с элементами меню. В этом случае элементы меню отправляются как массив `choices`.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

Мы определим набор диалогов как подкласс **DialogSet**.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Choices;

public class OrderDinnerDialogs : DialogSet
{
    public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
        : base(dialogStateAccessor)
    {
    }
}
```

Мы определим несколько внутренних классов для описания меню.

```cs
/// <summary>
/// Contains information about an item on the menu.
/// </summary>
public class DinnerItem
{
    public string Description { get; set; }

    public double Price { get; set; }
}

/// <summary>
/// Describes the dinner menu, including the items on the menu and options for
/// interrupting the ordering process.
/// </summary>
public class DinnerMenu
{
    /// <summary>Gets the items on the menu.</summary>
    public static Dictionary<string, DinnerItem> MenuItems { get; } = new Dictionary<string, DinnerItem>
    {
        ["Potato salad"] = new DinnerItem { Description = "Potato Salad", Price = 5.99 },
        ["Tuna sandwich"] = new DinnerItem { Description = "Tuna Sandwich", Price = 6.89 },
        ["Clam chowder"] = new DinnerItem { Description = "Clam Chowder", Price = 4.50 },
    };

    /// <summary>Gets all the "interruptions" the bot knows how to process.</summary>
    public static List<string> Interrupts { get; } = new List<string>
    {
        "More info", "Process order", "Help", "Cancel",
    };

    /// <summary>Gets all of the valid inputs a user can make.</summary>
    public static List<string> Choices { get; }
        = MenuItems.Select(c => c.Key).Concat(Interrupts).ToList();
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Мы начнем с базового шаблона EchoBot. Инструкции см. в [кратком руководстве по JavaScript](~/javascript/bot-builder-javascript-quickstart.md).

Библиотеку `botbuilder-dialogs` можно скачать из NPM. Чтобы установить библиотеку `botbuilder-dialogs`, выполните следующую команду npm:

```cmd
npm install --save botbuilder-dialogs
```

В файле **bot.js** добавьте ссылки на классы и определите идентификаторы, которые мы будем использовать.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, ChoicePrompt, WaterfallDialog, DialogTurnStatus } = require('botbuilder-dialogs');

// Name for the dialog state property accessor.
const DIALOG_STATE_PROPERTY = 'dialogStateProperty';

// Name of the order-prompt dialog.
const ORDER_PROMPT = 'orderingDialog';

// Name for the choice prompt for use in the dialog.
const CHOICE_PROMPT = 'choicePrompt';
```

Определите варианты выбора, которые должны отображаться в диалоге заказа.

```javascript
// The options on the dinner menu, including commands for the bot.
const dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50",
        "Process order", "Cancel", "More info", "Help"],
    "Potato Salad - $5.99": {
        description: "Potato Salad",
        price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        description: "Tuna Sandwich",
        price: 6.89
    },
    "Clam Chowder - $4.50": {
        description: "Clam Chowder",
        price: 4.50
    }
}
```

---

В логике упорядочивания их можно проверить с помощью строки сопоставления или регулярных выражений.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

Сначала нам нужно определить вспомогательное приложение для отслеживания заказов.

```cs
/// <summary>Helper class for storing the order.</summary>
public class Order
{
    public double Total { get; set; } = 0.0;

    public List<DinnerItem> Items { get; set; } = new List<DinnerItem>();

    public bool ReadyToProcess { get; set; } = false;

    public bool OrderProcessed { get; set; } = false;
}
```

Добавьте несколько констант для отслеживания нужных идентификаторов.

```csharp
/// <summary>The ID of the top-level dialog.</summary>
public const string MainDialogId = "mainMenu";

/// <summary>The ID of the choice prompt.</summary>
public const string ChoicePromptId = "choicePrompt";

/// <summary>The ID of the order card value, tracked inside the dialog.</summary>
public const string OrderCartId = "orderCart";
```

Добавьте в конструктор строку запроса и каскадный диалог. Также мы определим методы, которые реализуют каскадные действия.

```cs
public OrderDinnerDialogs(IStatePropertyAccessor<DialogState> dialogStateAccessor)
    : base(dialogStateAccessor)
{
    // Add a choice prompt for the dialog.
    Add(new ChoicePrompt(ChoicePromptId));

    // Define and add the main waterfall dialog.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptUserAsync,
        ProcessInputAsync,
    };

    Add(new WaterfallDialog(MainDialogId, steps));
}

/// <summary>
/// Defines the first step of the main dialog, which is to ask for input from the user.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> PromptUserAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Initialize order, continuing any order that was passed in.
    Order order = (stepContext.Options is Order oldCart && oldCart != null)
        ? new Order
        {
            Items = new List<DinnerItem>(oldCart.Items),
            Total = oldCart.Total,
            ReadyToProcess = oldCart.ReadyToProcess,
            OrderProcessed = oldCart.OrderProcessed,
        }
        : new Order();

    // Set the order cart in dialog state.
    stepContext.Values[OrderCartId] = order;

    // Prompt the user.
    return await stepContext.PromptAsync(
        "choicePrompt",
        new PromptOptions
        {
            Prompt = MessageFactory.Text("What would you like for dinner?"),
            RetryPrompt = MessageFactory.Text(
                "I'm sorry, I didn't understand that. What would you like for dinner?"),
            Choices = ChoiceFactory.ToChoices(DinnerMenu.Choices),
        },
        cancellationToken);
}

/// <summary>
/// Defines the second step of the main dialog, which is to process the user's input, and
/// repeat or exit as appropriate.
/// </summary>
/// <param name="stepContext">The current waterfall step context.</param>
/// <param name="cancellationToken">The cancellation token.</param>
/// <returns>The task to perform.</returns>
private async Task<DialogTurnResult> ProcessInputAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the order cart from dialog state.
    Order order = stepContext.Values[OrderCartId] as Order;

    // Get the user's choice from the previous prompt.
    string response = (stepContext.Result as FoundChoice).Value;

    if (response.Equals("process order", StringComparison.InvariantCultureIgnoreCase))
    {
        order.ReadyToProcess = true;

        await stepContext.Context.SendActivityAsync(
            "Your order is on it's way!",
            cancellationToken: cancellationToken);

        // In production, you may want to store something more helpful.
        // "Process" the order and exit.
        order.OrderProcessed = true;
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
    {
        // Cancel the order.
        await stepContext.Context.SendActivityAsync(
            "Your order has been canceled",
            cancellationToken: cancellationToken);

        // Exit without processing the order.
        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    else if (response.Equals("more info", StringComparison.InvariantCultureIgnoreCase))
    {
        // Send more information about the options.
        string message = "More info: <br/>" +
            "Potato Salad: contains 330 calories per serving. Cost: 5.99 <br/>"
            + "Tuna Sandwich: contains 700 calories per serving. Cost: 6.89 <br/>"
            + "Clam Chowder: contains 650 calories per serving. Cost: 4.50";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else if (response.Equals("help", StringComparison.InvariantCultureIgnoreCase))
    {
        // Provide help information.
        string message = "To make an order, add as many items to your cart as " +
            "you like. Choose the `Process order` to check out. " +
            "Choose `Cancel` to cancel your order and exit.";
        await stepContext.Context.SendActivityAsync(
            message,
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }

    // We've checked for expected interruptions. Check for a valid item choice.
    if (!DinnerMenu.MenuItems.ContainsKey(response))
    {
        await stepContext.Context.SendActivityAsync("Sorry, that is not a valid item. " +
            "Please pick one from the menu.");

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
    else
    {
        // Add the item to cart.
        DinnerItem item = DinnerMenu.MenuItems[response];
        order.Items.Add(item);
        order.Total += item.Price;

        // Acknowledge the input.
        await stepContext.Context.SendActivityAsync(
            $"Added `{response}` to your order; your total is ${order.Total:0.00}.",
            cancellationToken: cancellationToken);

        // Continue the ordering process, passing in the current order cart.
        return await stepContext.ReplaceDialogAsync(MainDialogId, order, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Определите диалоговое окно в конструкторе ботов. Обратите внимание, что код _сначала_ проверяет прерывания и обрабатывает их, и лишь затем переходит к следующим логическим шагам.

```javascript
constructor(conversationState) {
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_PROPERTY);
    this.conversationState = conversationState;

    this.dialogs = new DialogSet(this.dialogStateAccessor)
        .add(new ChoicePrompt(CHOICE_PROMPT))
        .add(new WaterfallDialog(ORDER_PROMPT, [
            async (step) => {
                if (step.options && step.options.orders) {
                    // If an order cart was passed in, continue to use it.
                    step.values.orderCart = step.options;
                } else {
                    // Otherwise, start a new cart.
                    step.values.orderCart = {
                        orders: [],
                        total: 0
                    };
                }
                return await step.prompt(CHOICE_PROMPT, "What would you like?", dinnerMenu.choices);
            },
            async (step) => {
                const choice = step.result;
                if (choice.value.match(/process order/ig)) {
                    if (step.values.orderCart.orders.length > 0) {
                        // If the cart is not empty, process the order by returning the order to the parent context.
                        await step.context.sendActivity("Your order has been processed.");
                        return await step.endDialog(step.values.orderCart);
                    } else {
                        // Otherwise, prompt again.
                        await step.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                        return await step.replaceDialog(ORDER_PROMPT);
                    }
                } else if (choice.value.match(/cancel/ig)) {
                    // Cancel the order, and return "cancel" to the parent context.
                    await step.context.sendActivity("Your order has been canceled.");
                    return await step.endDialog("cancelled");
                } else if (choice.value.match(/more info/ig)) {
                    // Provide more information, and then continue the ordering process.
                    var msg = "More info: <br/>Potato Salad: contains 330 calories per serving. <br/>"
                        + "Tuna Sandwich: contains 700 calories per serving. <br/>"
                        + "Clam Chowder: contains 650 calories per serving."
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else if (choice.value.match(/help/ig)) {
                    // Provide help information, and then continue the ordering process.
                    var msg = `Help: <br/>`
                        + `To make an order, add as many items to your cart as you like then choose `
                        + `the "Process order" option to check out.`
                    await step.context.sendActivity(msg);
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                } else {
                    // The user has chosen a food item from the menu. Add the item to cart.
                    var item = dinnerMenu[choice.value];
                    step.values.orderCart.orders.push(item.description);
                    step.values.orderCart.total += item.price;

                    await step.context.sendActivity(`Added ${item.description} to your cart. <br/>`
                        + `Current total: $${step.values.orderCart.total}`);

                    // Continue the ordering process.
                    return await step.replaceDialog(ORDER_PROMPT, step.values.orderCart);
                }
            }
        ]));
}
```

Обновите обработчик шагов для вызова диалога и отображения результатов из процесса оформления заказа.

```javascript
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        let dc = await this.dialogs.createContext(turnContext);
        let dialogTurnResult = await dc.continueDialog();
        if (dialogTurnResult.status === DialogTurnStatus.complete) {
            // The dialog completed this turn.
            const result = dialogTurnResult.result;
            if (!result || result === "cancelled") {
                await turnContext.sendActivity('You cancelled your order.');
            } else {
                await turnContext.sendActivity(`Your order came to $${result.total}`);
            }
        } else if (!turnContext.responded) {
            // No dialog was active.
            await turnContext.sendActivity("Let's order dinner...");
            await dc.cancelAllDialogs();
            await dc.beginDialog(ORDER_PROMPT);
        } else {
            // The dialog is active.
        }
    } else {
        await turnContext.sendActivity(`[${turnContext.activity.type} event detected]`);
    }
    // Save state changes
    await this.conversationState.saveChanges(turnContext);
}
```

---

## <a name="handle-unexpected-interruptions"></a>Обработка непредвиденных прерываний

Существуют прерывания в работе, которые находятся вне области функций бота.
Хотя невозможно предвидеть все прерывания, существуют шаблоны, на использование которых можно запрограммировать бот.

### <a name="switching-topic-of-conversations"></a>Переключение темы диалогов

Что делать, если пользователь находится в середине одного диалога и хочет перейти к другому? Например, бот может зарезервировать столик и заказать ужин.
Находясь в потоке _резервирования столов_, пользователь не отвечает на вопрос "Сколько пользователей будет на вечеринке?", а отправляет сообщение "Заказать ужин". В этом случае пользователь изменил свое мнение и хочет открыть диалог о заказе ужина. Как обрабатывать это прерывание?

Вы можете переключить темы на поток заказа ужина или сделать его прикрепленной проблемой, сообщая пользователю, что ожидается ввод количества людей и переспросить его. Если вы разрешите переключать темы, необходимо решить, будет ли сохраняться прогресс, чтобы пользователь мог начать с того места, где он остановился, или все сведения будут удалены и в следующий раз, когда ему нужно будет заказать столик, придется начать процесс сначала. Дополнительные сведения об управлении пользовательскими данными см. в статье [Save state using conversation and user properties](bot-builder-howto-v4-state.md) (Сохранение состояния с помощью свойств общения и пользователя).

### <a name="apply-artificial-intelligence"></a>Применение искусственного интеллекта

Для прерываний, которые находятся вне области функций, можно попытаться угадать намерения пользователя. Вы можете это сделать с помощью служб искусственного интеллекта, например QnAMaker, LUIS или пользовательской логики, а затем добавить предположения бота о желаниях пользователя.

Например, в середине потока резервирования стола пользователь говорит: "Я хочу заказать гамбургер". В этом случае бот не знает, как обработать это намерение из этой последовательности диалога. Так как текущий поток не имеет ничего общего с заказом, а другая команда беседы бота — "заказать ужин", бот не знает, что делать с этими входными данными. Например, если применить LUIS, вы сможете обучить модель на распознавание того, что пользователь хочет заказать еду (например, LUIS может возвращать намерение orderFood). Таким образом бот может отвечать: "Похоже, что вы хотите заказать еду. Вы хотите переключиться на процесс заказа ужина?" Дополнительные сведения об обучении LUIS и обнаружении намерений пользователя см. в статье [Using LUIS for Language Understanding](bot-builder-howto-v4-luis.md) (Использование LUIS для распознавания речи).

### <a name="default-response"></a>Ответ по умолчанию

Если все остальное не удастся, следует отправить ответ по умолчанию, чтобы пользователь не оставался в недоумении от молчания бота. Ответ по умолчанию должен сообщать, какие команды понимает бот, чтобы пользователь мог вернуться в нужное русло.

Вы можете проверить контекст флага **ответа** в конце логики бота, чтобы узнать, отправил ли бот что-либо пользователю в ответ. Если бот обрабатывает входные данные пользователя, но не отвечает, скорее всего, бот не знает, что делать со входными данными. В этом случае можно перехватить и отправить пользователю сообщение по умолчанию.

Если ваш бот по умолчанию предлагает пользователю диалог `mainMenu`. Именно вы решаете, какие возможности получит пользователь от вашего бота в такой ситуации.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check whether we replied. If not then clear the dialog stack and present the main menu.
if (!turnContext.Responded)
{
    await dc.CancelAllDialogsAsync(cancellationToken);
    await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.cancelAllDialogs();
    await step.beginDialog('mainMenu');
}
```

---

## <a name="handling-global-interruptions"></a>Обработка глобальных прерываний

В приведенном выше примере мы обрабатываем прерывания, которые могут возникнуть на определенном шаге определенного диалога. Как же нам поступать с глобальными прерываниями, то есть такими, которые могут произойти в любое время?

Логику обработки таких прерываний можно вынести в основной обработчик бота. Это функция, которая обрабатывает входящий `turnContext` и решает, что с ним делать.

В следующем примере _первое действие_ бота — проверка входящего текстового сообщения и поиск указаний пользователя на то, что он хочет получить помощь или отменить заказ — это два самых распространенных прерывания в работе ботов. После завершения этой проверки бот вызывает `dc.continueDialog()` для обработки всех незавершенных активных диалогов.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
// Check for top-level interruptions.
string utterance = turnContext.Activity.Text.Trim().ToLowerInvariant();

if (utterance == "help")
{
    // Start a general help dialog. Dialogs already on the stack remain and will continue
    // normally if the help dialog exits normally.
    await dc.BeginDialogAsync(OrderDinnerDialogs.HelpDialogId, null, cancellationToken);
}
else if (utterance == "cancel")
{
    // Cancel any dialog on the stack.
    await turnContext.SendActivityAsync("Canceled.", cancellationToken: cancellationToken);
    await dc.CancelAllDialogsAsync(cancellationToken);
}
else
{
    await dc.ContinueDialogAsync(cancellationToken);

    // Check whether we replied. If not then clear the dialog stack and present the main menu.
    if (!turnContext.Responded)
    {
        await dc.CancelAllDialogsAsync(cancellationToken);
        await dc.BeginDialogAsync(OrderDinnerDialogs.MainDialogId, null, cancellationToken);
    }
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Мы можем обрабатывать прерывания перед отправкой ответа в диалог с пользователем.

В этом фрагменте предполагается, что в нашем наборе диалогов есть `helpDialog` и `mainMenu`.

```javascript
const utterance = (context.activity.text || '').trim();

// Let's look for some interruptions first!
if (utterance.match(/help/ig)) {
    // Launch a new help dialog if the user asked for help
    await dc.beginDialog('helpDialog');
} else if (utterance.match(/cancel/ig)) {
    // Cancel any active dialogs if the user says cancel
    await dc.context.sendActivity('Canceled.');
    await dc.cancelAllDialogs();
}

// If the bot hasn't yet responded...
if (!context.responded) {
    // Continue any active dialog, which might send a response...
    await dc.continueDialog();

    // Finally, if the bot still hasn't sent a response, send instructions.
    if (!context.responded) {
        await dc.cancelAllDialogs();
        await context.sendActivity("Starting the main menu...");
        await dc.beginDialog('mainMenu');
    }
}
```

---

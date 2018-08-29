---
title: Управление сложным процессом общения с помощью диалогов | Документация Майкрософт
description: Сведения о том, как управлять сложным процессом общения с помощью диалогов в пакете SDK Bot Builder для Node.js.
keywords: complex conversation flow, repeat, loop, menu, dialogs, prompts, waterfalls, dialog set
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 304de6783a268140bf74f95d96cd9c24e12c4c05
ms.sourcegitcommit: 1abc32353c20acd103e0383121db21b705e5eec3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/21/2018
ms.locfileid: "40236368"
---
# <a name="manage-complex-conversation-flows-with-dialogs"></a>Управление сложными процессами общения с помощью диалогов

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

С помощью библиотеки диалогов вы можете управлять простыми и сложными процессами общения. В рамках [простого процесса общения](bot-builder-dialog-manage-conversation-flow.md) пользователь выполняет все действия в *каскадной последовательности*, после чего обмен сообщениями прекращается. В этой статье мы будем использовать диалоги для управления более сложными диалогами, части которого могут ответвляться и образовывать цикл. Для этого мы будем использовать метод _replace_ контекста диалога, а также передавать аргументы между разными частями диалога.

<!-- TODO: We need a dialogs conceptual topic to link to, so we can reference that here, in place of describing what they are and what their features are in a how-to topic. -->

Чтобы обеспечить больше возможностей управления *стеком диалогов*, в библиотеке **диалогов** предоставляется метод _replace_. Этот метод извлекает текущий диалог из стека, отправить новый диалог на вершину стека и начать новый диалог. Это позволяет усложнить процесс общения. Эти методы можно использовать для создания диалогов произвольной сложности. Если же сложность вашего диалога увеличится до такой степени, что им станет сложно управлять, можно создать собственную последовательность логики управления, чтобы отслеживать состояние диалога с пользователем.

<!-- TODO: This is probably a good place to add a link to the modular/composite dialogs topic. -->

В рамках этой статьи мы создадим диалоги для бота гостиницы, с помощью которого гость сможет зарезервировать столик или заказать еду себе в номер. Эти два варианта предоставляются гостю в диалоге верхнего уровня. Если гость хочет зарезервировать столик, в диалоге верхнего уровня начинается соответствующий диалог. Если же гость хочет заказать ужин, в диалоге верхнего уровня начинается диалог для заказа ужина. В рамках этого диалога сначала пользователю предлагается меню для выбора блюд, а затем запрашивается номер комнаты. Выбор блюд также является диалогом, в котором гость может выбрать несколько блюд, прежде чем заказ будет обработан.

На схеме ниже показаны диалоги, которые мы создадим в рамках этой статьи, и связь между ними.

![Схема с диалогами, используемыми в рамках этой статьи](~/media/complex-conversation-flows.png)

## <a name="how-to-branch"></a>Создание ветвей

В контексте диалога содержится _стек диалогов_, и для каждого диалога из стека отслеживается, какой шаг будет следующим. Чтобы поместить диалог на вершину стека, используется метод _begin_. А для извлечения диалога с вершины стека используется метод _end_.

В рамках одного диалога может начаться новый диалог (или ветвь) из того же набора диалогов. Это происходит при вызове метода _begin_ и предоставлении идентификатора нового диалога. Исходный диалог по-прежнему находится в стеке, но вызовы метода _continue_ для контекста диалога отправляются только тому диалогу, который находится на вершине стека, то есть _активному диалогу_. Когда диалог извлекается из стека, в стеке возобновляется контекст диалога со следующим действием в каскадной последовательности, на котором остановился диалог.

Таким образом, можно создать ветвь в процессе общения, добавив действие в один диалог, которое условно позволяет из набора возможных диалогов выбрать диалог для начала разговора.

## <a name="how-to-loop"></a>Создание цикла

С помощью метода _replace_ контекста диалога можно заменить диалог, который находится на вершине стека. Состояние предыдущего диалога будет отклонено, и новый диалог начнется с начала. Этот метод можно использовать для создания цикла, в котором диалог будет заменяться собой. Но чтобы [сохранить состояние](bot-builder-howto-v4-state.md), вам потребуется передать сведения новому экземпляру диалога в вызове метода _replace_. В примере ниже вы увидите, что корзина заказов для ужина хранится в данных о состоянии диалога, а когда диалог `orderPrompt` повторяется, передается текущее состояние диалога, чтобы новое состояние диалога позволило продолжить добавление элементов в корзину. Если вы предпочитаете хранить эти сведения в других расположениях для данных о состоянии, см. статью [Хранение данных пользователя](bot-builder-tutorial-persist-user-inputs.md).

## <a name="create-the-dialogs-for-the-hotel-bot"></a>Создание диалогов для бота гостиницы

В этом разделе объясняется, как создавать диалоги, чтобы управлять процессом общения с описанным выше ботом гостиницы.

### <a name="install-the-dialogs-library"></a>Установка библиотеки диалогов

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
npm install --save botbuilder-dialogs
```

Чтобы использовать **диалоги** в боте, включите эту библиотеку в код бота. Например, добавьте в файл **app.js** следующее:

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

### <a name="create-a-dialog-set"></a>Создание набора диалогов

Создайте _набор диалогов_, в который мы добавим все диалоги для этого примера.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Создайте класс **HotelDialog** и добавьте инструкции using, которые нам потребуются.

```csharp
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts.Choices;
using Microsoft.Bot.Schema;
using Microsoft.Recognizers.Text;
using System;
using System.Collections.Generic;
using System.Linq;
```

Создайте производный от **DialogSet** класс и определите идентификаторы и ключи, которые потребуются нам для идентификации диалогов, запросов и сведений о состоянии для этого набора диалогов.

```csharp
/// <summary>Contains the set of dialogs and prompts for the hotel bot.</summary>
public class HotelDialog : DialogSet
{
    /// <summary>The ID of the top-level dialog.</summary>
    public const string MainMenu = "mainMenu";

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

Чтобы использовать **диалоги** в боте, включите эту библиотеку в код бота.

Создайте ссылку на библиотеку в файле **app.js**.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

Затем создайте набор диалогов.

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```

---

### <a name="add-the-prompts-to-the-set"></a>Добавление запросов в набор

Мы будем использовать класс **ChoicePrompt**, чтобы спросить у гостя, будет ли он заказывать ужин или резервировать столик, а также чтобы предложить меню для выбора блюд на ужин. А класс **NumberPrompt** мы будем использовать, чтобы узнать у гостя номер комнаты, если он решил заказать ужин.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В конструкторе **HotelDialog** добавьте два запроса.

```csharp
// Add the prompts.
this.Add(Inputs.Choice, new ChoicePrompt(Culture.English));
this.Add(Inputs.Number, new NumberPrompt<int>(Culture.English));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте эти два запроса в набор диалогов.

```javascript
dialogs.add('choicePrompt', new botbuilder_dialogs.ChoicePrompt());
dialogs.add('numberPrompt', new botbuilder_dialogs.NumberPrompt());
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
    /// <summary>The text to show the guest for this option.</summary>
    public string Description { get; set; }

    /// <summary>The ID of the associated dialog for this option.</summary>
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

    /// <summary>The name of the meal item or the request.</summary>
    public string Name { get; set; }

    /// <summary>The price of the meal item; or NaN for a request.</summary>
    public double Price { get; set; }

    /// <summary>The text to show the guest for this option.</summary>
    public string Description => (double.IsNaN(Price)) ? Name : $"{Name} - ${Price:0.00}";
}

/// <summary>Contains the lists used to present options to the guest.</summary>
private static class Lists
{
    /// <summary>The options for the top-level dialog.</summary>
    public static List<WelcomeChoice> WelcomeOptions { get; } = new List<WelcomeChoice>
    {
        new WelcomeChoice { Description = "Order dinner", DialogName = Dialogs.OrderDinner },
        new WelcomeChoice { Description = "Reserve a table", DialogName = Dialogs.ReserveTable },
    };

    private static List<string> WelcomeList { get; } = WelcomeOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the top-level dialog.</summary>
    public static List<Choice> WelcomeChoices { get; } = ChoiceFactory.ToChoices(WelcomeList);

    /// <summary>The reprompt action for the top-level dialog.</summary>
    public static Activity WelcomeReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(WelcomeList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }

    /// <summary>The options for the food-selection dialog.</summary>
    public static List<MenuChoice> MenuOptions { get; } = new List<MenuChoice>
    {
        new MenuChoice { Name = "Potato Salad", Price = 5.99 },
        new MenuChoice { Name = "Tuna Sandwich", Price = 6.89 },
        new MenuChoice { Name = "Clam Chowder", Price = 4.50 },
        new MenuChoice { Name = MenuChoice.Process, Price = double.NaN },
        new MenuChoice { Name = MenuChoice.Cancel, Price = double.NaN },
    };

    private static List<string> MenuList { get; } = MenuOptions.Select(x => x.Description).ToList();

    /// <summary>The choices to present in the choice prompt for the food-selection dialog.</summary>
    public static List<Choice> MenuChoices { get; } = ChoiceFactory.ToChoices(MenuList);

    /// <summary>The reprompt action for the food-selection dialog.</summary>
    public static Activity MenuReprompt
    {
        get
        {
            var reprompt = MessageFactory.SuggestedActions(MenuList, "Please choose an option");
            reprompt.AttachmentLayout = AttachmentLayoutTypes.List;
            return reprompt as Activity;
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Создайте константу **dinnerMenu**, которая будет содержать эти сведения.

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

В этом диалоге используется метод `ChoicePrompt` для отображения меню, затем пользователь должен выбрать пункт меню. Когда пользователь выбирает `Order dinner` или `Reserve a table`, запускается диалог для соответствующего варианта. После завершения диалог `mainMenu` не завершается на последнем шаге, оставляя пользователя в недоумении от происходящего, а запускается повторно. Бот с такой простой структурой всегда будет выводить меню, и пользователь всегда будете знать, что он выбрал.

Диалог **MainMenu** содержит такую каскадную последовательность действий:

* На первом шаге каскадной последовательности мы инициализируем или удаляем состояние диалога, приветствуем гостя и просим его выбрать один из доступных вариантов: `Order dinner` или `Reserve a table`.
* На втором шаге мы получаем результат выбора и запускаем дочерний диалог, связанный с выбранным вариантом. Когда дочерний диалог завершается, возобновляется диалог со следующим действием.
* На последнем шаге мы используем метод **DialogContext.Replace**, чтобы заменить этот диалог его же новым экземпляром, что фактически превращает приветственный диалог в цикл вывода меню.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the main welcome dialog.
this.Add(MainMenu, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Greet the guest and ask them to choose an option.
        await dc.Context.SendActivity("Welcome to Contoso Hotel and Resort.");
        await dc.Prompt(Inputs.Choice, "How may we serve you today?", new ChoicePromptOptions
        {
            Choices = Lists.WelcomeChoices,
            RetryPromptActivity = Lists.WelcomeReprompt,
        });
    },
    async (dc, args, next) =>
    {
        // Begin a child dialog associated with the chosen option.
        var choice = (FoundChoice)args["Value"];
        var dialogId = Lists.WelcomeOptions[choice.Index].DialogName;

        await dc.Begin(dialogId, dc.ActiveDialog.State);
    },
    async (dc, args, next) =>
    {
        // Start this dialog over again.
        await dc.Replace(MainMenu);
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Display a menu and ask user to choose a menu item. Direct user to the item selected otherwise, show
// the menu again.
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
    },
    async function(dc, result){
        // Start over
        await dc.endAll().begin('mainMenu');
    }
]);
```

---

### <a name="create-the-order-dinner-dialog"></a>Создание диалога для заказа ужина

В диалоге для заказа ужина мы приветствуем гостя в службе заказа, после чего сразу запускается диалог для выбора блюд, который мы рассмотрим в следующем разделе. Важно отметить, что если гость просит службу обработать заказ, в диалоге возвращается список заказанных блюд. Чтобы завершить весь процесс, в диалоге запрашивается номер комнаты, в которую нужно доставить заказ, а затем отправляется сообщение с подтверждением. Если гость отменяет заказ, диалог не запрашивает номер комнаты, а сразу же завершается.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В класс **HotelDialog** добавьте структуру данных, которая будет использоваться для отслеживания заказа гостя.

```csharp
/// <summary>Contains the guest's dinner order.</summary>
private class OrderCart : List<MenuChoice> { }
```

В конструкторе **HotelDialog** добавьте диалог для заказа ужина.

* На первом шаге мы отправляем приветственное сообщение и запускаем диалог для выбора блюд.
* На втором шаге мы проверяем, возвращает ли диалог для выбора блюд корзину заказов.
  * Если возвращает, мы запрашиваем у гостя номер комнаты и сохраняем данные корзины.
  * Если не возвращает, считаем, что гость отменил заказа и вызываем **DialogContext.End**, чтобы завершить диалог.
* На последнем шаге мы записываем номер комнаты гостя и отправляем ему сообщение с подтверждением, чтобы завершить диалог. На этом этапе бот отправляет заказ в службу на обработку.

```csharp
// Add the order-dinner dialog.
this.Add(Dialogs.OrderDinner, new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        await dc.Context.SendActivity("Welcome to our Dinner order service.");

        // Start the food selection dialog.
        await dc.Begin(Dialogs.OrderPrompt);
    },
    async (dc, args, next) =>
    {
        if (args.TryGetValue(Outputs.OrderCart, out object arg) && arg is OrderCart cart)
        {
            // If there are items in the order, record the order and ask for a room number.
            dc.ActiveDialog.State[Outputs.OrderCart] = cart;
            await dc.Prompt(Inputs.Number, "What is your room number?", new PromptOptions
            {
                RetryPromptString = "Please enter your room number."
            });
        }
        else
        {
            // Otherwise, assume the order was cancelled by the guest and exit.
            await dc.End();
        }
    },
    async (dc, args, next) =>
    {
        // Get and save the guest's answer.
        var roomNumber = args["Text"] as string;
        dc.ActiveDialog.State[Outputs.RoomNumber] = roomNumber;

        // Process the dinner order using the collected order cart and room number.

        await dc.Context.SendActivity($"Thank you. Your order will be delivered to room {roomNumber} within 45 minutes.");
        await dc.End();
    },
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Order dinner:
// Help user order dinner from a menu
dialogs.add('orderDinner', [
    async function (dc){
        await dc.context.sendActivity("Welcome to our Dinner order service.");
        
        await dc.begin('orderPrompt', dc.activeDialog.state.orderCart); // Prompt for orders
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

В конструкторе **HotelDialog** добавьте диалог для выбора блюд.

* На первом шаге мы инициализируем состояние диалога. Если входные аргументы для диалога содержат сведения о корзине, мы сохраняем их в состоянии о диалоге. В противном случае мы создаем и добавляем пустую корзину. Затем создаем запрос для гостя на выбор блюд из меню ужина.
* На следующем шаге изучаем вариант, выбранный гостем.
  * Если это запрос на обработку заказа, проверяем содержит ли корзина какие-либо элементы.
    * Если да, диалог завершается, возвращая содержимое корзины.
    * В противном случае гостю отправляется сообщения об ошибке и диалог начинается сначала.
  * Если это запрос на отмену заказа, диалог завершается, возвращая пустой словарь.
  * Если это выбранное на ужин блюдо, оно добавляется в корзину, отправляется сообщение о состоянии и запускается диалог, передавая текущее состояние заказа.

```csharp
// Add the food-selection dialog.
this.Add(Dialogs.OrderPrompt, new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            if (args is null || !args.ContainsKey(Outputs.OrderCart))
            {
                // First time through, initialize the order state.
                dc.ActiveDialog.State[Outputs.OrderCart] = new OrderCart();
                dc.ActiveDialog.State[Outputs.OrderTotal] = 0.0;
            }
            else
            {
                // Otherwise, set the order state to that of the arguments.
                dc.ActiveDialog.State = new Dictionary<string, object>(args);
            }

            await dc.Prompt(Inputs.Choice, "What would you like?", new ChoicePromptOptions
            {
                Choices = Lists.MenuChoices,
                RetryPromptActivity = Lists.MenuReprompt,
            });
        },
        async (dc, args, next) =>
        {
            // Get the guest's choice.
            var choice = (FoundChoice)args["Value"];
            var option = Lists.MenuOptions[choice.Index];

            // Get the current order from dialog state.
            var cart = (OrderCart)dc.ActiveDialog.State[Outputs.OrderCart];

            if (option.Name is MenuChoice.Process)
            {
                if (cart.Count > 0)
                {
                    // If there are any items in the order, then exit this dialog,
                    // and return the list of selected food items.
                    await dc.End(new Dictionary<string, object>
                    {
                        [Outputs.OrderCart] = cart
                    });
                }
                else
                {
                    // Otherwise, send an error message and restart from
                    // the beginning of this dialog.
                    await dc.Context.SendActivity(
                        "Your cart is empty. Please add at least one item to the cart.");
                    await dc.Replace(Dialogs.OrderPrompt);
                }
            }
            else if (option.Name is MenuChoice.Cancel)
            {
                await dc.Context.SendActivity("Your order has been cancelled.");

                // Exit this dialog, returning an empty property bag.
                dc.ActiveDialog.State.Clear();
                await dc.End(new Dictionary<string, object>());
            }
            else
            {
                // Add the selected food item to the order and update the order total.
                cart.Add(option);
                var total = (double)dc.ActiveDialog.State[Outputs.OrderTotal] + option.Price;
                dc.ActiveDialog.State[Outputs.OrderTotal] = total;

                await dc.Context.SendActivity($"Added {option.Name} (${option.Price:0.00}) to your order." +
                    Environment.NewLine + Environment.NewLine +
                    $"Your current total is ${total:0.00}.");

                // Present the order options again, passing in the current order state.
                await dc.Replace(Dialogs.OrderPrompt, dc.ActiveDialog.State);
            }
        },
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc, orderCart){
        // Define a new cart of one does not exists
        if(!orderCart){
            // Initialize a new cart
            // convoState = conversationState.get(dc.context);
            dc.activeDialog.state.orderCart = {
                orders: [],
                total: 0
            };
        }
        else {
            dc.activeDialog.state.orderCart = orderCart;
        }
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        // Get state object
        // convoState = conversationState.get(dc.context);

        if(choice.value.match(/process order/ig)){
            if(dc.activeDialog.state.orderCart.orders.length > 0) {
                // Process the order
                // ...
                dc.activeDialog.state.orderCart = undefined; // Reset cart
                await dc.context.sendActivity("Processing your order.");
                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            //dc.activeDialog.state.orderCart = undefined; // Reset cart
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else {
            var item = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!item){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                dc.activeDialog.state.orderCart.orders.push(item);
                dc.activeDialog.state.orderCart.total += item.Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${dc.activeDialog.state.orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt', dc.activeDialog.state.orderCart);
            }
        }
    }
]);
```

В приведенном выше примере кода основной диалог `orderDinner` использует вспомогательный диалог с именем `orderPrompt` для обработки выбираемых пользователем вариантов. Диалог `orderPrompt` отображает меню с вариантами блюд, предлагает пользователю выбрать элемент, добавляет его в корзину и предлагает выбрать элемент еще раз в рамках цикла. Это позволяет пользователю добавить несколько элементов в заказ. Цикл в диалоге продолжает работать, пока пользователь не выберет `Process order` или `Cancel`. В этот момент выполнение возвращается в родительский диалог (в диалог `orderDinner`). Диалог `orderDinner` выполняет последние необходимые действия в случае, если пользователь хочет обработать заказ; в противном случае он завершается и возвращает выполнение в родительский диалог (например: `mainMenu`). Диалог `mainMenu`, в свою очередь, выполняет последнее действие, которое представляет собой повторное отображение элементов главного меню.

---
### <a name="create-the-reserve-table-dialog"></a>Создание диалога для резервирования столика

<!-- TODO: Update the "Manage simple conversation flows" topic to use a reserveTable dialog, and then reference that from here. -->

Для простоты в этом примере мы покажем только минимальную реализацию диалога `reserveTable`.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Add the table-reservation dialog.
this.Add(Dialogs.ReserveTable, new WaterfallStep[]
    {
        // Replace this waterfall with your reservation steps.
        async (dc, args, next) =>
        {
            await dc.Context.SendActivity("Your table has been reserved.");
            await dc.End();
        }
    });
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    // Replace this waterfall with your reservation steps.
    async function(dc, args, next){
        await dc.context.sendActivity("Your table has been reserved");
        await dc.end();
    }
]);
```

---

## <a name="next-steps"></a>Дополнительная информация

Вы можете расширить этот бот, добавив в меню другие варианты, например "Дополнительные сведения" или "Справка". Дополнительные сведения о реализации таких прерываний см. в статье [Обработка прерываний со стороны пользователя](bot-builder-howto-handle-user-interrupt.md).

Теперь, когда вы знаете, как использовать диалоги, запросы и каскады для управления сложным процессом общения, рассмотрим, как разбить наши диалоги (`orderDinner` и`reserveTable`) на отдельные объекты вместо того, чтобы собирать их вместе в одном большом наборе диалогов.

> [!div class="nextstepaction"]
> [Создание модульной логики бота с помощью составных элементов управления](bot-builder-compositcontrol.md)

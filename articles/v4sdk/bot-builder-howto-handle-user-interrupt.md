---
title: Обработка прерываний со стороны пользователя | Документация Майкрософт
description: Узнайте, как обрабатывать прерывания со стороны пользователя и прямой поток беседы.
keywords: interrupt, interruptions, switching topic, break
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/17/2018
ms.reviewer: ''
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 651a7410893f7a66f5941121edc7b34055807ba7
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301263"
---
# <a name="handle-user-interrupt"></a>Обработка прерываний со стороны пользователя

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Обработка прерываний со стороны пользователя является важным аспектом надежных ботов. При этом вы можете подумать, что ваши пользователи будут следить за потоком беседы шаг за шагом, но скорее всего они изменят свое мнение или вместо того, чтобы отвечать, зададут вопрос в середине процесса. В таких ситуациях как вы хотите, чтобы бот обрабатывал входные данные пользователя? На что должно быть похоже взаимодействие с пользователем? Как поддерживать пользовательские данные о состоянии?

Нет единственного правильного ответа на эти вопросы, так как каждая ситуация является уникальной для сценария, который обрабатывает ваш бот. В этом разделе рассматривается несколько распространенных способов обработки прерываний со стороны пользователя и предлагаются некоторые способы их реализации в боте.

## <a name="handle-expected-interruptions"></a>Обработка ожидаемых прерываний

Поток процедурной беседы имеет базовый набор шагов, которым должен следовать пользователь, и любые его действия, которые отличаются от этих шагов, являются потенциальными прерываниями. В обычном потоке существуют ожидаемые прерывания.

**Резервирование столов.** Основные шаги бота, который резервирует стол, могут заключаться в том, чтобы запросить у пользователя задать дату и время, количество людей и имя для резервирования. Некоторые ожидаемые прерывания в этом процессе, которые можно спрогнозировать: 
 * `cancel`. Выход из процесса.
 * `help`. Предоставление дополнительных сведений об этом процессе.
 * `more info`. Предоставление указаний и предложений или альтернативных способов резервирования столов (например, адрес электронной почты или номер телефона для связи).
 * `show list of available tables`. Отображение списка доступных столов на дату и время, которое нужно пользователю (если возможно).

**Заказ ужина.** В боте заказа ужина основные действия включают предоставление списка пунктов меню и разрешение пользователю добавлять предметы в корзину. Некоторые ожидаемые прерывания в этом процессе, которые можно спрогнозировать: 
 * `cancel`. Выход из процесса заказа.
 * `more info`. Предоставление диетических сведений о каждом пункте меню.
 * `help`. Предоставление справки по использованию системы.
 * `process order`. Обработка заказа.

Можно предоставить их пользователю как список **предлагаемых действий** или как указание, чтобы он, по крайней мере, знал, какие команды нужно отправить боту, чтобы он понял.

Например, в потоке заказа ужина ожидаемые прерывания можно указать вместе с элементами меню. В этом случае элементы меню отправляются как массив `choices`.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
public class dinnerItem
{
    public string Description;
    public double Price;
}

public class dinnerMenu
{
    static public Dictionary<string, dinnerItem> dinnerChoices = new Dictionary<string, dinnerItem>
    {
        { "potato salad", new dinnerItem { Description="Potato Salad", Price=5.99 } },
        { "tuna sandwich", new dinnerItem { Description="Tuna Sandwich", Price=6.89 } },
        { "clam chowder", new dinnerItem { Description="Clam Chowder", Price=4.50 } }
    };

    static public string[] choices = new string[] {"Potato Salad", "Tuna Sandwich", "Clam Chowder", "more info", "Process order", "help", "Cancel"};
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
var dinnerMenu = {
    choices: ["Potato Salad - $5.99", "Tuna Sandwich - $6.89", "Clam Chowder - $4.50", 
            "more info", "Process order", "Cancel"],
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

В логике упорядочивания их можно проверить с помощью строки сопоставления или регулярных выражений.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

Сначала необходимо определить вспомогательное приложение для отслеживания заказов.

```cs
// Helper class for storing the order in the dictionary
public class Orders
{
    public double total;
    public string order;
    public bool processOrder;

    // Initialize order values
    public Orders()
    {
        total = 0;
        order = "";
        processOrder = false;
    }
}
```

Затем добавить диалог в бот.

```cs
dialogs.Add("orderPrompt", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt the user
        await dc.Prompt("choicePrompt",
            "What would you like for dinner?",
            new ChoicePromptOptions
            {
                Choices = dinnerMenu.choices.Select( s => new Choice { Value = s }).ToList(),
                RetryPromptString = "I'm sorry, I didn't understand that. What would you " +
                    "like for dinner?"
            });
    },
    async(dc, args, next) =>
    {
        var convo = ConversationState<Dictionary<string,object>>.Get(dc.Context);

        // Get the user's choice from the previous prompt
        var response = (args["Value"] as FoundChoice).Value.ToLower();

        if(response == "process order")
        {
            try 
            {
                var order = convo["order"];

                await dc.Context.SendActivity("Order is on it's way!");
                
                // In production, you may want to store something more helpful, 
                // such as send order off to be made
                (order as Orders).processOrder = true;

                // Once it's submitted, clear the current order
                convo.Remove("order");
                await dc.End();
            }
            catch
            {
                await dc.Context.SendActivity("Your order is empty, please add your order choice");
                // Ask again
                await dc.Replace("orderPrompt");
            }
        }
        else if(response == "cancel" )
        {
            // Get rid of current order
            convo.Remove("order");
            await dc.Context.SendActivity("Your order has been canceled");
            await dc.End();
        }
        else if(response == "more info")
        {
            // Send more information about the options
            var msg = "More info: <br/>" +
                "Potato Salad: contains 330 calaries per serving. Cost: 5.99 <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. Cost: 6.89 <br/>"
                + "Clam Chowder: contains 650 calaries per serving. Cost: 4.50";
            await dc.Context.SendActivity(msg);

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else if(response == "help")
        {
            // Provide help information
            await dc.Context.SendActivity("To make an order, add as many items to your cart as " +
                "you like then choose the \"Process order\" option to check out.");

            // Ask again
            await dc.Replace("orderPrompt");
        }
        else
        {
            // Unlikely to get past the prompt verification, but this will catch 
            // anything that isn't a valid menu choice
            if(!dinnerMenu.dinnerChoices.ContainsKey(response))
            {
                await dc.Context.SendActivity("Sorry, that is not a valid item. " +
                    "Please pick one from the menu.");
    
                // Ask again
                await dc.Replace("orderPrompt");
            }
            else {
                // Add the item to cart
                Orders currentOrder;

                // If there is a current order going, add to it. If not, start a new one
                try
                {
                    currentOrder = convo["order"] as Orders;
                }
                catch
                {
                    convo["order"] = new Orders();
                    currentOrder = convo["order"] as Orders;
                }

                // Add to the current order
                currentOrder.order += (dinnerMenu.dinnerChoices[$"{response}"].Description) + ", ";
                currentOrder.total += (double)dinnerMenu.dinnerChoices[$"{response}"].Price;

                // Save back to the conversation state
                convo["order"] = currentOrder;

                await dc.Context.SendActivity($"Added to cart. Current order: " +
                    $"{currentOrder.order} " +
                    $"<br/>Current total: ${currentOrder.total}");

                // Ask again to allow user to add more items or process order
                await dc.Replace("orderPrompt");
            }
        }
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Helper dialog to repeatedly prompt user for orders
dialogs.add('orderPrompt', [
    async function(dc){
        await dc.prompt('choicePrompt', "What would you like?", dinnerMenu.choices);
    },
    async function(dc, choice){
        if(choice.value.match(/process order/ig)){
            if(orderCart.orders.length > 0) {
                // Process the order
                dc.context.activity.conversation.dinnerOrder = orderCart;

                await dc.end();
            }
            else {
                await dc.context.sendActivity("Your cart was empty. Please add at least one item to the cart.");
                // Ask again
                await dc.replace('orderPrompt');
            }
        }
        else if(choice.value.match(/cancel/ig)){
            orderCart.clear(context);
            await dc.context.sendActivity("Your order has been canceled.");
            await dc.end(choice.value);
        }
        else if(choice.value.match(/more info/ig)){
            var msg = "More info: <br/>Potato Salad: contains 330 calaries per serving. <br/>"
                + "Tuna Sandwich: contains 700 calaries per serving. <br/>" 
                + "Clam Chowder: contains 650 calaries per serving."
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else if(choice.value.match(/help/ig)){
            var msg = `Help: <br/>To make an order, add as many items to your cart as you like then choose the "Process order" option to check out.`
            await dc.context.sendActivity(msg);
            
            // Ask again
            await dc.replace('orderPrompt');
        }
        else {
            var choice = dinnerMenu[choice.value];

            // Only proceed if user chooses an item from the menu
            if(!choice){
                await dc.context.sendActivity("Sorry, that is not a valid item. Please pick one from the menu.");
                
                // Ask again
                await dc.replace('orderPrompt');
            }
            else {
                // Add the item to cart
                orderCart.orders.push(choice);
                orderCart.total += dinnerMenu[choice.value].Price;

                await dc.context.sendActivity(`Added to cart: ${choice.value}. <br/>Current total: $${orderCart.total}`);

                // Ask again
                await dc.replace('orderPrompt');
            }
        }
    }
]);
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
Для прерываний, которые находятся вне области функций, можно попытаться угадать намерения пользователя. Вы можете это сделать с помощью служб искусственного интеллекта, например QnAMaker, LUIS или пользовательской логики, затем предложить варианты предположений бота о пользовательских намерениях. 

Например, в середине потока резервирования стола пользователь говорит: "Я хочу заказать гамбургер". В этом случае бот не знает, как обработать это намерение из этой последовательности диалога. Так как текущий поток не имеет ничего общего с заказом, а другая команда беседы бота — "заказать ужин", бот не знает, что делать с этими входными данными. Например, если применить LUIS, вы сможете обучить модель на распознавание того, что пользователь хочет заказать еду (например, LUIS может возвращать намерение orderFood). Таким образом бот может отвечать: "Похоже, что вы хотите заказать еду. Вы хотите переключиться на процесс заказа ужина?" Дополнительные сведения об обучении LUIS и обнаружении намерений пользователя см. в статье [Using LUIS for Language Understanding](bot-builder-howto-v4-luis.md) (Использование LUIS для распознавания речи).

### <a name="default-response"></a>Ответ по умолчанию
Если все остальное не удастся, можно отправить ответ по умолчанию, вместо того чтобы ничего не делать и оставлять пользователя недоумевать о том, что происходит. Ответ по умолчанию должен сообщать, какие команды понимает бот, чтобы пользователь мог вернуться в нужное русло.

Вы можете проверить контекст флага **ответа** в конце логики бота, чтобы узнать, отправил ли бот что-либо пользователю в ответ. Если бот обрабатывает входные данные пользователя, но не отвечает, скорее всего, бот не знает, что делать со входными данными. В этом случае можно перехватить и отправить пользователю сообщение по умолчанию.

По умолчанию этот бот предлагает пользователю диалог `mainMenu`. Именно вы решаете, какие возможности получит пользователь от вашего бота в такой ситуации.

# <a name="ctabcsharptab"></a>[C#](#tab/csharptab)

```cs
if(!context.Responded)
{
    await dc.EndAll().Begin("mainMenu");
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Check to see if anyone replied. If not then clear all the stack and present the main menu
if (!context.responded) {
    await dc.endAll().begin('mainMenu');
}
```

---


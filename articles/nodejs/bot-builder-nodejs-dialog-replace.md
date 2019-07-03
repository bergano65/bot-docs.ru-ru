---
title: Замена диалогов | Документация Майкрософт
description: Узнайте, как заменить диалоги, чтобы повторно запросить входные данные и управлять потоком беседы, используя пакет SDK Bot Framework для Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 7417e105ab20c3aca2c2a4e525248728ac49bf18
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404923"
---
# <a name="replace-dialogs"></a>Замена диалогов

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Возможность замены диалога может быть полезна для проверки входящих пользовательских данных или для повтора действия в ходе общения. Пакет SDK Bot Framework для Node.js позволяет заменить диалог с помощью метода [`session.replaceDialog`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog). Этот метод позволяет завершить текущий диалог и заменить его на новый без возврата к вызывающей стороне. 

## <a name="create-custom-prompts-to-validate-input"></a>Создание пользовательских запросов для проверки входных данных

Пакет SDK Bot Builder для Node.js включает проверку входных данных для некоторых типов [запросов](bot-builder-nodejs-dialog-prompt.md), например `Prompts.time` и `Prompts.choice`. Для проверки ввода текста, который вы получаете в ответ на `Prompts.text`, необходимо создать собственную логику проверки и пользовательские запросы. 

Может потребоваться проверка входных данных, если они соответствуют определенным значениям, шаблону, диапазону или критериям, которые определяете вы. Если входящие данные не проходят проверку, бот может запрашивать эти сведения еще раз с помощью метода `session.replaceDialog`.

В следующем примере кода показано, как создать пользовательский запрос для проверки входящих данных для номера телефона.

```javascript
// This dialog prompts the user for a phone number. 
// It will re-prompt the user if the input does not match a pattern for phone number.
bot.dialog('phonePrompt', [
    function (session, args) {
        if (args && args.reprompt) {
            builder.Prompts.text(session, "Enter the number using a format of either: '(555) 123-4567' or '555-123-4567' or '5551234567'")
        } else {
            builder.Prompts.text(session, "What's your phone number?");
        }
    },
    function (session, results) {
        var matched = results.response.match(/\d+/g);
        var number = matched ? matched.join('') : '';
        if (number.length == 10 || number.length == 11) {
            session.userData.phoneNumber = number; // Save the number.
            session.endDialogWithResult({ response: number });
        } else {
            // Repeat the dialog
            session.replaceDialog('phonePrompt', { reprompt: true });
        }
    }
]);
```

В этом примере пользователю сначала предлагается указать номер своего телефона. Логика проверки использует регулярное выражение, которое соответствует ряду цифр из входных данных пользователя. Если входные данные содержат цифры 10 или 11, это число возвращается в ответе. В противном случае выполняется метод `session.replaceDialog`, чтобы повторить диалог `phonePrompt`, в котором снова запрашиваются входные данные пользователя, на этот раз предоставляя более конкретные указания относительно формата ожидаемых входных данных.

При вызове метода `session.replaceDialog` укажите имя диалога, который нужно повторить, и список аргументов. В этом примере список аргументов содержит `{ reprompt: true }`, позволяющую ботам предоставлять разные запрашивающие сообщения, в зависимости от того, является ли это начальным запросом или повторным, но можно указать любые аргументы, которые могут понадобиться вашему боту. 

## <a name="repeat-an-action"></a>Повтор действия

В ходе разговора могут появляться моменты, когда нужно повторить диалог, чтобы пользователь мог выполнить определенное действие несколько раз. Например, если бот предлагает широкий спектр служб, изначально может понадобиться отображение меню служб, затем ознакомление пользователя с процессом запроса службы, а затем снова меню службы, что позволяет пользователю запросить другую службу. Для этого можно использовать метод [`session.replaceDialog`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#replacedialog), чтобы отобразить меню служб снова, а не заканчивать диалог с помощью метода [session.endConversation](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation). 

В следующем примере показано, как использовать метод `session.replaceDialog` для реализации сценария такого типа. Сначала определяется меню служб:

```javascript
// Main menu
var menuItems = { 
    "Order dinner": {
        item: "orderDinner"
    },
    "Dinner reservation": {
        item: "dinnerReservation"
    },
    "Schedule shuttle": {
        item: "scheduleShuttle"
    },
    "Request wake-up call": {
        item: "wakeupCall"
    },
}
```

Диалог `mainMenu` вызывается по умолчанию, поэтому меню для пользователя будет отображаться в начале диалога. Кроме того, [`triggerAction`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction) присоединяется к диалогу `mainMenu`, чтобы меню отображалось в любое время, когда входные данные пользователя — "main menu". Когда меню отобразится для пользователя и он выберет параметр, соответствующий выбору диалог вызовется с помощью метода `session.beginDialog`.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a reservation bot that has a menu of offerings.
var bot = new builder.UniversalBot(connector, [
    function(session){
        session.send("Welcome to Contoso Hotel and Resort.");
        session.beginDialog("mainMenu");
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Display the main menu and start a new request depending on user input.
bot.dialog("mainMenu", [
    function(session){
        builder.Prompts.choice(session, "Main Menu:", menuItems);
    },
    function(session, results){
        if(results.response){
            session.beginDialog(menuItems[results.response.entity].item);
        }
    }
])
.triggerAction({
    // The user can request this at any time.
    // Once triggered, it clears the stack and prompts the main menu again.
    matches: /^main menu$/i,
    confirmPrompt: "This will cancel your request. Are you sure?"
});
```

В этом примере, если пользователь выбирает вариант 1 для заказа обеда с доставкой, будет вызываться диалог `orderDinner`, который проведет пользователя через процесс заказа обеда. В конце процесса бот подтвердит заказ и отобразит главное меню снова с помощью метода `session.replaceDialog`.

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner to be delivered to their hotel room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: %(Description)s for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}.`;
            session.send(msg);
            session.replaceDialog("mainMenu"); // Display the menu again.
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
)
.cancelAction(
    "cancelOrder", "Type 'Main Menu' to continue.", 
    {
        matches: /^cancel$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

Два триггера, позволяющие пользователю начать заново ("start over") или отменить ("cancel") заказ в любое время процесса заказа, прикреплены к диалогу `orderDinner`. 

Первый триггер — [`reloadAction`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction). Он позволяет пользователю начать процесс заказа заново, отправляя входные данные "start over". Если триггер соответствует высказыванию "start over", `reloadAction` перезапускает диалог с самого начала. 

Второй триггер — [`cancelAction`](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction). Он позволяет пользователю прервать процесс заказа полностью, отправляя входные данные "cancel". Этот триггер автоматически не отображает главное меню, но вместо этого отправляет сообщение, которое сообщает пользователю, что делать дальше, указывая "Введите "Main Menu", чтобы продолжить".

### <a name="dialog-loops"></a>Циклы диалогов

В приведенном выше примере пользователь может выбрать один элемент для каждого заказа. То есть если пользователь хочет заказать два элемента из меню, он должен завершить весь процесс заказа первого элемента, а затем снова повторить весь процесс для второго элемента. 

В следующем примере показано, как улучшить предыдущий бот путем рефакторинга меню обеда в отдельный диалог. Это позволяет боту повторять меню обеда циклически и, следовательно, позволяет пользователю выбирать несколько элементов в пределах одного заказа.

Сначала добавьте параметр "Извлечь" в меню. Данный параметр позволяет пользователю выйти из процесса выбора элемента и продолжить извлечение.

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0 // Order total. Updated as items are added to order.
    }
};
```

Затем выполните рефакторинг запроса заказа в его собственный диалог, чтобы бот мог повторить меню и позволить пользователю добавить несколько элементов в свой заказ.

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    function(session, args){
        if(args && args.reprompt){
            session.send("What else would you like to have for dinner tonight?");
        }
        else{
            // New order
            // Using the conversationData to store the orders
            session.conversationData.orders = new Array();
            session.conversationData.orders.push({ 
                Description: "Check out",
                Price: 0
            })
        }
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else {
                var order = dinnerMenu[results.response.entity];
                session.conversationData.orders[0].Price += order.Price; // Add to total.
                var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
                session.send(msg);
                session.conversationData.orders.push(order);
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu
            }
        }
    }
])
.reloadAction(
    "restartOrderDinner", "Ok. Let's start over.",
    {
        matches: /^start over$/i
    }
);
```

В этом примере заказы сохраняются в хранилище данных ботов, которое ограничено текущей беседой `session.conversationData.orders`. Для каждого нового заказа переменная повторно инициализируется новым массивом, и бот добавляет каждый выбранный пользователем элемент в массив `orders` и добавляет цену к общей сумме, которая хранится в контрольной переменной `Price`. Когда пользователь завершает выбор элементов для заказа, он может сказать "Извлечь", а затем продолжить остальную часть процесса заказа.

> [!NOTE]
> Дополнительные сведения о хранилище данных бота см. в разделе [Управление данными состояния](bot-builder-nodejs-state.md). 

Наконец, обновите второй шаг каскада в диалоге `orderDinner`, чтобы обработать заказ с подтверждением. 

```javascript
// Menu: "Order dinner"
// This dialog allows user to order dinner and have it delivered to their room.
bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        session.beginDialog("addDinnerItem");
    },
    function (session, results) {
        if (results.response) {
            // Display itemize order with price total.
            for(var i = 1; i < session.conversationData.orders.length; i++){
                session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
            }
            session.send(`Your total is: $${session.conversationData.orders[0].Price}`);

            // Continue with the check out process.
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${results.response}`;
            session.send(msg);
            session.replaceDialog("mainMenu");
        }
    }
])
//...attached triggers...
;
```

## <a name="cancel-a-dialog"></a>Отмена диалога

Хотя метод `session.replaceDialog` можно использовать для замены *текущего* диалога на новый, его нельзя использовать для замены диалога, находящегося ниже по стеку диалога. Для замены диалога в стеке, который не является*текущим*, используйте метод [`session.cancelDialog`](http://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#canceldialog) вместо этого. 

Метод `session.cancelDialog` можно использовать для завершения диалога, независимо от того, где он находится в стеке диалога, и вызова нового диалога вместо текущего при необходимости. Для вызова метода `session.cancelDialog` укажите идентификатор диалога для отмены и при необходимости укажите идентификатор диалога, который будет вызван вместо него. Например, этот фрагмент кода отменяет диалог `orderDinner` и заменяет его на диалог `mainMenu`:

```javascript
session.cancelDialog('orderDinner', 'mainMenu'); 
```

Когда вызывается метод `session.cancelDialog`, поиск по стеку диалога будет выполняться в обратном направлении и первое вхождение этого диалога будет отменено, в результате чего диалог и его дочерние диалоги (если таковые имеются) будут удалены из стека диалога. Затем управление будет возвращено в вызывающий диалог, который может проверить код [`results.resumed`](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.ipromptresult.html#resumed), равный [`ResumeReason.notCompleted`](http://docs.botframework.com/node/builder/chat-reference/enums/_botbuilder_d_.resumereason.html#notcompleted), чтобы обнаружить отмену.

В качестве альтернативы указанию идентификатора диалога для отмены при вызове метода `session.cancelDialog` можно вместо идентификатора указать нулевой индекс диалога для отмены, где индекс представляет позицию диалога в стеке диалога. Например, следующий фрагмент кода завершает текущий активный диалог (индекс = 0) и запускает диалог `mainMenu` вместо него. Диалог `mainMenu` вызывается в положении 0 стека диалога, тем самым становясь новым диалогом по умолчанию.

```javascript
session.cancelDialog(0, 'mainMenu');
```

Рассмотрим пример, описанный выше в [циклах диалога](#dialog-loops). При достижении пользователем меню выбора элемента этот диалог (`addDinnerItem`) является четвертым диалогом в стеке: `[default dialog, mainMenu, orderDinner, addDinnerItem]`. Как предоставить пользователю возможность отменить свой заказ в диалоге `addDinnerItem`? При добавлении триггера `cancelAction` в диалог `addDinnerItem` он вернет пользователя в предыдущий диалог (`orderDinner`), который перенаправит пользователя в диалог `addDinnerItem`.

Именно в этой ситуации полезен метод `session.cancelDialog`. Добавьте "cancel order" в качестве явного параметра в меню обеда, начиная с [примера циклов диалога](#dialog-loops).

```javascript
// The dinner menu
var dinnerMenu = { 
    //...other menu items...,
    "Check out": {
        Description: "Check out",
        Price: 0      // Order total. Updated as items are added to order.
    },
    "Cancel order": { // Cancel the order and back to Main Menu
        Description: "Cancel order",
        Price: 0
    }
};
```

Затем обновите диалог `addDinnerItem` для проверки запроса "cancel order". Если обнаружено "cancel", используйте метод `session.cancelDialog`, чтобы отменить диалог по умолчанию (то есть диалог с индексом стека 0) и вызвать диалог `mainMenu` вместо него. 

```javascript
// Add dinner items to the list by repeating this dialog until the user says `check out`. 
bot.dialog("addDinnerItem", [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            if(results.response.entity.match(/^check out$/i)){
                session.endDialog("Checking out...");
            }
            else if(results.response.entity.match(/^cancel/i)){
                // Cancel the order and start "mainMenu" dialog.
                session.cancelDialog(0, "mainMenu");
            }
            else {
                //...add item to list and prompt again...
                session.replaceDialog("addDinnerItem", { reprompt: true }); // Repeat dinner menu.
            }
        }
    }
])
//...attached triggers...
;
```

Используя метод `session.cancelDialog` таким образом, вы можете реализовать любой поток диалога, который требует бот.

## <a name="next-steps"></a>Дополнительная информация

Как видите, чтобы выполнить задачу с заменой диалогов в стеке, используются различные виды **действий**. Действия обеспечивают отличную гибкость при управлении потоком диалога. Давайте более подробно рассмотрим **действия** и лучший способ обработки действий пользователя.

> [!div class="nextstepaction"]
> [Обработка действий пользователя](bot-builder-nodejs-dialog-actions.md)

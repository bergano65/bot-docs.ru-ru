---
title: Управление последовательностью общения с помощью диалогов | Документация Майкрософт
description: Сведения об управлении общением между ботом и пользователем с помощью диалогов в пакете SDK Bot Builder для Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: d6c8ad06b9fb198e684deae26e9cbad05a86a611
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39306175"
---
# <a name="manage-conversation-flow-with-dialogs"></a>Управление последовательностью общения с помощью диалогов
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

Управление ходом общения является важнейшей задачей при разработке ботов. У бота должна быть возможность элегантно выполнять основные задачи и корректно обрабатывать прерывания. При использовании пакета SDK Bot Builder для Node.js с помощью диалогов можно управлять последовательностью общения.

Диалог — это как функция в программе. Обычно он предназначен для выполнения определенной операции и может вызываться так часто, как это необходимо. Чтобы поддержать практически любое общение можно объединить несколько диалогов в цепочку. Пакет SDK Bot Builder для Node.js обладает такими встроенными функциями как [запросы](bot-builder-nodejs-dialog-prompt.md) и [каскады](bot-builder-nodejs-dialog-waterfall.md), которые помогают управлять ходом общения.

В этой статье приведен ряд примеров, объясняющих, как управлять простыми и сложными сеансами общения, в которых бот с помощью диалогов может обрабатывать прерывания и корректно возобновлять последовательность общения. Примеры основываются на следующих сценариях. 

1. Бот будет резервировать столик для ужина.
2. Бот может обрабатывать запрос "Help" в любое время в период резервирования.
3. На текущем шаге резервирования бот может обрабатывать контекстно-зависимый запрос "Help".
4. Бот может обрабатывать несколько тем общения.

## <a name="manage-conversation-flow-with-a-waterfall"></a>Управление ходом общения с помощью каскада

[Каскад](bot-builder-nodejs-dialog-waterfall.md) — это диалог, с помощью которого бот может легко провести пользователя через ряд задач. Например, бот резервирования задаст пользователю ряд вопросов, которые помогут обработать запрос на резервирование. Бот предложит пользователю ввести следующую информацию:

1. дата и время бронирования столика;
2. количество приглашенных людей;
3. имя пользователя, который выполняет резервирование.

В следующем примере кода показано, как с помощью каскада помочь пользователю выполнить ряд запросов.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses a waterfall technique to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.number(session, "How many people are in your party?");
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        builder.Prompts.text(session, "Whose name will this reservation be under?");
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;
        
        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 
```

Основные функции бота выполняются в диалоге по умолчанию. Диалог по умолчанию определяется при создании бота. 

```javascript
var bot = new builder.UniversalBot(connector, [..waterfall steps..]); 
```

Также во время процесса создания можно настроить использование [хранилища данных](bot-builder-nodejs-state.md). Например, чтобы использовать размещенное в памяти хранилище, его можно разместить следующим образом.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..]).set('storage', inMemoryStorage); // Register in-memory storage 
```

Диалог по умолчанию создается как массив функций, определяющих действия каскада. В приведенном примере существуют четыре функции, поэтому каскад состоит из четырех шагов. На каждом шаге выполняется одно задание и его результаты обрабатываются на следующем шаге. Процесс продолжается до последнего шага, на котором подтверждается бронирование столика, и диалог заканчивается.

На следующем снимке экрана показаны результаты выполнения этого бота в [Bot Framework Emulator](../bot-service-debug-emulator.md).

![Управление последовательностью общения с помощью каскада](../media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

### <a name="prompt-user-for-input"></a>Запрос пользователю на ввод данных

На каждом шаге этого примера используется запрос пользователя на ввод данных. Запрос — это особый тип диалога, который запрашивает пользовательский ввод данных, ждет ответа и возвращает ответ в каскад на следующем шаге. Чтобы получить сведения о различных типах запросов, которые можно использовать в боте, см. статью [Запрос пользователю на ввод данных](bot-builder-nodejs-dialog-prompt.md).

В этом примере бот использует `Prompts.text()` для запроса от пользователя произвольного ответа в текстовом формате. Пользователь может ответить любым текстом, и бот должен решить, как его обработать. `Prompts.time()` использует библиотеку[Chrono](https://github.com/wanasit/chrono) для анализа информации о дате и времени, полученных из строки. Данные сведения позволяют боту распознать более естественный язык указания даты и времени. Например: "June 6th, 2017 at 9pm", "Today at 7:30pm", "next monday at 6pm" и т. д.

> [!TIP] 
> Время, которое пользователь вводит, преобразуется в формат UTC и зависит от часового пояса сервера, на котором размещен бот. Необходимо учитывать часовые пояса, так как сервер может находиться в другом часовом поясе, отличном от часового пояса пользователя. Для преобразования даты и времени в локальное время попросите пользователя указать, в каком часовом поясе он находится.

## <a name="manage-a-conversation-flow-with-multiple-dialogs"></a>Управление последовательностью общения с помощью нескольких диалогов

Другим способом управления последовательностью общения является использование комбинации каскада и нескольких диалогов. Каскад позволяет объединять функции в диалоге, в то время как диалоги позволяют разбить общение на более мелкие функциональные части, которые в любое время можно использовать повторно.

Например, рассмотрим бот резервирования ужина. В следующем пример кода показан предыдущий пример, переписанный для использования каскада и нескольких диалогов.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses multiple dialogs to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

// Dialog to ask for a date and time
bot.dialog('askForDateTime', [
    function (session) {
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);

// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
])

// Dialog to ask for the reservation name.
bot.dialog('askForReserverName', [
    function (session) {
        builder.Prompts.text(session, "Who's name will this reservation be under?");
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

Результаты выполнения этого бота идентичны результатам выполнения предыдущего бота, для которого использовался только каскад. Но в самом коде существует два главных отличия.

1. Диалог по умолчанию предназначен для управления последовательностью общения.
2. Задачей каждого шага общения управляет отдельный диалог. В таком случае боту требуется три ответа, поэтому он три раза выполняет запрос к пользователю. Теперь каждый запрос находится в собственном диалоге.

С помощью этого метода можно разделить ход общения и логическую задачу. Это позволяет при необходимости повторно использовать диалоги в разных последовательностях общения. 

## <a name="respond-to-user-input"></a>Ответ на пользовательский ввод

С помощью ряда задач в процессе руководства пользователем у него могут возникнуть вопросы или он может запросить дополнительную информацию перед ответом. Как можно справиться с такими запросами? Например, независимо от того, где в общении находится пользователь, как ответит бот, если пользователь введет запрос "Help", "Support" или "Cancel"? Что делать, если пользователю потребуются дополнительные сведения? Что случится, если пользователь изменит свое решение, захочет отказаться от текущей задачи и начать совершенно другую задачу?

Пакет SDK Bot Builder для Node.js позволяет боту просматривать определенные входные данные в области текущего диалога в глобальном или локальном контекстах. Эти входные данные называются [действиями](bot-builder-nodejs-dialog-actions.md), они разрешают боту просматривать вводимые пользователем данные, основываясь на предложении `matches`. Какая реакция последует на данные, вводимые пользователем, будет определять бот.

### <a name="handle-global-action"></a>Обработка глобальных действий

Используйте `triggerAction`, если требуется, чтобы бот мог обрабатывать действия в любой момент общения. Когда вводимые данные соответствует указанному термину, триггеры позволяют боту вызывать определенный диалог. Например, если требуется поддержать глобальный параметр "Help", можно создать диалог и присоединить к нему `triggerAction`, который ожидает входные данные, соответствующие "Help".

В следующем пример кода показано, как присоединить `triggerAction` к диалогу, чтобы указать, что диалог должен вызываться при вводе слова "help" пользователем.

```javascript
// The dialog stack is cleared and this dialog is invoked when the user enters 'help'.
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
});
```

После выполнения `triggerAction` по умолчанию стек диалога очищается, а запущенный диалог становится новым диалогом по умолчанию. В этом примере при выполнении `triggerAction` стек диалога очищается и диалог `help` добавляется в стек в качестве нового диалога по умолчанию. Если это поведение не является желаемым, параметр `onSelectAction` можно добавить в `triggerAction`. Параметр `onSelectAction` позволяет боту запустить новый диалог не очищая стек общения, что позволяет временно переадресовать общение, а затем возобновить его с места, на котором он был остановлен.

В следующем примере кода показано, как использовать параметр `onSelectAction` с `triggerAction` таким образом, чтобы диалог `help` был добавлен в существующий стек диалога (и при этом он не был очищен).

```javascript
bot.dialog('help', function (session, args, next) {
    session.endDialog("This is a bot that can help you make a dinner reservation. <br/>Please say 'next' to continue");
})
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

В этом примере диалог `help` перебирает управление общением на себя при вводе пользователем слова "help". Так как `triggerAction` содержит параметр `onSelectAction`, диалог `help` отправляется в существующий стек диалога, который не очищается. По завершению диалога `help` он удаляется из стека диалогов, а общение возобновляется с момента, в котором он был прерван командой `help`.

### <a name="handle-contextual-action"></a>Выполнение контекстуального действия

Диалог `help` из предыдущего примера вызывается в том случае, если в любой момент общения пользователь вводит слово "help". Таким образом, диалог может содержать только общие справочные сведения, так как для пользовательского запроса о помощи нет специального содержимого. Что случится, если пользователь захочет запросить помощь в отношении определенной точки общения? В таком случае диалог `help` должен быть запущен с содержимым текущего диалога.

Например, рассмотрим бот резервирования ужина. Что делать, если пользователь хочет узнать максимальное количество мест за столиком, когда его просят назвать количество людей приглашенных на ужин? Для обработки этого сценария можно присоединить `beginDialogAction` к диалогу `askForPartySize`, ожидая ввода пользователем слова "help".

В следующем примере кода показано, как с помощью `beginDialogAction` присоединить контекстную справку в диалог.

```javascript
// Dialog to ask for number of people in the party
bot.dialog('askForPartySize', [
    function (session) {
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
       session.endDialogWithResult(results);
    }
])
.beginDialogAction('partySizeHelpAction', 'partySizeHelp', { matches: /^help$/i });

// Context Help dialog for party size
bot.dialog('partySizeHelp', function(session, args, next) {
    var msg = "Party size help: Our restaurant can support party sizes up to 150 members.";
    session.endDialog(msg);
})
```

В данном примере всякий раз, когда пользователь вводит слово "help", бот передает диалог `partySizeHelp` в стек. Диалог отправляет пользователю справочное сообщение и завершается, возвращая управление диалогу `askForPartySize`, который повторяет пользователю запрос о количестве мест за столом.

Важно отметить, что эта контекстная справка выполняется, только если пользователь находится в диалоге `askForPartySize`. В противном случае вместо нее будет выполнено общее справочное сообщение из `triggerAction`. Другими словами, локальное предложение `matches` всегда обладает приоритетом над глобальный предложением `matches`. Например, если `beginDialogAction` соответствует **справке**, то совпадения для **справки** не будут выполнены в `triggerAction`. Дополнительные сведения см. в разделе [Action precedence](bot-builder-nodejs-dialog-actions.md#action-precedence) (Приоритет действий).

### <a name="change-the-topic-of-conversation"></a>Изменение темы общения

Выполнение `triggerAction` по умолчанию очищает стек диалога и сбрасывает общение, которое началось с указанного диалога. Такое поведение часто предпочтительнее в момент, когда бот должен переключиться с одной темы общения на другую, например, если пользователь в середине процесса резервирования решает вместо этого заказать ужин, который будет доставлен в номер. 

Следующий пример основан на предыдущем, так что бот позволяет пользователю либо сделать резервирование, либо заказать ужин. В этом боте диалог по умолчанию является диалогом приветствия, который представляет пользователю два варианта: `Dinner Reservation` и `Order Dinner`.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot enables users to either make a dinner reservation or order dinner.
var bot = new builder.UniversalBot(connector, function(session){
    var msg = "Welcome to the reservation bot. Please say `Dinner Reservation` or `Order Dinner`";
    session.send(msg);
}).set('storage', inMemoryStorage); // Register in-memory storage 
```

Логика резервирования, которая находилась по умолчанию в диалоговом окне предыдущего примера, теперь находится в своем собственном диалоге с именем `dinnerReservation`. Последовательность общения `dinnerReservation` остается такой же, как и в нескольких версиях диалога, которые обсуждались ранее. Единственное отличие состоит в том, что к диалогу прикреплен `triggerAction`. Обратите внимание, что в этой версии, перед тем как вызывать новый диалог, `confirmPrompt` просит пользователя подтвердить, что он хочет изменить тему общения. В таких сценариях рекомендуется включить `confirmPrompt`, так как после очистки стека пользователь будет направлен в новую тему общения, тем самым отказавшись от темы общения, которое ведется.

```javascript
// This dialog helps the user make a dinner reservation.
bot.dialog('dinnerReservation', [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        session.beginDialog('askForDateTime');
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        session.beginDialog('askForPartySize');
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        session.beginDialog('askForReserverName');
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
])
.triggerAction({
    matches: /^dinner reservation$/i,
    confirmPrompt: "This will cancel your current request. Are you sure?"
});
```

Второй раздел общения определяется с использованием каскада в диалоге `orderDinner`. В диалоге отображается меню ужина, а после того, как пользователь выполнит заказ, появляется номер комнаты. `triggerAction` прикреплен к диалогу, чтобы указать, что он должен активироваться, когда пользователь вводит "order dinner", а также, чтобы убедиться, что пользователь получил запрос на подтверждение своего выбора, если у него возникнет желание изменить тему общения.

```javascript
// This dialog help the user order dinner to be delivered to their hotel room.
var dinnerMenu = {
    "Potato Salad - $5.99": {
        Description: "Potato Salad",
        Price: 5.99
    },
    "Tuna Sandwich - $6.89": {
        Description: "Tuna Sandwich",
        Price: 6.89
    },
    "Clam Chowder - $4.50":{
        Description: "Clam Chowder",
        Price: 4.50
    }
};

bot.dialog('orderDinner', [
    function(session){
        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    },
    function (session, results) {
        if (results.response) {
            var order = dinnerMenu[results.response.entity];
            var msg = `You ordered: ${order.Description} for a total of $${order.Price}.`;
            session.dialogData.order = order;
            session.send(msg);
            builder.Prompts.text(session, "What is your room number?");
        } 
    },
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endDialog(msg);
        }
    }
])
.triggerAction({
    matches: /^order dinner$/i,
    confirmPrompt: "This will cancel your order. Are you sure?"
});
```

После того как пользователи начинают общение и выбирают `Dinner Reservation` или `Order Dinner`, они могут изменить свой выбор в любое время. Например, если пользователь находится в середине процесса резервирования и вводит "order dinner", бот подтвердит данное действие сообщением "This will cancel your current request". Вы уверены? Если пользователь ввел "no", запрос отменяется и пользователь может продолжить процесс бронирования столика. Если пользователь ввел "yes", бот очистит стек общения и передаст управление общением диалогу `orderDinner`.

## <a name="end-conversation"></a>Завершение общения.

В приведенных выше примерах диалоги закрываются с помощью `session.endDialog` или `session.endDialogWithResult`, каждый из которых завершает диалог, удаляет его из стека и возвращает управление диалогу вызова. В ситуации, когда требуется указать, что данное место является концом общения для пользователя, который его достиг, следует использовать `session.endConversation`.

Метод [`session.endConversation`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#endconversation) завершает общение и при необходимости может отправить сообщение пользователю. Например, диалог из предыдущего примера `orderDinner` может завершить общение с помощью `session.endConversation`, как показано в следующем примере кода.

```javascript
bot.dialog('orderDinner', [
    //...waterfall steps...
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
]);
```

Вызов `session.endConversation` завершит общение путем очистки стека диалога и сброса хранилища [`session.conversationData`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session#conversationdata). Дополнительные сведения о хранилище данных см. в статье [Управление данными состояния](bot-builder-nodejs-state.md).

Вызов `session.endConversation` является логическим действием, которое необходимо предпринять, когда пользователь завершает общение, для которого бот был предназначен. Также можно использовать `session.endConversation` для завершения общения в ситуациях, когда пользователь во время общения вводит "cancel" или "goodbye". Чтобы это сделать, просто прикрепите `endConversationAction` к диалогу и разрешите триггеру ожидать входных данных, которые соответствуют словам "cancel" или "goodbye".

В следующем примере кода показано, как присоединить `endConversationAction` к диалогу. Это позволит завершить общение, если пользователь введет слово "cancel" или "goodbye".

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

Поскольку завершение общения с помощью `session.endConversation` или `endConversationAction` очистит стек общения и заставит пользователя начать все заново, то необходимо включить `confirmPrompt`, чтобы убедиться, что пользователь действительно хочет это сделать.

## <a name="next-steps"></a>Дополнительная информация

В этой статье были исследованы способы управления общением, которые по своему характеру являются последовательными. Что случится, если потребуется повторить диалог или использовать последовательный цикл в общении? Давайте посмотрим, как можно это сделать, заменив диалоги в стеке.

> [!div class="nextstepaction"]
> [Replace dialogs](bot-builder-nodejs-dialog-replace.md) (Замена диалогов)


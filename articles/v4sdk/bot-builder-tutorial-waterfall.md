---
title: Задайте вопросы пользователю | Документация Майкрософт
description: Узнайте, как использовать каскадную модель, чтобы спросить пользователя о нескольких наборах входных данных в пакете SDK для Bot Builder.
keywords: каскады, диалоговые окна, задать вопрос, запросы
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 5/10/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 62ff445e4aabf2afd41cc4bf1f15badb3f47e945
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39306230"
---
# <a name="ask-the-user-questions"></a>Задайте вопросы пользователю

По своей сути, бот построен вокруг общения с пользователем. Общение может принимать [разные формы](bot-builder-conversations.md); оно может быть коротким или более сложным, задавать или отвечать на вопросы. Форма общения зависит от нескольких факторов, но все эти факторы завязаны на общении.

В этом руководстве вы узнаете, как создать общение, задав простой вопрос боту с множеством включений. Наш пример основан на резервирование столика, но вы можете представить себе бот, который делает множество вещей в общении с множеством включений, например, размещает заказ, отвечает на часто задаваемые вопросы, делает резервирование и т. д.

Бот интерактивного чата может отвечать на ввод пользователя или запрашивать у пользователя определенные вводные данные. В этом руководстве показано, как задать пользователю вопрос с помощью библиотеки `Prompts`, которая входит в состав `Dialogs`. [Диалоговые окна](../bot-service-design-conversation-flow.md) можно рассматривать как контейнеры, которые определяют структуру общения, а запросы в диалоговых окнах более подробно рассматриваются в [практической статье](bot-builder-prompts.md).

## <a name="prerequisite"></a>Предварительные требования

Код в этом руководстве опирается на **базовый бот**, который был создан в разделе [Начало работы](~/bot-service-quickstart.md).

## <a name="get-the-package"></a>Получите пакет

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Установите пакет **Microsoft.Bot.Builder.Dialogs** из диспетчера пакетов Nuget.

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)
Перейдите в папку проекта вашего бота и установите пакет `botbuilder-dialogs` из NPM:

```cmd
npm install --save botbuilder-dialogs@preview
```

---

## <a name="import-package-to-bot"></a>Импортируйте пакет в бот

# <a name="ctabcstab"></a>[C#](#tab/cstab)

В коде бота, добавьте следующую ссылку в диалоговые окна и запросы.

```cs
// ...
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Prompts;
// ...
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Откройте файл **app.js** и включить библиотеку `botbuilder-dialogs` в код бота.

```javascript
const botbuilder_dialogs = require('botbuilder-dialogs');
```

---

Это позволит получить доступ к библиотеке `DialogSet` и `Prompts`, которую вы будете использовать, чтобы задавать вопросы пользователю. `DialogSet` — это набор диалоговых окон, который структурируется в **каскадном** шаблоне. Каскадный шаблон подразумевает, что одно диалоговое окно следует за другим.

## <a name="instantiate-a-dialogs-object"></a>Создадите экземпляр объекта диалоговых окон

Создадите экземпляр объекта `dialogs`. Используете объект диалогового окна для управления вопросами и ответами.

# <a name="ctabcstab"></a>[C#](#tab/cstab)
Объявите переменную-член в классе бота и инициализируйте ее в конструктор для вашего бота. 
```cs
public class MyBot : IBot
{
    private readonly DialogSet dialogs;
    public MyBot()
    {
        dialogs = new DialogSet();
    }
    // The rest of the class definition is omitted here
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
const dialogs = new botbuilder_dialogs.DialogSet();
```
---

## <a name="define-a-waterfall-dialog"></a>Определите каскадное диалоговое окно

Чтобы задать вопрос, вам потребуется по крайней мере, двух шаговое **каскадное** диалоговое окно. В этом примере будет создано двух шаговое **каскадное** диалоговое окно, где, на первом шаге, запрашивается имя пользователя, а на втором — приветствуется пользователя по имени. 

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Измените конструктор бота, чтобы добавить диалоговое окно:
```csharp
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
}
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
// Greet user:
// Ask for the user name and then greet them by name.
dialogs.add('greetings',[
    async function (dc){
        await dc.prompt('textPrompt', 'What is your name?');
    },
    async function(dc, results){
        var userName = results;
        await dc.context.sendActivity(`Hello ${userName}!`);
        await dc.end(); // Ends the dialog
    }
]);
```

---

Вопрос задается с помощью метода `textPrompt`, который поставляется библиотекой `Prompts`. Библиотека `Prompts` предлагает набор запросов, с помощью которых можно получить от пользователей различную информацию. Дополнительные сведения о других типах запросов см. в разделе [Запрос пользователю на ввод данных](~/v4sdk/bot-builder-prompts.md).

Для запроса к работе нужно добавить запрос к объекту `dialogs` с помощью идентификатора диалога `textPrompt`, и создать его с помощью конструктора `TextPrompt()`.

# <a name="ctabcstab"></a>[C#](#tab/cstab)

```cs
public MyBot()
{
    dialogs = new DialogSet();
    dialogs.Add("greetings", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Prompt("textPrompt","What is your name?");
        },
        async(dc, args, next) =>
        {
            await dc.Context.SendActivity($"Hi {args["Text"]}!");
            await dc.End();
        }
    });
    // add the prompt, of type TextPrompt
    dialogs.Add("textPrompt", new Builder.Dialogs.TextPrompt());
}

```
Как только пользователь отвечает на вопрос, ответ можно найти в параметре `args` на шаге 2.

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

```javascript
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
```
Как только пользователь отвечает на вопрос, ответ можно найти в параметре `results` на шаге 2. В этом случае `results` присваивается локальной переменной `userName`. В следующем руководстве мы покажем вам, как перенести вводные данные пользователя в хранилище.

---


Теперь, когда вы определили свои `dialogs`, чтобы задать вопрос, нужно вызвать диалоговое окно для запроса.

## <a name="start-the-dialog"></a>Запуск диалогового окна

# <a name="ctabcstab"></a>[C#](#tab/cstab)

Логика вашего бота должна выглядеть примерно так:

```cs

public async Task OnTurn(ITurnContext context)
{
    // We'll cover state later, in the next tutorial
    var state = ConversationState<Dictionary<string, object>>.Get(context);
    var dc = dialogs.CreateContext(context, state);
    if (context.Activity.Type == ActivityTypes.Message)
    {
        await dc.Continue();
        
        if(!context.Responded)
        {
            await dc.Begin("greetings");
        }
    }
}
```

Логика бота реализуется в методе `OnTurn()`. Как только пользователь говорит "Привет", бот запускает диалоговое окно `greetings`. На первом шаге диалоговое окно `greetings` предлагает пользователю ввести свое имя. Пользователь отправляет ответ с именем, в качестве сообщения действия, которое отправляется на второй шаг каскада с помощью метода `dc.Continue()`. Второй шаг каскада, приветствует пользователя по имени и закрывает диалоговое окно. 

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Метод `processActivity()` **Базового** бота должен выглядеть следующим образом.

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi'.");
            }
        }
    });
});
```

Логика бота реализуется в методе `processActivity()`. Как только пользователь говорит "Привет", бот запускает диалоговое окно `greetings`. На первом шаге диалоговое окно `greetings` предлагает пользователю ввести свое имя. Пользователь отправляет ответ с именем, в качестве сообщения действия `text`. Поскольку сообщение не соответствовало ожидаемым намерениям, и бот не отправил ответ пользователю, результат передается на второй шаг каскада, с помощью метода `dc.continue()`. Второй шаг каскада, приветствует пользователя по имени и закрывает диалоговое окно. Если на втором шаге пользователь не получил приветствие, то метод `processActivity` заканчивает диалог, отправляя пользователю, *сообщение по умолчанию*.

---



## <a name="define-a-more-complex-waterfall-dialog"></a>Определите более сложные каскадные диалоговые окна

Теперь, когда вы знаете как работает каскадное диалоговое окно и как его построить, давайте ознакомимся с более сложным диалоговым окном, предназначенным для резервирования таблицы.

Чтобы управлять запросом резервирования таблицы, необходимо определить **каскадное** диалоговое окно с четырьмя шагами. В этом общении вы также будете использовать `DatetimePrompt` и `NumberPrompt` в дополнение к `TextPrompt`.



# <a name="ctabcstab"></a>[C#](#tab/cstab)

Начните с шаблона бота Echo и переименуйте бот как "CafeBot". Добавьте `DialogSet` и некоторые статические переменные-члены.

```cs

namespace CafeBot
{
    public class CafeBot : IBot
    {
        private readonly DialogSet dialogs;

        // Usually, we would save the dialog answers to our state object, which will be covered in a later tutorial.
        // For purpose of this example, let's use the three static variables to store our reservation information.
        static DateTime reservationDate;
        static int partySize;
        static string reservationName;

        // the rest of the class definition is omitted here
        // but is discussed in the rest of this article
    }
}
```

Затем определите диалоговое окно `reserveTable`. Вы можете добавить диалоговое окно в конструкторе классов бота.
```cs
public CafeBot()
{
    dialogs = new DialogSet();

    // Define our dialog
    dialogs.Add("reserveTable", new WaterfallStep[]
    {
        async (dc, args, next) =>
        {
            // Prompt for the guest's name.
            await dc.Context.SendActivity("Welcome to the reservation service.");

            await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
        },
        async(dc, args, next) =>
        {
            var dateTimeResult = ((DateTimeResult)args).Resolution.First();

            reservationDate = Convert.ToDateTime(dateTimeResult.Value);
            
            // Ask for next info
            await dc.Prompt("partySizePrompt", "How many people are in your party?");

        },
        async(dc, args, next) =>
        {
            partySize = (int)args["Value"];

            // Ask for next info
            await dc.Prompt("textPrompt", "Whose name will this be under?");
        },
        async(dc, args, next) =>
        {
            reservationName = args["Text"];
            string msg = "Reservation confirmed. Reservation details - " +
            $"\nDate/Time: {reservationDate.ToString()} " +
            $"\nParty size: {partySize.ToString()} " +
            $"\nReservation name: {reservationName}";
            await dc.Context.SendActivity(msg);
            await dc.End();
        }
    });

     // Add a prompt for the reservation date
     dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
     // Add a prompt for the party size
     dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
     // Add a prompt for the user's name
     dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
}
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Диалоговое окно `reserveTable` должно следующим образом.

```javascript
// Reserve a table:
// Help the user to reserve a table
var reservationInfo = {};

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Whose name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;
        
        // Reservation confirmation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

В потоке общения диалогового окна `reserveTable`, пользователю задается по одному вопросу на каждом из трех шагов каскада. На четвертом шаге обрабатывается ответ на последний вопрос, и отправляется пользователю подтверждение резервирования.



# <a name="ctabcstab"></a>[C#](#tab/cstab)
Каждый шаг каскада в диалоговом окне `reserveTable` запрашивает информацию у пользователя. Следующий код используется для добавления запросов к набору диалогового окна.

```cs
dialogs.Add("dateTimePrompt", new Microsoft.Bot.Builder.Dialogs.DateTimePrompt(Culture.English));
dialogs.Add("partySizePrompt", new Microsoft.Bot.Builder.Dialogs.NumberPrompt<int>(Culture.English));
dialogs.Add("textPrompt", new Microsoft.Bot.Builder.Dialogs.TextPrompt());
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Для работы этого каскада, также необходимо добавить запросы к объекту `dialogs`:

```javascript
// Define prompts
// Generic prompts
dialogs.add('textPrompt', new botbuilder_dialogs.TextPrompt());
dialogs.add('dateTimePrompt', new botbuilder_dialogs.DatetimePrompt());
dialogs.add('partySizePrompt', new botbuilder_dialogs.NumberPrompt());
```

---

Теперь объект готов для подключения к логике бота.

## <a name="start-the-dialog"></a>Запуск диалогового окна

# <a name="ctabcstab"></a>[C#](#tab/cstab)
`OnTurn` вашего бота должен содержать следующий код:
```cs
public async Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // The type parameter PropertyBag inherits from 
        // Dictionary<string,object>
        var state = ConversationState<Dictionary<string, object>>.Get(context);
        var dc = dialogs.CreateContext(context, state);
        await dc.Continue();

        // Additional logic can be added to enter each dialog depending on the message received
        
        if(!context.Responded)
        {
            if (context.Activity.Text.ToLowerInvariant().Contains("reserve table"))
            {
                await dc.Begin("reserveTable");
            }
            else
            {
                await context.SendActivity($"You said '{context.Activity.Text}'");
            }
        }
    }
}
```


В файле **Startup.cs** измените инициализацию ПО промежуточного слоя ConversationState, чтобы использовать класс, полученный из `Dictionary<string,object>`, вместо `EchoState`.

Например, для `Configure()`:
```cs
options.Middleware.Add(new ConversationState<Dictionary<string, object>>(dataStore));
```


# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

Чтобы записать намерения пользователя для запроса, добавьте несколько строк кода в метод `processActivity()`. Метод `processActivity()`, используемый ботом, должен выглядеть следующим образом.

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);
        const dc = dialogs.createContext(context, convo);

        if (isMessage) {
            // Check for valid intents
            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greetings');
            }
            else if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
        }

        if(!context.responded){
            // Continue executing the "current" dialog, if any.
            await dc.continue();

            if(!context.responded && isMessage){
                // Default message
                await context.sendActivity("Hi! I'm a simple bot. Please say 'Hi' or 'Reserve table'.");
            }
        }
    });
});
```

Каждый раз, когда пользователь отправляет сообщение, содержащее строку `reserve table`, бот запускает общение `reserveTable`.

---



## <a name="next-steps"></a>Дополнительная информация

В этом руководстве бот сохраняет вводные данные пользователя в виде переменных внутри бота. Если вы хотите сохранить эти данные, вам необходимо добавить состояние к ПО промежуточного слоя. В следующем руководстве мы более подробно рассмотрим, как сохранить данные состояния пользователя. 

> [!div class="nextstepaction"]
> [Хранение данных пользователя](bot-builder-tutorial-persist-user-inputs.md)

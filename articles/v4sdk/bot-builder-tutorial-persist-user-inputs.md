---
title: Хранение данных пользователя | Документация Майкрософт
description: Узнайте, как сохранить данные состояния пользователя в хранилище с помощью пакета SDK Bot Framework.
keywords: хранение данных пользователя, хранилище, данные общения
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bb1e909de69a7690e844701e182dddcebf91cc87
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904927"
---
# <a name="persist-user-data"></a>Хранение данных пользователя

[!INCLUDE[applies-to](../includes/applies-to.md)]

Когда бот попросит пользователя ввести данные, может возникнуть необходимость сохранить часть информации в некой форме в хранилище. Пакет SDK Bot Framework позволяет хранить данные, вводимые пользователем, с помощью *размещенного в памяти хранилища* или хранилища баз данных, например *Cosmos DB*. Хранилища локального типа преимущественно используются на этапе тестирования или создания прототипа бота. А постоянные хранилища, такие как хранилище базы данных, лучше всего подходят для ботов в рабочей среде.

В этой статье показано, как определить объект хранилища и сохранить в нем данные, вводимые пользователем. Используя диалог, мы попросим пользователя ввести имя, если вы еще не сделали это. Независимо от предпочитаемого типа хранилища, процессы подключения и сохранения данных одинаковы для всех типов. В коде, приведенном в этой статье, для хранения данных используется хранилище `CosmosDB`.

## <a name="prerequisites"></a>Предварительные требования

В зависимости от среды разработки, которую вы будете использовать, требуются определенные ресурсы.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

* [Установите Visual Studio 2015 или более поздней версии](https://www.visualstudio.com/downloads/).
* [Установите шаблон Bot Builder версии 4](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

* [Установите Visual Studio Code](https://www.visualstudio.com/downloads/).
* [Установите Node.js 8.5 или более поздней версии](https://nodejs.org/en/).
* [Установите Yeoman](http://yeoman.io/).
* Установите генератор шаблонов Node.js версии 4 для Bot Builder.

    ```shell
    npm install generator-botbuilder
    ```

---

Для работы с этим руководством используются указанные ниже пакеты.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Мы начнем с базового шаблона EchoBot. Инструкции см. в [кратком руководстве по .NET](~/dotnet/bot-builder-dotnet-quickstart.md).

Эти дополнительные пакеты можно установить из диспетчера пакетов NuGet.

* **Microsoft.Bot.Builder.Azure**;
* **Microsoft.Bot.Builder.Dialogs**;

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Мы начнем с базового шаблона EchoBot. Инструкции вы найдете в [кратком руководстве по JavaScript](~/javascript/bot-builder-javascript-quickstart.md).

Установите эти дополнительные пакеты npm.

```cmd
npm install --save botbuilder-dialogs
npm install --save botbuilder-azure
```

---

Чтобы протестировать бот, который вы создадите в рамках этого руководства, необходимо установить [Bot Framework Channel Emulator](https://github.com/Microsoft/BotFramework-Emulator).

## <a name="create-a-cosmosdb-service-and-update-your-application-settings"></a>Создание службы Cosmos DB и обновление параметров приложения

Чтобы настроить службу и базу данных Cosmos DB, следуйте инструкциям по [использованию Cosmos DB](bot-builder-howto-v4-storage.md#using-cosmos-db). Инструкции кратко перечислены ниже:

1. В новом окне браузера войдите на <a href="http://portal.azure.com/" target="_blank">портал Azure</a>.
1. Последовательно выберите **Создать ресурс > Базы данных > Azure Cosmos DB**.
1. На странице **Новая учетная запись** укажите уникальное имя в поле **Идентификатор**. В поле **API** выберите **SQL** и укажите нужные сведения в полях **Подписка**, **Расположение** и **Группа ресурсов**.
1. Нажмите кнопку **Создать**.

Затем добавьте в эту службу коллекцию, чтобы использовать ее с ботом.

Запишите идентификаторы базы данных и коллекции, которые вы использовали для добавления коллекции, а также URI и первичный ключ из параметров ключей коллекции. Они потребуются для подключения бота к службе.

### <a name="update-your-application-settings"></a>Обновление параметров приложения

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Обновите файл **appsettings.json**, добавив в него сведения о подключении для Cosmos DB.

```csharp
{
  // Settings for CosmosDB.
  "CosmosDB": {
    "DatabaseID": "<your-database-identifier>",
    "CollectionID": "<your-collection-identifier>",
    "EndpointUri": "<your-CosmosDB-endpoint>",
    "AuthenticationKey": "<your-primary-key>"
  }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В папке проекта найдите файл с расширением **.env** и добавьте в него указанные ниже записи с данными Cosmos DB.

Файл с расширением **.env**

```text
DB_SERVICE_ENDPOINT=<your-CosmosDB-endpoint>
AUTH_KEY=<authentication key>
DATABASE=<your-primary-key>
COLLECTION=<your-collection-identifier>
```

Затем в основном файле бота **index.js** замените значения для `storage`, чтобы использовать `CosmosDbStorage` вместо `MemoryStorage`. Во время выполнения будут получены переменные среды, которые заполнят эти поля.

```javascript
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT,
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
});
```

---

## <a name="create-storage-state-manager-and-state-property-accessor-objects"></a>Создание объектов хранилища, диспетчера состояний и метода доступа к свойству состояния

Для администрирования и хранения данных боты используют объекты хранилища и управления состоянием. Диспетчер обеспечивает абстракцию, позволяющую вам получить доступ к свойствам состояния с помощью соответствующих методов доступа независимо от типа базового хранилища. Записывайте данные в хранилище с использованием диспетчера состояний.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

### <a name="define-a-class-for-your-user-data"></a>Определение класса для пользовательских данных

Переименуйте файл **CounterState.cs** в **UserData.cs**, а класс **CounterState** в **UserData**.

Обновите этот класс для хранения собираемых данных.

```csharp
/// <summary>
/// Class for storing persistent user data.
/// </summary>
public class UserData
{
    public string Name { get; set; }
}
```

### <a name="define-a-class-for-your-state-and-state-property-accessor-objects"></a>Определение класса для объектов состояния и метода доступа к свойству состояния

Переименуйте файл **EchoBotAccessors.cs** в **BotAccessors.cs**, а класс **EchoBotAccessors** в **BotAccessors**.

Обновите этот класс для хранения объектов состояния и методов доступа к свойству состояния.

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using System;

public class BotAccessors
{
    public UserState UserState { get; }

    public ConversationState ConversationState { get; }

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }

    public IStatePropertyAccessor<UserData> UserDataAccessor { get; set; }

    public BotAccessors(UserState userState, ConversationState conversationState)
    {
        this.UserState = userState
            ?? throw new ArgumentNullException(nameof(userState));

        this.ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }
}
```

### <a name="update-the-startup-code-for-your-bot"></a>Обновление кода запуска для бота

В файле **Startup.cs** обновите инструкции using.

```csharp
using System;
using System.Linq;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
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

В методе `ConfigureServices` обновите вызов AddBot. Для этого сначала создайте объект резервного хранилища, а затем зарегистрируйте объект BotAccessors.

Нужно указать свойство состояния диалога для объекта `DialogState`, чтобы отслеживать состояние диалога. Мы зарегистрируем отдельные базы данных для метода доступа к свойству состояния диалога и набора диалогов, который будет использоваться ботом. Для состояния пользователя бот создаст собственный метод доступа к свойству состояния.

Применение метода доступа `BotAccessors` — это эффективный способ управления хранилищем для нескольких объектов состояния бота.

```cs
public void ConfigureServices(IServiceCollection services)
{
    // Register your bot.
    services.AddBot<UserDataBot>(options =>
    {
        // ...

        // Use persistent storage and create state management objects.
        var cosmosSettings = Configuration.GetSection("CosmosDB");
        IStorage storage = new CosmosDbStorage(
            new CosmosDbStorageOptions
            {
                DatabaseId = cosmosSettings["DatabaseID"],
                CollectionId = cosmosSettings["CollectionID"],
                CosmosDBEndpoint = new Uri(cosmosSettings["EndpointUri"]),
                AuthKey = cosmosSettings["AuthenticationKey"],
            });
        options.State.Add(new ConversationState(storage));
        options.State.Add(new UserState(storage));
    });

    // Register the bot's state and state property accessor objects.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var userState = options.State.OfType<UserState>().FirstOrDefault();
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();

        return new BotAccessors(userState, conversationState)
        {
            UserDataAccessor = userState.CreateProperty<UserData>("UserDataBot.UserData"),
            DialogStateAccessor = conversationState.CreateProperty<DialogState>("UserDataBot.DialogState"),
        };
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

### <a name="update-your-server-code"></a>Обновление кода сервера

В файле **index.js** вашего проекта обновите указанные ниже инструкции require.

```javascript
// Import required bot services.
const { BotFrameworkAdapter, ConversationState, UserState } = require('botbuilder');
const { CosmosDbStorage } = require('botbuilder-azure');
```

В рамках этого руководства для хранения данных мы будем использовать `UserState`. Нам нужно создать объект `userState` и обновить эту строку кода, чтобы передать второй параметр в класс `MainDialog`.

```javascript
// Create conversation state with in-memory storage provider.
const conversationState = new ConversationState(storage);
const userState = new UserState(storage);

// Create the main dialog.
const bot = new MyBot(conversationState, userState);
```

Если возникнет общая ошибка, очистите состояние беседы и пользователя.

```javascript
// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${error}`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.load(context);
    await conversationState.clear(context);
    await userState.load(context);
    await userState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
    await userState.saveChanges(context);
};
```

Также обновите цикл сервера HTTP для вызова объекта бота.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="update-your-bot-logic"></a>Обновление логики бота

В классе `MyBot` требуется указать библиотеки, необходимые для работы вашего бота. В рамках этого руководства мы будем использовать библиотеку **Dialogs**.

```javascript
// Required packages for this bot
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, TextPrompt, NumberPrompt } = require('botbuilder-dialogs');

```

Обновите конструктор класса `MyBot`, чтобы он принял `userState` в качестве второго параметра. Также обновите конструктор, определив состояния, диалоги и запросы, необходимые для работы с этим руководством. В этом случае мы определяем последовательность из двух шагов, где на _шаге 1_ запрашивается имя пользователя, а на _шаге 2_ возвращается ответ пользователя. Способ хранения этой информации зависит от основной логики бота.

```javascript
constructor(conversationState, userState) {

    // creates a new state accessor property.
    this.conversationState = conversationState;
    this.userState = userState;

    this.dialogState = this.conversationState.createProperty('dialogState');
    this.userDataAccessor = this.userState.createProperty('userData');

    this.dialogs = new DialogSet(this.dialogState);

    // Add prompts
    this.dialogs.add(new TextPrompt('textPrompt'));

    // Add a waterfall dialog to collect and return the user's name.
    this.dialogs.add(new WaterfallDialog('greetings', [
        async function (step) {
            return await step.prompt('textPrompt', "What is your name?");
        },
        async function (step) {
            return await step.endDialog(step.result);
        }
    ]));
}
```

---

При сохранении пользовательских данных вам будет предложено несколько вариантов. В пакете SDK на выбор предоставляется несколько объектов состояния с различными областями. В этом примере мы используем conversationState, чтобы управлять объектом состояния диалога, а userData применим для управления данными пользователя.

Общие сведения о хранилище и состоянии см. в статьях [Запись данных напрямую в хранилище](bot-builder-howto-v4-storage.md) и [Управление состоянием диалога и пользователя](bot-builder-howto-v4-state.md).

## <a name="create-a-greeting-dialog"></a>Создание приветственного диалога

Чтобы получить имя пользователя, мы будем использовать диалог. Чтобы не усложнять сценарий, диалог будет возвращать имя пользователя и бот будет управлять объектом данных пользователя и состоянием.

Создайте класс **GreetingsDialog** и вставьте приведенный ниже код.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;

/// <summary>Defines a dialog for collecting a user's name.</summary>
public class GreetingsDialog : DialogSet
{
    /// <summary>The ID of the main dialog.</summary>
    public const string MainDialog = "main";

    /// <summary>The ID of the text prompt to use in the dialog.</summary>
    private const string TextPrompt = "textPrompt";

    /// <summary>Creates a new instance of this dialog set.</summary>
    /// <param name="dialogState">The dialog state property accessor to use for dialog state.</param>
    public GreetingsDialog(IStatePropertyAccessor<DialogState> dialogState)
        : base(dialogState)
    {
        // Add the text prompt to the dialog set.
        Add(new TextPrompt(TextPrompt));

        // Define the main dialog and add it to the set.
        Add(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            async (stepContext, cancellationToken) =>
            {
                // Ask the user for their name.
                return await stepContext.PromptAsync(TextPrompt, new PromptOptions
                {
                    Prompt = MessageFactory.Text("What is your name?"),
                }, cancellationToken);
            },
            async (stepContext, cancellationToken) =>
            {
                // Assume that they entered their name, and return the value.
                return await stepContext.EndDialogAsync(stepContext.Result, cancellationToken);
            },
        }));
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

См. раздел выше, в котором приводится пример создания диалога в конструкторе `MainDialog`.

---

Дополнительные сведения о диалогах см. в статьях [Создание запросов на ввод данных пользователем с помощью библиотеки диалогов](bot-builder-prompts.md) и [Управление простым процессом общения с помощью диалогов](bot-builder-dialog-manage-conversation-flow.md).

## <a name="update-your-bot-to-use-the-dialog-and-user-state"></a>Обновление бота для использования данных о состоянии диалога и пользователя

Мы рассмотрим структуру бота и управление вводимых пользователем данных по отдельности.

### <a name="add-the-dialog-and-a-user-state-accessor"></a>Добавление экземпляра диалога и метода доступа к данным о состоянии пользователя

Нам нужно отслеживать экземпляр диалога и метод доступа к свойству состояния для данных пользователя.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Добавьте код для инициализации бота.

```cs
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Schema;

/// <summary>Defines the bot for the persisting user data tutorial.</summary>
public class UserDataBot : IBot
{
    /// <summary>The bot's state and state property accessor objects.</summary>
    private BotAccessors Accessors { get; }

    /// <summary>The dialog set that has the dialog to use.</summary>
    private GreetingsDialog GreetingsDialog { get; }

    /// <summary>Create a new instance of the bot.</summary>
    /// <param name="options">The options to use for our app.</param>
    /// <param name="greetingsDialog">An instance of the dialog set.</param>
    public UserDataBot(BotAccessors botAccessors)
    {
        // Retrieve the bot's state and accessors.
        Accessors = botAccessors;

        // Create the greetings dialog.
        GreetingsDialog = new GreetingsDialog(Accessors.DialogStateAccessor);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

См. раздел выше, в котором объясняется, как определить эти методы доступа к свойству состояния в конструкторе `MainDialog`.

---

### <a name="update-the-turn-handler"></a>Обновление обработчика реплик

Обработчик реплик будет отправлять приветственное сообщение пользователю, когда он впервые присоединяется к диалогу, и реагировать на сообщения, которые пользователь отправляет боту. Если в какой-либо точке будет обнаружено, что боту неизвестно имя пользователя, бот начнет диалог приветствия, чтобы запросить имя пользователя. По завершении диалога мы сохраним имя в данных о состоянии пользователя с помощью объекта состояния, созданного методом доступа к свойству состояния. Когда реплика будет завершена, метод доступа и диспетчер состояний запишут изменения в объекте в хранилище.

Также мы добавим поддержку для _удаления пользовательских данных_.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Обновите метод `OnTurnAsync` бота.

```cs
/// <summary>Handles incoming activities to the bot.</summary>
/// <param name="turnContext">The context object for the current turn.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve user data from state.
    UserData userData = await Accessors.UserDataAccessor.GetAsync(turnContext, () => new UserData());

    // Establish context for our dialog from the turn context.
    DialogContext dc = await GreetingsDialog.CreateContextAsync(turnContext);

    // Handle conversation update, message, and delete user data activities.
    switch (turnContext.Activity.Type)
    {
        case ActivityTypes.ConversationUpdate:

            // Greet any user that is added to the conversation.
            IConversationUpdateActivity activity = turnContext.Activity.AsConversationUpdateActivity();
            if (activity.MembersAdded.Any(member => member.Id != activity.Recipient.Id))
            {
                if (userData.Name is null)
                {
                    // If we don't already have their name, start a dialog to collect it.
                    await turnContext.SendActivityAsync("Welcome to the User Data bot.");
                    await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
                }
                else
                {
                    // Otherwise, greet them by name.
                    await turnContext.SendActivityAsync($"Hi {userData.Name}! Welcome back to the User Data bot.");
                }
            }

            break;

        case ActivityTypes.Message:

            // If there's a dialog running, continue it.
            if (dc.ActiveDialog != null)
            {
                var dialogTurnResult = await dc.ContinueDialogAsync();
                if (dialogTurnResult.Status == DialogTurnStatus.Complete
                    && dialogTurnResult.Result is string name
                    && !string.IsNullOrWhiteSpace(name))
                {
                    // If it completes successfully and returns a valid name, save the name and greet the user.
                    userData.Name = name;
                    await turnContext.SendActivityAsync($"Pleased to meet you {userData.Name}.");
                }
            }
            else if (userData.Name is null)
            {
                // Else, if we don't have the user's name yet, ask for it.
                await dc.BeginDialogAsync(GreetingsDialog.MainDialog);
            }
            else
            {
                // Else, echo the user's message text.
                await turnContext.SendActivityAsync($"{userData.Name} said, '{turnContext.Activity.Text}'.");
            }

            break;

        case ActivityTypes.DeleteUserData:

            // Delete the user's data.
            userData.Name = null;
            await turnContext.SendActivityAsync("I have deleted your user data.");

            break;
    }

    // Update the user data in the turn's state cache.
    await Accessors.UserDataAccessor.SetAsync(turnContext, userData, cancellationToken);

    // Persist any changes to storage.
    await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Обновите дескриптор `onTurn` бота.

```javascript
async onTurn(turnContext) {
    const dc = await this.dialogs.createContext(turnContext); // Create dialog context
    const userData = await this.userDataAccessor.get(turnContext, {});

    switch (turnContext.activity.type) {
        case ActivityTypes.ConversationUpdate:
            if (turnContext.activity.membersAdded[0].name !== 'Bot') {
                if (userData.name) {
                    await turnContext.sendActivity(`Hi ${userData.name}! Welcome back to the User Data bot.`);
                }
                else {
                    // send a "this is what the bot does" message
                    await turnContext.sendActivity('Welcome to the User Data bot.');
                    await dc.beginDialog('greetings');
                }
            }
            break;
        case ActivityTypes.Message:
            // If there is an active dialog running, continue it
            if (dc.activeDialog) {
                var turnResult = await dc.continueDialog();
                if (turnResult.status == "complete" && turnResult.result) {
                    // If it completes successfully and returns a value, save the name and greet the user.
                    userData.name = turnResult.result;
                    await this.userDataAccessor.set(turnContext, userData);
                    await turnContext.sendActivity(`Pleased to meet you ${userData.name}.`);
                }
            }
            // Else, if we don't have the user's name yet, ask for it.
            else if (!userData.name) {
                await dc.beginDialog('greetings');
            }
            // Else, echo the user's message text.
            else {
                await turnContext.sendActivity(`${userData.name} said, ${turnContext.activity.text}.`);
            }
            break;
        case ActivityTypes.DeleteUserData:
            // Delete the user's data.
            // Note: You can use the Emulator to send this activity.
            userData.name = null;
            await this.userDataAccessor.set(turnContext, userData);
            await turnContext.sendActivity("I have deleted your user data.");
            break;
    }

    // Save changes to the conversation and user states.
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
}
```

---

## <a name="start-your-bot-in-visual-studio"></a>Запуск бота в Visual Studio

Скомпилируйте и запустите приложение.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота

После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Open Bot** (Открыть бот) на вкладке приветствия в эмуляторе.
2. Выберите BOT-файл, расположенный в каталоге с созданным решением Visual Studio.

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом

Отправьте сообщение боту и получите от него сообщение в ответ.
![Работающий эмулятор](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Управление состоянием диалога и пользователя](bot-builder-howto-v4-state.md)

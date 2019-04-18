---
title: Принципы работы бота | Документация Майкрософт
description: Описание работы с действиями и HTTP в пакете SDK Bot Framework.
keywords: conversation flow, turn, bot conversation, dialogs, prompts, waterfalls, dialog set
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 1/10/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a7f6c22f35719eacf66598e79df5fe52ff19dd43
ms.sourcegitcommit: 103aa3316f9ff658cf2b0d341c5e76c3efc581ee
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/12/2019
ms.locfileid: "59540368"
---
# <a name="how-bots-work"></a>Принципы работы бота

[!INCLUDE[applies-to](../includes/applies-to.md)]

Бот — это приложение, с которым пользователи взаимодействуют, общаясь с помощью текста, графики (карт или изображений) или речи. Каждое взаимодействие между пользователем и ботом создает некоторое *действие*. Служба Bot Framework, которая входит в состав службы Azure Bot, передает информацию между пользовательскими приложениями (например, Facebook, Skype, Slack и другими *каналами*) и ботом. Каждый канал может передавать в отправляемых событиях дополнительные сведения. Прежде чем создавать ботов, важно хорошо разобраться в использовании объектов действия для общения с пользователями бота. Для начала мы рассмотрим действия, которые передаются при выполнении простого бота проверки связи. 

![Схема действий](media/bot-builder-activity.png)

Здесь представлены действия двух типов: *обновление диалога* и *сообщение*.

Служба Bot Framework Service может отправлять события обновления беседы при добавлении к общению новых участников. Например, при начале беседы с эмулятором Bot Framework вы увидите два действия обновления диалога (один для подключаемого пользователя и второй для подключаемого бота). Чтобы различить эти действия обновления диалога, проверьте наличие других участников, кроме самого бота, в свойстве *members added* (добавленные участники). 

Действие сообщения передает между сторонами информацию общения. В нашем примере бота проверки связи события сообщений передают простой текст, который будет отображаться в выбранном канале. Кроме того, действие сообщения может содержать голосовые сообщения, предлагаемые действия или карты для отображения.

В нашем примере создается бот и отправляется действие сообщения в ответ на полученное входящее действие сообщения. Но для реальной работы бот может использовать другие методы реагирования на полученные действия сообщения. Кроме того, часто боты отправляют действия сообщения с приветственным текстом в ответ на действие обновления диалога. См. дополнительные сведения о том, [как приветствовать пользователя](bot-builder-welcome-user.md).

### <a name="http-details"></a>Сведения об HTTP

Действия поступают в бот из службы Bot Framework через запросы HTTP POST. В ответ на входящий запрос POST бот возвращает код состояния HTTP 200. Действия, отправляемые из бота в канал, отправляются в отдельном запросе POST к службе Bot Framework. В ответ на него также поступает код состояния HTTP 200.

Протокол не определяет порядок отправки запросов POST подтверждений на них. Но на распространенных платформах служб HTTP такие запросы обычно являются вложенными, то есть исходящий HTTP-запрос выполняется ботом в процессе обработки входящего HTTP-запроса. Этот процесс представлен на схеме выше. Поскольку здесь есть два раздельных HTTP-подключения с откликами на них, модель безопасности должна учитывать оба из них.

### <a name="defining-a-turn"></a>Определение включения

При общении люди обычно говорят по очереди. Как правило, бот реагирует на вводимые пользователем данные. В пакете SDK Bot Framework _ход_ состоит из входящего действия пользователя и всех действий, которые бот немедленно возвращает пользователю в ответ. Ход можно рассматривать как обработку, связанную с поступлением того или иного действия.

Объект *контекста шага* предоставляет сведения о действии. К таким объектам относятся, например, идентификаторы канала, отправителя и получателя, а также другие необходимые данные для обработки действия. Кроме того, можно добавлять в объект контекста шага сведения при обработке шага на разных уровнях бота.

Контекст шага является одной из важнейших абстракций в пакете SDK. Он предоставляет всем компонентам ПО промежуточного слоя и логике приложения не только само входящее действие, но и механизм для отправки исходящих действий.

## <a name="the-activity-processing-stack"></a>Стек обработки действия

Теперь давайте подробнее рассмотрим представленную выше схему, начиная с прибытия действия сообщения.

![Стек обработки действия](media/bot-builder-activity-processing-stack.png)

В приведенном выше примере в ответ на действие сообщения бот отправляет другое действие сообщения, помещая в него текст из первого сообщения. Обработка начинается с поступления на веб-сервер запроса HTTP POST, в котором сведения о действии передаются в формате полезных данных JSON. В C# этот процесс обычно выполняется в проекте ASP.NET, а для проектов JavaScript Node.js часто используются такие платформы, как Express и Restify.

Встроенный компонент пакета SDK, который называется *адаптером*, является ядром среды выполнения SDK. Действие передается в тексте запроса HTTP POST в формате JSON. Этот код JSON десериализуется в объект Activity (Действие), который обрабатывается адаптером с помощью вызова метода *process activity*. При получении действия адаптер создает *turn context* (контекст шага) и вызывает ПО промежуточного слоя. Имя *turn context* (контекст шага) основано на понятии "шаг" и употребляется для описания всех операций, связанных с получением действия. Контекст шага является одной из важнейших абстракций в пакете SDK, которая предоставляет всем компонентам ПО промежуточного слоя и логике приложения не только само входящее действие, но и механизм для отправки исходящих действий. Объект контекста предоставляет методы _отправки, обновления и удаления действия_, которые можно использовать при обработке полученного действия. Каждый метод ответа выполняется в асинхронном процессе. 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]


## <a name="middleware"></a>ПО промежуточного слоя
ПО промежуточного слоя действует так же, как любое средство обработки сообщений, и состоит из линейного набора компонентов, каждый из которых выполняется в строгом порядке и получает возможность применить некоторую логику реагирования на событие. Последним этапом конвейера ПО промежуточного слоя является обратный вызов функции обработчика шагов (`OnTurnAsync` для C# и `onTurn` для JS) из класса бота, которую приложение регистрирует в адаптере. Обработчик шагов принимает в качестве аргумента контекст шага. Заключенная в обработчике шагов логика приложении обычно обрабатывает содержимое входящего действия и создает в ответ одно или несколько действий, которые затем отправляет с помощью функции *send activity* для контекста шага. Вызов *send activity* для контекста шага приводит к вызову компонентов ПО промежуточного слоя для исходящих действий. Компоненты ПО промежуточного слоя выполняются до и после функции обработчика шагов бота. Такое выполнение является конструктивно вложенным и иногда называется "матрешкой". Дополнительные сведения о ПО промежуточного слоя см. [здесь](~/v4sdk/bot-builder-concept-middleware.md).

## <a name="bot-structure"></a>Структура бота
В следующих разделах мы рассмотрим основные компоненты бота.

### <a name="prerequisites"></a>Предварительные требования
- Копия примера **EchoBotWithCounter** на языке **[C#](https://aka.ms/EchoBotWithStateCSharp) или [JS](https://aka.ms/EchoBotWithStateJS)**. Здесь представлен только тот код, который важен для этой статьи. Полный исходный код можно найти в самом примере.

# <a name="ctabcs"></a>[C#](#tab/cs)

Бот — это разновидность веб-приложения [ASP.NET Core](https://docs.microsoft.com/aspnet/core/?view=aspnetcore-2.1). Основные компоненты для [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x) содержат похожий код в таких файлах, как **Program.cs** и **Startup.cs**. Эти файлы являются обязательными для всех веб-приложений и не зависят от конкретного бота. 

### <a name="bot-logic"></a>Логика бота

Основная логика бота определена в классе `EchoWithCounterBot`, который наследует интерфейс `IBot`. `IBot` определяет единственный метод `OnTurnAsync`. Приложение обязано реализовать этот метод. `OnTurnAsync` имеет свойство turnContext, через которое предоставляются сведения о входящем действии. Входящее действие соответствует поступившему HTTP-запросу. Входящие действия могут иметь разные типы, поэтому первым делом нужно проверить, получил ли бот сообщение. Если это действительно новое сообщение, мы получаем состояние беседы из очереди контекстов, увеличиваем значение счетчика и сохраняем его новое значение в состояние беседы. После этого мы отправляем пользователю сообщение с помощью вызова SendActivityAsync. Исходящее действие соответствует созданному HTTP-запросу.

```cs
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context.
        var oldState = await _accessors.CounterState.GetAsync(turnContext, () => new CounterState());

        // Bump the turn count for this conversation.
        var newState = new CounterState { TurnCount = oldState.TurnCount + 1 };

        // Set the property using the accessor.
        await _accessors.CounterState.SetAsync(turnContext, newState);

        // Save the new turn count into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        // Echo back to the user whatever they typed.
        var responseMessage = $"Turn {newState.TurnCount}: You sent '{turnContext.Activity.Text}'\n";
        await turnContext.SendActivityAsync(responseMessage);
    }
    else
    {
        await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
    }
}
```

### <a name="set-up-services"></a>Настройка служб

Метод `ConfigureServices` в файле startup.cs загружает подключенные службы из файла [.bot](bot-builder-basics.md#the-bot-file), перехватывает все возникшие на текущем шаге беседы ошибки и сохраняет их в журнал, затем настраивает поставщик учетных данных и создает объект состояния общения для хранения в памяти данных беседы.

```csharp
services.AddBot<EchoWithCounterBot>(options =>
{
    // Creates a logger for the application to use.
    ILogger logger = _loggerFactory.CreateLogger<EchoWithCounterBot>();

    var secretKey = Configuration.GetSection("botFileSecret")?.Value;
    var botFilePath = Configuration.GetSection("botFilePath")?.Value;

    // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
    BotConfiguration botConfig = null;
    try
    {
        botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
    }
    catch
    {
        //...
    }

    services.AddSingleton(sp => botConfig);

    // Retrieve current endpoint.
    var environment = _isProduction ? "production" : "development";
    var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
    if (!(service is EndpointService endpointService))
    {
        throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
    }

    options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

    // Catches any errors that occur during a conversation turn and logs them.
    options.OnTurnError = async (context, exception) =>
    {
        logger.LogError($"Exception caught : {exception}");
        await context.SendActivityAsync("Sorry, it looks like something went wrong.");
    };

    // The Memory Storage used here is for local bot debugging only. When the bot
    // is restarted, everything stored in memory will be gone.
    IStorage dataStore = new MemoryStorage();

    // ...

    // Create Conversation State object.
    // The Conversation State object is where we persist anything at the conversation-scope.
    var conversationState = new ConversationState(dataStore);

    options.State.Add(conversationState);
});
```

Метод `ConfigureServices` также создает и регистрирует методы доступа `EchoBotAccessors`, которые определены в файле **EchoBotStateAccessors.cs** и передаются в открытом конструкторе `EchoWithCounterBot` через платформу внедрения зависимостей в ASP.NET Core.

```csharp
// Accessors created here are passed into the IBot-derived class on every turn.
services.AddSingleton<EchoBotAccessors>(sp =>
{
    var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
    // ...
    var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
    // ...

    // Create the custom state accessor.
    // State accessors enable other components to read and write individual properties of state.
    var accessors = new EchoBotAccessors(conversationState)
    {
        CounterState = conversationState.CreateProperty<CounterState>(EchoBotAccessors.CounterStateName),
    };

    return accessors;
});
```

Метод `Configure` указывается в конце конфигурации приложения вместе с информацией о том, что приложение использует Bot Framework и несколько других файлов. Для всех ботов, использующих Bot Framework, требуется выполнить вызов конфигурации. `ConfigureServices` и `Configure` вызываются средой выполнения при запуске приложения.

### <a name="manage-state"></a>Управление состоянием

Этот файл содержит простой класс, в котором бот сохраняет сведения о текущем состоянии. Он содержит только элемент `int`, который мы используем для увеличения значений счетчика.

```cs
public class CounterState
{
    public int TurnCount { get; set; } = 0;
}
```

### <a name="accessor-class"></a>Класс методов доступа

Класс `EchoBotAccessors` создается как singleton в классе `Startup` и передается в производный класс IBot. В этом случае — `public class EchoWithCounterBot : IBot`. Бот использует этот метод доступа для сохранения данных беседы. Конструктор `EchoBotAccessors` передается в объект беседы, который создается в файле Startup.cs.

```cs
public class EchoBotAccessors
{
    public EchoBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string CounterStateName { get; } = $"{nameof(EchoBotAccessors)}.CounterState";

    public IStatePropertyAccessor<CounterState> CounterState { get; set; }

    public ConversationState ConversationState { get; }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Генератор Yeoman создает веб-приложение с типом [restify](http://restify.com/). Краткое руководство по быстрому началу работы в документации по restify содержит приложение, аналогичное созданному файлу **index.js**. В этом разделе описаны главным образом файлы **package.json**, **.env** , **index.js**, **bot.js** и **echobot-with-counter.bot**. Код некоторых из них мы здесь не приводим, но вы увидите его при выполнении бота или можете найти в примере [echobot-with-counter для Node.js](https://aka.ms/js-echobot-with-counter).

### <a name="packagejson"></a>package.json

В файле **package.json** указываются зависимости и их связанные версии для бота. Все они настраиваются и системой и согласно шаблону.

### <a name="env-file"></a>Файл ENV

В файле **ENV** указываются сведения о конфигурации для вашего бота, в том числе и номер порта, идентификатор приложения и пароль. Если вы применяете некоторые технологии или используете этот бот в рабочей среде, нужно добавить в эту конфигурацию определенные ключи или URL-адрес. Сейчас для Echo Bot не нужно добавлять такие сведения. Можете не указывать идентификатор приложения и пароль на этом этапе.

Чтобы использовать файл конфигурации **ENV**, нужно добавить в шаблон пакет.  Сначала получите пакет `dotenv` из npm:

`npm install dotenv`

### <a name="indexjs"></a>Файл index.js

`index.js` настраивает бота и службы размещения, которые будут пересылать действия в логику бота.

#### <a name="required-libraries"></a>Обязательные библиотеки

В начале файла `index.js` вы найдете набор обязательных модулей или библиотек. Эти модули предоставляют доступ к набору функций, которые, возможно, потребуется добавить в приложение.

```javascript
// Import required packages
const path = require('path');
const restify = require('restify');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
// Import required bot configuration.
const { BotConfiguration } = require('botframework-config');

const { EchoBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
const env = require('dotenv').config({ path: ENV_FILE });
```

#### <a name="bot-configuration"></a>Конфигурация бота

На следующем этапе загружаются данные из файла конфигурации бота.

```javascript
// Get the .bot file path
// See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));
let botConfig;
try {
    // Read bot configuration from .bot file.
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.`);
    console.error(`\n - See https://aka.ms/about-bot-file to learn more about .bot file its use and bot configuration.\n\n`);
    process.exit();
}

// For local development configuration as defined in .bot file
const DEV_ENVIRONMENT = 'development';

// Define name of the endpoint configuration section from the .bot file
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Get bot endpoint configuration by service name
// Bot configuration as defined in .bot file
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);
```

#### <a name="bot-adapter-http-server-and-bot-state"></a>Адаптер бота, HTTP-сервер и состояние бота

На следующем этапе выполняется настройка сервера и адаптера, которые позволят боту общаться с пользователем и отправлять ответы. Сервер будет ожидать данные на порту, который указан в файле конфигурации **BotConfiguration.bot**. По умолчанию используется порт _3978_ для связи с эмулятором. Адаптер будет выступать в качестве проводника для вашего бота, управлять входящими и исходящими сообщениями, аутентификацией и т. д.

Также мы создаем объект состояния, который использует `MemoryStorage` в качестве поставщика хранилища. Это состояние определяется в виде свойства `ConversationState`, которое просто позволяет сохранить данные о состоянии диалога. `ConversationState` позволяет сохранить необходимые сведения. В нашем случае это просто данные счетчика реплик в памяти.

```javascript
// Create bot adapter.
// See https://aka.ms/about-bot-adapter to learn more about bot adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.microsoftAppID,
    appPassword: endpointConfig.appPassword || process.env.microsoftAppPassword
});

// Catch-all for any unhandled errors in your bot.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
    // Clear out state
    await conversationState.clear(context);
    // Save state changes.
    await conversationState.saveChanges(context);
};

// Define a state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state store to persist the dialog and user state between messages.
let conversationState;

// For local development, in-memory storage is used.
// CAUTION: The Memory Storage used here is for local bot debugging only. When the bot
// is restarted, anything stored in memory will be gone.
const memoryStorage = new MemoryStorage();
conversationState = new ConversationState(memoryStorage);

// Create the main dialog.
const bot = new EchoBot(conversationState);

// Create HTTP server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator`);
    console.log(`\nTo talk to your bot, open echoBot-with-counter.bot file in the Emulator`);
});
```

#### <a name="bot-logic"></a>Логика бота

Метод адаптера `processActivity` отправляет входящие действия в логику бота.
Третий параметр в `processActivity` представляет собой функцию-обработчик, которая вызывается для выполнения логики бота после того, как адаптер и ПО промежуточного слоя выполнят для полученного [действия](#the-activity-processing-stack) все этапы предварительной обработки. Переменную контекста шага, которая передается обработчику функций в виде аргумента, можно использовать для предоставления сведений о входящем действии, отправителе и получателе, канале, беседе и т.д. Для обработки действие передается в `onTurn` из EchoBot.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // Route to main dialog.
        await bot.onTurn(context);
    });
});
```

### <a name="echobot"></a>EchoBot

Все обрабатываемые действия перенаправляются к обработчику `onTurn` в этом классе. При создании класса ему передается объект состояния. Используя этот объект состояния, конструктор создает метод доступа `this.countProperty` для сохранения счетчика шагов бота.

На каждом шаге мы первым делом проверяем, получил ли бот сообщение. Если это не сообщение, мы отправляем обратно тип полученного действия. После этого создается переменная состояния, которая позволяет хранить информацию о диалоге с ботом. Если переменная счетчика имеет значение `undefined`, ей присваивается новое значение 1 (это произойдет при первом запуске бота), в противном случае ее значение увеличивается на единицу (для каждого нового сообщения). Значение этого счетчика мы возвращаем пользователю вместе с полученным от него сообщением. И, наконец, мы устанавливаем значение счетчика и сохраняем изменения в состоянии.

```javascript
const { ActivityTypes } = require('botbuilder');

// Turn counter property
const TURN_COUNTER_PROPERTY = 'turnCounterProperty';

class EchoBot {

    constructor(conversationState) {
        // Creates a new state accessor property.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors
        this.countProperty = conversationState.createProperty(TURN_COUNTER_PROPERTY);
        this.conversationState = conversationState;
    }

    async onTurn(turnContext) {
        // Handle message activity type. User's responses via text or speech or card interactions flow back to the bot as Message activity.
        // Message activities may contain text, speech, interactive cards, and binary or unknown attachments.
        // see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
        if (turnContext.activity.type === ActivityTypes.Message) {
            // read from state.
            let count = await this.countProperty.get(turnContext);
            count = count === undefined ? 1 : ++count;
            await turnContext.sendActivity(`${ count }: You said "${ turnContext.activity.text }"`);
            // increment and set turn counter.
            await this.countProperty.set(turnContext, count);
        } else {
            // Generic handler for all other activity types.
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
        // Save state changes
        await this.conversationState.saveChanges(turnContext);
    }
}

exports.EchoBot = EchoBot;
```

---

## <a name="the-bot-file"></a>Файл бота

Файл **.bot** содержит некоторые сведения, например конечную точку, идентификатор приложения, пароль и ссылки на службы, используемые этим ботом. Этот файл создается автоматически, когда вы создаете бот на основе шаблона. Но вы можете создавать свой файл с помощью эмулятора или других средств. Вы можете указать, какой файл .bot следует использовать для проверки бота через [эмулятор](../bot-service-debug-emulator.md).

```json
{
    "name": "echobot-with-counter",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "endpoint": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "id": "1"
        }
    ],
    "padlock": "",
    "version": "2.0"
}
```

## <a name="additional-resources"></a>Дополнительные ресурсы

- Общие сведения о роли состояния для ботов см. в статье [об управлении состоянием](bot-builder-concept-state.md).
- Чтобы понять, какую роль играет BOT-файл в управлении ресурсами, изучите статью [об управлении ресурсами в файле .bot](bot-file-basics.md).
- Чтобы создать свой первый бот, воспользуйтесь одним из кратких руководств: [для службы Azure Bot](../bot-service-quickstart.md), [для C#](../dotnet/bot-builder-dotnet-sdk-quickstart.md) или [для JavaScript](../javascript/bot-builder-javascript-quickstart.md)

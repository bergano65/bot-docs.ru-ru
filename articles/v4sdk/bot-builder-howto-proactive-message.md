---
title: Получение уведомлений от бота | Документация Майкрософт
description: Узнайте, как отправлять уведомления
keywords: proactive message, notification message, bot notification,
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/08/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fac1e026ac92fcbe1b5c5bb9363c29e1d9e9b02a
ms.sourcegitcommit: b6327fa0b4547556d2d45d8910796e0c02948e43
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/15/2018
ms.locfileid: "51681591"
---
# <a name="get-notification-from-a-bot"></a>Получение уведомлений от бота

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Как правило, каждое сообщение, которое бот отправляет пользователю, напрямую связано с данными, введенными пользователем ранее.
Иногда боту может потребоваться отправить пользователю сообщение, которое не имеет прямого отношения к текущей теме диалога или последнему сообщению пользователя. Такие сообщения называются _упреждающими_.

## <a name="uses"></a>Использование

Упреждающие сообщения можно использовать в различных сценариях. Если бот устанавливает таймер или напоминание, ему потребуется оповестить пользователя в соответствующий момент времени. Если бот получает уведомление от внешней системы, ему может потребоваться незамедлительно передать эту информацию пользователю. Например, если пользователь ранее попросил бот отслеживать стоимость продукта, то бот может оповестить пользователя в случае, если стоимость продукта опустилась на 20 %. Если боту требуется какое-то время, чтобы сформировать ответ на вопрос пользователя, он может проинформировать пользователя о задержке и продолжить диалог. Когда бот сформирует ответ на вопрос, он передаст его пользователю.

При реализации упреждающих сообщений бота обратите внимание на следующее:

- Не следует отправлять несколько упреждающих сообщений за короткое время. Некоторые каналы ограничивают частоту, с которой бот может отправлять сообщения пользователю, и отключают бот, если он нарушает эти ограничения.
- Не следует отправлять упреждающие сообщения пользователям, которые ранее не взаимодействовали с ботом или хотели связаться с ботом по другим каналам связи, например, по электронной почте или через SMS.

**Динамическое упреждающее сообщение** — это самый простой тип упреждающих сообщений.
Бот просто вставляет сообщение в диалог каждый раз, когда он активируется (вне зависимости от участия пользователя в отдельном разделе диалога с ботом в данный момент), и не пытается изменить диалог каким-либо способом.

Чтобы улучшить обработку уведомлений, рассмотрите другие варианты их интеграции в процесс общения, например указав флаг в состоянии диалога или добавив уведомление в очередь.

### <a name="prerequisites"></a>Предварительные требования
- Копия **примера упреждающих сообщений** на [ C# ](https://aka.ms/proactive-sample-cs) или [JS](https://aka.ms/proactive-sample-js).
- Если вы используете JS, установите [Bot Builder](https://www.npmjs.com/package/botbuilder) для Node.js


### <a name="about-the-sample-code"></a>Сведения о примере кода

В примере упреждающих сообщений смоделированы задачи пользователя, длительность которых заранее неизвестна. Бот сохраняет сведения о такой задаче и сообщает пользователю о том, что сообщит о ее завершении, а затем продолжает беседу. После завершения задачи бот отправляет упреждающее сообщение с подтверждением в исходный сеанс беседы.

#### <a name="define-job-data-and-state"></a>Определение данных и состояния задания

В этом случае отслеживаются произвольные задачи, которые могут быть созданы разными пользователями в нескольких сеансах беседы. Нам необходимо сохранить сведения о каждом из этих заданий, в том числе ссылку на беседу и идентификатор задания. Нам потребуется:
- Ссылка на беседу, чтобы отправить упреждающее сообщение в правильный сеанс.
- Способ идентификации заданий. В этом примере мы используем простую метку времени.
- Возможность хранить сведения о состоянии заданий независимо от сведений о состоянии беседы и (или) данных пользователя.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Нам нужно определить классы для данных задания и состояния задания. Также мы хотим зарегистрировать наш чат-бот и настроить метод доступа к свойству состояния для журнала заданий.

#### <a name="define-a-class-for-job-data"></a>Определение класса для данных задания

Класс `JobLog` отслеживает данные задания, индексируя их по номеру задания (метке времени). Класс `JobLog` отслеживает все невыполненные задания.  Каждое задание идентифицируется уникальным ключом. `Job data` описывает состояние задания и определяется как внутренний класс словаря.

```csharp
public class JobLog : Dictionary<long, JobLog.JobData>
{
    public class JobData
    {
        // Gets or sets the time-stamp for the job.
        public long TimeStamp { get; set; } = 0;

        // Gets or sets a value indicating whether indicates whether the job has completed.
        public bool Completed { get; set; } = false;

        // Gets or sets the conversation reference to which to send status updates.
        public ConversationReference Conversation { get; set; }
    }
}
```

#### <a name="define-a-state-middleware-class"></a>Определение класса ПО промежуточного слоя с поддержкой состояния

Класс **JobState** управляет сведениями о состоянии задания, независимо от состояния беседы и (или) пользователя.

```csharp
using Microsoft.Bot.Builder;

/// A BotState for managing bot state for "bot jobs".
public class JobState : BotState
{
    // The key used to cache the state information in the turn context.
    private const string StorageKey = "ProactiveBot.JobState";

    // Initializes a new instance of the JobState class.
    public JobState(IStorage storage)
        : base(storage, StorageKey)
    {
    }

    // Gets the storage key for caching state information.
    protected override string GetStorageKey(ITurnContext turnContext) => StorageKey;
}
```

### <a name="register-the-bot-and-required-services"></a>Регистрация бота и необходимых служб

Файл **Startup.cs** регистрирует чат-бот и связанные с ним службы.

1. Метод `ConfigureServices` регистрирует чат-бот, в том числе обработку ошибок и управление состоянием. Он также регистрирует службы конечной точки бота и метода доступа к состоянию задания.

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // ...

        // Create Job State object.
        // The Job State object is where we persist anything at the job-scope.
        // Note: It's independent of any user or conversation.
        var jobState = new JobState(dataStore);

        // Make it available to our bot
        services.AddSingleton(sp => jobState);

        // Register the proactive bot.
        services.AddBot<ProactiveBot>(options =>
        {
            var secretKey = Configuration.GetSection("botFileSecret")?.Value;
            var botFilePath = Configuration.GetSection("botFilePath")?.Value;

            // Loads .bot configuration file and adds a singleton that your Bot can access through dependency injection.
            var botConfig = BotConfiguration.Load(botFilePath ?? @".\BotConfiguration.bot", secretKey);
            services.AddSingleton(sp => botConfig ?? throw new InvalidOperationException($"The .bot config file could not be loaded. ({botConfig})"));

            // Retrieve current endpoint.
            var environment = _isProduction ? "production" : "development";
            var service = botConfig.Services.Where(s => s.Type == "endpoint" && s.Name == environment).FirstOrDefault();
            if (!(service is EndpointService endpointService))
            {
                throw new InvalidOperationException($"The .bot file does not contain an endpoint with name '{environment}'.");
            }

            options.CredentialProvider = new SimpleCredentialProvider(endpointService.AppId, endpointService.AppPassword);

            // Creates a logger for the application to use.
            ILogger logger = _loggerFactory.CreateLogger<ProactiveBot>();

            // Catches any errors that occur during a conversation turn and logs them.
            options.OnTurnError = async (context, exception) =>
            {
                logger.LogError($"Exception caught : {exception}");
                await context.SendActivityAsync("Sorry, it looks like something went wrong.");
            };

        });

        services.AddSingleton(sp =>
        {
            var config = BotConfiguration.Load(@".\BotConfiguration.bot");
            var endpointService = (EndpointService)config.Services.First(s => s.Type == "endpoint")
                                    ?? throw new InvalidOperationException(".bot file 'endpoint' must be configured prior to running.");

            return endpointService;
        });
    }
    ```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Файл **Index.js** предназначен для выполнения следующих задач:
- задает ссылку на класс бота и файл **.bot**;
- создание объектов HTTP-сервера, адаптера ботов и хранилища;
- создает бот и запускает сервер, передавая боту действия.

```javascript
const restify = require('restify');
const path = require('path');

// Import required bot services. See https://aka.ms/bot-services to learn more about the different part of a bot.
const { BotFrameworkAdapter, BotState, MemoryStorage } = require('botbuilder');
const { BotConfiguration } = require('botframework-config');

const { ProactiveBot } = require('./bot');

// Read botFilePath and botFileSecret from .env file.
// Note: Ensure you have a .env file and include botFilePath and botFileSecret.
const ENV_FILE = path.join(__dirname, '.env');
require('dotenv').config({ path: ENV_FILE });

// Create HTTP server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function() {
    console.log(`\n${ server.name } listening to ${ server.url }.`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/botframework-emulator.`);
    console.log(`\nTo talk to your bot, open proactive-messages.bot file in the Emulator.`);
});

// .bot file path
const BOT_FILE = path.join(__dirname, (process.env.botFilePath || ''));

// Read the bot's configuration from a .bot file identified by BOT_FILE.
// This includes information about the bot's endpoints and configuration.
let botConfig;
try {
    botConfig = BotConfiguration.loadSync(BOT_FILE, process.env.botFileSecret);
} catch (err) {
    console.error(`\nError reading bot file. Please ensure you have valid botFilePath and botFileSecret set for your environment.`);
    console.error(`\n - The botFileSecret is available under appsettings for your Azure Bot Service bot.`);
    console.error(`\n - If you are running this bot locally, consider adding a .env file with botFilePath and botFileSecret.\n\n`);
    process.exit();
}

const DEV_ENVIRONMENT = 'development';

// Define the name of the bot, as specified in .bot file.
// See https://aka.ms/about-bot-file to learn more about .bot files.
const BOT_CONFIGURATION = (process.env.NODE_ENV || DEV_ENVIRONMENT);

// Load the configuration profile specific to this bot identity.
const endpointConfig = botConfig.findServiceByNameOrId(BOT_CONFIGURATION);

// Create the adapter. See https://aka.ms/about-bot-adapter to learn more about using information from
// the .bot file when configuring your adapter.
const adapter = new BotFrameworkAdapter({
    appId: endpointConfig.appId || process.env.MicrosoftAppId,
    appPassword: endpointConfig.appPassword || process.env.MicrosoftAppPassword
});

// Define the state store for your bot. See https://aka.ms/about-bot-state to learn more about using MemoryStorage.
// A bot requires a state storage system to persist the dialog and user state between messages.
const memoryStorage = new MemoryStorage();

// Create state manager with in-memory storage provider.
const botState = new BotState(memoryStorage, () => 'proactiveBot.botState');

// Create the main dialog, which serves as the bot's main handler.
const bot = new ProactiveBot(botState, adapter);

// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (turnContext) => {
        // Route the message to the bot's main handler.
        await bot.onTurn(turnContext);
    });
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    context.sendActivity(`Oops. Something went wrong!`);
};
```

---

### <a name="define-the-bot"></a>Определение бота

Пользователь может попросить чат-бота создать и запустить некоторое задание. Отдельная служба заданий будет уведомлять бота о завершении таких заданий. Сам бот должен выполнять следующее:

- создавать задание в ответ на сообщения `run` или `run job` от пользователя;
- возвращать список уже зарегистрированных заданий в ответ на сообщения `show` или `show jobs` от пользователя;
- завершать задание в ответ на событие _задание выполнено_ с указанием завершенного задания;
- имитировать событие выполнения задания в ответ на сообщение `done <jobIdentifier>`;
- отправлять пользователю в исходный сеанс общения упреждающее сообщение о завершении задания.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Бот состоит из нескольких частей:

- код инициализации;
- обработчик шагов;
- методы для создания и завершения заданий.

#### <a name="declare-the-class"></a>Объявление класса

```csharp
namespace Microsoft.BotBuilderSamples
{
    // For each interaction from the user, an instance of this class is called.
    // This is a Transient lifetime service.  Transient lifetime services are created
    // each time they're requested. For each Activity received, a new instance of this
    // class is created. Objects that are expensive to construct, or have a lifetime
    // beyond the single Turn, should be carefully managed.
    public class ProactiveBot : IBot
    {
        // The name of events that signal that a job has completed.
        public const string JobCompleteEventName = "jobComplete";

        public const string WelcomeText = "Type 'run' or 'run job' to start a new job.\r\n" +
                                          "Type 'show' or 'show jobs' to display the job log.\r\n" +
                                          "Type 'done <jobNumber>' to complete a job.";
    }
}
```

#### <a name="add-initialization-code"></a>Добавление кода инициализации

```csharp
private readonly JobState _jobState;
private readonly IStatePropertyAccessor<JobLog> _jobLogPropertyAccessor;

public ProactiveBot(JobState jobState, EndpointService endpointService)
{
    _jobState = jobState ?? throw new ArgumentNullException(nameof(jobState));
    _jobLogPropertyAccessor = _jobState.CreateProperty<JobLog>(nameof(JobLog));

    // Validate AppId.
    // Note: For local testing, .bot AppId is empty for the Bot Framework Emulator.
    AppId = string.IsNullOrWhiteSpace(endpointService.AppId) ? "1" : endpointService.AppId;
}

private string AppId { get; }
```

#### <a name="add-a-turn-handler"></a>Добавление обработчика шагов

Каждый бот должен реализовать обработчик шагов. Адаптер пересылает действия именно в этот метод.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type != ActivityTypes.Message)
    {
        // Handle non-message activities.
        await OnSystemActivityAsync(turnContext);
    }
    else
    {
        // Get the job log.
        // The job log is a dictionary of all outstanding jobs in the system.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Get the user's text input for the message.
        var text = turnContext.Activity.Text.Trim().ToLowerInvariant();
        switch (text)
        {
            case "run":
            case "run job":

                // Start a virtual job for the user.
                JobLog.JobData job = CreateJob(turnContext, jobLog);

                // Set the new property
                await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

                // Now save it into the JobState
                await _jobState.SaveChangesAsync(turnContext);

                await turnContext.SendActivityAsync(
                    $"We're starting job {job.TimeStamp} for you. We'll notify you when it's complete.");

                break;

            case "show":
            case "show jobs":

                // Display information for all jobs in the log.
                if (jobLog.Count > 0)
                {
                    await turnContext.SendActivityAsync(
                        "| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>" +
                        "| :--- | :---: | :---: |<br>" +
                        string.Join("<br>", jobLog.Values.Select(j =>
                            $"| {j.TimeStamp} &nbsp; | {j.Conversation.Conversation.Id.Split('|')[0]} &nbsp; | {j.Completed} |")));
                }
                else
                {
                    await turnContext.SendActivityAsync("The job log is empty.");
                }

                break;

            default:
                // Check whether this is simulating a job completed event.
                string[] parts = text?.Split(' ', StringSplitOptions.RemoveEmptyEntries);
                if (parts != null && parts.Length == 2
                    && parts[0].Equals("done", StringComparison.InvariantCultureIgnoreCase)
                    && long.TryParse(parts[1], out long jobNumber))
                {
                    if (!jobLog.TryGetValue(jobNumber, out JobLog.JobData jobInfo))
                    {
                        await turnContext.SendActivityAsync($"The log does not contain a job {jobInfo.TimeStamp}.");
                    }
                    else if (jobInfo.Completed)
                    {
                        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is already complete.");
                    }
                    else
                    {
                        await turnContext.SendActivityAsync($"Completing job {jobInfo.TimeStamp}.");

                        // Send the proactive message.
                        await CompleteJobAsync(turnContext.Adapter, AppId, jobInfo);
                    }
                }

                break;
        }

        if (!turnContext.Responded)
        {
            await turnContext.SendActivityAsync(WelcomeText);
        }
    }
}

private static async Task SendWelcomeMessageAsync(ITurnContext turnContext)
{
    foreach (var member in turnContext.Activity.MembersAdded)
    {
        if (member.Id != turnContext.Activity.Recipient.Id)
        {
            await turnContext.SendActivityAsync($"Welcome to SuggestedActionsBot {member.Name}.\r\n{WelcomeText}");
        }
    }
}

// Handles non-message activities.
private async Task OnSystemActivityAsync(ITurnContext turnContext)
{
    // On a job completed event, mark the job as complete and notify the user.
    if (turnContext.Activity.Type is ActivityTypes.Event)
    {
        var jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());
        var activity = turnContext.Activity.AsEventActivity();
        if (activity.Name == JobCompleteEventName
            && activity.Value is long timestamp
            && jobLog.ContainsKey(timestamp)
            && !jobLog[timestamp].Completed)
        {
            await CompleteJobAsync(turnContext.Adapter, AppId, jobLog[timestamp]);
        }
    }
    else if (turnContext.Activity.Type is ActivityTypes.ConversationUpdate)
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            await SendWelcomeMessageAsync(turnContext);
        }
    }
}
```

#### <a name="add-job-creation-and-completion-methods"></a>Добавление методов создания и завершения задания

Чтобы запустить задание, чат-бот создает это задание и сохраняет в журнал заданий сведения о задании и текущей беседе. Когда бот получает событие выполнения задания для любого из сеансов общения, он проверяет идентификатор задания, а затем вызывает код обработки завершения задания.

Код обработки завершения задания получает журнал задания из данных состояния, помечает задание как завершенное и отправляет упреждающее сообщение с помощью метода _продолжения диалога_ в адаптере.

- Вызов продолжения диалога создает в канале следующий шаг, не зависящий от действий пользователя.
- Адаптер выполняет соответствующий обратный вызов вместо обычного обработчика шагов в боте. Этот шаг будет иметь собственный контекст, из который мы извлекаем из данных о состоянии и передаем вместе с упреждающим сообщением для пользователя.

```csharp
// Creates and "starts" a new job.
private JobLog.JobData CreateJob(ITurnContext turnContext, JobLog jobLog)
{
    JobLog.JobData jobInfo = new JobLog.JobData
    {
        TimeStamp = DateTime.Now.ToBinary(),
        Conversation = turnContext.Activity.GetConversationReference(),
    };

    jobLog[jobInfo.TimeStamp] = jobInfo;

    return jobInfo;
}

// Sends a proactive message to the user.
private async Task CompleteJobAsync(
    BotAdapter adapter,
    string botId,
    JobLog.JobData jobInfo,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await adapter.ContinueConversationAsync(botId, jobInfo.Conversation, CreateCallback(jobInfo), cancellationToken);
}

// Creates the turn logic to use for the proactive message.
private BotCallbackHandler CreateCallback(JobLog.JobData jobInfo)
{
    return async (turnContext, token) =>
    {
        // Get the job log from state, and retrieve the job.
        JobLog jobLog = await _jobLogPropertyAccessor.GetAsync(turnContext, () => new JobLog());

        // Perform bookkeeping.
        jobLog[jobInfo.TimeStamp].Completed = true;

        // Set the new property
        await _jobLogPropertyAccessor.SetAsync(turnContext, jobLog);

        // Now save it into the JobState
        await _jobState.SaveChangesAsync(turnContext);

        // Send the user a proactive confirmation message.
        await turnContext.SendActivityAsync($"Job {jobInfo.TimeStamp} is complete.");
    };
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Этот бот определен в файле **bot.js** и имеет несколько частей:

- код инициализации;
- обработчик шагов;
- методы для создания и завершения заданий.

#### <a name="declare-the-class-and-add-initialization-code"></a>Объявление класса и добавление кода инициализации

```javascript
const { ActivityTypes, TurnContext } = require('botbuilder');

const JOBS_LIST = 'jobs';

class ProactiveBot {
    constructor(botState, adapter) {
        this.botState = botState;
        this.adapter = adapter;

        this.jobsList = this.botState.createProperty(JOBS_LIST);
    }

    // ...
};

// Helper function to check if object is empty.
function isEmpty(obj) {
    for (var key in obj) {
        if (obj.hasOwnProperty(key)) {
            return false;
        }
    }
    return true;
};

module.exports.ProactiveBot = ProactiveBot;
```

#### <a name="the-turn-handler"></a>Обработчик шагов

Методы `onTurn` и `showJobs` определены в классе `ProactiveBot`. Метод `onTurn` обрабатывает входные данные от пользователей. Также он получает действия событий от системы обработки заданий, которую мы здесь не рассматриваем. Форматирует `showJobs` и отправляет журнал задания.

```javascript
/**
    *
    * @param {TurnContext} turnContext A TurnContext object representing an incoming message to be handled by the bot.
    */
async onTurn(turnContext) {
    // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
    if (turnContext.activity.type === ActivityTypes.Message) {
        const utterance = (turnContext.activity.text || '').trim().toLowerCase();
        var jobIdNumber;

        // If user types in run, create a new job.
        if (utterance === 'run') {
            await this.createJob(turnContext);
        } else if (utterance === 'show') {
            await this.showJobs(turnContext);
        } else {
            const words = utterance.split(' ');

            // If the user types done and a Job Id Number,
            // we check if the second word input is a number.
            if (words[0] === 'done' && !isNaN(parseInt(words[1]))) {
                jobIdNumber = words[1];
                await this.completeJob(turnContext, jobIdNumber);
            } else if (words[0] === 'done' && (words.length < 2 || isNaN(parseInt(words[1])))) {
                await turnContext.sendActivity('Enter the job ID number after "done".');
            }
        }

        if (!turnContext.responded) {
            await turnContext.sendActivity(`Say "run" to start a job, or "done <job>" to complete one.`);
        }
    } else if (turnContext.activity.type === ActivityTypes.Event && turnContext.activity.name === 'jobCompleted') {
        jobIdNumber = turnContext.activity.value;
        if (!isNaN(parseInt(jobIdNumber))) {
            await this.completeJob(turnContext, jobIdNumber);
        }
    }

    await this.botState.saveChanges(turnContext);
}

// Show a list of the pending jobs
async showJobs(turnContext) {
    const jobs = await this.jobsList.get(turnContext, {});
    if (Object.keys(jobs).length) {
        await turnContext.sendActivity(
            '| Job number &nbsp; | Conversation ID &nbsp; | Completed |<br>' +
            '| :--- | :---: | :---: |<br>' +
            Object.keys(jobs).map((key) => {
                return `${ key } &nbsp; | ${ jobs[key].reference.conversation.id.split('|')[0] } &nbsp; | ${ jobs[key].completed }`;
            }).join('<br>'));
    } else {
        await turnContext.sendActivity('The job log is empty.');
    }
}
```

#### <a name="logic-to-start-a-job"></a>Логика запуска задания

Метод `createJob` определен в классе `ProactiveBot`. Он создает для пользователя новое задание и заносит сведения о нем в журнал. В полной реализации он также должен отправлять эту информацию в систему выполнения заданий.

```javascript
// Save job ID and conversation reference.
async createJob(turnContext) {
    // Create a unique job ID.
    var date = new Date();
    var jobIdNumber = date.getTime();

    // Get the conversation reference.
    const reference = TurnContext.getConversationReference(turnContext.activity);

    // Get the list of jobs. Default it to {} if it is empty.
    const jobs = await this.jobsList.get(turnContext, {});

    // Try to find previous information about the saved job.
    const jobInfo = jobs[jobIdNumber];

    try {
        if (isEmpty(jobInfo)) {
            // Job object is empty so we have to create it
            await turnContext.sendActivity(`Need to create new job ID: ${ jobIdNumber }`);

            // Update jobInfo with new info
            jobs[jobIdNumber] = { completed: false, reference: reference };

            try {
                // Save to storage
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job has been processed
                await turnContext.sendActivity('Successful write to log.');
            } catch (err) {
                await turnContext.sendActivity(`Write failed: ${ err.message }`);
            }
        }
    } catch (err) {
        await turnContext.sendActivity(`Read rejected. ${ err.message }`);
    }
}
```

#### <a name="logic-to-complete-a-job"></a>Логика обработки завершения задания

Метод `completeJob` определен в классе `ProactiveBot`. Он выполняет операции внутреннего учета и отправляет пользователю упреждающее сообщение (в исходном диалоге пользователя) о завершении задания.

```javascript
async completeJob(turnContext, jobIdNumber) {
    // Get the list of jobs from the bot's state property accessor.
    const jobs = await this.jobsList.get(turnContext, {});

    // Find the appropriate job in the list of jobs.
    let jobInfo = jobs[jobIdNumber];

    // If no job was found, notify the user of this error state.
    if (isEmpty(jobInfo)) {
        await turnContext.sendActivity(`Sorry no job with ID ${ jobIdNumber }.`);
    } else {
        // Found a job with the ID passed in.
        const reference = jobInfo.reference;
        const completed = jobInfo.completed;

        // If the job is not yet completed and conversation reference exists,
        // use the adapter to continue the conversation with the job's original creator.
        if (reference && !completed) {
            // Since we are going to proactively send a message to the user who started the job,
            // we need to create the turnContext based on the stored reference value.
            await this.adapter.continueConversation(reference, async (proactiveTurnContext) => {
                // Complete the job.
                jobInfo.completed = true;
                // Save the updated job.
                await this.jobsList.set(turnContext, jobs);
                // Notify the user that the job is complete.
                await proactiveTurnContext.sendActivity(`Your queued job ${ jobIdNumber } just completed.`);
            });

            // Send a message to the person who completed the job.
            await turnContext.sendActivity('Job completed. Notification sent.');
        } else if (completed) { // The job has already been completed.
            await turnContext.sendActivity('This job is already completed, please start a new job.');
        };
    };
};
```

---

### <a name="test-your-bot"></a>Тестирование бота

Создайте бота и запустите его локально, открыв два окна Emulator.

1. Обратите внимание на то, что идентификатор диалога в этих окнах различается.
1. В первом окне несколько раз введите `run`, чтобы запустить несколько заданий.
1. Во втором окне введите `show`, чтобы просмотреть список заданий, внесенных в журнал.
1. Во втором окне введите `done <jobNumber>`, где `<jobNumber>` обозначает номер любого из перечисленных заданий без угловых скобок. (Код чат-бота будет воспринимать такое сообщение как событие завершения задания jobComplete.)
1. Вы увидите, что бот отправляет пользователю в первом окне упреждающее сообщение.

<!--TODO: Recreate the screen shots once we're happy with both the C# and JS versions of the code.-->

Теперь эта беседа будет выглядеть с точки зрения пользователя примерно так:

![Эмулятор сеанса для пользователя](~/v4sdk/media/how-to-proactive/user.png)

С точки зрения системы имитации заданий этот же сеанс выглядит так:

![Эмулятор сеанса для системы выполнения заданий](~/v4sdk/media/how-to-proactive/job-system.png)

<!-- Add a next steps section. -->

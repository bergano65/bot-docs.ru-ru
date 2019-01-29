---
title: Получение уведомлений от ботов | Документация Майкрософт
description: Узнайте, как отправлять уведомления
keywords: proactive message, notification message, bot notification,
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7a9a2e4f30d1e9b293e51a921afce57d243376d7
ms.sourcegitcommit: c6ce4c42fc56ce1e12b45358d2c747fb77eb74e2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/22/2019
ms.locfileid: "54453968"
---
# <a name="get-notification-from-bots"></a>Получение уведомлений от ботов

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Как правило, каждое сообщение, которое бот отправляет пользователю, напрямую связано с данными, введенными пользователем ранее.
Иногда боту может потребоваться отправить пользователю сообщение, которое не имеет прямого отношения к текущей теме диалога или последнему сообщению пользователя. Такие сообщения называются _упреждающими_.

## <a name="proactive-messages"></a>Упреждающие сообщения

Упреждающие сообщения можно использовать в различных сценариях. Если бот устанавливает таймер или напоминание, ему потребуется оповестить пользователя в соответствующий момент времени. Если бот получает уведомление от внешней системы, ему может потребоваться незамедлительно передать эту информацию пользователю. Например, если пользователь ранее попросил бот отслеживать стоимость продукта, то бот может оповестить пользователя в случае, если стоимость продукта опустилась на 20 %. Если боту требуется какое-то время, чтобы сформировать ответ на вопрос пользователя, он может проинформировать пользователя о задержке и продолжить диалог. Когда бот сформирует ответ на вопрос, он передаст его пользователю.

При реализации упреждающих сообщений бота обратите внимание на следующее:

- Не следует отправлять несколько упреждающих сообщений за короткое время. Некоторые каналы ограничивают частоту, с которой бот может отправлять сообщения пользователю, и отключают бот, если он нарушает эти ограничения.
- Не следует отправлять упреждающие сообщения пользователям, которые ранее не взаимодействовали с ботом или хотели связаться с ботом по другим каналам связи, например, по электронной почте или через SMS.

Динамическое упреждающее сообщение — это самый простой тип упреждающих сообщений. Бот просто вставляет сообщение в диалог каждый раз, когда он активируется (вне зависимости от участия пользователя в отдельном разделе диалога с ботом в данный момент), и не пытается изменить диалог каким-либо способом.

Чтобы улучшить обработку уведомлений, рассмотрите другие варианты их интеграции в поток общения, например указав флаг в состоянии диалога или добавив уведомление в очередь.

## <a name="prerequisites"></a>Предварительные требования

- Понимание основ [работы с ботами](bot-builder-basics.md) и [управления состоянием](bot-builder-concept-state.md).
- Копия **примера упреждающих сообщений** на [ C# ](https://aka.ms/proactive-sample-cs) или [JS](https://aka.ms/proactive-sample-js). Этот пример используется в статье в качестве иллюстрации для упреждающего обмена сообщениями. 

## <a name="about-the-sample-code"></a>Сведения о примере кода

В примере упреждающих сообщений смоделированы задачи пользователя, длительность которых заранее неизвестна. Бот сохраняет сведения о такой задаче и сообщает пользователю о том, что сообщит о ее завершении, а затем продолжает беседу. После завершения задачи бот отправляет упреждающее сообщение с подтверждением в исходный сеанс беседы.

## <a name="define-job-data-and-state"></a>Определение данных и состояния задания

В этом случае отслеживаются произвольные задачи, которые могут быть созданы разными пользователями в нескольких сеансах беседы. Нам необходимо сохранить сведения о каждом из этих заданий, в том числе ссылку на беседу и идентификатор задания. Нам потребуется:

- Ссылка на беседу, чтобы отправить упреждающее сообщение в правильный сеанс.
- Способ идентификации заданий. В этом примере мы используем простую метку времени.
- Возможность хранить сведения о состоянии заданий независимо от сведений о состоянии беседы и (или) данных пользователя.

Мы расширим свойство _bot state_, чтобы определить собственный объект управления состоянием на уровне бота. Bot Framework использует _ключ к хранилищу данных_ и контекст шага для сохранения и извлечения сведений о состоянии. См. дополнительные сведения об [управлении состоянием](bot-builder-concept-state.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Нам нужно определить классы для данных задания и состояния задания. Также мы хотим зарегистрировать наш чат-бот и настроить метод доступа к свойству состояния для журнала заданий.

### <a name="define-a-class-for-job-data"></a>Определение класса для данных задания

Класс `JobLog` отслеживает данные задания, индексируя их по номеру задания (метке времени). Класс `JobLog` отслеживает все невыполненные задания.  Каждое задание идентифицируется уникальным ключом. `JobData` описывает состояние задания и определяется как внутренний класс словаря.

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

### <a name="define-a-state-management-class"></a>Определение класса управления состоянием

Класс `JobState` управляет состоянием задания независимо от состояния беседы или пользователя.

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

Метод `ConfigureServices` регистрирует бот и службу конечной точки, в том числе обработку ошибок и управление состоянием. Он также регистрирует метод доступа к состоянию задания.

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

    // ...      
    }
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Бот использует систему хранения состояния для долгосрочного хранения состояний диалога и пользователя между сообщениями. В нашем примере для этого создается поставщик хранилища в памяти.

```javascript
// index.js

const memoryStorage = new MemoryStorage();
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

// ...
```

---

## <a name="define-the-bot"></a>Определение бота

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

### <a name="declare-the-class"></a>Объявление класса

Каждое действие, полученное от пользователя, создает экземпляр класса `ProactiveBot`. Такой подход, при котором служба создается каждый раз, когда она нужна, называется службой с временным временем существования. Следует уделять особое внимание объектам, на создание которых требуется много ресурсов и (или) которые должны существовать дольше одного шага диалога.

```csharp
namespace Microsoft.BotBuilderSamples
{
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

### <a name="add-initialization-code"></a>Добавление кода инициализации

```csharp
private readonly JobState _jobState;
private readonly IStatePropertyAccessor<JobLog> _jobLogPropertyAccessor;

public ProactiveBot(JobState jobState, EndpointService endpointService)
{
    _jobState = jobState ?? throw new ArgumentNullException(nameof(jobState));
    _jobLogPropertyAccessor = _jobState.CreateProperty<JobLog>(nameof(JobLog));

    //...
}

```

### <a name="add-a-turn-handler"></a>Добавление обработчика шагов

Адаптер перенаправляет действия обработчику шагов, который в свою очередь проверяет тип `Activity` и вызывает подходящий метод. Каждый бот должен реализовать обработчик шагов.

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
                // ...
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
```

### <a name="handle-non-message-activities"></a>Обработка действий без сообщения

Получив сообщение о выполнении задания, пометьте это задание как завершенное и уведомите пользователя.

```csharp
private async Task OnSystemActivityAsync(ITurnContext turnContext)
{
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

### <a name="add-job-creation-and-completion-methods"></a>Добавление методов создания и завершения задания

Чтобы запустить задание, чат-бот создает это задание и сохраняет в журнал заданий сведения о задании и текущей беседе. Когда бот получает событие выполнения задания для любого из сеансов общения, он проверяет идентификатор задания, а затем вызывает код обработки завершения задания.

Код обработки завершения задания получает журнал задания из данных состояния, помечает задание как завершенное и отправляет упреждающее сообщение с помощью метода `ContinueConversationAsync` в адаптере.

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
```

### <a name="sends-a-proactive-message-to-the-user"></a>Отправка пользователю упреждающего сообщения

```csharp
private async Task CompleteJobAsync(
    BotAdapter adapter,
    string botId,
    JobLog.JobData jobInfo,
    CancellationToken cancellationToken = default(CancellationToken))
{
    await adapter.ContinueConversationAsync(botId, jobInfo.Conversation, CreateCallback(jobInfo), cancellationToken);
}
```

### <a name="creates-the-turn-logic-to-use-for-the-proactive-message"></a>Создание логики шага, которую нужно использовать для упреждающего сообщения

```csharp
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

### <a name="declare-the-class-and-add-initialization-code"></a>Объявление класса и добавление кода инициализации

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

### <a name="the-turn-handler"></a>Обработчик шагов

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

### <a name="logic-to-start-a-job"></a>Логика запуска задания

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

### <a name="logic-to-complete-a-job"></a>Логика обработки завершения задания

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

## <a name="test-your-bot"></a>Тестирование бота

Создайте бота и запустите его локально, открыв два окна Emulator. Если вам нужны пошаговые инструкции, изучите файл [README](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/16.proactive-messages/README.md).

1. Обратите внимание на то, что идентификатор диалога в этих окнах различается.
1. В первом окне несколько раз введите `run`, чтобы запустить несколько заданий.
1. Во втором окне введите `show`, чтобы просмотреть список заданий, внесенных в журнал.
1. Во втором окне введите `done <jobNumber>`, где `<jobNumber>` обозначает номер любого из перечисленных заданий без угловых скобок. (Код чат-бота будет воспринимать такое сообщение как событие завершения задания jobComplete.)
1. Вы увидите, что бот отправляет пользователю в первом окне упреждающее сообщение.

Теперь эта беседа будет выглядеть с точки зрения пользователя примерно так:

![Эмулятор сеанса для пользователя](~/v4sdk/media/how-to-proactive/user.png)

С точки зрения системы имитации заданий этот же сеанс выглядит так:

![Эмулятор сеанса для системы выполнения заданий](~/v4sdk/media/how-to-proactive/job-system.png)

## <a name="additional-resources"></a>Дополнительные ресурсы
Изучите дополнительные примеры для языков в C# и JS на сайте [GitHub](https://github.com/Microsoft/BotBuilder-Samples/blob/master/readme.md).

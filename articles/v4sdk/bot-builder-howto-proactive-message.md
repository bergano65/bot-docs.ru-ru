---
title: Отправка упреждающих сообщений | Документация Майкрософт
description: Узнайте, как обмениваться упреждающими сообщениями с ботом.
keywords: упреждающее сообщение
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/01/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c22ce6a35d4d49506360a78a439f15137c429d9d
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905138"
---
# <a name="send-proactive-messages"></a>Отправка упреждающих сообщений 

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]


Часто боты отправляют _реактивные сообщения_, но бывают случаи, когда необходимо иметь возможность отправлять [упреждающие сообщения](bot-builder-proactive-messages.md). 

Чаще всего упреждающий обмен сообщениями возникает тогда, когда бот выполняет задачу, которая может занять неопределенное количество времени. В этом случае можно сохранить сведения о задаче, сообщить пользователю что бот вернется к нему после завершения задачи, и позволить продолжить общение. После завершения задачи бот может возобновить общение, отправив упреждающее сообщение с подтверждением.

# <a name="ctabcs"></a>[C#](#tab/cs)

## <a name="notes-about-this-sample"></a>Заметки об этом примере

Изменяется базовый пример EchoBot.
- Как пространство имен используется `Microsoft.Samples.Proactive`.
- Файл состояния заменяется файлом `JobData.cs`.
- Файл бота заменяется файлом `ProactiveBot.cs`.

> [!NOTE]
> В настоящее время упреждающий обмен сообщениями требует, чтобы бот имел допустимый ApplicationID и пароль.


## <a name="define-task-data"></a>Определение данных задачи

В этом случае отслеживаются произвольные задачи, которые могут быть созданы различными пользователями в различных сеансах общения. Таким образом используется ПО промежуточного слоя состояния бота, вместо ПО промежуточного слоя состояния пользователя или общения.

Следующий класс определяет структуру данных, которая будет использоваться для отдельных заданий.


```csharp
using Microsoft.Bot.Schema;
using System.Collections.Generic;

namespace Microsoft.Samples.Proactive
{
    /// <summary>
    /// Class for storing job state. 
    /// </summary>
    public class JobData
    {
        /// <summary>
        /// The name to use to read and write this bot state object to storage.
        /// </summary>
        public readonly static string PropertyName = $"BotState:{typeof(Dictionary<int, JobData>).FullName}";

        public int JobNumber { get; set; } = 0;
        public bool Completed { get; set; } = false;

        /// <summary>
        /// The conversation reference to which to send status updates.
        /// </summary>
        public ConversationReference Conversation { get; set; }
    }
}
```


Необходимо также добавить ПО промежуточного слоя состояния в код запуска.


В файле `StartUp.cs` обновите метод `ConfigureServices` для добавления словаря заданий к состоянию бота. В следующем коде это последний вызов `options.Middleware.Add`.
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<ProactiveBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot. 
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facillitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent. 
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity($"{nameof(ProactiveBot)} Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // Using the base BotState here, since the job log is not necessarily tied to a
        // specific user or conversation.
        options.Middleware.Add(
            new BotState<Dictionary<int, JobData>>(
                dataStore, JobData.PropertyName, (context) => $"jobs/{typeof(Dictionary<int, JobData>)}"));
    });
}
```


## <a name="update-your-bot-to-create-and-run-jobs"></a>Обновление бота для создания и выполнения заданий

Пользователь для каждого включения создает задание, вводя `run` или `run job`.

В ответ бот выполняет следующие действия в рамках этого включения:
- создание задания;
- запись сведений о текущем общении, чтобы иметь возможность отправить упреждающее сообщение позже;
- извещение пользователя о начале его задания и последующее извещение о его завершении;
- начало асинхронного задания;
- окончание включения.

Задание, которое запускается — это простой таймер на 5 секунд, который затем отправляет упреждающее сообщение.
- Вызов метода продолжения общения адаптера создает новое включение, инициированное ботом.
- Эта реплика содержит собственный [контекст](bot-builder-concept-activity-processing.md#turn-context), из которого извлекаются сведения о состоянии.
- Этот контекст используется для отправки упреждающего сообщения пользователю.



> [!NOTE]
> Метод `GetAppId` — это обходной путь для включения упреждающего обмена сообщениями в пакете SDK для .NET.

```csharp
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Connector.Authentication;
using Microsoft.Bot.Schema;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Security.Claims;
using System.Security.Principal;
using System.Threading;
using System.Threading.Tasks;

namespace Microsoft.Samples.Proactive
{
    public class ProactiveBot : IBot
    {
        /// <summary>
        /// Random number generator for job numbers.
        /// </summary>
        private static Random NumberGenerator = new Random();

        /// <summary>
        /// Gets the job log from the bot state.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The job log.</returns>
        private static Dictionary<int, JobData> GetJobLog(ITurnContext context)
        {
            return context.Services.Get<Dictionary<int, JobData>>(JobData.PropertyName);
        }

        /// <summary>
        /// Workaround to get the bot's app ID.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <returns>The application ID for the bot.</returns>
        private static string GetAppId(ITurnContext context)
        {
            // The BotFrameworkAdapter sets the identity provider on the context object.
            var claimsIdentity = context.Services.Get<IIdentity>("BotIdentity") as ClaimsIdentity;

            // For requests from a channel, the app ID is in the Audience claim of the JWT token.
            // For requests from the emulator, it is in the AppId claim.
            // For unauthenticated requests, we have anonymouse identity provided auth is disabled.
            // For Activities coming from Emulator AppId claim contains the Bot's AAD AppId.
            var botAppIdClaim =
                (claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AudienceClaim)
                ?? claimsIdentity.Claims?.SingleOrDefault(claim => claim.Type == AuthenticationConstants.AppIdClaim));

            return botAppIdClaim?.Value;
        }

        /// <summary>
        /// Every Conversation turn calls this method.
        /// When the user types "run" or "run job", the bot starts a "job".
        /// When the job finishes, the bot proactively notifies the user.
        /// </summary>
        /// <param name="context">The turn context.</param>
        /// <remarks>When our virtual job finishes, it sends a proactive message
        /// to notify the user that the job completed.</remarks>
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type is ActivityTypes.Message)
            {
                var text = context.Activity.AsMessageActivity()?.Text?.Trim().ToLower();
                switch (text)
                {
                    case "run":
                    case "run job":

                        var jobLog = GetJobLog(context);
                        var job = CreateJob(context, jobLog);
                        var appId = GetAppId(context);
                        var conversation = TurnContext.GetConversationReference(context.Activity);

                        await context.SendActivity($"We're starting job {job.JobNumber} for you. We'll notify you when it's complete.");

                        // Since the context is disposed at the end of the turn, extract and send the
                        // information we need to send the proactive message later.
                        var adapter = context.Adapter;
                        Task.Run(() =>
                        {
                            // Simulate a separate process to complete the user's job.
                            Thread.Sleep(5000);

                            // Perform bookkeeping and send the proactive message.
                            CompleteJob(adapter, appId, conversation, job.JobNumber);
                        });

                        break;

                    default:

                        await context.SendActivity("Type 'run' or 'run job' to start a new job.");

                        break;
                }
            }
        }

        /// <summary>
        /// Creates a simulated job and updates the job log.
        /// </summary>
        /// <param name="context">The current turn context.</param>
        /// <param name="jobLog">The job log.</param>
        /// <returns>A new job.</returns>
        private JobData CreateJob(ITurnContext context, Dictionary<int, JobData> jobLog)
        {
            // Generate a non-duplicate job number;
            int number;
            while (jobLog.ContainsKey(number = NumberGenerator.Next())) { }

            // Simulate creaing the job and logging it.
            var job = new JobData
            {
                JobNumber = number,
                Conversation = TurnContext.GetConversationReference(context.Activity)
            };
            jobLog.Add(job.JobNumber, job);

            // Return the created job.
            return job;
        }

        /// <summary>
        /// Performs bookkeeping and proactively notifies the user that their job completed.
        /// </summary>
        /// <param name="adapter">The bot adapter with which to send the message.</param>
        /// <param name="appId">The app ID of the bot to send the message from.</param>
        /// <param name="conversation">The conversation in which to put the message.</param>
        /// <param name="jobNumber">The number of the job that completed.</param>
        private async void CompleteJob(BotAdapter adapter, string appId, ConversationReference conversation, int jobNumber)
        {
            await adapter.ContinueConversation(appId, conversation, async context =>
            {
                // Get the job log from state, and retrieve the job.
                var jobLog = GetJobLog(context);
                var job = jobLog[jobNumber];

                // Perform bookkeeping.
                job.Completed = true;

                // Send the user a proactive confirmation message.
                await context.SendActivity($"Job {job.JobNumber} is complete.");
            });
        }
    }
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Прежде чем отправить упреждающее сообщение пользователю, этому пользователю придется отправить боту хотя бы одно сообщение реактивного стиля. 

Необходимо отправить одно сообщение боту, так как он должен получить ссылку на объект действия и сохранить его для будущего использования. Объект действия можно рассматривать как адрес пользователя, поскольку он содержит сведения о канале, через который тот вошел, идентификатор пользователя, идентификатор общения и даже сервер, который должен получать все будущие сообщения. Этот простой объект JSON и должен сохраняться целиком без изменения.

Начнем с небольшого фрагмента кода, показывающего, как сохранить ссылку на общение каждый раз, когда пользователь решает подписаться.
```javascript
const { MemoryStorage } = require('botbuilder');

const storage = new MemoryStorage();

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const utterances = (context.activity.text || '').trim().toLowerCase()
            if (utterances === 'subscribe') {
                var userId = await saveReference(TurnContext.getConversationReference(context.activity));
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe' to start proactive message");
            }
    
        }
    });
});
```
Фрагменты выше вызывают функцию `saveReference()`, которая сохраняет ссылку на пользователя с помощью `MemoryStorage` и возвращает `userId`. После успешного сохранения ссылки вызывается метод `subscribeUser()`, который уведомляет пользователя, что он подписался. 

Именно функция `subscribeUser()` настраивает подписку. Рассмотрим простую реализацию, которая запускает 2-секундный таймер и отправляет пользователю упреждающее сообщение по его истечению.

```javascript
// Persist info to storage
async function saveReference(reference){
    const userId = reference.activityId
    const changes = {};
    changes['reference/' + userId] = reference;
    await storage.write(changes); // Write reference info to persisted storage
    return userId;
}

// Subscribe user to a proactive call. In this case, we are using a setTimeOut() to trigger the proactive call
async function subscribeUser(userId) {
    setTimeout(async () => {
        const reference = await findReference(userId);
        if (reference) {
            await adapter.continueConversation(reference, async (context) => {
                await context.sendActivity("You have been notified");
            });
            
        }
    }, 2000); // Trigger after 2 secs
}

// Read the stored reference info from storage
async function findReference(userId){
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]

    return reference;
}
```

Функция `subscribeUser()` устанавливает таймер, который найдет необходимый объект, считав его из хранилища. Если необходимый объект найден, общение с пользователем продолжается. Метод `continueConversation` позволяет боту отправлять упреждающие сообщения в общение или пользователю, с которым бот уже связывался.

---

## <a name="test-your-bot"></a>Тестирование бота

Чтобы протестировать бота, разверните его в Azure как бота регистрации и протестируйте его в Интернете или локально с помощью эмулятора.

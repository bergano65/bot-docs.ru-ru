---
title: Краткий справочник по миграции из версии 3 в версию 4 для .NET | Документация Майкрософт
description: Описание основных различий между пакетами SDK Bot Framework для .NET версий 3 и 4.
keywords: .net, bot migration, dialogs, v3 bot
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 93f660820407d6caf4c29efb3c128851059f3b02
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215576"
---
# <a name="net-migration-quick-reference"></a>Краткий справочник по миграции для .NET

В пакете SDK Bot Builder для .NET версии 4 реализовано несколько важных изменений, связанных с процессом создания ботов. В этом руководстве описаны различия между способами выполнения задач в пакетах SDK версий 3 и 4.

- Способ передачи данных между каналами и ботом изменился. В версии 3 использовался объект _беседы_ и метод _SendAsync_ для обработки сообщения, а для загрузки разных зависимостей широко использовался Autofac. В версии 4 используются объекты _адаптера_ и _TurnContext_ для обработки сообщения, а для внедрения зависимостей можно использовать любую библиотеку.

- Кроме того, разграничены экземпляры диалогов и бота. В версии 3 диалоги встраивались в базовый пакет SDK, стек обрабатывался внутренними средствами, а дочерние диалоги загружались с использованием методов _вызова_ и _перенаправления_. В версии 4 диалоги передаются в экземпляры бота как аргументы, делая процесс создания более гибким. При этом разработчик контролирует стек диалогов, а дочерние диалоги загружаются с использованием методов _BeginDialogAsync_ и _ReplaceDialogAsync_.

- Кроме того, в версии 4 реализован класс `ActivityHandler` для автоматизации обработки разных типов действий, таких как _сообщение_, _обновление беседы_ и _событие_.

В результате изменен синтаксис для разработки ботов на .NET. В частности, это касается создания объектов бота, определения диалогов и создания логики обработки событий.

Ниже приведено сравнение конструкций в пакетах SDK Bot Framework для .NET версий 3 и 4.

## <a name="to-process-incoming-messages"></a>Обработка входящих сообщений

### <a name="v3"></a>Версия 3

```cs
[BotAuthentication]
public class MessagesController : ApiController
{
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {
        if (activity.GetActivityType() == ActivityTypes.Message)
        {
            await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
        }

        return Request.CreateResponse(HttpStatusCode.OK);
    }
}
```

### <a name="v4"></a>версия 4

```cs
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter Adapter;
    private readonly IBot Bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        Adapter = adapter;
        Bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        await Adapter.ProcessAsync(Request, Response, Bot);
    }
}
```

## <a name="to-send-a-message-to-a-user"></a>Отправка сообщения пользователю

### <a name="v3"></a>Версия 3

```cs
await context.PostAsync("Hello and welcome to the help desk bot.");
```

### <a name="v4"></a>версия 4

```cs
await turnContext.SendActivityAsync("Hello and welcome to the help desk bot.");
```

## <a name="to-load-a-root-dialog"></a>Загрузка корневого диалога

### <a name="v3"></a>Версия 3

```cs
await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
```

### <a name="v4"></a>версия 4

```cs
// Create a DialogExtensions class with a Run method.
public static class DialogExtensions
{
    public static async Task Run(
        this Dialog dialog,
        ITurnContext turnContext,
        IStatePropertyAccessor<DialogState> accessor,
        CancellationToken cancellationToken)
    {
        var dialogSet = new DialogSet(accessor);
        dialogSet.Add(dialog);

        var dialogContext = await dialogSet.CreateContextAsync(turnContext, cancellationToken);

        var results = await dialogContext.ContinueDialogAsync(cancellationToken);
        if (results.Status == DialogTurnStatus.Empty)
        {
            await dialogContext.BeginDialogAsync(dialog.Id, null, cancellationToken);
        }
    }
}

// Call it from the ActivityHandler's OnMessageActivityAsync override
protected override async Task OnMessageActivityAsync(
    ITurnContext<IMessageActivity> turnContext,
    CancellationToken cancellationToken)
{
    // Run the Dialog with the new message Activity.
    await Dialog.Run(
        turnContext,
        ConversationState.CreateProperty<DialogState>("DialogState"),
        cancellationToken);
}
```

## <a name="to-start-a-child-dialog"></a>Запуск дочернего диалога

### <a name="v3"></a>Версия 3

```cs
context.Call(new NextDialog(), this.ResumeAfterNextDialog);
```

или

```cs
await context.Forward(new NextDialog(), this.ResumeAfterNextDialog, message);
```

### <a name="v4"></a>версия 4

```cs
dialogContext.BeginDialogAsync("<child-dialog-id>", options);
```

или

```cs
dialogContext.ReplaceDialogAsync("<child-dialog-id>", options);
```

## <a name="to-end-a-dialog"></a>Завершение диалога

### <a name="v3"></a>Версия 3

```cs
context.Done(ReturnValue);
```

### <a name="v4"></a>версия 4

```cs
await context.EndDialogAsync(ReturnValue);
```

## <a name="to-prompt-a-user-for-input"></a>Запрос пользователю на ввод данных

### <a name="v3"></a>Версия 3

```cs
PromptDialog.Choice(
    context,
    this.OnOptionSelected,
    Options, PromptMessage,
    ErrorMessage,
    3,
    PromptStyle.PerLine);
```

### <a name="v4"></a>версия 4

```cs
// In the dialog's constructor, register the prompt, and waterfall steps.
AddDialog(new TextPrompt(nameof(TextPrompt)));
AddDialog(new WaterfallDialog(nameof(WaterfallDialog), new WaterfallStep[]
{
    FirstStepAsync,
    SecondStepAsync,
}));

// The initial child Dialog to run.
InitialDialogId = nameof(WaterfallDialog);

// ...

// In the first step, invoke the prompt.
private async Task<DialogTurnResult> FirstStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    return await stepContext.PromptAsync(
        nameof(TextPrompt),
        new PromptOptions { Prompt = MessageFactory.Text("Please enter your destination.") },
        cancellationToken);
}

// In the second step, retrieve the Result from the stepContext.
private async Task<DialogTurnResult> SecondStepAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    var destination = (string)stepContext.Result;
}
```

## <a name="to-save-information-to-dialog-state"></a>Сохранение информации в состоянии диалога

### <a name="v3"></a>Версия 3

В версии 3 все диалоги и их поля автоматически сериализовались.

### <a name="v4"></a>версия 4

```cs
// StepContext values are auto-serialized in V4, and scoped to the dialog.
stepContext.values.destination = destination;
```

## <a name="to-write-changes-in-state-to-the-persistance-layer"></a>Запись изменений в состоянии на уровне сохраняемости

### <a name="v3"></a>Версия 3

Данные о состоянии автоматически сохраняются по умолчанию в конце шага.

### <a name="v4"></a>версия 4

```cs
// You now must explicitly save state changes before the end of the turn.
await this.conversationState.saveChanges(context, false);
await this.userState.saveChanges(context, false);
```

## <a name="to-create-and-register-state-storage"></a>Создание и регистрация хранилища состояний

### <a name="v3"></a>Версия 3

```cs
// Autofac was used internally by the sdk, and state was automatic.
Conversation.UpdateContainer(
builder =>
{
    builder.RegisterModule(new AzureModule(Assembly.GetExecutingAssembly()));
    var store = new InMemoryDataStore();
    builder.Register(c => store)
        .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
        .AsSelf()
        .SingleInstance();
});
```

### <a name="v4"></a>версия 4

```cs
// Create the storage we'll be using for User and Conversation state.
// In-memory storage is great for testing purposes.
services.AddSingleton<IStorage, MemoryStorage>();

// Create the user state (used in this bot's Dialog implementation).
services.AddSingleton<UserState>();

// Create the conversation state (used by the Dialog system itself).
services.AddSingleton<ConversationState>();

// The dialog that will be run by the bot.
services.AddSingleton<MainDialog>();

// Create the bot as a transient. In this case the ASP.NET controller is expecting an IBot.
services.AddTransient<IBot, DialogBot>();

// In the bot's ActivityHandler implementation, call SaveChangesAsync after the OnTurnAsync completes.
public class DialogBot : ActivityHandler
{
    protected readonly Dialog Dialog;
    protected readonly BotState ConversationState;
    protected readonly BotState UserState;

    public DialogBot(ConversationState conversationState, UserState userState, Dialog dialog)
    {
        ConversationState = conversationState;
        UserState = userState;
    }

    public override async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        await base.OnTurnAsync(turnContext, cancellationToken);

        // Save any state changes that might have ocurred during the turn.
        await ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
        await UserState.SaveChangesAsync(turnContext, false, cancellationToken);
    }

    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        // Run the dialog, passing in the message activity for this turn.
        await Dialog.Run(turnContext, ConversationState.CreateProperty<DialogState>("DialogState"), cancellationToken);
    }
}
```

## <a name="to-catch-an-error-thrown-from-a-dialog"></a>Перехват ошибки, связанной с диалогом

### <a name="v3"></a>Версия 3

```cs
// Create a custom IPostToBot implementation to catch exceptions.
public sealed class CustomPostUnhandledExceptionToUser : IPostToBot
{
    private readonly IPostToBot inner;
    private readonly IBotToUser botToUser;
    private readonly ResourceManager resources;
    private readonly System.Diagnostics.TraceListener trace;

    public CustomPostUnhandledExceptionToUser(IPostToBot inner, IBotToUser botToUser, ResourceManager resources, System.Diagnostics.TraceListener trace)
    {
        SetField.NotNull(out this.inner, nameof(inner), inner);
        SetField.NotNull(out this.botToUser, nameof(botToUser), botToUser);
        SetField.NotNull(out this.resources, nameof(resources), resources);
        SetField.NotNull(out this.trace, nameof(trace), trace);
    }

    async Task IPostToBot.PostAsync(IActivity activity, CancellationToken token)
    {
        try
        {
            await this.inner.PostAsync(activity, token);
        }
        catch (Exception ex)
        {
            try
            {
                // Log exception and send custom error message here.
                await this.botToUser.PostAsync("custom error message");
            }
            catch (Exception inner)
            {
                this.trace.WriteLine(inner);
            }

            throw;
        }
    }
}

// Register this using AutoFac, replacing the default PostUnhandledExceptionToUser.
builder
  .RegisterType<CustomPostUnhandledExceptionToUser>()
  .Keyed<IPostToBot>(typeof(PostUnhandledExceptionToUser));
```

### <a name="v4"></a>версия 4

```cs
// Provide an error handler in your implementation of the BotFrameworkHttpAdapter.
public class AdapterWithErrorHandler : BotFrameworkHttpAdapter
{
    public AdapterWithErrorHandler(
        ICredentialProvider credentialProvider,
        ILogger<BotFrameworkHttpAdapter> logger,
        ConversationState conversationState = null)
        : base(credentialProvider)
    {
        OnTurnError = async (turnContext, exception) =>
        {
            // Log any leaked exception from the application.
            logger.LogError($"Exception caught : {exception.Message}");

            // Send a catch-all apology to the user.
            await turnContext.SendActivityAsync("Sorry, it looks like something went wrong.");

            if (conversationState != null)
            {
                try
                {
                    // Delete the conversation state for the current conversation, to prevent the
                    // bot from getting stuck in a error-loop caused by being in a bad state.
                    // Conversation state is similar to "cookie-state" in a web page.
                    await conversationState.DeleteAsync(turnContext);
                }
                catch (Exception e)
                {
                    logger.LogError(
                        $"Exception caught on attempting to Delete ConversationState : {e.Message}");
                }
            }
        };
    }
}
```

## <a name="to-process-different-activity-types"></a>Обработка других типов действий

### <a name="v3"></a>Версия 3

```cs
// Within your MessageController, check the message type.
string messageType = activity.GetActivityType();
if (messageType == ActivityTypes.Message)
{
    await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
}
else if (messageType == ActivityTypes.DeleteUserData)
{
}
else if (messageType == ActivityTypes.ConversationUpdate)
{
}
else if (messageType == ActivityTypes.ContactRelationUpdate)
{
}
else if (messageType == ActivityTypes.Typing)
{
}
```

### <a name="v4"></a>версия 4

```cs
// In the bot's ActivityHandler implementation, override relevant methods.

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle message activities here.
}

protected override Task OnConversationUpdateActivityAsync(ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle conversation update activities in general here.
}

protected override Task OnEventActivityAsync(ITurnContext<IEventActivity> turnContext, CancellationToken cancellationToken)
{
    // Handle event activities in general here.
}
```
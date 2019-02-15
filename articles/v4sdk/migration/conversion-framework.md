---
title: Перенос существующего бота в пределах проекта .NET Framework | Документация Майкрософт
description: Перенос существующего бота версии 3 в пакет SDK версии 4 в том же проекте.
keywords: bot migration, formflow, dialogs, v3 bot
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/11/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1904bb09d8bd387cc5cec0d85f82df24d1f6ec9d
ms.sourcegitcommit: 7f418bed4d0d8d398f824e951ac464c7c82b8c3e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/14/2019
ms.locfileid: "56240180"
---
# <a name="migrate-a-net-sdk-v3-bot-to-v4"></a>Перенос бота с версии 3 в версию 4 пакета SDK для .NET

В этой статье описывается, как преобразовать бот [ContosoHelpdeskChatBot](https://github.com/Microsoft/intelligent-apps/tree/master/ContosoHelpdeskChatBot/ContosoHelpdeskChatBot) версии 3 в бот версии 4, _не меняя тип проекта_. Это по-прежнему будет проект .NET Framework.
Преобразование включает в себя следующие действия:

1. Обновление и установка пакетов NuGet.
1. Обновление файла global.asax.cs.
1. Обновление класса MessagesController.
1. Преобразование диалогов.

<!--TODO: Link to the converted bot...[ContosoHelpdeskChatBot](https://github.com/EricDahlvang/intelligent-apps/tree/v4netframework/ContosoHelpdeskChatBot).-->

Пакет SDK версии 4 для Bot Framework использует тот же базовый REST API, что и пакет SDK версии 3. Но в пакете SDK версии 4 выполнен рефакторинг кода предыдущей версии пакета SDK для повышения гибкости и улучшения контроля над ботами. Основные изменения в пакете SDK:

- Управление состоянием осуществляется через соответствующие объекты и методы доступа к свойствам.
- Также изменились процессы настройки обработчика шагов и передачи ему действий.
- Диалоги с возможностью оценки Scoreable больше не существуют. Вы можете проверить наличие глобальных команд в обработчике шагов, прежде чем передавать средства управления в диалоги.
- Новая библиотека Dialogs, которая сильно отличается от предыдущей версии. Вам потребуется преобразовать все старые диалоги в новую систему, используя компонентные, каскадные диалоги и созданную сообществом реализацию диалогов Formflow для версии 4.

См. дополнительные сведения об [отличиях между версиями 3 и 4 пакета SDK для .NET](migration-about.md).

## <a name="update-and-install-nuget-packages"></a>Обновление и установка пакетов NuGet

1. Обновите **Microsoft.Bot.Builder.Azure** до последней стабильной версии.

    Так вы обновите пакеты **Microsoft.Bot.Builder** и **Microsoft.Bot.Connector**, которые указаны как зависимости.

1. Удалите пакет **Microsoft.Bot.Builder.History**. Он не входит в состав пакета SDK версии 4.
1. Добавьте **Autofac.WebApi2**.

    Мы будем его использовать для внедрения зависимостей в ASP.NET.

1. Добавьте **Bot.Builder.Community.Dialogs.Formflow**.

    Это библиотека сообщества для сборки диалогов версии 4 из файлов определения Formflow версии 3. Одно из зависимостей для нее является **Microsoft.Bot.Builder.Dialogs**, поэтому она тоже автоматически устанавливается.

Если на этом этапе выполнить сборку, вы получите ошибки компилятора. На них можно не обращать внимания. Когда преобразование полностью завершится, код будет полностью рабочим.

<!--
## Add a BotDataBag class

This file will contain wrappers to add a v3-style **IBotDataBag** to make dialog conversion simpler.

```csharp
using System.Collections.Generic;

namespace ContosoHelpdeskChatBot
{
    public class BotDataBag : Dictionary<string, object>, IBotDataBag
    {
        public bool RemoveValue(string key)
        {
            return base.Remove(key);
        }

        public void SetValue<T>(string key, T value)
        {
            this[key] = value;
        }

        public bool TryGetValue<T>(string key, out T value)
        {
            if (!ContainsKey(key))
            {
                value = default(T);
                return false;
            }

            value = (T)this[key];

            return true;
        }
    }

    public interface IBotDataBag
    {
        int Count { get; }

        void Clear();

        bool ContainsKey(string key);

        bool RemoveValue(string key);

        void SetValue<T>(string key, T value);

        bool TryGetValue<T>(string key, out T value);
    }
}
```
-->

## <a name="update-your-globalasaxcs-file"></a>Обновление файла global.asax.cs

Есть некоторые изменения в формировании шаблонов, и теперь для версии 4 нужно самостоятельно настроить элементы инфраструктуры [управления состоянием](/articles/v4sdk/bot-builder-concept-state.md). К примеру, в версии 4 применяется адаптер ботов для проверки подлинности и передачи действий в код бота, а это означает, что все свойства состояния нужно объявить заранее.

Мы создадим свойство состояния для `DialogState`, который потребуется в версии 4 для поддержки диалогов. Мы будем использовать внедрение зависимостей, чтобы получить информацию, которая нужна контроллеру и боту.

В файле **Global.asax.cs** выполните следующее:

1. Обновите инструкции `using`.
    ```csharp
    using System.Configuration;
    using System.Reflection;
    using System.Web.Http;
    using Autofac;
    using Autofac.Integration.WebApi;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    ```
1. Удалите следующие строки из метода **Application_Start**.
    ```csharp
    BotConfig.UpdateConversationContainer();
    this.RegisterBotModules();
    ```
    Теперь вставьте эту строку.
    ```csharp
    GlobalConfiguration.Configure(BotConfig.Register);
    ```
1. Удалите метод **RegisterBotModules**, на который уже нет ссылок.
1. Замените метод **BotConfig.UpdateConversationContainer** представленным здесь методом **BotConfig.Register**, в котором мы регистрируем объекты, необходимые для поддержки внедрения зависимостей.
    > [!NOTE]
    > Наш пример бота не использует состояния _пользователя_ или _закрытого диалога_. Поэтому здесь закомментированы строки кода, в которых они включаются.
    ```csharp
    public static void Register(HttpConfiguration config)
    {
        ContainerBuilder builder = new ContainerBuilder();
        builder.RegisterApiControllers(Assembly.GetExecutingAssembly());

        SimpleCredentialProvider credentialProvider = new SimpleCredentialProvider(
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppIdKey],
            ConfigurationManager.AppSettings[MicrosoftAppCredentials.MicrosoftAppPasswordKey]);

        builder.RegisterInstance(credentialProvider).As<ICredentialProvider>();

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        // Create Conversation State object.
        // The Conversation State object is where we persist anything at the conversation-scope.
        ConversationState conversationState = new ConversationState(dataStore);
        builder.RegisterInstance(conversationState).As<ConversationState>();

        //var userState = new UserState(dataStore);
        //var privateConversationState = new PrivateConversationState(dataStore);

        // Create the dialog state property acccessor.
        IStatePropertyAccessor<DialogState> dialogStateAccessor
            = conversationState.CreateProperty<DialogState>(nameof(DialogState));
        builder.RegisterInstance(dialogStateAccessor).As<IStatePropertyAccessor<DialogState>>();

        IContainer container = builder.Build();
        AutofacWebApiDependencyResolver resolver = new AutofacWebApiDependencyResolver(container);
        config.DependencyResolver = resolver;
    }
    ```

## <a name="update-your-messagescontroller-class"></a>Обновление класса MessagesController

Так как именно здесь в версии 4 выполняется обработчик шагов, нам нужно внести значительные изменения. Все остальное, кроме обработчика шагов, можно считать стандартным кодом. В файле **Controllers\MessagesController.cs** сделайте следующее:

1. Обновите инструкции `using`.
    ```csharp
    using System;
    using System.Net;
    using System.Net.Http;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Web.Http;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Dialogs;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    ```
1. Удалите из класса атрибут `[BotAuthentication]`. В версии 4 проверку подлинности выполняет адаптер бота.
1. Добавьте следующие поля. **ConversationState** будет управлять состоянием, относящимся к беседе, а **IStatePropertyAccessor\<DialogState>** требуется для поддержки диалогов в версии 4.
    ```csharp
    private readonly ConversationState _conversationState;
    private readonly ICredentialProvider _credentialProvider;
    private readonly IStatePropertyAccessor<DialogState> _dialogData;

    private readonly DialogSet _dialogs;
    ```
1. Добавьте конструктор, чтобы:
    - Инициализировать поля экземпляра.
    - Применить внедрение зависимостей в ASP.NET для получения значений параметров. (Чтобы поддерживать эту возможность, мы зарегистрировали экземпляры этих классов в **Global.asax.cs**.)
    - Создать и инициализировать набор диалогов, из которого можно создать контекст диалога. (В версии 4 это нужно делать явным образом.)
    ```csharp
    public MessagesController(
        ConversationState conversationState,
        ICredentialProvider credentialProvider,
        IStatePropertyAccessor<DialogState> dialogData)
    {
        _conversationState = conversationState;
        _dialogData = dialogData;
        _credentialProvider = credentialProvider;

        _dialogs = new DialogSet(dialogData);
        _dialogs.Add(new RootDialog(nameof(RootDialog)));
    }
    ```
1. Замените текст метода **Post**. Здесь мы создаем адаптер и применяем его для вызова цикла обработки сообщений (обработчика шагов). В конце каждого шага мы применяем `SaveChangesAsync`, чтобы сохранить любые изменения, внесенные ботом в состояние.

    ```csharp
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {

        var botFrameworkAdapter = new BotFrameworkAdapter(_credentialProvider);

        var invokeResponse = await botFrameworkAdapter.ProcessActivityAsync(
            Request.Headers.Authorization?.ToString(),
            activity,
            OnTurnAsync,
            default(CancellationToken));

        if (invokeResponse == null)
        {
            return Request.CreateResponse(HttpStatusCode.OK);
        }
        else
        {
            return Request.CreateResponse(invokeResponse.Status);
        }
    }
    ```
1. Добавьте метод **OnTurnAsync**, который содержит код для [обработчика шагов](/articles/v4sdk/bot-builder-basics.md#the-activity-processing-stack) бота.
    > [!NOTE]
    > Диалоги с возможностью оценки в версии 4 не существуют. Мы проверяем сообщение `cancel` от пользователя в обработчике шагов бота, прежде чем продолжать активный диалог.
    ```csharp
    protected async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken)
    {
        // We're only handling message activities in this bot.
        if (turnContext.Activity.Type == ActivityTypes.Message)
        {
            // Create the dialog context for our dialog set.
            DialogContext dc = await _dialogs.CreateContextAsync(turnContext, cancellationToken);

            // Globally interrupt the dialog stack if the user sent 'cancel'.
            if (turnContext.Activity.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
            {
                Activity reply = turnContext.Activity.CreateReply($"Ok restarting conversation.");
                await turnContext.SendActivityAsync(reply);
                await dc.CancelAllDialogsAsync();
            }

            try
            {
                // Continue the active dialog, if any. If we just cancelled all dialog, the
                // dialog stack will be empty, and this will return DialogTurnResult.Empty.
                DialogTurnResult dialogResult = await dc.ContinueDialogAsync();
                switch (dialogResult.Status)
                {
                    case DialogTurnStatus.Empty:
                        // There was no active dialog in the dialog stack; start the root dialog.
                        await dc.BeginDialogAsync(nameof(RootDialog));
                        break;

                    case DialogTurnStatus.Complete:
                        // The last dialog on the stack completed and the stack is empty.
                        await dc.EndDialogAsync();
                        break;

                    case DialogTurnStatus.Waiting:
                    case DialogTurnStatus.Cancelled:
                        // The active dialog is waiting for a response from the user, or all
                        // dialogs were cancelled and the stack is empty. In either case, we
                        // don't need to do anything here.
                        break;
                }
            }
            catch (FormCanceledException)
            {
                // One of the dialogs threw an exception to clear the dialog stack.
                await turnContext.SendActivityAsync("Cancelled.");
                await dc.CancelAllDialogsAsync();
                await dc.BeginDialogAsync(nameof(RootDialog));
            }
        }
    }
    ```
1. Так как мы обрабатываем только действия для _сообщений_, метод **HandleSystemMessage** можно смело удалять.

### <a name="delete-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>Удаление классов CancelScorable и GlobalMessageHandlersBotModule

Так как диалоги с возможностью оценки в версии 4 не существуют, а обработчик шагов теперь реагирует на сообщение `cancel`, можно спокойно удалить классы **CancelScorable** (в **Dialogs\CancelScorable.cs**) и  **GlobalMessageHandlersBotModule**.

## <a name="convert-your-dialogs"></a>Преобразование диалогов

Мы будем вносить в исходные диалоги изменения, позволяющие использовать их с пакетом SDK версии 4. Пока не беспокойтесь об ошибках компилятора. Они исчезнут, когда преобразование завершится.
Мы не будем изменять исходный код больше, чем необходимо, поэтому некоторые предупреждения компилятора сохранятся после завершения переноса.

Все диалоги теперь будут не реализацией интерфейса `IDialog<object>` из версии 3, а станут производными от `ComponentDialog`.

Этот бот использует четыре диалога, и все их нужно преобразовать:

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | Представляет варианты на выбор и запускает другие диалоги. |
| [InstallAppDialog](#update-the-install-app-dialog) | Обрабатывает запросы на установку приложения на компьютер. |
| [LocalAdminDialog](#update-the-local-admin-dialog) | Обрабатывает запросы на получение прав локального администратора на компьютере. |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | Обрабатывает запросы на сброс пароля. |

Все они собирают данные, но не выполняют никаких операций на компьютере.

### <a name="make-solution-wide-dialog-changes"></a>Внесение изменений в диалоги на уровне решения

1. Для всего решения измените все вхождения `IDialog<object>` на `ComponentDialog`.
1. Для всего решения измените все вхождения `IDialogContext` на `DialogContext`.
1. Для каждого класса диалогов удалите атрибут `[Serializable]`.

Поток управления и обмен сообщениями в диалогах теперь обрабатываются другими способами, и это нужно учитывать при преобразовании каждого диалога.

| Операция | Код для версии 3 | Код для версии 4 |
| :--- | :--- | :--- |
| Обработка запуска диалога | Реализуйте `IDialog.StartAsync` | Сделайте этот шаг первым в каскадном диалоге или реализуйте `Dialog.BeginDialogAsync` |
| Обработка продолжения диалога | Вызовите `IDialogContext.Wait` | Добавьте в каскадный диалог дополнительные шаги или реализуйте `Dialog.ContinueDialogAsync` |
| Отправка сообщения пользователю | Вызовите `IDialogContext.PostAsync` | Вызовите `ITurnContext.SendActivityAsync` |
| Запуск дочернего диалога | Вызовите `IDialogContext.Call` | Вызовите `DialogContext.BeginDialogAsync` |
| Информирование о завершении текущего диалога | Вызовите `IDialogContext.Done` | Вызовите `DialogContext.EndDialogAsync` |
| Получение введенных пользователем данных | Используйте параметр `IAwaitable<IMessageActivity>` | Используйте приглашение в каскадном диалоге или `ITurnContext.Activity` |

Примечания о коде для версии 4:

- Используйте свойство `DialogContext.Context`, чтобы получить текущий контекст шага.
- Каскадные шаги имеют параметр `WaterfallStepContext`, который является производным от `DialogContext`.
- Все конкретные классы диалогов и приглашений наследуют от абстрактного класса `Dialog`.
- Идентификатор присваивается при создании компонентного диалога. Каждый диалог в наборе диалогов должен иметь уникальный в пределах этого набора идентификатор.

### <a name="update-the-root-dialog"></a>Обновление корневого диалога

В этом боте корневой диалог предлагает пользователю выбрать один вариант из набора, а затем на основе этого выбора запускает дочерний диалог. Этот цикл повторяется неограниченно долго, пока существует беседа.

- Основной поток можно создать в формате каскадного диалога — это новая концепция в пакете SDK версии 4. Он выполняет фиксированный набор шагов в заданном порядке. Подробнее см. статью [Реализация последовательной беседы](/articles/v4sdk/bot-builder-dialog-manage-conversation-flow).
- Запросы теперь обрабатываются через классы prompt. Эти короткие дочерние диалоги предлагают ввести данные, выполняют минимальную обработку и проверку этих данных и возвращают полученное значение. Дополнительные сведения о сборе данных от пользователя с помощью запросов диалога см. в [этой статье](/articles/v4sdk/bot-builder-prompts.md).

В файле **Dialogs/RootDialog.cs** сделайте следующее:

1. Обновите инструкции `using`.
    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. Нам нужно преобразовать варианты `HelpdeskOptions` из списка строк в список вариантов. Они будут применяться в строке запроса, которая в качестве допустимых входных данных принимает номер выбранного варианта в списке, значение любого из вариантов или его синоним.
    ```csharp
    private static List<Choice> HelpdeskOptions = new List<Choice>()
    {
        new Choice(InstallAppOption) { Synonyms = new List<string>(){ "install" } },
        new Choice(ResetPasswordOption) { Synonyms = new List<string>(){ "password" } },
        new Choice(LocalAdminOption)  { Synonyms = new List<string>(){ "admin" } }
    };
    ```
1. Добавьте конструктор. Этот код делает следующее:
   - Каждому экземпляру диалога при создании присваивается идентификатор. Идентификатор диалога входит в набор диалогов, к которому добавляется этот диалог. Не забывайте, что для бота уже инициализирован набор диалогов в классе **MessageController**. Для всех `ComponentDialog` есть свой внутренний набор диалогов с отдельным набором идентификаторов диалогов.
   - Включает остальные диалоги, в том числе запросы выбора, в качестве дочерних диалогов. Здесь мы используем для каждого диалога в качестве идентификатора соответствующее имя класса.
   - Определяет каскадный диалог с тремя шагами. Их мы реализуем чуть позже.
     - Сначала диалог предлагает пользователю выбрать задание для выполнения.
     - Затем он запускает дочерний диалог, связанный с выбранным вариантом.
     - И уже потом он перезапускает сам себя.
   - Каждый шаг каскадного диалога является делегатом. Мы реализуем их далее, по возможности сохраняя существующий код каждого исходного диалога.
   - При запуске компонентного диалога будет запущен _начальный диалог_. По умолчанию это первый дочерний диалог, добавленный в компонентный диалог. Чтобы назначить другой дочерний диалог в качестве начального, следует вручную указать для компонентного диалога свойство `InitialDialogId`.
    ```csharp
    public RootDialog(string id)
        : base(id)
    {
        AddDialog(new WaterfallDialog("choiceswaterfall", new WaterfallStep[]
            {
                PromptForOptionsAsync,
                ShowChildDialogAsync,
                ResumeAfterAsync,
            }));
        AddDialog(new InstallAppDialog(nameof(InstallAppDialog)));
        AddDialog(new LocalAdminDialog(nameof(LocalAdminDialog)));
        AddDialog(new ResetPasswordDialog(nameof(ResetPasswordDialog)));
        AddDialog(new ChoicePrompt("options"));
    }
    ```
1. Метод **StartAsync** можно удалить. При запуске компонентный диалог автоматически выполняет свой _начальный_ диалог. В нашем примере это тот каскадный диалог, который мы определяем в конструкторе. Он автоматически начинает выполнение с первого действия.
1. Теперь мы удалим методы **MessageReceivedAsync** и **ShowOptions** и заменим их первым шагом каскадного диалога. Эти два метода ранее приветствовали пользователя и предлагали выбор из доступных вариантов.
   - Здесь вы видите список значений, приветствия и сообщения об ошибках, которые передаются в вызов запроса на выбор в качестве параметров.
   - Нам не нужно указывать следующий метод, который будет вызван в диалоге, так как каскадный диалог автоматически перейдет к следующему шагу после завершения запроса на выбор.
   - Запрос выбора повторяется в цикле, пока не получит допустимые входные данные или весь стек диалогов не будет отменен.
    ```csharp
    private async Task<DialogTurnResult> PromptForOptionsAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Prompt the user for a response using our choice prompt.
        return await stepContext.PromptAsync(
            "options",
            new PromptOptions()
            {
                Choices = HelpdeskOptions,
                Prompt = MessageFactory.Text(GreetMessage),
                RetryPrompt = MessageFactory.Text(ErrorMessage)
            },
            cancellationToken);
    }
    ```
1. Мы можем заменить **OnOptionSelected** вторым шагом каскадного диалога. Здесь мы также запускаем дочерний диалог в зависимости от ввода пользователя.
   - Запрос выбора возвращает значение `FoundChoice`. Оно передается в свойстве `Result` этого шага. Стек диалогов обрабатывает все возвращаемые значения как объекты. Если это значение возвращается из вашего диалога, вы получите тип значения для этого объекта. Список значений, возвращаемых разными типами, вы можете изучить [здесь](/articles/v4sdk/bot-builder-concept-dialog.md#prompt-types).
   - Так как запрос выбора не создает исключений, мы можем удалить блок try-catch.
   - Но нам нужно добавить механизм передачи управления, чтобы этот метод всегда возвращал допустимое значение. Этот код никогда не должен выполняться, но если это произойдет из-за сбоя, он корректно завершит работу диалога.
    ```csharp
    private async Task<DialogTurnResult> ShowChildDialogAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // string optionSelected = await userReply;
        string optionSelected = (stepContext.Result as FoundChoice).Value;

        switch (optionSelected)
        {
            case InstallAppOption:
                //context.Call(new InstallAppDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(InstallAppDialog),
                    cancellationToken);
            case ResetPasswordOption:
                //context.Call(new ResetPasswordDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(ResetPasswordDialog),
                    cancellationToken);
            case LocalAdminOption:
                //context.Call(new LocalAdminDialog(), this.ResumeAfterOptionDialog);
                //break;
                return await stepContext.BeginDialogAsync(
                    nameof(LocalAdminDialog),
                    cancellationToken);
        }

        // We shouldn't get here, but fail gracefully if we do.
        await stepContext.Context.SendActivityAsync(
            "I don't recognize that option.",
            cancellationToken: cancellationToken);
        // Continue through to the next step without starting a child dialog.
        return await stepContext.NextAsync(cancellationToken: cancellationToken);
    }
    ```
1. И наконец, замените старый метод **ResumeAfterOptionDialog** последним шагом каскадного диалога.
    - В исходном диалоге мы завершали работу и возвращали номер билета. Теперь же мы перезапускаем каскадный диалог, заменяя его старый экземпляр в стеке диалогов новым экземпляром того же диалога. Это стало возможным, так как исходное приложение всегда игнорирует возвращаемое значение (номер билета) и перезапускает корневой диалог.
    ```csharp
    private async Task<DialogTurnResult> ResumeAfterAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        try
        {
            //var message = await userReply;
            var message = stepContext.Context.Activity;

            int ticketNumber = new Random().Next(0, 20000);
            //await context.PostAsync($"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.");
            await stepContext.Context.SendActivityAsync(
                $"Thank you for using the Helpdesk Bot. Your ticket number is {ticketNumber}.",
                cancellationToken: cancellationToken);

            //context.Done(ticketNumber);
        }
        catch (Exception ex)
        {
            // await context.PostAsync($"Failed with message: {ex.Message}");
            await stepContext.Context.SendActivityAsync(
                $"Failed with message: {ex.Message}",
                cancellationToken: cancellationToken);

            // In general resume from task after calling a child dialog is a good place to handle exceptions
            // try catch will capture exceptions from the bot framework awaitable object which is essentially "userReply"
            logger.Error(ex);
        }

        // Replace on the stack the current instance of the waterfall with a new instance,
        // and start from the top.
        return await stepContext.ReplaceDialogAsync(
            "choiceswaterfall",
            cancellationToken: cancellationToken);
    }
    ```

### <a name="update-the-install-app-dialog"></a>Обновление диалога установки приложения

Диалог установки приложения выполняет несколько логических задач, которые мы сейчас настроим в четырех шагах каскадного диалога. Перенос существующего кода в каскадные шаги будет отдельной логической задачей для каждого из диалогов. Для каждого шага указан исходный метод, из которого взят его код.

1. Запрос на ввод строки поиска пользователем.
1. Запрос возможных совпадений в базе данных.
   - Если есть одно совпадение, выбор этого значения и продолжение работы.
   - Если совпадений несколько, запрос на выбор одного варианта пользователем.
   - Если совпадений нет, выход из диалога.
1. Запрос на выбор компьютера, где пользователем будет установлено приложение.
1. Сохранение сведений в базе данных и отправка сообщения с подтверждением.

В файле **Dialogs/InstallAppDialog.cs** сделайте следующее:

1. Обновите инструкции `using`.
    ```csharp
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Builder.Dialogs.Choices;
    ```
1. Определите константы для идентификаторов, которые мы будем использовать в запросах и диалогах. Теперь все строки определяются только в одном месте, что упрощает поддержку кода диалогов.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainId = "mainDialog";
    private const string TextId = "textPrompt";
    private const string ChoiceId = "choicePrompt";
    ```
1. Определите константы для ключей, которые мы будем использовать для отслеживания состояния диалога.
    ```csharp
    // Set up keys for managing collected information.
    private const string InstallInfo = "installInfo";
    ```
1. Добавьте конструктор и инициализируйте набор диалогов для компонентного диалога. На этот раз мы явным образом определяем свойство `InitialDialogId`. Это значит, что нам не нужно добавлять основной каскадный диалог первым в набор диалогов. Например, вы можете сначала добавить запросы, и это не вызовет никаких проблем во время выполнения.
    ```csharp
    public InstallAppDialog(string id)
        : base(id)
    {
        // Initialize our dialogs and prompts.
        InitialDialogId = MainId;
        AddDialog(new WaterfallDialog(MainId, new WaterfallStep[] {
            GetSearchTermAsync,
            ResolveAppNameAsync,
            GetMachineNameAsync,
            SubmitRequestAsync,
        }));
        AddDialog(new TextPrompt(TextId));
        AddDialog(new ChoicePrompt(ChoiceId));
    }
    ```
1. Мы можем заменить **StartAsync** первым шагом каскадного диалога.
    - Но нам придется самостоятельно управлять состоянием, для которого мы применим объект установки приложения в состоянии диалога.
    - Сообщение с запросом пользовательских данных становится необязательным параметром в вызове запроса.
    ```csharp
    private async Task<DialogTurnResult> GetSearchTermAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Create an object in dialog state in which to track our collected information.
        stepContext.Values[InstallInfo] = new InstallApp();

        // Ask for the search term.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text("Ok let's get started. What is the name of the application? "),
            },
            cancellationToken);
    }
    ```
1. Мы можем заменить методы **appNameAsync** и **multipleAppsAsync** вторым шагом каскадного диалога.
    - Теперь мы получаем результат запроса, а не просто проверяем последнее сообщение пользователя.
    - Запрос к базе данных и инструкции if организованы здесь так же, как в **appNameAsync**. Код в каждом блоке инструкции if обновлен для поддержки диалогов версии 4.
        - Если обнаружится только одно совпадение, мы обновляем состояние диалога и переходим к следующему шагу.
        - Если совпадений несколько, мы применяем запрос выбора и предлагаем пользователю выбрать вариант из списка. Это означает, что метод **multipleAppsAsync** можно просто удалить.
        - Если совпадений нет, мы завершаем диалог и возвращаем значение NULL в корневой диалог.
    ```csharp
    private async Task<DialogTurnResult> ResolveAppNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the text prompt.
        var appname = stepContext.Result as string;

        // Query the database for matches.
        var names = await this.getAppsAsync(appname);

        if (names.Count == 1)
        {
            // Get our tracking information from dialog state and add the app name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.AppName = names.First();

            return await stepContext.NextAsync();
        }
        else if (names.Count > 1)
        {
            // Ask the user to choose from the list of matches.
            return await stepContext.PromptAsync(
                ChoiceId,
                new PromptOptions
                {
                    Prompt = MessageFactory.Text("I found the following applications. Please choose one:"),
                    Choices = ChoiceFactory.ToChoices(names),
                },
                cancellationToken);
        }
        else
        {
            // If no matches, exit this dialog.
            await stepContext.Context.SendActivityAsync(
                $"Sorry, I did not find any application with the name '{appname}'.",
                cancellationToken: cancellationToken);

            return await stepContext.EndDialogAsync(null, cancellationToken);
        }
    }
    ```
1. Также метод **appNameAsync** запрашивает у пользователя имя компьютера после обработки запроса к базе данных. Мы реализуем эту часть логики в следующем шаге каскадного диалога.
    - Не забывайте, что в версии 4 нам нужно самостоятельно управлять состоянием. Единственной сложностью будет то, что на этот шаг с предыдущего мы можем попасть через две разные ветви логики.
    - Для запроса имени компьютера у пользователя мы применим тот же запрос текста, что и ранее, но с другими вариантами ответов.
    ```csharp
    private async Task<DialogTurnResult> GetMachineNameAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the tracking info. If we don't already have an app name,
        // Then we used the choice prompt to get it in the previous step.
        var install = stepContext.Values[InstallInfo] as InstallApp;
        if (install.AppName is null)
        {
            install.AppName = (stepContext.Result as FoundChoice).Value;
        }

        // We now need the machine name, so prompt for it.
        return await stepContext.PromptAsync(
            TextId,
            new PromptOptions
            {
                Prompt = MessageFactory.Text(
                    $"Found {install.AppName}. What is the name of the machine to install application?"),
            },
            cancellationToken);
    }
    ```
1. Логика из метода **machineNameAsync** попадает в заключительный шаг каскадного диалога.
    - Мы извлекаем имя компьютера из результата, возвращенного запросом текста, и обновляем состояние диалога.
    - Вызов для обновления базы данных мы удалим, так как вспомогательный код размещается в другом проекте.
    - Затем мы отправляем пользователю сообщение об успешном выполнении и заканчиваем диалог.
    ```csharp
    private async Task<DialogTurnResult> SubmitRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            // Get the tracking info and add the machine name.
            var install = stepContext.Values[InstallInfo] as InstallApp;
            install.MachineName = stepContext.Context.Activity.Text;

            //TODO: Save to this information to the database.
        }

        await stepContext.Context.SendActivityAsync(
            $"Great, your request to install {install.AppName} on {install.MachineName} has been scheduled.",
            cancellationToken: cancellationToken);

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. Чтобы имитировать работу с базой данных, в методе **getAppsAsync** запрос будет выполняться по статическому списку, а не к реальной базе данных.
    ```csharp
    private async Task<List<string>> getAppsAsync(string Name)
    {
        List<string> names = new List<string>();

        // Simulate querying the database for applications that match.
        return (from app in AppMsis
            where app.ToLower().Contains(Name.ToLower())
            select app).ToList();
    }

    // Example list of app names in the database.
    private static readonly List<string> AppMsis = new List<string>
    {
        "µTorrent 3.5.0.44178",
        "7-Zip 17.1",
        "Ad-Aware 9.0",
        "Adobe AIR 2.5.1.17730",
        "Adobe Flash Player (IE) 28.0.0.105",
        "Adobe Flash Player (Non-IE) 27.0.0.130",
        "Adobe Reader 11.0.14",
        "Adobe Shockwave Player 12.3.1.201",
        "Advanced SystemCare Personal 11.0.3",
        "Auslogics Disk Defrag 3.6",
        "avast! 4 Home Edition 4.8.1351",
        "AVG Anti-Virus Free Edition 9.0.0.698",
        "Bonjour 3.1.0.1",
        "CCleaner 5.24.5839",
        "Chmod Calculator 20132.4",
        "CyberLink PowerDVD 17.0.2101.62",
        "DAEMON Tools Lite 4.46.1.328",
        "FileZilla Client 3.5",
        "Firefox 57.0",
        "Foxit Reader 4.1.1.805",
        "Google Chrome 66.143.49260",
        "Google Earth 7.3.0.3832",
        "Google Toolbar (IE) 7.5.8231.2252",
        "GSpot 2701.0",
        "Internet Explorer 903235.0",
        "iTunes 12.7.0.166",
        "Java Runtime Environment 6 Update 17",
        "K-Lite Codec Pack 12.1",
        "Malwarebytes Anti-Malware 2.2.1.1043",
        "Media Player Classic 6.4.9.0",
        "Microsoft Silverlight 5.1.50907",
        "Mozilla Thunderbird 57.0",
        "Nero Burning ROM 19.1.1005",
        "OpenOffice.org 3.1.1 Build 9420",
        "Opera 12.18.1873",
        "Paint.NET 4.0.19",
        "Picasa 3.9.141.259",
        "QuickTime 7.79.80.95",
        "RealPlayer SP 12.0.0.319",
        "Revo Uninstaller 1.95",
        "Skype 7.40.151",
        "Spybot - Search & Destroy 1.6.2.46",
        "SpywareBlaster 4.6",
        "TuneUp Utilities 2009 14.0.1000.353",
        "Unlocker 1.9.2",
        "VLC media player 1.1.6",
        "Winamp 5.56 Build 2512",
        "Windows Live Messenger 2009 16.4.3528.331",
        "WinPatrol 2010 31.0.2014",
        "WinRAR 5.0",
    };
    ```

### <a name="update-the-local-admin-dialog"></a>Обновление диалога локального администратора

В версии 3 этот диалог приветствовал пользователя, запускал диалог Formflow и сохранял результат в базе данных. Эти действия легко преобразовать в каскадный диалог с двумя шагами.

1. Обновите инструкции `using`. Обратите внимание на то, что этот диалог включает диалог Formflow из версии 3. В версии 4 мы применим библиотеку Formflow, созданную сообществом.
    ```csharp
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. Мы можем удалить свойство экземпляра для `LocalAdmin`, так как результат будет доступен в состоянии диалога.
1. Определите константы для идентификаторов, которые мы будем использовать в диалогах. В библиотеке сообщества в качестве имени для созданного диалога всегда применяется имя класса, создаваемого этим диалогом.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string AdminDialog { get; } = nameof(LocalAdminPrompt);
    ```
1. Добавьте конструктор и инициализируйте набор диалогов для компонентного диалога. Диалог Formflow создается аналогичным образом. Мы просто добавляем его в набор диалогов для компонентного диалога в конструкторе.
    ```csharp
    public LocalAdminDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            SaveResultAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildLocalAdminForm, FormOptions.PromptInStart));
    }
    ```
1. Мы можем заменить **StartAsync** первым шагом каскадного диалога. Мы уже создали Formflow в конструкторе, а остальные две инструкции будут преобразованы соответствующим образом.
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Great I will help you request local machine admin.");

        // Begin the Formflow dialog.
        return await stepContext.BeginDialogAsync(AdminDialog, cancellationToken: cancellationToken);
    }
    ```
1. Мы можем заменить **ResumeAfterLocalAdminFormDialog** вторым шагом каскадного диалога. Нам нужно получать возвращаемое значение из контекста шага, а не свойства экземпляра.
    ```csharp
    private async Task<DialogTurnResult> SaveResultAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var admin = stepContext.Result as LocalAdminPrompt;

            //TODO: Save to this information to the database.
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```
1. **BuildLocalAdminForm** почти не изменяется, за исключением того, что теперь Formflow не будет обновлять свойство экземпляра.
    ```csharp
    // Nearly the same as before.
    private IForm<LocalAdminPrompt> BuildLocalAdminForm()
    {
        //here's an example of how validation can be used in form builder
        return new FormBuilder<LocalAdminPrompt>()
            .Field(nameof(LocalAdminPrompt.MachineName),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.MachineName = (string)value;
                return result;
            })
            .Field(nameof(LocalAdminPrompt.AdminDuration),
            validate: async (state, value) =>
            {
                ValidateResult result = new ValidateResult { IsValid = true, Value = value };
                //add validation here

                //this.admin.AdminDuration = Convert.ToInt32((long)value) as int?;
                return result;
            })
            .Build();
    }
    ```

### <a name="update-the-reset-password-dialog"></a>Обновление диалога смены пароля

В версии 3 этот диалог приветствует пользователя, авторизует пользователя по коду доступа, возвращает ошибку авторизации или запускает диалог Formflow и сбрасывает значение пароля. Все это неплохо преобразуется в формат каскада.

1. Обновите инструкции `using`. Обратите внимание на то, что этот диалог включает диалог Formflow из версии 3. В версии 4 мы применим библиотеку Formflow, созданную сообществом.
    ```csharp
    using System;
    using System.Threading;
    using System.Threading.Tasks;
    using Bot.Builder.Community.Dialogs.FormFlow;
    using ContosoHelpdeskChatBot.Models;
    using Microsoft.Bot.Builder.Dialogs;
    ```
1. Определите константы для идентификаторов, которые мы будем использовать в диалогах. В библиотеке сообщества в качестве имени для созданного диалога всегда применяется имя класса, создаваемого этим диалогом.
    ```csharp
    // Set up our dialog and prompt IDs as constants.
    private const string MainDialog = "mainDialog";
    private static string ResetDialog { get; } = nameof(ResetPasswordPrompt);
    ```
1. Добавьте конструктор и инициализируйте набор диалогов для компонентного диалога. Диалог Formflow создается аналогичным образом. Мы просто добавляем его в набор диалогов для компонентного диалога в конструкторе.
    ```csharp
    public ResetPasswordDialog(string dialogId) : base(dialogId)
    {
        InitialDialogId = MainDialog;

        AddDialog(new WaterfallDialog(MainDialog, new WaterfallStep[]
        {
            BeginFormflowAsync,
            ProcessRequestAsync,
        }));
        AddDialog(FormDialog.FromForm(BuildResetPasswordForm, FormOptions.PromptInStart));
    }
    ```
1. Мы можем заменить **StartAsync** первым шагом каскадного диалога. Мы уже создали Formflow в конструкторе. Для всего остального мы сохраняем прежнюю логику, а вызовы версии 3 просто преобразуем в эквиваленты для версии 4.
    ```csharp
    private async Task<DialogTurnResult> BeginFormflowAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        await stepContext.Context.SendActivityAsync("Alright I will help you create a temp password.");

        // Check the passcode and fail out or begin the Formflow dialog.
        if (sendPassCode(stepContext))
        {
            return await stepContext.BeginDialogAsync(ResetDialog, cancellationToken: cancellationToken);
        }
        else
        {
            //here we can simply fail the current dialog because we have root dialog handling all exceptions
            throw new Exception("Failed to send SMS. Make sure email & phone number has been added to database.");
        }
    }
    ```
1. **sendPassCode** используется в основном для примера. Мы закомментировали весь исходный код, и теперь этот метод просто возвращает значение true. Также мы можем удалить адрес электронной почты, так как он не использовался в старой версии бота.
    ```csharp
    private bool sendPassCode(DialogContext context)
    {
        //bool result = false;

        //Recipient Id varies depending on channel
        //refer ChannelAccount class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dd/def/class_microsoft_1_1_bot_1_1_connector_1_1_channel_account.html#a0b89cf01fdd73cbc00a524dce9e2ad1a
        //as well as Activity class https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html
        //int passcode = new Random().Next(1000, 9999);
        //Int64? smsNumber = 0;
        //string smsMessage = "Your Contoso Pass Code is ";
        //string countryDialPrefix = "+1";

        // TODO: save PassCode to database
        //using (var db = new ContosoHelpdeskContext())
        //{
        //    var reset = db.ResetPasswords.Where(r => r.EmailAddress == email).ToList();
        //    if (reset.Count >= 1)
        //    {
        //        reset.First().PassCode = passcode;
        //        smsNumber = reset.First().MobileNumber;
        //        result = true;
        //    }

        //    db.SaveChanges();
        //}

        // TODO: send passcode to user via SMS.
        //if (result)
        //{
        //    result = Helper.SendSms($"{countryDialPrefix}{smsNumber.ToString()}", $"{smsMessage} {passcode}");
        //}

        //return result;
        return true;
    }
    ```
1. **BuildResetPasswordForm** сохраняется в неизменном виде.
1. Мы можем заменить **ResumeAfterLocalAdminFormDialog** вторым шагом каскадного диалога, а возвращаемое значение теперь будем получать из контекста шага. Мы удалили адрес электронной почты, с которым исходный диалог не выполнял никаких действий, и вместо запроса к базе данных возвращаем фиктивный результат. Мы полностью сохраняем прежнюю логику, а вызовы версии 3 преобразуем в эквиваленты для версии 4.
    ```csharp
    private async Task<DialogTurnResult> ProcessRequestAsync(
        WaterfallStepContext stepContext,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Get the result from the Formflow dialog when it ends.
        if (stepContext.Reason != DialogReason.CancelCalled)
        {
            var prompt = stepContext.Result as ResetPasswordPrompt;
            int? passcode;

            // TODO: Retrieve the passcode from the database.
            passcode = 1111;

            if (prompt.PassCode == passcode)
            {
                string temppwd = "TempPwd" + new Random().Next(0, 5000);
                await stepContext.Context.SendActivityAsync(
                    $"Your temp password is {temppwd}",
                    cancellationToken: cancellationToken);
            }
        }

        return await stepContext.EndDialogAsync(null, cancellationToken);
    }
    ```

### <a name="update-models-as-necessary"></a>Обновление моделей при необходимости

Нам нужно обновить инструкции `using` в некоторых моделях, которые ссылаются на библиотеку Formflow.

1. В `LocalAdminPrompt` поместите вместо них этот код:
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    ```
1. В `ResetPasswordPrompt` поместите вместо них этот код:
    ```csharp
    using Bot.Builder.Community.Dialogs.FormFlow;
    using System;
    ```

## <a name="update-webconfig"></a>Обновление файла Web.config

Закомментируйте ключи конфигурации для **MicrosoftAppId** и **MicrosoftAppPassword**. Это позволит вам выполнять локальную отладку бота, не указывая эти значения в эмуляторе.

## <a name="run-and-test-your-bot-in-the-emulator"></a>Запуск и тестирование бота в эмуляторе

На этом этапе мы уже можем запустить бота локально в IIS и подключить его к эмулятору.

1. Запустите бот в IIS.
1. Запустите эмулятор и подключитесь к конечной точке бота (например, **http://localhost:3978/api/messages**).
    - Если бот запускается впервые, щелкните **Файл > Новая программа-робот** и следуйте инструкциям на экране. В противном случае, чтобы открыть существующий бот, щелкните **Файл > Открыть программу-робота**.
    - Внимательно проверьте параметры порта в конфигурации. Например, если в браузере бот открывается по адресу `http://localhost:3979/`, укажите в эмуляторе для бота конечную точку `http://localhost:3979/api/messages`.
1. Все четыре диалога должны работать. Вы можете установить точки останова в нужных шагах каскада, чтобы проверить контекст и состояние диалога в этих точках.

## <a name="additional-resources"></a>Дополнительные ресурсы

Тематические статьи по версии 4:

- [Принципы работы бота](../bot-builder-basics.md)
- [Управление состоянием](../bot-builder-concept-state.md)
- [Библиотека диалогов](../bot-builder-concept-dialog.md)

Пошаговые инструкции для версии 4:

- [Отправка и получение текстовых сообщений](../bot-builder-howto-send-messages.md)
- [Сохранение данных пользователя и диалога](../bot-builder-howto-v4-state.md)
- [Реализация процесса общения](../bot-builder-dialog-manage-conversation-flow.md)
- [Сбор данных, которые вводит пользователь, с помощью диалогового окна](../bot-builder-prompts.md)

<!-- TODO:
- The conceptual piece
- The migration to a .NET Core project
-->

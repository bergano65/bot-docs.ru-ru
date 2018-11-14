---
redirect_url: /bot-framework/bot-builder-howto-v4-state
ms.openlocfilehash: 081c7c1f3e354d4352baea029840c8175152116e
ms.sourcegitcommit: a54a70106b9fdf278fd7270b25dd51c9bd454ab1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/08/2018
ms.locfileid: "51273121"
---
<a name="--"></a><!--
---
title: Создание собственных запросов для получения данных, вводимых пользователем | Документация Майкрософт description: Сведения об управлении потоком беседы с помощью примитивных запросов в пакете SDK Bot Builder.
keywords: conversation flow, prompts author: v-ducvo ms.author: v-ducvo manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 10/31/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="create-your-own-prompts-to-gather-user-input"></a>Создание собственных запросов на сбор данных, вводимых пользователем

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Общение между ботом и пользователем часто подразумевает запрашивание у пользователя информации, анализ его ответа и выполнение действий с учетом этой информации. В разделе [Создание запросов на ввод данных пользователем с помощью библиотеки диалогов](bot-builder-prompts.md) описано, как запросить у пользователя сведения с помощью библиотеки **диалогов**. Помимо прочего, библиотека **диалогов** отвечает за отслеживание текущего диалога и текущего вопроса. Она также предоставляет методы для запроса и проверки информации различных типов, таких как текст, числа, дата и время и т. д.

В некоторых случаях встроенная библиотека **диалогов** может не подойти для вашего бота. Использование библиотеки **диалогов** может привести к большим затратам на обслуживание простых ботов. Также она может быть слишком негибкой или же неэффективной для реализации задач бота. В таком случае можно не использовать эту библиотеку, а реализовать собственную логику запросов. В этом разделе объясняется, как настроить базовый бот *EchoBot*, чтобы управлять диалогом с использованием собственных запросов.

## <a name="track-prompt-states"></a>Отслеживание состояний запросов

В диалоге на основе запросов необходимо отслеживать, в какой точке диалога вы сейчас находитесь и какой вопрос был только что задан. В коде это реализуется с помощью управления несколькими флагами.

Например, можно создать несколько свойств, которые нужно отслеживать.

В этих состояниях хранятся сведения о текущей теме и текущем запросе. Чтобы эти флаги функционировали должным образом после развертывания в облако, мы сохраняем их в данных о [состояния диалога](bot-builder-howto-v4-state.md). 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Мы создаем два класса для отслеживания состояния: **TopicState** для отслеживания хода диалога на основе запросов и **UserProfile** для отслеживания сведений, которые предоставляет пользователь. Эту информацию мы сохраним в [состоянии](bot-builder-howto-v4-state.md) бота позже.

```csharp
/// <summary>
/// Contains conversation state information about the conversation in progress.
/// </summary>
public class TopicState
{
    /// <summary>
    /// Identifies the current "topic" of conversation.
    /// </summary>
    public string Topic { get; set; }

    /// <summary>
    /// Indicates whether we asked the user a question last turn, and
    /// if so, what it was.
    /// </summary>
    public string Prompt { get; set; }
}
```

```csharp
/// <summary>
/// Contains user state information for the user's profile.
/// </summary>
public class UserProfile
{
    public string UserName { get; set; }

    public int? Age { get; set; }

    public string WorkPlace { get; set; }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В **index.js** добавьте `UserState` в оператор require.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

Затем создайте объект управления состоянием пользователя и передайте его при создании бота.

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);

// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

В файле **bot.js** определите идентификаторы для методов доступа к свойству состояния, которые нам потребуются для управления [состоянием](bot-builder-howto-v4-state.md) бота. Также определите запросы для получения необходимых сведений от пользователя.

Добавьте следующий код вне класса `MyBot`.

```javascript
// Define identifiers for our state property accessors.
const TOPIC_STATE_PROPERTY = 'topicStateProperty';
const USER_PROFILE_PROPERTY = 'userProfileProperty';

// Define the prompts to use to ask for user profile information.
const fields = {
    userName: "What is your name?",
    age: "How old are you?",
    workPlace: "Where do you work?"
}
```

---

## <a name="manage-a-topic-of-conversation"></a>Управление темой диалога

При последовательном общении, в ходе которого, например, собирается информация от пользователя, необходимо знать, в какой точке диалога пользователь присоединился и в какой находится сейчас. Это можно отслеживать в основном обработчике реплик бота. Для этого нужно установить и проверять флаги запросов, которые мы определили ранее, а затем реагировать соответствующим образом. В следующем примере показано, как использовать эти флаги для сбора данных профиля пользователя в процессе общения.

Ниже приведен код бота. Обсуждение кода идет в следующем разделе.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Для ASP.NET Core сначала нужно настроить бот и внедрение зависимостей.

Переименуйте файл **EchoWithCounterBot.cs** в **PrimitivePromptsBot.cs** и обновите имя класса. Этот класс содержит логику бота, и вскоре мы ее обновим.

Переименуйте файл **EchoBotAccessors.cs** в **BotAccessors.cs** и обновите имя класса. Этот класс содержит объекты управления состоянием и методы доступа к свойствам состояния бота. Обновите определение следующим образом.

```csharp
using Microsoft.Bot.Builder;
using System;

/// <summary>
/// Contains the state and state property accessors for the primitive prompts bot.
/// </summary>
public class BotAccessors
{
    public const string TopicStateName = "PrimitivePrompts.TopicStateAccessor";

    public const string UserProfileName = "PrimitivePrompts.UserProfileAccessor";

    public ConversationState ConversationState { get; }

    public UserState UserState { get; }

    public IStatePropertyAccessor<TopicState> TopicStateAccessor { get; set; }

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public BotAccessors(ConversationState conversationState, UserState userState)
    {
        if (conversationState is null)
        {
            throw new ArgumentNullException(nameof(conversationState));
        }

        if (userState is null)
        {
            throw new ArgumentNullException(nameof(userState));
        }

        this.ConversationState = conversationState;
        this.UserState = userState;
    }
}
```

Обновите метод `ConfigureServices` файла **Startup.cs**, начиная с того места, где вы настроили объект `IStorage`.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<PrimitivePromptsBot>(options =>
    {
        // ...

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, everything stored in memory will be gone.
        IStorage dataStore = new MemoryStorage();

        var conversationState = new ConversationState(dataStore);
        options.State.Add(conversationState);

        var userState = new UserState(dataStore);
        options.State.Add(userState);
    });

    // Create and register state accessors.
    // Accessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        var options = sp.GetRequiredService<IOptions<BotFrameworkOptions>>().Value;
        var conversationState = options.State.OfType<ConversationState>().FirstOrDefault();
        var userState = options.State.OfType<UserState>().FirstOrDefault();

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState, userState)
        {
            TopicStateAccessor = conversationState.CreateProperty<TopicState>(BotAccessors.TopicStateName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(BotAccessors.UserProfileName),
        };

        return accessors;
    });
}
```

Наконец, обновите логику бота в файле **PrimitivePromptsBot.cs**.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;

public class PrimitivePromptsBot : IBot
{
    public const string ProfileTopic = "profile";

    /// <summary>
    /// Describes a field in the user profile.
    /// </summary>
    private class UserFieldInfo
    {
        /// <summary>
        /// The ID to use for this field.
        /// </summary>
        public string Key { get; set; }

        /// <summary>
        /// The prompt to use to ask for a value for this field.
        /// </summary>
        public string Prompt { get; set; }

        /// <summary>
        /// Gets the value of the corresponding field.
        /// </summary>
        public Func<UserProfile, string> GetValue { get; set; }

        /// <summary>
        /// Sets the value of the corresponding field.
        /// </summary>
        public Action<UserProfile, string> SetValue { get; set; }
    }

    /// <summary>
    /// The prompts for the user profile, indexed by field name.
    /// </summary>
    private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
    {
        new UserFieldInfo {
            Key = nameof(UserProfile.UserName),
            Prompt = "What is your name?",
            GetValue = (profile) => profile.UserName,
            SetValue = (profile, value) => profile.UserName = value,
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.Age),
            Prompt = "How old are you?",
            GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
            SetValue = (profile, value) =>
            {
                if (int.TryParse(value, out int age))
                {
                    profile.Age = age;
                }
            },
        },
        new UserFieldInfo {
            Key = nameof(UserProfile.WorkPlace),
            Prompt = "Where do you work?",
            GetValue = (profile) => profile.WorkPlace,
            SetValue = (profile, value) => profile.WorkPlace = value,
        },
    };

    /// <summary>
    /// The state and state accessors for the bot.
    /// </summary>
    private BotAccessors Accessors { get; }

    public PrimitivePromptsBot(BotAccessors accessors)
    {
        Accessors = accessors ?? throw new ArgumentNullException(nameof(accessors));
    }

    public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
    {
        if (turnContext.Activity.Type is ActivityTypes.Message)
        {
            // Use the state property accessors to get the topic state and user profile.
            TopicState topicState = await Accessors.TopicStateAccessor.GetAsync(
                turnContext,
                () => new TopicState { Topic = ProfileTopic, Prompt = null },
                cancellationToken);
            UserProfile userProfile = await Accessors.UserProfileAccessor.GetAsync(
                turnContext,
                () => new UserProfile(),
                cancellationToken);

            // Check whether we need more information.
            if (topicState.Topic is ProfileTopic)
            {
                // If we're expecting input, record it in the user's profile.
                if (topicState.Prompt != null)
                {
                    UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }

                // Determine which fields are not yet set.
                List<UserFieldInfo> emptyFields = UserFields.Where(f => f.GetValue(userProfile) is null).ToList();

                if (emptyFields.Any())
                {
                    // If all the fields are empty, send a welcome message.
                    if (emptyFields.Count == UserFields.Count)
                    {
                        await turnContext.SendActivityAsync("Welcome new user, please fill out your profile information.");
                    }

                    // We have at least one empty field. Prompt for the next empty field,
                    // and update the prompt flag to indicate which prompt we just sent,
                    // so that the response can be captured at the beginning of the next turn.
                    UserFieldInfo field = emptyFields.First();
                    await turnContext.SendActivityAsync(field.Prompt);
                    topicState.Prompt = field.Key;
                }
                else
                {
                    // Our user profile is complete!
                    await turnContext.SendActivityAsync($"Thank you, {userProfile.UserName}. Your profile is complete.");
                    topicState.Prompt = null;
                    topicState.Topic = null;
                }
            }
            else if (turnContext.Activity.Text.Trim().Equals("hi", StringComparison.InvariantCultureIgnoreCase))
            {
                await turnContext.SendActivityAsync($"Hi. {userProfile.UserName}.");
            }
            else
            {
                await turnContext.SendActivityAsync("Hi. I'm the Contoso cafe bot.");
            }

            // Use the state property accessors to update the topic state and user profile.
            await Accessors.TopicStateAccessor.SetAsync(turnContext, topicState, cancellationToken);
            await Accessors.UserProfileAccessor.SetAsync(turnContext, userProfile, cancellationToken);

            // Save any state changes to storage.
            await Accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            await Accessors.UserState.SaveChangesAsync(turnContext, false, cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В файле **bot.js** обновите определение класса `MyBot`.

Методы доступа к свойству состояния мы включаем в конструктор бота: `topicStateAccessor` и `userProfileAccessor`. При определении состояния темы отслеживается тема беседы. Собранные сведения о пользователе хранятся в его профиле.

```javascript
constructor(conversationState, userState) {
    // Create state property accessors.
    this.topicStateAccessor = conversationState.createProperty(TOPIC_STATE_PROPERTY);
    this.userProfileAccessor = userState.createProperty(USER_PROFILE_PROPERTY);

    // Track the conversation and user state management objects.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

Теперь обновите обработчик реплик, чтобы с помощью сведений о состоянии бота управлять потоком беседы и сохранять собранные данные о пользователе.

```javascript
async onTurn(turnContext) {
    // Handle only message activities from the user.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get state properties using their accessors, providing default values.
        const topicState = await this.topicStateAccessor.get(turnContext, {
            prompt: undefined,
            topic: 'profile'
        });
        const userProfile = await this.userProfileAccessor.get(turnContext, {
            "userName": undefined,
            "age": undefined,
            "workPlace": undefined
        });

        if (topicState.topic === 'profile') {
            // If a prompt flag is set in the conversation state, use it to capture the incoming value
            // into the appropriate field of the user profile.
            if (topicState.prompt) {
                userProfile[topicState.prompt] = turnContext.activity.text;
            }

            // Determine which fields are not yet set.
            const empty_fields = [];
            Object.keys(fields).forEach(function (key) {
                if (!userProfile[key]) {
                    empty_fields.push({
                        key: key,
                        prompt: fields[key]
                    });
                }
            });

            if (empty_fields.length) {

                // If all the fields are empty, send a welcome message.
                if (empty_fields.length == Object.keys(fields).length) {
                    await turnContext.sendActivity('Welcome new user, please fill out your profile information.');
                }

                // We have at least one empty field. Prompt for the next empty field.
                await turnContext.sendActivity(empty_fields[0].prompt);

                // update the flag to indicate which prompt we just sent
                // so that the response can be captured at the beginning of the next turn.
                topicState.prompt = empty_fields[0].key;

            } else {
                // Our user profile is complete!
                await turnContext.sendActivity('Thank you. Your profile is complete.');
                topicState.prompt = null;
                topicState.topic = null;

            }
        } else if (turnContext.activity.text && turnContext.activity.text.match(/hi/ig)) {
            // Check to see if the user said "hi" and respond with a greeting
            await turnContext.sendActivity(`Hi ${userProfile.userName}.`);
        } else {
            // Default message
            await turnContext.sendActivity("Hi. I'm the Contoso bot.");
        }

        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

Приведенный выше пример кода задает для флага _topic_ (тема) значение `profile`, чтобы начать диалог для сбора данных профиля. В ходе диалога бот устанавливает для флага _prompt_ (запрос) значение, соответствующее текущему заданному вопросу. Если для флага установлено надлежащее значение, бот знает, что делать со следующим сообщением от пользователя и обрабатывает его соответствующим образом.

Когда бот заканчивает собирать сведения, флаги сбрасываются. В противном случае бот зациклится и не сможет продолжить работу после постановки последнего вопроса.

Определяя другие флаги или создавать дополнительные ветви диалога в зависимости от поступаемой от пользователя информации, этот шаблон можно приспособить для более сложных диалогов.

## <a name="manage-multiple-topics-of-conversations"></a>Управление несколькими темами диалога

Разобравшись с тем, как обрабатывать одну тему диалога, вы можете расширить логику бота, чтобы он обрабатывал несколько тем диалога. Обрабатывать несколько тем можно путем проверки на соответствие дополнительным условиям и последующим выбором соответствующего пути.

Приведенный выше пример можно расширить, предусмотрев другие функции и темы диалога, например резервирование столика или размещение заказа. Для отслеживания диалога добавляйте в класс TopicState дополнительные флаги.

Возможно, вам будет удобней разделить код на независимые функции или методы, чтобы было проще контролировать ход диалога. Распространенный подход заключается в том, чтобы бот оценивал сообщение и состояние, после чего передавал управление функциям, в которых реализованы соответствующие возможности.

Чтобы пользователям было проще ориентироваться в нескольких темах диалога, рекомендуем создать главное меню. Например, с помощью [предлагаемых действий](bot-builder-howto-add-suggested-actions.md) можно предложить пользователям выбрать одну из доступных тем диалога, а не вынуждать их гадать, какие темы может обсуждать бот.

## <a name="validate-user-input"></a>Проверка вводимых пользователем данных

В библиотеке **диалогов** предоставляются встроенные методы проверки введенных пользователем данных, но это можно сделать и с помощью собственных запросов. Например, запрашивая возраст пользователя, в качестве ответа мы ожидаем получить число, а не что-то наподобие "Кирилл".

Анализ чисел или даты и времени — это сложная задача, которая не рассматривается в этой статье. К счастью, вы можете использовать библиотеку. Для анализа этих сведений используется библиотека Майкрософт для [распознавания текста](https://github.com/Microsoft/Recognizers-Text). Этот пакет доступен через NuGet и npm. Кроме того, вы можете загрузить его непосредственно из репозитория. (Он также включен в библиотеку **диалогов**. Хотя она и не используется в рамках этой статьи, об этом стоит упомянуть.)

Эта библиотека особенно полезна для анализа сложных данных, вводимых пользователем, например даты, время или номера телефонов. В этом примере мы проверяем число гостей, на которых заказан ужин. Но этот же подход с определенными доработками можно применить и для более сложных проверок.

В примере ниже мы только рассмотрим использование метода `RecognizeNumber`. Сведения о том, как использовать другие методы распознавателя из библиотеки, можно найти в [документации репозитория](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы использовать библиотеку **Microsoft.Recognizers.Text.Number**, включите пакет NuGet и добавьте указанные ниже операторы using в файл бота.

```csharp
using System.Linq;
using Microsoft.Recognizers.Text;
using Microsoft.Recognizers.Text.Number;
```

Проверку можно выполнять множеством разных способов. Здесь мы включим эту проверку во вспомогательный класс.

Добавьте во внутренний класс `UserFieldInfo` бота перечисленные ниже методы.

```csharp
/// <summary>Delegate for validating input.</summary>
/// <param name="turnContext">The current turn context. turnContext.Activity.Text contains the input to validate.</param>
/// <returns><code>true</code> if the input is valid; otherwise, <code>false</code>.</returns>
public delegate Task<bool> ValidatorDelegate(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken));

/// <summary>By default, evaluate all input as valid.</summary>
private static readonly ValidatorDelegate NoValidator =
    async (ITurnContext turnContext, CancellationToken cancellationToken) => true;

/// <summary>Gets or sets the validation function to use.</summary>
public ValidatorDelegate ValidateInput { get; set; } = NoValidator;
```

Затем обновите запись _age_ (возраст) в файле `UserFields` бота, чтобы указать используемую проверку.
Так как мы будем проверять входные данные перед изменением значения возраста, функцию `SetValue` можно немного упростить, применив библиотеку средств распознавания текста.

```csharp
private static List<UserFieldInfo> UserFields { get; } = new List<UserFieldInfo>
{
    // ...
    new UserFieldInfo {
        Key = nameof(UserProfile.Age),
        Prompt = "How old are you?",
        GetValue = (profile) => profile.Age.HasValue? profile.Age.Value.ToString() : null,
        SetValue = (profile, value) =>
        {
            // As long as the input validates, this should work.
            List<ModelResult> result = NumberRecognizer.RecognizeNumber(value, Culture.English);
            profile.Age = int.Parse(result.First().Text);
        },
        ValidateInput = async (turnContext, cancellationToken) =>
        {
            try
            {
                // Recognize the input as a number. This works for responses such as
                // "twelve" as well as "12".
                List<ModelResult> result = NumberRecognizer.RecognizeNumber(
                    turnContext.Activity.Text, Culture.English);

                // Attempt to convert the Recognizer result to an integer
                int.TryParse(result.First().Text, out int age);

                if (age < 18)
                {
                    await turnContext.SendActivityAsync(
                        "You must be 18 or older.",
                        cancellationToken: cancellationToken);
                    return false;
                }
                else if (age > 120)
                {
                    await turnContext.SendActivityAsync(
                        "You must be 120 or younger.",
                        cancellationToken: cancellationToken);
                    return false;
                }
            }
            catch
            {
                await turnContext.SendActivityAsync(
                    "I couldn't understand your input. Please enter your age in years.",
                    cancellationToken: cancellationToken);
                return false;
            }

            // If we got through this, the number is valid.
            return true;
        },
    },
    // ...
};
```

И, наконец, мы обновим обработчик реплик, чтобы проверять все входные данные перед сохранением значения свойства.
По умолчанию для проверки используется функция NoValidator, которая принимает любые входные данные. Это означает, что поведение изменится только для запроса на ввод возраста. Если входные данные не пройдут проверку, значение для этого поля не указывается и бот в следующей реплике снова запросит входные данные для этого поля.

Здесь представлен только тот фрагмент обработчика реплик, который нам нужно изменить.

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type is ActivityTypes.Message)
    {
        // ...
        // Check whether we need more information.
        if (topicState.Topic is ProfileTopic)
        {
            // If we're expecting input, record it in the user's profile.
            if (topicState.Prompt != null)
            {
                UserFieldInfo field = UserFields.First(f => f.Key.Equals(topicState.Prompt));
                if (await field.ValidateInput(turnContext, cancellationToken))
                {
                    field.SetValue(userProfile, turnContext.Activity.Text.Trim());
                }
            }

            // ...
        }
        //...
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы использовать библиотеку **распознавателей**, добавьте пакет и включите для него инструкцию require в код бота (в файле **bot.js**):

```bash
npm i @microsoft/recognizers-text-suite --save
```

```javascript
// Required packages for this bot.
const Recognizers = require('@microsoft/recognizers-text-suite');
```

Затем обновите метаданные `fields`, добавив в них код распознавания текста и проверки:

```javascript
// Define the prompts to use to ask for user profile information.
const fields = {
    userName: { prompt: "What is your name?" },
    age: {
        prompt: "How old are you?",
        recognize: (turnContext) => {
            var result = Recognizers.recognizeNumber(
                turnContext.activity.text, Recognizers.Culture.English);
            return parseInt(result[0].resolution.value);
        },
        validate: async (turnContext) => {
            try {
                // Recognize the input as a number. This works for responses such as
                // "twelve" as well as "12".
                var result = Recognizers.recognizeNumber(
                    turnContext.activity.text, Recognizers.Culture.English);
                var age = parseInt(result[0].resolution.value);
                if (age < 18) {
                    await turnContext.sendActivity("You must be 18 or older.");
                    return false;
                }
                if (age > 120 ) {
                    await turnContext.sendActivity("You must be 120 or younger.");
                    return false;
                }
            } catch (_) {
                await turnContext.sendActivity(
                    "I couldn't understand your input. Please enter your age in years.");
                return false;
            }
            return true;
        }
    },
    workPlace: { prompt: "Where do you work?" }
}
```

В обработчике реплик бота обновите указанные ниже блоки, в которых сохраняются входные данные пользователя и пользователю подается запрос. Необходимо обновить эти разделы с учетом изменений в метаданных полей.

```javascript
async onTurn(turnContext) {
    // Handle only message activities from the user.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // ...

        if (topicState.topic === 'profile') {
            // If a prompt flag is set in the conversation state, use it to capture the incoming value
            // into the appropriate field of the user profile.
            if (topicState.prompt) {
                const field = fields[topicState.prompt];
                // If the prompt has validation, check whether the input validates.
                if (!field.validate || await field.validate(turnContext)) {
                    // Set the field, using a recognizer if one is defined.
                    userProfile[topicState.prompt] = (field.recognize)
                        ? field.recognize(turnContext)
                        : turnContext.activity.text;
                }
            }

            // ...

            if (empty_fields.length) {

                // ...

                // We have at least one empty field. Prompt for the next empty field.
                await turnContext.sendActivity(empty_fields[0].prompt.prompt);

                // ...

            } // ...
        } // ...

        // Save state changes
        await this.conversationState.saveChanges(turnContext);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

## <a name="next-step"></a>Дальнейшие действия

Теперь, когда вы знаете, как обрабатывать состояния запросов самостоятельно, узнайте, как с помощью библиотеки **диалогов** запросить у пользователя входные данные.

> [!div class="nextstepaction"]
> [Создание запросов на ввод данных пользователем с помощью библиотеки диалогов](bot-builder-prompts.md)

-->

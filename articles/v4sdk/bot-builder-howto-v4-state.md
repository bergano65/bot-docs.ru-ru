---
title: Сохранение данных пользователя и диалога | Документация Майкрософт
description: Узнайте, как сохранять и извлекать данные о состоянии с помощью пакета SDK Bot Builder.
keywords: conversation state, user state, conversation, saving state, managing bot state
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/26/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8f979aed3bc1c4bb4c74629bcffb258e139ce77d
ms.sourcegitcommit: bcde20bd4ab830d749cb835c2edb35659324d926
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/27/2018
ms.locfileid: "52338557"
---
# <a name="save-user-and-conversation-data"></a>Сохранение данных пользователя и диалога

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Бот по своей природе не учитывает состояния. Развернутый бот не обязан выполнять следующий шаг в том же процессе или на том же компьютере, что и предыдущий. Но иногда боту нужно отслеживать контекст общения, чтобы управлять его ходом и запоминать ответы на предыдущие вопросы. Функции состояния и хранения, предоставляемые пакетом SDK, позволяют реализовать в боте поддержку состояния.

## <a name="prerequisites"></a>Предварительные требования

- Требуется понимание [основных сведений о ботах](bot-builder-basics.md) и [управления состоянием](bot-builder-concept-state.md) для ботов.
- Код в этой статье основан на примере **StateBot**. Вам потребуется копия этого примера на языке [C#](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) или [JS]().
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) для локального тестирования бота.

## <a name="about-the-sample-code"></a>Сведения о примере кода

В этой статье рассматриваются аспекты конфигурации, имеющие отношение к управлению состоянием бота. Чтобы добавить состояние, мы настроим свойства состояния, управление состоянием и хранилищем, а затем применим их в боте.

- Каждое _свойство состояния_ содержит сведения о состоянии бота.
- Каждый метод доступа к свойству состояния позволяет получить или задать значение для соответствующего свойства состояния.
- Каждый объект управления состоянием автоматизирует процессы считывания и записи в хранилище соответствующих сведений о состоянии.
- Уровень хранилища подключается к резервному хранилищу, настроенному для состояния, например к хранилищу в памяти (для тестирования) или Azure Cosmos DB (для рабочей среды).

Для нашего примера бота потребуются методы доступа к свойствам состояния, которые позволят получать и задавать состояние во время выполнения по мере обработки действий. Метод доступа к свойству состояния создается на основе объекта управления, который в свою очередь создается из слоя хранилища. Итак, мы начнем работу на уровне хранилища и будем постепенно подниматься от него вверх.

## <a name="configure-storage"></a>Настройка хранилища

Мы не планируем развертывать этот бот, поэтому применим область _временной памяти_ для состояний пользователя и беседы, которые мы настроим на следующем шаге.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Настройте уровень хранилища в файле **Startup.cs**.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    IStorage storage = new MemoryStorage();
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Настройте уровень хранилища в файле **index.js**.

```javascript
// Define state store for your bot.
const memoryStorage = new MemoryStorage();
```

---

## <a name="create-state-management-objects"></a>Создание объектов управления состоянием

Мы отслеживаем состояния _пользователя_ и _беседы_, и для них на следующем шаге создадим _методы доступа к свойствам состояния_.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

В **Startup.cs** добавьте ссылку на уровень хранилища, создавая объекты управления состоянием.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    ConversationState conversationState = new ConversationState(storage);
    UserState userState = new UserState(storage);
    // ...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В файле **index.js** добавьте `UserState` в раздел инструкций require.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

Теперь добавьте ссылку на уровень хранилища, создавая объекты управления состоянием беседы и пользователя.

```javascript
// Create conversation and user state with in-memory storage provider.
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
```

---

## <a name="create-state-property-accessors"></a>Создание методов доступа к свойству состояния

Чтобы _объявить_ свойство состояния, прежде всего создайте метод доступа к свойству состояния на основе одного из созданных объектов управления состоянием. Нам нужно настроить в боте отслеживание следующих сведений:

- имя пользователя, которые мы определим в состоянии пользователя;
- сведения о том, запрашивали ли мы у пользователя имя и дополнительные сведения об отправленном пользователем сообщении.

Бот использует метод доступа для получения свойства состояния из контекста шага.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Сначала мы определим классы, в которых будут содержаться все необходимые сведения для каждого из типов состояния.

- Класс `UserProfile` для хранения собранных ботом сведений о пользователе.
- Класс `ConversationData` для отслеживания сведений о том, когда и от кого получено сообщение.

```csharp
// Defines a state property used to track information about the user.
public class UserProfile
{
    public string Name { get; set; }
}
```

```csharp
// Defines a state property used to track conversation data.
public class ConversationData
{
    // The time-stamp of the most recent incoming message.
    public string Timestamp { get; set; }

    // The ID of the user's channel.
    public string ChannelId { get; set; }

    // Track whether we have already asked the user's name
    public bool PromptedUserForName { get; set; } = false;
}
```

Далее мы определим класс для сведений об управлении состояниями, которые нам потребуются для настройки экземпляра бота.

```csharp
public class StateBotAccessors
{
    public StateBotAccessors(ConversationState conversationState, UserState userState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
        UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }
  
    public static string UserProfileName { get; } = "UserProfile";

    public static string ConversationDataName { get; } = "ConversationData";

    public IStatePropertyAccessor<UserProfile> UserProfileAccessor { get; set; }

    public IStatePropertyAccessor<ConversationData> ConversationDataAccessor { get; set; }
  
    public ConversationState ConversationState { get; }
  
    public UserState UserState { get; }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Мы передаем объекты управления состоянием напрямую в конструктор бота, чтобы он сам создал из них методы доступа к свойствам состояния.

В файл **index.js** при создании бота добавьте объекты управления состоянием.

```javascript
// Create the bot.
const myBot = new MyBot(conversationState, userState);
```

В файле **bot.js** определите идентификаторы, которые вам потребуются для отслеживания состояния и управления им.

```javascript
// The accessor names for the conversation data and user profile state property accessors.
const CONVERSATION_DATA_PROPERTY = 'conversationData';
const USER_PROFILE_PROPERTY = 'userProfile';
```

---

## <a name="configure-your-bot"></a>Настройка бота

Теперь мы готовы к тому, чтобы определить методы доступа к свойствам состояния и настроить бот.
Для метода доступа к свойству состояния беседы мы применим объект управления состоянием беседы.
Для метода доступа к свойству состояния профиля пользователя мы применим объект управления состоянием пользователя.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

В файле **Startup.cs** мы потребуем, чтобы платформа ASP.NET предоставляла связанные объекты свойств состояний и управления состояниями. Они будут извлекаться в конструкторе бота с помощью платформы внедрения зависимостей, реализованной в ASP.NET Core.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...
    services.AddSingleton<StateBotAccessors>(sp =>
    {
        // Create the custom state accessor.
        return new StateBotAccessors(conversationState, userState)
        {
            ConversationDataAccessor = conversationState.CreateProperty<ConversationData>(StateBotAccessors.ConversationDataName),
            UserProfileAccessor = userState.CreateProperty<UserProfile>(StateBotAccessors.UserProfileName),
        };
    });
}
```

В конструктор бота передается объект `CustomPromptBotAccessors`, когда ASP.NET создает этот бот.

```csharp
// Defines a bot for filling a user profile.
public class CustomPromptBot : IBot
{
    private readonly StateBotAccessors _accessors;

    public StateBot(StateBotAccessors accessors, ILoggerFactory loggerFactory)
    {
        // ...
        accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));
    }

    // The bot's turn handler and other supporting code...
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В конструкторе бота (в файле **bot.js**), мы создадим методы доступа к свойствам состояния и добавим их в бот. Мы также добавим ссылки на объекты управления состоянием, которые потребуются для сохранения любых изменений в состояниях.

```javascript
constructor(conversationState, userState) {
    // Create the state property accessors for the conversation data and user profile.
    this.conversationData = conversationState.createProperty(CONVERSATION_DATA_PROPERTY);
    this.userProfile = userState.createProperty(USER_PROFILE_PROPERTY);

    // The state management objects for the conversation and user state.
    this.conversationState = conversationState;
    this.userState = userState;
}
```

---

## <a name="access-state-from-your-bot"></a>Доступ к состояниям из кода бота

В предыдущих разделах мы рассматривали шаги, которые на этапе инициализации добавляют в бота методы доступа к свойствам состояния.
Теперь мы вызовем эти методы доступа во время выполнения, чтобы считать и (или) записать сведения о состоянии.

1. Прежде чем использовать свойства состояния, мы вызовем каждый метод доступа, чтобы загрузить свойства из хранилища в кэш состояния и получить их оттуда.
   - При получении свойства состояния с помощью метода доступа необходимо указать значение по умолчанию. В противном случае может возникать ошибка из-за отсутствия значения.
1. Прежде чем выйти из обработчика шага, следует выполнить следующее:
   1. вызвать метод доступа _set_ для отправки изменений, внесенных в состояние бота;
   1. вызвать метод _save changes_ (сохранить изменения) из объекта управления состоянием, чтобы сохранить эти изменения в хранилище.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// The bot's turn handler.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        UserProfile userProfile =
            await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());
        ConversationData conversationData =
            await _accessors.ConversationDataAccessor.GetAsync(turnContext, () => new ConversationData());

        if (string.IsNullOrEmpty(userProfile.Name))
        {
            // First time around this is set to false, so we will prompt user for name.
            if (conversationData.PromptedUserForName)
            {
                // Set the name to what the user provided
                userProfile.Name = turnContext.Activity.Text?.Trim();

                // Acknowledge that we got their name.
                await turnContext.SendActivityAsync($"Thanks {userProfile.Name}.");

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.PromptedUserForName = false;
            }
            else
            {
                // Prompt the user for their name.
                await turnContext.SendActivityAsync($"What is your name?");

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.PromptedUserForName = true;
            }

            // Save user state and save changes.
            await _accessors.UserProfileAccessor.SetAsync(turnContext, userProfile);
            await _accessors.UserState.SaveChangesAsync(turnContext);
        }
        else
        {
            // Add message details to the conversation data.
            conversationData.Timestamp = turnContext.Activity.Timestamp.ToString();
            conversationData.ChannelId = turnContext.Activity.ChannelId.ToString();

            // Display state data
            await turnContext.SendActivityAsync($"{userProfile.Name} sent: {turnContext.Activity.Text}");
            await turnContext.SendActivityAsync($"Message received at: {conversationData.Timestamp}");
            await turnContext.SendActivityAsync($"Message received from: {conversationData.ChannelId}");
        }

        // Update conversation state and save changes.
        await _accessors.ConversationDataAccessor.SetAsync(turnContext, conversationData);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const userProfile = await this.userProfile.get(turnContext, {});
        const conversationData = await this.conversationData.get(
            turnContext, { promptedForUserName: false });

        if (!userProfile.name) {
            // First time around this is undefined, so we will prompt user for name.
            if (conversationData.promptedForUserName) {
                // Set the name to what the user provided.
                userProfile.name = turnContext.activity.text;

                // Acknowledge that we got their name.
                await turnContext.sendActivity(`Thanks ${userProfile.name}.`);

                // Reset the flag to allow the bot to go though the cycle again.
                conversationData.promptedForUserName = false;
            } else {
                // Prompt the user for their name.
                await turnContext.sendActivity('What is your name?');

                // Set the flag to true, so we don't prompt in the next turn.
                conversationData.promptedForUserName = true;
            }
            // Save user state and save changes.
            await this.userProfile.set(turnContext, userProfile);
            await this.userState.saveChanges(turnContext);
        } else {
            // Add message details to the conversation data.
            conversationData.timestamp = turnContext.activity.timestamp.toLocaleString();
            conversationData.channelId = turnContext.activity.channelId;

            // Display state data.
            await turnContext.sendActivity(`${userProfile.name} sent: ${turnContext.activity.text}`);
            await turnContext.sendActivity(`Message received at: ${conversationData.timestamp}`);
            await turnContext.sendActivity(`Message received from: ${conversationData.channelId}`);
        }
        // Update conversation state and save changes.
        await this.conversationData.set(turnContext, conversationData);
        await this.conversationState.saveChanges(turnContext);
    }
}
```

---

## <a name="test-the-bot"></a>Тестирование бота

1. Выполните этот пример на локальном компьютере. Если потребуются дополнительные инструкции, изучите файл README для примера на [C#](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/dotnet_core/StateBot) или [JS](https://github.com/Microsoft/BotFramework-Samples/tree/master/SDKV4-Samples/js/stateBot).
1. Примените эмулятор для тестирования бота, как показано ниже.

![Пример тестирования бота с управлением состояния](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

**Конфиденциальность.** Если вы собираетесь хранить персональные данные пользователя, обеспечьте соблюдение [Общего регламента по защите данных](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr).

**Управление состоянием.** Все вызовы методов управления состоянием обрабатываются асинхронно, и по умолчанию применяется только последнее действие, выполняющее запись данных. На практике следует размещать методы get, set и save state как можно ближе друг к другу в коде бота.

**Критически важные для бизнеса данные.** Используйте состояния бота для хранения настроек, имени пользователя или сведений о последнем заказе, но не используйте его для хранения критически важных бизнес-данных. Для критически важных данных [создайте собственные компоненты хранилища](bot-builder-custom-storage.md) или записывайте их непосредственно в [хранилище](bot-builder-howto-v4-storage.md).

**Recognizer-Text.** В этом примере используются библиотеки Microsoft/Recognizer-Text для синтаксического анализа и проверки пользовательского ввода. Дополнительные сведения см. на странице [Использование Azure DNS для частных доменов](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview).

## <a name="next-step"></a>Дальнейшие действия

Теперь вы умеете настраивать состояние, которое упрощает чтение и запись данных бота в хранилище. Давайте перейдем к изучению методов, позволяющих задать пользователю ряд вопросов, проверить его ответы и сохранить полученные данные.

> [!div class="nextstepaction"]
> [Создание собственных запросов на сбор данных, вводимых пользователем](bot-builder-primitive-prompts.md).

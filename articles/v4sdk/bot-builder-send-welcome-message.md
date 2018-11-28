---
title: Отправка приветственного сообщения пользователям | Документация Майкрософт
description: Сведения о том, как создать приветствие, которое бот отправляет пользователю.
keywords: overview, develop, user experience, welcome, personalized experience, C#, JS, welcome message, bot, greet, greeting
author: dashel
ms.author: dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: eb62df9bd1f74ab6de9b67fe352b1af4620a6bc6
ms.sourcegitcommit: d92fd6233295856052305e0d9e3cba29c9ef496e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/16/2018
ms.locfileid: "51715108"
---
# <a name="send-welcome-message-to-users"></a>Отправка приветственного сообщения пользователям

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Основная цель создания любого бота — ведение осмысленного диалога с пользователем. Лучший способ достичь этой цели — сделать так, чтобы с момента присоединения к диалогу пользователь понимал основное назначение вашего бота, его возможности и причины создания. В этой статье представлены примеры кода, которые помогут создать приветствие, отправляемое ботом пользователю.

## <a name="prerequisites"></a>Предварительные требования
- Понимание [основных принципов работы ботов](bot-builder-basics.md). 
- Копия **примера с приветствием пользователя** на языке [C#](https://aka.ms/proactive-sample-cs) или [JS](https://aka.ms/proactive-sample-js). На примере кода в этой статье мы опишем, как отправлять приветственные сообщения.

## <a name="same-welcome-for-different-channels"></a>Отправка приветственного сообщения для различных каналов
Приветственное сообщение должно создаваться, когда пользователь начинает взаимодействовать с ботом. Можно реализовать это, отслеживая типы действий в боте и новые подключения. При каждом новом подключении может создаваться не больше двух действий обновления диалога в зависимости от канала:

- одно при подключении бота к диалогу;
- другое при присоединении пользователя к диалогу.

Казалось бы, нужно просто создать приветственное сообщение при обнаружении нового диалога. Но это может привести к непредвиденным результатам при доступе к боту через различные каналы.

Некоторые каналы обновляют диалог один раз, когда пользователь впервые подключается к каналу, а затем отдельно обновляют диалог, когда от пользователя поступает первое входящее сообщение. Другие каналы создают оба действия, когда пользователь впервые подключается к каналу. Если просто отслеживать событие обновления диалога и отправлять приветственное сообщение в канал после двух действий обновления диалога, ваш пользователь может получить следующее:

![Повторяющееся приветственное сообщение](./media/double_welcome_message.png)

Повтора сообщения можно избежать, создав начальное приветственное сообщение только после второго события обновления диалога. Второе событие считается обнаруженным, если соблюдены оба условия:
- произошло событие обновления диалога;
- в диалог добавлен новый участник (пользователь).

В приведенном ниже примере отслеживаются новые действия *обновления диалога*, отправляется одно приветственное сообщение, когда пользователь присоединяется к диалогу, и устанавливается флаг состояния Prompt, чтобы игнорировать первое сообщение пользователя. 

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Нам нужно создать объект состояния для определенного пользователя в диалоге и метод доступа к нему.

```csharp
/// The state object is used to keep track of various state related to a user in a conversation.
/// In this example, we are tracking if the bot has replied to customer first interaction.
public class WelcomeUserState
{
    public bool DidBotWelcomeUser { get; set; } = false;
}

/// Initializes a new instance of the <see cref="WelcomeUserStateAccessors"/> class.
public class WelcomeUserStateAccessors
{
    public WelcomeUserStateAccessors(UserState userState)
    {
        this.UserState = userState ?? throw new ArgumentNullException(nameof(userState));
    }

    public IStatePropertyAccessor<bool> DidBotWelcomeUser { get; set; }

    public UserState UserState { get; }
}
```

Затем просто проверяем обновление действия, чтобы узнать, добавлен ли пользователь в диалог, и отправляем ему приветственное сообщение.

```csharp
public class WelcomeUserBot : IBot
{
// Generic message to be sent to user
private const string _genericMessage = @"This is a simple Welcome Bot sample. You can say 'intro' to 
                                         see the introduction card. If you are running this bot in the Bot 
                                         Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel";

// The bot state accessor object. Use this to access specific state properties.
private readonly WelcomeUserStateAccessors _welcomeUserStateAccessors;

// Initializes a new instance of the <see cref="WelcomeUserBot"/> class.
public WelcomeUserBot(WelcomeUserStateAccessors statePropertyAccessor)
{
    _welcomeUserStateAccessors = statePropertyAccessor ?? throw new System.ArgumentNullException("state accessor can't be null");
}

// Every conversation turn for our WelcomeUser Bot will call this method, including
// any type of activities such as ConversationUpdate or ContactRelationUpdate which
// are sent when a user joins a conversation.
// This bot doesn't use any dialogs; it's "single turn" processing, meaning a single
// request and response.
// This bot uses UserState to keep track of first message a user sends.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // Use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.DidBotWelcomeUser.GetAsync(turnContext, () => false);

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Your bot should proactively send a welcome message to a personal chat the first time
        // (and only the first time) a user initiates a personal chat with your bot.
        if (didBotWelcomeUser == false)
        {
            // Update user state flag to reflect bot handled first user interaction.
            await _welcomeUserStateAccessors.DidBotWelcomeUser.SetAsync(turnContext, true);
            await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);

            // the channel should sends the user name in the 'From' object
            var userName = turnContext.Activity.From.Name;

            await turnContext.SendActivityAsync($"You are seeing this message because this was your first message ever to this bot.", cancellationToken: cancellationToken);
            await turnContext.SendActivityAsync($"It is a good practice to welcome the user and provide a personal greeting. For example, welcome {userName}.", cancellationToken: cancellationToken);
        }
    }
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate) // Greet when users are added to the conversation.
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            // Iterate over all new members added to the conversation
            foreach (var member in turnContext.Activity.MembersAdded)
            {
                // Greet anyone that was not the target (recipient) of this message
                // the 'bot' is the recipient for events from the channel,
                // turnContext.Activity.MembersAdded == turnContext.Activity.Recipient.Id indicates the
                // bot was added to the conversation.
                if (member.Id != turnContext.Activity.Recipient.Id)
                {
                    await turnContext.SendActivityAsync($"Hi there - {member.Name}. Welcome to the 'Welcome User' Bot. This bot will introduce you to welcoming and greeting users.", cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync($"You are seeing this message because the bot recieved atleast one 'ConversationUpdate' event,indicating you (and possibly others) joined the conversation. If you are using the emulator, pressing the 'Start Over' button to trigger this event again. The specifics of the 'ConversationUpdate' event depends on the channel. You can read more information at https://aka.ms/about-botframewor-welcome-user", cancellationToken: cancellationToken);
                    await turnContext.SendActivityAsync($"It is a good pattern to use this event to send general greeting to user, explaning what your bot can do. In this example, the bot handles 'hello', 'hi', 'help' and 'intro. Try it now, type 'hi'", cancellationToken: cancellationToken);
                }
            }
        }
    }
    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В этом коде JavaScript приветственное сообщение отправляется при добавлении пользователя. Для этого мы проверяем, является ли диалог активным и добавлен ли новый участник в диалог.

``` javascript
// Import required Bot Framework classes.
const { ActivityTypes } = require('botbuilder');
const { CardFactory } = require('botbuilder');

// Welcomed User property name
const WELCOMED_USER = 'DidBotWelcomeUser';

class MainDialog {
    constructor (userState) {
        // Creates a new user property accessor.
        // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
        this.welcomedUserPropery = userState.createProperty(WELCOMED_USER);
    }

    async onTurn(turnContext) 
    {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) 
        {
            // Read UserState. If the 'DidBotWelcomeUser' does not exist (first time ever for a user)
            // set the default to false.
            let didBotWelcomeUser = await this.welcomedUserPropery.get(turnContext, false);

            // Your bot should proactively send a welcome message to a personal chat the first time
            // (and only the first time) a user initiates a personal chat with your bot.
            if (didBotWelcomeUser === false) 
            {
                // The channel should send the user name in the 'From' object
                let userName = turnContext.activity.from.name;
                await turnContext.sendActivity("You are seeing this message because this was your first message ever sent to this bot.");
                await turnContext.sendActivity(`It is a good practice to welcome the user and provdie personal greeting. For example, welcome ${userName}.`);
                
                // Set the flag indicating the bot handled the user's first message.
                await this.welcomedUserPropery.set(turnContext, true);
            }
            . . .
            
            // Save state changes
            await this.userState.saveChanges(turnContext);
        } else if (turnContext.activity.type === ActivityTypes.ConversationUpdate) {
            // Send greeting when users are added to the conversation.
            await this.sendWelcomeMessage(turnContext);
        } else {
            // Generic message for all other activities
            await turnContext.sendActivity(`[${ turnContext.activity.type } event detected]`);
        }
    }
    
    // Sends welcome messages to conversation members when they join the conversation.
    // Messages are only sent to conversation members who aren't the bot.
    async sendWelcomeMessage(turnContext) {
        // If any new membmers added to the conversation
        if (turnContext.activity && turnContext.activity.membersAdded) {
            // Define a promise that will welcome the user
            async function welcomeUserFunc(conversationMember) {
                // Greet anyone that was not the target (recipient) of this message.
                // The bot is the recipient of all events from the channel, including all ConversationUpdate-type activities
                // turnContext.activity.membersAdded !== turnContext.activity.aecipient.id indicates 
                // a user was added to the conversation 
                if (conversationMember.id !== this.activity.recipient.id) {
                    // Because the TurnContext was bound to this function, the bot can call
                    // `TurnContext.sendActivity` via `this.sendActivity`;
                    await this.sendActivity(`Welcome to the 'Welcome User' Bot. This bot will introduce you to welcoming and greeting users.`);
                    await this.sendActivity("You are seeing this message because the bot recieved atleast one 'ConversationUpdate'" + 
                                            "event,indicating you (and possibly others) joined the conversation. If you are using the emulator, "+ 
                                            "pressing the 'Start Over' button to trigger this event again. The specifics of the 'ConversationUpdate' "+
                                            "event depends on the channel. You can read more information at https://aka.ms/about-botframewor-welcome-user");
                    await this.sendActivity(`It is a good pattern to use this event to send general greeting to user, explaining what your bot can do. `+ 
                                            `In this example, the bot handles 'hello', 'hi', 'help' and 'intro. ` +
                                            `Try it now, type 'hi'`);
                }
            }
    
            // Prepare Promises to greet the  user.
            // The current TurnContext is bound so `replyForReceivedAttachments` can also send replies.
            const replyPromises = turnContext.activity.membersAdded.map(welcomeUserFunc.bind(turnContext));
            await Promise.all(replyPromises);
        }
    }
}

module.exports = MainDialog;
```

---

## <a name="discard-initial-user-input"></a>Отмена исходного сообщения пользователя
Кроме того, необходимо учитывать, что введенные пользователем данные могут содержать ценную информацию, которая также по-разному интерпретируются каналами. Для удобства работы пользователей на всех каналах мы не обрабатываем недопустимые ответы. Для этого мы запрашиваем ввод данных и задаем ключевые слова для поиска в ответах пользователей.

## <a name="ctabcsharpmulti"></a>[C#](#tab/csharpmulti)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = new CancellationToken())
{
    // Use state accessor to extract the didBotWelcomeUser flag
    var didBotWelcomeUser = await _welcomeUserStateAccessors.DidBotWelcomeUser.GetAsync(turnContext, () => false);

    if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Your bot should proactively send a welcome message to a personal chat the first time
        // (and only the first time) a user initiates a personal chat with your bot.
        if (didBotWelcomeUser == false)
        {
            // Previous Code Sample
        }
        else
        {
            // This example hardcodes specific uterances. You should use LUIS or QnA for more advance language understanding.
            var text = turnContext.Activity.Text.ToLowerInvariant();
            switch (text)
            {
                case "hello":
                case "hi":
                    await turnContext.SendActivityAsync($"You said {text}.", cancellationToken: cancellationToken);
                    break;
                case "intro":
                case "help":
                default:
                    await turnContext.SendActivityAsync(_genericMessage, cancellationToken: cancellationToken);
                    break;
            }
        }
    }
    else if (turnContext.Activity.Type == ActivityTypes.ConversationUpdate) // Greet when users are added to the conversation.
    {
        if (turnContext.Activity.MembersAdded.Any())
        {
            // Iterate over all new members added to the conversation
            foreach (var member in turnContext.Activity.MembersAdded)
            {
                // Previous Code Sample
            }
        }
    }
    else
    {
        // Default behaivor for all other type of events.
        var ev = turnContext.Activity.AsEventActivity();
        await turnContext.SendActivityAsync($"Received event: {ev.Name}");
    }
    // save any state changes made to your state objects.
    await _welcomeUserStateAccessors.UserState.SaveChangesAsync(turnContext);
}

```

## <a name="javascripttabjsmulti"></a>[JavaScript](#tab/jsmulti)

``` javascript
class MainDialog 
{ 
    // Previous Code Sample
    
    async onTurn(turnContext) 
    {
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        if (turnContext.activity.type === ActivityTypes.Message) {
            
            // Previous Code Sample
            
            if (didBotWelcomeUser === false) 
            {
                // Previous Code Sample
            }
            else 
            {
                // This example hardcodes specific uterances. You should use LUIS or QnA for more advance language understanding.
                let text = turnContext.activity.text.toLowerCase();
                switch (text) 
                {
                    case "hello":
                    case "hi":
                        await turnContext.sendActivity(`You said "${turnContext.activity.text}"`);
                        break;
                    case "intro":
                    case "help":
                    default :
                        await turnContext.sendActivity(`This is a simple Welcome Bot sample. You can say 'intro' to 
                                                        see the introduction card. If you are running this bot in the Bot 
                                                        Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel`);
                }
            }
        }
       
       // Previous Sample Code
    } 
}
```

---

## <a name="using-adaptive-card-greeting"></a>Использование приветственных адаптивных карточек

Еще один способ поприветствовать пользователя — использовать адаптивные карточки. Дополнительные сведения о таких приветственных адаптивных карточках см. в разделе [Отправка адаптивной карточки](./bot-builder-howto-add-media-attachments.md).

## <a name="ctabcsharpwelcomeback"></a>[C#](#tab/csharpwelcomeback)

```csharp
// Sends an adaptive card greeting.
private static async Task SendIntroCardAsync(ITurnContext turnContext, CancellationToken cancellationToken)
{
    var response = turnContext.Activity.CreateReply();

    var introCard = File.ReadAllText(@".\Resources\IntroCard.json");

    response.Attachments = new List<Attachment>
    {
        new Attachment()
        {
            ContentType = "application/vnd.microsoft.card.adaptive",
            Content = JsonConvert.DeserializeObject(introCard),
        },
    };

    await turnContext.SendActivityAsync(response, cancellationToken);
}
```

Теперь можно отправить карточку с помощью приведенной ниже команды await. Давайте добавим этот пример в раздел _switch (text) case "help"_ в коде бота.

```csharp
switch (text)
{
    case "hello":"
    case "hi":
        await turnContext.SendActivityAsync($"You said {text}.", cancellationToken: cancellationToken);
        break;
    case "intro":
    case "help":
        await SendIntroCardAsync(turnContext, cancellationToken);
        break;
    default:
        await turnContext.SendActivityAsync(_genericMessage, cancellationToken: cancellationToken);
        break;
}
```


## <a name="javascripttabjswelcomeback"></a>[JavaScript](#tab/jswelcomeback)

Сначала добавим нашу адаптивную карточку в раздел Imports в начале файла бота _index.js_.

``` javascript
// Adaptive Card content
const IntroCard = require('./Resources/IntroCard.json');
```

Затем мы просто используем код бота в разделе _switch (text)_ _case "help"_, как показано ниже, чтобы отправить пользователю ответ с адаптивной карточкой.

``` javascript
switch (text) 
{
    case "hello":
    case "hi":
        await turnContext.sendActivity(`You said "${turnContext.activity.text}"`);
        break;
    case "intro":
    case "help":
        await turnContext.sendActivity({
            text: 'Intro Adaptive Card',
            attachments: [CardFactory.adaptiveCard(IntroCard)]
        });
        break;
    default :
        await turnContext.sendActivity(`This is a simple Welcome Bot sample. You can say 'intro' to 
                                        see the introduction card. If you are running this bot in the Bot 
                                        Framework Emulator, press the 'Start Over' button to simulate user joining a bot or a channel`);
}
```
---
## <a name="test-the-bot"></a>Тестирование бота
В файле [README](https://github.com/Microsoft/BotBuilder-Samples/blob/master/samples/csharp_dotnetcore/03.welcome-user/readme.md) вы найдете инструкции по запуску и тестированию бота. 

## <a name="next-steps"></a>Дополнительная информация
> [!div class="nextstepaction"]
> [Сбор вводимых пользователем данных](bot-builder-prompts.md)

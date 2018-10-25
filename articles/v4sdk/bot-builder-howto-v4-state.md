---
title: Управление состоянием диалога и пользователя | Документация Майкрософт
description: Узнайте, как сохранять и извлекать данные о состоянии с помощью пакета SDK Bot Builder для .NET.
keywords: conversation state, user state, conversation flow
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 21f864ba6f5beba5205e860f4a56697997048dfb
ms.sourcegitcommit: 6c2426c43cd2212bdea1ecbbf8ed245145b3c30d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/08/2018
ms.locfileid: "48852299"
---
# <a name="manage-conversation-and-user-state"></a>Управление состоянием диалога и пользователя

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Важным аспектом качественной разработки бота является отслеживание контекста общения, то есть бот должен запоминать, например, ответы на предыдущие вопросы. В зависимости от того, для чего используется ваш бот, вам может понадобиться отслеживать информацию о состоянии или хранении данных дольше, чем время существования общения. *Состояние* бота — это информация, которую программа запоминает, чтобы адекватно отвечать на входящие сообщения. Пакет SDK Bot Builder предоставляет два класса для хранения и извлечения данных о состоянии в качестве объекта, связанного с пользователем или беседой.

- **Состояние беседы** помогает боту отслеживать текущую беседу с пользователем. Если ваш бот должен выполнить последовательные шаги или переключиться между темами общения, вы можете использовать свойства общения для управления шагами в последовательности или для отслеживания текущей темы. 

- **Состояние пользователя** можно применять в различных целях, например для определения места остановки предыдущей беседы с пользователем или просто для приветствия вернувшегося пользователя по имени. Если настройки пользователя сохраняются, можно использовать эту информацию для настройки диалога в следующий раз. Например, можно оповестить пользователя о новостной статье на интересующую его тему или оповестить его о том, что освободилось время для встречи. 

`ConversationState` и `UserState` — это классы состояния, являющиеся специализациями класса `BotState` с политиками, которые контролируют время существования и количество хранящихся в них объектов. Компоненты, которым нужно хранить состояние, создают и регистрируют свойство с определенным типом и используют метод доступа к свойству для доступа к состоянию. Диспетчер состояний бота может использовать память, хранилище, CosmosDB и хранилище BLOB-объектов. 

> [!NOTE] 
> Используйте диспетчер состояний бота для хранения настроек, имени пользователя или сведений о последнем заказе, но не используйте его для хранения критически важных бизнес-данных. Для критически важных данных создайте собственные компоненты хранилища или записывайте их непосредственно в [хранилище](bot-builder-howto-v4-storage.md).
> Хранилище данных в памяти предназначено только для тестирования. Это временное и нестабильное хранилище. Данные очищаются при каждом перезапуске бота.

## <a name="using-conversation-state-and-user-state-to-direct-conversation-flow"></a>Использование состояния беседы и состояния пользователя для направления процесса общения
При разработке последовательности общения полезно определить флаг состояния для направления последовательности общения. Флаг может быть простого логического типа или типа, который включает имя текущего раздела. Флаг может помочь отслеживать местонахождение в общении. Например, флаг логического типа позволяет узнать, участвуете ли вы в беседе или нет, в то время как свойство имени раздела может сообщить, в какой беседе вы сейчас находитесь.



# <a name="ctabcsharp"></a>[C#](#tab/csharp)
### <a name="conversation-and-user-state"></a>Состояние беседы и пользователя
В качестве отправной точки в этом руководстве можно использовать [пример Echo Bot With Counter](https://aka.ms/EchoBot-With-Counter). Сначала создайте класс `TopicState` для управления текущим разделом беседы в файле `TopicState.cs`, как показано ниже:

```csharp
public class TopicState
{
   public string Prompt { get; set; } = "askName";
}
``` 
Затем создайте класс `UserProfile` в файле `UserProfile.cs` для управления состоянием пользователя.
```csharp
public class UserProfile
{
    public string UserName { get; set; }
    public string TelephoneNumber { get; set; }
}
``` 
Класс `TopicState` имеет флаг для отслеживания места в беседе и для его хранения использует состояние беседы. Запрос инициализируется значением askName, что позволяет начать беседу. После получения ботом ответа от пользователя запрос будет переопределен с использованием значения askNumber для начала следующей беседы. Класс `UserProfile` отслеживает имя и номер телефона пользователя и хранит эти данные в состоянии пользователя.

### <a name="property-accessors"></a>Методы доступа к свойствам
Класс `EchoBotAccessors` в нашем примере создается как шаблон singleton и передается в конструктор `class EchoWithCounterBot : IBot` путем внедрения зависимостей. Конструктор класса `EchoBotAccessors` инициализирует новый экземпляр класса `EchoBotAccessors`. Он содержит классы `ConversationState` и `UserState` и связанный с ними класс `IStatePropertyAccessor`. Объект `conversationState` хранит состояние раздела и объект `userState`, в котором хранятся сведения о профиле пользователя. Объекты `ConversationState` и `UserState` создаются в файле Startup.cs. В объектах состояния беседы и пользователя мы сохраняем все сведения из области беседы и пользователя. 

Обновите конструктор, чтобы включить `UserState`, как показано ниже:
```csharp
using EchoBotWithCounter;

public EchoBotAccessors(ConversationState conversationState, UserState userState)
{
    ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    UserState = userState ?? throw new ArgumentNullException(nameof(userState));
}
```
Создайте уникальные имена для методов доступа к `TopicState` и `UserProfile`.
```csharp
public static string UserProfileName { get; } = $"{nameof(EchoBotAccessors)}.UserProfile";
public static string TopicStateName { get; } = $"{nameof(EchoBotAccessors)}.TopicState";
```
Затем определите два метода доступа. В первом хранится тема беседы, во втором — имя и номер телефона пользователя.
```csharp
public IStatePropertyAccessor<TopicState> TopicState { get; set; }
public IStatePropertyAccessor<UserProfile> UserProfile { get; set; }
```

Свойства для получения класса ConversationState уже определены, но необходимо добавить `UserState`, как показано ниже:
```csharp
public ConversationState ConversationState { get; }
public UserState UserState { get; }
```
Закончив вносить изменения, сохраните файл. После этого мы обновим класс Startup, чтобы создать объект `UserState` для сохранения любых сведений из области пользователя. `ConversationState` уже существует. 
```csharp

services.AddBot<EchoWithCounterBot>(options =>
{
    ...

    IStorage dataStore = new MemoryStorage();
    
    var conversationState = new ConversationState(dataStore);
    options.State.Add(conversationState);
        
    var userState = new UserState(dataStore);  
    options.State.Add(userState);
});
```
Строки `options.State.Add(ConversationState);` и `options.State.Add(userState);` добавляют состояние беседы и состояние пользователя соответственно. Затем создайте и зарегистрируйте методы доступа к состоянию. Создаваемые здесь методы доступа при каждой реплике передаются в класс, который является производным от IBot. Измените код таким образом, чтобы он включал состояние пользователя, как показано ниже:
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var userState = options.State.OfType<UserState>().FirstOrDefault();
    if (userState == null)
    {
        throw new InvalidOperationException("UserState must be defined and added before adding user-scoped state accessors.");
    }
   ...
 });
```

Затем создайте два метода доступа с использованием `TopicState` и `UserProfile` и передайте их в класс `class EchoWithCounterBot : IBot` путем внедрения зависимостей.
```csharp
services.AddSingleton<EchoBotAccessors>(sp =>
{
   ...
    var accessors = new BotAccessors(conversationState, userState)
    {
        TopicState = conversationState.CreateProperty<TopicState>("TopicState"),
        UserProfile = userState.CreateProperty<UserProfile>("UserProfile"),
     };

     return accessors;
 });
```

Состояния диалога и пользователя связаны с шаблоном singleton с помощью блока кода `services.AddSingleton` и сохранены с помощью метода доступа к хранилищу состояний в коде, начинающемся со строки `var accessors = new BotAccessor(conversationState, userState)`.

### <a name="use-conversation-and-user-state-properties"></a>Использование свойств состояния общения и пользователя 
В обработчике `OnTurnAsync` класса `EchoWithCounterBot : IBot` измените код таким образом, чтобы запросить имя пользователя, а затем его номер телефона. Для отслеживания места остановки беседы мы используем свойство Prompt, определенное в классе TopicState. Это свойство было инициализировано значением askName. После получения имени пользователя для этого свойства задается значение askNumber, а в качестве значения UserName устанавливается введенное пользователем имя. После получения номера телефона вы отправляете сообщение с подтверждением и устанавливаете для запроса значение confirmation, так как это конец беседы.

```csharp
using EchoBotWithCounter;

if (turnContext.Activity.Type == ActivityTypes.Message)
{
    // Get the conversation state from the turn context.
    var convo = await _accessors.TopicState.GetAsync(turnContext, () => new TopicState());
    
    // Get the user state from the turn context.
    var user = await _accessors.UserProfile.GetAsync(turnContext, () => new UserProfile());
    
    // Ask user name. The Prompt was initialiazed as "askName" in the TopicState.cs file.
    if (convo.Prompt == "askName")
    {
        await turnContext.SendActivityAsync("What is your name?");
        
        // Set the Prompt to ask the next question for this conversation
        convo.Prompt = "askNumber";
        
        // Set the property using the accessor
        await _accessors.TopicState.SetAsync(turnContext, convo);
        
        //Save the new prompt into the conversation state.
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "askNumber")
    {
        // Set the UserName that is defined in the UserProfile class
        user.UserName = turnContext.Activity.Text;
        
        // Use the user name to prompt the user for phone number
        await turnContext.SendActivityAsync($"Hello, {user.UserName}. What's your telephone number?");
        
        // Set the Prompt now that we have collected all the data
        convo.Prompt = "confirmation";
                 
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfile.SetAsync(turnContext, user);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
    else if (convo.Prompt == "confirmation")
    { 
        // Set the TelephoneNumber that is defined in the UserProfile class
        user.TelephoneNumber = turnContext.Activity.Text;

        await turnContext.SendActivityAsync($"Got it, {user.UserName}. I'll call you later.");

        // reset initial prompt state
        convo.Prompt = "askName"; // Reset for a new conversation.
        
        await _accessors.TopicState.SetAsync(turnContext, convo);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="conversation-and-user-state"></a>Состояние беседы и пользователя

В качестве отправной точки в этом руководстве можно использовать [пример Echo Bot With Counter](https://aka.ms/EchoBot-With-Counter-JS). В этом примере уже используется `ConversationState` для хранения данных о количестве сообщений. Нужно будет добавить объект `TopicStates` для отслеживания состояния беседы и `UserState` — для отслеживания информации о пользователе в объекте `userProfile`. 

В главном файле `index.js` бота добавьте `UserState` в список обязательных элементов:

**index.js**

```javascript
// Import required bot services. See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState } = require('botbuilder');
```

Затем создайте `UserState` с помощью `MemoryStorage` в качестве поставщика хранилища, а затем передайте его в качестве второго аргумента в класс `MainDialog`.

**index.js**

```javascript
// Create conversation state with in-memory storage provider. 
const conversationState = new ConversationState(memoryStorage);
const userState = new UserState(memoryStorage);
// Create the main bot.
const bot = new EchBot(conversationState, userState);
```

В файле `bot.js` обновите конструктор, чтобы он принял `userState` в качестве второго аргумента. Затем создайте свойство `topicState` из `conversationState` и свойство `userProfile` из `userState`.

**bot.js**

```javascript
const TOPIC_STATE = 'topic';
const USER_PROFILE = 'user';

constructor (conversationState, userState) {
    // creates a new state accessor property.see https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors 
    this.conversationState = conversationState;
    this.topicState = this.conversationState.createProperty(TOPIC_STATE);

    // User state
    this.userState = userState;
    this.userProfile = this.userState.createProperty(USER_PROFILE);
}
```

### <a name="use-conversation-and-user-state-properties"></a>Использование свойств состояния общения и пользователя

В обработчике `onTurn` класса `MainDialog` измените код таким образом, чтобы запросить имя пользователя, а затем его номер телефона. Для отслеживания места остановки беседы мы используем свойство `prompt`, определенное в `topicState`. Это свойство инициализируется значением askName. После получения имени пользователя для этого свойства задается значение askNumber, а в качестве значения UserName устанавливается введенное пользователем имя. После получения номера телефона вы отправляете сообщение с подтверждением и устанавливаете для запроса значение `undefined`, так как это конец беседы.

**dialogs/mainDialog/index.js**

```javascript
// see https://aka.ms/about-bot-activity-message to learn more about the message and other activity types
if (turnContext.activity.type === 'message') {
    // read from state and set default object if object does not exist in storage.
    let topicState = await this.topicState.get(turnContext, {
        //Define the topic state object
        prompt: "askName"
    });
    let userProfile = await this.userProfile.get(turnContext, {  
        // Define the user's profile object
        "userName": "",
        "telephoneNumber": ""
    });

    if(topicState.prompt == "askName"){
        await turnContext.sendActivity("What is your name?");

        // Set next prompt state
        topicState.prompt = "askNumber";

        // Update state
        await this.topicState.set(turnContext, topicState);
    }
    else if(topicState.prompt == "askNumber"){
        // Set the UserName that is defined in the UserProfile class
        userProfile.userName = turnContext.activity.text;

        // Use the user name to prompt the user for phone number
        await turnContext.sendActivity(`Hello, ${userProfile.userName}. What's your telephone number?`);

        // Set next prompt state
        topicState.prompt = "confirmation";

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    else if(topicState.prompt == "confirmation"){
        // Set the phone number
        userProfile.telephoneNumber = turnContext.activity.text;

        // Sent confirmation
        await turnContext.sendActivity(`Got it, ${userProfile.userName}. I'll call you later.`)

        // reset initial prompt state
        topicState.prompt = "askName"; // Reset for a new conversation

        // Update states
        await this.topicState.set(turnContext, topicState);
        await this.userProfile.set(turnContext, userProfile);
    }
    
    // Save state changes to storage
    await this.conversationState.saveChanges(turnContext);
    await this.userState.saveChanges(turnContext);
    
}
else {
    await turnContext.sendActivity(`[${context.activity.type} event detected]`);
}
```

---

## <a name="start-your-bot"></a>Запуск бота
Запустите бот на локальном компьютере.

### <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота
После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Open Bot** (Открыть бот) на вкладке приветствия в эмуляторе. 
2. Выберите BOT-файл, расположенный в каталоге с созданным решением Visual Studio.

### <a name="interact-with-your-bot"></a>Взаимодействие с ботом

Отправьте сообщение боту и получите от него сообщение в ответ.
![Работающий эмулятор](../media/emulator-v4/emulator-running.png)

Если вы захотите управлять состоянием самостоятельно, см. статью [Создание собственных запросов на ввод данных пользователем](bot-builder-primitive-prompts.md). Альтернативой является использование каскадного диалога. Диалоговое окно отслеживает состояние общения, поэтому нет необходимости создавать флаги для отслеживания состояния. Дополнительные сведения см. в разделе [Управление простым процессом общения с помощью диалогов](bot-builder-dialog-manage-conversation-flow.md).

## <a name="next-steps"></a>Дополнительная информация
Теперь, когда вы знаете, как использовать состояние для чтения и записи данных бота в хранилище, давайте взглянем на то, как можно читать и записывать непосредственно в хранилище.

> [!div class="nextstepaction"]
> [Сохранение данных напрямую в хранилище](bot-builder-howto-v4-storage.md).

---
title: Сохранение состояния с помощью свойств общения и пользователя | Документация Майкрософт
description: Узнайте, как сохранять и извлекать данные с помощью пакета SDK Bot Builder версии 4 для .NET.
keywords: состояние общения, состояние пользователя, ПО промежуточного слоя, последовательность общения, хранилище файлов, хранилище таблиц Azure
author: ivorb
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/03/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 16df371b1cabb4b3eb47d1f491a5d45e26627d38
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352853"
---
# <a name="save-state-using-conversation-and-user-properties"></a>Сохранение состояния с помощью свойств общения и пользователя

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Чтобы бот сохранял общение и пользовательское состояние, сначала инициализируйте ПО промежуточного слоя диспетчера состояний, а затем используйте свойства общения и состояния пользователя.
Дополнительные сведения об использовании состояния см. в разделе [Состояние и хранилище](./bot-builder-storage-concept.md).

## <a name="initialize-state-manager-middleware"></a>Инициализация ПО промежуточного слоя диспетчера состояний

В пакете SDK необходимо инициализировать адаптер бота для использования ПО промежуточного слоя диспетчера состояний, прежде чем можно будет использовать хранилище свойств общения или пользователя. _Состояние общения_ используется для свойств общения, а _состояние пользователя_ — для свойств пользователя. (Доступ к свойствам состояния пользователя можно получить через несколько сеансов общения.) ПО промежуточного слоя диспетчера состояний обеспечивает абстракцию, позволяющую получить доступ к свойствам с помощью простого хранилища ключей или хранилища объектов, независимо от типа базового хранилища. Диспетчер состояний отвечает за записи данных для хранения и управления параллелизмом, независимо от того, является ли базовым тип хранилища в памяти, хранилище файлов или Хранилище таблиц Azure.


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
Дополнительные сведения об инициализации `ConversationState` см. в коде `Startup.cs` примера Microsoft.Bot.Samples.EchoBot-AspNetCore.

```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

    IStorage dataStore = new MemoryStorage();
    options.Middleware.Add(new ConversationState<EchoState>(dataStore));
});
```

В строке `options.Middleware.Add(new ConversationState<EchoState>(dataStore));` `ConversationState` — это объект диспетчера состояния общения, который добавляется к боту в качестве ПО промежуточного слоя. Параметр типа `EchoState` — это тип, представляющий способ хранения информации о состоянии общения. Бот может использовать любой тип класса для данных об общении или состоянии пользователя.

Реализация `EchoState` находится в `EchoBot.cs`.
```csharp
public class EchoState
{
    public int TurnNumber { get; set; }
}
``` 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Определите поставщик хранилища и назначьте его диспетчеру состояния, который будет использоваться.
Можно использовать один и тот же поставщик хранилища для ПО промежуточного слоя управления `ConversationState` и `UserState`.
Затем нужно использовать библиотеку `BotStateSet` для подключения к ПО промежуточного слоя, которое будет управлять хранением данных.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware.
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Alternatively, use both conversation and user state middleware.
// const storage = new MemoryStorage;
// const conversationState = new ConverstationState(storage);
// const userState = new UserState(storage);
// adapter.use(new BotStateSet(conversationState, userState));
```    
---

> [!NOTE] 
> Хранилище данных в памяти предназначено только для тестирования. Это временное и нестабильное хранилище. Данные очищаются при каждом перезапуске бота. Чтобы настроить другие базовые носители данных для состояния общения и состояния пользователя, см. далее статьи [Хранилище файлов](#file-storage) и [Хранилище таблиц Azure](#azure-table-storage). 

### <a name="configuring-state-manager-middleware"></a>Настройка ПО промежуточного слоя диспетчера состояний

При инициализации ПО промежуточного слоя состояния необязательный параметр _состояния_ позволяет изменить поведение сохранения свойств по умолчанию. Параметры по умолчанию:

* Сохранять свойства за пределами времени существования контекста.
* Если свойство записывают более одного экземпляра бота, разрешите последнему экземпляру бота перезаписать предыдущее.

## <a name="use-conversation-and-user-state-properties"></a>Использование свойств состояния общения и пользователя 
<!-- middleware and message context properties -->

После настройки ПО промежуточного слоя диспетчера состояний можно получить свойства состояния общения и пользователя из объекта контекста.
<!-- Changes are written to storage before the `SendActivity()` pipeline completes. -->

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Используя пример `Microsoft.Bot.Samples.EchoBot` в пакете SDK для Bot Builder, можно увидеть, как это работает. 

В обработчике `OnTurn` параметр `context.GetConversationState` получает состояние общения для доступа к данным, которые были определены, состояние также можно изменить с помощью свойств (в этом случае, увеличивая `TurnNumber`).

```csharp
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // Bump the turn count. 
        state.TurnCount++;

        // Echo back to the user whatever they typed.
        await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
    }
}
```   

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Используя созданный EchoBot из примера генератора Yeoman, можно увидеть, как это работает.

В этом примере кода показано, как можно хранить счетчик включений в состоянии общения.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="using-conversation-state-to-direct-conversation-flow"></a>Использование состояния общения для направления последовательности общения

При разработке последовательности общения полезно определить флаг состояния для направления последовательности общения. Флаг может быть простого **логического** типа или типа, который включает имя текущего раздела. Флаг может помочь отслеживать местонахождение в общении. Например, флаг **логического** типа позволяет узнать, находитесь ли вы в общении или нет, в то время как свойство имени раздела может сообщить, в каком из этих сеансов общения в текущий момент вы находитесь.

В следующем примере логическое свойство _наличия запрошенного имени_ указывает, когда бот спросил у пользователя его имя. При получении следующего сообщения бот проверяет свойство. Если установлено значение `true`, бот знает, что у пользователя было запрошено его имя, и он интерпретирует входящее сообщение как имя для сохранения в качестве свойства пользователя.


# <a name="ctabcsharp"></a>[C#](#tab/csharp)
```csharp
public class ConversationInfo
{
    public bool haveAskedNameFlag { get; set; }
    public bool haveAskedNumberFlag { get; set; }
}

public class UserInfo
{
    public string name { get; set; }
    public string telephoneNumber { get; set; }
    public bool done { get; set; }
}

public async Task OnTurn(ITurnContext context)
{
    // Get state objects. Default objects are created if they don't already exist.
    var convo = ConversationState<ConversationInfo>.Get(context);
    var user = UserState<UserInfo>.Get(context);

    if (context.Activity.Type is ActivityTypes.Message)
    {
        if (string.IsNullOrEmpty(user.name) && !convo.haveAskedNameFlag)
        {
            // Ask for the name.
            await context.SendActivity("Hello. What's your name?");

            // Set flag to show we've asked for the name. We save this out so the
            // context object for the next turn of the conversation can check haveAskedName
            convo.haveAskedNameFlag = true;
        }
        else if (!convo.haveAskedNumberFlag)
        {
            // Save the name.
            var name = context.Activity.AsMessageActivity().Text;
            user.name = name;
            convo.haveAskedNameFlag = false; // Reset flag

            // Ask for the phone number. You might want a flag to track this, too.
            await context.SendActivity($"Hello, {name}. What's your telephone number?");
            convo.haveAskedNumberFlag = true;
        }
        else if (convo.haveAskedNumberFlag)
        {
            // save the telephone number
            var telephonenumber = context.Activity.AsMessageActivity().Text;

            user.telephoneNumber = telephonenumber;
            convo.haveAskedNumberFlag = false; // Reset flag
            await context.SendActivity($"Got it. I'll call you later.");
        }
    }
}
```

Чтобы настроить состояние пользователя так, чтобы оно могло быть возвращено командой `UserState<UserInfo>.Get(context)`, добавьте ПО промежуточного слоя состояния пользователя. Например, в `Startup.cs` ASP.NET Core EchoBot изменяя код в ConfigureServices.cs:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        
        IStorage dataStore = new MemoryStorage();
        options.Middleware.Add(new ConversationState<ConversationInfo>(dataStore));
        options.Middleware.Add(new UserState<UserInfo>(dataStore));
    });
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**app.js**

```js
const { BotFrameworkAdapter, MemoryStorage, ConversationState, UserState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter (it's ok for MICROSOFT_APP_ID and MICROSOFT_APP_PASSWORD to be blank for now)  
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Storage
const storage = new MemoryStorage();
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        const convo = conversationState.get(context);
        const user = userState.get(context);

        if (isMessage) {
            if(!user.name && !convo.haveAskedNameFlag){
                // Ask for the name.
                await context.sendActivity("What is your name?")
                // Set flag to show we've asked for the name. We save this out so the
                // context object for the next turn of the conversation can check haveAskedNameFlag
                convo.haveAskedNameFlag = true;
            } else if(convo.haveAskedNameFlag){
                // Save the name.
                user.name = context.activity.text;
                convo.haveAskedNameFlag = false; // Reset flag

                await context.sendActivity(`Hello, ${user.name}. What's your telephone number?`);
                convo.haveAskedNumberFlag = true; // Set flag
            } else if(convo.haveAskedNumberFlag){
                // save the phone number
                user.telephonenumber = context.activity.text;
                convo.haveAskedNumberFlag = false; // Reset flag
                await context.sendActivity(`Got it. I'll call you later.`);
            }
        }

        // ...
    });
});

```

---

Альтернативой является использование _каскадной_ модели диалога. Диалоговое окно отслеживает состояние общения, поэтому нет необходимости создавать флаги для отслеживания состояния. Дополнительные сведения см. в разделе [Управление общением с помощью диалогов](bot-builder-dialog-manage-conversation-flow.md).

## <a name="file-storage"></a>Хранилище файлов

Поставщик хранилища памяти использует хранилище в памяти, которое удаляется при перезапуске бота. Это рекомендуется только для тестирования. Если необходимо сохранить данные без подключения бота к базе данных, можно использовать поставщик хранилища файлов. Хотя этот поставщик также предназначен для тестирования, он сохраняет данные состояния в файл, чтобы его можно было проверить. Данные записываются в файл в формате JSON.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Перейдите к `Startup.cs` в примере Microsoft.Bot.Samples.EchoBot-AspNetCore и отредактируйте код в методе `ConfigureServices`.
```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // Using file storage instead of in-memory storage.
        IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());
        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
``` 

Запустите код и позвольте EchoBot повторить ввод данных несколько раз.

Перейдите в каталог, указанный `System.IO.Path.GetTempPath()`. Вы увидите файл с именем, начинающимся со слова "conversation". Откройте его и просмотрите JSON. Файл содержит примерно следующий код.
```json
{
  "$type": "Microsoft.Bot.Samples.Echo.EchoState, Microsoft.Bot.Samples.EchoBot",
  "TurnNumber": "3",
  "eTag": "ecfe2a23566b4b52b2fe697cffc59385"
}
```

Параметр `$type` указывает тип структуры данных, которая используется в боте для сохранения состояния общения. Поле `TurnNumber` соответствует свойству `TurnNumber` в классе `EchoState`. Поле `eTag` унаследовано из `IStoreItem` и является уникальным значением, которое автоматически обновляется каждый раз, когда бот обновляет состояние общения.  Поле eTag позволяет боту включить оптимистическую блокировку.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Чтобы использовать `FileStorage`, обновите пример EchoBot, описанный в разделе [Использование свойств состояния общения и пользователя](#use-conversation-and-user-state-properties). Убедитесь, что для `storage` установлено значение `FileStorage` вместо `MemoryStorage` и требуется `FileStorage` из Bot Builder. Это единственные необходимые изменения. 

```javascript
// Storage
const storage = new FileStorage("c:/temp");
const conversationState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(conversationState, userState));
```

Поставщик `FileStorage` использует "path" в качестве параметра. Указанный путь позволяет легко найти файл с сохраненной информацией от бота. Каждое *общение* будет иметь созданный для него новый файл. Таким образом, в *пути* можно найти несколько имен файлов, начиная с `conversation!`. Можно отсортировать по дате, чтобы найти самое последнее общение. С другой стороны, можно найти только один файл для состояния *пользователя*. Имя файла начинается с `user!`. В любое время, когда состояние любого из объектов изменится, диспетчер состояний обновит файл, чтобы он отразил то, что было изменено.

Запустите бот и отправьте ему несколько сообщений. Затем найдите файл хранилища и откройте его. Вот как может выглядеть содержимое JSON для EchoBot, отслеживающего счетчик включений.

```json
{
  "turnNumber": "3",
  "eTag": "322"
}
```
---

## <a name="azure-table-storage"></a>Хранилище таблиц Azure

Также можно использовать хранилище таблиц Azure в качестве носителя данных.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В примере Microsoft.Bot.Samples.EchoBot-AspNetCore добавьте ссылку на пакет NuGet `Microsoft.Bot.Builder.Azure`.

Затем перейдите к `Startup.cs`, добавьте оператор `using Microsoft.Bot.Builder.Azure;` и отредактируйте код в методе `ConfigureServices`.
```csharp
services.AddBot<EchoBot>(options =>
{
    options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
    // The parameters are the connection string and table name.
    // "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
    // Replace it with your own connection string if you're not using the emulator
    options.Middleware.Add(new ConversationState<EchoState>(new AzureTableStorage("UseDevelopmentStorage=true","conversationstatetable")));
    // you could also specify the cloud storage account instead of the connection string
    /* options.Middleware.Add(new ConversationState<EchoState>(
        new AzureTableStorage(WindowsAzure.Storage.CloudStorageAccount.DevelopmentStorageAccount, "conversationstatetable"))); */
    options.EnableProactiveMessages = true;
});
```
`UseDevelopmentStorage=true` — это строка подключения, которую можно использовать с [Эмулятором службы хранилища Azure][AzureStorageEmulator]. Замените ее строкой подключения, если не используется эмулятор.

Если таблица с именем, которое указано в конструкторе для `AzureTableStorage`, не существует, она создается.

<!-- 
TODO: step-by-step inspection of the stored table
-->

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Можно создать ConversationState с помощью `AzureTableStorage` в `app.js` примера EchoBot. 

```bash
npm install --save botbuilder-azure@preview
```

```javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState } = require('botbuilder');
const { TableStorage } = require('botbuilder-azure');

// ...

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add conversation state middleware
// The parameters are the connection string and table name.
// "UseDevelopmentStorage=true" is the connection string to use if you are using the Azure Storage Emulator.
// Replace it with your own connection string if you're not using the emulator
var azureStorage = new TableStorage({ tableName: "TestAzureTable1", storageAccountOrConnectionString: "UseDevelopmentStorage=true"})

// You can alternatively use your account name and table name
// var azureStorage = new TableStorage({tableName: "TestAzureTable2", storageAccessKey: "V3ah0go4DLkMtQKUPC6EbgFeXnE6GeA+veCwDNFNcdE6rqSVE/EQO/kjfemJaitPwtAkmR9lMKLtcvgPhzuxZg==", storageAccountOrConnectionString: "storageaccount"});

const conversationState = new ConversationState(azureStorage);
adapter.use(conversationState);

```

`UseDevelopmentStorage=true` — это строка подключения, которую можно использовать с [Эмулятором службы хранилища Azure][AzureStorageEmulator].
Если таблица с именем, которое указано в конструкторе для `AzureTableStorage`, не существует, она создается.

---

Чтобы просмотреть сохраненные данные состояния общения, запустите пример, а затем откройте таблицу, используя [Обозреватель службы хранилища Azure][AzureStorageExplorer].

![Данные состояния общения EchoBot в Обозревателе службы хранилища Azure](media/how-to-state/echostate-azure-storage-explorer.png)


**Ключ раздела** является уникальным ключом, специфичным для текущего общения. При перезапуске бота или запуске нового общения общение получит строку с собственным ключом раздела. Структура данных `EchoState` сериализуется в JSON и сохраняется в столбце **Json** таблицы Azure. 

```json
{
    "$type":"Microsoft.Bot.Samples.Echo.AspNetCore.EchoState, Microsoft.Bot.Samples.EchoBot-AspNetCore",
    "TurnNumber":2,
    "LastMessage":"second message",
    "eTag":"*"
}
```

Поля `$type` и `eTag` добавляются в пакет SDK для Bot Builder. Дополнительные сведения о eTag см. в разделе [Управление параллелизмом с помощью тегов eTag](bot-builder-howto-v4-storage.md#manage-concurrency-using-etags)


## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знаете, как использовать состояние для чтения и записи данных бота в хранилище, давайте взглянем на то, как можно читать и записывать непосредственно в хранилище.

> [!div class="nextstepaction"]
> [Сохранение данных напрямую в хранилище](bot-builder-howto-v4-storage.md).

## <a name="additional-resources"></a>Дополнительные ресурсы
Дополнительные сведения о хранилище см. в разделе [Save state and access data](bot-builder-storage-concept.md) (Сохранение состояния и доступ к данным)

<!-- Links -->
[AzureStorageEmulator]: https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator
[AzureStorageExplorer]: https://azure.microsoft.com/en-us/features/storage-explorer/

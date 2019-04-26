---
title: Запись данных напрямую в хранилище | Документация Майкрософт
description: Сведения о том, как выполнять операции чтения и записи непосредственно в хранилище с помощью пакета SDK Bot Framework для .NET.
keywords: хранилище, чтение и запись, хранилище в памяти, eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 4/13/19
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1f74e34c0eaf303e612f94605104482cd5f1f080
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904503"
---
# <a name="write-directly-to-storage"></a>Запись данных напрямую в хранилище

[!INCLUDE[applies-to](../includes/applies-to.md)]

Вы можете выполнять операции записи и чтения непосредственно в объекте хранилища, не используя ПО промежуточного слоя или объект контекста. Это может быть необходимо для данных бота, которые поступают из источника за пределами потока общения бота. Например, предположим, что пользователь запрашивает у бота прогноз погоды. Тогда бот извлекает прогноз для указанной даты, считывая его из внешней базы данных. Содержимое базы данных о погоде не зависит от сведений о пользователе или от контекста общения, поэтому вы можете только читать его напрямую из хранилища, вместо использования диспетчера состояния. Примеры кода в этой статье демонстрируют, как выполнять операции записи и чтения в хранилище с использованием **хранилища в памяти**, **Cosmos DB**, **хранилища BLOB-объектов** и **хранилища расшифровок в BLOB-объектах Azure**. 

## <a name="prerequisites"></a>Предварительные требования
- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/en-us/free/) учетную запись Azure, прежде чем начинать работу.
- Установка Bot Framework [Emulator](https://aka.ms/Emulator-wiki-getting-started)

## <a name="memory-storage"></a>Хранилище в памяти

Сначала мы создадим бот, который будет выполнять операции записи и чтения данных в хранилище в памяти. Хранилище в памяти используется только для тестирования и не предназначено для рабочей среды. Прежде чем публиковать бот, настройте использование хранилища в Cosmos DB или хранилища BLOB-объектов.

#### <a name="build-a-basic-bot"></a>Создание базового бота

Остальная часть этой статьи описывает использование эхо-бота. Пример кода эхо-бота для создания этого проекта можно найти здесь: [пример C#](https://aka.ms/cs-echobot-sample) или [примере JS](https://aka.ms/js-echobot-sample). Для подключения к боту, взаимодействия с ним и его тестирования можно использовать эмулятор Bot Framework. В следующем примере каждое сообщение от пользователя добавляется в список. Структура данных, содержащая список, записывается в хранилище.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**MyBot.cs**
```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Create local Memory Storage.
private static readonly MemoryStorage _myStorage = new MemoryStorage();

// Create cancellation token (used by Async Write operation).
public CancellationToken cancellationToken { get; private set; }

// Class for storing a log of utterances (text of messages) as a list.
public class UtteranceLog : IStoreItem
{
     // A list of things that users have said to the bot
     public List<string> UtteranceList { get; } = new List<string>();

     // The number of conversational turns that have occurred        
     public int TurnNumber { get; set; } = 0;

     // Create concurrency control where this is used.
     public string ETag { get; set; } = "*";
}

// Every Conversation turn for our Bot calls this method.
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{     
   if (turnContext.Activity.Type == ActivityTypes.Message)
   {
      // Replace the two lines of code from original MyBot code with the following:
      
      // preserve user input.
      var utterance = turnContext.Activity.Text;  
      // make empty local logitems list.
      UtteranceLog logItems = null;
          
      // see if there are previous messages saved in storage.
      try
      {
         string[] utteranceList = { "UtteranceLog" };
         logItems = _myStorage.ReadAsync<UtteranceLog>(utteranceList).Result?.FirstOrDefault().Value;
      }
      catch
      {
         // Inform the user an error occured.
         await turnContext.SendActivityAsync("Sorry, something went wrong reading your stored messages!");
      }
         
      // If no stored messages were found, create and store a new entry.
      if (logItems is null)
      {
            // add the current utterance to a new object.
            logItems = new UtteranceLog();
            logItems.UtteranceList.Add(utterance);
            // set initial turn counter to 1.
            logItems.TurnNumber++;

            // Show user new user message.
            await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");

            // Create Dictionary object to hold received user messages.
            var changes = new Dictionary<string, object>();
            {
               changes.Add("UtteranceLog", logItems);
            }
         try
         {
            // Save the user message to your Storage.
            await _myStorage.WriteAsync(changes, cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
      // Else, our Storage already contained saved user messages, add new one to the list.
      else
      {
         // add new message to list of messages to display.
         logItems.UtteranceList.Add(utterance);
         // increment turn counter.
         logItems.TurnNumber++;
         
         // show user new list of saved messages.
         await turnContext.SendActivityAsync($"{logItems.TurnNumber}: The list is now: {string.Join(", ", logItems.UtteranceList)}");
         
         // Create Dictionary object to hold new list of messages.
         var changes = new Dictionary<string, object>();
         {
            changes.Add("UtteranceLog", logItems);
         };
         
         try
         {
            // Save new list to your Storage.
            await _myStorage.WriteAsync(changes,cancellationToken);
         }
         catch
         {
            // Inform the user an error occured.
            await turnContext.SendActivityAsync("Sorry, something went wrong storing your message!");
         }
      }
   }
}

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**
```javascript
const { BotFrameworkAdapter, ConversationState, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Add memory storage.
var storage = new MemoryStorage();

// const conversationState = new ConversationState(storage);
// adapter.use(conversationState);

// Listen for incoming requests - adds storage for messages.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {

        if (context.activity.type === 'message') {
            // Route to main dialog.
            await myBot.onTurn(context);
            // Save updated utterance inputs.
            await logMessageText(storage, context);
        }
        else {
            // Just route to main dialog.
            await myBot.onTurn(context);
        } 
    });
});

// This function stores new user messages. Creates new utterance log if none exists.
async function logMessageText(storage, context) {
    let utterance = context.activity.text;
    try {
        // Read from the storage.
        let storeItems = await storage.read(["UtteranceLogJS"])
        // Check the result.
        var UtteranceLogJS = storeItems["UtteranceLogJS"];

        if (typeof (UtteranceLogJS) != 'undefined') {
            // The log exists so we can write to it.
            storeItems["UtteranceLogJS"].turnNumber++;
            storeItems["UtteranceLogJS"].UtteranceList.push(utterance);
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                context.sendActivity(`${numStored}: You stored: ${storedString}`);
            } catch (err) {
                context.sendActivity(`Write failed of UtteranceLogJS: ${err}`);
            }

         } else {
            context.sendActivity(`Creating and saving new utterance log`);
            var turnNumber = 1;
            storeItems["UtteranceLogJS"] = { UtteranceList: [`${utterance}`], "eTag": "*", turnNumber }
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                context.sendActivity(`${numStored}: You stored: ${storedString}`);
            } catch (err) {
                context.sendActivity(`Write failed: ${err}`);
            }
        }
    } catch (err) {
        context.sendActivity(`Read rejected. ${err}`);
    };
}

```

---

### <a name="start-your-bot"></a>Запуск бота
Запустите бот на локальном компьютере.

### <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота
После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Open Bot** (Открыть бот) на вкладке приветствия в эмуляторе. 
2. Выберите файл с расширением .bot, расположенный в каталоге созданного проекта.

### <a name="interact-with-your-bot"></a>Взаимодействие с ботом
Отправьте сообщение боту, и он отобразит список полученных сообщений.
![Работающий эмулятор](../media/emulator-v4/emulator-running.png)

 
## <a name="using-cosmos-db"></a>Использование Cosmos DB
Начав использовать хранилище в памяти, мы изменим код, чтобы начать работу с Azure Cosmos DB. Cosmos DB — это глобально распределенная многомодельная база данных Майкрософт. Azure Cosmos DB позволяет гибко и независимо масштабировать пропускную способность и ресурсы хранилища в любом количестве регионов Azure. Она гарантирует пропускную способность, задержку, доступность и согласованность в соответствии с комплексными Соглашениями об уровне обслуживания (SLA). 

### <a name="set-up"></a>Настройка
Для использования Cosmos DB в боте необходимо выполнить некоторое настройки, прежде чем приступать к написанию кода.

#### <a name="create-your-database-account"></a>Создание учетной записи базы данных
1. В новом окне браузера войдите на [портал Azure](http://portal.azure.com).
2. Выберите **Создать ресурс > Базы данных > Azure Cosmos DB**.
3. На странице **Новая учетная запись** укажите уникальное имя в поле **Идентификатор**. В поле **API** выберите **SQL** и укажите нужные сведения в полях **Подписка**, **Расположение** и **Группа ресурсов**.
4. Затем щелкните **Создать**.

Создание учетной записи займет несколько минут. Подождите, пока на портале не откроется страница с сообщением "Поздравляем! Ваша учетная запись Azure Cosmos DB создана".

##### <a name="add-a-collection"></a>Добавление коллекции
1. Выберите **Параметры > Создать коллекцию**. Справа отобразится область **Добавление коллекции** (вам может потребоваться прокрутить вправо, чтобы увидеть ее). Из-за последних обновлений Cosmos DB теперь обязательным является ключ секции: _/id_. Этот ключ позволит избежать ошибок запросов между секциями.

![Добавление коллекции Cosmos DB](./media/add_database_collection.png)

2. Имя новой базы данных — bot-cosmos-sql-db, а идентификатор коллекции — bot-storage. Эти значения будут использованы в примерах кода ниже.
 -
![База данных Cosmos](./media/cosmos-db-sql-database.png)

3. Универсальный код ресурса (URI) и ключ конечной точки доступны на вкладке **Ключи** параметров базы данных. Эти значения понадобятся для настройки кода далее. 

![Ключи Cosmos DB](./media/comos-db-keys.png)

#### <a name="add-configuration-information"></a>Добавление сведений о конфигурации
Данные конфигурации, добавляемые в хранилище Cosmos DB, короткие и простые. По мере усложнения бота вы можете добавлять дополнительные параметры конфигурации, используя те же методы. В этом примере используются имена базы данных и коллекции Cosmos DB из примера выше.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**MyBot.cs**
```csharp
private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
private const string CosmosDBKey = "<your-cosmos-db-account-key>";
private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
private const string CosmosDBCollectionName = "bot-storage";
```


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте следующие сведения в файл `.env`.

Файл с расширением **.env**
```javascript
ACTUAL_SERVICE_ENDPOINT=<your database URI>
ACTUAL_AUTH_KEY=<your database key>
DATABASE=bot-cosmos-sql-db
COLLECTION=bot-storage
```
---

#### <a name="installing-packages"></a>Установка пакетов
Убедитесь, что вы установили пакеты, необходимые для работы с Cosmos DB.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Вы можете добавить ссылки на пакет botbuilder-azure в свой проект с помощью npm: -->


```powershell
npm install --save botbuilder-azure 
```

Чтобы использовать файл конфигурации с расширением .env, нужно установить дополнительный пакет. Сначала установите пакет dotnet с помощью npm:

```powershell
npm install --save dotenv
```

---

### <a name="implementation"></a>Реализация 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Следующий пример кода использует тот же код бота, что и в примере с [хранилищем в памяти](#memory-storage) выше.
Пример кода ниже демонстрирует реализацию хранилища Cosmos DB для свойства _myStorage_ вместо локального хранилища в памяти.

**MyBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;

// Create local Memory Storage - commented out.
// private static readonly MemoryStorage _myStorage = new MemoryStorage();

// Replaces Memory Storage with reference to Cosmos DB.
private static readonly CosmosDbStorage _myStorage = new CosmosDbStorage(new CosmosDbStorageOptions
{
   AuthKey = CosmosDBKey,
   CollectionId = CosmosDBCollectionName,
   CosmosDBEndpoint = new Uri(CosmosServiceEndpoint),
   DatabaseId = CosmosDBDatabaseName,
});
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Следующий пример кода соответствует примеру с [хранилищем в памяти](#memory-storage), но имеет некоторые отличия.

Импортируйте `CosmosDbStorage` из botbuilder-azure и настройте dotenv для чтения файла `.env`.

**index.js**
```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
```
Закомментируйте хранилище в памяти, заменив ссылкой на Cosmos DB.

**index.js**
```javascript
// Create local Memory Storage - commented out.
// var storage = new MemoryStorage();

// Create access to Cosmos DB storage.
//Add CosmosDB 
const storage = new CosmosDbStorage({
    serviceEndpoint: process.env.ACTUAL_SERVICE_ENDPOINT, 
    authKey: process.env.ACTUAL_AUTH_KEY, 
    databaseId: process.env.DATABASE,
     collectionId: process.env.COLLECTION
})

```

---

## <a name="start-your-bot"></a>Запуск бота
Запустите бот на локальном компьютере.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота
После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Open Bot** (Открыть бот) на вкладке приветствия в эмуляторе. 
2. Выберите файл с расширением .bot, расположенный в каталоге созданного проекта.

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом
Отправьте сообщение боту, и он отобразит список полученных сообщений.
![Работающий эмулятор](../media/emulator-v4/emulator-running.png)


### <a name="view-your-data"></a>Просмотр данных
Если вы запустили бот и сохранили информацию, ее можно просмотреть на портале Azure на вкладке **Обозреватель данных**. 

![Пример Data Explorer](./media/data_explorer.PNG)

### <a name="manage-concurrency-using-etags"></a>Управление параллелизмом с помощью тегов eTag
В нашем примере кода бота мы установили свойство `eTag` каждого экземпляра `IStoreItem` равным `*`. Член `eTag` (тег сущности) объекта хранилища используется в Cosmos DB для управления параллелизмом. `eTag` указывает базе данных, что делать, если другой экземпляр бота изменил объект в том же хранилище, в которое записывает данные ваш бот. 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>Приоритет последней записи — разрешить перезапись
Если указать звездочку (`*`) в качестве значения свойства `eTag`, это означает, то приоритет принадлежит последнему боту, который выполняет запись. При создании хранилища данных можно указать значение `*` для свойства `eTag`. Это означает, что вы не сохранили записываемые данные, или что вы хотите, чтобы последний бот перезаписал все ранее сохраненные свойства. Если параллелизм не является проблемой для вашего бота, укажите для свойства `eTag` значение `*`. Это разрешает перезапись.

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>Поддержание параллелизма и предотвращение перезаписи
Если нужно предотвратить параллельный доступ к свойству и избежать перезаписи данных другим экземпляром бота, при сохранении данных в Cosmos DB используйте другое значение свойства `eTag`, отличное от `*`. Если бот попытается сохранить данные о состоянии, а параметр `eTag` не соответствует параметру `eTag` в хранилище, бот получит сообщение об ошибке `etag conflict key=`. <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

По умолчанию хранилище Cosmos DB проверяет равенство свойства `eTag` в объекте хранилища каждый раз, когда бот записывает данные в этот элемент, а затем включает в это свойство новое уникальное значение после каждой операции записи. Если значение свойства `eTag` во время записи не соответствует значению свойства `eTag` в хранилище, это означает, что другой бот или поток изменили данные. 

Например, предположим, что ваш бот должен изменять сохраненные заметки, но вы не хотите, чтобы он перезаписывал изменения, выполненные другим экземпляром бота. Если другой экземпляр бота внес изменения, пользователь должен изменять последнюю версию со всеми внесенными изменениями.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Во-первых, создайте класс, который реализует `IStoreItem`.

```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

Затем создайте начальную заметку, создав объект хранилища и добавив объект в это хранилище.

```csharp
// create a note for the first time, with a non-null, non-* ETag.
var note = new Note { Name = "Shopping List", Contents = "eggs", ETag = "x" };

var changes = Dictionary<string, object>();
{
    changes.Add("Note", note);
};
await NoteStore.WriteAsync(changes, cancellationToken);
```

Впоследствии обновите заметку, сохранив значение `eTag`, прочитанное из хранилища.

```csharp
var note = NoteStore.ReadAsync<Note>("Note").Result?.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new Dictionary<string, object>();
    {
         changes.Add("Note1", note);
    };
    await NoteStore.WriteAsync(changes, cancellationToken);
}
```

Если заметка в хранилище была изменена перед записью ваших изменений, то при вызове `Write` возникнет исключение.


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте в конец кода бота вспомогательную функцию, которая будет записывать пример заметки в хранилище данных.
Сначала создайте объект `myNoteData`.

```javascript
// Helper function for writing a sample note to a data store
async function createSampleNote(storage, context) {
    var myNoteData = {
        name: "Shopping List",
        contents: "eggs",
        // If any Note file is already stored, the eTag field
        // must be set to "*" in order to allow writing without first reading the stored eTag
        // otherwise you'll likely get an exception indicating an eTag conflict. 
        eTag: "*"
    }
}
```

Во вспомогательной функции `createSampleNote` инициализируйте объект `changes`, добавьте в него *заметки* и запишите объект в хранилище.

```javascript
// Write the note data to the "Note" key
var changes = {};
changes["Note"] = myNoteData;
// Creates a file named Note, if it doesn't already exist.
// specifying eTag= "*" will overwrite any existing contents.
// The act of writing to the file automatically updates the eTag property
// The first time you write to Note, the eTag is changed from *, and file contents will become:
//    {"name":"Shopping List","contents":"eggs","eTag":"1"}
try {
    await storage.write(changes);
    await context.sendActivity('Successful created a note.');
} catch (err) {
    await context.sendActivity(`Could not create note: ${err}`);
}
```

Позже для доступа к заметке и ее обновления мы создадим другую вспомогательную функцию. Эта функция вызывается, когда пользователь вводит команду update note для обновления заметки.

```javascript
async function updateSampleNote(storage, context) {
    try {
        // Read in a note
        var note = await storage.read(["Note"]);
        console.log(`note.eTag=${note["Note"].eTag}\n note=${JSON.stringify(note)}`);
        // update the note that we just read
        note["Note"].contents += ", bread";
        console.log(`Updated note=${JSON.stringify(note)}`);

        try {
            await storage.write(note); // Write the changes back to storage
            await context.sendActivity('Successfully updated to note.');
        } catch (err) {            console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

Если заметка в хранилище была изменена перед записью ваших изменений, то при вызове `write` возникнет исключение.

---

Для поддержания параллелизма всегда считывайте значение свойства из хранилища, затем изменяйте прочитанное свойство, чтобы сохранить `eTag`. При считывании сведений о пользователях из хранилища ответ будет содержать свойство eTag. Если вы изменили данные и записываете обновленные данные в хранилище, ваш запрос должен включать свойство eTag, содержащее то же значение, которое было прочитано ранее. Однако при записи объекта со свойством `eTag`, имеющим значение `*`, будет разрешена перезапись любых других изменений.

## <a name="using-blob-storage"></a>Использование хранилища BLOB-объектов 
Хранилище BLOB-объектов Azure — это решение корпорации Майкрософт для хранения объектов в облаке. Хранилище BLOB-объектов оптимизировано для хранения больших объемов неструктурированных данных, например текстовых или двоичных данных.

### <a name="create-your-blob-storage-account"></a>Создание учетной записи хранилища BLOB-объектов
Для использования хранилища BLOB-объектов в боте необходимо выполнить некоторые настройки, прежде чем приступать к написанию кода.
1. В новом окне браузера войдите на [портал Azure](http://portal.azure.com).
2. Нажмите **Создать ресурс > Хранилище > Учетная запись хранения — BLOB-объект, файл, таблица, очередь**.
3. На странице **Создание учетной записи** введите **Имя учетной записи хранения**, выберите **Хранилище BLOB-объектов** в поле **Тип учетной записи**, а также укажите нужные сведения в полях **Расположение**, **Группа ресурсов** и **Подписка**.  
4. Затем щелкните **Создать**.

![Создание хранилища BLOB-объектов](./media/create-blob-storage.png)

#### <a name="add-configuration-information"></a>Добавление сведений о конфигурации

Найдите ключи к хранилищу BLOB-объектов, которые необходимы для настройки хранилища в вашем боте, как показано выше:
1. На портале Azure откройте свою учетную запись хранилища BLOB-объектов и выберите **Настройки > Ключи доступа**.

![Нахождение ключей доступа к хранилищу BLOB-объектов](./media/find-blob-storage-keys.png)

Теперь с помощью этих двух ключей мы предоставим доступ из нашего кода к учетной записи хранилища BLOB-объектов.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder.Azure;
```
Измените строку кода со свойством _myStorage_ и присвойте этому свойству значение существующей учетной записи хранилища BLOB-объектов.

```csharp


private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
```javascript
const mystorage = new BlobStorage({
   <youy_containerName>,
   <your_storageAccountOrConnectionString>,
   <your_storageAccessKey>
})
```
---

Когда свойству _myStorage_ будет присвоено значение учетной записи хранилища BLOB-объектов, бот сможет сохранять данные в это хранилище и извлекать их оттуда.

## <a name="start-your-bot"></a>Запуск бота
Запустите бот на локальном компьютере.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота
После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Open Bot** (Открыть бот) на вкладке приветствия в эмуляторе. 
2. Выберите файл с расширением .bot, расположенный в каталоге созданного проекта.

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом
Отправьте сообщение боту, и он отобразит список полученных сообщений.
![Работающий эмулятор](../media/emulator-v4/emulator-running.png)

### <a name="view-your-data"></a>Просмотр данных
Если вы запустили бот и сохранили информацию, ее можно просмотреть на портале Azure на вкладке **Обозреватель службы хранилища**.

## <a name="blob-transcript-storage"></a>Хранилище расшифровок разговоров в BLOB-объектах
Хранилище расшифровок в BLOB-объектах Azure — это специализированное хранилище, которое позволяет легко сохранять и получать разговоры пользователей в виде записей расшифровок. Использовать хранилище расшифровок разговоров в BLOB-объектах Azure особенно удобно для автоматической записи данных, вводимых пользователями и дальнейшего их анализа при отладке работы бота.

### <a name="set-up"></a>Настройка
Хранилище расшифровок разговоров в BLOB-объектах Azure использует ту же учетную запись хранилища BLOB-объектов, которая описана в разделах _Создание учетной записи хранилища BLOB-объектов_ и _Добавление сведений о конфигурации_ выше. Для этой статьи мы добавили новый контейнер BLOB-объектов _mybottranscripts_. 

### <a name="implementation"></a>Реализация 
В коде ниже указатель на хранилище расшифровок _transcriptStore_ подключается к новой учетной записи хранилища расшифровок в BLOB-объектах. Показанный здесь исходный код решения для хранения бесед пользователя основан на примере [журнала бесед](https://aka.ms/bot-history-sample-code). 

```csharp
// In Startup.cs
using Microsoft.Bot.Builder.Azure;

// Enable the conversation transcript middleware.
blobStore = new AzureBlobTranscriptStore(blobStorageConfig.ConnectionString, storageContainer);
var transcriptMiddleware = new TranscriptLoggerMiddleware(blobStore);
options.Middleware.Add(transcriptMiddleware);

// In ConversationHistoryBot.cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;

private readonly AzureBlobTranscriptStore _transcriptStore;

/// <param name="transcriptStore">Injected via ASP.NET dependency injection.</param>
public ConversationHistoryBot(AzureBlobTranscriptStore transcriptStore)
{
    _transcriptStore = transcriptStore ?? throw new ArgumentNullException(nameof(transcriptStore));
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>Хранение расшифровок разговоров пользователей в виде BLOB-объектов Azure
После добавления TranscriptLoggerMiddleware хранилище расшифровок будет запущено автоматически для сохранения бесед пользователя с ботом. Эти разговоры можно позже использовать для отладки, чтобы проанализировать взаимодействие пользователей с ботом. Указанный ниже код извлекает расшифровку, а затем отправляет ее в текущую беседу, если свойство activity.text получает входящее сообщение _!history_. Примечание. Метод SendConversationHistoryAsync поддерживается в каналах Direct Line, веб-чата и эмулятора.


```csharp
/// <summary>
/// Every Conversation turn for our EchoBot will call this method. 
/// </summary>
/// <param name="turnContext">A <see cref="ITurnContext"/> containing all the data needed
/// for processing this conversation turn. </param>        
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    var activity = turnContext.Activity;
    if (activity.Type == ActivityTypes.Message)
    {
        if (activity.Text == "!history")
        {
           // Download the activities from the Transcript (blob store) when a request to upload history arrives.
           var connectorClient = turnContext.TurnState.Get<ConnectorClient>(typeof(IConnectorClient).FullName);
           // Get all the message type activities from the Transcript.
           string continuationToken = null;
           var count = 0;
           do
           {
               var pagedTranscript = await _transcriptStore.GetTranscriptActivitiesAsync(activity.ChannelId, activity.Conversation.Id, continuationToken);
               var activities = pagedTranscript.Items
                  .Where(a => a.Type == ActivityTypes.Message)
                  .Select(ia => (Activity)ia)
                  .ToList();
               
               var transcript = new Transcript(activities);

               await connectorClient.Conversations.SendConversationHistoryAsync(activity.Conversation.Id, transcript, cancellationToken: cancellationToken);

               continuationToken = pagedTranscript.ContinuationToken;
           }
           while (continuationToken != null);
```

### <a name="find-all-stored-transcripts-for-your-channel"></a>Поиск всех сохраненных расшифровок разговоров для вашего канала
Для просмотра сохраненных данных можно использовать следующий код, который позволяет найти все объекты _ConversationID_ для всех сохраненных расшифровок разговоров программным способом. Если используется эмулятор бота для тестирования кода, при выборе функции _Начать снова_ начинается новая расшифровка с новым идентификатором _ConversationID_.

```csharp
List<string> storedTranscripts = new List<string>();
PagedResult<Transcript> pagedResult = null;
var pageSize = 0;
do
{
    pagedResult = await _transcriptStore.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
    
    // transcript item contains ChannelId, Created, Id.
    // save the converasationIds (Id) found by "ListTranscriptsAsync" to a local list.
    foreach (var item in pagedResult.Items)
    {
         // Make sure we store an unescaped conversationId string.
         var strConversationId = item.Id;
         storedTranscripts.Add(Uri.UnescapeDataString(strConversationId));
    }
} while (pagedResult.ContinuationToken != null);
```

### <a name="retrieve-user-conversations-from-azure-blob-transcript-storage"></a>Извлечение разговоров пользователей из хранилища расшифровок в виде BLOB-объектов Azure
Сохранив расшифровки разговоров с ботом в хранилище BLOB-объектов Azure, можно извлечь их программным способом для тестирования или отладки, используя метод _GetTranscriptActivities_ экземпляра AzureBlobTranscriptStorage. Следующий пример кода извлекает все расшифровки разговоров с пользователями, каждая из которых содержит данные, собранные за последние 24 часа.


```csharp
var numTranscripts = storedTranscripts.Count();
for (int i = 0; i < numTranscripts; i++)
{
    PagedResult<IActivity> pagedActivities = null;
    do
    {
        string thisConversationId = storedTranscripts[i];
        // Find all inputs in the last 24 hours.
        DateTime yesterday = DateTime.Now.AddDays(-1);
        // Retrieve iActivities for this transcript.
        pagedActivities = await _myTranscripts.GetTranscriptActivitiesAsync("emulator", thisConversationId, pagedActivities?.ContinuationToken, yesterday);
        foreach (var item in pagedActivities.Items)
        {
            // View as message and find value for key "text" :
            var thisMessage = item.AsMessageActivity();
            var userInput = thisMessage.Text;
         }
    } while (pagedActivities.ContinuationToken != null);
}
```

### <a name="remove-stored-transcripts-from-azure-blob-transcript-storage"></a>Удаление сохраненных расшифровок из хранилища BLOB-объектов Azure
Завершив тестирование или отладку с использованием записанных разговоров с пользователями, сохраненные расшифровки можно удалить из хранилища BLOB-объектов Azure программным способом. Следующий пример кода удаляет все сохраненные расшифровки разговоров с ботом из хранилища.


```csharp
for (int i = 0; i < numTranscripts; i++)
{
   // Remove all stored transcripts except the last one found.
   if (i > 0)
   {
       string thisConversationId = storedTranscripts[i];    
       await _transcriptStore.DeleteTranscriptAsync("emulator", thisConversationId);
    }
}
```


По ссылке ниже можно найти дополнительные сведения о [хранилище расшифровок в виде BLOB-объектов Azure](https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore).  

## <a name="next-steps"></a>Дополнительная информация
Теперь, когда вы знаете, как напрямую считывать данные из хранилища и записывать данные в хранилища, посмотрим, как сделать это с помощью диспетчера состояний.

> [!div class="nextstepaction"]
> [Сохранение состояния с помощью свойств conversation и user](bot-builder-howto-v4-state.md)


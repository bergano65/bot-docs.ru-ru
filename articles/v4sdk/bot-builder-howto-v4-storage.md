---
title: Запись данных напрямую в хранилище | Документация Майкрософт
description: Сведения о том, как выполнять операции чтения и записи непосредственно в хранилище с помощью пакета SDK Bot Framework для .NET.
keywords: хранилище, чтение и запись, хранилище в памяти, eTag
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1213e7d0c03fd6faa3768fac85a59bf4c7b5b28a
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/03/2019
ms.locfileid: "68756984"
---
# <a name="write-directly-to-storage"></a>Запись данных напрямую в хранилище

[!INCLUDE[applies-to](../includes/applies-to.md)]

Вы можете выполнять операции записи и чтения непосредственно в объекте хранилища, не используя ПО промежуточного слоя или объект контекста. Это может требоваться для данных, которые бот использует для сохранения беседы, или данных, которые поступают из источника за пределами потока общения бота. В рамках этой модели хранилища данные считываются непосредственно из хранилища без использования диспетчера состояний. Примеры кода в этой статье демонстрируют, как выполнять операции записи и чтения в хранилище с использованием **хранилища в памяти**, **Cosmos DB**, **хранилища BLOB-объектов** и **хранилища расшифровок в BLOB-объектах Azure**. 

## <a name="prerequisites"></a>Предварительные требования
- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.
- Ознакомьтесь с руководством по локальному созданию бота для [dotnet](https://aka.ms/bot-framework-www-c-sharp-quickstart) или [NodeJs](https://aka.ms/bot-framework-www-node-js-quickstart).
- Шаблон для пакета SDK Bot Framework версии 4 для [C#](https://aka.ms/bot-vsix) или [NodeJS](https://nodejs.org) и [Yeoman](http://yeoman.io).

## <a name="about-this-sample"></a>Об этом примере
Пример кода в этой статье определяет структуру базового эхо-бота. Добавив дополнительный код (см. ниже), вы сможете расширить функции этого бота. Этот расширенный код создает список для хранения вводимых пользователем данных по мере получения. На каждом шаге полный список этих данных возвращается пользователю. Структура данных, содержащая этот список, записывается в хранилище в конце соответствующего шага. Мы рассмотрим разные типы хранилищ по мере добавления расширенных функций в пример кода.

## <a name="memory-storage"></a>Хранилище в памяти

Пакет SDK Bot Framework позволяет хранить вводимые пользователем данные с помощью размещенного в памяти хранилища. Хранилище в памяти используется только для тестирования и не предназначено для рабочей среды. Постоянные хранилища, такие как хранилище базы данных, лучше всего подходят для ботов в рабочей среде. Прежде чем публиковать бот, настройте использование хранилища в Cosmos DB или хранилища BLOB-объектов.

#### <a name="build-a-basic-bot"></a>Создание базового бота

Остальная часть этой статьи описывает использование эхо-бота. Пример кода эхо-бота можно создать локально для [C#](https://aka.ms/bot-framework-www-c-sharp-quickstart) или [JS](https://aka.ms/bot-framework-www-node-js-quickstart).

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
using System;
using System.Threading.Tasks;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Collections.Generic;
using System.Linq;
using System.Threading;

// Represents a bot saves and echoes back user input.
public class EchoBot : ActivityHandler
{
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
     
   // Echo back user input.
   protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
   {
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
      ...  // OnMessageActivityAsync( )
   }
}

```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы использовать файл конфигурации .env, нужно установить дополнительный пакет. Если он не установлен, получите пакет dotnet с помощью npm:

```powershell
npm install --save dotenv
```

**bot.js**
```javascript
const { ActivityHandler, MemoryStorage } = require('botbuilder');
const restify = require('restify');

// Add memory storage.
var storage = new MemoryStorage();

// Process incoming requests - adds storage for messages.
class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async turnContext => { console.log('this gets called (message)'); 
        await turnContext.sendActivity(`You said '${ turnContext.activity.text }'`); 
        // Save updated utterance inputs.
        await logMessageText(storage, turnContext);
    });
        this.onConversationUpdate(async turnContext => { console.log('this gets called (conversation update)'); 
        await turnContext.sendActivity('[conversationUpdate event detected]'); });
    }
}

// This function stores new user messages. Creates new utterance log if none exists.
async function logMessageText(storage, turnContext) {
    let utterance = turnContext.activity.text;
    // debugger;
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
                turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                turnContext.sendActivity(`Write failed of UtteranceLogJS: ${err}`);
            }
        }
        else{
            turnContext.sendActivity(`Creating and saving new utterance log`);
            var turnNumber = 1;
            storeItems["UtteranceLogJS"] = { UtteranceList: [`${utterance}`], "eTag": "*", turnNumber }
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                turnContext.sendActivity(`Write failed: ${err}`);
            }
        }
    }
    catch (err){
        turnContext.sendActivity(`Read rejected. ${err}`);
    }
}

module.exports.MyBot = MyBot;

```
---

### <a name="start-your-bot"></a>Запуск бота
Запустите бот на локальном компьютере.

### <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота
- Установите [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme). После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Create a new bot configuration** (Создать конфигурацию бота) на вкладке Welcome (Приветствие) эмулятора. 
2. Заполните поля для подключения к боту, используя сведения на веб-странице, отображаемой при запуске бота.

### <a name="interact-with-your-bot"></a>Взаимодействие с ботом
Отправьте сообщение боту. Он отобразит список полученных сообщений.

![Окно Bot Framework Emulator](./media/emulator-direct-storage-test.png)

## <a name="using-cosmos-db"></a>Использование Cosmos DB
Начав использовать хранилище в памяти, мы изменим код, чтобы начать работу с Azure Cosmos DB. Cosmos DB — это глобально распределенная многомодельная база данных Майкрософт. Azure Cosmos DB позволяет гибко и независимо масштабировать пропускную способность и ресурсы хранилища в любом количестве регионов Azure. Она гарантирует пропускную способность, задержку, доступность и согласованность в соответствии с комплексными Соглашениями об уровне обслуживания (SLA). 

### <a name="set-up"></a>Настройка
Для использования Cosmos DB в боте необходимо выполнить некоторое настройки, прежде чем приступать к написанию кода. См. подробнее о создании приложения и базы данных Cosmos DB для [dotnet](https://aka.ms/Bot-framework-create-dotnet-cosmosdb) и [Node.js](https://aka.ms/Bot-framework-create-nodejs-cosmosdb).

### <a name="create-your-database-account"></a>Создание учетной записи базы данных

1. В новом окне браузера войдите на [портал Azure](http://portal.azure.com).

![Создание базы данных Cosmos DB](./media/create-cosmosdb-database.png)

2. Выберите **Создать ресурс > Базы данных > Azure Cosmos DB**.

![Страница создания учетной записи Cosmos DB](./media/cosmosdb-new-account-page.png)

3. На **странице создания учетной записи** укажите сведения о **подписке** и **группе ресурсов**. Укажите уникальное имя в поле **Имя учетной записи**. Это имя будет включено в URL-адрес для доступа к данным. В поле **API**выберите **Core(SQL)** и укажите ближайшее к вам **расположение**, чтобы ускорить доступ к данным.
4. Щелкните **Просмотреть и создать**.
5. Проверив сведения, щелкните **Создать**.

Создание учетной записи займет несколько минут. Подождите, пока на портале не откроется страница с сообщением "Поздравляем! Ваша учетная запись Azure Cosmos DB создана".

### <a name="add-a-collection"></a>Добавление коллекции

![Добавление коллекции Cosmos DB](./media/add_database_collection.png)

1. Выберите **Параметры > Создать коллекцию**. Справа отобразится область **Добавление коллекции** (вам может потребоваться прокрутить вправо, чтобы увидеть ее). Из-за последних обновлений Cosmos DB теперь обязательным является ключ секции: _/id_. Этот ключ позволит избежать ошибок запросов между секциями.

![Cosmos DB](./media/cosmos-db-sql-database.png)

2. Имя новой базы данных — bot-cosmos-sql-db, а идентификатор коллекции — bot-storage. Эти значения будут использованы в примерах кода ниже.

![Ключи Cosmos DB](./media/comos-db-keys.png)

3. Универсальный код ресурса (URI) и ключ конечной точки доступны на вкладке **Ключи** параметров базы данных. Эти значения понадобятся для настройки кода далее. 

### <a name="add-configuration-information"></a>Добавление сведений о конфигурации
Данные конфигурации, добавляемые в хранилище Cosmos DB, короткие и простые. По мере усложнения бота вы можете добавлять дополнительные параметры конфигурации, используя те же методы. В этом примере используются имена базы данных и коллекции Cosmos DB из примера выше.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
public class EchoBot : ActivityHandler
{
   private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
   private const string CosmosDBKey = "<your-cosmos-db-account-key>";
   private const string CosmosDBDatabaseName = "bot-cosmos-sql-db";
   private const string CosmosDBCollectionName = "bot-storage";
   ...
   
}
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте следующие сведения в файл `.env`.

Файл с расширением **.env**
```javascript
DB_SERVICE_ENDPOINT="<your-Cosmos-db-URI>"
AUTH_KEY="<your-cosmos-db-account-key>"
DATABASE="<bot-cosmos-sql-db>"
COLLECTION="<bot-storage>"
```
---

#### <a name="installing-packages"></a>Установка пакетов
Убедитесь, что вы установили пакеты, необходимые для работы с Cosmos DB.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Вы можете добавить ссылки на пакет botbuilder-azure в свой проект с помощью npm.
>**Примечание.** Этот пакет npm зависит от версии Python, установленной на компьютере разработки. Если вы еще не установили Python, перейдите по следующей ссылке: [Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure 
```

При необходимости вы можете получить пакет dotnet с помощью npm, чтобы получить доступ к параметрам файла `.env`.

```powershell
npm install --save dotenv
```
---

### <a name="implementation"></a>Реализация 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Следующий пример кода использует тот же код бота, что и в примере с [хранилищем в памяти](#memory-storage) выше.
Пример кода ниже демонстрирует реализацию хранилища Cosmos DB для свойства _myStorage_ вместо локального хранилища в памяти. Хранилище в памяти закомментировано и заменено ссылкой на Cosmos DB.

**EchoBot.cs**
```csharp

using System;
...
using Microsoft.Bot.Builder.Azure;
...
public class EchoBot : ActivityHandler
{
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
   
   ...
}

```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Следующий пример кода соответствует примеру с [хранилищем в памяти](#memory-storage), но имеет некоторые отличия.

Импортируйте `CosmosDbStorage` из `botbuilder-azure` и настройте dotenv для чтения файла `.env`.

**bot.js**

```javascript
const { CosmosDbStorage } = require("botbuilder-azure");
```
Закомментируйте хранилище в памяти, заменив ссылкой на Cosmos DB.

**bot.js**
```javascript
// initialized to access values in .env file.
var dotenv = require('dotenv');
dotenv.load();

// Create local Memory Storage - commented out.
// var storage = new MemoryStorage();

// Create access to CosmosDb Storage - this replaces local Memory Storage.
var storage = new CosmosDbStorage({
    serviceEndpoint: process.env.DB_SERVICE_ENDPOINT, 
    authKey: process.env.AUTH_KEY, 
    databaseId: process.env.DATABASE,
    collectionId: process.env.COLLECTION
})

```
---

## <a name="start-your-bot"></a>Запуск бота
Запустите бот на локальном компьютере.

## <a name="test-your-bot-with-bot-framework-emulator"></a>Тестирование бота с помощью Bot Framework Emulator
Подключитесь к боту через Bot Framework Emulator.

1. Щелкните ссылку **Create a new bot configuration** (Создать конфигурацию бота) на вкладке Welcome (Приветствие) эмулятора. 
2. Заполните поля для подключения к боту, используя сведения на веб-странице, отображаемой при запуске бота.

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом
Отправьте сообщение боту, и он отобразит список полученных сообщений.
![Работающий эмулятор](./media/emulator-direct-storage-test.png)


### <a name="view-your-data"></a>Просмотр данных
Если вы запустили бот и сохранили информацию, ее можно просмотреть на портале Azure на вкладке **Обозреватель данных**. 

![Пример Data Explorer](./media/data_explorer.PNG)


## <a name="using-blob-storage"></a>Использование хранилища BLOB-объектов 
Хранилище BLOB-объектов Azure — это решение корпорации Майкрософт для хранения объектов в облаке. Хранилище BLOB-объектов оптимизировано для хранения больших объемов неструктурированных данных, например текстовых или двоичных данных.

### <a name="create-your-blob-storage-account"></a>Создание учетной записи хранилища BLOB-объектов
Для использования хранилища BLOB-объектов в боте необходимо выполнить некоторые настройки, прежде чем приступать к написанию кода.
1. В новом окне браузера войдите на [портал Azure](http://portal.azure.com).

![Создание хранилища BLOB-объектов](./media/create-blob-storage.png)

2. Нажмите **Создать ресурс > Хранилище > Учетная запись хранения — BLOB-объект, файл, таблица, очередь**.

![Страница создания учетной записи хранилища BLOB-объектов](./media/blob-storage-new-account.png)

3. На странице **Создание учетной записи** введите **имя учетной записи хранения**, выберите **Хранилище BLOB-объектов** в поле **Тип учетной записи**, а также укажите нужные сведения в полях **Расположение**, **Группа ресурсов** и **Подписка**.  
4. Щелкните **Просмотреть и создать**.
5. Проверив сведения, щелкните **Создать**.

### <a name="create-blob-storage-container"></a>Создание контейнера хранилища BLOB-объектов
Создав учетную запись хранилища BLOB-объектов, откройте ее, сделав следующее: 
1. Выберите ресурс.
2. Откройте учетную запись с помощью Обозревателя службы хранилища (предварительная версия).

![Создание контейнера хранилища BLOB-объектов](./media/create-blob-container.png)

3. Щелкните правой кнопкой "Контейнеры больших двоичных объектов" и выберите _Создать контейнер BLOB-объектов_.
4. Укажите имя. Это имя будет использоваться в качестве значения <your-blob-storage-container-name> при получении доступа к учетной записи хранилища BLOB-объектов.

#### <a name="add-configuration-information"></a>Добавление сведений о конфигурации
Найдите ключи к хранилищу BLOB-объектов, которые необходимы для настройки хранилища в вашем боте, как показано выше:
1. На портале Azure откройте свою учетную запись хранилища BLOB-объектов и выберите **Настройки > Ключи доступа**.

![Нахождение ключей доступа к хранилищу BLOB-объектов](./media/find-blob-storage-keys.png)

_Строка подключения_ key1 будет использоваться в качестве значения <your-blob-storage-account-string>при получении доступа к учетной записи хранилища BLOB-объектов.

#### <a name="installing-packages"></a>Установка пакетов
При необходимости установите следующие пакеты для использования Cosmos DB.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Вы можете добавить ссылки на пакет botbuilder-azure в свой проект с помощью npm.
>**Примечание.** Этот пакет npm зависит от версии Python, установленной на компьютере разработки. Если вы еще не установили Python, перейдите по следующей ссылке: [Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure 
```

При необходимости вы можете получить пакет dotnet с помощью npm, чтобы получить доступ к параметрам файла `.env`.

```powershell
npm install --save dotenv
```
---

### <a name="implementation"></a>Реализация 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**EchoBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;
```
Измените строку кода со свойством _myStorage_ и присвойте этому свойству значение существующей учетной записи хранилища BLOB-объектов.

**EchoBot.cs**
```csharp
private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте следующие сведения в файл `.env`.

Файл с расширением **.env**
```javascript
BLOB_NAME="<your-blob-storage-container-name>"
BLOB_STRING="<your-blob-storage-account-string>"
```

Обновите файл `bot.js` следующим образом. Вам нужно использовать `BlobStorage` из `botbuilder-azure`.

**bot.js**
```javascript
const { BlobStorage } = require("botbuilder-azure");
```

При необходимости добавьте код для загрузки файла `.env` для реализации хранилища Cosmos DB.

```javascript
// initialized to access values in .env file.
var dotenv = require('dotenv');
dotenv.load();
```
Теперь обновите код, чтобы связать _storage_ с существующей учетной записью хранилища BLOB-объектов, закомментировав предыдущие определения хранилища и добавив следующее.

**bot.js**
```javascript
var storage = new BlobStorage({
    containerName: process.env.BLOB_NAME,
    storageAccountOrConnectionString: process.env.BLOB_STRING
});
```
---

Когда вы свяжете storage с учетной записью хранилища BLOB-объектов, бот сможет сохранять данные в это хранилище и извлекать их оттуда.

## <a name="start-your-bot"></a>Запуск бота
Запустите бот на локальном компьютере.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота
После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Create a new bot configuration** (Создать конфигурацию бота) на вкладке Welcome (Приветствие) эмулятора. 
2. Заполните поля для подключения к боту, используя сведения на веб-странице, отображаемой при запуске бота.

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом
Отправьте сообщение боту, и он отобразит список полученных сообщений.

![Окно Bot Framework Emulator](./media/emulator-direct-storage-test.png)

### <a name="view-your-data"></a>Просмотр данных
Если вы запустили бот и сохранили информацию, ее можно просмотреть на портале Azure на вкладке **Обозреватель службы хранилища**.

## <a name="blob-transcript-storage"></a>Хранилище расшифровок разговоров в BLOB-объектах
Хранилище расшифровок в BLOB-объектах Azure — это специализированное хранилище, которое позволяет легко сохранять и получать разговоры пользователей в виде записей расшифровок. Использовать хранилище расшифровок разговоров в больших двоичных объектах Azure особенно удобно для автоматической записи данных, вводимых пользователями, и дальнейшего их анализа при отладке бота.

### <a name="set-up"></a>Настройка
Хранилище расшифровок разговоров в BLOB-объектах Azure использует ту же учетную запись хранилища BLOB-объектов, которая описана в разделах _Создание учетной записи хранилища BLOB-объектов_ и _Добавление сведений о конфигурации_ выше. Теперь добавьте контейнер для хранения расшифровок.

![Создание контейнера для расшифровок](./media/create-blob-transcript-container.png)

1. Откройте учетную запись хранилища BLOB-объектов Azure
1. Откройте _Обозреватель службы хранилища_.
1. Щелкните правой кнопкой мыши _Контейнеры больших двоичных объектов_ и выберите _Создать контейнер BLOB-объектов_.
1. Присвойте имя контейнеру для расшифровок и щелкните _ОК_. (Мы указали mybottranscripts.)

### <a name="implementation"></a>Реализация 
В коде ниже указатель на хранилище расшифровок `_myTranscripts` подключается к новой учетной записи хранилища BLOB-объектов с расшифровками. Чтобы создать эту ссылку с новым именем контейнера <your-blob-transcript-container-name>, в хранилище BLOB-объектов создается новый контейнер для хранения файлов с расшифровками.

**echoBot.cs**
```csharp
using Microsoft.Bot.Builder.Azure;

public class EchoBot : ActivityHandler
{
   ...
   
   private readonly AzureBlobTranscriptStore _myTranscripts = new AzureBlobTranscriptStore("<your-blob-transcript-storage-account-string>", "<your-blob-transcript-container-name>");
   
   ...
}

```

### <a name="store-user-conversations-in-azure-blob-transcripts"></a>Хранение расшифровок разговоров пользователей в виде BLOB-объектов Azure
Создав контейнер BLOB-объектов для хранения расшифровок, можно будет сохранять разговоры пользователей с ботом. Эти диалоги можно позже использовать для отладки, чтобы проанализировать взаимодействие пользователей с ботом. Каждое действие _перезапуска диалога_ инициирует создание нового списка диалогов с расшифровками. Следующий код сохраняет диалог с пользователем в сохраненном файле с расшифровками.
- Текущая расшифровка сохраняется с использованием `LogActivityAsync`.
- Сохраненные расшифровки можно получить с использованием `ListTranscriptsAsync`.
В этом примере кода идентификатор каждой сохраненной расшифровки включается в список storedTranscripts. Позже этот список используется для управления количеством расшифровок, хранимых в больших двоичных объектах.

**echoBot.cs**
```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);
    
    ...
}

```

### <a name="manage-stored-blob-transcripts"></a>Управление расшифровками, хранимыми в больших двоичных объектах
Хотя хранимые расшифровки можно использовать как средство отладки, со временем их количество превышает возможности хранилища. Дополнительный код ниже использует `DeleteTranscriptAsync` для удаления из хранилища BLOB-объектов всех извлеченных элементов расшифровок, кроме последних трех.

**echoBot.cs**
```csharp

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    await _myTranscripts.LogActivityAsync(turnContext.Activity);

    List<string> storedTranscripts = new List<string>();
    PagedResult<Microsoft.Bot.Builder.TranscriptInfo> pagedResult = null;
    var pageSize = 0;
    do
    {
       pagedResult = await _myTranscripts.ListTranscriptsAsync("emulator", pagedResult?.ContinuationToken);
       pageSize = pagedResult.Items.Count();

       // transcript item contains ChannelId, Created, Id.
       // save the channelIds found by "ListTranscriptsAsync" to a local list.
       foreach (var item in pagedResult.Items)
       {
          storedTranscripts.Add(item.Id);
       }
    } while (pagedResult.ContinuationToken != null);
    
    // Manage the size of your transcript storage.
    for (int i = 0; i < pageSize; i++)
    {
       // Remove older stored transcripts, save just the last three.
       if (i < pageSize - 3)
       {
          string thisTranscriptId = storedTranscripts[i];
          try
          {
             await _myTranscripts.DeleteTranscriptAsync("emulator", thisTranscriptId);
           }
           catch (System.Exception ex)
           {
              await turnContext.SendActivityAsync("Debug Out: DeleteTranscriptAsync had a problem!");
              await turnContext.SendActivityAsync("exception: " + ex.Message);
           }
       }
    }
    ...
}

```

По ссылке ниже можно найти дополнительные сведения о [хранилище BLOB-объектов Azure с расшифровками](https://docs.microsoft.com/dotnet/api/microsoft.bot.builder.azure.azureblobtranscriptstore). 

## <a name="additional-information"></a>Дополнительная информация

### <a name="manage-concurrency-using-etags"></a>Управление параллелизмом с помощью тегов eTag
В нашем примере кода бота мы установили свойство `eTag` каждого экземпляра `IStoreItem` равным `*`. Член `eTag` (тег сущности) объекта хранилища используется в Cosmos DB для управления параллелизмом. `eTag` указывает базе данных, что делать, если другой экземпляр бота изменил объект в том же хранилище, в которое записывает данные ваш бот. 

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>Приоритет последней записи — разрешить перезапись
Если указать звездочку (`*`) в качестве значения свойства `eTag`, это означает, то приоритет принадлежит последнему боту, который выполняет запись. При создании хранилища данных можно указать значение `*` для свойства `eTag`. Это означает, что вы не сохранили записываемые данные, или что вы хотите, чтобы последний бот перезаписал все ранее сохраненные свойства. Если параллелизм не является проблемой для вашего бота, укажите для свойства `eTag` значение `*`. Это разрешает перезапись.

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>Поддержание параллелизма и предотвращение перезаписи
Если нужно предотвратить параллельный доступ к свойству и избежать перезаписи данных другим экземпляром бота, при сохранении данных в Cosmos DB используйте другое значение свойства `eTag`, отличное от `*`. Если бот попытается сохранить данные о состоянии, а параметр `eTag` не соответствует параметру `eTag` в хранилище, бот получит сообщение об ошибке `etag conflict key=`. <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

По умолчанию хранилище Cosmos DB проверяет равенство свойства `eTag` в объекте хранилища каждый раз, когда бот записывает данные в этот элемент, а затем включает в это свойство новое уникальное значение после каждой операции записи. Если значение свойства `eTag` во время записи не соответствует значению свойства `eTag` в хранилище, это означает, что другой бот или поток изменили данные. 

Например, предположим, что ваш бот должен изменять сохраненные заметки, но вы не хотите, чтобы он перезаписывал изменения, выполненные другим экземпляром бота. Если другой экземпляр бота внес изменения, пользователь должен изменять последнюю версию со всеми внесенными изменениями.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Во-первых, создайте класс, который реализует `IStoreItem`.

**EchoBot.cs**
```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string ETag { get; set; }
}
```

Затем создайте начальную заметку, создав объект хранилища и добавив объект в это хранилище.

**EchoBot.cs**
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

**EchoBot.cs**
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


### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте в конец кода бота вспомогательную функцию, которая будет записывать пример заметки в хранилище данных.
Сначала создайте объект `myNoteData`.

**bot.js**
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

**bot.js**
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
     var list = changes["Note"].contents;
     await context.sendActivity(`Successful created a note: ${list}`);
} catch (err) {
     await context.sendActivity(`Could not create note: ${err}`);
}
```

Доступ к вспомогательной функции осуществляется с использованием логики бота путем добавления следующего вызова:

**bot.js**
```javascript
// Save a note with etag.
await createSampleNote(storage, turnContext);
```

Для последующего получения и обновления созданной заметки мы создадим другую вспомогательную функцию. Она вызывается, когда пользователь вводит команду update note.

**bot.js**
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
             var list = note["Note"].contents;
             await context.sendActivity(`Successfully updated note: ${list}`);
        } catch (err) {            
             console.log(`Write failed: ${err}`);
        }
    }
    catch (err) {
        await context.sendActivity(`Unable to read the Note: ${err}`);
    }
}
```

Доступ к вспомогательной функции осуществляется с использованием логики бота путем добавления следующего вызова:

**bot.js**
```javascript
// Update a note with etag.
await updateSampleNote(storage, turnContext);
```

Если примечание обновлено в хранилище другим пользователем перед попыткой выполнить обратную запись изменений, значение `eTag` не будет больше соответствовать, и при вызове `write` возникнет исключение.

---

Для поддержания параллелизма всегда считывайте значение свойства из хранилища, затем изменяйте прочитанное свойство, чтобы сохранить `eTag`. При считывании сведений о пользователях из хранилища ответ будет содержать свойство eTag. Если вы изменили данные и записываете обновленные данные в хранилище, ваш запрос должен включать свойство eTag, содержащее то же значение, которое было прочитано ранее. Однако при записи объекта со свойством `eTag`, имеющим значение `*`, будет разрешена перезапись любых других изменений.

## <a name="next-steps"></a>Дополнительная информация
Теперь, когда вы знаете, как напрямую считывать данные из хранилища и записывать данные в хранилища, посмотрим, как сделать это с помощью диспетчера состояний.

> [!div class="nextstepaction"]
> [Сохранение состояния с помощью свойств conversation и user](bot-builder-howto-v4-state.md)


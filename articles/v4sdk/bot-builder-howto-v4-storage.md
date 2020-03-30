---
title: Непосредственная запись в хранилище в службе Bot
description: Сведения о том, как выполнять операции чтения и записи непосредственно в хранилище с помощью пакета SDK Bot Framework для .NET.
keywords: хранилище, чтение и запись, хранилище в памяти, eTag
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 35f113bacb84aa9e712ffe112780f0d121c1423e
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117739"
---
# <a name="write-directly-to-storage"></a>Запись данных напрямую в хранилище

[!INCLUDE[applies-to](../includes/applies-to.md)]

Вы можете выполнять операции записи и чтения непосредственно в объекте хранилища, не используя ПО промежуточного слоя или объект контекста. Это может требоваться для данных, которые бот использует для сохранения беседы, или данных, которые поступают из источника за пределами потока общения бота. В рамках этой модели хранилища данные считываются непосредственно из хранилища без использования диспетчера состояний. Примеры кода в этой статье демонстрируют, как выполнять операции записи и чтения в хранилище с использованием **хранилища в памяти**, **Cosmos DB**, **хранилища BLOB-объектов**, **хранилища таблиц Azure** и **хранилища расшифровок в больших двоичных объектах Azure**.

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, создайте [бесплатную](https://azure.microsoft.com/free/) учетную запись Azure, прежде чем начинать работу.
- Ознакомьтесь с руководством по локальному созданию бота для [C#](https://aka.ms/bot-framework-www-c-sharp-quickstart), [Node.js](https://aka.ms/bot-framework-www-node-js-quickstart) или [Python](https://aka.ms/bot-framework-www-node-python-quickstart).
- Шаблон для пакета SDK Bot Framework версии 4 для [C#](https://aka.ms/bot-vsix) или [Node.JS](https://nodejs.org) и [Yeoman](http://yeoman.io).

## <a name="about-this-sample"></a>Об этом примере

Пример кода в этой статье определяет структуру базового эхо-бота. Добавив дополнительный код (см. ниже), вы сможете расширить функции этого бота. Этот расширенный код создает список для хранения вводимых пользователем данных по мере получения. На каждом шаге полный список этих данных возвращается пользователю. Структура данных, содержащая этот список, записывается в хранилище в конце соответствующего шага. Мы рассмотрим разные типы хранилищ по мере добавления расширенных функций в пример кода.

## <a name="memory-storage"></a>Хранилище в памяти

Пакет SDK Bot Framework позволяет хранить вводимые пользователем данные с помощью размещенного в памяти хранилища. Хранилище в памяти используется только для тестирования и не предназначено для рабочей среды. Хранилище в памяти является непостоянным и временным, так как его данные очищаются при каждом запуске бота. Постоянные хранилища, такие как хранилище базы данных, лучше всего подходят для ботов в рабочей среде. Прежде чем публиковать бот, настройте для использования хранилище **Cosmos DB**, **хранилище BLOB-объектов** или [**хранилище таблиц Azure**](~/nodejs/bot-builder-nodejs-state-azure-table-storage.md).

## <a name="build-a-basic-bot"></a>Создание базового бота

Остальная часть этой статьи описывает использование эхо-бота. Пример кода Echo Bot можно создать локально для [C#](https://aka.ms/bot-framework-www-c-sharp-quickstart), [JS](https://aka.ms/bot-framework-www-node-js-quickstart) и [Python](https://aka.ms/bot-framework-www-node-python-quickstart).

### <a name="c"></a>[C#](#tab/csharp)

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

### <a name="javascript"></a>[JavaScript](#tab/javascript)

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
                await turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                await turnContext.sendActivity(`Write failed of UtteranceLogJS: ${err}`);
            }
        }
        else{
            await turnContext.sendActivity(`Creating and saving new utterance log`);
            var turnNumber = 1;
            storeItems["UtteranceLogJS"] = { UtteranceList: [`${utterance}`], "eTag": "*", turnNumber }
            // Gather info for user message.
            var storedString = storeItems.UtteranceLogJS.UtteranceList.toString();
            var numStored = storeItems.UtteranceLogJS.turnNumber;

            try {
                await storage.write(storeItems)
                await turnContext.sendActivity(`${numStored}: The list is now: ${storedString}`);
            } catch (err) {
                await turnContext.sendActivity(`Write failed: ${err}`);
            }
        }
    }
    catch (err){
        await turnContext.sendActivity(`Read rejected. ${err}`);
    }
}

module.exports.MyBot = MyBot;

```

### <a name="python"></a>[Python](#tab/python)

**bot.py**

```py
from botbuilder.core import ActivityHandler, TurnContext, StoreItem, MemoryStorage


class UtteranceLog(StoreItem):
    """
    Class for storing a log of utterances (text of messages) as a list.
    """

    def __init__(self):
        super(UtteranceLog, self).__init__()
        self.utterance_list = []
        self.turn_number = 0
        self.e_tag = "*"


class MyBot(ActivityHandler):
    """
    Represents a bot saves and echoes back user input.
    """

    def __init__(self):
        self.storage = MemoryStorage()

    async def on_message_activity(self, turn_context: TurnContext):
        utterance = turn_context.activity.text

        # read the state object
        store_items = await self.storage.read(["UtteranceLog"])

        if "UtteranceLog" not in store_items:
            # add the utterance to a new state object.
            utterance_log = UtteranceLog()
            utterance_log.utterance_list.append(utterance)
            utterance_log.turn_number = 1
        else:
            # add new message to list of messages existing state object.
            utterance_log: UtteranceLog = store_items["UtteranceLog"]
            utterance_log.utterance_list.append(utterance)
            utterance_log.turn_number = utterance_log.turn_number + 1

        # Show user list of utterances.
        await turn_context.send_activity(f"{utterance_log.turn_number}: "
                                         f"The list is now: {','.join(utterance_log.utterance_list)}")

        try:
            # Save the user message to your Storage.
            changes = {"UtteranceLog": utterance_log}
            await self.storage.write(changes)
        except Exception as exception:
            # Inform the user an error occurred.
            await turn_context.send_activity("Sorry, something went wrong storing your message!")
```

---

### <a name="start-your-bot"></a>Запуск бота

Запустите бот на локальном компьютере.

### <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота

Установите [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme). После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Create a new bot configuration** (Создать конфигурацию бота) на вкладке Welcome (Приветствие) эмулятора.
2. Заполните поля для подключения к боту, используя сведения на веб-странице, отображаемой при запуске бота.

### <a name="interact-with-your-bot"></a>Взаимодействие с ботом

Отправьте сообщение боту. Он отобразит список полученных сообщений.

![Окно Bot Framework Emulator](./media/emulator-direct-storage-test.png)

## <a name="using-cosmos-db"></a>Использование Cosmos DB

Начав использовать хранилище в памяти, мы изменим код, чтобы начать работу с Azure Cosmos DB. Cosmos DB — это глобально распределенная многомодельная база данных Майкрософт. Azure Cosmos DB позволяет гибко и независимо масштабировать пропускную способность и ресурсы хранилища в любом количестве регионов Azure. Она гарантирует пропускную способность, задержку, доступность и согласованность в соответствии с комплексными Соглашениями об уровне обслуживания (SLA).

### <a name="set-up"></a>Настройка

Чтобы использовать в боте Cosmos DB, необходимо создать ресурс базы данных, прежде чем приступать к написанию кода. См. подробнее о создании приложения и базы данных Cosmos DB для [dotnet](https://aka.ms/Bot-framework-create-dotnet-cosmosdb) и [Node.js](https://aka.ms/Bot-framework-create-nodejs-cosmosdb).

### <a name="create-your-database-account"></a>Создание учетной записи базы данных

1. В новом окне браузера войдите на [портал Azure](https://portal.azure.com).

    ![Создание учетной записи базы данных Cosmos DB](./media/create-cosmosdb-database.png)

2. Выберите **Создать ресурс > Базы данных > Azure Cosmos DB**.

    ![Страница создания учетной записи Cosmos DB](./media/cosmosdb-new-account-page.png)

3. На **странице создания учетной записи** укажите сведения о **подписке** и **группе ресурсов**. Укажите уникальное имя в поле **Имя учетной записи**. Это имя будет включено в URL-адрес для доступа к данным. В поле **API**выберите **Core(SQL)** и укажите ближайшее к вам **расположение**, чтобы ускорить доступ к данным.
4. Щелкните **Просмотреть и создать**.
5. Проверив сведения, щелкните **Создать**.

Создание учетной записи займет несколько минут. Подождите, пока на портале не откроется страница с сообщением "Поздравляем! Ваша учетная запись Azure Cosmos DB создана".

### <a name="add-a-database"></a>Добавление базы данных

1. Перейдите на страницу **Обозреватель данных** для созданной учетной записи Cosmos DB, а затем выберите **Создать базу данных** в раскрывающемся списке рядом с кнопкой **Создать контейнер**. В правой части окна откроется панель, на которой можно ввести сведения о новом контейнере.

    ![Cosmos DB](./media/create-cosmosdb-database-resource.png)

2. Введите идентификатор для новой базы данных и задайте пропускную способность (необязательно, можно изменить позднее), а затем щелкните **ОК**, чтобы создать базу данных. Запишите идентификатор базы данных, чтобы использовать его позже для настройки бота.

    ![Cosmos DB](./media/create-cosmosdb-database-resource-details.png)

3. Завершив создание учетной записи Cosmos DB и базы данных, скопируйте некоторые значения для интеграции новой базы данных в бота.  Чтобы получить их, перейдите на вкладку **Ключи** в разделе параметров базы данных, которую вы создали в учетной записи Cosmos DB.  На этой странице вам потребуются значения конечной точки Cosmos DB (**URI**) и ключа авторизации (**PRIMARY KEY**).

    ![Ключи Cosmos DB](./media/comos-db-keys.png)

Теперь у вас есть учетная запись Cosmos DB с базой данных и перечисленные ниже сведения для настройки бота.

- Конечная точка Cosmos DB
- Ключ авторизации
- Идентификатор базы данных

### <a name="add-configuration-information"></a>Добавление сведений о конфигурации

В этом примере мы используем короткий и простой фрагмент конфигурации, который добавляет хранилище Cosmos DB.  Используйте ранее сохраненные сведения, чтобы задать конечную точку, ключ авторизации и идентификатор базы данных.  Наконец, выберите подходящее имя для контейнера, который будет создан в базе данных для хранения состояния бота. В приведенном ниже примере этот контейнер будет называться bot-storage.

> [!NOTE]
> Не следует создавать контейнер самостоятельно. Ваш бот создаст его одновременно с внутренним клиентом Cosmos DB и правильно настроит для сохранения состояния бота.

### <a name="c"></a>[C#](#tab/csharp)

**EchoBot.cs**

```csharp
public class EchoBot : ActivityHandler
{
   private const string CosmosServiceEndpoint = "<your-cosmos-db-URI>";
   private const string CosmosDBKey = "<your-authorization-key>";
   private const string CosmosDBDatabaseId = "<your-database-id>";
   private const string CosmosDBContainerId = "bot-storage";
   ...

}
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

Добавьте следующие сведения в файл `.env`.

Файл с расширением **.env**

```javascript
DB_SERVICE_ENDPOINT="<your-cosmos-db-URI>"
AUTH_KEY="<your-authorization-key>"
DATABASE_ID="<your-database-id>"
CONTAINER="bot-storage"
```

### <a name="python"></a>[Python](#tab/python)

Добавьте следующие сведения в файл `bot.py`.

```python
COSMOSDB_SERVICE_ENDPOINT = "<your-cosmos-db-URI>"
COSMOSDB_KEY = "<your-authorization-key>"
COSMOSDB_DATABASE_ID = "<your-database-id>"
COSMOSDB_CONTAINER_ID = "bot-storage"
```

---

#### <a name="installing-packages"></a>Установка пакетов

Убедитесь, что вы установили пакеты, необходимые для работы с Cosmos DB.

### <a name="c"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

Вы можете добавить ссылки на пакет botbuilder-azure в свой проект с помощью npm.
>**Примечание.** Этот пакет npm зависит от версии Python, установленной на компьютере разработки. Если вы еще не установили Python, перейдите по следующей ссылке: [Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure
```

При необходимости вы можете получить пакет dotnet с помощью npm, чтобы получить доступ к параметрам файла `.env`.

```powershell
npm install --save dotenv
```

### <a name="python"></a>[Python](#tab/python)

Вы можете добавить ссылки на пакет botbuilder-azure в свой проект с помощью pip.

```powershell
pip install botbuilder-azure
```

---

### <a name="implementation"></a>Реализация

> [!NOTE]
> В версии 4.6 появился новый поставщик хранилища Cosmos DB `CosmosDbPartitionedStorage`. Существующие боты, которые используют прежний поставщик `CosmosDbStorage`, должны и далее использовать `CosmosDbStorage`. Такие боты продолжат нормально работать. Для новых ботов следует использовать `CosmosDbPartitionedStorage`, так как секционирование повышает производительность.

### <a name="c"></a>[C#](#tab/csharp)

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
   private static readonly CosmosDbStorage _myStorage = new CosmosDbPartitionedStorage(new CosmosDbPartitionedStorageOptions
   {
        CosmosDbEndpoint = CosmosServiceEndpoint,
        AuthKey = CosmosDBKey,
        DatabaseId = CosmosDBDatabaseId,
        ContainerId = CosmosDBContainerId,
   });

   ...
}

```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

Следующий пример кода соответствует примеру с [хранилищем в памяти](#memory-storage), но имеет некоторые отличия.

Импортируйте `CosmosDbPartitionedStorage` из `botbuilder-azure` и настройте dotenv для чтения файла `.env`.

**bot.js**

```javascript
const { CosmosDbPartitionedStorage } = require("botbuilder-azure");
```

Закомментируйте хранилище в памяти, заменив ссылкой на Cosmos DB.

**bot.js**

```javascript
// initialized to access values in .env file.
const ENV_FILE = path.join(__dirname, '.env');
require('dotenv').config({ path: ENV_FILE });

// Create local Memory Storage - commented out.
// var storage = new MemoryStorage();

// Create access to CosmosDb Storage - this replaces local Memory Storage.
var storage = new CosmosDbPartitionedStorage({
    cosmosDbEndpoint: process.env.DB_SERVICE_ENDPOINT,
    authKey: process.env.AUTH_KEY,
    databaseId: process.env.DATABASE_ID,
    containerId: process.env.CONTAINER
})

```

### <a name="python"></a>[Python](#tab/python)

Следующий пример кода соответствует примеру с [хранилищем в памяти](#memory-storage), но имеет некоторые отличия.

Запросите `CosmosDbStorage` из `botbuilder-azure` и создайте объект CosmosDBStorage.

**bot.py**

```py
from botbuilder.azure import CosmosDbStorage, CosmosDbConfig
```

Закомментируйте хранилище в памяти в `__init__`, заменив ссылкой на Cosmos DB.  Используйте конечную точку, ключ аутентификации, а также идентификаторы базы данных и контейнера, с которыми вы работали выше.

**bot.py**

```py
def __init__(self):
    cosmos_config = CosmosDbConfig(
        endpoint=COSMOSDB_SERVICE_ENDPOINT,
        masterkey=COSMOSDB_KEY,
        database=COSMOSDB_DATABASE_ID,
        container=COSMOSDB_CONTAINER_ID
    )
    self.storage = CosmosDbStorage(cosmos_config)
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

1. В новом окне браузера войдите на [портал Azure](https://portal.azure.com).

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

### <a name="c"></a>[C#](#tab/csharp)

```powershell
Install-Package Microsoft.Bot.Builder.Azure
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

Вы можете добавить ссылки на пакет botbuilder-azure в свой проект с помощью npm.
>**Примечание.** Этот пакет npm зависит от версии Python, установленной на компьютере разработки. Если вы еще не установили Python, перейдите по следующей ссылке: [Python.org](https://www.python.org/downloads/)

```powershell
npm install --save botbuilder-azure
```

При необходимости вы можете получить пакет dotnet с помощью npm, чтобы получить доступ к параметрам файла `.env`.

```powershell
npm install --save dotenv
```

### <a name="python"></a>[Python](#tab/python)

Вы можете добавить ссылки на пакет botbuilder-azure в свой проект с помощью pip.

```powershell
pip install botbuilder-azure
```

---

### <a name="implementation"></a>Реализация

### <a name="c"></a>[C#](#tab/csharp)

**EchoBot.cs**

```csharp
using Microsoft.Bot.Builder.Azure;
```

Измените строку кода со свойством _myStorage_ и присвойте этому свойству значение существующей учетной записи хранилища BLOB-объектов.

**EchoBot.cs**

```csharp
private static readonly AzureBlobStorage _myStorage = new AzureBlobStorage("<your-blob-storage-account-string>", "<your-blob-storage-container-name>");
```

### <a name="javascript"></a>[JavaScript](#tab/javascript)

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
const ENV_FILE = path.join(__dirname, '.env');
require('dotenv').config({ path: ENV_FILE });
```

Теперь обновите код, чтобы связать _storage_ с существующей учетной записью хранилища BLOB-объектов, закомментировав предыдущие определения хранилища и добавив следующее.

**bot.js**

```javascript
var storage = new BlobStorage({
    containerName: process.env.BLOB_NAME,
    storageAccountOrConnectionString: process.env.BLOB_STRING
});
```

### <a name="python"></a>[Python](#tab/python)

Следующий пример кода соответствует примеру с [хранилищем в памяти](#memory-storage), но имеет некоторые отличия.

Запросите `BlobStorage` из `botbuilder-azure` и создайте объект CosmosDBStorage.

**bot.py**

```py
from botbuilder.azure import BlobStorage, BlobStorageSettings
```

Закомментируйте хранилище в памяти в `__init__`, заменив ссылкой на Cosmos DB.  Используйте имя контейнера и строку подключения, с которыми вы работали выше.

**bot.py**

```py
def __init__(self):
    blob_settings = BlobStorageSettings(
        container_name="<your_container_name>",
        connection_string="<your_connection_string>"
    )
    self.storage = BlobStorage(blob_settings)
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

**ПРИМЕЧАНИЕ. JavaScript и Python сейчас не поддерживают AzureBlobTranscriptStore.  Инструкции ниже предназначены только для C#.**

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

## <a name="additional-information"></a>Дополнительные сведения

### <a name="manage-concurrency-using-etags"></a>Управление параллелизмом с помощью тегов eTag
В нашем примере кода бота мы установили свойство `eTag` каждого экземпляра `IStoreItem` равным `*`. Член `eTag` (тег сущности) объекта хранилища используется в Cosmos DB для управления параллелизмом. `eTag` указывает базе данных, что делать, если другой экземпляр бота изменил объект в том же хранилище, в которое записывает данные ваш бот.

<!-- define optimistic concurrency -->

#### <a name="last-write-wins---allow-overwrites"></a>Приоритет последней записи — разрешить перезапись
Если указать звездочку (`*`) в качестве значения свойства `eTag`, это означает, то приоритет принадлежит последнему боту, который выполняет запись. При создании хранилища данных можно указать значение `*` для свойства `eTag`. Это означает, что вы не сохранили записываемые данные, или что вы хотите, чтобы последний бот перезаписал все ранее сохраненные свойства. Если параллелизм не является проблемой для вашего бота, укажите для свойства `eTag` значение `*`. Это разрешает перезапись.

#### <a name="maintain-concurrency-and-prevent-overwrites"></a>Поддержание параллелизма и предотвращение перезаписи
Если нужно предотвратить параллельный доступ к свойству и избежать перезаписи данных другим экземпляром бота, при сохранении данных в Cosmos DB используйте другое значение свойства `eTag`, отличное от `*`. Если бот попытается сохранить данные о состоянии, а параметр `eTag` не соответствует параметру `eTag` в хранилище, бот получит сообщение об ошибке `etag conflict key=`. <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

По умолчанию хранилище Cosmos DB проверяет равенство свойства `eTag` в объекте хранилища каждый раз, когда бот записывает данные в этот элемент, а затем включает в это свойство новое уникальное значение после каждой операции записи. Если значение свойства `eTag` во время записи не соответствует значению свойства `eTag` в хранилище, это означает, что другой бот или поток изменили данные.

Например, предположим, что ваш бот должен изменять сохраненные заметки, но вы не хотите, чтобы он перезаписывал изменения, выполненные другим экземпляром бота. Если другой экземпляр бота внес изменения, пользователь должен изменять последнюю версию со всеми внесенными изменениями.

### <a name="c"></a>[C#](#tab/csharp)

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

### <a name="javascript"></a>[JavaScript](#tab/javascript)

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

### <a name="python"></a>[Python](#tab/python)

Во-первых, создайте класс, который реализует `StoreItem`.

**bot.py**

```py
class Note(StoreItem):
    def __init__(self, name: str, contents: str, e_tag="*"):
        super(Note, self).__init__()
        self.name = name
        self.contents = contents
        self.e_tag = e_tag
```

Затем создайте начальную заметку, создав объект хранилища и добавив объект в это хранилище.

**bot.py**

```py
# create a note for the first time, with a non-null, non-* ETag.
changes = {"Note": Note(name="Shopping List", contents="eggs", e_tag="x")}

await self.storage.write(changes)
```

Впоследствии обновите заметку, сохранив значение `eTag`, прочитанное из хранилища.

**bot.py**

```py
store_items = await self.storage.read(["Note"])
    note = store_items["Note"]
    note.contents = note.contents + ", bread"

    changes = {"Note": note}
    await self.storage.write(changes)
```

Если заметка в хранилище была изменена перед записью ваших изменений, то при вызове `write` возникнет исключение.

---

Для поддержания параллелизма всегда считывайте значение свойства из хранилища, затем изменяйте прочитанное свойство, чтобы сохранить `eTag`. При считывании сведений о пользователях из хранилища ответ будет содержать свойство eTag. Если вы изменили данные и записываете обновленные данные в хранилище, ваш запрос должен включать свойство eTag, содержащее то же значение, которое было прочитано ранее. Однако при записи объекта со свойством `eTag`, имеющим значение `*`, будет разрешена перезапись любых других изменений.

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы знаете, как напрямую считывать данные из хранилища и записывать данные в хранилища, посмотрим, как сделать это с помощью диспетчера состояний.

> [!div class="nextstepaction"]
> [Сохранение состояния с помощью свойств conversation и user](bot-builder-howto-v4-state.md)

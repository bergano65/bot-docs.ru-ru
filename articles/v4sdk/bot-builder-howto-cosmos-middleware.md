---
title: Создание ПО промежуточного слоя с помощью Cosmos DB | Документы Майкрософт
description: Сведения о создании пользовательского ПО промежуточного слоя, которое выполняет ведение журнала в Cosmos DB.
keywords: ПО промежуточного слоя, Cosmos DB, база данных, Azure, onTurn
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3f36b00a91644859445676676374f7d321087800
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42906105"
---
# <a name="create-middleware-that-logs-in-cosmos-db"></a>Создание ПО промежуточного слоя, которое выполняет ведение журнала в Cosmos DB

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Хотя в состав пакета SDK входит полезное ПО промежуточного слоя, в некоторых ситуациях необходимо реализовать собственное ПО промежуточного слоя для достижения желаемой цели.

В этом примере мы создадим промежуточный слой, который подключается к Cosmos DB и записывает в журнал все полученные сообщения и отправленные ответы. Приведенный здесь код доступен в составе полного исходного кода, предоставляемого с нашими [примерами](../dotnet/bot-builder-dotnet-samples.md).

## <a name="set-up"></a>Настройка

Перед тем как перейти к написанию кода, необходимо кое-что сделать.

### <a name="create-your-database"></a>Создание базы данных

Во-первых, нужно выбрать место, в котором будет находиться наша база данных.  Это можно сделать через учетную запись Azure или (для тестирования) воспользоваться [эмулятором Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator). Независимо от выбранного метода нам понадобятся URI конечной точки и ключ базы данных.

В этом примере и в приведенном коде используется эмулятор. Конечная точка и ключ указаны для учетной записи тестирования Cosmos DB. Они указаны в документации эмулятора и не могут использоваться в рабочей среде.

Если вы хотите использовать настоящую базу данных Azure Cosmos DB, обратитесь к шагу 1 в разделе [Приступая к работе с Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/sql-api-get-started). После этого URI конечной точки и ключ будут указаны на вкладке **Ключи** в параметрах базы данных. Эти значения потребуется указать в файле конфигурации ниже.

### <a name="build-a-basic-bot"></a>Создание базового бота

Остальная часть этого раздела посвящена созданию базового бота, такого как бот HelloBot, описанный в обзорной статье. Для подключения к боту, взаимодействия с ботом и проверки бота можно использовать [Эмулятор Bot Framework](https://github.com/Microsoft/BotFramework-Emulator).

Наряду со ссылками на библиотеки для разработки стандартных ботов, использующих Bot Framework, необходимо добавить ссылки на следующие пакеты NuGet:

* Microsoft.Azure.Documentdb.Core
* System.Configuration.ConfigurationManager

Эти библиотеки используются для взаимодействия с базой данных и для использования файла конфигурации соответственно.

### <a name="add-configuration-file"></a>Добавление файла конфигурации

Использовать файлы конфигурации рекомендуется по ряду причин. Поэтому мы будем использовать файл конфигурации. Наша конфигурация короткая и простая, однако по мере усложнения бота вы сможете добавить в этот файл дополнительные параметры конфигурации.

# <a name="ctabcs"></a>[C#](#tab/cs)
1. Добавьте в свой проект файл с именем `app.config`.
2. Добавьте в него следующее содержимое. Конечная точка и ключ указаны для локального эмулятора. Но вы можете изменить их в соответствии со своим экземпляром Cosmos DB.
    ```
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="EmulatorDbUrl" value="https://localhost:8081"/>
            <add key="EmulatorDbKey" value="C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw=="/>
        </appSettings>
    </configuration>
    ```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
1. Добавьте в свой проект файл с именем `config.js`.
2. Добавьте в него следующее содержимое. Конечная точка и ключ указаны для локального эмулятора. Но вы можете изменить их в соответствии со своим экземпляром Cosmos DB.
    ```javascript
        var config = {}

        config.EmulatorServiceEndpoint = "https://localhost:8081";
        config.EmulatorAuthKey = "C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWE+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
        config.database = {
            "id": "Tasks"
        };
        config.collection = {
            "id": "Items"
        };

        module.exports = config;
    ```

---

Если вы используете Cosmos DB, то можете заменить значения двух ключей, указанных выше, на свои параметры Cosmos DB. Кроме того, можно добавить в свою конфигурацию еще два ключа, что позволит вам переключаться между эмулятором для тестирования и Cosmos DB:

# <a name="ctabcs"></a>[C#](#tab/cs)
```
    <add key="ActualDbUrl" value="<your database URI>"/>
    <add key="ActualDbKey" value="<your database key>"/>
```
 Если вы добавите два дополнительных ключа и захотите использовать настоящую базу данных, обязательно укажите правильный ключ в конструкторе ниже вместо `EmulatorDbUrl` и `EmulatorDbKey`.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```
    config.ActualServiceEndpoint = "your database URI;
    config.ActualAuthKey = "your database key";
```

Если вы добавите два дополнительных ключа и захотите использовать настоящую базу данных, обязательно укажите правильный ключ в конструкторе ниже вместо `EmulatorServiceEndpoint` и `EmulatorAuthKey`.

---



## <a name="creating-your-middleware"></a>Создание ПО промежуточного слоя

# <a name="ctabcs"></a>[C#](#tab/cs)
Перед началом работы над логикой ПО промежуточного слоя создайте новый класс в проекте бота для ПО промежуточного слоя. Создав этот класс, измените пространство имен на то пространство имен, которое используется для нашего бота, `Microsoft.Bot.Samples`. Вы видите, что мы добавили ссылки на несколько новых библиотек:

* `Newtonsoft.Json;`
* `Microsoft.Azure.Documents;`
* `Microsoft.Azure.Documents.Client;`
* `Microsoft.Azure.Documents.Linq;`

Различные `Microsoft.Azure.Documents.*` предназначены для различных этапов взаимодействия с базой данных, а `Newtonsoft.Json` помогает нам анализировать код JSON, используемый при обмене информацией с базой данных.

Наконец, в каждом компоненте ПО промежуточного слоя реализуется `IMiddleware`, и для правильной работы ПО промежуточного слоя нам необходимо определить соответствующие обработчики.

```cs
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using System.Configuration;
using Newtonsoft.Json;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using Microsoft.Azure.Documents;
using Microsoft.Azure.Documents.Client;
using Microsoft.Azure.Documents.Linq;

namespace Microsoft.Bot.Samples
{
    public class CosmosMiddleware : IMiddleware
    {
        ...
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
Перед началом работы над логикой ПО промежуточного слоя необходимо создать собственное ПО промежуточного слоя, которое начинается с метода `onTurn()`. 

```javascript
adapter.use({onTurn: async (context, next) =>{
    // Middleware logic here...
}})
    
```

---

#### <a name="defining-local-variables"></a>Определение локальных переменных

# <a name="ctabcs"></a>[C#](#tab/cs)
Теперь нам необходимы локальные переменные для работы с базой данных и класс для хранения сведений, которые необходимо записать в журнал. Этот информационный класс под названием `Log` определяет, как будут называться свойства JSON, связанные с каждым элементом. Мы вернемся к нему позже.

```cs
    private string Endpoint;
    private string Key;
    private static readonly string Database = "BotData";
    private static readonly string Collection = "BotCollection";
    public DocumentClient docClient;

    public class Log
    {
        [JsonProperty("Time")]
        public string Time;

        [JsonProperty("MessageReceived")]
        public string Message;

        [JsonProperty("ReplySent")]
        public string Reply;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
Теперь нам необходима локальная переменная для хранения сведений, которые необходимо записать в журнал. В ПО промежуточного слоя необходимо обратиться к объекту conversationState, который использует Cosmos DB в качестве поставщика хранилища. В качестве объекта состояния, который мы будет считывать и записывать, используется переменная `info`. 

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(storage);
adapter.use(conversationState);

adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);
    // More middleware logic 
}})
```

--- 


#### <a name="initialize-database-connection"></a>Инициализация подключения к базе данных

# <a name="ctabcs"></a>[C#](#tab/cs)
Наш конструктор ПО промежуточного слоя получает необходимую информацию из файла конфигурации, определенного выше, и создает объект `DocumentClient`, который позволяет считывать данные из базы данных и записывать данные в базу данных. Мы также настроили базу данных и коллекцию.

После использования объект `DocumentClient` необходимо удалить, это происходит в деструкторе.

При создании базы данных предпринимается попытка считывания базы данных и затем — коллекции в этой базе данных. Если мы получаем исключение `NotFound`, мы перехватываем его и создаем базу данных или коллекцию вместо того, чтобы передавать исключение вверх по стеку. В большинстве случаев база данных и коллекция уже будут созданы, но это позволит нам не беспокоиться о создании базы данных перед первым запуском приложения. В примере кода база данных и коллекция для простоты разделены на две функции, но в приведенном ниже фрагменте кода они объединены.

Дополнительные сведения о CosmosDB см. на [веб-сайте](https://docs.microsoft.com/en-us/azure/cosmos-db/) CosmosDB. Для оставшейся части этой статьи мы кратко ознакомимся с самой Cosmos DB и сосредоточимся на том, что необходимо вашему боту для использования Cosmos DB.

Определения `Key`, `Endpoint` и `docClient` были включены в приведенный выше пример кода. Мы скопировали их в этот раздел для ясности.

```cs
    private string Endpoint;
    private string Key;
    // ...
    public DocumentClient docClient;
    // ...

    public CosmosMiddleware()
    {
        Endpoint = ConfigurationManager.AppSettings["EmulatorDbUrl"];
        Key = ConfigurationManager.AppSettings["EmulatorDbKey"];
        docClient = new DocumentClient(new Uri(Endpoint), Key);
        CreateDatabaseAndCollection().ConfigureAwait(false);
    }

    ~CosmosMiddleware()
    {
        docClient.Dispose();
    }

    private async Task CreateDatabaseAndCollection()
    {
        try
        {
            await docClient.ReadDatabaseAsync(UriFactory.CreateDatabaseUri(Database));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDatabaseAsync(new Database { Id = Database });
            }
            else
            {
                throw;
            }
        }

        try
        {
            await docClient.ReadDocumentCollectionAsync(UriFactory.CreateDocumentCollectionUri
                (Database, Collection));
        }
        catch (DocumentClientException e)
        {
            if (e.StatusCode == System.Net.HttpStatusCode.NotFound)
            {
                await docClient.CreateDocumentCollectionAsync(
                    UriFactory.CreateDatabaseUri(Database),
                    new DocumentCollection { Id = Collection });
            }
            else
            {
                throw;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
При создании базы данных предпринимается попытка считывания базы данных и затем — коллекции в этой базе данных. Если мы получаем исключение `undefined`, мы перехватываем его и создаем объект `log`, в который записываем пустой массив. В большинстве случаев объект `log` уже будет создан, но это позволит нам не беспокоиться о создании базы данных перед первым запуском приложения. В примере кода мы проверяем, определен ли объект `info.log`. Если да, мы записываем в него пустой массив.

```javascript
adapter.use({onTurn: async (context, next) =>{

    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    // More bot logic below
}})
```
---

#### <a name="read-from-database-logic"></a>Логика чтения из базы данных

Последняя вспомогательная функция считывает данные из базы данных и возвращает последнее указанное количество записей. Стоит отметить, что для получения данных существуют более удобные и правильные методы, особенно для хранилищ данных значительно большего размера.

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task<string> ReadFromDatabase(int numberOfRecords)
    {
        var documents = docClient.CreateDocumentQuery<Log>(
                UriFactory.CreateDocumentCollectionUri(Database, Collection))
                .AsDocumentQuery();
        List<Log> messages = new List<Log>();
        while (documents.HasMoreResults)
        {
            messages.AddRange(await documents.ExecuteNextAsync<Log>());
        }

        // Create a sublist of messages containing the number of requested records.
        List<Log> messageSublist = messages.GetRange(messages.Count - numberOfRecords, numberOfRecords);

        string history = "";

        // Send the last 3 messages.
        foreach (Log logEntry in messageSublist)
        {
            history += ("Message was: " + logEntry.Message + " Reply was: " + logEntry.Reply + "  ");
        }

        return history;
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Приведенный ниже код находится в функции `onTurn()` и будет вызван, если пользователь введет строку "history".

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through the info.log array
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    }
    //...
}})

```

---

#### <a name="define-onturn"></a>Определение метода OnTurn()

Стандартный метод ПО промежуточного слоя `OnTurn()` выполняет остальную часть работы. Мы хотим записывать данные в журнал только в том случае, если текущее действие — сообщение. Мы проверяем это до и после вызова `next()`.

Первое, что нужно проверить, — не является ли это сообщение особым сообщением, в котором пользователь запрашивает последние записи журнала. Если это особое сообщение, мы считываем последние записи из базы данных и отправляем их в диалог. Так как на этом текущее действие закончено, мы замыкаем конвейер вместо того, чтобы продолжать выполнение.

Чтобы получить ответ, который отправляет бот, мы создаем обработчик, который будет получать эти ответы, каждый раз при создании сообщения, что можно увидеть в лямбда-выражении, которое передается в метод `OnSendActivity()`. Оно формирует строку для сбора всех сообщений, отправленных с помощью метода `SendActivity()`, для этого объекта контекста.

После завершения работы конвейера в `next()` мы собираем данные журнала и записываем их в базу данных. 

Посмотрите в окно обозревателя данных Cosmos DB после отправки нескольких сообщений боту, и вы должны увидеть свои данные в отдельных записях. Просмотрев одну из таких записей, вы увидите, что первые три элемента представляют собой три значения для данных журнала с именем в виде строки, указанной в соответствующем свойстве `JsonProperty` для этих значений.

# <a name="ctabcs"></a>[C#](#tab/cs)
```cs
    public async Task OnTurn
        (ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        string botReply = "";

        if (context.Activity.Type == ActivityTypes.Message)
        {
            if (context.Activity.Text == "history")
            {
                // Read last 3 responses from the database, and short circuit future execution.
                await context.SendActivity(await ReadFromDatabase(3));
                return;
            }

            // Create a send activity handler to grab all response activities 
            // from the activity list.
            context.OnSendActivity(async (activityContext, activityList, activityNext) =>
            {
                foreach (Activity activity in activityList)
                {
                    botReply += (activity.Text + " ");
                }
                await activityNext();
            });
        }

        // Pass execution on to the next layer in the pipeline.
        await next();

        // Save logs for each conversational exchange only.
        if (context.Activity.Type == ActivityTypes.Message)
        {
            // Build a log object to write to the database.
            var logData = new Log
            {
                Time = DateTime.Now.ToString(),
                Message = context.Activity.Text,
                Reply = botReply
            };

            // Write our log to the database.
            try
            {
                var document = await docClient.CreateDocumentAsync(UriFactory.
                    CreateDocumentCollectionUri(Database, Collection), logData);
            }
            catch (Exception ex)
            {
                // More logic for what to do on a failed write can be added here
                throw ex;
            }
        }
    }
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Процесс разработки на основе нашей функции `onTurn` описан выше.

```javascript
adapter.use({onTurn: async (context, next) =>{

    const utterance = (context.activity.text || '').trim().toLowerCase();
    const isMessage = context.activity.type === 'message';
    const info = conversationState.get(context);

    if(info.log == undefined){
        info.log = [];
    }

    if(isMessage && utterance == 'history'){
        // Loop through info.log 
        var logHistory = "";
        for(var i = 0; i < info.log.length; i++){
            logHistory += info.log[i] + " ";
        }
        await context.sendActivity(logHistory);
        // Short circuit the middleware
        return
    } else if(isMessage){
        // Store the users response
        info.log.push(`Message was: ${context.activity.text}`); 

        await context.onSendActivities(async (handlerContext, activities, handlerNext) => 
        {
            // Store the bot's reply
            info.log.push(`Reply was: ${activities[0].text}`);
            
            await handlerNext(); 
        });
        await next();
    }
}})

```

---

## <a name="sample-output"></a>Пример выходных данных

В полном примере бот использует библиотеку запросов для запроса имени пользователя и любимого числа пользователя. Сведения о библиотеке запросов можно найти в разделе, посвященном [отправке запросов пользователям](~/v4sdk/bot-builder-prompts.md).

Ниже приведен пример диалога с нашим ботом, созданным на основе приведенного выше кода.

![Пример диалога с использованием ПО промежуточного слоя и Cosmos](./media/cosmos-middleware-conversation.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

* [База данных Cosmos](https://docs.microsoft.com/en-us/azure/cosmos-db/)
* [Эмулятор Cosmos DB](https://docs.microsoft.com/en-us/azure/cosmos-db/local-emulator)


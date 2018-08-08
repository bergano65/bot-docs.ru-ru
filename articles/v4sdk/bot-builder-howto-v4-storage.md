---
title: Хранение данных | Документы Майкрософт
description: Сведения о том, как записать данные напрямую в хранилище с помощью пакета SDK Bot Builder для .NET версии 4.
keywords: хранилище, чтение и запись, хранилище в памяти, eTag
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 05/2/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 653ec6a1983dd59c485a91b2c08ea07d9f2a34c8
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305575"
---
# <a name="save-data-directly-to-storage"></a>Сохранение данных напрямую в хранилище

<!--
 Note for V4: You can write directly to storage without using the state manager. Therefore, this topic isn't called "managing state". State is in a separate topic.
 ## Manage state data by writing directly to storage
-->
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Вы можете записывать данные напрямую в хранилище без использования объекта контекста или ПО промежуточного слоя. Для этого используйте чтение и запись напрямую в объект хранилища.

В данном примере кода показано, как считать данные из хранилища и записать данные в хранилище. В данном примере хранилище — это файл, но вы можете легко изменить код, так чтобы инициализировать объект хранилища с хранилищем в памяти для хранилища таблиц Azure. 

# <a name="ctabcsharpechorproperty"></a>[C#](#tab/csharpechorproperty)
Мы определим объект и воспользуемся методами `IStorage.Write` и `IStorage.Read` для сохранения и получения состояния. 

В следующем примере каждое сообщение от пользователя добавляется в список. Структура данных, содержащая список, сохраняется в файл в каталоге, который передается в качестве аргумента в конструктор `FileStorage`.

Начните работу с шаблоном EchoBot Visual Studio в пакете SDK BotBuilder версии 4.
Отредактируйте файл `EchoBot.cs` . Добавьте следующие операторы using и замените определение класса `EchoBot`.

```csharp
using System.Collections.Generic;
using System.Linq;
```

```csharp
// In the constructor initialize file storage
public class EchoBot : IBot
{
    private readonly FileStorage _myStorage;

    public EchoBot()
    {
        _myStorage = new FileStorage(System.IO.Path.GetTempPath());
    }

    // Add a class for storing a log of utterances (text of messages) as a list
    public class UtteranceLog : IStoreItem
    {
        // A list of things that users have said to the bot
        public List<string> UtteranceList { get; private set; } = new List<string>();

        // The number of conversational turns that have occurred        
        public int TurnNumber { get; set; } = 0;

        public string eTag { get; set; } = "*";
    }

    // Replace the OnTurn in EchoBot.cs with the following:
    public async Task OnTurn(ITurnContext context)
    {
        var activityType = context.Activity.Type;

        await context.SendActivity($"Activity type: {context.Activity.Type}.");

        if (activityType == ActivityTypes.Message)
        {
            // *********** begin (create or add to log of messages)
            var utterance = context.Activity.Text;
            bool restartList = false;

            if (utterance.Equals("restart"))
            {
                restartList = true;
            }

            // Attempt to read the existing property bag
            UtteranceLog logItems = null;
            try
            {
                logItems = _myStorage.Read<UtteranceLog>("UtteranceLog").Result?.FirstOrDefault().Value;
            }
            catch (System.Exception ex)
            {
                await context.SendActivity(ex.Message);
            }

            // If the property bag wasn't found, create a new one
            if (logItems is null)
            {
                try
                {
                    // add the current utterance to a new object.
                    logItems = new UtteranceLog();
                    logItems.UtteranceList.Add(utterance);

                    await context.SendActivity($"The list is now: {string.Join(", ",logItems.UtteranceList)}");

                    var changes = new KeyValuePair<string, object>[]
                    {
                        new KeyValuePair<string, object>("UtteranceLog", logItems)
                    };
                    await _myStorage.Write(changes);
                }
                catch (System.Exception ex)
                {
                    await context.SendActivity(ex.Message);
                }
            }
            // logItems.ContainsKey("UtteranceLog") == true, we were able to read a log from storage
            else
            {
                // Modify its property
                if (restartList)
                {
                    logItems.UtteranceList.Clear();
                }
                else
                {
                    logItems.UtteranceList.Add(utterance);
                    logItems.TurnNumber++;
                }

                await context.SendActivity($"The list is now: {string.Join(", ", logItems.UtteranceList)}");

                var changes = new KeyValuePair<string, object>[]
                {
                        new KeyValuePair<string, object>("UtteranceLog", logItems)
                };
                await _myStorage.Write(changes);
            }
        }

        return;
    }
}
```
# <a name="javascripttabjsechoproperty"></a>[JavaScript](#tab/jsechoproperty)

В следующем примере каждое сообщение от пользователя добавляется в список. Структура данных, содержащая список, сохраняется в файл в каталоге, который передается в качестве аргумента в конструктор `FileStorage`.

Вставьте следующий код в `app.js`.
``` javascript
const { BotFrameworkAdapter, FileStorage, MemoryStorage, ConversationState, BotStateSet } = require('botbuilder');
const restify = require('restify');

// Create server.
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID,
    appPassword: process.env.MICROSOFT_APP_PASSWORD
});

// Add storage.
var storage = new FileStorage("C:/temp");
const conversationState = new ConversationState(storage);
adapter.use(new BotStateSet(conversationState));

// Listen for incoming activities.
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing.
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            let utterance = context.activity.text;
            let storeItems = await storage.read(["UtteranceLog"])
            try {
                // check result
                var utteranceLog = storeItems["UtteranceLog"];
                
                if (typeof (utteranceLog) != 'undefined') {
                    // log exists so we can write to it
                    storeItems["UtteranceLog"].UtteranceList.push(utterance);
                    
                    await storage.write(storeItems)
                    try {
                        context.sendActivity('Successful write.');
                    } catch (err) {
                        context.sendActivity(`Srite failed of UtteranceLog: ${err}`);
                    }

                } else {
                    context.sendActivity(`need to create new utterance log`);
                    storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }
                    await storage.write(storeItems)
                    try {
                        context.sendActivity('Successful write.');
                    } catch (err) {
                        context.sendActivity(`Write failed: ${err}`);
                    }
                }
            } catch (err) {
                context.sendActivity(`Read rejected. ${err}`);
            };

            await context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```
---

## <a name="manage-concurrency-using-etags"></a>Управление параллелизмом с помощью тегов eTag

В предыдущем примере вы установили значение `*` для свойства `eTag`. Член `eTag` (тег сущности) объекта хранилища используется для управления параллелизмом. `eTag` указывает, что делать, если другой экземпляр бота изменил объект в хранилище, в которое записывает данные ваш бот. 

<!-- define optimistic concurrency -->

### <a name="last-write-wins---allow-overwrites"></a>Приоритет последней записи — разрешить перезапись

Установите значение eTag, равное `*`, чтобы разрешить другим экземплярам бота перезаписывать ранее записанные данные. Если указать звездочку (`*`) в качестве значения свойства `eTag`, это означает, то приоритет принадлежит последнему боту, который выполняет запись. При создании хранилища данных можно указать значение `*` для свойства `eTag`. Это означает, что вы не сохранили записываемые данные, или что вы хотите, чтобы последний бот перезаписал все ранее сохраненные свойства. Если параллелизм не является проблемой для вашего бота, укажите значение `*` для свойства `eTag`, это означает, что перезапись допустима.

Это показано в следующем примере кода.

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)
Используйте класс `UtteranceLog`, который был определен ранее.
```csharp
// Add the current utterance to a new object and save it to storage.
logItems = new UtteranceLog();
logItems.UtteranceList.Add(utterance);
logItems.eTag = "*";

var changes = new KeyValuePair<string, object>[]
{
    new KeyValuePair<string, object>("UtteranceLog", logItems)
};
await _myStorage.Write(changes);

```
# <a name="javascripttabjstagoverwrite"></a>[JavaScript](#tab/jstagoverwrite)
Добавьте новое высказывание в журнал и разрешите перезапись.

```javascript
storeItems["UtteranceLog"] = { UtteranceList: [`${utterance}`], "eTag": "*" }
await storage.write(storeItems)
```
---

### <a name="maintain-concurrency-and-prevent-overwrites"></a>Поддержание параллелизма и предотвращение перезаписи
Чтобы предотвратить одновременный доступ к свойствам и избежать перезаписи изменений другим экземпляром бота, используйте значение параметра `eTag`, отличное от `*`. Если бот пытается сохранить данные о состоянии и свойство не соответствует параметру хранилища `eTag`, бот получает сообщение об ошибке `etag conflict key=`. <!-- To control concurrency of data that is stored using `IStorage`, the BotBuilder SDK checks the entity tag (ETag) for `Storage.Write()` requests. -->

По умолчанию хранилище проверяет свойство `eTag` на соответствие параметру объекта хранилища каждый раз, когда бот записывает данные в этот элемент, и записывает в это свойство новое уникальное значение после каждой записи. Если значение свойства `eTag` во время записи не соответствует значению свойства `eTag` в хранилище, это означает, что другой бот или поток изменили данные. 

Например, предположим, что ваш бот должен изменять сохраненные заметки, но вы не хотите, чтобы он перезаписывал изменения, выполненные другим экземпляром бота. Если другой экземпляр бота внес изменения, пользователь должен изменять последнюю версию со всеми внесенными изменениями.

# <a name="ctabcsetag"></a>[C#](#tab/csetag)
Во-первых, создайте класс, который реализует `IStoreItem`.
```csharp
public class Note : IStoreItem
{
    public string Name { get; set; }
    public string Contents { get; set; }
    public string eTag { get; set; }
}
```
Затем создайте начальную заметку, создав объект хранилища и добавив объект в это хранилище.
```csharp
// create a note for the first time, with a non-null, non-* eTag.
var note = new Note { Name = "Shopping List", Contents = "eggs", eTag = "x" };

var changes = new KeyValuePair<string, object>[]
{
    new KeyValuePair<string, object>("Note", note)
};
await NoteStore.Write(changes);
```
Впоследствии обновите заметку, сохранив значение `eTag`, прочитанное из хранилища.
```csharp
var note = NoteStore.Read<Note>("Note").Result.FirstOrDefault().Value;

if (note != null)
{
    note.Contents += ", bread";
    var changes = new KeyValuePair<string, object>[]
    {
        new KeyValuePair<string, object>("Note1", note)
    };
    await NoteStore.Write(changes);
}
```
Если заметка в хранилище была изменена перед записью ваших изменений, то при вызове `Write` возникнет исключение.

# <a name="javascripttabjsetag"></a>[JavaScript](#tab/jsetag)

Во-первых, создайте объект `myNote`.
```javascript
var myNote = {
    name: "Shopping List",
    contents: "eggs",
    eTag: "*"
}
```
Затем инициализируйте объект `changes` и добавьте к нему ваши *заметки*, после этого запишите объект в хранилище.

```javascript
// Write a note
var changes = {};
changes["Note"] = myNote;
await storage.write(changes);
```
Впоследствии обновите заметку, сохранив значение `eTag`, прочитанное из хранилища.
```javascript
// Read in a note
var note = await storage.read(["Note"]);
try {
    // Someone has updated the note. Need to update our local instance to match
    if(myNote.eTag != note.eTag){
        myNote = note;
    }

    // Add any updates to the note and write back out
    myNote.contents += ", bread";   // Add to the note
    changes["Note"] = note;
    await storage.write(changes); // Write the changes back to storage
    try {
        context.sendActivity('Successful write.');
    } catch (err) {
        context.sendActivity(`Write failed: ${err}`);
    }
}
catch (err) {
    context.sendActivity(`Unable to read the Note: ${err}`);
}
```
Если заметка в хранилище была изменена перед записью ваших изменений, то при вызове `Write` возникнет исключение.

---

Для поддержания параллелизма всегда считывайте значение свойства из хранилища, затем изменяйте прочитанное свойство, чтобы сохранить `eTag`. При считывании сведений о пользователях из хранилища ответ будет содержать свойство eTag. Если вы изменили данные и записываете обновленные данные в хранилище, ваш запрос должен включать свойство eTag, содержащее то же значение, которое было прочитано ранее. Однако при записи объекта со свойством `eTag`, имеющим значение `*`, будет разрешена перезапись любых других изменений.

<!-- If the ETag specified in your `Storage.Write()` request matches the current value in the store, the server will save the data and specify a new eTag value in the body of the response, that indicates that the data has been updated. If the ETag specified in your Storage.Write() request does not match the current value in the store, the bot responds with an error indicating an eTag conflict, to indicate that the user's data in the store has changed since you last saved or retrieved it. -->

<!-- TODO: new snippet -->

<!-- jf: I think this section can be cut entirely.

## Save a conversation reference using storage

You can use IStorage to save BotBuilder SDK objects as well as user-defined data. This code snippet uses `Storage.Write()` to save a ConversationReference object for use in sending a proactive message later.

# [C#](#tab/csharpwriteconvref)
```csharp
            var reference = context.ConversationReference;
            var userId = reference.User.Id;

            // save the ConversationReference to a global variable of this class
            conversationReference = reference;

            StoreItems storeItems = new StoreItems();
            StoreItem conversationReferenceToStore = new StoreItem();
            // set the eTag to "*" to indicate you're overwriting previous data
            conversationReferenceToStore.eTag = "*";
            conversationReferenceToStore.Add("ref", reference);
            storeItems[$"ConversationReference/{userId}"] = conversationReferenceToStore;

            await storage.Write(storeItems).ConfigureAwait(false);

            SubscribeUser(userId);

            await context.SendActivity("Thank You! We will message you shortly.");
        }

public void SubscribeUser(string userId)
{
    CreateContextForUserAsync(userId, async (ITurnContext context) =>
    {
        await context.SendActivity("You've been notified.");
        await Task.Delay(2000);
    });
                
}
```
# [JavaScript](#tab/jswriteconvref)
```javascript
const { MemoryStorage } = require('botbuilder');

const storage = new MemoryStorage();

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            const utterances = (context.activity.text || '').trim().toLowerCase()
            if (utterances === 'subscribe') {
                const reference = context.activity;
                const userId = reference.id;
                const changes = {};
                changes['reference/' + userId] = reference;
                await storage.write(changes)
                await subscribeUser(userId)
                await context.sendActivity(`Thank You! We will message you shortly.`);
               
            } else{
                await context.sendActivity("Say 'subscribe'");
            }
    
        }
    });
});
```
---

To read the saved object from storage, call `Storage.Read()`.

# [C#](#tab/csharpread)
```csharp


        private async void CreateContextForUserAsync(String userId,Func<ITurnContext, Task> onReady)
        {
            var referenceKey = $"ConversationReference/{userId}";
            
            ConversationReference localRef = null;
            StoreItems conversationReferenceFromStorage = await storage.Read(referenceKey);
            foreach (var item in conversationReferenceFromStorage)
            {
                StoreItem value = item.Value as StoreItem;
                localRef = value["ref"];
            }
            
            await bot.CreateContext(this.conversationReference, onReady);

        }
```
# [JavaScript](#tab/jsread)
```javascript
async function subscribeUser(userId) {
    setTimeout(() => {
        createContextForUser(userId, (context) => {
            context.sendActivity("You have been notified");
        })
    }, 2000);
}

async function createContextForUser(userId, callback) {
    const referenceKey = 'reference/' + userId;
    var rows = await storage.read([referenceKey])
    var reference = await rows[referenceKey]
    await callback(adapter.createContext(reference))
          
}
```
---

-->

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знаете, как напрямую считывать данные из хранилища и записывать данные в хранилища, посмотрим, как сделать это с помощью диспетчера состояний.

> [!div class="nextstepaction"]
> [Сохранение состояния с помощью свойств conversation и user](bot-builder-howto-v4-state.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Концепция: реализация хранилища с использованием пакета SDK Bot Builder](bot-builder-storage-concept.md)



---
title: Распознавание намерений и сущностей с помощью LUIS | Документы Майкрософт
description: Интеграция бота с LUIS для определения намерений пользователя и выполнения соответствующих ответных действий путем активации диалоговых окон с помощью пакета SDK Bot Framework для Node.js.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/28/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: acdc6053f7d666c2f086dca554efafc93c8af769
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225289"
---
# <a name="recognize-intents-and-entities-with-luis"></a>Распознавание намерений и сущностей с помощью LUIS 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

В этой статье используется пример бота для ведения заметок. Этот пример показывает, как реализовать естественную реакцию бота на входные данные на естественном языке с помощью распознавания речи ([LUIS][LUIS]). Бот понимает, что хочет сделать пользователь, определяя **намерение** пользователя. Это намерение определяется на основе голосовых или текстовых входных данных, или **высказываний**. Намерение сопоставляет высказывания с действиями, которые выполняет бот, например, активация диалогового окна. Боту также может потребоваться извлекать **сущности** — важные слова в высказываниях. Иногда сущности должны заполнять намерение. Для бота, который ведет заметки, сущность `Notes.Title` определяет заголовок каждой заметки.

## <a name="create-a-language-understanding-bot-with-bot-service"></a>Создание бота для распознавания речи с помощью службы Azure Bot

1. На [портале Azure](https://portal.azure.com) в колонке меню выберите **Создать ресурс**, а затем **Показать все**.

    ![Создание ресурса](../media/bot-builder-nodejs-use-luis/bot-service-creation.png)

2. В поле поиска введите **Бот веб-приложения**. 

    ![Создание ресурса](../media/bot-builder-nodejs-use-luis/bot-service-selection.png)

3. В колонке **Служба ботов** введите необходимые сведения и нажмите кнопку **Создать**. В Azure будут созданы и развернуты служба ботов и приложение LUIS. 
   * В поле **Имя приложения** укажите имя бота. При развертывании бота в облаке имя используется в качестве поддомена (например, mynotesbot.azurewebsites.net). Это имя также используется как имя приложения LUIS, связанного с ботом. Скопируйте его для последующего использования. Оно поможет найти приложение LUIS, связанное с ботом.
   * Заполните поля "Подписка", [Группа ресурсов](/azure/azure-resource-manager/resource-group-overview), "План службы приложений" и [Расположение](https://azure.microsoft.com/en-us/regions/).
   * В поле **Шаблон бота** выберите шаблон **Распознавание речи (Node.js)**.

     ![Колонка "Служба ботов"](../media/bot-builder-nodejs-use-luis/bot-service-setting-callout-template.png)

   * Установите этот флажок, чтобы подтвердить условия предоставления услуг.

4. Убедитесь, что служба Bot Service развернута.
    * Щелкните "Notifications" (Уведомления) (значок колокольчика, расположенный в верхней части портала Azure). Уведомление изменится с **Развертывание начато** на **Развертывание выполнено**.
    * После того как уведомление изменится на **Развертывание прошло успешно**, в этом уведомлении щелкните **Go to resource** (Перейти к ресурсу).

## <a name="try-the-bot"></a>Проверка работы бота

Убедитесь, что бот был развернут, проверив раздел **Уведомления**. Уведомление изменится с **Развертывание выполняется...** на **Развертывание выполнено**. Чтобы открыть колонку ресурсов бота, нажмите кнопку **Перейти к ресурсу**.

После регистрации бота выберите пункт **Test in Web Chat** (Тестировать в веб-чате), чтобы открыть панель веб-чата. В веб-чате введите "hello" (привет).

  ![Тестирование бота в веб-чате](../media/bot-builder-nodejs-use-luis/bot-service-web-chat.png)

Бот отвечает: "You have reached Greeting. You said: hello" (Вы находитесь на этапе приветствия. Вы сказали: привет). Этот ответ подтверждает, что бот получил ваше сообщение и передал его в созданное приложение LUIS по умолчанию. Это приложение LUIS по умолчанию обнаружило намерение приветствия.

## <a name="modify-the-luis-app"></a>Изменение приложения LUIS

Войдите в [https://www.luis.ai](https://www.luis.ai) с той же учетной записью, которая используется для входа Azure. Щелкните **Мои приложения**. В списке приложений найдите приложение, которое начинается с имени, указанного в поле **Имя приложения** в колонке **Служба ботов** при создании службы ботов. 

Приложение LUIS изначально понимает 4 намерения: Cancel (Отмена), Greeting (Приветствие), Help (Справка) и None (Отсутствует). <!-- picture -->

Выполнив следующие действия, вы сможете добавить намерения Note.Create, Note.ReadAloud и Note.Delete: 

1. Щелкните **Предварительно созданные домены** в левом нижнем углу страницы. Найдите домен **Note** и нажмите кнопку **Добавить домен**.

2. В этом учебнике используются не все намерения, включенные в предварительно созданный домен **Note**. На странице **Намерения** щелкните каждое из указанных имен намерений и нажмите кнопку **Удалить намерение**.
   * Note.ShowNext
   * Note.DeleteNoteItem
   * Note.Confirm
   * Note.Clear
   * Note.CheckOffItem
   * Note.AddToNote

   Ниже перечислены только те намерения, которые должны остаться в приложении LUIS: 
   * Note.ReadAloud
   * Note.Create
   * Note.Delete
   * Нет
   * Справка
   * Greeting
   * Отмена

     ![Намерения, отображаемые в приложении LUIS](../media/bot-builder-nodejs-use-luis/luis-intent-list.png)


3.  Нажмите кнопку **Обучить** в правом верхнем углу, чтобы обучить приложение.
4.  Нажмите кнопку **Опубликовать** на панели навигации вверху, чтобы открыть страницу **Публикация**. Нажмите кнопку **Опубликовать в рабочем слоте**. После успешной публикации приложение LUIS развертывается по URL-адресу, который указан в столбце **Конечная точка** и в строке, которая начинается с имени ресурса Starter_Key, на странице **Публикация приложения**. Формат URL-адреса похож на формат, используемый в следующем примере: `https://westus.api.cognitive.microsoft.com/luis/v2.0/apps/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx?subscription-key=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx&timezoneOffset=0&verbose=true&q=`. Идентификатор приложения и ключ подписки в этом URL-адресе соответствуют параметрам LuisAppId и LuisAPIKey в разделе ** Параметры службы приложений > Параметры приложения > Параметры приложения **


## <a name="modify-the-bot-code"></a>Изменение кода бота

Выберите пункт **Build** (Сборка) и щелкните ссылку **Open online code editor** (Открыть сетевой редактор кода).

   ![Открытие сетевого редактора кода](../media/bot-builder-nodejs-use-luis/bot-service-build.png)

В редакторе кода откройте `app.js`. Он содержит следующий код:

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send('You reached the default message handler. You said \'%s\'.', session.message.text);
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// Add a dialog for each intent that the LUIS app recognizes.
// See https://docs.microsoft.com/en-us/bot-framework/nodejs/bot-builder-nodejs-recognize-intent-luis 
bot.dialog('GreetingDialog',
    (session) => {
        session.send('You reached the Greeting intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Greeting'
})

bot.dialog('HelpDialog',
    (session) => {
        session.send('You reached the Help intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Help'
})

bot.dialog('CancelDialog',
    (session) => {
        session.send('You reached the Cancel intent. You said \'%s\'.', session.message.text);
        session.endDialog();
    }
).triggerAction({
    matches: 'Cancel'
})

```


> [!TIP] 
> Пример кода, описанный в этой статье, можно найти в разделе [Пример бота для работы с заметками][NotesSample].



## <a name="edit-the-default-message-handler"></a>Изменение обработчика сообщений по умолчанию
Этот бот содержит обработчик сообщений по умолчанию. Измените его следующим образом: 
```javascript
// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});
```

## <a name="handle-the-notecreate-intent"></a>Обработка намерения Note.Create

Скопируйте следующий код и вставьте его в конец файла app.js:

```javascript
// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});
```

Любые сущности в высказывании передаются в диалоговое окно с помощью параметра `args`. На первом шаге [каскада][waterfall] вызывается метод [EntityRecognizer.findEntity][EntityRecognizer_findEntity], который получает заголовок заметки из любых сущностей `Note.Title` в ответе LUIS. Если приложение LUIS не обнаружило сущность `Note.Title`, бот запрашивает имя сущности у пользователя. На втором шаге каскада у пользователя запрашивается текст, включаемый в заметку. После того как бот получил текст заметки, на третьем шаге каскада вызывается метод [session.userData][session_userData], который сохраняет заметку в объекте `notes`, используя заголовок в качестве ключа. Дополнительные сведения о `session.UserData` см. в разделе [Управление данными состояния](./bot-builder-nodejs-state.md). 



## <a name="handle-the-notedelete-intent"></a>Обработка намерения Note.Delete
Как и для намерения `Note.Create`, бот проверяет наличие заголовка в параметре `args`. Если заголовок не обнаружен, бот запрашивает его у пользователя. Заголовок используется для поиска заметки, которую необходимо удалить из `session.userData.notes`. 



Скопируйте следующий код и вставьте его в конец файла app.js:
```javascript
// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});
```

Код, который обрабатывает `Note.Delete`, использует функцию `noteCount`, чтобы определить, содержатся ли заметки в объекте `notes`. 

Вставьте вспомогательную функцию `noteCount` в конце `app.js`.

[!code-js[Add a helper function that returns the number of notes (JavaScript)](../includes/code/node-basicNote.js#CountNotesHelper)]

## <a name="handle-the-notereadaloud-intent"></a>Обработка намерения Note.ReadAloud

Скопируйте следующий код и вставьте его в `app.js` после обработчика `Note.Delete`:

```javascript
// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});

```

Объект `session.userData.notes` передается в качестве третьего аргумента `builder.Prompts.choice`, поэтому в строке запроса отображается список заметок пользователя.

Теперь, когда вы добавили обработчики для новых намерений, полный код `app.js` выглядит так:

```javascript
var restify = require('restify');
var builder = require('botbuilder');
var botbuilder_azure = require("botbuilder-azure");

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log('%s listening to %s', server.name, server.url); 
});
  
// Create chat connector for communicating with the Bot Framework Service
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword,
    openIdMetadata: process.env.BotOpenIdMetadata 
});

// Listen for messages from users 
server.post('/api/messages', connector.listen());

/*----------------------------------------------------------------------------------------
* Bot Storage: This is a great spot to register the private state storage for your bot. 
* We provide adapters for Azure Table, CosmosDb, SQL Azure, or you can implement your own!
* For samples and documentation, see: https://github.com/Microsoft/BotBuilder-Azure
* ---------------------------------------------------------------------------------------- */

var tableName = 'botdata';
var azureTableClient = new botbuilder_azure.AzureTableClient(tableName, process.env['AzureWebJobsStorage']);
var tableStorage = new botbuilder_azure.AzureBotStorage({ gzipData: false }, azureTableClient);

// Create your bot with a function to receive messages from the user.
// This default message handler is invoked if the user's utterance doesn't
// match any intents handled by other dialogs.
var bot = new builder.UniversalBot(connector, function (session, args) {
    session.send("Hi... I'm the note bot sample. I can create new notes, read saved notes to you and delete notes.");

   // If the object for storing notes in session.userData doesn't exist yet, initialize it
   if (!session.userData.notes) {
       session.userData.notes = {};
       console.log("initializing userData.notes in default message handler");
   }
});

bot.set('storage', tableStorage);

// Make sure you add code to validate these fields
var luisAppId = process.env.LuisAppId;
var luisAPIKey = process.env.LuisAPIKey;
var luisAPIHostName = process.env.LuisAPIHostName || 'westus.api.cognitive.microsoft.com';

const LuisModelUrl = 'https://' + luisAPIHostName + '/luis/v2.0/apps/' + luisAppId + '?subscription-key=' + luisAPIKey;

// Create a recognizer that gets intents from LUIS, and add it to the bot
var recognizer = new builder.LuisRecognizer(LuisModelUrl);
bot.recognizer(recognizer);

// CreateNote dialog
bot.dialog('CreateNote', [
    function (session, args, next) {
        // Resolve and store any Note.Title entity passed from LUIS.
        var intent = args.intent;
        var title = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');

        var note = session.dialogData.note = {
          title: title ? title.entity : null,
        };
        
        // Prompt for title
        if (!note.title) {
            builder.Prompts.text(session, 'What would you like to call your note?');
        } else {
            next();
        }
    },
    function (session, results, next) {
        var note = session.dialogData.note;
        if (results.response) {
            note.title = results.response;
        }

        // Prompt for the text of the note
        if (!note.text) {
            builder.Prompts.text(session, 'What would you like to say in your note?');
        } else {
            next();
        }
    },
    function (session, results) {
        var note = session.dialogData.note;
        if (results.response) {
            note.text = results.response;
        }
        
        // If the object for storing notes in session.userData doesn't exist yet, initialize it
        if (!session.userData.notes) {
            session.userData.notes = {};
            console.log("initializing session.userData.notes in CreateNote dialog");
        }
        // Save notes in the notes object
        session.userData.notes[note.title] = note;

        // Send confirmation to user
        session.endDialog('Creating note named "%s" with text "%s"',
            note.title, note.text);
    }
]).triggerAction({ 
    matches: 'Note.Create',
    confirmPrompt: "This will cancel the creation of the note you started. Are you sure?" 
}).cancelAction('cancelCreateNote', "Note canceled.", {
    matches: /^(cancel|nevermind)/i,
    confirmPrompt: "Are you sure?"
});

// Delete note dialog
bot.dialog('DeleteNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify that the title is in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to delete?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to delete.");
        }
    },
    function (session, results) {
        delete session.userData.notes[results.response.entity];        
        session.endDialog("Deleted the '%s' note.", results.response.entity);
    }
]).triggerAction({
    matches: 'Note.Delete'
}).cancelAction('cancelDeleteNote', "Ok - canceled note deletion.", {
    matches: /^(cancel|nevermind)/i
});


// Read note dialog
bot.dialog('ReadNote', [
    function (session, args, next) {
        if (noteCount(session.userData.notes) > 0) {
           
            // Resolve and store any Note.Title entity passed from LUIS.
            var title;
            var intent = args.intent;
            var entity = builder.EntityRecognizer.findEntity(intent.entities, 'Note.Title');
            if (entity) {
                // Verify it's in our set of notes.
                title = builder.EntityRecognizer.findBestMatch(session.userData.notes, entity.entity);
            }
            
            // Prompt for note name
            if (!title) {
                builder.Prompts.choice(session, 'Which note would you like to read?', session.userData.notes);
            } else {
                next({ response: title });
            }
        } else {
            session.endDialog("No notes to read.");
        }
    },
    function (session, results) {        
        session.endDialog("Here's the '%s' note: '%s'.", results.response.entity, session.userData.notes[results.response.entity].text);
    }
]).triggerAction({
    matches: 'Note.ReadAloud'
}).cancelAction('cancelReadNote', "Ok.", {
    matches: /^(cancel|nevermind)/i
});


// Helper function to count the number of notes stored in session.userData.notes
function noteCount(notes) {

    var i = 0;
    for (var name in notes) {
        i++;
    }
    return i;
}
```

## <a name="test-the-bot"></a>Тестирование бота

На портале Azure щелкните **Test in Web Chat** (Тестировать в веб-чате), чтобы протестировать бот. Попробуйте ввести сообщения, такие как "Create a note", "read my notes" и "delete notes", чтобы вызвать добавленные намерения.
   ![Тестирование бота, выполняющего ведение заметок, в веб-чате](../media/bot-builder-nodejs-use-luis/bot-service-test-notebot.png)

> [!TIP]
> Если ваш бот не всегда распознает правильное намерение или сущности, необходимо улучшить производительность приложения LUIS, предоставив ему больше примеров высказываний для обучения. Вы можете повторно обучить приложение LUIS, не изменяя код бота. См. разделы [Добавление примеров высказываний](/azure/cognitive-services/LUIS/add-example-utterances) и [Обучение и тестирование приложений LUIS](/azure/cognitive-services/LUIS/train-test).


## <a name="next-steps"></a>Дополнительная информация

При работе с ботом вы увидите, что распознаватель может прерывать работу текущего активного диалогового окна. Использование и обработка прерываний — гибкий механизм, учитывающий реальные действия пользователей. Дополнительные сведения о различных действиях, которые можно связать с распознаваемым намерением.

> [!div class="nextstepaction"]
> [Обработка действий пользователя](bot-builder-nodejs-dialog-actions.md)


[LUIS]: https://www.luis.ai/

[intentDialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html

[intentDialog_matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.intentdialog.html#matches 

[NotesSample]: https://github.com/Microsoft/BotFramework-Samples/tree/master/docs-samples/Node/basics-naturalLanguage

[triggerAction]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[confirmPrompt]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#confirmprompt

[waterfall]: bot-builder-nodejs-dialog-manage-conversation-flow.md#manage-conversation-flow-with-a-waterfall

[session_userData]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata

[EntityRecognizer_findEntity]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.entityrecognizer.html#findentity

[matches]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[Dialog]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html

[IntentRecognizerSetOptions]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISConcepts]: https://docs.botframework.com/en-us/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample

[UniversalBot]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html

---
title: Краткий справочник по миграции из версии 3 в версию 4 для JavaScript | Документация Майкрософт
description: Описание основных различий между пакетами SDK Bot Framework для JavaScript версий 3 и 4.
keywords: JavaScript, bot migration, dialogs, v3 bot
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e0ad87c9b2896767d7053322b510da182a44ec69
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299018"
---
# <a name="javascript-migration-quick-reference"></a>Краткий справочник по миграции для JavaScript

В пакете SDK Bot Builder для Javascript версии 4 реализовано несколько важных изменений, связанных с процессом создания ботов. В этом руководстве описаны различия между способами выполнения задач в пакетах SDK версий 3 и 4.

- Способ передачи данных между каналами и ботом изменился.
    В версии 3 вы использовали объекты _соединителя_ и _сеанса_.
    В версии 4 они заменены объектами _адаптера_ и _контекста шага_.

- Кроме того, разграничены экземпляры диалогов и бота.
    В версии 3 диалоги регистрировались непосредственно в конструкторе бота.
    В версии 4 диалоги передаются в экземпляры бота как аргументы, делая процесс создания более гибким.

- Кроме того, в версии 4 реализован класс `ActivityHandler` для автоматизации обработки разных типов действий, таких как _сообщение_, _обновление беседы_ и _событие_.

В результате изменен синтаксис для разработки ботов на Javascript. В частности это касается создания объектов бота, определения диалогов и создания логики обработки событий.

Ниже приведено сравнение конструкций в пакетах SDK Bot Framework для JavaScript версий 3 и 4.

## <a name="to-listen-for-incoming-requests"></a>Прослушивание входящих запросов

### <a name="v3"></a>Версия 3

```javascript
var connector = new builder.ChatConnector({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

server.post('/api/messages', connector.listen());
```

### <a name="v4"></a>версия 4

```javascript
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        await bot.run(context);
    });
});
```

## <a name="to-create-a-bot-instance"></a>Создание экземпляра бота

### <a name="v3"></a>Версия 3

```javascript
var bot = new builder.UniversalBot(connector, [ ...DIALOG_STEPS ]);
```

### <a name="v4"></a>версия 4

```javascript
// Define the bot class as extending ActivityHandler.
const { ActivityHandler } = require('botbuilder');

class MyBot extends ActivityHandler {
    // ...
}

// Instantiate a new bot instance.
const bot = new MyBot(conversationState, dialog);
```

## <a name="to-send-a-message-to-a-user"></a>Отправка сообщения пользователю

### <a name="v3"></a>Версия 3

```javascript
session.send('Hello and welcome to the help desk bot.');
```

### <a name="v4"></a>версия 4

```javascript
await context.sendActivity('Hello and welcome to the help desk bot.');
```

## <a name="to-define-a-default-dialog"></a>Определение каталога по умолчанию

### <a name="v3"></a>Версия 3

```javascript
var bot = new builder.UniversalBot(connector, [
    // Add default dialog waterfall steps...
]);
```

### <a name="v4"></a>версия 4

```javascript
// In the main dialog class, define a run method.
async run(turnContext, accessor) {
    const dialogSet = new DialogSet(accessor);
    dialogSet.add(this);

    const dialogContext = await dialogSet.createContext(turnContext);
    const results = await dialogContext.continueDialog();
    if (results.status === DialogTurnStatus.empty) {
        await dialogContext.beginDialog(this.id);
    }
}

// Pass conversation state management and a main dialog objects to the bot (in index.js).
const bot = new MyBot(conversationState, dialog);

// Inside the bot's constructor, add the dialog as a member property and define a DialogState property accessor.
this.dialog = dialog;
this.dialogState = this.conversationState.createProperty('DialogState');

// Inside the bot's message handler, invoke the dialog's run method, passing in the turn context and DialogState accessor.
this.onMessage(async (context, next) => {
    // Run the Dialog with the new message Activity.
    await this.dialog.run(context, this.dialogState);
    await next();
});
```

## <a name="to-start-a-child-dialog"></a>Запуск дочернего диалога

Родительский диалог возобновляется после завершения дочернего диалога.

### <a name="v3"></a>Версия 3

```javascript
session.beginDialog(DIALOG_ID);
```

### <a name="v4"></a>версия 4

```javascript
await context.beginDialog(DIALOG_ID);
```

## <a name="to-replace-a-dialog"></a>Замена диалога

Текущий диалог в стеке заменяется новым диалогом.

### <a name="v3"></a>Версия 3

```javascript
session.replaceDialog(DIALOG_ID);
```

### <a name="v4"></a>версия 4

```javascript
await context.replaceDialog(DIALOG_ID);
```

## <a name="to-end-a-dialog"></a>Завершение диалога

### <a name="v3"></a>Версия 3

```javascript
session.endDialog();
```

### <a name="v4"></a>версия 4

Вы можете включить необязательное возвращаемое значение.

```javascript
await context.endDialog(returnValue);
```

## <a name="to-register-a-dialog-with-a-bot-instance"></a>Регистрация диалога с использованием экземпляра бота

### <a name="v3"></a>Версия 3

```javascript
// Create the bot.
var bot = new builder.UniversalBot(connector, [
    // ...
]};

// Add the dialog to the bot.
bot.dialog('myDialog', require('./mydialog'));
```

### <a name="v4"></a>версия 4

```javascript
// In the main dialog class constructor.
this.addDialog(new MyDialog(DIALOG_ID));

// In the app entry point file (index.js)
const { MainDialog } = require('./dialogs/main');

// Create the base dialog and bot, passing the main dialog as an argument.
const dialog = new MainDialog();
const reservationBot = new ReservationBot(conversationState, dialog);
```

## <a name="to-prompt-a-user-for-input"></a>Запрос пользователю на ввод данных

### <a name="v3"></a>Версия 3

```javascript
var builder = require('botbuilder');
builder.Prompts.text(session, 'Please enter your destination');
```

### <a name="v4"></a>версия 4

```javascript
const { TextPrompt } = require('botbuilder-dialogs');

// In the dialog's constructor, register the prompt.
this.addDialog(new TextPrompt('initialPrompt'));

// In the dialog step, invoke the prompt.
return await stepContext.prompt(
    'initialPrompt', {
        prompt: 'Please enter your destination'
    }
);
```

## <a name="to-retrieve-the-users-response-to-a-prompt"></a>Получение ответа пользователя на запрос

### <a name="v3"></a>Версия 3

```javascript
function (session, result) {
    var destination = result.response;
    // ...
}
```

### <a name="v4"></a>версия 4

```javascript
// In the dialog step after the prompt step, retrieve the result from the waterfall step context.
async nextStep(stepContext) {
    const destination = stepContext.result;
    // ...
}
```

## <a name="to-save-information-to-dialog-state"></a>Сохранение информации в состоянии диалога

### <a name="v3"></a>Версия 3

```javascript
session.dialogData.destination = results.response;
```

### <a name="v4"></a>версия 4

```javascript
// In a dialog, set the value in the waterfall step context.
stepContext.values.destination = destination;
```

## <a name="to-write-changes-in-state-to-the-persistance-layer"></a>Запись изменений в состоянии на уровне сохраняемости

### <a name="v3"></a>Версия 3

```javascript
// State data is saved by default at the end of the turn.
// Do this to save it manually.
session.save();
```

### <a name="v4"></a>версия 4

```javascript
// State changes are not saved automatically at the end of the turn.
await this.conversationState.saveChanges(context, false);
await this.userState.saveChanges(context, false);
```

## <a name="to-create-and-register-state-storage"></a>Создание и регистрация хранилища состояний

### <a name="v3"></a>Версия 3

```javascript
var builder = require('botbuilder');

// Create conversation state with in-memory storage provider.
var inMemoryStorage = new builder.MemoryBotStorage();

// Create the base dialog and bot
var bot = new builder.UniversalBot(connector, [ /*...*/ ]).set('storage', inMemoryStorage);
```

### <a name="v4"></a>версия 4

```javascript
const { MemoryStorage } = require('botbuilder');

// Create the memory layer object.
const memoryStorage = new MemoryStorage();

// Create the conversation state management object, using the storage provider.
const conversationState = new ConversationState(memoryStorage);

// Create the base dialog and bot.
const dialog = new MainDialog();
const reservationBot = new ReservationBot(conversationState, dialog);
```

## <a name="to-catch-an-error-thrown-from-a-dialog"></a>Перехват ошибки, связанной с диалогом

### <a name="v3"></a>Версия 3

```javascript
// Set up the error handler.
session.on('error', function (err) {
    session.send('Failed with message:', err.message);
    session.endDialog();
});

// Throw an error.
session.error('An error has occurred.');
```

### <a name="v4"></a>версия 4

```javascript
// Set up the error handler, using the adapter.
adapter.onTurnError = async (context, error) => {
    const errorMsg = error.message

    // Clear out conversation state to avoid keeping the conversation in a corrupted bot state.
    await conversationState.delete(context);

    // Send a message to the user.
    await context.sendActivity(errorMsg);
};

// Throw an error.
async () => {
    throw new Error('An error has occurred.');
}
```

## <a name="to-register-activity-handlers"></a>Регистрация обработчиков действий

### <a name="v3"></a>Версия 3

```javascript
bot.on('conversationUpdate', function (message) {
    // ...
}

bot.on('contactRelationUpdate', function (message) {
    // ...
}
```

### <a name="v4"></a>версия 4

```javascript
// In the bot's constructor, add the handlers.
this.onMessage(async (context, next) => {
    // ...
}
  
this.onDialog(async (context, next) => {
    // ...
}

this.onMembersAdded(async (context, next) => {
    // ...
}
```

## <a name="to-send-attachments"></a>Отправка вложений

### <a name="v3"></a>Версия 3

```javascript
var message = new builder.Message()
    .attachments(hotelHeroCards
    .attachmentLayout(builder.AttachmentLayout.carousel));
```

### <a name="v4"></a>версия 4

```javascript
await context.sendActivity({
    attachments: hotelHeroCards,
    attachmentLayout: AttachmentLayoutTypes.Carousel
});
```

## <a name="to-use-natural-language-recognition-luis"></a>Использование распознавания естественного языка (LUIS)

### <a name="v3"></a>Версия 3

```javascript
// The LUIS recognizer was part of the 'botbuilder' library
var builder = require('botbuilder');

var recognizer = new builder.LuisRecognizer(LUIS_MODEL_URL);
bot.recognizer(recognizer);
```

### <a name="v4"></a>версия 4

```javascript
// The LUIS recognizer is now part of the 'botbuilder-ai' library
const { LuisRecognizer } = require('botbuilder-ai');

const luisApp = { applicationId: LuisAppId, endpointKey: LuisAPIKey, endpoint: `https://${ LuisAPIHostName }` };
const recognizer = new LuisRecognizer(luisApp);

const recognizerResult = await recognizer.recognize(context);
const intent = LuisRecognizer.topIntent(recognizerResult);
```

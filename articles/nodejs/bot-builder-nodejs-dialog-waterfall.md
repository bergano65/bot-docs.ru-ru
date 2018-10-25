---
title: Определение шагов диалога с помощью каскадов | Документы Майкрософт
description: Узнайте, как использовать каскады для определения шагов диалога с помощью пакета SDK для построителя ботов для Node.js.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2fa857e57d5be4751476874b8c193c7053a1bf39
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000281"
---
# <a name="define-conversation-steps-with-waterfalls"></a>Определение шагов диалога с помощью каскадов

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Диалог — это последовательность сообщений, передаваемых между пользователем и ботом. Когда цель бота — провести пользователя через последовательность действий, вы можете использовать каскад, чтобы определить шаги диалога.

Каскад является конкретной реализацией [диалога](bot-builder-nodejs-dialog-overview.md), которая обычно используется для сбора информации от пользователя или предоставления пользователю инструкций по выполнению рядя задач. Задачи реализуются как массив функций, где результаты первой функции передаются в качестве входных данных в следующую функцию и так далее. Каждая функция обычно представляет один шаг в общем процессе. На каждом шаге бот запрашивает у пользователя входные данные, ожидает ответа, а затем передает результат в следующий шаг.

Эта статья поможет вам понять, как работает каскад и как можно использовать его для [управления потоком диалога](bot-builder-nodejs-dialog-manage-conversation.md).

## <a name="conversation-steps"></a>Шаги диалога

Диалог обычно состоит из обмена несколькими запросами и ответами между пользователем и ботом. Каждый обмен запросом и ответом представляет собой шаг диалога. С помощью каскада вы можете создать диалог всего из двух шагов.

Например, рассмотрим следующий диалог `greetings`. На первом шаге каскада бот запрашивает имя пользователя, а на втором шаге бот на основе ответа приветствует пользователя по имени.

```javascript
bot.dialog('greetings', [
    // Step 1
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    // Step 2
    function (session, results) {
        session.endDialog(`Hello ${results.response}!`);
    }
]);
```

Это возможно благодаря запросам. Пакет SDK для построителя ботов для Node.js предоставляет несколько разных типов встроенных [запросов](bot-builder-nodejs-dialog-prompt.md), с помощью которых можно узнавать у пользователя различную информацию.

В следующем примере кода показан диалог, в котором с помощью запросов собирается различная информация о пользователе через каскад из четырех шагов.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This is a dinner reservation bot that uses a waterfall technique to prompt users for input.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.send("Welcome to the dinner reservation.");
        builder.Prompts.time(session, "Please provide a reservation date and time (e.g.: June 6th at 5pm)");
    },
    function (session, results) {
        session.dialogData.reservationDate = builder.EntityRecognizer.resolveTime([results.response]);
        builder.Prompts.text(session, "How many people are in your party?");
    },
    function (session, results) {
        session.dialogData.partySize = results.response;
        builder.Prompts.text(session, "Whose name will this reservation be under?");
    },
    function (session, results) {
        session.dialogData.reservationName = results.response;

        // Process request and display reservation details
        session.send(`Reservation confirmed. Reservation details: <br/>Date/Time: ${session.dialogData.reservationDate} <br/>Party size: ${session.dialogData.partySize} <br/>Reservation name: ${session.dialogData.reservationName}`);
        session.endDialog();
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 
```

В этом примере диалог по умолчанию имеет четыре функции, каждая из которых представляет шаг каскада. На каждом шаге у пользователя запрашиваются сведения, а результаты отправляются на следующий шаг для обработки. Этот процесс продолжается, пока не будет выполнен последний шаг. Бронирование будет подтверждено, диалог завершится.

На следующем снимке экрана показаны результаты выполнения этого бота в [эмуляторе Bot Framework](~/bot-service-debug-emulator.md).

![Управление потоком беседы с помощью каскада](~/media/bot-builder-nodejs-dialog-manage-conversation/waterfall-results.png)

## <a name="create-a-conversation-with-multiple-waterfalls"></a>Создание диалога с помощью нескольких каскадов

Вы можете использовать в диалоге несколько каскадов, чтобы определить необходимую структуру диалога. Например, вы можете использовать один каскад для управления потоком диалога и дополнительные каскады для сбора сведений от пользователя. Каждый каскад заключен в диалог и запускается с помощью вызова функции `session.beginDialog`.

В следующем примере кода показано, как использовать несколько каскадов в диалоге. Каскад в диалоге `greetings` управляет потоком диалога, а каскад в диалоге `askName` собирает информацию от пользователя.

```javascript
bot.dialog('greetings', [
    function (session) {
        session.beginDialog('askName');
    },
    function (session, results) {
        session.endDialog('Hello %s!', results.response);
    }
]);
bot.dialog('askName', [
    function (session) {
        builder.Prompts.text(session, 'Hi! What is your name?');
    },
    function (session, results) {
        session.endDialogWithResult(results);
    }
]);
```

## <a name="advance-the-waterfall"></a>Прогресс каскада

Каскад переходит от шага к шагу в последовательности, в которой функции определены в массиве. Первая функция в каскаде может получить аргументы, которые передаются в диалог. Любая функция в каскаде может использовать функцию `next`, чтобы перейти к следующему шагу без запроса данных от пользователя.

В следующем примере кода показано, как использовать функцию `next` в диалоге, чтобы провести пользователя через процесс сбора сведения для профиля. На каждом шаге каскада бот запрашивает у пользователя информацию (при необходимости), и если пользователь дает ответ, этот ответ обрабатывается на следующем шаге каскада. Последний шаг каскада в диалоге `ensureProfile` завершает этот диалог и возвращает заполненный профиль вызывающему диалогу, который отправляет пользователю индивидуальное приветствие.

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();

// This bot ensures user's profile is up to date.
var bot = new builder.UniversalBot(connector, [
    function (session) {
        session.beginDialog('ensureProfile', session.userData.profile);
    },
    function (session, results) {
        session.userData.profile = results.response; // Save user profile.
        session.send(`Hello ${session.userData.profile.name}! I love ${session.userData.profile.company}!`);
    }
]).set('storage', inMemoryStorage); // Register in-memory storage 

bot.dialog('ensureProfile', [
    function (session, args, next) {
        session.dialogData.profile = args || {}; // Set the profile or create the object.
        if (!session.dialogData.profile.name) {
            builder.Prompts.text(session, "What's your name?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results, next) {
        if (results.response) {
            // Save user's name if we asked for it.
            session.dialogData.profile.name = results.response;
        }
        if (!session.dialogData.profile.company) {
            builder.Prompts.text(session, "What company do you work for?");
        } else {
            next(); // Skip if we already have this info.
        }
    },
    function (session, results) {
        if (results.response) {
            // Save company name if we asked for it.
            session.dialogData.profile.company = results.response;
        }
        session.endDialogWithResult({ response: session.dialogData.profile });
    }
]);
```

> [!NOTE]
> В этом примере используется два разных контейнера для хранения данных: `dialogData` и `userData`. Если бот распределяется между несколькими вычислительными узлами, каждый шаг каскада может обрабатываться разными узлами. Дополнительные сведения о хранении данных бота см. в разделе [Управление данными состояния](bot-builder-nodejs-state.md).

## <a name="end-a-waterfall"></a>Завершение каскада

Необходимо явным образом завершить диалог, который создается с помощью каскада, в противном случае бот будет повторять каскад бесконечно. Вы можете завершить каскад с помощью одного из следующих методов:

* `session.endDialog`. Используйте этот метод для завершения каскада, если нет данных, которые можно вернуть вызывающему диалогу.

* `session.endDialogWithResult`. Используйте этот метод для завершения каскада, если есть данные, которые можно вернуть вызывающему диалогу. Возвращаемый аргумент `response` может быть объектом JSON или любым примитивным типом данных JavaScript. Например: 
  ```javascript
  session.endDialogWithResult({
    response: { name: session.dialogData.name, company: session.dialogData.company }
  });
  ```

* `session.endConversation`. Используйте этот метод для завершения каскада, если конец каскада является концом диалога.

В качестве альтернативы одному из этих трех методов для завершения каскада вы можете подключить к диалогу триггер `endConversationAction`. Например: 

```javascript
bot.dialog('dinnerOrder', [
    //...waterfall steps...,
    // Last step
    function(session, results){
        if(results.response){
            session.dialogData.room = results.response;
            var msg = `Thank you. Your order will be delivered to room #${session.dialogData.room}`;
            session.endConversation(msg);
        }
    }
])
.endConversationAction(
    "endOrderDinner", "Ok. Goodbye.",
    {
        matches: /^cancel$|^goodbye$/i,
        confirmPrompt: "This will cancel your order. Are you sure?"
    }
);
```

## <a name="next-steps"></a>Дополнительная информация

Используя каскад, вы можете собирать сведения от пользователя с помощью *запросов*. Давайте рассмотрим подробнее, как можно запросить у пользователя данные.

> [!div class="nextstepaction"]
> [Запрос на ввод данных пользователем](bot-builder-nodejs-dialog-prompt.md)

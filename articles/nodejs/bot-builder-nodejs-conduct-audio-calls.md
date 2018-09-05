---
title: Голосовые вызовы | Документация Майкрософт
description: Узнайте, как выполнять голосовые вызовы из бота через Skype с помощью Node.js
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 35af3a339a20fe0e7e70d001db035aec2647aa35
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905323"
---
# <a name="support-audio-calls-with-skype"></a>Поддержка голосовых вызовов через Skype

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Skype поддерживает функцию Calling Bots.  Когда эта функция включена, пользователи могут выполнять голосовые вызовы бота и взаимодействовать с ним с помощью функции интерактивного речевого ответа.  Пакет SDK Bot Builder для Node.js содержит специальный [пакет SDK для вызовов][calling_sdk], с помощью которого разработчики могут добавлять в свои боты функции голосовых вызовов.   

Этот пакет SDK очень похож на [пакет SDK для чатов][chat_sdk]. Они используют аналогичные классы и конструкции, позволяя отправить сообщение с помощью SDK для чатов вызываемому пользователю.  Хотя эти два пакета SDK специально разработаны для параллельного использования, они имеют ряд важных различий. Старайтесь не путать аналогичные классы из разных библиотек.  

## <a name="create-a-calling-bot"></a>Создание бота с поддержкой голосовых вызовов
В следующем примере кода показано, как создать простейший бот с поддержкой голосовых вызовов, который очень похож на обычный чат-бот. 

```javascript
var restify = require('restify');
var calling = require('botbuilder-calling');

// Setup Restify Server
var server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
   console.log(`${server.name} listening to ${server.url}`); 
});

// Create calling bot
var connector = new calling.CallConnector({
    callbackUrl: 'https://<your host>/api/calls',
    appId: '<your bots app id>',
    appPassword: '<your bots app password>'
});
var bot = new calling.UniversalCallBot(connector);
server.post('/api/calls', connector.listen());

// Add root dialog
bot.dialog('/', function (session) {
    session.send('Watson... come here!');
});
```

> [!NOTE]
> Сведения о том, как найти значения **AppID** и **AppPassword** для бота, см. в разделе [MicrosoftAppID и MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

Эмулятор сейчас не поддерживает тестирование ботов с голосовыми вызовами. Чтобы протестировать такого бота, нужно выполнить почти всю процедуру публикации бота.  Также вам потребуется клиент Skype для взаимодействия с ботом. 

### <a name="enable-the-skype-channel"></a>Включение канала Скайп
[Зарегистрируйте бот](../bot-service-quickstart-registration.md) и включите для него канал Skype. При регистрации бота нужно указать конечную точку для обмена сообщениями. Мы рекомендуем связать этот бот с чат-ботом, чтобы указать в этом поле конечную точку этого чат-бота.  Если же вы регистрируете только бот для голосовых вызовов, скопируйте в это поле его конечную точку.  

Чтобы включить функцию голосового вызова, перейдите в канал Skype, созданный для вашего бота, и включите функцию голосового вызова. Станет доступным поле, в которое нужно скопировать адрес конечной точки. В качестве узла для этой конечной точки укажите https-ссылку ngrok.

Регистрируемому боту присваиваются идентификатор и пароль, которые следует указать в параметрах соединителя для тестового бота. Также скопируйте полный адрес ссылки для голосовых вызовов и вставьте его в параметр callbackUrl.

### <a name="add-bot-to-contacts"></a>Добавление бота в контакты
На странице регистрации бота на портале разработчика вы увидите кнопку **Добавить в Skype** рядом с каналом Skype, созданным для бота. Нажмите эту кнопку, чтобы добавить бота в список контактов Skype.  После этого вы (и все пользователи, которым вы предоставите эту ссылку) сможете взаимодействовать с ботом с помощью Skype.

### <a name="test-your-bot"></a>Тестирование бота
Чтобы проверить работу бота, используйте клиент Skype. Обратите внимание, что при выборе записи в списке контактов бота теперь подсвечивается значок голосового вызова (чтобы увидеть это, сначала найдите бота в списке).  Если вы добавили возможность вызова к уже существующему боту, активация этого значка может занять нескольких минут.  

Нажатие кнопки вызова инициирует голосовой вызов. Вы должны услышать его ответ "Ватсон, подойдите сюда!" После этого голосовой вызов прерывается.

## <a name="calling-basics"></a>Основные сведения о голосовых вызовах
Классы [UniversalCallBot](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.universalcallbot) и [CallConnector](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callconnector) позволяют создать бот с поддержкой голосовых вызовов точно так же, как обычный чат-бот. Вы сможете создать для бота диалоги, идентичные [диалогам чата](bot-builder-nodejs-manage-conversation-flow.md). Также в бот можно добавить [каскадные диаграммы](bot-builder-nodejs-prompts.md). Класс [CallSession](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession) используется как объект сеанса и содержит дополнительные методы [answer()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#answer), [hangup()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#hangup) и [reject()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#reject) для управления голосовым вызовом. Но, скорее всего, эти методы управления вызовом вам не понадобятся, так как CallSession реализует логику автоматического управления голосовым вызовом. Сеанс автоматически отвечает на вызов, когда выполняются такие действия, как отправка сообщения или обращение к встроенной строке приглашения. Он также автоматически разрывает соединение, если вы вызовете метод [endConversation()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#endconversation) или прекратите задавать вопросы собеседнику (то есть при отсутствии обращений к встроенной строке приглашения).

Еще одно различие между ботами для голосовых вызовов и чатов заключается в том, что чат-бот обычно оперирует отправкой пользователю сообщений, карт и клавиатур, а бот для голосовых вызовов использует действия и результаты. Боты для голосовых вызовов в Skype должны создавать [рабочие процессы](http://docs.botframework.com/en-us/node/builder/calling-reference/interfaces/_botbuilder_d_.iworkflow), состоящие из одного или нескольких [действий](http://docs.botframework.com/en-us/node/builder/calling-reference/interfaces/_botbuilder_d_.iaction).  Но и об этом вам можно не беспокоиться, так как пакет SDK Bot Builder с поддержкой голосовых вызовов сам управляет большинством этих функций. Метод [CallSession.send()](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.callsession#send) позволяет передавать действия или строки, которые он преобразует в [PlayPromptActions](http://docs.botframework.com/en-us/node/builder/calling-reference/classes/_botbuilder_d_.playpromptaction).  Сеанс содержит логику автоматического объединения нескольких действий в единый рабочий процесс, который передается в службу голосовых вызовов. Это означает, что вы можете спокойно вызывать метод send() несколько раз подряд.  Для сбора входных данных от пользователя лучше всего использовать встроенные [строки приглашения](bot-builder-nodejs-prompts.md) пакета SDK, так как они обрабатывают любые сценарии.  

[calling_sdk]: http://docs.botframework.com/en-us/node/builder/calling-reference/modules/_botbuilder_d_
[chat_sdk]: http://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_

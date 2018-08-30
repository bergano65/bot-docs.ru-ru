---
title: Управление данными состояния | Документы Майкрософт
description: Узнайте, как сохранять и извлекать данные о состоянии с помощью пакета SDK построителя ботов для Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cdd35bc5b487b5bf0d49006cf168f2541e17a057
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904376"
---
# <a name="manage-state-data"></a>Управление данными состояния

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>Хранилище данных в памяти

Хранилище данных в памяти предназначено только для тестирования. Это временное и нестабильное хранилище. Данные очищаются при каждом перезапуске бота. Чтобы использовать хранилище в памяти для тестирования, необходимо сделать две вещи. Сначала создайте новый экземпляр хранилища в памяти:

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
```

Затем задайте его для бота при создании **UniversalBot**:

```javascript
var inMemoryStorage = new builder.MemoryBotStorage();
var bot = new builder.UniversalBot(connector, [..waterfall steps..])
                    .set('storage', inMemoryStorage); // Register in-memory storage 
```

Этот метод можно использовать для задания пользовательского хранилища данных или использования любого из *расширений Azure*.

## <a name="manage-custom-data-storage"></a>Управление пользовательским хранилищем данных

Для повышения производительности и (или) безопасности в рабочей среде можно реализовать собственное хранилище данных или выбрать любой из следующих вариантов для хранения данных:

1. [управление данными о состоянии с помощью Cosmos DB](bot-builder-nodejs-state-azure-cosmosdb.md);

2. [Управление данными о состоянии с помощью хранилища таблиц](bot-builder-nodejs-state-azure-table-storage.md)

При выборе любого из этих [расширений Azure](https://www.npmjs.com/package/botbuilder-azure) механизм установки и сохранения данных с помощью пакета SDK Bot Framework для Node.js остается тем же, что для хранилища данных в памяти.

## <a name="storage-containers"></a>Контейнеры хранилища

В пакете SDK построителя ботов для Node.js объект `session` предоставляет следующие свойства для хранения данных о состоянии.

| Свойство | Область действия | ОПИСАНИЕ |
| ---- | ---- | ---- |
| [`userData`][userDataURL] | Пользователь | Содержит данные, которые сохраняются для пользователя в указанном канале. Эти данные будут сохраняться между несколькими беседами. |
| [`privateConversationData`][privateConversationDataURL] | Беседа | Содержит данные, которые сохраняются для пользователя в указанном канале в контексте конкретной беседы. Эти данные являются частными для конкретного пользователя и будут сохраняться только для текущей беседы. Свойство очищается, когда беседа завершается или когда `endConversation` вызывается явным образом. |
| [`conversationData`][conversationDataURL] | Беседа | Содержит данные, которые сохраняются в указанном канале в контексте конкретной беседы. Эти данные используются совместно всеми пользователями, участвующими в беседе, и будут сохраняться только для текущей беседы. Свойство очищается, когда беседа завершается или когда `endConversation` вызывается явным образом. |
| [`dialogData`][dialogDataURL] | Диалог | Содержит данные, сохраненные только для текущего диалога. Каждый диалог поддерживает собственную копию этого свойства. Свойство очищается, когда диалог удаляется из стека диалогов. |

Эти четыре свойства соответствуют четырем контейнерам хранилища данных, которые могут использоваться для хранения данных. Свойства, которые можно использовать для хранения данных, зависят от соответствующей области для хранения данных, природы данных и продолжительности хранения. Например, если вам необходимо сохранить данные пользователя, которые должны быть доступны для нескольких бесед, рассмотрите возможность использования свойства `userData`. Если вам необходимо временно хранить значения локальной переменной в области диалога, рассмотрите возможность использования свойства `dialogData`. Если вам необходимо временно хранить данные, которые должны быть доступны для нескольких диалогов, рассмотрите возможность использования свойства `conversationData`.

## <a name="data-persistence"></a>Сохраняемость данных

По умолчанию данные, которые хранятся с помощью свойств `userData`, `privateConversationData` и `conversationData`, сохраняются после завершения беседы. Если вы не хотите, чтобы эти данные сохранялись в контейнере `userData`, для `persistUserData` установите флаг **false**. Если вы не хотите, чтобы эти данные сохранялись в контейнере `conversationData`, для `persistConversationData` установите флаг **false**. 

```javascript
// Do not persist userData
bot.set(`persistUserData`, false);

// Do not persist conversationData
bot.set(`persistConversationData`, false);
```

> [!NOTE]
> Невозможно полностью отключить функцию сохранения данных для контейнера `privateConversationData`.

## <a name="set-data"></a>Задание данных

Вы можете хранить простые объекты JavaScript, сохраняя их непосредственно в контейнер хранилища. Сложные объекты, например `Date`, следует преобразовать в `string`. Это связано с тем, что данные о состоянии сериализуются и сохраняются в формате JSON. В следующих примерах кода показано, как сохранить примитивные данные, массив данных, карту объекта и сложный объект `Date`. 

**Сохранение примитивных данных**

```javascript
session.userData.userName = "Kumar Sarma";
session.userData.userAge = 37;
session.userData.hasChildren = true;
```

**Сохранение массива данных**

```javascript
session.userData.profile = ["Kumar Sarma", "37", "true"];
```

**Сохранение карты объекта**

```javascript
session.userData.about = {
    "Profile": {
        "Name": "Kumar Sarma",
        "Age": 37,
        "hasChildren": true
    },
    "Job": {
        "Company": "Contoso",
        "StartDate": "June 8th, 2010",
        "Title": "Developer"
    }
}
```
**Сохранение времени и даты** 

При работе со сложным объектом JavaScript перед сохранением в контейнер хранилища его следует преобразовать в строку. 

```javascript 
var startDate = builder.EntityRecognizer.resolveTime([results.response]); 

// Date as string: "2017-08-23T05:00:00.000Z" 
session.userdata.start = startDate.toISOString(); 
``` 

### <a name="saving-data"></a>Сохранение данных

Данные, созданные в каждом контейнере хранилища, будут оставаться в памяти до сохранения контейнера. Пакет SDK построителя ботов для Node.js отправляет данные в службу `ChatConnector` в виде пакетов, сохраняемых при появлении сообщений для отправки. Чтобы сохранить данные, которые находятся в контейнерах хранилища без отправки сообщений, можно вручную вызвать метод [`save`](https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#save). Если метод `save` не вызывается, данные, находящиеся в контейнерах хранилища, будут сохранены в ходе обработки пакета.

```javascript
session.userData.favoriteColor = "Red";
session.userData.about.job.Title = "Senior Developer"; 
session.save();
```

## <a name="get-data"></a>Получение данных

Для доступа к данным, которые сохраняются в конкретном контейнере хранилища, просто сошлитесь на соответствующее свойство. В следующих примерах кода показано, как получить доступ к данным, которые были сохранены как примитивные данные, массив данных, карта объекта или сложный объект даты.

**Доступ к примитивным данным**

```javascript
var userName = session.userData.userName;
var userAge = session.userData.userAge;
var hasChildren = session.userData.hasChildren;
```

**Доступ к массиву данных**

```javascript
var userProfile = session.userData.userProfile;

session.send("User Profile:");
for(int i = 0; i < userProfile.length, i++){
    session.send(userProfile[i]);
}
```

**Доступ к карте объекта**

```javascript
var about = session.userData.about;

session.send("User %s works at %s.", about.Profile.Name, about.Job.Company);
```

**Доступ к объекту даты** 

Извлеките данные даты в виде строки, а затем преобразуйте их в объект даты JavaScript. 

```javascript 
// startDate as a JavaScript Date object. 
var startDate = new Date(session.userdata.start); 
``` 

## <a name="delete-data"></a>Удаление данных

По умолчанию данные, хранящиеся в контейнере `dialogData`, очищаются, когда диалог удаляется из стека диалогов. Аналогично, данные, хранящиеся в контейнерах `conversationData` и `privateConversationData`, очищаются после вызова метода `endConversation`. Тем не менее для удаления данных, хранимых в контейнере `userData`, вам необходимо очистить их явным образом.

Чтобы явным образом очистить данные, которые хранятся в одном из контейнеров хранилища, просто сбросьте контейнер, как показано в следующем примере кода. 

```javascript
// Clears data stored in container.
session.userData = {}; 
session.privateConversationData = {};
session.conversationData = {};
session.dialogData = {};
```

Не задавайте для контейнера данных значение `null` и не удаляйте его из объекта `session`, так как это вызовет ошибку при следующей попытке доступа к контейнеру. Кроме того, возможно, потребуется вручную вызвать `session.save();` после того, как вы вручную удалили контейнер в памяти, чтобы очистить все соответствующие данные, которые были сохранены ранее.

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы ознакомились с механизмами управления данными о состоянии, давайте разберемся, как можно их использовать для более эффективного управления потоком беседы.

> [!div class="nextstepaction"]
> [Manage conversation flow with dialogs](bot-builder-nodejs-dialog-manage-conversation-flow.md) (Управление последовательностью общения с помощью диалоговых окон)

## <a name="additional-resources"></a>Дополнительные ресурсы
- [Запрос пользователю на ввод данных](bot-builder-nodejs-dialog-prompt.md)

[userDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#userdata
[conversationDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#conversationdata
[privateConversationDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#privateconversationdata
[dialogDataURL]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.session.html#dialogdata

[ChatConnector]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html

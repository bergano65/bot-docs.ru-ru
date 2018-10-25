---
title: Сущности и типы действий | Документация Майкрософт
description: Сущности и типы действий.
keywords: Упоминание сущности. Типы действий. Использование сущностей
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/01/2018
ms.openlocfilehash: d2f4692580843f530641827707d250fa726830e3
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000401"
---
# <a name="entities-and-activity-types"></a>Сущности и типы действий

Сущности являются частью действия и предоставляют дополнительную информацию о действии или общении.

[!include[Entity boilerplate](includes/snippet-entity-boilerplate.md)]

## <a name="entities"></a>Сущности

Свойство *entities* сообщения представляет собой массив открытых объектов <a href="http://schema.org/" target="_blank">schema.org</a>, что позволяет осуществлять обмен контекстно-зависимых метаданных между каналом и ботом.

### <a name="mention-entities"></a>Сущности Mention

Многие каналы поддерживают способность бота или пользователя "упоминать" кого-то в контексте общения.
Чтобы упомянуть пользователя в сообщении, заполните свойство сущностей сообщения *упомянутым* объектом.
Объект mention содержит следующие свойства.

| Свойство | ОПИСАНИЕ |
|----|----|
| type | Тип сущности (Mention). |
| Mentioned | Объект учетной записи канала, который показывает, какой пользователь был упомянут. | 
| текст | Текст внутри свойства *activity.text* представляет собой само упоминание (может быть пустым или иметь значение NULL). |

Данный пример кода показывает, как добавить сущность mention в коллекцию сущностей.

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set Mention](includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> При попытке определить намерение пользователя бот может захотеть проигнорировать ту часть сообщения, где о ней упоминается. Назовите метод `GetMentions` и оцените объекты `Mention`, которые возвращены в ответ.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const mention = {
    type: "Mention",
    text: "@johndoe",
    mentioned: {
        name: "John Doe",
        id: "UV341235"
    }
}

entity = [mention];
```

---

### <a name="place-objects"></a>Объекты Place

<a href="https://schema.org/Place" target="_blank">Сведения, связанные с местоположением</a>, могут быть переданы в сообщении путем заполнения свойства entities сообщения с помощью объекта *Place* или *GeoCoordinates*.

Объект place содержит следующие свойства.

| Свойство | ОПИСАНИЕ |
|----|----|
| type | Тип сущности (Place). |
| Адрес | Описание или почтовый адрес объекта (будущее) |
| Географические | GeoCoordinates |
| HasMap | URL-адрес карты или объект map (будущее) |
| ИМЯ | Название расположения. |

Объект geoCoordinates содержит следующие свойства.

| Свойство | ОПИСАНИЕ |
|----|----|
| type | Тип сущности (GeoCoordinates). |
| ИМЯ | Название расположения. |
| Долгота | Долгота расположения (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>). |
| Долгота | Широта расположения (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>). |
| Elevation | Высота расположения (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>) |

Данный пример кода показывает, как добавить упоминание сущности в коллекцию.

# <a name="ctabcs"></a>[C#](#tab/cs)
[!code-csharp[set GeoCoordinates](includes/code/dotnet-create-messages.cs#setGeoCoord)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
```javascript
var entity = context.activity.entities;

const place = {
    elavation: 100,
    type: "GeoCoordinates",
    name : "myPlace",
    latitude: 123,
    longitude: 234
};

entity = [place];

```

---

### <a name="consume-entities"></a>Использование сущностей

# <a name="ctabcs"></a>[C#](#tab/cs)

Чтобы использовать сущности, используйте `dynamic` ключевое слово или классы со строгой типизацией.

Данный пример кода показывает, как использовать `dynamic` ключевое слово для обработки сущности внутри `Entities` свойства сообщения.

[!code-csharp[examine entity using dynamic keyword](includes/code/dotnet-create-messages.cs#examineEntity1)]

В этом примере кода показано, как использовать строго типизированный класс для обработки объекта в свойстве `Entities` сообщения.

[!code-csharp[examine entity using typed class](includes/code/dotnet-create-messages.cs#examineEntity2)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Данный пример кода показывает, как использовать сущность `entity` сообщения:

```javascript
if (entity[0].type === "GeoCoordinates" && entity[0].latitude > 34) {
    // do something
}
```

---

## <a name="activity-types"></a>Типы действий

В этом примере кода показано, как обрабатывать действие типа **message**:

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
if (context.Activity.Type == ActivityTypes.Message){
    // do something
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```js
if(context.activity.type === 'message'){
    // do something
}
```

---

Действия могут быть различных типов, самыми распространенными являются **сообщение**. Существует несколько типов действий.

| Тип действия | Интерфейс | ОПИСАНИЕ |
|-----|-----|-----|
| [message](#message) | IMessageActivity (C#) <br> Действие (JS) | Представляет связь между ботом и пользователем. |
| [contactRelationUpdate](#contactrelationupdate) | IContactRelationUpdateActivity (C#) <br> Действие (JS) | Указывает, что бот был добавлен или удален из списка контактов пользователя. |
| [conversationUpdate](#conversationupdate) | IConversationUpdateActivity (C#) <br> Действие (JS) | Указывает, что бот был добавлен в общение, другие участники были добавлены или удалены из него или метаданные общения изменились. |
| [deleteUserData](#deleteuserdata) | Недоступно | Указывает боту, что пользователь выполнил запрос на удаление любых пользовательских данных, которые бот мог сохранить. |
| [endOfConversation](#endofconversation) | IEndOfConversationActivity (C#) <br> Действие (JS) | Указывает на конец общения. |
| [event](#event) | IEventActivity (C#) <br> Действие (JS) | Представляет связь, отправляемую боту, который не отображается для пользователя. |
| [installationUpdate](#installationupdate) | IInstallationUpdateActivity (C#) <br> Действие (JS) | Представляет установки или удаления бота, внутри подразделения (например, клиента или "команды") канала. |
| [invoke](#invoke) | IInvokeActivity (C#) <br> Действие (JS) | Представляет связь, отправляемую боту для запроса, что он исполняет определенную операцию. Этот тип действия зарезервирован для внутреннего использования в Microsoft Bot Framework. |
| [messageReaction](#messagereaction) | IMessageReactionActivity (C#) <br> Действие (JS) | Указывает, что пользователь отреагировал на существующее действие. Например, в сообщение, пользователь нажимает кнопку "Like". |
| [typing](#typing) | ITypingActivity (C#) <br> Действие (JS) | Указывает, что пользователь или бот на другом конце общения составляет ответ. |

## <a name="message"></a>Message

<!-- Only the last link is different. -->
::: moniker range="azure-bot-service-3.0"
Бот отправляет действия message для передачи сведений пользователям и получения действий message от пользователей.
Некоторые сообщения могут состоять из простого текста, в то время как другие могут содержать более богатое содержимое, такое как [произносимый текст](v4sdk/bot-builder-howto-send-messages.md#send-a-spoken-message), [предлагаемые действия](v4sdk/bot-builder-howto-add-suggested-actions.md), [мультимедийные вложения](v4sdk/bot-builder-howto-add-media-attachments.md), [форматированные карточки](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card) и [данные по каналу](~/dotnet/bot-builder-dotnet-channeldata.md).
::: moniker-end
::: moniker range="azure-bot-service-4.0"
Бот отправляет действия message для передачи сведений пользователям и получения действий message от пользователей.
Некоторые сообщения могут состоять из простого текста, в то время как другие могут содержать более богатое содержимое, такое как [произносимый текст](v4sdk/bot-builder-howto-send-messages.md#send-a-spoken-message), [предлагаемые действия](v4sdk/bot-builder-howto-add-suggested-actions.md), [мультимедийные вложения](v4sdk/bot-builder-howto-add-media-attachments.md), [форматированные карточки](v4sdk/bot-builder-howto-add-media-attachments.md#send-a-hero-card) и [данные по каналу](~/v4sdk/bot-builder-channeldata.md).
::: moniker-end

## <a name="contactrelationupdate"></a>contactRelationUpdate

Бот совершает действие обновления отношений контактов при каждом добавлении или удалении из списка контактов пользователя. Значение свойства action действия (add | remove) указывает, был ли бот добавлен или удален из списка контактов пользователя.

## <a name="conversationupdate"></a>conversationUpdate

Бот получает активность обновления общения всякий раз, когда его добавляют в общение, добавляются или удаляются из общения другие участники или меняются метаданные общения.

Если элементы были добавлены к общению, элементы действия добавленного свойства будут содержать массив объектов учетной записи канала для идентификации новых элементов.

Чтобы определить, был ли добавлен бот к общению (является одним из новых элементов), оценить соответствует ли получатель идентификатора значению действия (идентификатор бота) свойству идентификатора для добавления каких-либо учетных записей в элементы массива.

Если элементы были удалены из общения, элементы действия добавленного свойства будут содержать массив объектов учетной записи канала для идентификации новых элементов.

> [!TIP]
> Если бот получает действия обновления общения, указывающее, что пользователь вступает в общение, можно ответить на него, отправив приветственное сообщение.

## <a name="deleteuserdata"></a>deleteUserData

Бот получает действие удаления пользовательских данных, когда пользователь запрашивает удаление любых данных, которые были ранее сохранены. Если бот получает действия такого типа, следует удалить все личные сведения, который ранее хранились для пользователя, выполнившего запрос.

## <a name="endofconversation"></a>endOfConversation

Бот получает действие конец общения, чтобы указать, что пользователь закончил общение. Бот может отправить действие конца общения, чтобы сказать пользователю, что общение окончено.

## <a name="event"></a>event

Бот может получить действие event от внешнего процесса или службы, которые передают информацию боту, чтобы эта информация не была видна пользователям. Отправитель действия event обычно не ожидает, что бот каким-либо способом подтвердит получение.

## <a name="installationupdate"></a>installationUpdate

Действия обновления установки представляют собой установку или удаление бота внутри организационного подразделения (например, арендатора клиента или команды) канала. Действия обновления установки обычно не представляют собой добавление или удаление канала. Каналы могут отправлять действия установки, когда бот добавляется или удаляется из клиента, команды или других подразделений в канале. Каналам не следует отправлять действия установки, когда туда установлен или удален бот.

## <a name="invoke"></a>invoke

Бот может использовать действие invoke, которое представляет запрос для выполнения конкретной операции.
Отправитель действия invoke обычно предполагает, что бот подтвердит получение с помощью HTTP-ответа.
Этот тип действия зарезервирован для внутреннего использования в Microsoft Bot Framework.

## <a name="messagereaction"></a>messageReaction

Некоторые каналы отправляют действия реакции на сообщение боту, когда пользователь отреагировал на существующее действие. Например, в сообщение, пользователь нажимает кнопку "Like". Свойство replyToId будет указывать, на которое действие отреагировал пользователь.

Действию message reaction может соответствовать любое количество типов реакции сообщений, которые определены в канале. Например, канал может отправлять тип реакции "Like" или "PlusOne".

## <a name="typing"></a>typing

Бот получает действие typing, чтобы указать, что пользователь вводит ответ.
Бот может отправлять действие typing, чтобы указать пользователю, что выполняется запрос или составление ответа.

::: moniker range="azure-bot-service-3.0"
## <a name="additional-resources"></a>Дополнительные ресурсы

- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
::: moniker-end

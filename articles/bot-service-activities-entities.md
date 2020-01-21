---
title: Сущности и типы действий — Служба Azure Bot
description: Сущности и типы действий.
keywords: Упоминание сущности. Типы действий. Использование сущностей
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/01/2018
ms.openlocfilehash: cd11cc1fbbacb7e555da4e00337d6fd4b79a4df6
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789081"
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

| Свойство | Description |
|----|----|
| Тип | Тип сущности (Mention). |
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

| Свойство | Description |
|----|----|
| Тип | Тип сущности (Place). |
| Адрес | Описание или почтовый адрес объекта (будущее) |
| Географические | GeoCoordinates |
| HasMap | URL-адрес карты или объект map (будущее) |
| Имя | Название расположения. |

Объект geoCoordinates содержит следующие свойства.

| Свойство | Description |
|----|----|
| Тип | Тип сущности (GeoCoordinates). |
| Имя | Название расположения. |
| Долгота | Долгота расположения (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>). |
| Широта | Широта расположения (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>). |
| Elevation | Высота расположения (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>). |

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
<!-- 
This code example show how to process an activity of type **message**:

# [C#](#tab/cs)

```cs
if (context.Activity.Type == ActivityTypes.Message){
    // do something
}
```

# [JavaScript](#tab/js)

```js
if(context.activity.type === 'message'){
    // do something
}
```

--- -->

Действия могут быть различных типов, самыми распространенными являются **сообщение**. Объяснения и дополнительные сведения о разных типах действий см. в [схеме действий Bot Framework](https://aka.ms/botSpecs-activitySchema).

::: moniker range="azure-bot-service-3.0"

## <a name="additional-resources"></a>Дополнительные ресурсы

- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
::: moniker-end

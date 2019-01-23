---
title: Создание сообщений с помощью пакета SDK Bot Framework для .NET | Документация Майкрософт
description: Сведения о типичных свойствах сообщений в пакете SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 96a0c63575b1e77418262a22050013413f39141f
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225919"
---
# <a name="create-messages"></a>Создание сообщений

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Бот будет отправлять [действия](bot-builder-dotnet-activities.md) **Message** для передачи информации пользователям, а также будет получать действия **Message** от пользователей. Некоторые сообщения могут состоять из простого текста, в то время как другие могут содержать более богатое содержимое, такое как [произносимый текст](bot-builder-dotnet-text-to-speech.md), [предлагаемые действия](bot-builder-dotnet-add-suggested-actions.md), [мультимедийные вложения](bot-builder-dotnet-add-media-attachments.md), [форматированные карточки](bot-builder-dotnet-add-rich-card-attachments.md) и [данные по каналу](bot-builder-dotnet-channeldata.md). 

В этой статье описаны некоторые из наиболее часто используемых свойств сообщений.

## <a name="customizing-a-message"></a>Настройка сообщения

Чтобы обеспечить больший контроль над форматированием текста сообщений, можно создать пользовательский объект [Activity](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) и задать необходимые свойства перед его отправкой пользователю.

В этом примере показано, как создать пользовательский объект `message` и задать свойства `Text`, `TextFormat` и `Local`.

[!code-csharp[Set message properties](../includes/code/dotnet-create-messages.cs#setBasicProperties)]

Свойство `TextFormat` сообщения может использоваться для указания формата текста. Свойству `TextFormat` может быть присвоено значение **plain**, **markdown** или **xml**. Значение `TextFormat` по умолчанию — **markdown**. 

## <a name="attachments"></a>Вложения

Свойство `Attachments` действия Message может использоваться для отправки и получения простых мультимедийных вложений (изображений, аудио, видео, файлов) и форматированных карточек. Дополнительные сведения см. в разделах [Добавление мультимедийных вложений в сообщения](bot-builder-dotnet-add-media-attachments.md) и [Добавление вложений в виде форматированных карточек в сообщения](bot-builder-dotnet-add-rich-card-attachments.md).

## <a name="entities"></a>Сущности

Свойство `Entities` сообщения представляет собой массив открытых объектов <a href="http://schema.org/" target="_blank">schema.org</a>, что позволяет осуществлять обмен контекстно-зависимыми метаданными между каналом и ботом.

### <a name="mention-entities"></a>Сущности Mention

Многие каналы поддерживают способность бота или пользователя "упоминать" кого-то в контексте общения. Чтобы упомянуть пользователя в сообщении, заполните свойство `Entities` сообщения объектом `Mention`. Объект `Mention` содержит следующие свойства. 

| Свойство | ОПИСАНИЕ | 
|----|----|
| type | Тип сущности (Mention). | 
| Mentioned | Объект `ChannelAccount`, который показывает, какой пользователь был упомянут. | 
| текст | Текст внутри свойства `Activity.Text` представляет собой само упоминание (может быть пустым или иметь значение NULL). |

Данный пример кода показывает, как добавить сущность `Mention` в коллекцию `Entities`.

[!code-csharp[set Mention](../includes/code/dotnet-create-messages.cs#setMention)]

> [!TIP]
> При попытке определить намерение пользователя бот может захотеть проигнорировать ту часть сообщения, где о ней упоминается. Назовите метод `GetMentions` и оцените объекты `Mention`, которые возвращены в ответ.

### <a name="place-objects"></a>Объекты Place

<a href="https://schema.org/Place" target="_blank">Сведения, связанные с местоположением</a>, могут быть переданы в сообщении путем заполнения свойства `Entities` сообщения с помощью объекта `Place` или `GeoCoordinates`. 

Объект `Place` содержит следующие свойства.

| Свойство | ОПИСАНИЕ | 
|----|----|
| type | Тип сущности (Place). |
| Адрес | Описание или `PostalAddress` объект (будущее). | 
| Географические | GeoCoordinates | 
| HasMap | URL-адрес карты или объект `Map` (будущее). |
| ИМЯ | Название расположения. |

Объект `GeoCoordinates` содержит следующие свойства.

| Свойство | ОПИСАНИЕ | 
|----|----|
| type | Тип сущности (GeoCoordinates). |
| ИМЯ | Название расположения. |
| Долгота | Долгота расположения (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>). | 
| Широта | Широта расположения (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>). | 
| Elevation | Высота расположения (<a href="https://en.wikipedia.org/wiki/World_Geodetic_System" target="_blank">WGS 84</a>). | 

Данный пример кода показывает, как добавить сущность `Place` в коллекцию `Entities`.

[!code-csharp[set GeoCoordinates](../includes/code/dotnet-create-messages.cs#setGeoCoord)]

### <a name="consume-entities"></a>Использование сущностей

Чтобы использовать сущности, используйте `dynamic` ключевое слово или классы со строгой типизацией.

Данный пример кода показывает, как использовать `dynamic` ключевое слово для обработки сущности внутри `Entities` свойства сообщения.

[!code-csharp[examine entity using dynamic keyword](../includes/code/dotnet-create-messages.cs#examineEntity1)]

В этом примере кода показано, как использовать строго типизированный класс для обработки объекта в свойстве `Entities` сообщения.

[!code-csharp[examine entity using typed class](../includes/code/dotnet-create-messages.cs#examineEntity2)]

## <a name="channel-data"></a>Данные канала

Свойство `ChannelData` действия Message может использоваться для реализации функциональных возможностей канала. Дополнительные сведения см. в разделе [Реализация функциональных возможностей канала](bot-builder-dotnet-channeldata.md).

## <a name="text-to-speech"></a>Преобразование текста в речь

Свойство `Speak` действия Message может использоваться для указания текста, произносимого ботом через канал с поддержкой речевых функций. Свойство `InputHint` действия Message может использоваться для управления состоянием микрофона и поля ввода клиента (если таковые имеются). Дополнительные сведения см. в разделе [Добавление речи в сообщения](bot-builder-dotnet-text-to-speech.md).

## <a name="suggested-actions"></a>Предлагаемые действия

Свойство `SuggestedActions` действия Message может использоваться для представления кнопок, которых пользователь может коснуться для ввода входных данных. В отличие от кнопок, которые появляются в функциональных карточках (и которые остаются видимыми и доступными для пользователя даже после касания), кнопки, отображаемые в области предлагаемых действий, исчезнут, как только будет сделан выбор. Дополнительные сведения см. в разделе [Добавление предлагаемых действий к сообщениям](bot-builder-dotnet-add-suggested-actions.md).

## <a name="next-steps"></a>Дополнительная информация

Бот и пользователь могут отправлять друг другу сообщения. Если сообщение более сложное, бот может отправить в сообщении пользователю форматированную карточку. Форматированные карточки охватывают множество сценариев представления и взаимодействия, часто требуемые в большинстве ботов.

> [!div class="nextstepaction"]
> [Добавление вложений в виде форматированных карточек в сообщения](bot-builder-dotnet-add-rich-card-attachments.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Общие сведения о действиях](bot-builder-dotnet-activities.md)
- [Действия отправки и получения](bot-builder-dotnet-connector.md)
- [Добавление мультимедийных вложений в сообщения](bot-builder-dotnet-add-media-attachments.md)
- [Добавление вложений в виде форматированных карточек в сообщения](bot-builder-dotnet-add-rich-card-attachments.md)
- [Добавление речи в сообщения](bot-builder-dotnet-text-to-speech.md)
- [Добавление предлагаемых действий в сообщения](bot-builder-dotnet-add-suggested-actions.md)
- [Реализация возможностей для определенных каналов](bot-builder-dotnet-channeldata.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">Интерфейс IMessageActivity</a>


---
title: Создание сообщений с помощью API службы Bot Connector — Служба Azure Bot
description: Узнайте о часто используемых свойствах сообщений в API службы Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: f3f084f65c1fb4fc84a6c8c75ce36b56487ebd0d
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789969"
---
# <a name="create-messages"></a>Создание сообщений

Бот будет отправлять объекты [Действие][] типа **message** для передачи информации пользователям, а также будет получать действия **message** от пользователей. Некоторые сообщения могут состоять из простого текста, в то время как другие могут содержать более богатое содержимое, такое как [произносимый текст](bot-framework-rest-connector-text-to-speech.md), [предлагаемые действия](bot-framework-rest-connector-add-suggested-actions.md), [мультимедийные вложения](bot-framework-rest-connector-add-media-attachments.md), [форматированные карточки](bot-framework-rest-connector-add-rich-cards.md) и [данные по каналу](bot-framework-rest-connector-channeldata.md). В этой статье описаны некоторые из наиболее часто используемых свойств сообщений.

## <a name="message-text-and-formatting"></a>Текст сообщения и форматирование

Текст сообщения можно форматировать с помощью **plain**, **markdown** или **xml**. Формат по умолчанию для свойства `textFormat` — **markdown**, который интерпретирует текст с использованием стандартов форматирования Markdown. Уровень поддержки формата текста меняется в зависимости от каналов. 

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

Свойство `textFormat` объекта [Действие][] может использоваться для указания формата текста. Например, чтобы создать простое сообщение, которое содержит только обычный текст, задайте для свойства `textFormat` объекта `Activity` значение **plain**, для свойства `text` — значение содержимого сообщения, а для свойства `locale` — языковой стандарт отправителя. 

## <a name="attachments"></a>Вложения

Свойство `attachments` объекта [Действие][] может использоваться для отправки простых мультимедийных вложений (изображений, аудио, видео, файлов) и форматированных карточек. Дополнительные сведения см. в разделах [Добавление мультимедийных вложений в сообщения](bot-framework-rest-connector-add-media-attachments.md) и [Добавление вложений в виде форматированных карточек в сообщения](bot-framework-rest-connector-add-rich-cards.md).

## <a name="entities"></a>Сущности

Свойство `entities` объекта [Действие][] представляет собой массив открытых объектов <a href="http://schema.org/" target="_blank">schema.org</a>, который позволяет осуществлять обмен контекстно-зависимыми метаданными между каналом и ботом.

### <a name="mention-entities"></a>Сущности Mention

Многие каналы поддерживают способность бота или пользователя "упоминать" кого-то в контексте общения. Чтобы упомянуть пользователя в сообщении, заполните свойство `entities` сообщения объектом [Mention][]. 

### <a name="place-entities"></a>Сущность Place

Чтобы передать в сообщение <a href="https://schema.org/Place" target="_blank">сведения, связанные с расположением</a>, заполните свойство `entities` сообщения объектом [Place][]. 

## <a name="channel-data"></a>Данные канала

Свойство `channelData` объекта [Действие][] может использоваться для реализации функциональных возможностей канала. Дополнительные сведения см. в разделе [Реализация функциональных возможностей канала](bot-framework-rest-connector-channeldata.md).

## <a name="text-to-speech"></a>Преобразование текста в речь

Свойство `speak` объекта [Действие][] может использоваться для указания текста, который будет произноситься ботом по каналу с поддержкой речевых функций, а свойство `inputHint` объекта `Activity` может использоваться для воздействия на состояние микрофона клиента. Дополнительные сведения см. в разделах [Добавление речи в сообщения](bot-framework-rest-connector-text-to-speech.md) и [Добавление подсказок ввода к сообщениям](bot-framework-rest-connector-add-input-hints.md).

## <a name="suggested-actions"></a>Предлагаемые действия

Свойство `suggestedActions` объекта [Действие][] может использоваться для представления кнопок, которых пользователь может коснуться для ввода данных. В отличие от кнопок, которые появляются в функциональных карточках (и которые остаются видимыми и доступными для пользователя даже после касания), кнопки, отображаемые в области предлагаемых действий, исчезнут, как только будет сделан выбор. Дополнительные сведения см. в разделе [Добавление предлагаемых действий к сообщениям](bot-framework-rest-connector-add-suggested-actions.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Справочник по каналам][ChannelInspector]
- [Общие сведения о действиях](https://aka.ms/botSpecs-activitySchema)
- [Отправка и получение сообщений](bot-framework-rest-connector-send-and-receive-messages.md)
- [Добавление мультимедийных вложений в сообщения](bot-framework-rest-connector-add-media-attachments.md)
- [Добавление вложений в виде форматированных карточек в сообщения](bot-framework-rest-connector-add-rich-cards.md)
- [Добавление речи в сообщения](bot-framework-rest-connector-text-to-speech.md)
- [Добавление подсказок для ввода в сообщения](bot-framework-rest-connector-add-input-hints.md)
- [Добавление предлагаемых действий в сообщения](bot-framework-rest-connector-add-suggested-actions.md)
- [Реализация возможностей для определенных каналов](bot-framework-rest-connector-channeldata.md)

[ChannelInspector]: ../bot-service-channels-reference.md
[textFormating]: ../bot-service-channel-inspector.md#text-formatting

[Действие]: bot-framework-rest-connector-api-reference.md#activity-object
[Mention]: bot-framework-rest-connector-api-reference.md#mention-object
[Place]: bot-framework-rest-connector-api-reference.md#place-object

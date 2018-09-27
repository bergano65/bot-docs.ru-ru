---
title: Создание сообщений с помощью API службы Bot Connector | Документация Майкрософт
description: Узнайте о часто используемых свойствах сообщений в API службы Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 62088ecdf2f8402ec5456eea758f5db994de0cf9
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707320"
---
# <a name="create-messages"></a>Создание сообщений

Бот будет отправлять объекты [Действия][Activity] типа **message** для передачи информации пользователям, а также будет получать действия **message** от пользователей. Некоторые сообщения могут состоять из простого текста, в то время как другие могут содержать более богатое содержимое, такое как [произносимый текст](bot-framework-rest-connector-text-to-speech.md), [предлагаемые действия](bot-framework-rest-connector-add-suggested-actions.md), [мультимедийные вложения](bot-framework-rest-connector-add-media-attachments.md), [форматированные карточки](bot-framework-rest-connector-add-rich-cards.md) и [данные по каналу](bot-framework-rest-connector-channeldata.md). В этой статье описаны некоторые из наиболее часто используемых свойств сообщений.

## <a name="message-text-and-formatting"></a>Текст сообщения и форматирование

Текст сообщения можно форматировать с помощью **plain**, **markdown** или **xml**. Формат по умолчанию для свойства `textFormat` — **markdown**, который интерпретирует текст с использованием стандартов форматирования Markdown. Уровень поддержки формата текста меняется в зависимости от каналов. Чтобы узнать, поддерживается ли функция, которую вы хотите использовать, на целевом канале, просмотрите функцию с помощью [Channel Inspector][ChannelInspector]. 

Свойство `textFormat` объекта [Действие][Activity] может использоваться для указания формата текста. Например, чтобы создать простое сообщение, которое содержит только обычный текст, установите свойству `textFormat` объекта [Действие][Activity] значение **plain**, свойству `text` — значение содержимого сообщения, а свойству `locale` — языковой стандарт отправителя. 

## <a name="attachments"></a>Вложения

Свойство `attachments` объекта [Действие][Activity] может использоваться для отправки простых мультимедийных вложений (изображений, аудио, видео, файлов) и расширенных карточек. Дополнительные сведения см. в разделах [Добавление мультимедийных вложений в сообщения](bot-framework-rest-connector-add-media-attachments.md) и [Добавление вложений в виде форматированных карточек в сообщения](bot-framework-rest-connector-add-rich-cards.md).

## <a name="entities"></a>Сущности

Свойство `entities` объекта [Действие][Activity] представляет собой массив открытых объектов <a href="http://schema.org/" target="_blank">schema.org</a>, который позволяет осуществлять обмен контекстно-зависимыми метаданными между каналом и ботом.

### <a name="mention-entities"></a>Сущности Mention

Многие каналы поддерживают способность бота или пользователя "упоминать" кого-то в контексте общения. Чтобы упомянуть пользователя в сообщении, заполните свойство `entities` сообщения объектом [Mention][Mention]. 

### <a name="place-entities"></a>Сущность Place

Для передачи <a href="https://schema.org/Place" target="_blank">сведений, связанных с местоположением</a>, в сообщении заполните свойство `entities` сообщения объектом [Place][Place]. 

## <a name="channel-data"></a>Данные канала

Свойство `channelData` объекта [Действие][Activity] может использоваться для реализации функциональных возможностей канала. Дополнительные сведения см. в разделе [Реализация функциональных возможностей канала](bot-framework-rest-connector-channeldata.md).

## <a name="text-to-speech"></a>Преобразование текста в речь

Свойство `speak` объекта [Действие][Activity] может использоваться для указания текста, который будет произноситься ботом по каналу с поддержкой речевых функций, а свойство `inputHint` объекта [Действие][Activity] может использоваться для воздействия на состояние микрофона клиента. Дополнительные сведения см. в разделах [Добавление речи в сообщения](bot-framework-rest-connector-text-to-speech.md) и [Добавление подсказок ввода к сообщениям](bot-framework-rest-connector-add-input-hints.md).

## <a name="suggested-actions"></a>Рекомендуемые действия

Свойство `suggestedActions` объекта [Действие][Activity] может использоваться для представления кнопок, которых пользователь может коснуться для ввода входных данных. В отличие от кнопок, которые появляются в расширенных карточках (и которые остаются видимыми и доступными для пользователя даже после касания), кнопки, отображаемые в области предлагаемых действий, исчезнут, как только будет сделан выбор. Дополнительные сведения см. в разделе [Добавление предлагаемых действий к сообщениям](bot-framework-rest-connector-add-suggested-actions.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Предварительный просмотр компонентов с помощью Channel Inspector][ChannelInspector]
- [Общие сведения о действиях](bot-framework-rest-connector-activities.md)
- [Отправка и получение сообщений](bot-framework-rest-connector-send-and-receive-messages.md)
- [Добавление мультимедийных вложений в сообщения](bot-framework-rest-connector-add-media-attachments.md)
- [Добавление вложений в виде форматированных карточек в сообщения](bot-framework-rest-connector-add-rich-cards.md)
- [Добавление речи в сообщения](bot-framework-rest-connector-text-to-speech.md)
- [Добавление подсказок ввода к сообщениям](bot-framework-rest-connector-add-input-hints.md)
- [Добавление предлагаемых действий к сообщениям](bot-framework-rest-connector-add-suggested-actions.md)
- [Реализация функциональных возможностей канала](bot-framework-rest-connector-channeldata.md)

[Mention]: bot-framework-rest-connector-api-reference.md#mention-object
[Place]: bot-framework-rest-connector-api-reference.md#place-object
[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
[ChannelInspector]: ../bot-service-channel-inspector.md
[textFormating]: ../bot-service-channel-inspector.md#text-formatting

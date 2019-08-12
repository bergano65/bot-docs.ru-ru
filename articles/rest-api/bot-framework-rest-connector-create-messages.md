---
title: Создание сообщений с помощью API службы Bot Connector | Документация Майкрософт
description: Узнайте о часто используемых свойствах сообщений в API службы Bot Connector.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 7a66e8a89156e9f55daa549d75285f6de8fa9610
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757160"
---
# <a name="create-messages"></a>Создание сообщений

Бот будет отправлять объекты `Activity` с типом **message** для передачи информации пользователям, а также будет получать действия **message** от пользователей. Некоторые сообщения могут состоять из простого текста, в то время как другие могут содержать более богатое содержимое, такое как [произносимый текст](bot-framework-rest-connector-text-to-speech.md), [предлагаемые действия](bot-framework-rest-connector-add-suggested-actions.md), [мультимедийные вложения](bot-framework-rest-connector-add-media-attachments.md), [форматированные карточки](bot-framework-rest-connector-add-rich-cards.md) и [данные по каналу](bot-framework-rest-connector-channeldata.md). В этой статье описаны некоторые из наиболее часто используемых свойств сообщений.

## <a name="message-text-and-formatting"></a>Текст сообщения и форматирование

Текст сообщения можно форматировать с помощью **plain**, **markdown** или **xml**. Формат по умолчанию для свойства `textFormat` — **markdown**, который интерпретирует текст с использованием стандартов форматирования Markdown. Уровень поддержки формата текста меняется в зависимости от каналов. Чтобы узнать, поддерживается ли функция, которую вы хотите использовать, на целевом канале, просмотрите функцию с помощью [Channel Inspector][ChannelInspector]. 

Свойство `textFormat` объекта `Activity` может использоваться для указания формата текста. Например, чтобы создать простое сообщение, которое содержит только обычный текст, задайте для свойства `textFormat` объекта `Activity` значение **plain**, для свойства `text` — значение содержимого сообщения, а для свойства `locale` — языковой стандарт отправителя. 

## <a name="attachments"></a>Вложения

Свойство `attachments` объекта `Activity` может использоваться для отправки простых мультимедийных вложений (изображений, аудио, видео, файлов) и форматированных карточек. Дополнительные сведения см. в разделах [Добавление мультимедийных вложений в сообщения](bot-framework-rest-connector-add-media-attachments.md) и [Добавление вложений в виде форматированных карточек в сообщения](bot-framework-rest-connector-add-rich-cards.md).

## <a name="entities"></a>Сущности

Свойство `entities` объекта `Activity` представляет собой массив открытых объектов <a href="http://schema.org/" target="_blank">schema.org</a>, который позволяет обмениваться метаданными между каналом и ботом с учетом контекста.

### <a name="mention-entities"></a>Сущности Mention

Многие каналы поддерживают способность бота или пользователя "упоминать" кого-то в контексте общения. Чтобы упомянуть пользователя в сообщении, заполните свойство `entities` сообщения объектом `Mention`. 

### <a name="place-entities"></a>Сущность Place

Чтобы включить в сообщение <a href="https://schema.org/Place" target="_blank">сведения, связанные с местоположением</a>, в его свойстве `entities` передайте объект `Place`. 

## <a name="channel-data"></a>Данные канала

Свойство `channelData` объекта `Activity` может использоваться для реализации функциональных возможностей конкретного канала. Дополнительные сведения см. в разделе [Реализация функциональных возможностей канала](bot-framework-rest-connector-channeldata.md).

## <a name="text-to-speech"></a>Преобразование текста в речь

Свойство `speak` объекта `Activity` позволяет указать текст, который будет произноситься ботом по каналу с поддержкой речевых функций, а свойство `inputHint` объекта `Activity` может использоваться для воздействия на состояние микрофона клиента. Дополнительные сведения см. в разделах [Добавление речи в сообщения](bot-framework-rest-connector-text-to-speech.md) и [Добавление подсказок ввода к сообщениям](bot-framework-rest-connector-add-input-hints.md).

## <a name="suggested-actions"></a>Предлагаемые действия

Свойство `suggestedActions` объекта `Activity` может использоваться для представления кнопок, с помощью которых пользователь сможете предоставлять входные данные. В отличие от кнопок, которые появляются в функциональных карточках (и которые остаются видимыми и доступными для пользователя даже после касания), кнопки, отображаемые в области предлагаемых действий, исчезнут, как только будет сделан выбор. Дополнительные сведения см. в разделе [Добавление предлагаемых действий к сообщениям](bot-framework-rest-connector-add-suggested-actions.md).

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

[ChannelInspector]: ../bot-service-channel-inspector.md
[textFormating]: ../bot-service-channel-inspector.md#text-formatting

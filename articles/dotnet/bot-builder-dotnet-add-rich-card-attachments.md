---
title: Добавление вложений в виде форматированных карточек в сообщения | Документы Майкрософт
description: Сведения о добавлении форматированных карточек в сообщения с помощью пакета SDK Bot Builder для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 4552fd8a38468b000837ef0f580d3a0e504a882b
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305454"
---
# <a name="add-rich-card-attachments-to-messages"></a>Добавление вложений в виде форматированных карточек в сообщения
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

В обмене сообщениями между пользователем и ботом может быть задействована одна или несколько форматированных карточек, отображаемых в виде списка или карусели. Свойство `Attachments` объекта <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> содержит массив объектов <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment</a>, представляющих вложения в виде форматированных карточек и файлов мультимедиа. 

> [!NOTE]
> Сведения о добавлении мультимедийных вложений в сообщения см. в статье [Добавление мультимедийных вложений в сообщения](bot-builder-dotnet-add-media-attachments.md).

## <a name="types-of-rich-cards"></a>Типы форматированных карточек

Сейчас Bot Framework поддерживает восемь типов форматированных карточек. 

| Тип карточки | ОПИСАНИЕ |
|----|----|
| <a href="/adaptive-cards/get-started/bots">Адаптивная карточка</a> | Настраиваемая карточка, которая может содержать любое сочетание текста, речи, изображений, кнопок и полей для ввода. См. раздел с [поддержкой для каждого канала](/adaptive-cards/get-started/bots#channel-status).  |
| [Анимационная карточка][animationCard] | Карточка, который может воспроизводить GIF-файлы с анимацией или короткие видеоролики. |
| [Карточка со звуком][audioCard] | Карточка, которая может воспроизводить звуковой файл. |
| [Карточка для имиджевого баннера][heroCard] | Карточка, которая обычно содержит одно большое изображение, одну или нескольких кнопок и текст. |
| [Карточка с эскизом][thumbnailCard] | Карточка, которая обычно содержит один эскиз, одну или несколько кнопок и текст. |
| [Карточка квитанции][receiptCard] | Карточка, с помощью которой бот выдает квитанцию пользователю. Обычно она содержит список элементов, включаемых в квитанцию, налог, а также общую информацию и другой текст. |
| [Карточка для входа][signinCard] | Карточка, позволяющая боту запрашивать вход пользователя. Обычно она содержит текст и одну или несколько кнопок, которые можно нажать для инициации процесса входа. |
| [Карточка с видео][videoCard] | Карточка, которая может воспроизводить видео. |

> [!TIP]
> Чтобы отобразить несколько форматированных карточек в формате списка, задайте свойству `AttachmentLayout` действия значение "list" (Список). Чтобы отобразить несколько форматированных карточек в режиме карусели, задайте свойству `AttachmentLayout` действия значение "carousel" (Карусель). Если канал не поддерживает формат карусели, форматированная карточка будет показана в формате списка, даже если свойству `AttachmentLayout` задано значение "carousel" (Карусель).

## <a name="process-events-within-rich-cards"></a>Обработка событий в форматированных карточках

Для обработки событий в форматированных карточках определите объекты `CardAction`, чтобы указать, какие действия должны происходить, когда пользователь нажимает кнопку или касается части карточки. Каждый объект `CardAction` содержит следующие свойства.

| Свойство | type | ОПИСАНИЕ | 
|----|----|----|
| type | строка | Тип действия (одно из значений, указанных в таблице ниже) |
| Название | строка | Название кнопки |
| Образ — | строка | URL-адрес изображения для кнопки |
| Значение | строка | Значение, необходимое для выполнения указанного типа действия |

> [!NOTE]
> Для создания кнопок в адаптивных карточках используются не объекты `CardAction`, а схема, которая определяется <a href="http://adaptivecards.io" target="_blank">адаптивными карточками</a>. Пример, демонстрирующий добавление кнопок в адаптивную карточку, см. в разделе о [добавлении адаптивной карточки в сообщение](#adaptive-card).

В таблице ниже перечисляются допустимые значения для `CardAction.Type` и описывается ожидаемое содержимое `CardAction.Value` для каждого типа.

| CardAction.Type | CardAction.Value | 
|----|----|
| openUrl | URL-адрес, который будет открыт во встроенном браузера |
| imBack | Текст сообщения для отправки боту (от пользователя, который нажал кнопку или коснулся карты). Это сообщение (от пользователя боту) все участники беседы увидят через клиентское приложение, в котором ведется беседа. |
| postBack | Текст сообщения для отправки боту (от пользователя, который нажал кнопку или коснулся карты). Некоторые клиентские приложения могут отображать этот текст в канале сообщений, где он будет виден всем участникам беседы. |
| вызывает | Место назначения телефонного звонка в следующем формате: **tel:123123123123** |
| playAudio | URL-адрес аудио для воспроизведения |
| playVideo | URL-адрес видео для воспроизведения |
| showImage | URL-адрес изображения для отображения |
| downloadFile | URL-адрес файла для скачивания |
| signin | URL-адрес потока OAuth для инициализации |

## <a name="add-a-hero-card-to-a-message"></a>Добавление имиджевой карточки в сообщение

Эта карточка обычно содержит одно большое изображение, одну или нескольких кнопок и текст. 

В примере кода ниже показано создание ответного сообщения, которое содержит три имиджевые карточки, отображаемые в формате карусели. 

[!code-csharp[Add HeroCard attachment](../includes/code/dotnet-add-attachments.cs#addHeroCardAttachment)]

## <a name="add-a-thumbnail-card-to-a-message"></a>Добавление карточки эскиза в сообщение

Эта карточка обычно содержит один эскиз, одну или несколько кнопок и текст. 

В примере кода ниже показано создание ответного сообщения, которое содержит две карточки эскиза, отображаемые в формате списка. 

[!code-csharp[Add ThumbnailCard attachment](../includes/code/dotnet-add-attachments.cs#addThumbnailCardAttachment)]

## <a name="add-a-receipt-card-to-a-message"></a>Добавление карточки квитанции в сообщение

С помощью этой карточки бот выдает квитанцию пользователю. Обычно она содержит список элементов, включаемых в квитанцию, налог, а также общую информацию и другой текст. 

В примере кода ниже показано создание ответного сообщения, которое содержит карточку квитанции. 

[!code-csharp[Add ReceiptCard attachment](../includes/code/dotnet-add-attachments.cs#addReceiptCardAttachment)]

## <a name="add-a-sign-in-card-to-a-message"></a>Добавление карточки для входа в сообщение

Эта карточка позволяет боту запрашивать вход пользователя. Обычно она содержит текст и одну или несколько кнопок, которые можно нажать для инициации процесса входа. 

В примере кода ниже показано создание ответного сообщения, которое содержит карточку для входа.

[!code-csharp[Add SignInCard attachment](../includes/code/dotnet-add-attachments.cs#addSignInCardAttachment)]

## <a id="adaptive-card"></a> Добавление адаптивной карточки в сообщение

Адаптивная карточка, которая может содержать любое сочетание текста, речи, изображений, кнопок и полей для ввода. Адаптивные карточки создаются в формате JSON, указанном на веб-сайте по <a href="http://adaptivecards.io" target="_blank">адаптивным карточкам</a>, что позволяет получить больший контроль над содержимым и форматом карточек. 

Чтобы создать адаптивную карточку с помощью .NET, установите пакет NuGet `Microsoft.AdaptiveCards`. Затем ознакомьтесь со сведениями на веб-сайте по <a href="http://adaptivecards.io" target="_blank">адаптивным карточкам</a>, чтобы получить представление о схеме адаптивных карточек, изучить элементы адаптивных карточек и просмотреть примеры JSON, которые можно использовать для создания карточек различного состава и уровня сложности. А воспользовавшись интерактивным визуализатором, вы сможете разрабатывать соответствующие полезные нагрузки и просматривать выходные данные карточки.

В примере кода ниже показано создание сообщения, содержащего адаптивную карточку для напоминания календаря. 

[!code-csharp[Add Adaptive Card attachment](../includes/code/dotnet-add-attachments.cs#addAdaptiveCardAttachment)]

Итоговая карточка содержит три блока текста, поле для ввода (список значений) и три кнопки.

![Адаптивная карточка с напоминанием календаря](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Предварительный просмотр компонентов с помощью инспектора каналов][inspector]
- <a href="http://adaptivecards.io" target="_blank">Адаптивные карточки</a>
- [Общие сведения о действиях](bot-builder-dotnet-activities.md)
- [Создание сообщений](bot-builder-dotnet-create-messages.md)
- [Добавление мультимедийных вложений в сообщения](bot-builder-dotnet-add-media-attachments.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Класс Attachment</a>

[animationCard]: /dotnet/api/microsoft.bot.connector.animationcard

[audioCard]: /dotnet/api/microsoft.bot.connector.audiocard 

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard 

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 

[videoCard]: /dotnet/api/microsoft.bot.connector.videocard

[inspector]: ../bot-service-channel-inspector.md
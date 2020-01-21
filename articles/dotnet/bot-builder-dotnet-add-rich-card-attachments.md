---
title: Добавление вложений в виде форматированных карточек в сообщения (C# версии 3) — Служба Azure Bot
description: Сведения о добавлении форматированных карточек в сообщения с помощью пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e30ac144d4e960672f3d129935a657c42ed1aa6d
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75796517"
---
# <a name="add-rich-card-attachments-to-messages"></a>Добавление вложений в виде форматированных карточек в сообщения

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

В обмене сообщениями между пользователем и ботом может быть задействована одна или несколько форматированных карточек, отображаемых в виде списка или карусели. 

Свойство `Attachments` объекта <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> содержит массив объектов <a href="https://docs.microsoft.com/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment</a>, представляющих вложения в виде форматированных карточек и файлов мультимедиа. 

> [!NOTE]
> Сведения о добавлении вложений мультимедиа в сообщения см. в статье [Добавление мультимедийных вложений в сообщения](bot-builder-dotnet-add-media-attachments.md).

## <a name="types-of-rich-cards"></a>Типы функциональных карточек

Сейчас Bot Framework поддерживает восемь типов форматированных карточек. 

| Тип карточки | Description |
|----|----|
| <a href="/adaptive-cards/get-started/bots">Адаптивная карточка</a> | Настраиваемая карточка, которая может содержать любое сочетание текста, речи, изображений, кнопок и полей для ввода. См. описание [поддержки для каждого канала](/adaptive-cards/get-started/bots#channel-status).  |
| [Анимационная карточка][animationCard] | Карточка, которая может воспроизводить GIF-файлы с анимацией или короткие видеоролики. |
| [Карточка с аудио][audioCard] | Карточка, которая может воспроизводить звуковой файл. |
| [Карточка для имиджевого баннера][heroCard] | Карточка, которая обычно содержит одно большое изображение, одну или несколько кнопок и текст. |
| [Карточка с эскизом][thumbnailCard] | Карточка, которая обычно содержит один эскиз, одну или несколько кнопок и текст. |
| [Карточка квитанции][receiptCard] | Карточка, с помощью которой бот выдает квитанцию пользователю. Обычно она содержит список элементов, включаемых в квитанцию, налог, а также общую информацию и другой текст. |
| [Карточка для входа][signinCard] | Карточка, в которой бот запрашивает вход пользователя. Обычно она содержит текст и одну или несколько кнопок, которые можно нажать, чтобы начать процесс входа. |
| [Карточка с видео][videoCard] | Карточка, которая может воспроизводить видео. |

> [!TIP]
> Чтобы отобразить несколько форматированных карточек в формате списка, задайте свойству `AttachmentLayout` действия значение "list" (Список). Чтобы отобразить несколько форматированных карточек в режиме карусели, задайте свойству `AttachmentLayout` действия значение "carousel" (Карусель). Если канал не поддерживает формат карусели, форматированные карточки будут отображены в виде списка, даже если свойству `AttachmentLayout` задано значение carousel.

## <a name="process-events-within-rich-cards"></a>Обработка событий в форматированных карточках

Для обработки событий в форматированных карточках определите объекты `CardAction`, чтобы указать, какие действия должны происходить, когда пользователь нажимает кнопку или касается части карточки. Каждый объект `CardAction` содержит следующие свойства.

| Свойство | Тип | Description | 
|----|----|----|
| Тип | строка | тип действия (одно из значений, указанных в таблице ниже) |
| Title | строка | название кнопки |
| Образ — | строка | URL-адрес изображения для кнопки |
| Значение | строка | значение, необходимое для выполнения указанного типа действия |

> [!NOTE]
> Для создания кнопок в адаптивных карточках используются не объекты `CardAction`, а схема, которая определяется <a href="http://adaptivecards.io" target="_blank">адаптивными карточками</a>. О том, как добавить кнопку в адаптивную карточку, см. в разделе [Добавление адаптивной карточки в сообщение](#adaptive-card).

В таблице ниже перечисляются допустимые значения для `CardAction.Type` и описывается ожидаемое содержимое `CardAction.Value` для каждого типа.

| CardAction.Type | CardAction.Value | 
|----|----|
| openUrl | URL-адрес, который будет открыт во встроенном браузере |
| imBack | Текст сообщения для отправки боту (от пользователя, который нажал кнопку или коснулся карты). Это сообщение (от пользователя к боту) увидят все участники общения через клиентское приложение, в котором ведется общение. |
| postBack | Текст сообщения для отправки боту (от пользователя, который нажал кнопку или коснулся карты). Некоторые клиентские приложения могут отображать этот текст на канале сообщений, где он будет виден всем участникам общения. |
| вызывает | Место назначения телефонного звонка в следующем виде: **tel:123123123123** |
| playAudio | URL-адрес аудио для воспроизведения |
| playVideo | URL-адрес видео для воспроизведения |
| showImage | URL-адрес изображения для отображения |
| downloadFile | URL-адрес файла для загрузки |
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

Эта карточка позволяет боту запрашивать вход пользователя. Обычно она содержит текст и одну или несколько кнопок, которые можно нажать, чтобы начать процесс входа. 

В примере кода ниже показано создание ответного сообщения, которое содержит карточку для входа.

[!code-csharp[Add SignInCard attachment](../includes/code/dotnet-add-attachments.cs#addSignInCardAttachment)]

## <a id="adaptive-card"></a> Добавление адаптивной карточки в сообщение

Адаптивная карточка может содержать любое сочетание текста, речи, изображений, кнопок и полей для ввода. Адаптивные карточки создаются в формате JSON (см. <a href="http://adaptivecards.io" target="_blank">здесь</a>), что позволяет получить больший контроль над содержимым и форматом карточек. 

Чтобы создать адаптивную карточку с помощью .NET, установите пакет NuGet `AdaptiveCards`. Затем ознакомьтесь со сведениями на веб-сайте по <a href="http://adaptivecards.io" target="_blank">адаптивным карточкам</a>, чтобы получить представление о схеме адаптивных карточек, изучить элементы адаптивных карточек и просмотреть примеры JSON, которые можно использовать для создания карточек различного состава и уровня сложности. А воспользовавшись интерактивным визуализатором, вы сможете разрабатывать соответствующие полезные нагрузки и просматривать выходные данные карточки.

В примере кода ниже показано, как создать сообщение, содержащее адаптивную карточку для напоминания календаря: 

[!code-csharp[Add Adaptive Card attachment](../includes/code/dotnet-add-attachments.cs#addAdaptiveCardAttachment)]

Созданная карточка содержит три блока текста, поле для ввода (список значений) и три кнопки:

![Адаптивная карточка с напоминанием календаря](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Справочник по каналам](../bot-service-channels-reference.md)
- <a href="http://adaptivecards.io" target="_blank">Адаптивные карточки</a>
- [Общие сведения о действиях](bot-builder-dotnet-activities.md)
- [Создание сообщений](bot-builder-dotnet-create-messages.md)
- [Добавление мультимедийных вложений в сообщения](bot-builder-dotnet-add-media-attachments.md)
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
- <a href="https://docs.microsoft.com/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment class</a> (Класс Attachment)

[animationCard]: /dotnet/api/microsoft.bot.connector.animationcard

[audioCard]: /dotnet/api/microsoft.bot.connector.audiocard 

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard 

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 

[videoCard]: /dotnet/api/microsoft.bot.connector.videocard

[inspector]: ../bot-service-channels-reference.md

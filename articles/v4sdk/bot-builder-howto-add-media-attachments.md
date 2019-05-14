---
title: Добавление мультимедиа в сообщения | Документы Майкрософт
description: Сведения о добавлении мультимедиа в сообщения с помощью пакета SDK Bot Framework.
keywords: media, messages, images, audio, video, files, MessageFactory, rich cards, messages, adaptive cards, hero card, suggested actions
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7093f13d1958c741b497a50535eb70a255dfcbe8
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032576"
---
# <a name="add-media-to-messages"></a>Добавление мультимедиа в сообщения

[!INCLUDE[applies-to](../includes/applies-to.md)]

Обмен сообщениями между пользователем и ботом может включать вложения мультимедиа, такие как изображения, видео, аудио и файлы. Пакет SDK Bot Framework поддерживает задачу отправки пользователю форматированного сообщения. Чтобы определить, какой тип форматированных сообщений поддерживает некоторый канал (Slack, Facebook, Скайп, и т. д.), изучите сведения об ограничениях в документации по этому каналу.

Примеры доступных карточек см. в статье [Проектирование взаимодействия с пользователем](../bot-service-design-user-experience.md).

## <a name="send-attachments"></a>Отправка вложений

Чтобы отправить пользователю содержимое, например изображение или видео, нужно добавить вложение или список вложений в сообщение.

Примеры доступных карточек см. в статье [Проектирование взаимодействия с пользователем](../bot-service-design-user-experience.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Свойство `Attachments` объекта `Activity` содержит массив объектов `Attachment`, представляющих вложения в виде форматированных карточек и файлов мультимедиа. Чтобы добавить мультимедийное вложение в сообщение, создайте объект `Attachment` для действия `reply` (которое было создано из действия с помощью `CreateReply()`) и задайте свойства `ContentType`, `ContentUrl` и `Name`.

Представленный здесь исходный код основан на примере [обработки вложений](https://aka.ms/bot-attachments-sample-code).

Чтобы создать ответное сообщение, определите текст и настройте вложения. Присвоение вложений ответному сообщению выполняется одинаково для всех типов вложений, но настройка и определение разных вложений будут отличаться, как показано в следующих фрагментах. Ниже приведен код для настройки ответа со встроенным вложением:

**Bots/AttachmentsBot.cs** [!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=108-109)]

Далее мы рассмотрим разные типы вложений. Во-первых, это встроенные вложения:

**Bots/AttachmentsBot.cs** [!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=165-176)]

Во-вторых, отправленные вложения:

**Bots/AttachmentsBot.cs** [!code-csharp[uploaded attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=179-215)]

И, в-третьих, вложения из Интернета:

**Bots/AttachmentsBot.cs** [!code-csharp[online attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=218-227)]


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Представленный здесь исходный код основан на примере [обработки вложений на JS](https://aka.ms/bot-attachments-sample-code-js).

Чтобы использовать вложения, включите в бота следующие библиотеки:

**bots/attachmentsBot.js** [!code-javascript[attachments libraries](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=4)]

Чтобы создать ответное сообщение, определите текст и настройте вложения. Присвоение вложений ответному сообщению выполняется одинаково для всех типов вложений, но настройка и определение разных вложений будут отличаться, как показано в следующих фрагментах. Ниже приведен код для настройки ответа со встроенным вложением:

**bots/attachmentsBot.js** [!code-javascript[attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=119,128-129)]

У вас есть несколько разных методов для отправки пользователю мультимедийного содержимого (например, изображения или видео). Во-первых, это встроенные вложения:

**bots/attachmentsBot.js** [!code-javascript[inline attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=170-179)]

Во-вторых, отправленные вложения:

**bots/attachmentsBot.js** [!code-javascript[uploaded attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=197-215)]

И представленные URL-адресом вложения из Интернета:

**bots/attachmentsBot.js** [!code-javascript[internet attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=184-191)]

---

Если вложение представляет собой изображение, аудиофайл или видео, служба соединителя будет передавать данные вложения каналу так, чтобы позволить [каналу](bot-builder-channeldata.md) обрабатывать это вложение в диалоге. Если вложение представляет собой файл, URL-адрес файла будет отображаться в беседе как гиперссылка.

## <a name="send-a-hero-card"></a>Отправка карточки для имиджевого баннера

Помимо изображений или видео, вы можете прикрепить **карточку для имиджевого баннера**, которая позволяет совмещать изображения и кнопки в один объект и отправлять их в таком виде пользователю. Markdown поддерживается для большинства текстовых полей, но особенности поддержки зависят от канала.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы составить сообщение с карточкой имиджевого баннера и кнопкой, вложите `HeroCard` в сообщение. 

Представленный здесь исходный код основан на примере [обработки вложений](https://aka.ms/bot-attachments-sample-code).

**Bots/AttachmentsBot.cs** [!code-csharp[Hero card](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=39-62)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы составить сообщение с карточкой имиджевого баннера и кнопкой, вложите `HeroCard` в сообщение. 

Представленный здесь исходный код основан на примере [обработки вложений на JS](https://aka.ms/bot-attachments-sample-code-js).

**bots/attachmentsBot.js** [!code-javascript[hero card](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=148-164)]

---

## <a name="process-events-within-rich-cards"></a>Обработка событий в форматированных карточках

Для обработки событий в форматированных карточках используйте объекты _действий карточек_, чтобы указать, какие действия должны происходить, когда пользователь нажимает кнопку или щелкает сегмент карточки. Каждое действие карточки имеет определенный _тип_ и _значение_.

Во избежание ошибок следует назначить тип действия для каждого активного элемента карточки. В этой таблице перечислены и описаны доступные типы действий и требуемый формат для связанного свойства.

| type | ОПИСАНИЕ | Значение |
| :---- | :---- | :---- |
| openUrl | Открывает URL-адрес в окне встроенного браузера. | URL-адрес, который нужно открыть. |
| imBack | Отправляет боту сообщение и отображает полученный ответ в чате. | Текст отправляемого сообщения. |
| postBack | Отправляет боту сообщение, но не всегда отображает полученный ответ в чате. | Текст отправляемого сообщения. |
| вызывает | Инициирует телефонный звонок. | Целевое назначение телефонного звонка в следующем формате: `tel:123123123123`. |
| playAudio | Воспроизводит звук. | URL-адрес для воспроизведения звука. |
| playVideo | Воспроизводит видео. | URL-адрес для воспроизведения видео. |
| showImage | Отображает изображение. | URL-адрес для отображения изображения. |
| downloadFile | Скачивает файл. | URL-адрес для скачивания файла. |
| signin | Инициирует процесс входа OAuth. | URL-адрес потока OAuth, который нужно запустить. |

## <a name="hero-card-using-various-event-types"></a>Карточка для имиджевого баннера с различными типами событий

В следующем коде показаны примеры использования различных событий форматированных карточек.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Примеры для всех доступных типов карточек представлены [в этом примере на C#](https://aka.ms/bot-cards-sample-code).

**Cards.cs** [!code-csharp[hero cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=27-40)]

**Cards.cs** [!code-csharp[cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=91-100)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Примеры для всех доступных типов карточек представлены [в этом примере на JavaScript](https://aka.ms/bot-cards-js-sample-code).

**dialogs/mainDialog.js** [!code-javascript[hero cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=213-225)]

**dialogs/mainDialog.js** [!code-javascript[sign in cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=266-272)]

---

## <a name="send-an-adaptive-card"></a>Отправка адаптивной карточки
Адаптивные карточки и MessageFactory используются для отправки форматированных сообщений, включая тексты, изображения, видео, аудио и файлы для взаимодействия с пользователями. Но между ними существуют некоторые отличия. 

Во-первых, только некоторые каналы поддерживают адаптивные карточки и среди таких каналов есть те, которые могут поддерживать их частично. Например, при отправке адаптивной карточки в Facebook, кнопки не будут работать, а тексты и изображения будут отображаться. MessageFactory — это просто вспомогательный класс в пакете SDK Bot Framework. Он позволяет автоматизировать действия по созданию и поддерживается большинством каналов. 

Во-вторых, адаптивная карточка отправляет сообщения в формате карточки и канал определяет макет карточки. Формат сообщений, предоставляемый MessageFactory, зависит от канала, и это необязательно формат карточек, если только адаптивная карточка не является частью вложения. 

Последние сведения о поддержке каналов адаптивных карточек см. на странице <a href="http://adaptivecards.io/designer/">конструктора адаптивных карточек</a>.

> [!NOTE]
> Вы должны протестировать эту функцию, выбрав каналы, которые будут использоваться ботом, чтобы определить, поддерживают ли они адаптивные карточки.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы использовать адаптивные карточки, обязательно добавьте пакет NuGet `AdaptiveCards`.

Представленный здесь исходный код основан на примере [использования карточек](https://aka.ms/bot-cards-sample-code):

**Cards.cs** [!code-csharp[adaptive cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=13-25)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы использовать адаптивные карточки, обязательно добавьте пакет npm `adaptivecards`.

Представленный здесь исходный код основан на примере [использования карточек в JavaScript](https://aka.ms/bot-cards-js-sample-code): 

Здесь демонстрируется хранение адаптивных карточек в отдельном файле и их включение в бота:

**resources/adaptiveCard.json** [!code-json[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/resources/adaptiveCard.json)]

Здесь карточка создается с помощью CardFactory:

**dialogs/mainDialog.js** [!code-javascript[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=177-179)]

---

## <a name="send-a-carousel-of-cards"></a>Отправка карусели карточек

Сообщения также могут включать несколько вложений в макете карусели, где вложения помещаются одно за другим и пользователь может их прокручивать.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Представленный здесь исходный код основан на [примере карточек](https://aka.ms/bot-cards-sample-code):

Сначала создайте ответ и определите вложения в виде списка.

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=61-66)]

Затем добавьте вложения. Здесь мы добавляем их по одному, но вы можете управлять этим списком и добавлять в него карточки любым удобным методом.

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=105-113)]

Завершив добавление вложений, вы можете отправить этот ответ так же, как и любой другой.

**Dialogs/MainDialog.cs** [!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=117-118)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Представленный здесь исходный код основан на [примере карточек на JS](https://aka.ms/bot-cards-js-sample-code):

Чтобы отправить карусель карточек, создайте ответ с вложениями в виде массива и типом макета `Carousel`:

**dialogs/mainDialog.js** [!code-javascript[carousel of cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=104-116)]

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>Дополнительные ресурсы

Примеры доступных карточек см. в статье [Проектирование взаимодействия с пользователем](../bot-service-design-user-experience.md).

См. дополнительные сведения о [схеме карточек Bot Framework](https://aka.ms/botSpecs-cardSchema) и [действиях в беседах](https://aka.ms/botSpecs-activitySchema#message-activity).

| Пример кода | C# | JS |
| :------ | :----- | :---|
| Карточки | [Пример на языке C#](https://aka.ms/bot-cards-sample-code) | [Пример на языке JavaScript](https://aka.ms/bot-cards-js-sample-code) |
| Вложения | [Пример на языке C#](https://aka.ms/bot-attachments-sample-code) | [Пример на языке JavaScript](https://aka.ms/bot-attachments-sample-code-js) |
| Предлагаемые действия | [Пример на языке C#](https://aka.ms/SuggestedActionsCSharp) | [Пример на языке JavaScript](https://aka.ms/SuggestedActionsJS) |

Изучите репозиторий образцов для Bot Builder на [GitHub](https://aka.ms/bot-samples-readme), где есть дополнительные примеры.

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Добавление кнопок для управления действиями пользователя](./bot-builder-howto-add-suggested-actions.md)

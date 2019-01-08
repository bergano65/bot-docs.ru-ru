---
title: Добавление мультимедиа в сообщения | Документы Майкрософт
description: Сведения о добавлении мультимедиа в сообщения с помощью пакета SDK построителя ботов.
keywords: media, messages, images, audio, video, files, MessageFactory, rich cards, messages, adaptive cards, hero card, suggested actions
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/17/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fd908335c69aab7c8b68925b8ecdece79e89ab4b
ms.sourcegitcommit: f7a8f05fc05ff4a7212a437d540485bf68831604
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/21/2018
ms.locfileid: "53735964"
---
# <a name="add-media-to-messages"></a>Добавление мультимедиа в сообщения

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Обмен сообщениями между пользователем и ботом может включать вложения мультимедиа, такие как изображения, видео, аудио и файлы. Пакет SDK для Bot Builder поддерживает задачу отправки пользователю форматированного сообщения. Чтобы определить, какой тип форматированных сообщений поддерживает некоторый канал (Slack, Facebook, Скайп, и т. д.), изучите сведения об ограничениях в документации по этому каналу. Список доступных карточек вы найдете в статье [о проектировании взаимодействия с пользователем](../bot-service-design-user-experience.md). 

## <a name="send-attachments"></a>Отправка вложений

Чтобы отправить пользователю содержимое, например изображение или видео, нужно добавить вложение или список вложений в сообщение.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Свойство `Attachments` объекта `Activity` содержит массив объектов `Attachment`, представляющих вложения в виде форматированных карточек и файлов мультимедиа. Чтобы добавить мультимедийное вложение в сообщение, создайте объект `Attachment` для действия `message` и задайте свойства `ContentType`, `ContentUrl` и `Name`. Свойство `Attachments` объекта `Activity` содержит массив объектов `Attachment`, представляющих вложения в виде форматированных карточек и файлов мультимедиа. Чтобы добавить мультимедийное вложение в сообщение, с помощью метода `Attachment` создайте объект `Attachment` для действия `message` и задайте свойства `ContentType`, `ContentUrl` и `Name`. Представленный здесь исходный код основан на примере [обработки вложений](https://aka.ms/bot-attachments-sample-code). 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create an attachment.
var attachment = new Attachment
    {
        ContentUrl = "imageUrl.png",
        ContentType = "image/png",
        Name = "imageName",
    };

// Add the attachment to our reply.
reply.Attachments = new List<Attachment>() { attachment };

// Send the activity to the user.
await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Представленный здесь исходный код основан на примере [обработки вложений на JS](https://aka.ms/bot-attachments-sample-code-js).
Для отправки содержимого (например, изображения или видео) пользователю можно отправить данные в составе URL-адреса:

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');

// Call function to get an attachment.
const reply = { type: ActivityTypes.Message };
reply.attachments = [this.getInternetAttachment()];
reply.text = 'This is an internet attachment.';
// Send the activity to the user.
await turnContext.sendActivity(reply);

/* function getInternetAttachment - Returns an attachment to be sent to the user from a HTTPS URL */
getInternetAttachment() {
        return {
            name: 'imageName.png',
            contentType: 'image/png',
            contentUrl: 'imageUrl.png'}
}
```

---

Если вложение представляет собой изображение, аудиофайл или видео, служба соединителя будет передавать данные вложения каналу так, чтобы позволить [каналу](bot-builder-channeldata.md) обрабатывать это вложение в диалоге. Если вложение представляет собой файл, URL-адрес файла будет отображаться в беседе как гиперссылка.

## <a name="send-a-hero-card"></a>Отправка карточки для имиджевого баннера

Помимо изображений или видео, вы можете прикрепить **карточку для имиджевого баннера**, которая позволяет совмещать изображения и кнопки в один объект и отправлять их в таком виде пользователю.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы составить сообщение с карточкой имиджевого баннера и кнопкой, вложите `HeroCard` в сообщение. Представленный здесь исходный код основан на примере [обработки вложений](https://aka.ms/bot-attachments-sample-code). 

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

// Create a HeroCard with options for the user to choose to interact with the bot.
var card = new HeroCard
{
    Text = "You can upload an image or select one of the following choices",
    Buttons = new List<CardAction>()
    {
        new CardAction(ActionTypes.ImBack, title: "1. Inline Attachment", value: "1"),
        new CardAction(ActionTypes.ImBack, title: "2. Internet Attachment", value: "2"),
        new CardAction(ActionTypes.ImBack, title: "3. Uploaded Attachment", value: "3"),
    },
};

// Add the card to our reply.
reply.Attachments = new List<Attachment>() { card.ToAttachment() };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы составить сообщение с карточкой имиджевого баннера и кнопкой, вложите `HeroCard` в сообщение. Представленный здесь исходный код основан на примере [обработки вложений на JS](https://aka.ms/bot-attachments-sample-code-js):

```javascript
const { ActionTypes, ActivityTypes, CardFactory } = require('botbuilder');
// build buttons to display.
const buttons = [
            { type: ActionTypes.ImBack, title: '1. Inline Attachment', value: '1' },
            { type: ActionTypes.ImBack, title: '2. Internet Attachment', value: '2' },
            { type: ActionTypes.ImBack, title: '3. Uploaded Attachment', value: '3' }
];

// construct hero card.
const card = CardFactory.heroCard('', undefined,
buttons, { text: 'You can upload an image or select one of the following choices.' });

// add card to Activity.
const reply = { type: ActivityTypes.Message };
reply.attachments = [card];

// Send hero card to the user.
await turnContext.sendActivity(reply);
```

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

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

var reply = turnContext.Activity.CreateReply();

var card = new HeroCard
{
    Buttons = new List<CardAction>()
    {
        new CardAction(title: "Much Quieter", type: ActionTypes.PostBack, value: "Shh! My Bot friend hears me."),
        new CardAction(ActionTypes.OpenUrl, title: "Azure Bot Service", value: "https://azure.microsoft.com/en-us/services/bot-service/"),
    },
};

```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
const {ActionTypes} = require("botbuilder");

const hero = MessageFactory.attachment(
    CardFactory.heroCard(
        'Holler Back Buttons',
        ['https://example.com/whiteShirt.jpg'],
        [{
            type: ActionTypes.ImBack,
            title: 'ImBack',
            value: 'You can ALL hear me! Shout Out Loud'
        },
        {
            type: ActionTypes.PostBack,
            title: 'PostBack',
            value: 'Shh! My Bot friend hears me. Much Quieter'
        },
        {
            type: ActionTypes.OpenUrl,
            title: 'OpenUrl',
            value: 'https://en.wikipedia.org/wiki/{cardContent.Key}'
        }]
    )
);

await context.sendActivity(hero);

```

---

## <a name="send-an-adaptive-card"></a>Отправка адаптивной карточки
Адаптивные карточки и MessageFactory используются для отправки форматированных сообщений, включая тексты, изображения, видео, аудио и файлы для взаимодействия с пользователями. Но между ними существуют некоторые отличия. 

Во-первых, только некоторые каналы поддерживают адаптивные карточки и среди таких каналов есть те, которые могут поддерживать их частично. Например, при отправке адаптивной карточки в Facebook, кнопки не будут работать, а тексты и изображения будут отображаться. MessageFactory — это только вспомогательный класс в пакете SDK для Bot Builder. Этот класс позволяет автоматизировать действия по созданию и поддерживается большинством каналов. 

Во-вторых, адаптивная карточка отправляет сообщения в формате карточки и канал определяет макет карточки. Формат сообщений, предоставляемый MessageFactory, зависит от канала, и это необязательно формат карточек, если только адаптивная карточка не является частью вложения. 

Последние сведения о поддержке каналами адаптивных карточек см. в разделе, посвященном <a href="http://adaptivecards.io/visualizer/">визуализатору адаптивных карточек</a>.

Чтобы использовать адаптивные карточки, не забудьте добавить пакет NuGet `Microsoft.AdaptiveCards`. 


> [!NOTE]
> Вы должны протестировать эту функцию, выбрав каналы, которые будут использоваться ботом, чтобы определить, поддерживают ли они адаптивные карточки.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Представленный здесь исходный код основан на примере [использования адаптивных карточек](https://aka.ms/bot-adaptive-cards-sample-code):

```csharp
using AdaptiveCards;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Creates an attachment that contains an adaptive card
// filePath is the path to JSON file
private static Attachment CreateAdaptiveCardAttachment(string filePath)
{
    var adaptiveCardJson = File.ReadAllText(filePath);
    var adaptiveCardAttachment = new Attachment()
    {
        ContentType = "application/vnd.microsoft.card.adaptive",
        Content = JsonConvert.DeserializeObject(adaptiveCardJson),
    };
    return adaptiveCardAttachment;
}

// Create adaptive card and attach it to the message 
var cardAttachment = CreateAdaptiveCardAttachment(adaptiveCardJsonFilePath);
var reply = turnContext.Activity.CreateReply();
reply.Attachments = new List<Attachment>() { cardAttachment };

await turnContext.SendActivityAsync(reply, cancellationToken);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Представленный здесь исходный код основан на примере [использования адаптивных карточек на JS](https://aka.ms/bot-adaptive-cards-js-sample-code):

```javascript
const { BotFrameworkAdapter } = require('botbuilder');

// Import AdaptiveCard content.
const FlightItineraryCard = require('./resources/FlightItineraryCard.json');
const ImageGalleryCard = require('./resources/ImageGalleryCard.json');
const LargeWeatherCard = require('./resources/LargeWeatherCard.json');
const RestaurantCard = require('./resources/RestaurantCard.json');
const SolitaireCard = require('./resources/SolitaireCard.json');

// Create array of AdaptiveCard content, this will be used to send a random card to the user.
const CARDS = [
    FlightItineraryCard,
    ImageGalleryCard,
    LargeWeatherCard,
    RestaurantCard,
    SolitaireCard
];
// Select a random card to send.
const randomlySelectedCard = CARDS[Math.floor((Math.random() * CARDS.length - 1) + 1)];
// Send adaptive card.
await context.sendActivity({
      text: 'Here is an Adaptive Card:',
       attachments: [CardFactory.adaptiveCard(randomlySelectedCard)]
});
```

---

## <a name="send-a-carousel-of-cards"></a>Отправка карусели карточек

Сообщения также могут включать несколько вложений в макете карусели, где вложения помещаются одно за другим и пользователь может их прокручивать.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

// Create the activity and attach a set of Hero cards.
var activity = MessageFactory.Carousel(
    new Attachment[]
    {
        new HeroCard(
            title: "title1",
            images: new CardImage[] { new CardImage(url: "imageUrl1.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button1", type: ActionTypes.ImBack, value: "item1")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title2",
            images: new CardImage[] { new CardImage(url: "imageUrl2.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button2", type: ActionTypes.ImBack, value: "item2")
            })
        .ToAttachment(),
        new HeroCard(
            title: "title3",
            images: new CardImage[] { new CardImage(url: "imageUrl3.png") },
            buttons: new CardAction[]
            {
                new CardAction(title: "button3", type: ActionTypes.ImBack, value: "item3")
            })
        .ToAttachment()
    });

// Send the activity as a reply to the user.
await context.SendActivity(activity);
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// require MessageFactory and CardFactory from botbuilder.
const {MessageFactory, CardFactory} = require('botbuilder');

//  init message object
let messageWithCarouselOfCards = MessageFactory.carousel([
    CardFactory.heroCard('title1', ['imageUrl1'], ['button1']),
    CardFactory.heroCard('title2', ['imageUrl2'], ['button2']),
    CardFactory.heroCard('title3', ['imageUrl3'], ['button3'])
]);

await context.sendActivity(messageWithCarouselOfCards);
```

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>Дополнительные ресурсы

Ознакомьтесь с [принципами использования карточек в Bot Framework](https://aka.ms/botSpecs-cardSchema).

Примеры исходного кода для карточек: [C#](https://aka.ms/bot-cards-sample-code)/[JS](https://aka.ms/bot-cards-js-sample-code), для адаптивных карточек: [C#](https://aka.ms/bot-adaptive-cards-sample-code)/[JS](https://aka.ms/bot-adaptive-cards-js-sample-code), для вложений: [C#](https://aka.ms/bot-attachments-sample-code)/[JS](https://aka.ms/bot-attachments-sample-code-js), для предлагаемых действий: [C#](https://aka.ms/SuggestedActionsCSharp)/[JS](https://aka.ms/SuggestedActionsJS).
Изучите репозиторий образцов для Bot Builder на [GitHub](https://aka.ms/bot-samples-readme), где есть дополнительные примеры.

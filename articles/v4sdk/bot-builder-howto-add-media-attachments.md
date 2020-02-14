---
title: Добавление мультимедиа в сообщения — Служба Azure Bot
description: Сведения о добавлении мультимедиа в сообщения с помощью пакета SDK Bot Framework.
keywords: media, messages, images, audio, video, files, MessageFactory, rich cards, messages, adaptive cards, hero card, suggested actions
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/03/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 56d9b0485cb6e9073cb577d8494c18ab42cd39af
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071825"
---
# <a name="add-media-to-messages"></a>Добавление мультимедиа в сообщения

[!INCLUDE[applies-to](../includes/applies-to.md)]

Обмен сообщениями между пользователем и ботом может включать вложения мультимедиа, такие как изображения, видео, аудио и файлы. Пакет SDK Bot Framework поддерживает задачу отправки пользователю форматированного сообщения. Чтобы определить, какой тип форматированных сообщений поддерживает некоторый канал (Slack, Facebook, Скайп, и т. д.), изучите сведения об ограничениях в документации по этому каналу.

## <a name="prerequisites"></a>Предварительные требования

- Базовые знания о [ботах](bot-builder-basics.md).
- Код в этой статье основан на следующих примерах:

  | Образец кода | C# | JS | Python |
  | :------ | :----- | :---| :---|
  | Карточки | [Пример на языке C#](https://aka.ms/bot-cards-sample-code) | [Пример на языке JavaScript](https://aka.ms/bot-cards-js-sample-code) |[Пример для Python](https://aka.ms/bot-cards-python-sample-code) |
  | Вложения | [Пример на языке C#](https://aka.ms/bot-attachments-sample-code) | [Пример на языке JavaScript](https://aka.ms/bot-attachments-sample-code-js) | [Пример для Python](https://aka.ms/bot-media-attachments-python-sample-code) |
  | Предлагаемые действия | [Пример на языке C#](https://aka.ms/SuggestedActionsCSharp) | [Пример на языке JavaScript](https://aka.ms/SuggestedActionsJS) | [Пример для Python](https://aka.ms/SuggestedActionsPython) |

## <a name="send-attachments"></a>Отправка вложений

Чтобы отправить пользователю содержимое, например изображение или видео, нужно добавить вложение или список вложений в сообщение.

Примеры доступных карточек см. в статье [Проектирование взаимодействия с пользователем](../bot-service-design-user-experience.md).

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Свойство `Attachments` объекта `Activity` содержит массив объектов `Attachment`, представляющих вложения в виде форматированных карточек и файлов мультимедиа. Чтобы добавить мультимедийное вложение в сообщение, создайте объект `Attachment` для действия `reply` (которое было создано из действия с помощью `CreateReply()`) и задайте свойства `ContentType`, `ContentUrl` и `Name`.

Представленный здесь исходный код основан на примере [обработки вложений](https://aka.ms/bot-attachments-sample-code).

Чтобы создать ответное сообщение, определите текст и настройте вложения. Присвоение вложений ответному сообщению выполняется одинаково для всех типов вложений, но настройка и определение разных вложений будут отличаться, как показано в следующих фрагментах. Ниже приведен код для настройки ответа со встроенным вложением:

**Bots/AttachmentsBot.cs**  
[!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=105-106)]

Далее мы рассмотрим разные типы вложений. Во-первых, это встроенные вложения:

**Bots/AttachmentsBot.cs**  
[!code-csharp[inline attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=167-178)]

Во-вторых, отправленные вложения:

**Bots/AttachmentsBot.cs**  
[!code-csharp[uploaded attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=181-214)]

И, в-третьих, вложения из Интернета:

**Bots/AttachmentsBot.cs**  
[!code-csharp[online attachment](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=217-226)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Представленный здесь исходный код основан на примере [обработки вложений на JS](https://aka.ms/bot-attachments-sample-code-js).

Чтобы использовать вложения, включите в бота следующие библиотеки:

**bots/attachmentsBot.js**  
[!code-javascript[attachments libraries](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=4)]

Чтобы создать ответное сообщение, определите текст и настройте вложения. Присвоение вложений ответному сообщению выполняется одинаково для всех типов вложений, но настройка и определение разных вложений будут отличаться, как показано в следующих фрагментах. Ниже приведен код для настройки ответа со встроенным вложением:

**bots/attachmentsBot.js**  
[!code-javascript[attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=119,128-129)]

У вас есть несколько разных методов для отправки пользователю мультимедийного содержимого (например, изображения или видео). Во-первых, это встроенные вложения:

**bots/attachmentsBot.js**  
[!code-javascript[inline attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=170-179)]

Во-вторых, отправленные вложения:

**bots/attachmentsBot.js**  
[!code-javascript[uploaded attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=197-215)]

И представленные URL-адресом вложения из Интернета:

**bots/attachmentsBot.js**  
[!code-javascript[internet attachments](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=184-191)]

### <a name="pythontabpython"></a>[Python](#tab/python)

Чтобы создать ответное сообщение, определите текст и настройте вложения. Присвоение вложений ответному сообщению выполняется одинаково для всех типов вложений, но настройка и определение разных вложений будут отличаться, как показано в следующих фрагментах.

Представленный здесь исходный код основан на примере [обработки вложений](https://aka.ms/bot-media-attachments-python-sample-code).

Ниже приведен код для настройки ответа со встроенным вложением:

**bots/attachments_bot.py**  
[!code-python[attachments](~/../botbuilder-python/samples/python/15.handling-attachments/bots/attachments_bot.py?range=112-113)]

У вас есть несколько разных методов для отправки пользователю мультимедийного содержимого (например, изображения или видео). Во-первых, это встроенные вложения:

**bots/attachments_bot.py** [!code-python[inline attachments](~/../botbuilder-python/samples/python/15.handling-attachments/bots/attachments_bot.py?range=153-170)]

Во-вторых, отправленные вложения:

**bots/attachments_bot.py** [!code-python[upload attachments](~/../botbuilder-python/samples/python/15.handling-attachments/bots/attachments_bot.py?range=172-207)]

И представленные URL-адресом вложения из Интернета:

**bots/attachments_bot.py** [!code-python[internet attachments](~/../botbuilder-python/samples/python/15.handling-attachments/bots/attachments_bot.py?range=209-218)]

---

Если вложение представляет собой изображение, аудиофайл или видео, служба соединителя будет передавать данные вложения каналу так, чтобы позволить [каналу](bot-builder-channeldata.md) обрабатывать это вложение в диалоге. Если вложение представляет собой файл, URL-адрес файла будет отображаться в диалоге как гиперссылка.

## <a name="send-a-hero-card"></a>Отправка карточки для имиджевого баннера

Помимо изображений или видео, вы можете прикрепить **карточку для имиджевого баннера**, которая позволяет совмещать изображения и кнопки в один объект и отправлять их в таком виде пользователю. Markdown поддерживается для большинства текстовых полей, но особенности поддержки зависят от канала.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы составить сообщение с карточкой имиджевого баннера и кнопкой, вложите `HeroCard` в сообщение.

Представленный здесь исходный код основан на примере [обработки вложений](https://aka.ms/bot-attachments-sample-code).

**Bots/AttachmentsBot.cs**  
[!code-csharp[Hero card](~/../botbuilder-samples/samples/csharp_dotnetcore/15.handling-attachments/Bots/AttachmentsBot.cs?range=39-58)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы составить сообщение с карточкой имиджевого баннера и кнопкой, вложите `HeroCard` в сообщение.

Представленный здесь исходный код основан на примере [обработки вложений на JS](https://aka.ms/bot-attachments-sample-code-js).

**bots/attachmentsBot.js**  
[!code-javascript[hero card](~/../botbuilder-samples/samples/javascript_nodejs/15.handling-attachments/bots/attachmentsBot.js?range=147-165)]

### <a name="pythontabpython"></a>[Python](#tab/python)

Чтобы составить сообщение с карточкой имиджевого баннера и кнопкой, вложите `HeroCard` в сообщение.

Представленный здесь исходный код основан на примере [обработки вложений](https://aka.ms/bot-media-attachments-python-sample-code).

**bots/attachments_bot.py** [!code-python[hero card](~/../botbuilder-python/samples/python/15.handling-attachments/bots/attachments_bot.py?range=125-148)]

---

## <a name="process-events-within-rich-cards"></a>Обработка событий в форматированных карточках

Для обработки событий в форматированных карточках используйте объекты _действий карточек_, чтобы указать, какие действия должны происходить, когда пользователь нажимает кнопку или щелкает сегмент карточки. Каждое действие карточки имеет определенный _тип_ и _значение_.

Во избежание ошибок следует назначить тип действия для каждого активного элемента карточки. В этой таблице перечислены и описаны доступные типы действий и требуемый формат для связанного свойства.

| Тип | Описание | Значение |
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

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Примеры для всех доступных типов карточек представлены [в этом примере на C#](https://aka.ms/bot-cards-sample-code).

**Cards.cs**  
[!code-csharp[hero cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=27-40)]

**Cards.cs**  
[!code-csharp[cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=91-100)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Примеры для всех доступных типов карточек представлены [в этом примере на JavaScript](https://aka.ms/bot-cards-js-sample-code).

**dialogs/mainDialog.js**  
[!code-javascript[hero cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=206-218)]

**dialogs/mainDialog.js**  
[!code-javascript[sign in cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=259-265)]

### <a name="pythontabpython"></a>[Python](#tab/python)

Примеры всех доступных типов карт представлены в [этом примере для Python](https://aka.ms/bot-cards-python-sample-code).

**dialogs/main_dialog.py** [!code-python[hero cards](~/../botbuilder-samples/samples/python/06.using-cards/dialogs/main_dialog.py?range=163-179)]

**dialogs/main_dialog.py** [!code-python[hero cards](~/../botbuilder-samples/samples/python/06.using-cards/dialogs/main_dialog.py?range=245-256)]

---

## <a name="send-an-adaptive-card"></a>Отправка адаптивной карточки
Адаптивные карточки и MessageFactory используются для отправки форматированных сообщений, включая тексты, изображения, видео, аудио и файлы для взаимодействия с пользователями. Но между ними существуют некоторые отличия.

Во-первых, только некоторые каналы поддерживают адаптивные карточки и среди таких каналов есть те, которые могут поддерживать их частично. Например, при отправке адаптивной карточки в Facebook, кнопки не будут работать, а тексты и изображения будут отображаться. MessageFactory — это просто вспомогательный класс в пакете SDK Bot Framework. Он позволяет автоматизировать действия по созданию и поддерживается большинством каналов.

Во-вторых, адаптивная карточка отправляет сообщения в формате карточки и канал определяет макет карточки. Формат сообщений, предоставляемый MessageFactory, зависит от канала, и это необязательно формат карточек, если только адаптивная карточка не является частью вложения.

Последние сведения о поддержке каналов адаптивных карточек см. на странице <a href="http://adaptivecards.io/designer/">конструктора адаптивных карточек</a>.

Чтобы использовать адаптивные карточки, не забудьте добавить пакет NuGet `AdaptiveCards`.

> [!NOTE]
> Вы должны протестировать эту функцию, выбрав каналы, которые будут использоваться ботом, чтобы определить, поддерживают ли они адаптивные карточки.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы использовать адаптивные карточки, обязательно добавьте пакет NuGet `AdaptiveCards`.

Представленный здесь исходный код основан на примере [использования карточек](https://aka.ms/bot-cards-sample-code).

**Cards.cs**  
[!code-csharp[adaptive cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Cards.cs?range=13-25)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы использовать адаптивные карточки, обязательно добавьте пакет npm `adaptivecards`.

Представленный здесь исходный код основан на примере [использования карточек в JavaScript](https://aka.ms/bot-cards-js-sample-code): 

Здесь демонстрируется хранение адаптивных карточек в отдельном файле и их включение в бота:

**resources/adaptiveCard.json**  
[!code-json[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/resources/adaptiveCard.json)]

Карта создается следующим образом:

**dialogs/mainDialog.js** [!code-javascript[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=6)]
[!code-javascript[adaptive cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=170-172)]

### <a name="pythontabpython"></a>[Python](#tab/python)

Представленный здесь исходный код основан на примере [использования карточек](https://aka.ms/bot-cards-python-sample-code).

**dialogs/resources/adaptive_card_example.py** [!code-python[adaptive cards](~/../botbuilder-python/samples/python/06.using-cards/dialogs/resources/adaptive_card_example.py)]

Карта создается следующим образом:

**bots/main_dialog.py** [!code-python[hero cards](~/../botbuilder-samples/samples/python/06.using-cards/dialogs/main_dialog.py?range=127-128)]

---

## <a name="send-a-carousel-of-cards"></a>Отправка карусели карточек

Сообщения также могут включать несколько вложений в макете карусели, где вложения помещаются одно за другим и пользователь может их прокручивать.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Представленный здесь исходный код основан на [примере карточек](https://aka.ms/bot-cards-sample-code):

Сначала создайте ответ и определите вложения в виде списка.

**Dialogs/MainDialog.cs**  
[!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=61-66)]

Затем добавьте вложения. Здесь мы добавляем их по одному, но вы можете управлять этим списком и добавлять в него карточки любым удобным методом.

**Dialogs/MainDialog.cs**  
[!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=104-113)]

Завершив добавление вложений, вы можете отправить этот ответ так же, как и любой другой.

**Dialogs/MainDialog.cs**  
[!code-csharp[carousel of cards](~/../botbuilder-samples/samples/csharp_dotnetcore/06.using-cards/Dialogs/MainDialog.cs?range=117-118)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Представленный здесь исходный код основан на [примере карточек на JS](https://aka.ms/bot-cards-js-sample-code):

Чтобы отправить карусель карточек, создайте ответ с вложениями в виде массива и типом макета `Carousel`:

**dialogs/mainDialog.js**  
[!code-javascript[carousel of cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=97-108)]

[!code-javascript[carousel of cards](~/../botbuilder-samples/samples/javascript_nodejs/06.using-cards/dialogs/mainDialog.js?range=113-116)]

### <a name="pythontabpython"></a>[Python](#tab/python)

Представленный здесь исходный код основан на [примере карт для Python](https://aka.ms/bot-cards-python-sample-code).

Чтобы отправить карусель карточек, создайте ответ с вложениями в виде массива и типом макета `Carousel`:

**dialogs/main_dialog.py** [!code-python[hero cards](~/../botbuilder-samples/samples/python/06.using-cards/dialogs/main_dialog.py?range=104-112)]

Завершив добавление вложений, вы можете отправить этот ответ.

**dialogs/main_dialog.py** [!code-python[hero cards](~/../botbuilder-samples/samples/python/06.using-cards/dialogs/main_dialog.py?range=114-115)]

---

<!-- TODO: Add a media card, such as video or audion. Revisit which examples we put here and link to the 06 through 08 samples. -->

## <a name="additional-resources"></a>Дополнительные ресурсы

Примеры доступных карточек см. в статье [Проектирование взаимодействия с пользователем](../bot-service-design-user-experience.md).

См. дополнительные сведения о [схеме карточек Bot Framework](https://aka.ms/botSpecs-cardSchema) и [действиях в беседах](https://aka.ms/botSpecs-activitySchema#message-activity).

### <a name="code-sample-for-processing-adaptive-card-input"></a>Пример кода для обработки входных данных адаптивной карточки

В этом примере кода показан один из способов использования входных данных адаптивной карточки в пределах класса диалога бота.
Он расширяет текущий пример 06.using-cards, включая проверку входных данных, полученных в текстовом поле отвечающего клиента.
Сначала мы добавили функцию ввода текста и кнопку к имеющейся адаптивной карточке, добавив следующий код перед последней скобкой adaptiveCard.json в папке ресурсов:

```json
...
  "actions": [
    {
      "type": "Action.ShowCard",
      "title": "Text",
      "card": {
      "type": "AdaptiveCard",
      "body": [
        {
          "type": "Input.Text",
          "id": "text",
          "isMultiline": true,
          "placeholder": "Enter your comment"
        }
      ],
      "actions": [
        {
          "type": "Action.Submit",
          "title": "OK"
        }
      ]
    }
  }
]

```

Обратите внимание, что поле ввода называется "text", поэтому наша адаптивная карточка присоединит данные текста комментария как Value.[text.]

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

Наш проверяющий элемент управления использует Newtonsoft.json, чтобы сначала преобразовать его в JObject, а затем создать обрезанную текстовую строку для сравнения. Поэтому добавьте:

```csharp
using System;
using System.Linq;
using Newtonsoft.Json.Linq;
```

в MainDialog.cs и установите последнюю стабильную версию пакета nuget из Newtonsoft.Json.
В коде проверяющего элемента управления мы добавили поток логики в комментариях в коде.
Этот код ChoiceValidator() помещается в пример 06.using-cards, открытом для объявления MainDialog, сразу после закрытой фигурной скобки:

```csharp
private async Task ChoiceValidator(
    PromptValidatorContext promptContext,
    CancellationToken cancellationToken)
{
    // Retrieves Adaptive Card comment text as JObject.
    // looks for JObject field "text" and converts that input into a trimmed text string.
    var jobject = promptContext.Context.Activity.Value as JObject;
    var jtoken = jobject?["text"];
    var text = jtoken?.Value().Trim();

    // Logic: 1. if succeeded = true, just return promptContext
    //        2. if false, see if JObject contained Adaptive Card input.
    //               No = (bad input) return promptContext
    //               Yes = update Value field with JObject text string, return "true".
    if (!promptContext.Recognized.Succeeded && text != null)
    {
        var choice = promptContext.Options.Choices.FirstOrDefault(
        c => c.Value.Equals(text, StringComparison.InvariantCultureIgnoreCase));
        if (choice != null)
        {
            promptContext.Recognized.Value = new FoundChoice
            {
                Value = choice.Value,
            };
            return true;
        }
    }
    return promptContext.Recognized.Succeeded;
}
```

Теперь выше в объявлении MainDialog измените:

```csharp
// Define the main dialog and its related components.
AddDialog(new ChoicePrompt(nameof(ChoicePrompt)));
```

на:

```csharp
// Define the main dialog and its related components.
AddDialog(new ChoicePrompt(nameof(ChoicePrompt), ChoiceValidator));
```

При этом будет вызван проверяющий элемент управления для поиска входных данных адаптивной карточки каждый раз, когда создается ChoicePrompt.

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Откройте mainDialog.js и найдите метод Run _async run(turnContext, accessor)_ . Этот метод обрабатывает входящие действия.
Сразу после вызова _dialogSet.add(this);_ добавьте следующий код:

```JavaScript
// The following check looks for a non-existant text input
// plus Adaptive Card input in _activity.value.text
// If both conditions exist, the Activity Card text
// is copied into the text input field.
if(turnContext._activity.text == null
    && turnContext._activity.value.text != null) {
    this.logger.log('replacing null text with Activity Card text input');
    turnContext._activity.text = turnContext._activity.value.text;
  }
```

Если в ходе этой проверки будет найден несуществующий текстовый ввод из клиента, выполняется проверка того, существуют ли входные данные адаптивной карточки.
Если входные данные адаптивной карточки уже существуют в \_activity.value.text, они копируются в обычное текстовое поле ввода.

### <a name="pythontabpython"></a>[Python](#tab/python)

Представленный здесь исходный код основан на примере [предложенных действий](https://aka.ms/SuggestedActionsPython).

Создание и отправка действия с предлагаемыми действиями пользователю.

Этот код choice_validator() включается в пример 06.using-cards, открытом для объявления MainDialog, сразу после закрытой фигурной скобки:

```python
@staticmethod
async def choice_validator(prompt_context: PromptValidatorContext) -> bool:
    if prompt_context.context.activity.value:
        text = prompt_context.context.activity.value["text"].lower()
        if not prompt_context.recognized.succeeded and text:
            matching_choices = [choice for choice in prompt_context.options.choices if choice.value.lower() == text]
            if matching_choices:
                choice = matching_choices[0]
                prompt_context.recognized.value = FoundChoice(
                    value=choice.value,
                    index=0,
                    score=1.0
                )
                return True

    return prompt_context.recognized.succeeded
```

Теперь выше в объявлении MainDialog измените:

```python
self.add_dialog(ChoicePrompt(CARD_PROMPT))
```

на:

```python
self.add_dialog(ChoicePrompt(CARD_PROMPT, MainDialog.choice_validator))
```

При этом будет вызван проверяющий элемент управления для поиска входных данных адаптивной карточки каждый раз, когда создается ChoicePrompt.

---

Чтобы протестировать код, после отображения адаптивной карточки нажмите кнопку Text (Текст), введите допустимый вариант, например "Карточка имиджевого баннера", и нажмите кнопку OK.

![Проверка адаптивной карточки](media/adaptive-card-input.png)

1. Первые входные данные будут использоваться для запуска нового диалога.
2. Снова нажмите кнопку OK, и эти входные данные будут использоваться для выбора новой карточки.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Добавление кнопок для управления действиями пользователя](./bot-builder-howto-add-suggested-actions.md)

---
title: Добавление вложений в виде форматированных карточек в сообщения (JS версии 3) — Служба Azure Bot
description: Узнайте, как отправить привлекательные интерактивные функциональные карточки с помощью пакета SDK Bot Framework для Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: cd2997bc4795a922d25f6c4b4f5fef3d7dd4f366
ms.sourcegitcommit: 4e1af50bd46debfdf9dcbab9a5d1b1633b541e27
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/25/2020
ms.locfileid: "76752916"
---
# <a name="add-rich-card-attachments-to-messages"></a>Добавление вложений в виде форматированных карточек в сообщения

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-rich-card-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-rich-cards.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-rich-cards.md)

Некоторые каналы, например Skype и Facebook, поддерживают отправку пользователям функциональных графических карточек, которые содержат интерактивные кнопки для запуска определенных действий.
Пакет SDK предоставляет несколько классов для создания и отправки карточек, прикрепленных к сообщениям. Служба соединителя Azure Bot Framework подготавливает эти карточки к просмотру, используя схему для конкретного канала, позволяя организовать обмен данными между разными платформами. В канале, который не поддерживает карточки (например, в текстовых сообщениях) Bot Framework по возможности отобразит требуемое содержимое пользователям.

## <a name="types-of-rich-cards"></a>Типы функциональных карточек

Сейчас Bot Framework поддерживает восемь типов форматированных карточек.

| Тип карточки | Описание |
|------|------|
| [Адаптивная карточка](/adaptive-cards/get-started/bots) | Настраиваемая карточка, которая может содержать любое сочетание текста, речи, изображений, кнопок и полей для ввода.  См. описание [поддержки для каждого канала](/adaptive-cards/get-started/bots#channel-status). |
| [Анимационная карточка][animationCard] | Карточка, которая может воспроизводить GIF-файлы с анимацией или короткие видеоролики. |
| [Карточка с аудио][audioCard] | Карточка, которая может воспроизводить звуковой файл. |
| [Карточка для имиджевого баннера][heroCard] | Карточка, которая обычно содержит одно большое изображение, одну или несколько кнопок и текст. |
| [Карточка с эскизом][thumbnailCard] | Карточка, которая обычно содержит один эскиз, одну или несколько кнопок и текст.|
| [Карточка квитанции][receiptCard] | Карточка, с помощью которой бот выдает квитанцию пользователю. Обычно она содержит список элементов, включаемых в квитанцию, налог, а также общую информацию и другой текст. |
| [Карточка для входа][signinCard] | Карточка, в которой бот запрашивает вход пользователя. Обычно она содержит текст и одну или несколько кнопок, которые можно нажать, чтобы начать процесс входа. |
| [Карточка с видео][videoCard] | Карточка, которая может воспроизводить видео. |

## <a name="send-a-carousel-of-hero-cards"></a>Отправка карусели имиджевых карточек

В следующем примере представлен бот, который отправляет карусель карточек от имени вымышленной компании по производству футболок в ответ на сообщение пользователя "show shirts" (Показать футболки).

```javascript
// Create your bot with a function to receive messages from the user
// Create bot and default message handler
var bot = new builder.UniversalBot(connector, function (session) {
    session.send("Hi... We sell shirts. Say 'show shirts' to see our products.");
});

// Add dialog to return list of shirts available
bot.dialog('showShirts', function (session) {
    var msg = new builder.Message(session);
    msg.attachmentLayout(builder.AttachmentLayout.carousel)
    msg.attachments([
        new builder.HeroCard(session)
            .title("Classic White T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/whiteshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic white t-shirt", "Buy")
            ]),
        new builder.HeroCard(session)
            .title("Classic Gray T-Shirt")
            .subtitle("100% Soft and Luxurious Cotton")
            .text("Price is $25 and carried in sizes (S, M, L, and XL)")
            .images([builder.CardImage.create(session, 'http://petersapparel.parseapp.com/img/grayshirt.png')])
            .buttons([
                builder.CardAction.imBack(session, "buy classic gray t-shirt", "Buy")
            ])
    ]);
    session.send(msg).endDialog();
}).triggerAction({ matches: /^(show|list)/i });
```

В этом примере для создания карусели используется класс [Message][Message].  
Карусель состоит из нескольких классов [HeroCard][heroCard], каждый из которых содержит изображение, текст и одну кнопку для приобретения товара.  
Нажатие кнопки **Buy** (Купить) приводит к отправке сообщения, следовательно, требуется еще один диалог для перехвата нажатий.

## <a name="handle-button-input"></a>Обработка ввода от кнопок

Диалог `buyButtonClick` активируется каждый раз при получении сообщений, начинающихся со слова "buy" (Купить) или "add" (Добавить) и содержащих любой текст со словом "shirt" (Футболка).
В начале диалога проверяются несколько регулярных выражений, которые ищут цвет и (необязательно) размер футболки в запросе пользователя.
Такая гибкость позволяет обрабатывать не только нажатия кнопок, но и запросы от пользователя на естественном языке, например "please add a large gray shirt to my cart" (Добавьте в мою корзину большую серую футболку).
Если цвет удается распознать, но размер не известен, бот предложит пользователю выбрать размер из списка доступных, прежде чем добавлять элемент в корзину.
Когда бот получит все необходимые сведения, он поместит элемент в корзину, для сохранения которой используется параметр **session.userData**, а затем отправит пользователю сообщение с подтверждением.

```javascript
// Add dialog to handle 'Buy' button click
bot.dialog('buyButtonClick', [
    function (session, args, next) {
        // Get color and optional size from users utterance
        var utterance = args.intent.matched[0];
        var color = /(white|gray)/i.exec(utterance);
        var size = /\b(Extra Large|Large|Medium|Small)\b/i.exec(utterance);
        if (color) {
            // Initialize cart item
            var item = session.dialogData.item = {
                product: "classic " + color[0].toLowerCase() + " t-shirt",
                size: size ? size[0].toLowerCase() : null,
                price: 25.0,
                qty: 1
            };
            if (!item.size) {
                // Prompt for size
                builder.Prompts.choice(session, "What size would you like?", "Small|Medium|Large|Extra Large");
            } else {
                //Skip to next waterfall step
                next();
            }
        } else {
            // Invalid product
            session.send("I'm sorry... That product wasn't found.").endDialog();
        }
    },
    function (session, results) {
        // Save size if prompted
        var item = session.dialogData.item;
        if (results.response) {
            item.size = results.response.entity.toLowerCase();
        }

        // Add to cart
        if (!session.userData.cart) {
            session.userData.cart = [];
        }
        session.userData.cart.push(item);

        // Send confirmation to users
        session.send("A '%(size)s %(product)s' has been added to your cart.", item).endDialog();
    }
]).triggerAction({ matches: /(buy|add)\s.*shirt/i });
```

<!-- 
> [!NOTE]
> When sending a message that contains images, keep in mind that some channels download images before displaying a message to the user. >  
> As a result, a message containing an image followed immediately by a message without images may sometimes be flipped in the user's feed.
> For information on how to avoid messages being sent out of order, see [Message ordering][MessageOrder].  
-->

## <a name="add-a-message-delay-for-image-downloads"></a>Добавление задержки сообщения для скачивания изображений

В некоторых каналах сообщения отображаются только после скачивания изображений. Это означает, что при отправке текстового сообщения сразу после сообщения с изображением эти сообщения могут поступить к получателю в обратном порядке. Чтобы снизить вероятность такой проблемы, следует использовать только изображения из сетей доставки содержимого (CDN) и избегать слишком больших изображений. В исключительных случаях мы рекомендуем добавлять задержку в 1–2 секунды между сообщением с изображением и следующим сообщением. Чтобы такая задержка выглядела для пользователя более естественной, вызовите метод **session.sendTyping()** перед началом задержки, чтобы у пользователя отображался индикатор набора сообщения.

<!-- 
To learn more about sending a typing indicator, see [How to send a typing indicator](bot-builder-nodejs-send-typing-indicator.md).
-->

Bot Framework реализует пакетную обработку, чтобы избежать нарушения порядка отображения нескольких сообщений, отправляемых ботом. <!-- Unfortunately, not all channels can guarantee this. --> Если бот отправляет пользователю несколько ответов, отдельные сообщения автоматически группируются в пакеты и передаются пользователю, чтобы сохранить исходный порядок сообщений. Автоматическое пакетирование по умолчанию ожидает 250 мс после каждого вызова **session.send()** , и лишь затем инициирует следующий вызов **send()** .

Вы можете настроить задержку для пакетной обработки. Чтобы полностью отключить логику пакетной обработки, реализованную в пакете SDK, установите большое значение задержки и вручную вызовите **sendBatch()** с поддержкой обратного вызова после доставки пакета.

## <a name="send-an-adaptive-card"></a>Отправка адаптивной карточки

Адаптивная карточка может содержать любое сочетание текста, речи, изображений, кнопок и полей для ввода.
Адаптивные карточки создаются в формате JSON (см. [здесь](http://adaptivecards.io)), что позволяет получить больший контроль над содержимым и форматом карточек.

Чтобы создать адаптивную карточку с помощью Node.js, перейдите [сюда](http://adaptivecards.io), чтобы получить представление о схеме адаптивных карточек, изучить элементы адаптивных карточек и просмотреть примеры JSON, которые можно использовать для создания карточек разного состава и уровня сложности. А воспользовавшись интерактивным визуализатором, вы сможете разрабатывать соответствующие полезные нагрузки и просматривать выходные данные карточки.

В примере кода ниже показано, как создать сообщение, содержащее адаптивную карточку для напоминания календаря:

[!code-javascript[Add Adaptive Card attachment](../includes/code/node-send-card-buttons.js#addAdaptiveCardAttachment)]

Созданная карточка содержит три блока текста, поле для ввода (список значений) и три кнопки:

![Адаптивная карточка с напоминанием календаря](../media/adaptive-card-reminder.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Справочник по каналам][inspector]
- [Адаптивные карточки](http://adaptivecards.io)
- [AnimationCard][animationCard]
- [AudioCard][audioCard]
- [HeroCard][heroCard];
- [ThumbnailCard][thumbnailCard].
- [ReceiptCard][receiptCard];
- [SigninCard][signinCard]
- [VideoCard][videoCard]
- [Сообщение][Message]
- [Отправка вложений](bot-builder-nodejs-send-receive-attachments.md)

[MessageOrder]: bot-builder-nodejs-manage-conversation-flow.md#message-ordering
[Message]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.message
[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[animationCard]: https://docs.microsoft.com/javascript/api/botbuilder/animationcard?view=botbuilder-ts-3.0
[audioCard]: https://docs.microsoft.com/javascript/api/botbuilder/audiocard?view=botbuilder-ts-3.0
[heroCard]: https://docs.microsoft.com/javascript/api/botbuilder/herocard?view=botbuilder-ts-3.0
[thumbnailCard]: https://docs.microsoft.com/javascript/api/botbuilder/thumbnailcard?view=botbuilder-ts-3.0
[receiptCard]: https://docs.microsoft.com/javascript/api/botbuilder/receiptcard?view=botbuilder-ts-3.0
[signinCard]: https://docs.microsoft.com/javascript/api/botbuilder/signincard?view=botbuilder-ts-3.0
[videoCard]: https://docs.microsoft.com/javascript/api/botbuilder/videocard?view=botbuilder-ts-3.0

[inspector]: ../bot-service-channels-reference.md

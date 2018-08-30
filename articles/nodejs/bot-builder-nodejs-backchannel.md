---
title: Обмен данными с помощью веб-элемента управления | Документация Майкрософт
description: Сведения о том, как реализовать обмен данными между ботом и веб-страницей, используя пакет SDK Bot Builder для Node.js.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5286f0cc6a6a5d7558de997fc776671219c70aa5
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904873"
---
# <a name="use-the-backchannel-mechanism"></a>Использование механизма обратного канала

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to backchannel mechanism](../includes/snippet-backchannel.md)]

## <a name="walk-through"></a>Пошаговое руководство

Элемент управления "Веб-чат" с открытым кодом обращается к API для Direct Line, используя класс JavaScript с именем <a href="https://github.com/microsoft/botframework-DirectLinejs" target="_blank">DirectLineJS</a>. Элемент управления может создать собственный экземпляр Direct Line или совместно использовать такой экземпляр со страницей размещения. Если элемент управления использует экземпляр Direct Line совместно со страницей размещения, то элемент управления и страница смогут отправлять и получать действия. На следующей схеме показана архитектура веб-сайта высокого уровня, поддерживающая функции бота с помощью элемента управления "Веб-чат" с открытым кодом и API для Direct Line. 

![Обратный канал](../media/designing-bots/patterns/back-channel.png)

### <a name="sample-code"></a>Пример кода 

В этом примере бот и веб-страница будут использовать механизм обратного канала для обмена данными, невидимыми для пользователя. Бот отправляет веб-странице запрос на изменение цвета фона, а веб-страница отправит боту уведомление, когда пользователь нажмет кнопку на странице. 

> [!NOTE]
> Фрагменты кода в этой статье взяты из <a href="https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/backchannel/index.html" target="_blank">примера механизма обратного канала</a> и <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">бота с механизмом обратного канала</a>. 

#### <a name="client-side-code"></a>Код на стороне клиента

Сначала веб-страница создает объект **DirectLine**.

```javascript
var botConnection = new BotChat.DirectLine(...);
```

Затем она предоставляет общий доступ к объекту **DirectLine** при создании экземпляра веб-чата.

```javascript
BotChat.App({
    botConnection: botConnection,
    user: user,
    bot: bot
}, document.getElementById("BotChatGoesHere"));
```

Когда пользователь нажимает кнопку на веб-странице, веб-страница публикует действие с типом event, чтобы сообщить боту о нажатии кнопки.

```javascript
const postButtonMessage = () => {
    botConnection
        .postActivity({type: "event", value: "", from: {id: "me" }, name: "buttonClicked"})
        .subscribe(id => console.log("success"));
    }
```

> [!TIP]
> Используйте атрибуты `name` и `value` для передачи всех данных, которые могут потребоваться боту, чтобы правильно интерпретировать событие и (или) отреагировать на него. 

В результате веб-страница ожидает передачи данных об определенном событии от бота.
В нашем примере веб-страница ожидает передачи данных о действии с типом event и именем changeBackground. При получении данных о действии такого типа на веб-странице меняется цвет фона в соответствии с элементом `value`, определяемым действием. 

```javascript
botConnection.activity$
    .filter(activity => activity.type === "event" && activity.name === "changeBackground")
    .subscribe(activity => changeBackgroundColor(activity.value))
```

#### <a name="server-side-code"></a>Код серверной части

Для создания события <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">бот с механизмом обратного канала</a> использует вспомогательную функцию.

```javascript
var bot = new builder.UniversalBot(connector, 
    function (session) {
        var reply = createEvent("changeBackground", session.message.text, session.message.address);
        session.endDialog(reply);
    }
);

const createEvent = (eventName, value, address) => {
    var msg = new builder.Message().address(address);
    msg.data.type = "event";
    msg.data.name = eventName;
    msg.data.value = value;
    return msg;
}
```

Как и в описанном выше процессе, бот ожидает передачи данных о событии от клиента. Когда бот из нашего примера получит данные о событии с `name="buttonClicked"`, он отправит пользователю сообщение "I see that you clicked a button" (Вижу, что Вы нажали кнопку).

```javascript
bot.on("event", function (event) {
    var msg = new builder.Message().address(event.address);
    msg.data.textLocale = "en-us";
    if (event.name === "buttonClicked") {
        msg.data.text = "I see that you clicked a button.";
    }
    bot.send(msg);
})
```

## <a name="additional-resources"></a>Дополнительные ресурсы

- [API для Direct Line][directLineAPI]
- <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Элемент управления "Веб-чат" Microsoft Bot Framework</a>
- <a href="https://github.com/Microsoft/BotFramework-WebChat/blob/master/samples/backchannel/index.html" target="_blank">Пример механизма обратного канала</a>
- <a href="https://github.com/ryanvolum/backChannelBot" target="_blank">Back Channel Bot</a> (Бот с механизмом обратного канала)

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle

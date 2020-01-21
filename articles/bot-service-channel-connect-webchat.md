---
title: Подключение бота к каналу "Веб-чат" — Служба Azure Bot
description: Узнайте, как использовать элемент управления "Веб-чат" на веб-странице для бота, подключенного к каналу "Веб-чат".
keywords: web chat, bot channel, web page, secret key, iframe, HTML
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/22/2019
ms.openlocfilehash: a77f621d46b9ce69d92b83a9f6ada0c9fbf6286e
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791755"
---
# <a name="connect-a-bot-to-web-chat"></a>Подключение бота к веб-чату

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

При [создании бота](bot-service-quickstart.md) с помощью службы Bot канал "Веб-чат" настраивается автоматически. Канал "Веб-чат" включает элемент управления "Веб-чат", который предоставляет пользователям возможность взаимодействовать с ботом непосредственно на веб-странице.

![Пример веб-чата](./media/bot-service-channel-webchat/create-a-bot.png)

Канал "Веб-чат" на портале Bot Framework содержит все необходимое для внедрения элемента управления "Веб-чат" на веб-странице. Для использования элемента управления "Веб-чат" нужно всего лишь получить секретный ключ бота и внедрить элемент управления на веб-странице.

## <a name="web-chat-and-direct-line-considerations"></a>Рекомендации по веб-чатам и Direct Line

> [!IMPORTANT]
> Учитывайте эти важные [вопросы, связанные с безопасностью](rest-api/bot-framework-rest-direct-line-3-0-authentication.md#security-considerations).

## <a name="get-your-bot-secret-key"></a>Получение секретного ключа бота

1. Откройте бот на [портале Azure](https://portal.azure.com) и выберите колонку **Каналы**.

2. Щелкните **Изменить** для канала **Веб-чат**.  
![Канал "Веб-чат"](./media/bot-service-channel-webchat/bot-service-channel-list.png)

3. В разделе **Secret keys** (Секретные ключи) щелкните **Show** (Показать) для первого ключа.  
![Секретный ключ](./media/bot-service-channel-webchat/secret-key.png)

4. Скопируйте **секретный ключ** и **код внедрения**.

5. Нажмите кнопку **Done**(Готово).

## <a name="embed-the-web-chat-control-in-your-website"></a>Внедрение элемента управления "Веб-чат" на веб-сайте

Для внедрения элемента управления "Веб-чат" на веб-сайте можно воспользоваться одним из двух вариантов.

### <a name="option-1---keep-your-secret-hidden-exchange-your-secret-for-a-token-and-generate-the-embed"></a>Вариант 1. Скрытие секрета, обмен секрета на токен и создание внедрения

Используйте этот вариант, если можно выполнить запрос между серверами для обмена секрета веб-чата на временный токен, а также если вы хотите усложнить внедрение бота на веб-сайтах для других разработчиков. Этот вариант не помешает другим разработчикам внедрить ваш бот на своих веб-сайтах, но усложнит эту задачу.

Для обмена секрета на токен и создания внедрения сделайте следующее:

1. Выдайте запрос **GET** к `https://webchat.botframework.com/api/tokens` и передайте секрет веб-чата с помощью заголовка `Authorization`. Заголовок `Authorization` использует схему `BotConnector` и включает в себя ваш секрет, как показано в следующем примере запроса.

2. Ответ на ваш запрос **GET** будет содержать токен (в кавычках), который можно использовать, чтобы начать общение путем рендеринга элемента управления "Веб-чат" в пределах **iframe**. Токен действителен только для одного диалога. Чтобы запустить другой диалог, необходимо создать токен.

3. В **коде внедрения** `iframe`, скопированном в канале Web Chat на портале Bot Framework (см. сведения о [получении секретного ключа бота](#get-your-bot-secret-key) выше), измените параметр `s=` на `t=` и замените YOUR_SECRET_HERE своим токеном.

> [!NOTE]
> Токены будут автоматически продлены до истечения срока действия. 

##### <a name="example-request"></a>Пример запроса

```
requestGET https://webchat.botframework.com/api/tokens
Authorization: BotConnector YOUR_SECRET_HERE
```

##### <a name="example-response"></a>Пример ответа 

```response
"IIbSpLnn8sA.dBB.MQBhAFMAZwBXAHoANgBQAGcAZABKAEcAMwB2ADQASABjAFMAegBuAHYANwA.bbguxyOv0gE.cccJjH-TFDs.ruXQyivVZIcgvosGaFs_4jRj1AyPnDt1wk1HMBb5Fuw"
```

##### <a name="example-iframe-using-token"></a>Пример iframe (с использованием токена)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?t=YOUR_TOKEN_HERE"></iframe>
```

##### <a name="example-html-code"></a>Пример HTML-кода
```html
<!DOCTYPE html>
<html>
<body>
  <iframe id="chat" style="width: 400px; height: 400px;" src=''></iframe>
</body>
<script>

    var xhr = new XMLHttpRequest();
    xhr.open('GET', "https://webchat.botframework.com/api/tokens", true);
    xhr.setRequestHeader('Authorization', 'BotConnector ' + 'YOUR SECRET HERE');
    xhr.send();
    xhr.onreadystatechange = processRequest;

    function processRequest(e) {
      if (xhr.readyState == 4  && xhr.status == 200) {
        var response = JSON.parse(xhr.responseText);
        document.getElementById("chat").src="https://webchat.botframework.com/embed/lucas-direct-line?t="+response
      }
    }

  </script>
</html>
```

### <a id="option-2"></a> Вариант 2. Внедрение элемента управления "Веб-чат" на веб-сайте с помощью секрета

Используйте этот вариант, если вы хотите разрешить другим разработчикам с легкостью внедрять бот на их веб-сайтах. 

> [!WARNING]
> При использовании этого варианта другие разработчики смогут внедрять бот на своих веб-сайтах, просто скопировав ваш код внедрения.

Чтобы внедрить бот на вашем веб-сайте, указав секрет в теге `iframe`, сделайте следующее:

1. Скопируйте **код внедрения** `iframe` в канале Web Chat на портале Bot Framework (см. сведения о [получении секретного ключа бота](#get-your-bot-secret-key) выше).

2. В этом **коде внедрения** замените YOUR_SECRET_HERE значением **секретного ключа**, скопированным с той же страницы.

##### <a name="example-iframe-using-secret"></a>Пример iframe (с использованием секрета)

```html
<iframe src="https://webchat.botframework.com/embed/YOUR_BOT_ID?s=YOUR_SECRET_HERE"></iframe>
```

## <a name="style-the-web-chat-control"></a>Стилизация элемента управления "Веб-чат"

Вы можете изменить размер элемента управления "Веб-чат" с помощью атрибута `style``iframe`, чтобы указать `height` и `width`.

```html
<iframe style="height:480px; width:402px" src="... SEE ABOVE ..."></iframe>
```

![Клиент элемента управления чата](./media/chatwidget-client.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

Вы можете [загрузить исходный код](https://aka.ms/BotFramework-WebChat-V4) для элемента управления "Веб-чат" с сайта GitHub.

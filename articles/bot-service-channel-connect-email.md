---
title: Подключение бота к электронной почте Office 365 в службе Bot
description: Сведения о настройке бота для отправки и приема электронной почты в Office 365.
keywords: Office 365, каналы бота, электронная почта, учетные данные электронной почты, портал Azure, пользовательская электронная почта
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/15/2019
ms.openlocfilehash: 8ca2735dac838982af3ba97221e07d80155153b7
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75792095"
---
# <a name="connect-a-bot-to-office-365-email"></a>Подключение бота к электронной почте Office 365

В дополнении к [каналам](~/bot-service-manage-channels.md), бот может обмениваться данными с пользователями через электронную почту Office 365. Если у бота настроен доступ к электронной почте, то когда приходит новое письмо, он получает сообщение. Затем, согласно своей бизнес-логике, он может на него ответить. Например, бот может отправить в ответ письмо электронной почты со следующим текстом: "Hi! Благодарим вас за заказ. We will begin processing it immediately."

> [!WARNING]
> Нарушением [Соглашения по интерактивным службам](https://www.botframework.com/Content/Microsoft-Bot-Framework-Preview-Online-Services-Agreement.htm) в Bot Framework является создание "спам-ботов", в том числе ботов, которые отправляют ненужные или нежелательные сообщения электронной почты.

> [!NOTE]
> Если вы используете Microsoft Exchange Server, перед настройкой канала электронной почты убедитесь, что [автообнаружение](https://docs.microsoft.com/exchange/client-developer/exchange-web-services/autodiscover-for-exchange) включено. 

## <a name="configure-email-credentials"></a>Настройка учетных данных электронной почты

Чтобы подключить бота к каналу электронной почты, необходимо ввести учетные данные Office 365 в настройки канала электронной почты.
Федеративная проверка подлинности с использованием любого поставщика, который заменяет AAD, не поддерживается.

> [!NOTE]
> Не используйте личные учетные записи электронной почты для ботов, так как каждое сообщение, отправленное на эту учетную запись электронной почты, также будет отправлено боту. Это может привести к тому, что бот отправит неправильный ответ отправителю. Поэтому боты должны использовать только специальные учетные записи электронной почты O365.

Чтобы добавить канал электронной почты, откройте бот на [портале Azure](https://portal.azure.com/), щелкните колонку **Каналы**, а затем выберите **Адрес эл. почты**. Введите действительные учетные данные электронной почты и нажмите кнопку **Сохранить**.

![Ввод учетных данных электронной почты](~/media/bot-service-channel-connect-email/bot-service-channel-connect-email-credentials.png)

В настоящее время канал электронной почты работает только с Office 365. Другие службы электронной почты не поддерживаются.

## <a name="customize-emails"></a>Настройка электронной почты

Канал электронной почты поддерживает отправку пользовательских свойств, что позволяет создавать более сложные, настраиваемые сообщения электронной почты, в которых используется свойство `channelData`.

[!INCLUDE [Email channelData table](~/includes/snippet-channelData-email.md)]

В следующем сообщении показан JSON-файл, который включает эти свойства `channelData`.

```json
{
    "type": "message",
    "locale": "en-Us",
    "channelID": "email",
    "from": { "id": "mybot@mydomain.com", "name": "My bot"},
    "recipient": { "id": "joe@otherdomain.com", "name": "Joe Doe"},
    "conversation": { "id": "123123123123", "topic": "awesome chat" },
    "channelData":
    {
        "htmlBody": "<html><body style = /"font-family: Calibri; font-size: 11pt;/" >This is more than awesome.</body></html>",
        "subject": "Super awesome message subject",
        "importance": "high",
        "ccRecipients": "Yasemin@adatum.com;Temel@adventure-works.com"
    }
}
```

::: moniker range="azure-bot-service-3.0"
Дополнительные сведения об использовании `channelData` см. в примере документации [ Node.js](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-ChannelData) или [.NET](~/dotnet/bot-builder-dotnet-channeldata.md).
::: moniker-end

::: moniker range="azure-bot-service-4.0"
Дополнительные сведения об использовании `channelData` см. в статье [Реализация возможностей для определенных каналов](~/v4sdk/bot-builder-channeldata.md).
::: moniker-end

## <a name="other-considerations"></a>Другие замечания

Если ваш бот не возвращает код состояния HTTP 200 ОК в течение 15 секунд в ответ на входящие сообщения электронной почты, канал электронной почты попытается повторно отправить сообщение, и бот может получить это же действие сообщения электронной почты еще раз. Дополнительные сведения см. в руководствах по [использованию HTTP](v4sdk/bot-builder-basics.md#http-details) со **ботами** и [устранении ошибок времени ожидания](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md).

> [!NOTE]
> При использовании учетной записи Office 365 с поддержкой многофакторной проверки подлинности (MFA) перед настройкой указанной учетной записи для канала электронной почты обязательно отключите MFA. В противном случае подключение завершится ошибкой. 

## <a name="additional-resources"></a>Дополнительные ресурсы

<!-- Put whole list in monikers, even though it's just the second item that needs to be different. -->
::: moniker range="azure-bot-service-3.0"
* Подключение бота к [каналам](~/bot-service-manage-channels.md).
* [Реализация функциональных возможностей канала](dotnet/bot-builder-dotnet-channeldata.md) с помощью пакета SDK Bot Framework для .NET.
* Дополнительные сведения о функциях, поддерживаемых в каждом канале, см. в статье [Разделенные на категории действия по каналам](bot-service-channels-reference.md).
::: moniker-end
::: moniker range="azure-bot-service-4.0"
* Подключение бота к [каналам](~/bot-service-manage-channels.md).
* [Реализация функциональных возможностей канала](~/v4sdk/bot-builder-channeldata.md) с помощью пакета SDK Bot Framework для .NET.
* Дополнительные сведения о функциях, поддерживаемых в каждом канале, см. в статье [Разделенные на категории действия по каналам](bot-service-channels-reference.md).
::: moniker-end


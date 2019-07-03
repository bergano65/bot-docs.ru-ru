---
title: Защита для бота | Документация Майкрософт
description: Сведения о том, как обеспечить защиту бота с помощью HTTPS и проверки подлинности Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8741dee2724d5e3c1e650b86957aabec5716da59
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496640"
---
# <a name="secure-your-bot"></a>Защита для бота

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Бот может подключаться к различным каналам связи (Skype, SMS, электронная почта и др.) через службу соединителя Bot Framework. В этой статье представлены сведения о том, как обеспечить защиту бота с помощью HTTPS и проверки подлинности Bot Framework.

## <a name="use-https-and-bot-framework-authentication"></a>Использование HTTPS и проверки подлинности Bot Framework

Чтобы убедиться, что доступ к конечной точке бота может осуществляться только через [соединитель](bot-builder-dotnet-concepts.md#connector) Bot Framework, настройте конечную точку бота так, чтобы использовался только протокол HTTPS, и включите проверку подлинности Bot Framework, выполнив [регистрацию](~/bot-service-quickstart-registration.md) бота для получения его параметров AppID и Password. 

## <a name="configure-authentication-for-your-bot"></a>Настройка проверки подлинности для бота

Укажите значения параметров AppID и Password в файле web.config бота. 

> [!NOTE]
> Сведения о том, как найти значения **AppID** и **AppPassword** для бота, см. в разделе [MicrosoftAppID и MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

```xml
<appSettings>
    <add key="MicrosoftAppId" value="_appIdValue_" />
    <add key="MicrosoftAppPassword" value="_passwordValue_" />
</appSettings>
```

Затем во время создания бота с помощью пакета SDK Bot Framework для .NET используйте атрибут `[BotAuthentication]`, чтобы указать учетные данные проверки подлинности. 

Чтобы использовать учетные данные проверки подлинности, которые хранятся в файле web.config, укажите `[BotAuthentication]` без параметров.

[!code-csharp[Use botAuthentication attribute](../includes/code/dotnet-security.cs#attribute1)]

Чтобы использовать другие значения для учетных данных проверки подлинности, укажите атрибут `[BotAuthentication]` и передайте эти значения.

[!code-csharp[Use botAuthentication attribute with parameters](../includes/code/dotnet-security.cs#attribute2)]

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Пакет SDK Bot Framework для .NET](bot-builder-dotnet-overview.md)
- [Основные понятия пакета SDK Bot Builder для .NET](bot-builder-dotnet-concepts.md)
- [Register a bot with the Bot Framework](~/bot-service-quickstart-registration.md) (Регистрация бота на платформе Bot Framework)

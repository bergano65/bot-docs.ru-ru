---
title: Перехват сообщений | Документация Майкрософт
description: Узнайте, как перехватывать сообщения между пользователем и ботом с помощью пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 54ab447fac81ba17f1fa277347cf7a5aaaab8850
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54223779"
---
# <a name="intercept-messages"></a>Перехват сообщений

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-middleware.md)
> - [Node.js](../nodejs/bot-builder-nodejs-intercept-messages.md)

[!INCLUDE [Introducton to message logging](../includes/snippet-message-logging-intro.md)]

## <a name="intercept-and-log-messages"></a>Перехват и запись сообщений в журнал

В следующем примере кода показано, как перехватывать сообщения, которые передаются между пользователем и ботом, с помощью **ПО промежуточного слоя** в пакете SDK Bot Framework для .NET. 

Во-первых, создайте класс `DebugActivityLogger` и определите метод `LogAsync`, чтобы указать, какое действие выполняется над каждым перехваченным сообщением. В этом примере просто выводятся некоторые сведения о каждом сообщении.

```cs
public class DebugActivityLogger : IActivityLogger
{
    public async Task LogAsync(IActivity activity)
    {
        Debug.WriteLine($"From:{activity.From.Id} - To:{activity.Recipient.Id} - Message:{activity.AsMessageActivity()?.Text}");
    }
}
```

Затем добавьте следующий код в `Global.asax.cs`.  Каждое сообщение, которым обменялись пользователь и бот (в любом направлении), является триггером метода `LogAsync` в классе `DebugActivityLogger`. 

```cs
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            var builder = new ContainerBuilder();
            builder.RegisterType<DebugActivityLogger>().AsImplementedInterfaces().InstancePerDependency();
            builder.Update(Conversation.Container);

            GlobalConfiguration.Configure(WebApiConfig.Register);
        }
    }
```

Несмотря на то что в этом примере просто выводятся некоторые сведения о каждом сообщении, можно обновить метод `LogAsync`, чтобы указать действия, которые необходимо выполнить над каждым сообщением. 

## <a name="sample-code"></a>Пример кода 

Полный пример, в котором показано, как перехватывать сообщения и записывать их в журнал с помощью пакета SDK Bot Framework для .NET, см. в примере <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">Middleware Bot Sample</a> (Пример бота ПО промежуточного слоя) на сайте GitHub. 

## <a name="additional-resources"></a>Дополнительные ресурсы

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочная информация по пакету SDK Bot Framework для .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-Middleware" target="_blank">Middleware Bot Sample</a> (Пример бота ПО промежуточного слоя) в GitHub

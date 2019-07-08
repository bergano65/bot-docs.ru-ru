---
title: Разработка бота DirectLine Speech | Документация Майкрософт
description: Разработка бота DirectLine Speech
keywords: Разработка бота речь driect line, речевой бот
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 73a675c6e54d676f74dad2df24b3668d5e4e98be
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252382"
---
## <a name="use-direct-line-speech-in-your-bot"></a>Использование канала Direct Line Speech в боте 

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Direct Line Speech использует новую возможность потоковой передачи на основе WebSocket, которую предоставляет Bot Framework, чтобы организовать обмен сообщениями между каналом Direct Line Speech и ботом. Настроив канал Direct Line Speech на портале Azure, следует обновить бота для прослушивания и приема этих подключений WebSocket. Здесь мы приводим инструкции, как это сделать.

## <a name="add-the-nuget-package"></a>Добавление пакета NuGet
Для предварительной версии канала Direct Line Speech существуют дополнительные пакеты NuGet, которые нужно добавить в бота. Эти пакеты размещаются на веб-канале NuGet сайта myget.org:
1.  Откройте проект бота в Visual Studio.

2.  Перейдите в раздел управления пакетами NuGet в свойствах проекта бота.

3.  Добавьте `https://botbuilder.myget.org/F/experimental/api/v3/index.json` в качестве веб-канала, если у вас еще не настроен этот источник, с помощью параметров канала NuGet в правом верхнем углу.

4.  Выберите этот источник NuGet и добавьте один из пакетов `Microsoft.Bot.Protocol.StreamingExtensions.NetCore`.

5.  Подтвердите все запросы, чтобы завершить добавление пакета в проект.

## <a name="option-1-update-your-net-core-bot-code-if-your-bot-has-a-botcontrollercs"></a>Вариант 1. Обновление кода .NET Core для бота, _в котором есть файл BotController.cs_
При создании нового бота на портале Azure с помощью одного из шаблонов, например EchoBot, полученный бот будет содержать контроллер MVC для ASP.NET, который предоставляет одну конечную точку POST. Здесь описано, как дополнить эту систему еще одной конечной точкой для приема потоковой передачи WebSocket, которая использует механизм GET.
1.  Откройте BotController.cs в папке Controllers своего решения.

2.  Найдите в классе метод PostAsync и измените для него оформление [HttpPost] на [HttpPost, HttpGet]:
```cs
[HttpPost, HttpGet]
public async Task PostAsync()
{ 
    await _adapter.ProcessAsync(Request, Response, _bot);
}
```

3.  Сохраните и закройте BotController.cs.

4.  Откройте Startup.cs из корневого каталога решения.

5.  Добавьте новое пространство имен:

```cs
using Microsoft.Bot.Protocol.StreamingExtensions.NetCore;
```

6.  В методе ConfigureServices замените упоминание AdapterWithErrorHandler на WebSocketEnabledHttpAdapter в соответствующем вызове services.AddSingleton:

```cs
public void ConfigureServices(IServiceCollection services)
{
    ...    

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

    services.AddTransient<IBot, EchoBot>();

    ...
}
```

7. В том же файле Startup.cs перейдите в нижнюю часть метода Configure. Перед вызовом app.UseMvc(); добавьте вызов app.UseWebSockets(); (порядок вызовов Use имеет большое значение). Теперь конец метода должен выглядеть примерно так:

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...

    app.UseDefaultFiles();
    app.UseStaticFiles();
    app.UseWebSockets();
    app.UseMvc();

    ...
}
```

8.  Остальная часть кода бота остается неизменной.

## <a name="option-2-update-your-net-core-bot-code-if-your-bot-uses-addbot-and-usebotframework-instead-of-a-botcontroller"></a>Вариант 2. Обновление кода .NET Core для бота, _в котором вместо BotController используются AddBot и UseBotFramework_

Если вы создавали бота с помощью пакета SDK Bot Builder v4 в версии, предшествующей 4.3.2, в нем, скорее всего, нет BotController, а вместо него в файле Startup.cs применяются методы AddBot() и UseBotFramework() для предоставления конечной точки POST, в которой бот получает сообщения. Чтобы предоставить новую конечную точку потоковой передачи, следует добавить BotController и удалить методы AddBot() и UseBotFramework(). Здесь представлены инструкции по внесению таких изменений.

1.  Добавьте в проект бота новый контроллер MVC, добавив файл с именем BotController.cs. Добавьте в этот файл код контроллера:

```cs
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter _adapter;
    private readonly IBot _bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        _adapter = adapter;
        _bot = bot;
    }

    [HttpPost, HttpGet]
    public async Task ProcessMessageAsync()
    {
        await _adapter.ProcessAsync(Request, Response, _bot);
    }
}
```
2.  В файле Startup.cs найдите метод Configure. Удалите строку UseBotFramework() и убедитесь, что здесь есть следующие строки:

```cs
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    ...

    app.UseDefaultFiles();
    app.UseStaticFiles();
    app.UseWebSockets();
    app.UseMvc();

    ...
}
```

3.  В этом же файле Startup.cs найдите метод ConfigureServices. Удалите строку AddBot() и убедитесь, что здесь есть следующие строки:

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, WebSocketEnabledHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```
4.  Остальная часть кода бота остается неизменной.

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Подключение бота к каналу Direct Line Speech (предварительная версия)](./bot-service-channel-connect-directlinespeech.md)

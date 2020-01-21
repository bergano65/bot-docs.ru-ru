---
title: Разработка бота DirectLine Speech — Служба Azure Bot
description: Разработка бота DirectLine Speech
keywords: develop Direct Line speech bot, speech bot
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2aaafb7c46097a178761b356299612d874b1a823
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75795967"
---
# <a name="use-direct-line-speech-in-your-bot"></a>Использование канала Direct Line Speech в боте

[!INCLUDE [applies-to-v4](includes/applies-to.md)]

Direct Line Speech использует новую возможность потоковой передачи на основе WebSocket, которую предоставляет Bot Framework, чтобы организовать обмен сообщениями между каналом Direct Line Speech и ботом. Настроив канал Direct Line Speech на портале Azure, следует обновить бота для прослушивания и приема этих подключений WebSocket. Здесь мы приводим инструкции, как это сделать.  

## <a name="step-1-upgrade-to-the-46-sdk"></a>Шаг 1. Обновление до пакета SDK версии 4.6 

Для использования Direct Line Speech требуется пакет SDK для Bot Builder версии 4.6 или выше. 

## <a name="step-2-update-your-net-core-bot-codeif-your-bot-uses-addbot-and-usebotframework-instead-of-a-botcontroller"></a>Шаг 2. Обновите код бота для .NET Core, если в нем вместо BotController используются AddBot и UseBotFramework. 

Если вы создавали бота с помощью пакета SDK Bot Builder v4 в версии, предшествующей 4.3.2, в нем, скорее всего, нет BotController, а вместо него в файле Startup.cs применяются методы AddBot() и UseBotFramework() для предоставления конечной точки POST, в которой бот получает сообщения. Чтобы предоставить новую конечную точку потоковой передачи, следует добавить BotController и удалить методы AddBot() и UseBotFramework(). Здесь представлены инструкции по внесению таких изменений. Если вы уже внесли эти изменения, перейдите к следующему шагу. 

Добавьте в проект бота новый контроллер MVC, добавив файл с именем BotController.cs. Добавьте в этот файл код контроллера: 

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

В файле **Startup.cs** найдите метод Configure. Удалите строку `UseBotFramework()` и убедитесь, что в нем есть следующие строки для `UseWebSockets`: 

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

В этом же файле Startup.cs найдите метод ConfigureServices. Удалите строку `AddBot()` и убедитесь, что в нем есть строки для добавления `IBot` и `BotFrameworkHttpAdapter`: 

```cs

public void ConfigureServices(IServiceCollection services) 
{ 
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1); 
    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>(); 
    services.AddSingleton<IChannelProvider, ConfigurationChannelProvider>(); 
    
    // Create the Bot Framework Adapter. 
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>(); 

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot. 
    services.AddTransient<IBot, EchoBot>(); 
} 
```

Остальная часть кода бота остается неизменной. 

## <a name="step3-ensure-websockets-are-enabled"></a>Шаг 3. Убедитесь, что включены WebSocket 

При создании нового бота на портале Azure с помощью одного из шаблонов, например EchoBot, полученный бот будет содержать контроллер MVC для ASP.NET, который предоставляет одну конечную точку GET и POST и поддерживает WebSocket. В этих инструкциях описано, как добавить в бот эти элементы, если вы его обновляете или создавали не из шаблона бота. 

Откройте **BotController.cs** в папке Controllers для своего решения. 

Найдите в классе метод `PostAsync` и измените для него маркер [HttpPost] на [HttpPost, HttpGet]: 

```cs

[HttpPost, HttpGet] 
public async Task PostAsync() 
{ 
    await _adapter.ProcessAsync(Request, Response, _bot); 
} 
```

Сохраните и закройте BotController.cs. 

Откройте файл **Startup.cs** из корневого каталога решения. 

В файле Startup.cs перейдите в нижнюю часть метода Configure. Перед вызовом  `app.UseMvc()` добавьте вызов  `app.UseWebSockets()`. Порядок вызовов этих инструкций use имеет значение. Теперь конец метода должен выглядеть примерно так: 

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
Остальная часть кода бота остается неизменной. 

 

Шаг 4. При необходимости заполните поле Speak (Сказать) в разделе действий, чтобы изменить проговариваемые для пользователя сообщения. 

По умолчанию проговариваются все сообщения, отправляемые пользователю через Direct Line Speech.  

По желанию вы можете настроить режим озвучки сообщений, установив флажок Speak (Сказать) для любого действия, отправляемого из бота. 

```cs 

public IActivity Speak(string message) 
{ 
    var activity = MessageFactory.Text(message); 
    string body = @"<speak version='1.0' xmlns='https://www.w3.org/2001/10/synthesis' xml:lang='en-US'> 

        <voice name='Microsoft Server Speech Text to Speech Voice (en-US, JessaRUS)'>" + 
        $"{message}" + "</voice></speak>"; 

    activity.Speak = body; 
    return activity; 
} 
```

Следующий фрагмент кода демонстрирует, как использовать предыдущую функцию Speak. 

```cs

protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken) 
{ 
    await turnContext.SendActivityAsync(Speak($"Echo: {turnContext.Activity.Text}"), cancellationToken); 
} 
``` 

## <a name="additional-information"></a>Дополнительные сведения 

- Полный пример создания и использования бота с поддержкой голосовых функций см. в  [этом руководстве](https://docs.microsoft.com/azure/cognitive-services/speech-service/tutorial-voice-enable-your-bot-speech-sdk). 

- Сведения о работе с действиями см. в руководствах по  [использованию ботов](https://docs.microsoft.com/azure/bot-service/bot-builder-basics) и  [отправке и получению текстовых сообщений https://docs.microsoft.com/azure/bot-service/bot-builder-howto-send-messages?view=azure-bot-service-4.0](). 

 

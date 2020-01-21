---
title: Подключение бота к WeChat — Служба Azure Bot
description: Сведения о настройке подключения бота к WeChat.
keywords: WeChat, Tencent, канал бота, приложение WeChat, бот WeChat, идентификатор приложения, секрет приложения, учетные данные
author: seaen
manager: kamrani
ms.topic: article
ms.author: egorn
ms.service: bot-service
ms.date: 11/01/2019
ms.openlocfilehash: 9abba3093ce819f7ebc07bb03e342da797971f25
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791795"
---
# <a name="connect-a-bot-to-wechat"></a>Подключение бота к WeChat

Вы можете настроить взаимодействие бота с пользователями с помощью официальной платформы учетных записей WeChat.

## <a name="download-wechat-adapter-for-bot-framework"></a>Скачивание адаптера WeChat для Bot Framework

WeChat для Microsoft Bot Framework представляет собой адаптер с открытым исходным кодом, который размещен на сайте GitHub. Скачать адаптер WeChat для Bot Framework можно [здесь](https://github.com/microsoft/BotFramework-WeChat/).

## <a name="create-a-wechat-account"></a>Создание учетной записи WeChat

Чтобы настроить бота для общения через WeChat, необходимо создать официальную учетную запись WeChat на [платформе официальных учетных записей WeChat](https://mp.weixin.qq.com/?lang=en_US), а затем подключить бота к приложению. В настоящее время поддерживается только учетная запись службы.

### <a name="change-your-prefer-language"></a>Изменение предпочитаемого языка

Вы можете изменить предпочитаемый язык интерфейса, прежде чем выполнять вход.

 ![change_language](./media/channels/wechat-change-language.png)

### <a name="register-a-service-account"></a>Регистрация учетной записи службы

Реальная учетная запись службы должна быть проверена в WeChat, и вы не сможете включить веб-перехватчик до проверки учетной записи. Следуйте [этим](https://kf.qq.com/product/weixinmp.html#hid=87) инструкциям, чтобы создать учетную запись службы.
Для ускорения процесса просто щелкните ссылку на регистрацию в верхней части страницы, выберите Service Account (Учетная запись службы) и следуйте инструкциям.

 ![register_account](./media/channels/wechat-register-account.png)

### <a name="sandbox-account"></a>Учетная запись песочницы

Если вы просто хотите протестировать интеграцию бота с WeChat, вместо новой учетной записи службы можно использовать учетную запись песочницы. Дополнительные сведения об учетной записи песочницы см. в [этой статье](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=sandbox/login).

## <a name="enable-wechat-adapter-to-bot"></a>Включение адаптера WeChat для бота

Проект Bot представляет собой обычный проект на основе пакета SDK для Bot Framework версии 4. Прежде чем открывать проект, необходимо попробовать запустить бота. Скачайте [адаптер WeChat для Bot Framework](https://github.com/microsoft/BotFramework-WeChat/).

### <a name="prerequisites"></a>предварительные требования

    .NET Core SDK (version 2.2.x)

### <a name="add-reference-to-wechat-adapter-source"></a>Добавление ссылки на источник адаптера WeChat

Используйте прямую ссылку на проект адаптера WeChat или добавьте ~/BotFramework-WeChat/libraries/csharp_dotnetcore/outputpackages в качестве локального источника NuGet.

### <a name="inject-wechat-adapter-in-your-bot-startupcs"></a>Включение адаптера WeChat в файл Startup.cs бота

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);

    // Create the storage we'll be using for User and Conversation state. (Memory is great for testing purposes.)
    services.AddSingleton<IStorage, MemoryStorage>();

    // Create the User state. (Used in this bot's Dialog implementation.)
    services.AddSingleton<UserState>();

    // Create the Conversation state. (Used by the Dialog system itself.)
    services.AddSingleton<ConversationState>();

    // Load WeChat settings.
    var wechatSettings = new WeChatSettings();
    Configuration.Bind("WeChatSettings", wechatSettings);
    services.AddSingleton<WeChatSettings>(wechatSettings);

    // Configure hosted serivce.
    services.AddSingleton<IBackgroundTaskQueue, BackgroundTaskQueue>();
    services.AddHostedService<QueuedHostedService>();
    services.AddSingleton<WeChatHttpAdapter>();

    // The Dialog that will be run by the bot.
    services.AddSingleton<MainDialog>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

### <a name="update-your-bot-controller"></a>Обновление контроллера бота

```csharp
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{  
    private readonly IBot _bot;
    private readonly WeChatHttpAdapter _weChatHttpAdapter;
    private readonly string Token;
    public BotController(IBot bot, WeChatHttpAdapter weChatAdapter)
    {
        _bot = bot;
        _weChatHttpAdapter = weChatAdapter;
    }

    [HttpPost("/WeChat")]
    [HttpGet("/WeChat")]
    public async Task PostWeChatAsync([FromQuery] SecretInfo secretInfo)
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await _weChatHttpAdapter.ProcessAsync(Request, Response, _bot, secretInfo);
    }
}
```

### <a name="setup-appsettingsjson"></a>Настройка appsettings.json

Вам необходимо настроить файл appsettings.json, как описано ниже, прежде чем запускать бота.

```json
"WeChatSettings": {
    "UploadTemporaryMedia": true,
    "PassiveResponseMode": false,
    "Token": "",
    "EncodingAESKey": "",
    "AppId": "",
    "AppSecret": ""
}
```

#### <a name="service-account"></a>Учетная запись службы

Если у вас уже есть учетная запись службы и вы готовы развернуть бота, получите значения **AppID** (идентификатор приложения), **AppSecret** (секрет приложения), **EncodingAESKey** (ключ шифрования) и **Token** (токен) на панели навигации слева, как показано ниже.

Не забудьте настроить список разрешенных IP-адресов, иначе WeChat не примет ваш запрос.

 ![serviceaccount_console](./media/channels/wechat-serviceaccount-console.png)

#### <a name="sandbox-account"></a>Учетная запись песочницы

У учетной записи песочницы нет параметра **EncodingAESKey**, то есть сообщения от WeChat не шифруются. Для нее просто оставьте поле EncodingAESKey пустым. В этом случае нужно указать только три параметра: **appID** (идентификатор приложения), **appsecret** (секрет приложения) и **Token** (токен).

 ![sandbox_account](./media/channels/wechat-sandbox-account.png)

### <a name="start-bot-and-set-endpoint-url"></a>Запуск бота и настройка URL-адреса конечной точки

Теперь переходите к настройке серверной части бота. Но сначала, прежде чем сохранить параметры, нужно запустить бота, чтобы WeChat отправил запрос для проверки URL-адреса.
Укажите конечную точку в формате **https://your_end_point/WeChat** или настройте те же параметры, которые вы указали в файле BotController.cs.

 ![sandbox_account2](./media/channels/wechat-sandbox-account-2.png)

### <a name="subscribe-your-official-account"></a>Подписка для официальной учетной записи

С помощью QR-кода вы можете оформить подписку в WeChat для тестовой учетной записи.

 ![subscribe](./media/channels/wechat-subscribe.png)

## <a name="test-through-wechat"></a>Тестирование через WeChat

Итак, все готово и вы можете проверить работу в клиенте WeChat. Например, используйте пример бота из папки tests. Этот бот интегрирован с адаптером WeChat, а также ботами Echo и Cards.

 ![чат](./media/channels/wechat-chat.png)

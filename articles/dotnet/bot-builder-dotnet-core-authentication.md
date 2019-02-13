---
redirect_url: /bot-framework/bot-builder-tutorial-authentication
ms.openlocfilehash: 6dd1013e870fb749919f272e82b60ee633cddadb
ms.sourcegitcommit: c7d2e939ec71f46f48383c750fddaf6627b6489d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/07/2019
ms.locfileid: "55783373"
---
<a name="--"></a><!--
---
заголовок: Проверка подлинности действий с помощью .NET Core | Документация Майкрософт description: Сведения о проверке подлинности действий ботов с помощью .NET Core.
author: v-ducvo ms.author: v-ducvo manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 12/13/17 monikerRange: 'azure-bot-service-3.0'
---

# <a name="authenticating-activities-using-net-core"></a>Проверка подлинности действий с помощью .NET Core

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Если вы выбрали вариант разработки бота с помощью [.NET Core](/dotnet/core/index), можно использовать [соединитель Bot Framework](bot-builder-dotnet-connector.md) для отправки и получения сообщений о [действиях](https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html) от вашего бота. Чтобы использовать службу соединителя, необходимо настроить соответствующую модель проверки подлинности для целевой версии платформы.

Bot Framework Connector.AspNetCore поддерживает следующие версии ASP.NET:
* для .NET Core версии 1.1 и AspNetCore1.x используйте **Microsoft.Bot.Connector.AspNetCore 1.x**;
* для .NET Core версии 2.0 и AspNetCore2.x используйте **Microsoft.Bot.Connector.AspNetCore 2.x**.

В этой статье показано, как настроить модель проверки подлинности для конкретной платформы, для которой предназначен бот.

## <a name="prerequisites"></a>Предварительные требования

* [Visual Studio 2017](https://www.visualstudio.com/downloads/)
* [.NET Core](https://www.microsoft.com/net/download/windows). Установите целевую версию .NET Core (например, .NET Core версии 1.1 или .NET Core версии 2.0).
* [Регистрация бота](~/bot-service-quickstart-registration.md). Зарегистрируйте бот и получите идентификатор приложения и пароль, необходимые для выполнения проверки подлинности.

## <a name="create-a-net-core-project"></a>Создание проекта .NET Core

Чтобы создать проект .NET Core, сделайте следующее:

1. Откройте Visual Studio 2017 и последовательно выберите **Файл > Создать > Проект…**.
2. Разверните узел **Visual C#** и щелкните **.NET Core**.
3. Выберите тип проекта **Веб-приложение ASP.NET Core** и укажите сведения о проекте (например, заполните поля "Имя", "Расположение" и "Имя решения").
4. Последовательно выберите **ОК**.
5. Убедитесь, что проект предназначен для нужных вам версий *.NET Core* и *ASP.NET Core*. Например, на следующем снимке экрана показан проект, предназначенный для **.NET Core** и **ASP.NET Core 2.0**:

![Создание проекта для ASP.NET Core версии 2.0](~/media/dotnet-core-authentication/create-asp-net-core-2x-project.png)
 
  > [!NOTE]
  > Если ваша целевая платформа — **ASP.NET Core 2.0**, убедитесь, что на компьютере установлена по крайней мере версия 2.x или выше.

6. Выберите тип проекта **Веб-API**.
7. Нажмите кнопку **ОК** , чтобы создать проект.

## <a name="download-the-nuget-package"></a>Скачивание пакета NuGet

Чтобы использовать **соединитель Bot Framework**, установите пакет NuGet, подходящий для вашей версии .NET Core. Чтобы установить пакет NuGet, сделайте следующее:

1. В **обозревателе решений** правой кнопкой мыши щелкните имя проекта и выберите пункт **Управление пакетами NuGet…**.
2. Нажмите кнопку **Обзор** и найдите **Connector.ASPNetCore**. 
3. Выберите версию, которую нужно использовать в качестве целевой. Например, на следующем снимке экрана выбрана версия **2.0.0.3**. Чтобы вместо нее установить версию 1.1.3.2, щелкните раскрывающийся список **Версия** и выберите версию **1.1.3.2**.
![Пакет NuGet для ASP.NET Core версии 2.0.0.3](~/media/dotnet-core-authentication/nuget-package-net-core-version.png)
4. Щелкните **Install**(Установить).

## <a name="update-the-appsettingsjson"></a>Обновление файла appsettings.json

Для проверки подлинности бота соединитель Bot Framework запрашивает идентификатор приложения и пароль. Эти значения можно задать в файле **appsettings.json** вашего проекта веб-приложения.

> [!NOTE]
> Сведения о том, как найти значения **AppID** и **AppPassword** для бота, см. в разделе [MicrosoftAppID и MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

### <a name="appsettingsjson-for-net-core-v11"></a>Файл appSettings.JSON для .NET Core версии 1.1:

```json
{
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

### <a name="appsettingsjson-for-net-core-v20"></a>Файл appSettings.JSON для .NET Core версии 2.0:

```json
{
  "Logging": {
    "IncludeScopes": false,
    "Debug": {
      "LogLevel": {
        "Default": "Warning"
      }
    },
    "Console": {
      "LogLevel": {
        "Default": "Warning"
      }
    }
  },
  "MicrosoftAppId": "<MicrosoftAppId>",
  "MicrosoftAppPassword": "<MicrosoftAppPassword>"
}
```

## <a name="update-the-startupcs-class"></a>Обновление класса startup.cs

Обновите класс **startup.cs**. В зависимости от версии проекта обновите соответствующий фрагмент кода, чтобы обеспечить выполнение проверки подлинности.

### <a name="startupcs-for-net-core-v11"></a>Класс startup.cs для .NET Core версии 1.1:

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfigurationRoot Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            services.AddSingleton(typeof(ICredentialProvider), new StaticCredentialProvider(
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value));

            // Add framework services.
            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IServiceProvider serviceProvider, IHostingEnvironment env, ILoggerFactory loggerFactory)
        {
            loggerFactory.AddConsole(Configuration.GetSection("Logging"));
            loggerFactory.AddDebug();

            app.UseStaticFiles();

            ICredentialProvider credentialProvider = serviceProvider.GetService<ICredentialProvider>();

            app.UseBotAuthentication(credentialProvider);

            app.UseMvc();
        }
    }
```

### <a name="startupcs-for-net-core-v20"></a>Класс startup.cs для .NET Core версии 2.0:

```cs
public class Startup
    {
        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        public IConfiguration Configuration { get; }
        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);

            var credentialProvider = new StaticCredentialProvider(Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppIdKey)?.Value,
                Configuration.GetSection(MicrosoftAppCredentials.MicrosoftAppPasswordKey)?.Value);

            services.AddAuthentication(
                    options =>
                    {
                        options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                        options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                    }
                )
                .AddBotAuthentication(credentialProvider);

            services.AddSingleton(typeof(ICredentialProvider), credentialProvider);

            services.AddMvc(options =>
            {
                options.Filters.Add(typeof(TrustServiceUrlAttribute));
            });
        }


        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            app.UseStaticFiles();
            app.UseAuthentication();
            app.UseMvc();
        }
    }
```

## <a name="create-the-messagescontrollercs-class"></a>Создание класса MessagesController.cs

В **обозревателе решений** добавьте новый пустой класс с именем **MessagesController.cs**. В классе **MessagesController.cs** обновите метод **Post** с помощью кода, указанного ниже. Благодаря этому бот сможет отправлять и получать сообщения для пользователя.

```cs
private IConfiguration configuration;

public MessagesController(IConfiguration configuration)
{
    this.configuration = configuration;
}


[Authorize(Roles = "Bot")]
[HttpPost]
public async Task<OkResult> Post([FromBody] Activity activity)
{
    if (activity.Type == ActivityTypes.Message)
    {
        //MicrosoftAppCredentials.TrustServiceUrl(activity.ServiceUrl);
        var appCredentials = new MicrosoftAppCredentials(configuration);
        var connector = new ConnectorClient(new Uri(activity.ServiceUrl), appCredentials);

        // return our reply to the user
        var reply = activity.CreateReply("HelloWorld");
        await connector.Conversations.ReplyToActivityAsync(reply);
    }
    else
    {
        //HandleSystemMessage(activity);
    }
    return Ok();
}
```
-->

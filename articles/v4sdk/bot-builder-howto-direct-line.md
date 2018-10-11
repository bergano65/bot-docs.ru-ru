---
title: Создание бота и клиента Direct Line | Документация Майкрософт
description: Узнайте, как создавать боты и клиенты Direct Line с помощью пакета SDK Bot Builder V4 для .NET.
keywords: direct line bot, direct line client, custom channel, console-based, publish
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/16/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: dee0f9700fefede2a231ff2395e50ff17522806e
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707400"
---
# <a name="create-a-direct-line-bot-and-client"></a>Создание бота и клиента Direct Line

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Боты Microsoft Bot Framework для Direct Line могут работать с пользовательским клиентом вашей собственной разработки. Боты Direct Line очень похожи на обычные. Им просто не нужно использовать предоставленные каналы.

Клиенты Direct Line могут быть написаны произвольным образом. Вы можете написать клиент Android, клиент iOS или даже консольное клиентское приложение.

В этом разделе вы узнаете, как создавать и развертывать бот Direct Line, а также как создавать и запускать консольное клиентское приложение Direct Line.

## <a name="create-your-bot"></a>Создание бота

Для создания бота на разных языках требуется разный подход. Javascript создает бот в Azure, а затем модифицирует код, а C # создает бот локально, а затем публикует его в Azure, но оба являются действительными методами и могут использоваться с любым из этих языков. Если вам нужна информация о публикации вашего бота в Azure, см. статью [Deploy your bot to Azure](../bot-builder-howto-deploy-azure.md) (Развертывание бота в Azure).

# <a name="ctabcscreatebot"></a>[C#](#tab/cscreatebot)

### <a name="create-the-solution-in-visual-studio"></a>Создание решения в Visual Studio

Чтобы создать решение для бота Direct Line в Visual Studio 2015 или более поздней версии:

1. Создайте **веб-приложение ASP.NET Core** в **Visual C #** > **.NET Core**.

1. В качестве имени введите **DirectLineBotSample** и нажмите кнопку **ОК**.

1. Убедитесь, что выбраны **.NET Core** и **ASP.NET Core 2.0**, выберите **пустой** шаблон проекта, затем нажмите кнопку **ОК**.

#### <a name="add-dependencies"></a>Добавление зависимостей

1. В **обозревателе решений** щелкните правой кнопкой мыши **Зависимости**, а затем выберите **Управление пакетами NuGet**.

1. Щелкните **Обзор**, затем убедитесь, что установлен флажок **Включить предварительные выпуски**.

1. Найдите и установите следующие пакеты NuGet:
    - Microsoft.Bot.Builder
    - Microsoft.Bot.Builder.Core.Extensions
    - Microsoft.Bot.Builder.Integration.AspNet.Core
    - Newtonsoft.Json.

### <a name="create-the-appsettingsjson-file"></a>Создание файла appsettings.json

Файл appsettings.json будет содержать идентификатор приложения Майкрософт, пароль приложения и строку подключения к данным. Так как этот бот не будет хранить информацию о состоянии, строка подключения к данным остается пустой. И если вы используете Bot Framework Emulator, все они могут оставаться пустыми.

Чтобы создать файл **appsettings.json**, сделайте следующее:

1. Щелкните правой кнопкой мыши проект **DirectLineBotSample**, выберите пункты **Добавить** > **Новый элемент**.

1. В **ASP.NET Core** щелкните **Файл конфигурации ASP.NET**.

1. Введите **appsettings.json** в качестве имени и нажмите кнопку**Добавить**.

Замените содержимое файла appsettings.json приведенным ниже кодом.

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

    "MicrosoftAppId": "",
    "MicrosoftAppPassword": "",
    "DataConnectionString": ""
}
```

### <a name="edit-startupcs"></a>Изменение Startup.cs

Замените содержимое файла Startup.cs следующим кодом:

**Примечание**. **DirectBot** будет определен на следующем шаге, поэтому вы можете игнорировать красное подчеркивание.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace DirectLineBotSample
{
    public class Startup
    {
        public IConfiguration Configuration { get; }

        public Startup(IHostingEnvironment env)
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(env.ContentRootPath)
                .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
                .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
                .AddEnvironmentVariables();
            Configuration = builder.Build();
        }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton(_ => Configuration);
            services.AddBot<DirectBot>(options =>
            {
                options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
            });
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles();
            app.UseStaticFiles();
            app.UseBotFramework();
        }
    }
}
```

### <a name="create-the-directbot-class"></a>Создание класса DirectBot

Класс DirectBot содержит большую часть логики для этого бота.

1. Щелкните правой кнопкой мыши **DirectLineBotSample** > **Добавить** > **Новый элемент**.

1. В **ASP.NET Core** щелкните **Класс**.

1. Введите **DirectBot.cs** в качестве имени и нажмите кнопку **Добавить**.

Замените содержимое файла DirectBot.cs следующим кодом:

```csharp
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace DirectLineBotSample
{
    public class DirectBot : IBot
    {
        public Task OnTurn(ITurnContext context)
        {
            // Respond to the various activity types.
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                    // Respond to the incoming text message.
                    RespondToMessage(context);
                    break;

                case ActivityTypes.ConversationUpdate:
                    break;

                case ActivityTypes.ContactRelationUpdate:
                    break;

                case ActivityTypes.Typing:
                    break;

                case ActivityTypes.DeleteUserData:
                    break;
            }

            return Task.CompletedTask;
        }

        /// <summary>
        /// Responds to the incoming message by either sending a hero card, an image, 
        /// or echoing the user's message.
        /// </summary>
        /// <param name="context">The context of this conversation.</param>
        private void RespondToMessage(ITurnContext context)
        {
            switch (context.Activity.Text.Trim().ToLower())
            {
                case "hi":
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    context.SendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case "show me a hero card":
                    // Create the hero card.
                    HeroCard heroCard = new HeroCard()
                    {
                        Title = "Sample Hero Card",
                        Text = "Displayed in the DirectLine client"
                    };

                    // Attach the hero card to a new activity.
                    context.SendActivity(MessageFactory.Attachment(heroCard.ToAttachment()));
                    break;

                case "send me a botframework image":
                    // Create the image attachment.
                    Attachment imageAttachment = new Attachment()
                    {
                        ContentType = "image/png",
                        ContentUrl = "https://docs.microsoft.com/en-us/bot-framework/media/how-it-works/architecture-resize.png",
                    };

                    // Attach the image attachment to a new activity.
                    context.SendActivity(MessageFactory.Attachment(imageAttachment));
                    break;

                default:
                    // No command was encountered. Echo the user's message.
                    context.SendActivity($"You said \"{context.Activity.Text}\"");
                    break;
            }
        }
    }
}
```

**Примечание**. Выход из файла Program.cs не требуется.

Чтобы убедиться, что все в порядке, нажмите клавишу **F6**, чтобы создать проект. Не должно возникать никаких предупреждений или ошибок.

### <a name="publish-the-bot-from-visual-studio"></a>Публикация бота из Visual Studio

1. В обозревателе решений в Visual Studio щелкните правой кнопкой мыши проект **​​DirectLineBotSample** и выберите **Назначить запускаемым проектом**.

    ![Страница публикации в Visual Studio](media/bot-builder-howto-direct-line/visual-studio-publish-page.png)

1. Щелкните правой кнопкой мыши **DirectLineBotSample** и нажмите **Опубликовать**. Появится страница Visual Studio **Опубликовать**.

1. Если у вас уже есть профиль публикации этого бота, выберите его и нажмите кнопку **​​Опубликовать**, а затем перейдите к следующему разделу.

1. Чтобы создать профиль публикации, щелкните значок **Служба приложений Microsoft Azure**.

1. Выберите **Создать**.

1. Нажмите кнопку **Опубликовать** . Откроется диалоговое окно **Создать службу приложений**.

    ![Диалоговое окно "Создать службу приложений"](media/bot-builder-howto-direct-line/create-app-service-dialog.png)

    - В качестве имени приложения выберите имя, которое вы можете найти позже. Например, вы можете добавить свое имя электронной почты в начало имени приложения.

    - Убедитесь, что вы используете правильную подписку.

    - Убедитесь, что вы используете правильную группу ресурсов. Группу ресурсов можно найти в колонке **Обзор** вашего бота. **Примечание**. Неправильную группу ресурсов сложно исправить.

    - Убедитесь, что вы используете правильный план службы приложений.
    
1. Нажмите кнопку **Создать** . Visual Studio начнет развертывание вашего бота.

После того как ваш бот будет опубликован, откроется браузер с конечной точкой URL вашего бота.

### <a name="create-bot-channels-registration-bot-on-microsoft-azure"></a>Создание бота службы "Регистрация каналов бота" в Microsoft Azure

Бот Direct Line может быть размещен на любой платформе. В этом примере бот будет размещен в Microsoft Azure. 

Чтобы создать бот в Microsoft Azure, сделайте следующее:

1. На портале Microsoft Azure щелкните **Создайте ресурс**, затем выполните поиск службы "Регистрация каналов ботов".

1. Нажмите кнопку **Создать**. Появится колонка службы "Регистрация каналов бота".

    ![Колонка службы "Регистрация каналов бота" с полями имени бота, подписки, группы ресурсов, местоположения, ценовой категории, конечной точки обмена сообщениями и другими полями.](media/bot-builder-howto-direct-line/bot-service-registration-blade.png)

1. В колонке службы "Регистрация каналов бота" введите **имя бота**, **подписку**, **группу ресурсов**, **местоположение** и **ценовую категорию**.

1. Оставьте поле **Messaging endpoint** (Конечная точка обмена сообщениями) пустым. Это значение будет заполнено позже.

1. Щелкните **Microsoft App ID and password** (Идентификатор и пароль приложения Майкрософт),а затем щелкните **Auto create App ID and password** (Автоматическое создание идентификатора и пароля приложения).

1. Установите флажок **Закрепить на панели мониторинга**.

1. Нажмите кнопку **Создать** .

Развертывание займет несколько минут и появится на панели управления.

### <a name="update-the-appsettingsjson-file"></a>Обновление файла appsettings.json

1. Щелкните бот на панели управления или перейдите к новому ресурсу, щелкнув **Все ресурсы** и найдя имя службы "Регистрация каналов ботов".

1. Щелкните **Обзор**.

1. Скопируйте имя группы ресурсов в строку **botId** в файле **Program.cs** проекта **DirectLineClientSample**.

1. Щелкните **Параметры**, затем щелкните **Управление** рядом с полем **Microsoft App ID** (Идентификатор приложения Майкрософт).

1. Скопируйте идентификатор приложения и вставьте его в поле **MicrosoftAppId** файла **appsettings.json**.

1. Нажмите кнопку **​​Создать новый пароль**.

1. Скопируйте новый пароль и вставьте его в поле **MicrosoftAppPassword** файла **appsettings.json**.

### <a name="set-the-messaging-endpoint"></a>Настройка конечной точки обмена сообщениями

Чтобы добавить конечную точку для вашего бота, сделайте следующее:

1. Скопируйте URL-адрес из всплывающего окна браузера, которое появится после публикации вашего бота.

    ![Колонка параметров настроек службы "Регистрация каналов ботов", выделение конечной точки обмена сообщениями](media/bot-builder-howto-direct-line/bot-channels-registration-settings.png)

1. Откройте колонку **Параметры** вашего бота.

1. Вставьте адрес в поле **Messaging endpoint** (Конечная точка обмена сообщениями).

1. Измените адрес так, чтобы он начинался с "https://" и заканчивался на "/api/messages". Например, если адрес, скопированный из браузера, это http://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net, измените его на https://v-royhar-dlbot-directlinebotsample20180329044602.azurewebsites.net/api/messages.

1. Нажмите кнопку **​​Сохранить** на колонке параметров.

**Примечание**. Если публикация не удалась, вам может потребоваться остановить ваш бот в Azure, прежде чем вы сможете публиковать изменения в нем.

1. Найдите имя вашей службы приложений (оно находится между "https://" и ".azurewebsites.net".

1. Найдите этот ресурс на портале Azure на вкладке "Все ресурсы".

1. Щелкните ресурс службы приложений. Появится колонка службы приложений.

1. Нажмите кнопку **​​Стоп** в колонке службы приложений.

1. В Visual Studio опубликуйте бот еще раз.

1. Нажмите кнопку **Пуск** в колонке службы приложений.

### <a name="test-your-bot-in-webchat"></a>Тестирование бота в веб-чате

Чтобы убедиться, что ваш бот работает, проверьте его в веб-чате:

1. В колонке службы "Регистрация каналов бота" вашего бота щелкните **Параметры**, а затем **Test in Web Chat** (Тестирование в веб-чате).

1. Введите "Привет". Бот должен ответить приветственным сообщением.

1. Введите "показать карточку для имиджевого баннера". Бот должен отобразить карточку для имиджевого баннера.

1. Введите "отправить изображение botframework". Бот должен отобразить изображение из документации Bot Framework.

1. Введите что-нибудь еще, и бот должен ответить "Вы сказали" и ваше сообщение в кавычках.

### <a name="troubleshooting"></a>Устранение неполадок

Если ваш бот не работает, проверьте или повторно введите идентификатор и пароль приложения Майкрософт. Даже если вы скопировали его ранее, сопоставьте идентификатор приложения Майкрософт в колонке настроек вашей службы "Регистрация каналов бота" со значением в поле **MicrosoftAppId** в файле appsettings.json.

Проверьте свой пароль или создайте и используйте новый: 

1. Щелкните **Управление** рядом с полем **Microsoft App ID** (Идентификатор приложения Майкрософт) в колонке службы "Регистрация каналов бота".

1. Войдите на портал регистрации приложений.

1. Убедитесь, что первые три буквы вашего пароля совпадают с полем **MicrosoftAppPassword** в файле **appsettings.json**. 

1. Если значения не совпадают, создайте пароль и сохраните это значение в поле **MicrosoftAppPassword** в файле **appsettings.json**.

# <a name="javascripttabjscreatebot"></a>[JavaScript](#tab/jscreatebot)

### <a name="create-web-app-bot-on-microsoft-azure"></a>Создание бота веб-приложения в Microsoft Azure

Чтобы создать бот в Microsoft Azure, сделайте следующее: 

1. На портале Microsoft Azure щелкните **Создайте ресурс**, затем выполните поиск "Бот веб-приложения".

1. Нажмите кнопку **Создать**. Откроется колонка бота веб-приложения.

![Регистрация бота веб-приложения](media/bot-builder-howto-direct-line/web-app-bot-registration.png)

1. В колонке бота веб-приложения введите **имя бота**, **подписку**, **группу ресурсов**, **местоположение**, **ценовую категорию** и **имя приложения**.

1. Выберите шаблон бота, который вы хотите использовать. **Пакет SDK версии 4** и **язык SDK Node.js**. 

1. Выберите основной шаблон предварительной версии 4. 

1. Выберите **план обслуживания приложения или местоположение**, **службу хранилища Azure** и **местоположение Application Insights**.

1. Установите флажок "Закрепить на панели мониторинга".

1. Нажмите кнопку "Создать".

1. Подождите, пока ваш бот будет развернут. Так как вы установили флажок "Закрепить на панели мониторинга", ваш бот появится на панели.

### <a name="edit-your-direct-line-bot"></a>Изменение бота Direct Line

Теперь добавим еще несколько функций для бота echo.

1. В колонке веб-приложения нажмите кнопку "Создать". 

1. Щелкните "Скачать ZIP-файл". 

1. Извлеките ZIP-файл в локальный каталог.

1. Перейдите в извлеченную папку и откройте исходные файлы в предпочитаемом интерфейсе IDE.

1. В файле app.js скопируйте и вставьте приведенный ниже код.

```javascript
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License.

const { BotFrameworkAdapter, MessageFactory, CardFactory } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Responds to the incoming message by either sending a hero card, an image, 
// or echoing the user's message.

// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const text = (context.activity.text || '').trim().toLowerCase()

            switch(text){
                case 'hi':
                case "hello":
                case "help":
                    // Send the user an instruction message.
                    return context.sendActivity("Welcome to the Bot to showcase the DirectLine API. " +
                    "Send \"Show me a hero card\" or \"Send me a BotFramework image\" to see how the " +
                    "DirectLine client supports custom channel data. Any other message will be echoed.");
                    break;

                case 'show me a hero card':
                    // Create the hero card.
                    const message = MessageFactory.attachment(
                        CardFactory.heroCard(   
                        'Sample Hero Card', //cards title
                        'Displayed in the DirectLine client' //cards text
                        )
                    );
                    return context.sendActivity(message);
                    break;
                    
                case 'send me a botframework image':
                    // Create the image attachment.
                    const imageOrVideoMessage = MessageFactory.contentUrl('https://docs.microsoft.com/en-us/azure/bot-service/media/how-it-works/architecture-resize.png', 'image/png')
                    return context.sendActivity(imageOrVideoMessage);
                    break;
                
                default:
                    // No command was encountered. Echo the user's message.
                    return context.sendActivity(`You said ${context.activity.text}`);
                    break;
                    
            }
        }
    });
});

```

Когда будете готовы, можно опубликовать источники в Azure. Выполните следующие шаги, чтобы узнать, как опубликовать бот обратно в [Azure](../bot-service-build-download-source-code.md).

---


## <a name="create-the-console-client-app"></a>Создание консольного клиента для приложения

# <a name="ctabcsclientapp"></a>[C#](#tab/csclientapp)

Клиентское приложение консоли работает в двух потоках. Основной поток принимает входные данные пользователя и отправляет сообщения боту. Вторичный поток проверяет бот один раз в секунду, чтобы получить любые сообщения от него, а затем отображает полученные сообщения.

Чтобы создать консольный проект, сделайте следующее:

1. Щелкните правой кнопкой мыши решение **DirectLineBotSample** в обозревателе решений.

1. Выберите **Добавить** > **Новый проект**.

1. В разделе **Visual C#** > **Классический рабочий стол Windows* выберите **Консольное приложение (.NET Framework)**.
 
    **Примечание**. Не выбирайте **Консольное приложение (.NET Core)**.

1. Введите имя **DirectLineClientSample**.

1. Последовательно выберите **ОК**.

### <a name="add-the-nuget-packages-to-the-console-app"></a>Добавление пакетов NuGet в консольное приложение

1. Щелкните правой кнопкой мыши **References**(Ссылки).

1. Выберите **Управление пакетами NuGet...**.

1. Щелкните **Обзор**. Убедитесь, что установлен флажок **Включить предварительные выпуски**.

1. Найдите и установите следующие пакеты NuGet:
    - Microsoft.Bot.Connector.DirectLine (версия 3.0.2)
    - Newtonsoft.Json.

### <a name="edit-the-programcs-file"></a>Изменение файла Program.cs

Замените содержимое файла DirectLineClientSample **Program.cs** следующим образом:

```csharp
using System;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.Bot.Connector.DirectLine;
using Newtonsoft.Json;

namespace DirectLineClientSample
{
    class Program
    {
        // ************
        // Replace the following values with your Direct Line secret and the name of your bot resource ID.
        //*************
        private static string directLineSecret = "*** Replace with Direct Line secret ***";
        private static string botId = "*** Replace with the resource name of your bot ***";

        // This gives a name to the bot user.
        private static string fromUser = "DirectLineClientSampleUser";

        static void Main(string[] args)
        {
            StartBotConversation().Wait();
        }


        /// <summary>
        /// Drives the user's conversation with the bot.
        /// </summary>
        /// <returns></returns>
        private static async Task StartBotConversation()
        {
            // Create a new Direct Line client.
            DirectLineClient client = new DirectLineClient(directLineSecret);

            // Start the conversation.
            var conversation = await client.Conversations.StartConversationAsync();

            // Start the bot message reader in a separate thread.
            new System.Threading.Thread(async () => await ReadBotMessagesAsync(client, conversation.ConversationId)).Start();

            // Prompt the user to start talking to the bot.
            Console.Write("Type your message (or \"exit\" to end): ");

            // Loop until the user chooses to exit this loop.
            while (true)
            {
                // Accept the input from the user.
                string input = Console.ReadLine().Trim();

                // Check to see if the user wants to exit.
                if (input.ToLower() == "exit")
                {
                    // Exit the app if the user requests it.
                    break;
                }
                else
                {
                    if (input.Length > 0)
                    {
                        // Create a message activity with the text the user entered.
                        Activity userMessage = new Activity
                        {
                            From = new ChannelAccount(fromUser),
                            Text = input,
                            Type = ActivityTypes.Message
                        };

                        // Send the message activity to the bot.
                        await client.Conversations.PostActivityAsync(conversation.ConversationId, userMessage);
                    }
                }
            }
        }


        /// <summary>
        /// Polls the bot continuously and retrieves messages sent by the bot to the client.
        /// </summary>
        /// <param name="client">The Direct Line client.</param>
        /// <param name="conversationId">The conversation ID.</param>
        /// <returns></returns>
        private static async Task ReadBotMessagesAsync(DirectLineClient client, string conversationId)
        {
            string watermark = null;

            // Poll the bot for replies once per second.
            while (true)
            {
                // Retrieve the activity set from the bot.
                var activitySet = await client.Conversations.GetActivitiesAsync(conversationId, watermark);
                watermark = activitySet?.Watermark;

                // Extract the activies sent from our bot.
                var activities = from x in activitySet.Activities
                                 where x.From.Id == botId
                                 select x;

                // Analyze each activity in the activity set.
                foreach (Activity activity in activities)
                {
                    // Display the text of the activity.
                    Console.WriteLine(activity.Text);

                    // Are there any attachments?
                    if (activity.Attachments != null)
                    {
                        // Extract each attachment from the activity.
                        foreach (Attachment attachment in activity.Attachments)
                        {
                            switch (attachment.ContentType)
                            {
                                // Display a hero card.
                                case "application/vnd.microsoft.card.hero":
                                    RenderHeroCard(attachment);
                                    break;

                                // Display the image in a browser.
                                case "image/png":
                                    Console.WriteLine($"Opening the requested image '{attachment.ContentUrl}'");
                                    Process.Start(attachment.ContentUrl);
                                    break;
                            }
                        }
                    }

                    // Redisplay the user prompt.
                    Console.Write("\nType your message (\"exit\" to end): ");
                }

                // Wait for one second before polling the bot again.
                await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
            }
        }


        /// <summary>
        /// Displays the hero card on the console.
        /// </summary>
        /// <param name="attachment">The attachment that contains the hero card.</param>
        private static void RenderHeroCard(Attachment attachment)
        {
            const int Width = 70;
            // Function to center a string between asterisks.
            Func<string, string> contentLine = (content) => string.Format($"{{0, -{Width}}}", string.Format("{0," + ((Width + content.Length) / 2).ToString() + "}", content));

            // Extract the hero card data.
            var heroCard = JsonConvert.DeserializeObject<HeroCard>(attachment.Content.ToString());

            // Display the hero card.
            if (heroCard != null)
            {
                Console.WriteLine("/{0}", new string('*', Width + 1));
                Console.WriteLine("*{0}*", contentLine(heroCard.Title));
                Console.WriteLine("*{0}*", new string(' ', Width));
                Console.WriteLine("*{0}*", contentLine(heroCard.Text));
                Console.WriteLine("{0}/", new string('*', Width + 1));
            }
        }
    }
}
```

Чтобы убедиться, что все в порядке, нажмите клавишу **F6**, чтобы создать проект. Не должно возникать никаких предупреждений или ошибок.

# <a name="javascripttabjsclientapp"></a>[JavaScript](#tab/jsclientapp)

### <a name="create-a-direct-line-client"></a>Создание клиента Direct Line 

Теперь, когда вы развернули бот веб-приложения, мы можем создать клиент Direct Line.

```
    md DirectLineClient
    cd DirectLineClient
    npm init
```

Затем установите эти пакеты из npm.

```
    npm install --save open
    npm install --save request
    npm install --save request-promise
    npm install --save swagger-client
```

Создайте файл **app.js**. Скопируйте и вставьте код, приведенный ниже, в этот файл.

На следующем шаге мы получим directLineSecret.
```javascript
var Swagger = require('swagger-client');
var open = require('open');
var rp = require('request-promise');

// config items
var pollInterval = 1000;
// Change the Direct Line Secret to your own 
var directLineSecret = 'your secret here';
var directLineClientName = 'DirectLineClient';
var directLineSpecUrl = 'https://docs.botframework.com/en-us/restapi/directline3/swagger.json';

var directLineClient = rp(directLineSpecUrl)
    .then(function (spec) {
        // client
        return new Swagger({
            spec: JSON.parse(spec.trim()),
            usePromise: true
        });
    })
    .then(function (client) {
        // add authorization header to client
        client.clientAuthorizations.add('AuthorizationBotConnector', new Swagger.ApiKeyAuthorization('Authorization', 'Bearer ' + directLineSecret, 'header'));
        return client;
    })
    .catch(function (err) {
        console.error('Error initializing DirectLine client', err);
    });

// once the client is ready, create a new conversation
directLineClient.then(function (client) {
    client.Conversations.Conversations_StartConversation()                          // create conversation
        .then(function (response) {
            return response.obj.conversationId;
        })                            // obtain id
        .then(function (conversationId) {
            sendMessagesFromConsole(client, conversationId);                        // start watching console input for sending new messages to bot
            pollMessages(client, conversationId);                                   // start polling messages from bot
        })
        .catch(function (err) {
            console.error('Error starting conversation', err);
        });
});

// Read from console (stdin) and send input to conversation using DirectLine client
function sendMessagesFromConsole(client, conversationId) {
    var stdin = process.openStdin();
    process.stdout.write('Command> ');
    stdin.addListener('data', function (e) {
        var input = e.toString().trim();
        if (input) {
            // exit
            if (input.toLowerCase() === 'exit') {
                return process.exit();
            }

            // send message
            client.Conversations.Conversations_PostActivity(
                {
                    conversationId: conversationId,
                    activity: {
                        textFormat: 'plain',
                        text: input,
                        type: 'message',
                        from: {
                            id: directLineClientName,
                            name: directLineClientName
                        }
                    }
                }).catch(function (err) {
                    console.error('Error sending message:', err);
                });

            process.stdout.write('Command> ');
        }
    });
}

// Poll Messages from conversation using DirectLine client
function pollMessages(client, conversationId) {
    console.log('Starting polling message for conversationId: ' + conversationId);
    var watermark = null;
    setInterval(function () {
        client.Conversations.Conversations_GetActivities({ conversationId: conversationId, watermark: watermark })
            .then(function (response) {
                watermark = response.obj.watermark;                                 // use watermark so subsequent requests skip old messages
                return response.obj.activities;
            })
            .then(printMessages);
    }, pollInterval);
}

// Helpers methods
function printMessages(activities) {
    if (activities && activities.length) {
        // ignore own messages
        activities = activities.filter(function (m) { return m.from.id !== directLineClientName });

        if (activities.length) {
            process.stdout.clearLine();
            process.stdout.cursorTo(0);

            // print other messages
            activities.forEach(printMessage);

            process.stdout.write('Command> ');
        }
    }
}

function printMessage(activity) {
    if (activity.text) {
        console.log(activity.text);
    }

    if (activity.attachments) {
        activity.attachments.forEach(function (attachment) {
            switch (attachment.contentType) {
                case "application/vnd.microsoft.card.hero":
                    renderHeroCard(attachment);
                    break;

                case "image/png":
                    console.log('Opening the requested image ' + attachment.contentUrl);
                    open(attachment.contentUrl);
                    break;
            }
        });
    }
}

function renderHeroCard(attachment) {
    var width = 70;
    var contentLine = function (content) {
        return ' '.repeat((width - content.length) / 2) +
            content +
            ' '.repeat((width - content.length) / 2);
    }

    console.log('/' + '*'.repeat(width + 1));
    console.log('*' + contentLine(attachment.content.title) + '*');
    console.log('*' + ' '.repeat(width) + '*');
    console.log('*' + contentLine(attachment.content.text) + '*');
    console.log('*'.repeat(width + 1) + '/');
}
```

---

## <a name="configure-the-direct-line-channel"></a>Настройки канала Direct Line

При добавлении канала Direct Line этот бот становится ботом Direct Line.

Чтобы настроить канал Direct Line, сделайте следующее:

![Колонка подключения к каналам с подсвеченной кнопкой канала Direct Line.](media/bot-builder-howto-direct-line/direct-line-channel-button.png)

1. В колонке **Bot Channels Registration** (Регистрация каналов бота) щелкните **Каналы**. Появится колонка **Connect to Channels** (Подключение к каналам).

    ![Колонка настройки Direct Line с двумя секретными ключами.](media/bot-builder-howto-direct-line/configure-direct-line.png)

1. В колонке **Connect to Channels** (Подключение к каналам) нажмите кнопку **Configure Direct Line channel** (Настройка канала Direct Line). 

1. Если не установлен флажок **3.0**, установите его.

1. Щелкните **Показать**, по крайней мере, для одного секретного ключа.

1. Скопируйте один из секретных ключей и вставьте его в свое клиентское приложение в строку **​directLineSecret**.

1. Нажмите кнопку Done (Готово).

## <a name="run-the-client-app"></a>Запуск клиентского приложения

# <a name="ctabcsrunclient"></a>[C#](#tab/csrunclient)

Теперь ваш бот готов к общению с клиентским приложением Direct Line. Чтобы запустить консольное приложение, выполните следующие действия:

1. В Visual Studio щелкните правой кнопкой мыши проект **DirectLineClientSample** и выберите **Назначить запускаемым проектом**.

1. Изучите файл **Program.cs** в проекте **DirectLineClientSample**.

    - Убедитесь, что один секретный код Direct Line находится в строке **directLineSecret**.

1. Если строка **directLineSecret** неверна, перейдите на портал Azure, щелкните бот Direct Line, выберите **Каналы**, **Configure Direct Line channel** (Настройка канала Direct Line) (или **Изменение**), отобразите ключ, а затем скопируйте этот ключ в строку **directLineSecret**.

    - Убедитесь, что группа ресурсов находится в строке **botId**.

1. Если это не так, в колонке **Обзор** скопируйте и вставьте **группу ресурсов** в строку **botId**.

1. Нажмите клавишу F5 или запустите отладку.

Запустится консольное клиентское приложение. Чтобы протестировать приложение: 

1. Введите"Привет". Бот должен отобразить "Вы сказали "Привет"".

1. Введите "показать карточку для имиджевого баннера". Бот должен отобразить карточку для имиджевого баннера.

1. Введите "отправить изображение botframework". Бот должен запустить браузер, чтобы отобразить изображение из документации Bot Framework.

1. Введите что-нибудь еще, и бот должен ответить "Вы сказали" и ваше сообщение в кавычках.

# <a name="javascripttabjsrunclient"></a>[JavaScript](#tab/jsrunclient)

Чтобы запустить пример, вам нужно запустить приложение DirectLineClient.

1. Открытие консоли CMD и непрерывного развертывания в каталоге DirectLineClient

1. Запустите `node app.js`

Чтобы проверить тип пользовательских сообщений в консоли клиента, введите "Показать карточку для имиджевого баннера" или "Отправить изображение botframework", и вы увидите следующий результат.

![результат](media/bot-builder-howto-direct-line/outcome.png)

---


### <a name="next-steps"></a>Дополнительная информация

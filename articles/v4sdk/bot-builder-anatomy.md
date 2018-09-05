---
title: Структура бота | Документация Майкрософт
description: Подробное описание разных частей бота и их работы
keywords: ''
author: ivorb
ms.author: ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/25/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 238be22eff746edff30a5446d20991dfaedc945a
ms.sourcegitcommit: 44f100a588ffda19c275b118f4f97029f12d1449
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/26/2018
ms.locfileid: "42928292"
---
# <a name="anatomy-of-a-bot"></a>Структура бота

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

В самом [общем понимании](bot-builder-basics.md) бот — это приложение для общения, с которым пользователи взаимодействуют с помощью сообщений. Бот создается на основе базовой структуры веб-приложения на соответствующем языке с помощью пакета SDK для Bot Framework.

Бот состоит из нескольких компонентов, которые позволяют [отправлять сообщения](./bot-builder-howto-send-messages.md) и отслеживать [состояние](./bot-builder-storage-concept.md). Взаимодействовать с ботом можно с помощью [эмулятора](../bot-service-debug-emulator.md), [WebChat](../bot-service-manage-test-webchat.md) или одного из [канала](bot-concepts.md) в рабочей среде. 

В этом руководстве мы подробно рассмотрим простую программу Echo Bot и проанализируем все ее части, которые обеспечивают работу бота.

## <a name="files-created-for-echo-bot"></a>Файлы, создаваемые для Echo Bot

Для разных языков программирования создаются разные файлы для Echo Bot (в этом примере мы используем имя **BasicEcho**). Эти файлы бывают трех типов: системные файлы, вспомогательные элементы и ядро бота. В следующей таблице перечислены основные файлы для C# и JavaScript.

| C# | JavaScript |
| --- | --- |
| `Properties > launchSettings.json` <br> `wwwroot > default.htm` <br> `appsettings.json` <br> `BasicEcho.bot` <br> `EchoBot.cs` <br> `EchoState.cs` <br> `Program.cs` <br> `readme.md` <br> `Startup.cs` | `.env` <br> `app.js` <br> `package.json` <br> `README.md` <br> `basicEcho.bot` |

Ниже мы подробно рассмотрим некоторые ключевые компоненты в этих файлах по типам. Некоторые из этих файлов содержат собственный набор элементов, которые включают как уникальные для бота, так и общие программные библиотеки. Эти элементы содержат набор необходимых и дополнительных функций, которые могут понадобиться вам в приложении. Мы не будет подробно рассматривать эти элементы, но если вам интересно, в Visual Studio можете просмотреть их определения и узнать, в какие пространства имен они входят.

## <a name="system-section"></a>Системная часть

Системная часть содержит все элементы, необходимые для того, чтобы бот работал как стандартное приложение. В ней содержатся файлы конфигурации, адаптера и некоторые файлы JSON. Эти элементы можно (и часто нужно) оставить без изменений, кроме некоторых файлов конфигурации, в которые, возможно, потребуется указать определенные данные.

# <a name="ctabcs"></a>[C#](#tab/cs)

Бот — это разновидность исполняющей веб-среды [веб-ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/?view=aspnetcore-2.1). В примерах из статьи [Основы ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x) вы увидите похожий код в таких файлах, как appsettings.json, Program.cs и Startup.cs, которые описаны ниже. Эти файлы являются обязательными для всех веб-приложений и не зависят от конкретного бота. Мы не копировали код некоторых этих файлов для примера, но вы увидите его при запуске бота.

### <a name="appsettingsjson"></a>appsettings.json

Этот файл просто содержит сведения о параметрах для бота, как правило, идентификатор приложения и пароль в основном формате JSON.  При тестировании с использованием [эмулятора](../bot-service-debug-emulator.md) эти сведения не потребуются и их можно не указывать. Но они требуются в рабочей среде.

### <a name="programcs"></a>Program.cs

Этот файл является обязательным для работы веб-приложения ASP.NET. В нем указывается класс `Startup` для запуска.

### <a name="startupcs"></a>Startup.cs

**Startup.cs** содержит другие интересные части, которые мы рассмотрим позже. Но сейчас речь идет о системных файлах.

Конструктор для `Startup` позволяет задать параметры приложения и переменные среды, а также создать конфигурацию для веб-приложения.

Метод `Configure` указывается в конце конфигурации приложения вместе с информацией о том, что приложение использует Bot Framework и несколько других файлов. Для всех ботов, использующих Bot Framework, требуется выполнить вызов конфигурации.

```cs
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Bot.Builder.TraceExtensions;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

namespace BasicEcho
{
    public class Startup
    {
        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
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

        // ...
        // Definition of ConfigureServices covered in the next section
        // ...

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseDefaultFiles()
                .UseStaticFiles()
                .UseBotFramework();
        }
    }
}

```

### <a name="launchsettingsjson-and-readmemd"></a>launchSettings.json и readme.md

Файл **launchSettings.json** просто содержит некоторые заданные настройки для веб-приложений, а **readme.md** — только одну строку с описанием Echo Bot. Содержимое этих файлов не является важным для понимания структуры бота. 

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В системной части, в основном содержатся файлы **package.json**, файл с расширением **.env**, **app.js** и **README.md**. Мы не копировали код некоторых файлов для примера, но вы увидите его при запуске бота.

### <a name="packagejson"></a>package.json

В файле **package.json** указываются зависимости и их связанные версии для бота. Все они настраиваются и системой и согласно шаблону.

### <a name="env-file"></a>Файл ENV

В файле **ENV** указываются сведения о конфигурации для вашего бота, в том числе и номер порта, идентификатор приложения и пароль. Если вы применяете некоторые технологии или используете этот бот в рабочей среде, нужно добавить в эту конфигурацию определенные ключи или URL-адрес. Сейчас для Echo Bot не нужно добавлять такие сведения. Можете не указывать идентификатор приложения и пароль на этом этапе. 

Чтобы использовать файл конфигурации **ENV**, нужно добавить в шаблон пакет.  Сначала получите пакет `dotenv` из npm:

`npm install dotenv`

Затем добавьте следующую строку в код бота с другими необходимыми библиотеками:

```javascript
const dotenv = require('dotenv');
```

### <a name="appjs"></a>app.js

В верхней части файла `app.js` указаны сервер и адаптер, которые позволяют боту общаться с пользователем и отправлять ответы. Сервер будет ожидать передачи данных на порту, указанном в файле конфигурации **ENV**, или вернется к порту 3978, чтобы установить подключение к эмулятору. Адаптер будет выступать в качестве проводника для вашего бота, управлять входящими и исходящими сообщениями, аутентификацией и т. д. 

```javascript
// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});
``` 

### <a name="readmemd"></a>README.md

Файл **README.md** содержит полезные сведения по некоторым аспектам создания бота, например описание основной структуре бота или объяснение для *Dialogs*, но не является важным для понимания бота.

---


## <a name="helper-items"></a>Вспомогательные элементы

Как и для любой программы вспомогательные методы и другие определения позволяют упростить наш код. Под эту категорию попадают две части нашего бота: файл `.bot`, который является важным для нашего бота, но не для основной логики бота.

### <a name="bot-file"></a>Файл BOT

Файл с расширением `.bot` содержит такие сведения, как конечная точка, идентификатор приложения и пароль, которые используются для подключения [каналов](bot-concepts.md) к боту. Этот файл создается автоматически, когда вы создаете бот на основе шаблона. Но вы можете создавать свой файл с помощью эмулятора или других средств.

Содержимое файла должно соответствовать приведенному ниже, только имя бота будет отличаться от *BasicEcho*. Возможно, потребуется изменить и обновить эти сведения в зависимости от того, как будет использоваться ваш бот. Но сейчас для запуска Echo Bot не требуется вносить изменения.

```json
{
  "name": "BasicEcho",
  "secretKey": "",
  "services": [
    {
      "appId": "",
      "id": "http://localhost:3978/api/messages",
      "type": "endpoint",
      "appPassword": "",
      "endpoint": "http://localhost:3978/api/messages",
      "name": "BasicEcho"
    }
  ]
}
```

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="echostatecs"></a>EchoState.cs

**EchoState.cs** содержит простой класс, который бот использует для сохранения информации о текущем состоянии. Он содержит только элемент `int`, который мы используем для увеличения значений счетчика реплик. Мы подробнее рассмотрим состояние в следующем разделе, но сейчас важно только понять, что `EchoState` — это наш класс, содержащий счетчик реплик.

```cs
namespace BasicEcho
{
    /// <summary>
    /// Class for storing conversation state.
    /// </summary>
    public class EchoState
    {
        public int TurnCount { get; set; } = 0;
    }
}
```

### <a name="defaulthtm"></a>default.htm

**Default.htm** — это написанная на `html` веб-страница, которая отображается при запуске бота. На странице отображаются полезные сведения о том, как подключиться к боту и взаимодействовать с ним. Но содержимое страницы не влияет на поведение бота. Всплывающее окно с кодом появится при запуске бота.

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="required-libraries"></a>Обязательные библиотеки

В начале файла `app.js` вы найдете набор обязательных модулей или библиотек. Эти модули предоставляют доступ к набору функций, которые, возможно, потребуется добавить в приложение. 

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');
```

---

## <a name="core-of-the-bot"></a>Ядро бота

А теперь перейдем к самому интересному. В ядре бота определяется, как бот взаимодействует с пользователем, включая ПО промежуточного слоя и логику бота.

# <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="middleware"></a>ПО промежуточного слоя

Файл **Startup.cs** содержит один метод с именем `ConfigureServices`. Этот метод позволяет настроить поставщика учетных данных и добавить ПО промежуточного слоя.

Добавляемое на этом этапе [ПО промежуточного слоя](bot-builder-concept-middleware.md) выполняет две функции. Первая — оно выступает в качестве обработчика исключений, чтобы корректно обрабатывать сбои бота. Эта функция определена в коде как лямбда-выражение, которое просто выводит исключение в терминал и сообщает пользователю, что произошла ошибка.

Вторая — ПО промежуточного слоя хранит информацию о состоянии, используя определенный ранее класс `MemoryStorage`. Это состояние определяется в виде свойства `ConversationState`, которое просто позволяет сохранить данные о состоянии диалога. Для этого свойства используется класс `EchoState`, который определен счетчиком реплик для хранения нужных сведений.

``` cs
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<EchoBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);

        // The CatchExceptionMiddleware provides a top-level exception handler for your bot.
        // Any exceptions thrown by other Middleware, or by your OnTurn method, will be 
        // caught here. To facilitate debugging, the exception is sent out, via Trace, 
        // to the emulator. Trace activities are NOT displayed to users, so in addition
        // an "Ooops" message is sent.
        options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
        {
            await context.TraceActivity("EchoBot Exception", exception);
            await context.SendActivity("Sorry, it looks like something went wrong!");
        }));

        // The Memory Storage used here is for local bot debugging only. When the bot
        // is restarted, anything stored in memory will be gone. 
        IStorage dataStore = new MemoryStorage();

        // The File data store, shown here, is suitable for bots that run on 
        // a single machine and need durable state across application restarts.

        // IStorage dataStore = new FileStorage(System.IO.Path.GetTempPath());

        // For production bots use the Azure Table Store, Azure Blob, or 
        // Azure CosmosDB storage provides, as seen below. To include any of 
        // the Azure based storage providers, add the Microsoft.Bot.Builder.Azure 
        // Nuget package to your solution. That package is found at:
        //      https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/

        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureTableStorage("AzureTablesConnectionString", "TableName");
        // IStorage dataStore = new Microsoft.Bot.Builder.Azure.AzureBlobStorage("AzureBlobConnectionString", "containerName");

        options.Middleware.Add(new ConversationState<EchoState>(dataStore));
    });
}
```

### <a name="bot-logic"></a>Логика бота

Основная логика бота определяется в рамках обработчика бота `OnTurn`. `OnTurn` указывается в переменной [context](./bot-builder-concept-activity-processing.md#turn-context) в качестве аргумента, который предоставляет сведения о входящем [действии](bot-builder-concept-activity-processing.md), сообщении и т. д.

Входящие действия могут быть [разных типов](../bot-service-activities-entities.md#activity-types), поэтому сначала мы проверяем, получил ли бот сообщение. Если это сообщение, мы создаем переменную состояния, содержащую сведения о текущем состоянии диалога с ботом. Затем увеличиваем значение переменной `int`, которая позволяет отслеживать число реплик, прежде чем вернуть пользователю это число с сообщением пользователя.

```cs
using System.Threading.Tasks;
using Microsoft.Bot;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Schema;

namespace BasicEcho
{
    public class EchoBot : IBot
    {
        /// <summary>
        /// Every Conversation turn for our EchoBot will call this method. In here
        /// the bot checks the Activty type to verify it's a message, bumps the 
        /// turn conversation 'Turn' count, and then echoes the users typing
        /// back to them. 
        /// </summary>
        /// <param name="context">Turn scoped context containing all the data needed
        /// for processing this conversation turn. </param>        
        public async Task OnTurn(ITurnContext context)
        {
            // This bot is only handling Messages
            if (context.Activity.Type == ActivityTypes.Message)
            {
                // Get the conversation state from the turn context
                var state = context.GetConversationState<EchoState>();

                // Bump the turn count. 
                state.TurnCount++;

                // Echo back to the user whatever they typed.
                await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
            }
        }
    }    
}
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="middleware"></a>ПО промежуточного слоя

В файле **app.js** для сохранения сведений о состоянии этого бота мы используем [по промежуточного слоя](bot-builder-concept-middleware.md) с `MemoryStorage`, определенным как один из обязательных модулей в начале файла `botbuilder`. Это состояние определяется в виде свойства `ConversationState`, которое просто позволяет сохранить данные о состоянии диалога. `ConversationState` позволяет сохранить необходимые сведения. В нашем случае это просто данные счетчика реплик в памяти.

```javascript
// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

### <a name="bot-logic"></a>Логика бота

Третий параметр в *processActivity* представляет собой обработчик функций, который вызывается для выполнения логики бота после того, как адаптер предварительно обработает полученное [действие](bot-builder-concept-activity-processing.md) и направит его через любое ПО промежуточного слоя. Переменную [context](./bot-builder-concept-activity-processing.md#turn-context), которая передается обработчику функций в виде аргумента, можно использовать для предоставления сведений о входящем действии, отправителе и получателе, канале, диалоге и т.д.

Сначала проверяется, получил ли бот сообщение. Если это не сообщение, мы отправляем обратно тип полученного действия. После этого создается переменная состояния, которая позволяет хранить информацию о диалоге с ботом. Затем назначаем переменной значение 0, если она не определена (это происходит при запуске бота), или увеличиваем это значение с каждым новым сообщением. Наконец, мы отправляем пользователю число и полученное от него сообщение.

```javascript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {

            // Get the conversation state
            const state = conversationState.get(context);

            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;

            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---

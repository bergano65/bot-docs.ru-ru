---
title: Подключение бота к Webex Teams | Документация Майкрософт
description: Узнайте, как настроить подключение бота к Webex через адаптер Slack.
keywords: bot adapter, Webex, Webex bot
author: garypretty
manager: kamrani
ms.topic: article
ms.author: gapretty
ms.service: bot-service
ms.date: 12/04/2019
ms.openlocfilehash: 590a4e3d4f2331580bb1fd823bdcd7a00f5efaab
ms.sourcegitcommit: 86495b597e55c94309a0c73fc1945a3393ddcbbf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/08/2020
ms.locfileid: "75757308"
---
# <a name="connect-a-bot-to-webex-teams-using-the-webex-adapter"></a>Подключение бота к Webex Teams с помощью адаптера Slack

Из этой статьи вы узнаете, как подключить бота к Webex с помощью адаптера, предоставляемого в пакете SDK.  В этой статье описано, как изменить пример EchoBot для его подключения к приложению Webex.

> [!NOTE]
> Ниже приводятся инструкции для реализации адаптера Slack на C#. Инструкции по использованию адаптера для JS, который входит в состав библиотек BotKit, см. в [документации по BotKit для Slack](https://botkit.ai/docs/v4/platforms/webex.html).

## <a name="prerequisites"></a>предварительные требования

* [Пример кода EchoBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/02.echo-bot).

* Доступ к команде Webex с достаточными разрешениями для создания приложений и управления ими в [https://developer.webex.com/my-apps](https://developer.webex.com/my-apps). Если у вас нет доступа к команде Webex, вы можете бесплатно создать учетную запись на странице https://www.webex.com.

## <a name="create-a-webex-bot-app"></a>Создание приложения для бота Webex

1. Войдите на [панель мониторинга разработчика Webex](https://developer.webex.com/my-apps) и нажмите кнопку Create a new app (Создать приложение).

2. На следующем экране выберите создание бота Webex, щелкнув Create a bot (Создать бот).

3. На следующем экране введите имя бота, имя пользователя и описание бота, а также выберите значок или передайте собственное изображение.

![Настройка бота](~/media/bot-service-adapter-connect-webex/create-bot.png)

Нажмите кнопку Add bot (Добавить бота).

4. На следующей странице вы получите маркер доступа для нового приложения Webex. Запишите его значение, так как оно потребуется при настройке бота.

![Настройка бота](~/media/bot-service-adapter-connect-webex/create-bot-settings.png)

## <a name="wiring-up-the-webex-adapter-in-your-bot"></a>Подключение адаптера Webex к боту

Прежде чем выполнять настройку приложения Webex, необходимо подключить адаптер Webex к боту.

### <a name="install-the-webex-adapter-nuget-package"></a>Установка пакета NuGet для адаптера Webex

Добавьте пакет NuGet [Microsoft.Bot.Builder.Adapters.Webex](https://www.nuget.org/packages/Microsoft.Bot.Builder.Adapters.Webex/). Подробные сведения об использовании NuGet см. в руководстве по [установке пакетов и управлении ими в Visual Studio](https://aka.ms/install-manage-packages-vs).

### <a name="create-a-webex-adapter-class"></a>Создание класса адаптера Webex

Создайте новый класс, который наследует класс ***WebexAdapter***. Этот класс будет использоваться в качестве адаптера для канала Webex. Он содержит встроенные возможности обработки ошибок (аналогично классу ***BotFrameworkAdapterWithErrorHandler***, который уже используется в этом примере для обработки запросов от службы Azure Bot).

```csharp
public class WebexAdapterWithErrorHandler : WebexAdapter
{
    public WebexAdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger)
    : base(configuration, logger)
    {
        OnTurnError = async (turnContext, exception) =>
        {
            // Log any leaked exception from the application.
            logger.LogError(exception, $"[OnTurnError] unhandled error : {exception.Message}");

            // Send a message to the user
            await turnContext.SendActivityAsync("The bot encountered an error or bug.");
            await turnContext.SendActivityAsync("To continue to run this bot, please fix the bot source code.");

            // Send a trace activity, which will be displayed in the Bot Framework Emulator
            await turnContext.TraceActivityAsync("OnTurnError Trace", exception.Message, "https://www.botframework.com/schemas/error", "TurnError");
        };
    }
}
```

### <a name="create-a-new-controller-for-handling-webex-requests"></a>Создание контроллера для обработки запросов Webex

Мы создадим контроллер, который будет выполнять запросы из приложения Webex, в новой конечной точке api/webex вместо api/messages, которая используется по умолчанию для запросов от каналов службы Azure Bot.  Добавив к боту дополнительную конечную точку, вы сможете с помощью одного бота принимать запросы одновременно от каналов (или других адаптеров) службы Bot и Webex.

```csharp
[Route("api/webex")]
[ApiController]
public class WebexController : ControllerBase
{
    private readonly WebexAdapter _adapter;
    private readonly IBot _bot;

    public WebexController(WebexAdapter adapter, IBot bot)
    {
        _adapter = adapter;
        _bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await _adapter.ProcessAsync(Request, Response, _bot);
    }
}
```

### <a name="inject-webex-adapter-in-your-bot-startupcs"></a>Включение адаптера Webex в файл Startup.cs бота

Добавьте следующую строку в метод ***ConfigureServices*** в файле Startup.cs, которая будет регистрировать адаптер Webex и делать его доступным для нового класса контроллера.  Описанные в следующем шаге параметры конфигурации применяются адаптером автоматически.

```csharp
services.AddSingleton<SlackAdapter, WebexAdapterWithErrorHandler>();
```

После добавления метод ***ConfigureServices*** должен выглядеть следующим образом.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the default Bot Framework Adapter (used for Azure Bot Service channels and emulator).
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkAdapterWithErrorHandler>();

    // Create the Slack Adapter
    services.AddSingleton<WebexAdapter, WebexAdapterWithErrorHandler>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

### <a name="add-webex-adapter-settings-to-your-bots-configuration-file"></a>Добавление параметров адаптера Webex в файл конфигурации бота

1. Добавьте четыре указанных ниже параметра в файл appSettings.json в проекте бота.

```json
  "WebexAccessToken": "",
  "WebexPublicAddress": "",
  "WebexSecret": "",
  "WebexWebhookName": ""
```

2. В параметре **WebexAccessToken** укажите маркер доступа бота Webex, который вы получили на предыдущих шагах при создании приложения бота Webex. Остальные три параметра пока оставьте пустыми. Информацию для них мы соберем на следующих шагах.

## <a name="complete-configuration-of-your-webex-app-and-bot"></a>Завершение настройки приложения Webex и бота

### <a name="create-and-update-a-webex-webhook"></a>Создание и обновление веб-перехватчика Webex

Теперь, когда вы создали приложение Webex и подключили адаптер в проекте бота, осталось лишь настроить веб-перехватчик Webex и направить его на конечную точку бота, а затем оформить для приложения подписку, чтобы бот получал нужные сообщения и вложения. Для этого процесса бот должен работать, чтобы решение Webex успешно проверило допустимость URL-адреса конечной точки.

1. Чтобы выполнить этот шаг, [разверните бота в Azure](https://aka.ms/bot-builder-deploy-az-cli) и запишите URL-адрес этого развертывания. Конечной точкой Webex для обмена сообщениями является URL-адрес бота, который совпадает с URL-адресом развернутого приложения (или конечной точки ngrok) с добавленным префиксом /api/webex (например, `https://yourbotapp.azurewebsites.net/api/webex`).

> [!NOTE]
> Если вы еще не готовы развернуть бота в Azure или вам нужна отладка бота с адаптером Webex, можно использовать средство [ngrok](https://www.ngrok.com) (оно должно быть уже установлено, если вы ранее использовали эмулятор Bot Framework), которое создаст туннель к запущенному в локальной среде боту и предоставит для него общедоступный URL-адрес. 
> 
> Если вы хотите создать туннель и получить для бота URL-адрес с помощью ngrok, выполните следующую команду в окне терминала. Здесь предполагается, что локальный бот работает на порту 3978. Если это не так, измените номера портов в команде.
> 
> ```
> ngrok.exe http 3978 -host-header="localhost:3978"
> ```

2. Перейдите по адресу [https://developer.webex.com/docs/api/v1/webhooks](https://developer.webex.com/docs/api/v1/webhooks).

3. Щелкните ссылку для метода PUT "https://api.ciscospark.com/v1/webhooks/{webhookId}" с описанием Update a Webhook (Обновить веб-перехватчик). В результате откроется форма, где вы можете отправить запрос на конечную точку.

![Настройка бота](~/media/bot-service-adapter-connect-webex/webex-webhook-put-endpoint.png)

4. Заполните форму следующими данными.

* Name (Имя): укажите имя веб-перехватчика, например Messages Webhook.
* Target URL (Целевой URL-адрес): это полный URL-адрес конечной точки бота для Webex, например https://yourbotapp.azurewebsites.net/api/webex).
* Secret (Секрет): здесь нужно указать секрет, который вы выбрали для защиты веб-перехватчика.
* Status (Состояние): оставьте здесь значение по умолчанию active (активное).

![Настройка бота](~/media/bot-service-adapter-connect-webex/webex-webhook-form.png)

5. Нажмите кнопку Run (Выполнить), чтобы создать веб-перехватчик и получить сообщение об успешном выполнении.

### <a name="complete-the-remaining-settings-in-your-bot-application"></a>Заполнение оставшихся параметров приложения бота

Заполните оставшиеся три параметра в файле appSettings.json для бота (значение **WebexAccessToken** вы уже указали на предыдущем шаге).

* WebexPublicAddress (это полный URL-адрес конечной точки бота для Webex).
* WebexSecret (секрет, указанный при создании веб-перехватчика на предыдущем шаге).
* WebexWebhookName (имя веб-перехватчика, которое вы указали на предыдущем шаге).

## <a name="re-deploy-your-bot-in-your-webex-team"></a>Повторное развертывание бота в команде Webex

Теперь, когда вы завершили настройку параметров бота в файле appSettings.json, следует повторно развернуть бота или просто перезапустить его, если вы используете туннель ngrok к локальной конечной точке.  Теперь настройка приложения Webex и бота завершена.  
Вы можете войти в команду Webex по адресу [https://www.webex.com](https://www.webex.com) и начать чат с ботом, отправляя ему сообщения точно так же, как другому человеку.

![Настройка бота](~/media/bot-service-adapter-connect-webex/webex-contact-person.png)

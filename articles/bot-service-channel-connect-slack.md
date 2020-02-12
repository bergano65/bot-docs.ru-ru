---
title: Подключение бота к Slack (служба Azure Bot)
description: Сведения о настройке подключения бота к Slack.
keywords: connect a bot, bot channel, Slack bot, Slack messaging app, slack adapter
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/09/2019
ms.openlocfilehash: 3147e202a615e29d51f1e3fa3a9d5d70ed54fe83
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071742"
---
# <a name="connect-a-bot-to-slack"></a>Подключение бота к Slack

Существует два способа настроить приложение Slack для обмена сообщениями:
- подключение бота с помощью портала службы Azure Bot;
- использование адаптера Slack.

## <a name="azure-bot-service-portaltababs"></a>[Портал службы Azure Bot](#tab/abs)
## <a name="create-a-slack-application-for-your-bot"></a>Создание приложения Slack для бота

Войдите в [Slack](https://slack.com/signin) и перейдите на канал для [создания приложения Slack](https://api.slack.com/apps).

![Настройка бота](~/media/channels/slack-NewApp.png)

## <a name="create-an-app-and-assign-a-development-slack-team"></a>Создание приложения и назначение группы разработки Slack

Введите имя приложения и выберите группу разработчиков Slack. Если вы еще не является членом команды разработчиков Slack, [создайте ее или присоединитесь к ней](https://slack.com/).

![Создание приложения](~/media/channels/slack-CreateApp.png)

Нажмите кнопку **Create App** (Создать приложение). Slack создаст приложение, а также идентификатор и секрет клиента.

## <a name="add-a-new-redirect-url"></a>Добавление нового URL-адреса перенаправления

Далее добавьте новый URL-адрес перенаправления.

1. Выберите вкладку **OAuth & Permissions** (OAuth и разрешения).
2. Щелкните **Add a new Redirect URL** (Добавить новый URL-адрес перенаправления).
3. Укажите [https://slack.botframework.com](https://slack.botframework.com).
4. Нажмите кнопку **Добавить**.
5. Нажмите кнопку **Save URLs** (Сохранить URL-адреса).

![Добавление URL-адреса перенаправления](~/media/channels/slack-RedirectURL.png)

## <a name="create-a-slack-bot-user"></a>Создание пользователя бота Slack

Добавив пользователя бота, можно назначить боту имя и выбрать, будет ли он всегда находиться в сети.

1. Выберите вкладку **Bot Users** (Пользователи бота).
2. Щелкните **Add a Bot User** (Добавить пользователя бота).

![Создание бота](~/media/channels/slack-CreateBot.png)

Щелкните **Add a Bot User** (Добавить пользователя бота) для проверки параметров, выберите для параметра **Always Show My Bot as Online** (Бот постоянно в сети) значение **Вкл.** и нажмите кнопку **Сохранить изменения**.

![Создание бота](~/media/channels/slack-CreateApp-AddBotUser.png)

## <a name="subscribe-to-bot-events"></a>Подписка на события бота

Выполните следующие действия для подписки на шесть определенных событий бота. При этом приложение будет получать уведомления о действиях пользователя по указанному вами URL-адресу.

> [!TIP]
> Дескриптор бота — это его имя. Для получения дескриптора бота перейдите по адресу [https://dev.botframework.com/bots](https://dev.botframework.com/bots), выберите бот и запишите имя этого бота.

1. Выберите вкладку **Подписки на события**.
2. Выберите для параметра **Enable Events** (Включить события) значение **Вкл**.
3. В поле **URL-адрес запроса** введите значение `https://slack.botframework.com/api/Events/{YourBotHandle}`, где `{YourBotHandle}` обозначает дескриптор бота без фигурных скобок. Для этого примера используется дескриптор **ContosoBot**.

   ![Подписка на события (верх)](~/media/channels/slack-SubscribeEvents-a.png)

4. В разделе **Subscribe to Bot Events** (Подписаться на события ботов) щелкните **Add Bot User Event** (Добавить пользовательское событие бота).
5. В списке событий выберите следующие шесть типов событий:
    * `member_joined_channel`
    * `member_left_channel`
    * `message.channels`
    * `message.groups`
    * `message.im`
    * `message.mpim`

   ![Подписка на события (середина)](~/media/channels/slack-SubscribeEvents-b.png)

6. Щелкните **Сохранить изменения**.

   ![Подписка на события (низ)](~/media/channels/slack-SubscribeEvents-c.png)

## <a name="add-and-configure-interactive-messages-optional"></a>Добавление и настройка интерактивных сообщений (необязательно)

Если ваш бот будет использовать функции Slack, например кнопки, выполните следующие действия.

1. Выберите вкладку **Interactive Components** (Интерактивные компоненты) и щелкните **Enable Interactive Components** (Включить интерактивные компоненты).
2. Введите `https://slack.botframework.com/api/Actions` в качестве **URL-адреса запроса**.
3. Нажмите кнопку **Save changes** (Сохранить изменения).

![Включение сообщений](~/media/channels/slack-MessageURL.png)

## <a name="gather-credentials"></a>Получение учетных данных

Выберите вкладку **Основные сведения** и перейдите к разделу **Учетные данные приложения**.
Вы увидите идентификатор клиента, секрет клиента и токен проверки, требуемые для настройки бота Slack.

![Получение учетных данных](~/media/channels/slack-AppCredentials.png)

## <a name="submit-credentials"></a>Отправка учетных данных

В отдельном окне браузера вернитесь на сайт Bot Framework `https://dev.botframework.com/`.

1. Выберите **My bots** (Мои боты) и выберите бот, который нужно подключить к Slack.
2. В разделе **Channels** (Каналы) щелкните значок Slack.
3. В разделе **Enter your Slack credentials** (Ввод учетных данных Slack) вставьте учетные данные приложения с веб-сайта Slack в соответствующие поля.
4. **URL-адрес целевой страницы** является необязательным. Его можно опустить или изменить.
5. Выберите команду **Сохранить**.

![Отправка учетных данных](~/media/channels/slack-SubmitCredentials.png)

Следуйте инструкциям, чтобы авторизовать доступ приложения Slack к команде разработчиков Slack.

## <a name="enable-the-bot"></a>Включение бота

На странице Configure Slack (Настройка Slack) убедитесь, что ползунок возле кнопки "Сохранить" установлен в положение **Включено**.
Бот настроен для взаимодействия с пользователями в Slack.

## <a name="create-an-add-to-slack-button"></a>Создание кнопки Add to Slack (Добавить в Slack)

Slack предоставляет HTML-код, с помощью которого можно упростить поиск вашего бота в разделе *Add the Slack button* (Создание кнопки "Добавить в Slack") на [этой странице](https://api.slack.com/docs/slack-button).
Чтобы использовать этот HTML-код с ботом, замените значение href (начинается с `https://`) URL-адресом, найденным в параметрах канала Slack вашего бота.
Выполните следующие действия, чтобы получить URL-адрес на замену.

1. На странице [https://dev.botframework.com/bots](https://dev.botframework.com/bots) щелкните свой бот.
2. Выберите **Channels** (Каналы), щелкните правой кнопкой мыши запись с именем **Slack** и выберите пункт **Copy link** (Копировать ссылку). Этот URL-адрес теперь находится в буфере обмена.
3. Вставьте его из буфера обмена в HTML-код, предоставленный для кнопки Slack. Этот URL-адрес заменяет значение href, предоставленное Slack для этого бота.

Авторизованные пользователи могут нажать кнопку **Add to Slack** (Добавить в Slack), предоставленную этим измененным HTML, для доступа к боту в Slack.

## <a name="slack-adaptertabadapter"></a>[Адаптер Slack](#tab/adapter)
## <a name="connect-a-bot-to-slack-using-the-slack-adapter"></a>Подключение бота к Slack с помощью адаптера Slack

Для подключения бота к Slack можно использовать не только канал, доступный в службе Azure Bot, но и адаптер Slack. Из этой статьи вы узнаете, как подключить бота к Slack с помощью адаптера.  В этой статье описано, как изменить пример EchoBot для его подключения к приложению Slack.

> [!NOTE]
> Ниже приводятся инструкции для реализации адаптера Slack на C#. Инструкции по использованию адаптера для JS, который входит в состав библиотек BotKit, см. в [документации по BotKit для Slack](https://botkit.ai/docs/v4/platforms/slack.html).

## <a name="prerequisites"></a>Предварительные требования

* [Пример кода EchoBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/02.echo-bot).

* Доступ к рабочей области Slack с достаточными разрешениями для создания приложений и управления ими в [https://api.slack.com/apps](https://api.slack.com/apps). Если у вас нет доступа к среде Slack, вы можете [бесплатно создать ее](https://www.slack.com).

## <a name="create-a-slack-application-for-your-bot"></a>Создание приложения Slack для бота

Войдите в [Slack](https://slack.com/signin) и перейдите на канал для [создания приложения Slack](https://api.slack.com/apps).

![Настройка бота](~/media/channels/slack-NewApp.png)

Нажмите кнопку Create new app (Создать приложение).

### <a name="create-an-app-and-assign-a-development-slack-team"></a>Создание приложения и назначение группы разработки Slack

Введите значение в поле **App Name** (Имя приложения) и выберите один из пунктов в списке **Development Slack Workspace** (Рабочая область разработки Slack). Если вы еще не является членом команды разработчиков Slack, [создайте новую команду или присоединитесь к существующей](https://slack.com/).

![Создание приложения](~/media/channels/slack-CreateApp.png)

Нажмите кнопку **Create App** (Создать приложение). Slack создаст приложение, а также идентификатор и секрет клиента.

### <a name="gather-required-configuration-settings-for-your-bot"></a>Сбор обязательных параметров конфигурации для бота

После создания приложения соберите следующие сведения. Они вам понадобятся, чтобы подключить бота к Slack.

1. Найдите значения **Verification Token** (Маркер проверки) и **Signing Secret** (Секрет подписи) на вкладке **Basic Information** (Базовая информация) и сохраните их для последующей настройки параметров бота.

![Маркеры Slack](~/media/bot-service-adapter-connect-slack/slack-tokens.png)

2. Перейдите на страницу **Install App** (Установка приложения) в меню **Settings** (Параметры) и следуйте инструкциям по установке приложения для команды Slack.  После установки скопируйте значение **Bot User OAuth Access Token** (Маркер доступа OAuth для пользователя бота) и также сохраните его для последующей настройки параметров бота.

## <a name="wiring-up-the-slack-adapter-in-your-bot"></a>Подключение адаптера Slack к боту

### <a name="install-the-slack-adapter-nuget-package"></a>Установка пакета NuGet для адаптера Slack

Добавьте пакет NuGet [Microsoft.Bot.Builder.Adapters.Slack](https://www.nuget.org/packages/Microsoft.Bot.Builder.Adapters.Slack/). Подробные сведения об использовании NuGet см. в руководстве по [установке пакетов и управлении ими в Visual Studio](https://aka.ms/install-manage-packages-vs).

### <a name="create-a-slack-adapter-class"></a>Создание класса адаптера Slack

Создайте новый класс, который наследует класс ***SlackAdapter***. Этот класс будет для нас адаптером канала Slack со встроенными возможностями обработки ошибок (аналогично классу ***BotFrameworkAdapterWithErrorHandler***, который уже используется в этом примере для обработки других запросов от службы Azure Bot).

```csharp
public class SlackAdapterWithErrorHandler : SlackAdapter
{
    public SlackAdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger)
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

### <a name="create-a-new-controller-for-handling-slack-requests"></a>Создание контроллера для обработки запросов Slack

Мы создадим контроллер, который будет выполнять запросы из приложения Slack, в новой конечной точке api/slack вместо api/messages, которая используется по умолчанию для запросов от каналов службы Azure Bot.  Добавив к боту дополнительную конечную точку, вы сможете с помощью одного бота принимать запросы одновременно от каналов службы Bot и Slack.

```csharp
[Route("api/slack")]
[ApiController]
public class SlackController : ControllerBase
{
    private readonly SlackAdapter _adapter;
    private readonly IBot _bot;

    public SlackController(SlackAdapter adapter, IBot bot)
    {
        _adapter = adapter;
        _bot = bot;
    }

    [HttpPost]
    [HttpGet]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await _adapter.ProcessAsync(Request, Response, _bot);
    }
}
```

### <a name="add-slack-app-settings-to-your-bots-configuration-file"></a>Добавление параметров приложения Slack в файл конфигурации бота

Добавьте три представленных ниже параметра в файл appSettings.json в проекте бота, указав для каждого из них сохраненные ранее значения.

```json
  "SlackVerificationToken": "",
  "SlackBotToken": "",
  "SlackClientSigningSecret": ""
```

### <a name="inject-the-slack-adapter-in-your-bot-startupcs"></a>Включение адаптера Slack в файл startup.cs бота

Добавьте следующую строку в метод ***ConfigureServices*** в файле startup.cs. Это действие регистрирует адаптер Slack и делает его доступным для нового класса контроллера.  Добавленные на предыдущем шаге параметры конфигурации применяются адаптером автоматически.

```csharp
services.AddSingleton<SlackAdapter, SlackAdapterWithErrorHandler>();
```

После добавления метод ***ConfigureServices*** должен выглядеть следующим образом.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the default Bot Framework Adapter (used for Azure Bot Service channels and emulator).
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkAdapterWithErrorHandler>();

    // Create the Slack Adapter
    services.AddSingleton<SlackAdapter, SlackAdapterWithErrorHandler>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

## <a name="complete-configuration-of-your-slack-app"></a>Завершение настройки приложения Slack

### <a name="obtain-a-url-for-your-bot"></a>Получение URL-адреса для бота

Теперь, когда вы создали приложение Slack и подключили адаптер в проекте бота, осталось лишь направить приложение Slack на конечную точку бота и оформить для приложения подписку, чтобы бот получал нужные сообщения.  Для этого процесса бот должен работать, чтобы решение Slack успешно проверило допустимость URL-адреса конечной точки.

Чтобы выполнить этот шаг, [разверните бота в Azure](https://aka.ms/bot-builder-deploy-az-cli) и запишите URL-адрес этого развертывания.

> [!NOTE]
> Если вы еще не готовы развернуть бота в Azure или вам нужна отладка бота с адаптером Slack, можно использовать средство [ngrok](https://www.ngrok.com) (оно должно быть уже установлено, если вы ранее использовали эмулятор Bot Framework), которое создаст туннель к запущенному в локальной среде боту и предоставит для него общедоступный URL-адрес. 
> 
> Если вы хотите создать туннель и получить для бота URL-адрес с помощью ngrok, выполните следующую команду в окне терминала. Здесь предполагается, что локальный бот работает на порту 3978. Если это не так, измените номера портов в команде.
> 
> ```
> ngrok.exe http 3978 -host-header="localhost:3978"
> ```

### <a name="update-your-slack-app"></a>Обновление приложения Slack

Вернитесь на [панель мониторинга API Slack]([https://api.slack.com/apps]) и выберите свое приложение.  Теперь вам нужно настроить два URL-адреса для приложения и оформить подписку на соответствующие события.

1. На вкладке **OAuth и разрешения** должен быть указан **URL-адрес перенаправления**, который принадлежит вашему боту, а также конечная точка `api/slack` в только что созданном контроллере. Например, `https://yourboturl.com/api/slack`.

![URL-адрес перенаправления Slack](~/media/bot-service-adapter-connect-slack/redirect-url.png)

2. На вкладке **Event Subscriptions** (Подписки на события) введите **URL-адрес запроса**, то есть тот же URL-адрес, что и на шаге 1.

3. Включите события, используя переключатель в верхней части страницы.

4. Разверните раздел **Subscribe to bot events** (Подписка на события бота) и нажмите кнопку **Add Bot User Event** (Добавить пользовательское событие бота), чтобы подписаться на события **im_created** и **message.im**.

![Подписки на события Slack](~/media/bot-service-adapter-connect-slack/event-subscriptions.png)

## <a name="test-your-bot-with-adapter-in-slack"></a>Тестирование работы бота с адаптером в Slack

Теперь приложение Slack полностью настроено и вы можете войти в рабочую область Slack, где установлено приложение. (Вы увидите его в списке раздела Apps (Приложения) в меню слева.) Выберите приложение и попробуйте отправить сообщение. Вы увидите, что оно появится в окне мгновенных сообщений.

Эту функцию также можно протестировать с помощью [примера бота для адаптера Slack](https://aka.ms/csharp-60-slack-adapter-sample), заполнив файл appSettings.json теми же значениями, что и в описанных выше шагах. В файле README для этого примера есть дополнительные действия для предоставления ссылок, получения вложений и отправки интерактивных сообщений.

---

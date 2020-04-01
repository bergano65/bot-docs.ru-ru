---
title: Подключение бота к Facebook Messenger (служба Azure Bot)
description: Узнайте, как настроить подключение бота к Facebook Messenger.
keywords: Facebook Messenger, bot channel, Facebook App, App ID, App Secret, Facebook bot, credentials
manager: kamrani
ms.topic: article
ms.author: kamrani
ms.service: bot-service
ms.date: 01/16/2020
ms.openlocfilehash: 84544f5c4082f88991718c5d84dc2aead05f5eb2
ms.sourcegitcommit: 64b25f796f89e8bb6fa53d3c824b73b8ce4d6ed8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2020
ms.locfileid: "80250143"
---
# <a name="connect-a-bot-to-facebook"></a>Подключение бота к Facebook

Вы можете подключить бот к Facebook Messenger и Facebook Workplace, чтобы он мог взаимодействовать с пользователями на обеих платформах. В этом руководстве описано, как подключить бота к этим двум каналам.

> [!NOTE]
> Пользовательский интерфейс Facebook может выглядеть несколько иначе, в зависимости от используемой версии.

## <a name="connect-a-bot-to-facebook-messenger"></a>Подключение бота к Facebook Messenger

Дополнительные сведения о разработке приложений для Facebook Messenger см. в [документации по платформе Messenger](https://developers.facebook.com/docs/messenger-platform). Вы можете просмотреть [инструкции перед запуском](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public), [краткое руководство](https://developers.facebook.com/docs/messenger-platform/guides/quick-start) и [руководство по настройке](https://developers.facebook.com/docs/messenger-platform/guides/setup) Facebook.

Чтобы настроить бот для обмена данными с Facebook Messenger, включите Facebook Messenger на странице Facebook и подключите бота.

### <a name="copy-the-page-id"></a>Копирование идентификатора страницы

Доступ к боту осуществляется на странице Facebook.

1. [Создайте страницу Facebook](https://www.facebook.com/bookmarks/pages) или перейдите на имеющуюся.

1. Перейдите к странице **About** (О программе) на странице Facebook, а затем скопируйте и сохраните **идентификатор страницы**.

### <a name="create-a-facebook-app"></a>Создание приложения Facebook

1. В браузере перейдите на страницу [создания приложения Facebook](https://developers.facebook.com/quickstarts/?platform=web).
1. Укажите имя приложения и щелкните **Create New Facebook App ID** (Создать идентификатор нового приложения Facebook).

    ![Создание приложения](media/channels/fb-create-messenger-bot-app.png)

1. В появившемся диалоговом окне введите адрес электронной почты и щелкните **Create App ID** (Создать идентификатор приложения).

    ![Создание идентификатора приложения](media/channels/fb-create-messenger-bot-app-id.png)

1. Выполните указания мастера.

1. Введите необходимые сведения для проверки и щелкните **Skip Quick Start** (Пропустить быстрый запуск) вверху справа.

1. В следующем окне слева откройте раздел *Settings* (Параметры) и щелкните **Basic** (Общие).

1. В области справа скопируйте и сохраните **идентификатор приложения** и **секрет приложения**.

    ![Копирование идентификатора приложения и секрета приложения](media/channels/fb-messenger-bot-get-appid-secret.png)

1. В области слева в разделе *Settings* (Параметры) щелкните **Advanced** (Дополнительно).

1. В области справа установите ползунок **Allow API Access to App Settings** (Разрешить доступ API к параметрам приложения) в положение **Yes** (Да).

    ![Копирование идентификатора приложения и секрета приложения](media/channels/fb-messenger-bot-api-settings.png)

1. Внизу справа щелкните **Save Changes** (Сохранить изменения).

### <a name="enable-messenger"></a>Включение Messenger

1. В области слева щелкните **Dashboard** (Панель мониторинга).
1. В области справа прокрутите вниз и в поле **Messenger** щелкните **Set Up** (Настроить). Запись Messenger отобразится в разделе *PRODUCTS* (Продукты) в области слева.  

    ![Включение Messenger](media/channels/fb-messenger-bot-enable-messenger.png)

### <a name="generate-a-page-access-token"></a>Создание маркера доступа к странице

1. В области слева под записью Messenger щелкните **Settings** (Параметры).
1. В области справа прокрутите вниз и в разделе **Token Generation** (Создание токена) выберите целевую страницу.

    ![Включение Messenger](media/channels/fb-messenger-bot-select-messenger-page.png)

1. Щелкните **Edit Permissions** (Изменить разрешения), чтобы использовать pages_messaging приложения для создания маркера доступа.
1. Следуйте указаниям мастера. Наконец, примите параметры по умолчанию и щелкните **Done** (Готово). Будет создан **маркер доступа к странице**.

    ![Разрешения Messenger](media/channels/fb-messenger-bot-permissions.png)

1. Скопируйте и сохраните **маркер доступа к странице**.

### <a name="enable-webhooks"></a>Включение веб-перехватчиков

Чтобы отправлять сообщения и другие события из бота в Facebook Messenger, необходимо включить интеграцию веб-перехватчиков. На этом этапе можно оставить параметры Facebook как есть. Мы еще вернемся к ним.

1. В браузере откройте новое окно и перейдите на [портал Azure.](https://portal.azure.com/)

1. В списке ресурсов щелкните регистрацию ресурса бота и в соответствующей колонке щелкните **Каналы**.

1. Справа щелкните значок **Facebook**.

1. В мастере введите данные Facebook, сохраненные на предыдущих шагах. Если информация правильная, в мастере внизу вы увидите **URL-адрес обратного вызова** и **токен проверки**. Скопируйте и сохраните эти значения.  

    ![Конфигурация канала Facebook Messenger](media/channels/fb-messenger-bot-config-channel.PNG)

1. Нажмите кнопку **Сохранить** .

1. Вернемся к параметрам Facebook. Справа прокрутите вниз и в разделе **Webhooks** (Веб-перехватчики) щелкните **Subscribe To Events** (Подписаться на события). Так вы настроите переадресацию событий обмена сообщениями из Facebook Messenger в бота.

    ![Включение веб-перехватчиков](media/channels/fb-messenger-bot-webhooks.PNG)

1. В появившемся диалоговом окне введите сохраненные ранее значения **URL-адреса обратного вызова** и **токена проверки**. В разделе **Subscription Fields** (Поля подписки) выберите *message\_deliveries*, *messages*, *messaging\_options* и *messaging\_postbacks*.

    ![Настройка веб-перехватчика](media/channels/fb-messenger-bot-config-webhooks.png)

1. Щелкните **Verify and Save** (Проверить и сохранить).

1. Подпишите веб-перехватчик на нужную страницу Facebook. Щелкните **Subscribe** (Подписать).

    ![Страница настройки веб-перехватчика](media/channels/fb-messenger-bot-config-webhooks-page.PNG)

### <a name="submit-for-review"></a>Отправка на проверку

На странице базовых параметров приложения Facebook требуется указать URL-адрес политики конфиденциальности и условий использования. На странице [Code of Conduct](https://investor.fb.com/corporate-governance/code-of-conduct/default.aspx) (Правила поведения) содержатся ссылки на сторонние ресурсы для создания политики конфиденциальности. На странице [Terms of Use](https://www.facebook.com/terms.php) (Условия использования) содержится пример условий, помогающий создать соответствующий документ с условиями использования.

После завершения работы с ботом в Facebook будет реализован собственный механизм [рассмотрения](https://developers.facebook.com/docs/messenger-platform/app-review) для приложений, которые публикуются в Messenger. Бот будет проверен на соответствие [политикам платформы](https://developers.facebook.com/docs/messenger-platform/policy-overview) Facebook.

### <a name="make-the-app-public-and-publish-the-page"></a>Предоставления общего доступа к приложению и публикация страницы

> [!NOTE]
> До публикации приложение находится в [режиме разработки](https://developers.facebook.com/docs/apps/managing-development-cycle). Функции подключаемого модуля и API будут работать только для администраторов, разработчиков и тест-инженеров.

После успешной проверки на панели мониторинга приложения в разделе рассмотрения приложения выберите значение Public (общедоступное).
Убедитесь, что страница Facebook, связанная с этим ботом, опубликована. Состояние появляется в параметрах страницы.

## <a name="connect-a-bot-to-facebook-workplace"></a>Подключение бота к Facebook Workplace

> [!NOTE]
> 16 декабря 2019 г. в Workplace by Facebook изменилась модель обеспечения безопасности для пользовательских интеграций. Интеграции, созданные ранее с помощью Microsoft Bot Framework версии 4, необходимо обновить до 28 февраля 2020 г. для использования адаптеров Facebook для Bot Framework в соответствии с приведенными ниже инструкциями.
>
> Далее Facebook будет считать допустимыми для продолжения использования (до 31 декабря 2020 г.) только интеграции с ограниченным доступом к данным Workplace (разрешения с низким уровнем конфиденциальности), которые успешно прошли проверки безопасности RFI и по которым разработчик не позднее 15 января 2020 г. связался со службой поддержки [Direct Support](https://my.workplace.com/work/admin/direct_support) с запросом на продолжение работы приложения.
>
> Адаптеры Bot Framework доступны для ботов, созданных с помощью [JavaScript и Node.js](https://aka.ms/npm-botbuilder-adapter-facebook), а также [C# и .NET](https://aka.ms/botbuilder-dotnet-facebook-adapter).

Facebook Workplace — это ориентированная на бизнес версия Facebook, которая позволяет сотрудникам легко подключаться и совместно работать. Она содержит видео в реальном времени, веб-каналы новостей, группы, приложение для обмена сообщениями, реакции, функции поиска и популярные записи. Она также поддерживает следующие возможности:

- Аналитика и интеграция. Панель мониторинга с функциями аналитики, интеграции и единого входа, а также поставщиками удостоверений, которые используются компаниями для интеграции Workplace с существующими ИТ-системами.
- Группы из нескольких компаний. Общие пространства, в которых сотрудники из разных организаций могут совместно работать.

Откройте [центр справки Workplace](https://workplace.facebook.com/help/work/), чтобы изучить сведения о Facebook Workplace и [документацию для разработчиков Workplace](https://developers.facebook.com/docs/workplace), с подробными рекомендациями.

Чтобы использовать Facebook Workplace с ботом, создайте учетную запись Workplace и настраиваемую интеграцию для подключения бота.

### <a name="create-a-workplace-premium-account"></a>Создание учетной записи Workplace Premium

1. Отправьте приложение в [Workplace](https://www.facebook.com/workplace) от имени компании.
1. Когда приложение будет утверждено, вы получите сообщение электронной почты с приглашением присоединиться. Это может занять некоторое время.
1. В полученном приглашении щелкните **Get Started**. (Начать работу).
1. Укажите данные профиля.
    > [!TIP]
    > Назначьте себя системным администратором. Помните, что только системные администраторы могут создавать настраиваемые интеграции.
1. Щелкните **Preview Profile** (Предварительный просмотр профиля) и проверьте указанные данные.
1. Получите доступ к *бесплатной пробной версии*.
1. Укажите **пароль**.
1. Щелкните **Invite Coworkers** (Пригласить сотрудников), чтобы предоставить сотрудникам возможность входа. Приглашенные сотрудники станут участниками, как только они войдут, следуя приведенным здесь инструкциям.

### <a name="create-a-custom-integration"></a>Создание пользовательской интеграции

Создайте [настраиваемую интеграцию](https://developers.facebook.com/docs/workplace/custom-integrations-new) для Workplace, как описано ниже. При создании пользовательской интеграции автоматически создаются приложение с нужными разрешениями и страница типа "Бот", доступная только в пределах вашего сообщества Workplace.

1. На **панели администратора** откройте вкладку **Интеграции**.
1. Нажмите кнопку **Create your own custom App** (Создать пользовательское приложение).

    ![Интеграция Workplace](media/channels/fb-integration.png)

1. Выберите для приложения отображаемое имя и изображение профиля. Эта информация будет использоваться совместно со страницей типа "Бот".
1. Установите для параметра **Allow API Access to App Settings** (Разрешить доступ API к параметры приложения) значение Yes (Да).
1. Скопируйте и безопасно сохраните код приложения, секрет приложения и маркер приложения, которые вы здесь видите.

    ![Ключи Workplace](media/channels/fb-keys.png)

1. Итак, вы завершили создание пользовательской интеграции. Страницу типа "Бот" можно найти своем сообществе Workplace, как показано ниже.

    ![Страница Workplace](media/channels/fb-page.png)

### <a name="update-your-bot-code-with-facebook-adapter"></a>Включение адаптера Facebook в код бота

Исходный код бота следует обновить, включив в него адаптер для взаимодействия с Workplace by Facebook. Эти адаптеры доступны для ботов, созданных с помощью [JavaScript и Node.js](https://aka.ms/npm-botbuilder-adapter-facebook), а также [C# и .NET](https://aka.ms/botbuilder-dotnet-facebook-adapter).

### <a name="provide-facebook-credentials"></a>Указание учетных данных Facebook

Необходимо обновить файл бота appsettings.json, добавив значения **Facebook App ID** (Идентификатор приложения Facebook), **Facebook App Secret** (Секрет приложения Facebook) и **Page Access Token** (Маркер доступа к странице), скопированные ранее из Facebook Workplace. Вместо традиционного идентификатора страницы используйте числа, указанные после имени интегрируемого приложения на соответствующей **странице** со сведениями. Выполните эти инструкции, чтобы обновить исходный код бота для [JavaScript и Node.js](https://aka.ms/npm-botbuilder-adapter-facebook) или [C# и .NET](https://aka.ms/botbuilder-dotnet-facebook-adapter).

### <a name="submit-for-review"></a>Отправка на проверку

Подробные сведения см. в разделе **о подключении бота к Facebook Messenger** и в [документации для разработчика Workplace](https://developers.facebook.com/docs/workplace).

### <a name="make-the-app-public-and-publish-the-page"></a>Предоставления общего доступа к приложению и публикация страницы

Подробные сведения см. в разделе **о подключении бота к Facebook Messenger**.

### <a name="setting-the-api-version"></a>Указание версии API

Если вы получите уведомление от Facebook о том, что определенная версия API Graph теперь считается нерекомендуемой, перейдите на [страницу для разработчиков Facebook](https://developers.facebook.com). Перейдите к **параметрам приложения** своего бота и выберите **Параметры > Дополнительные параметры > Обновить версию API**, а затем укажите для параметра **Обновить все вызовы** версию 3.0.

![Обновление версии API](media/channels/fb-version-upgrade.png)

## <a name="connect-a-bot-to-facebook-using-the-facebook-adapter"></a>Подключение бота к Facebook с помощью адаптера Facebook

Чтобы подключить бот к Facebook Workplace, используйте адаптер Facebook для Bot Framework. Для подключения к Facebook Messenger, можно использовать канал или адаптер Facebook.
Адаптеры Facebook доступны для ботов, созданных с помощью [JavaScript и Node.js](https://aka.ms/npm-botbuilder-adapter-facebook), а также [C# и .NET](https://aka.ms/botbuilder-dotnet-facebook-adapter).

Из этой статьи вы узнаете, как подключить бота к Facebook с помощью адаптера.  Здесь представлена пошаговая инструкция, которая позволит изменить пример EchoBot для подключения его к Facebook.

Ниже приводятся инструкции для реализации адаптера Facebook на C#. Инструкции по использованию адаптера для JavaScript, который входит в состав библиотек BotKit, см. в [документации по BotKit для Facebook](https://botkit.ai/docs/v4/platforms/facebook.html).

### <a name="prerequisites"></a>Предварительные требования

- [Пример кода EchoBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/02.echo-bot).
- Учетная запись Facebook для разработчиков. Если у вас нет учетной записи, [создайте ее](https://developers.facebook.com).

### <a name="create-a-facebook-app-page-and-gather-credentials"></a>Создание приложения и страницы Facebook, сбор учетных данных

1. Войдите на сайт [https://developers.facebook.com](https://developers.facebook.com). Щелкните **Мои приложения** в главном меню и выберите в раскрывающемся меню пункт **Создать приложение**.

![Создание приложения](media/bot-service-channel-connect-facebook/create-app-button.png)

1. Откроется всплывающее окно, в котором нужно ввести имя нового приложения и нажать кнопку **Создайте ID приложения**.

![Определение имени приложения](media/bot-service-channel-connect-facebook/app-name.png)

#### <a name="set-up-messenger-and-associate-a-facebook-page"></a>Настройка Messenger и связывание страницы Facebook

1. После создания приложения отобразится список доступных для настройки продуктов. Нажмите кнопку **Настроить** рядом с продуктом **Messenger**.

1. Теперь необходимо связать новое приложение со страницей Facebook (если у вас нет подходящей для использования страницы, вы можете создать новую, щелкнув **Создать новую страницу** в разделе **Маркеры доступа**). Щелкните **Добавить или удалить страницы**, затем выберите страницу для связывания с приложением и щелкните **Далее**. Оставьте включенным параметр **Manage and access Page conversations on Messenger** (Управление беседами на странице и доступ к ним через Messenger) и щелкните **Готово**.

![Настройка Messenger](media/bot-service-channel-connect-facebook/app-page-permissions.png)

1. После связывания страницы нажмите кнопку **Generate Token** (Создать маркер), чтобы создать маркер доступа к странице.  Запишите этот маркер, так как он понадобится позже при настройке приложения бота.

#### <a name="obtain-your-app-secret"></a>Получение секрета приложения

1. В меню слева щелкните **Настройки**, а затем **Основное**, чтобы открыть страницу основных параметров приложения.

1. На странице основных параметров нажмите кнопку **Показать** в поле **Секрет приложения**.  Запишите этот секрет, так как он потребуется позже при настройке приложения бота.

### <a name="wiring-up-the-facebook-adapter-in-your-bot"></a>Подключение адаптера Facebook к боту

Теперь, когда у вас есть приложение, страница и учетные данные Facebook, можно настроить приложение бота.

#### <a name="install-the-facebook-adapter-nuget-package"></a>Установка пакета NuGet для адаптера Facebook

Добавьте пакет NuGet [Microsoft.Bot.Builder.Adapters.Facebook](https://www.nuget.org/packages/Microsoft.Bot.Builder.Adapters.Facebook/). Подробные сведения об использовании NuGet см. в руководстве по [установке пакетов и управлении ими в Visual Studio](https://aka.ms/install-manage-packages-vs).

#### <a name="create-a-facebook-adapter-class"></a>Создание класса адаптера Facebook

Создайте новый класс, который наследует класс ***FacebookAdapter***. Этот класс будет для нас адаптером канала Facebook со встроенными возможностями обработки ошибок (аналогично классу ***BotFrameworkAdapterWithErrorHandler***, который уже используется в этом примере для обработки других запросов от службы Azure Bot).

```csharp
public class FacebookAdapterWithErrorHandler : FacebookAdapter
{
    public FacebookAdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger)
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

#### <a name="create-a-new-controller-for-handling-facebook-requests"></a>Создание контроллера для обработки запросов Facebook

Создайте контроллер, который будет выполнять запросы из Facebook, в новой конечной точке api/facebook вместо api/messages, которая используется по умолчанию для запросов от каналов службы Azure Bot.  Добавив к боту дополнительную конечную точку, вы сможете с помощью одного бота принимать запросы одновременно от каналов службы Bot и Facebook.

```csharp
[Route("api/facebook")]
[ApiController]
public class FacebookController : ControllerBase
{
    private readonly FacebookAdapter _adapter;
    private readonly IBot _bot;

    public FacebookController(FacebookAdapter adapter, IBot bot)
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

#### <a name="inject-the-facebook-adapter-in-your-bot-startupcs"></a>Включение адаптера Facebook в файл startup.cs бота

Добавьте следующую строку в метод ***ConfigureServices*** в файле startup.cs. Это действие регистрирует адаптер Facebook и делает его доступным для нового класса контроллера.  Добавленные на предыдущем шаге параметры конфигурации применяются адаптером автоматически.

```csharp
services.AddSingleton<FacebookAdapter, FacebookAdapterWithErrorHandler>();
```

После добавления метод ***ConfigureServices*** должен выглядеть следующим образом.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the default Bot Framework Adapter (used for Azure Bot Service channels and emulator).
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkAdapterWithErrorHandler>();

    // Create the Facebook Adapter
    services.AddSingleton<FacebookAdapter, FacebookAdapterWithErrorHandler>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

#### <a name="obtain-a-url-for-your-bot"></a>Получение URL-адреса для бота

Теперь, когда вы подключили адаптер в проекте бота, необходимо передать в Facebook правильную конечную точку этого приложения, чтобы бот мог получать сообщения. Этот URL-адрес также потребуется для завершения настройки приложения бота.

Чтобы выполнить этот шаг, [разверните бота в Azure](https://aka.ms/bot-builder-deploy-az-cli) и запишите URL-адрес этого развертывания.

> [!NOTE]
> Если вы еще не готовы развернуть бота в Azure или вам нужна отладка бота с адаптером Facebook, можно использовать средство [ngrok](https://www.ngrok.com) (оно должно быть уже установлено, если вы ранее использовали эмулятор Bot Framework), которое создаст туннель к запущенному в локальной среде боту и предоставит для него общедоступный URL-адрес.
>
> Если вы хотите создать туннель и получить для бота URL-адрес с помощью ngrok, выполните следующую команду в окне терминала. Здесь предполагается, что локальный бот работает на порту 3978. Если это не так, измените номера портов в команде.
>
> ```cmd
> ngrok.exe http 3978 -host-header="localhost:3978"
> ```

#### <a name="add-facebook-app-settings-to-your-bots-configuration-file"></a>Добавление параметров приложения Facebook в файл конфигурации бота

Добавьте перечисленные ниже параметры в файл appSettings.json в проекте бота. Параметры **FacebookAppSecret** и **FacebookAccessToken** заполняются значениями, которые вы записали при создании и настройке приложения Facebook. Маркер **FacebookVerifyToken** должен содержать случайную строку, которую вы создаете самостоятельно, и применяется для подтверждения подлинности конечной точки бота при вызове из Facebook.

```json
  "FacebookVerifyToken": "",
  "FacebookAppSecret": "",
  "FacebookAccessToken": ""
```

Теперь, когда вы заполнили указанные выше параметры, следует повторно развернуть бота или просто перезапустить его, если вы используете туннель ngrok к локальной среде.

### <a name="complete-configuration-of-your-facebook-app"></a>Завершение настройки приложения Facebook

Заключительным этапом является настройка конечной точки Messenger приложения Facebook, чтобы бот мог получать из нее сообщения.

1. На панели мониторинга приложения щелкните **Messenger** в меню слева и выберите **Настройки**.

1. В разделе **Webhooks** щелкните **Добавить URL обратного вызова**.

1. В текстовом поле **URL обратного вызова** введите URL-адрес бота и конечную точку `api/facebook`, которую вы указали в только что созданном контроллере. Например, `https://yourboturl.com/api/facebook`. В текстовом поле **Маркер подтверждения** введите созданный ранее маркер проверки, который вы указали в файле appSettings.json приложения бота.

    ![Изменение URL-адреса обратного вызова](media/bot-service-channel-connect-facebook/edit-callback-url.png)

1. Нажмите кнопку **Verify and Save** (Проверить и сохранить). Убедитесь, что бот запущен, так как Facebook выполнит запрос к конечной точке этого приложения и проверит его с помощью значения, указанного в поле **Маркер подтверждения**.

1. После проверки URL-адреса обратного вызова нажмите появившуюся кнопку **Add Subscriptions** (Добавить подписки).  Во всплывающем окне выберите следующие подписки и щелкните **Сохранить**.

    - **messages**
    - **messaging_postbacks**
    - **messaging_optins**
    - **messaging_deliveries**

    ![Подписки веб-перехватчиков](media/bot-service-channel-connect-facebook/webhook-subscriptions.png)

### <a name="test-your-bot-with-adapter-in-facebook"></a>Тестирование работы бота с адаптером в Facebook

Теперь можно проверить, правильно ли бот подключен к Facebook, отправив сообщение через страницу Facebook, которую вы связали с новым приложением Facebook.  

1. Перейдите к нужной странице Facebook.

1. Нажмите кнопку **Add a Button** (Добавить кнопку).

    ![Добавление кнопки](media/bot-service-channel-connect-facebook/add-button.png)

1. Выберите действие **Contact You** (Связаться со мной), а затем **Send Message** (Отправить сообщение) и щелкните **Далее**.

    ![Добавление кнопки](media/bot-service-channel-connect-facebook/button-settings.png)

1. При появлении запроса **Where would you like this button to send people to?** (Куда эта кнопка должна направлять пользователей?) выберите **Messenger** и щелкните **Готово**.

    ![Добавление кнопки](media/bot-service-channel-connect-facebook/button-settings-2.png)

1. Наведите указатель мыши на новую кнопку **Send Message** (Отправить сообщение), которая теперь отображается на вашей странице Facebook, и щелкните **Test Button** (Проверить кнопку) во всплывающем меню.  Это действие запускает новый диалог с приложением через Facebook Messenger, в котором вы можете проверить обмен сообщениями с ботом. Каждый раз, когда ваш бот получает сообщение, он отправляет вам ответ с копией текста вашего сообщения.

Эту функцию также можно протестировать с помощью [примера бота для адаптера Facebook](https://aka.ms/csharp-61-facebook-adapter-sample), заполнив файл appSettings.json теми же значениями, что и в описанных выше шагах.

## <a name="see-also"></a>См. также раздел

- **Пример кода**. См. пример бота [Facebook-events](https://aka.ms/facebook-events), чтобы узнать об обмене данными между ботом и Facebook Messenger.

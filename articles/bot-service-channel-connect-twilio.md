---
title: Подключение бота к Twilio (служба Azure Bot)
description: Сведения о настройке подключения бота к Twilio.
keywords: Twilio, bot channels, SMS, App, phone, configure Twilio, cloud communication, text
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 10/9/2018
ms.openlocfilehash: 42e79bbf79418c30ff0b4b6c584ab4791cd246c0
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75793095"
---
# <a name="connect-a-bot-to-twilio"></a>Подключение бота к Twilio

Вы можете настроить взаимодействие бота с пользователями с помощью облачной платформы обмена данными Twilio.

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>Вход в учетную запись Twilio или ее создание для отправки и получения SMS-сообщений

Если у вас нет учетной записи Twilio, <a href="https://www.twilio.com/try-twilio" target="_blank">создайте ее</a>.

## <a name="create-a-twiml-application"></a>Создание приложения TwiML

<a href="https://support.twilio.com/hc/articles/223180928-How-Do-I-Create-a-TwiML-App-" target="_blank">Создайте приложения TwiML</a> в соответствии с инструкциями.

![Создание приложения](~/media/channels/twi-StepTwiml.png)

В разделе **Свойства**, введите **FRIENDLY NAME** (Понятное имя). В этом руководстве для примера используется имя "My TwiML app". Поле **REQUEST URL** (URL-адрес запроса) под Voice можно оставить пустым. В разделе **Messaging** (Обмен сообщениями) для параметра **Request URL** (URL-адрес запроса) введите значение `https://sms.botframework.com/api/sms`.

## <a name="select-or-add-a-phone-number"></a>Выбор или добавление номера телефона

Следуйте представленным <a href = "https://support.twilio.com/hc/articles/223180048-Adding-a-Verified-Phone-Number-or-Caller-ID-with-Twilio" target="_blank">здесь</a> инструкциям, чтобы добавить идентификатор проверенного вызывающего объекта через сайт консоли. После завершения вы увидите проверенный номер в поле **Active Numbers** (Активные номера) раздела **Manage Numbers** (Управление номерами).

![Установка номера телефона](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-voice-and-messaging"></a>Указание приложений для голосовой связи и обмена сообщениями

Щелкните число и перейдите к разделу **Настройка**. Для голосовой связи и обмена сообщениями укажите в поле **CONFIGURE WITH** (Настройка с помощью) значение "TwiML App", а в поле **TWIML APP** (Приложение TwiML) выберите "My TwiML app". Завершив настройку, щелкните **Сохранить**.

![Выбор приложения](~/media/channels/twi-StepPhone2.png)

Вернитесь к разделу **Управление номерами**. Вы увидите, что для голосовой связи и обмена сообщениями теперь в конфигурации указано приложение TwiML.

![Указанный номер](~/media/channels/twi-StepPhone3.png)


## <a name="gather-credentials"></a>Получение учетных данных

Вернитесь к [домашней странице консоли](https://www.twilio.com/console/), где на панели мониторинга проекта вы найдете идентификатор безопасности учетной записи и маркер проверки подлинности, как показано ниже.

![Сбор учетных данных приложения](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>Отправка учетных данных

В отдельном окне браузера откройте сайт Bot Framework, расположенный по адресу https://dev.botframework.com/. 

- Щелкните **My bots** (Мои боты) и выберите бот, который нужно подключить к Twilio. Это действие перенесет вас на портал Azure.
- Выберите **Каналы** в разделе **Управление ботами**. Щелкните значок Twilio (SMS).
- Введите номер телефона, идентификатор безопасности учетной записи и маркер проверки подлинности, которые вы записали ранее. Завершив настройку, щелкните **Сохранить**.

![Отправка учетных данных](~/media/channels/twi-StepSubmit.png)

Как только вы выполните эти действия, ваш бот будет настроен для взаимодействия с пользователями с помощью Twilio.

## <a name="connect-a-bot-to-twilio-using-the-twilio-adapter"></a>Подключение бота к Twilio с помощью адаптера Twilio

Для подключения бота к Twilio можно использовать не только канал, доступный в службе Azure Bot, но и адаптер Twilio. Из этой статьи вы узнаете, как подключить бота к Twilio с помощью адаптера.  Здесь представлена пошаговая инструкция, которая позволит изменить пример EchoBot для подключения его к Twilio.

> [!NOTE]
> Ниже приводятся инструкции для реализации адаптера Twilio на C#. Инструкции по использованию адаптера для JS, который входит в состав библиотек BotKit, см. в [документации по BotKit для Twilio](https://botkit.ai/docs/v4/platforms/twilio.html).

### <a name="prerequisites"></a>предварительные требования

* [Пример кода EchoBot](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/02.echo-bot).

* Учетная запись Twilio. Если у вас нет учетной записи Twilio, [создайте ее](https://www.twilio.com/try-twilio).

### <a name="get-a-twilio-number-and-gather-account-credentials"></a>Получение номера Twilio и сбор учетных данных учетной записи

1. Войдите в [Twilio](https://twilio.com/console). В правой части страницы вы увидите параметры **ACCOUNT SID** (ИД безопасности учетной записи) и **AUTH TOKEN** (Маркер аутентификации) для вашей учетной записи. Запишите их, так как они понадобятся позднее при настройке приложения бота.

2. Выберите элемент **Programmable Voice** (Программируемый голос) из раздела **Get Started with Twilio** (Начало работы с Twilio).

![Начало работы с программируемым голосом](~/media/bot-service-channel-connect-twilio/get-started-voice.png)

3. На следующей странице нажмите кнопку **Get your first Twilio number** (Получить первый номер Twilio).  Во всплывающем окне отобразится новый номер. Вы можете принять его щелчком по кнопке **Choose this number** (Выбрать этот номер), или найти другой номер, следуя инструкциям на экране.

4. Завершив выбор номера, запишите его, так как он потребуется позже при настройке приложения бота.

### <a name="wiring-up-the-twilio-adapter-in-your-bot"></a>Подключение адаптера Twilio к боту

Теперь, когда у вас есть номер Twilio и учетные данные, можно настроить приложение бота.

#### <a name="install-the-twilio-adapter-nuget-package"></a>Установка пакета NuGet для адаптера Twilio

Добавьте пакет NuGet [Microsoft.Bot.Builder.Adapters.Twilio](https://www.nuget.org/packages/Microsoft.Bot.Builder.Adapters.Twilio/). Подробные сведения об использовании NuGet см. в руководстве по [установке пакетов и управлении ими в Visual Studio](https://aka.ms/install-manage-packages-vs).

#### <a name="create-a-twilio-adapter-class"></a>Создание класса адаптера Twilio

Создайте новый класс, который наследует класс ***TwilioAdapter***. Этот класс будет для нас адаптером канала Twilio со встроенными возможностями обработки ошибок (аналогично классу ***BotFrameworkAdapterWithErrorHandler***, который уже используется в этом примере для обработки других запросов от службы Azure Bot).

```csharp
public class TwilioAdapterWithErrorHandler : TwilioAdapter
{
    public TwilioAdapterWithErrorHandler(IConfiguration configuration, ILogger<BotFrameworkHttpAdapter> logger)
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

#### <a name="create-a-new-controller-for-handling-twilio-requests"></a>Создание контроллера для обработки запросов Twilio

Создайте контроллер, который будет выполнять запросы из приложения Twilio, в новой конечной точке api/twilio вместо api/messages, которая используется по умолчанию для запросов от каналов службы Azure Bot.  Добавив к боту дополнительную конечную точку, вы сможете с помощью одного бота принимать запросы одновременно от каналов службы Bot и Twilio.

```csharp
[Route("api/twilio")]
[ApiController]
public class TwilioController : ControllerBase
{
    private readonly TwilioAdapter _adapter;
    private readonly IBot _bot;

    public TwilioController(TwilioAdapter adapter, IBot bot)
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

#### <a name="inject-the-twilio-adapter-in-your-bot-startupcs"></a>Включение адаптера Twilio в файл startup.cs бота

Добавьте следующую строку в метод ***ConfigureServices*** в файле startup.cs. Это действие регистрирует адаптер Twilio и делает его доступным для нового класса контроллера.  Добавленные на предыдущем шаге параметры конфигурации применяются адаптером автоматически.

```csharp
services.AddSingleton<TwilioAdapter, TwilioAdapterWithErrorHandler>();
```

После добавления метод ***ConfigureServices*** должен выглядеть следующим образом.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the default Bot Framework Adapter (used for Azure Bot Service channels and emulator).
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkAdapterWithErrorHandler>();

    // Create the Twilio Adapter
    services.AddSingleton<TwilioAdapter, TwilioAdapterWithErrorHandler>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

#### <a name="obtain-a-url-for-your-bot"></a>Получение URL-адреса для бота

Теперь, когда вы настроили адаптер в проекте бота, необходимо найти правильную конечную точку и передать ее в Twilio, чтобы бот мог получать сообщения. Этот URL-адрес также потребуется для завершения настройки приложения бота.

Чтобы выполнить этот шаг, [разверните бота в Azure](https://aka.ms/bot-builder-deploy-az-cli) и запишите URL-адрес этого развертывания.

> [!NOTE]
> Если вы еще не готовы развернуть бота в Azure или вам нужна отладка бота с адаптером Twilio, можно использовать средство [ngrok](https://www.ngrok.com) (оно должно быть уже установлено, если вы ранее использовали эмулятор Bot Framework), которое создаст туннель к запущенному в локальной среде боту и предоставит для него общедоступный URL-адрес. 
> 
> Если вы хотите создать туннель и получить для бота URL-адрес с помощью ngrok, выполните следующую команду в окне терминала. Здесь предполагается, что локальный бот работает на порту 3978. Если это не так, измените номера портов в команде.
> 
> ```
> ngrok.exe http 3978 -host-header="localhost:3978"
> ```

#### <a name="add-twilio-app-settings-to-your-bots-configuration-file"></a>Добавление параметров приложения Twilio в файл конфигурации бота

Добавьте перечисленные ниже параметры в файл appSettings.json в проекте бота. Для параметров **TwilioNumber**, **TwilioAccountSid** и **TwilioAuthToken** используются значения, которые вы получили при создании номера Twilio. В поле **TwilioValidationUrl** введите URL-адрес бота и конечную точку `api/twilio`, которую вы указали в только что созданном контроллере. Например, `https://yourboturl.com/api/twilio`.


```json
  "TwilioNumber": "",
  "TwilioAccountSid": "",
  "TwilioAuthToken": "",
  "TwilioValidationUrl", ""
```

Теперь, когда вы заполнили указанные выше параметры, следует повторно развернуть бота или просто перезапустить его, если вы используете туннель ngrok к локальной среде.

### <a name="complete-configuration-of-your-twilio-number"></a>Завершение настройки номера Twilio

Заключительным этапом является настройка конечной точки отправки сообщений в Twilio, чтобы бот мог получать из нее сообщения.

1. Перейдите на [страницу Active Numbers (Активные номера)](https://www.twilio.com/console/phone-numbers/incoming) в Twilio.

2. Щелкните номер телефона, который вы создали на предыдущем шаге.

3. В разделе **Messaging** (Сообщения) заполните раздел **A MESSAGE COMES IN** (Поступает сообщение), выбрав **Webhook** (Веб-перехватчик) из раскрывающегося списка и заполнив текстовое поле значением конечной точки бота, которое вы указали в параметре **TwilioValidationUrl** на предыдущем шаге, например `https://yourboturl.com/api/twilio`.

![Настройка веб-перехватчика для номера Twilio](~/media/bot-service-channel-connect-twilio/twilio-number-messaging-settings.png)

4. Нажмите кнопку **Сохранить** .

### <a name="test-your-bot-with-adapter-in-twilio"></a>Тестирование работы бота с адаптером в Twilio

Теперь вы можете проверить, правильно ли подключен бот к Twilio, отправив SMS-сообщение на свой номер Twilio.  Каждый раз, когда ваш бот получает сообщение, он отправляет вам ответ с копией текста вашего сообщения.

Эту функцию также можно протестировать с помощью [примера бота для адаптера Twilio](https://aka.ms/csharp-63-twilio-adapter-sample), заполнив файл appSettings.json теми же значениями, что и в описанных выше шагах.

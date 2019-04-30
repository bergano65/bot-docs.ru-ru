---
title: Добавление проверки подлинности к боту с помощью службы Azure Bot | Документация Майкрософт
description: Сведения об использовании функции проверки подлинности службы Azure Bot для добавления боту функции единого входа.
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 04/09/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2f15817abe087650bc3f2bb998a32f177848cf50
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/22/2019
ms.locfileid: "59904537"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Добавление проверки подлинности к боту с помощью службы Azure Bot

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Служба Azure Bot и пакет SDK версии 4 содержат новые функции проверки подлинности для ботов, которые упрощают разработку бота для проверки подлинности пользователей в различных поставщиках удостоверений, например AAD (Azure Active Directory), GitHub, Uber и т. д. Эти возможности помогают улучшить взаимодействие с пользователем путем устранения _проверки шифра_ для некоторых клиентов.

До этого бот должен включать контроллеры OAuth и ссылки для входа, хранить целевые идентификаторы и секреты клиента и выполнять управление токенами пользователей. Бот будет предлагать пользователю выполнить вход на веб-сайте с последующим созданием _шифра_ для подтверждения личности пользователя.

Теперь разработчикам ботов больше не нужно размещать контроллеры OAuth или управлять жизненным циклом токена, так как все это может выполнять служба Azure Bot.

Доступны следующие функции:

- Усовершенствования каналов поддержки новых функций проверки подлинности, таких как новые библиотеки WebChat и DirectLineJS, позволяют устранить необходимость в шестизначной проверке шифра.
- Усовершенствования портала Azure по добавлению, удалению и настройке параметров подключения к различным поставщикам удостоверений OAuth.
- Поддержка множества встроенных поставщиков удостоверений, включая Azure AD (конечные точки версий 1 и 2), GitHub и другие.
- Обновления пакетов SDK Bot Framework для C# и Node.js, которые дают возможность извлекать токены, создавать OAuthCards и обрабатывать события TokenResponse.
- Примеры создания бота с аутентификацией в Azure AD.

Чтобы добавить к существующему боту следующие функции, можно экстраполировать шаги, описанные в этой статье. Эти примеры ботов демонстрируют новые функции проверки подлинности.

> [!NOTE]
> Функции аутентификации также работают с Bot Builder версии 3. Но в этой статье рассматривается только пример кода версии 4.

### <a name="about-this-sample"></a>Об этом примере

Вам нужно создать ресурс бота Azure, а также новое приложение AAD (версии 1 или 2), чтобы разрешить боту обращаться к Office 365. Ресурс бота регистрирует учетные данные бота. Эти учетные данные вам потребуются для тестирования функций проверки подлинности даже при локальном выполнении кода.

> [!IMPORTANT]
> Каждому зарегистрированному в Azure боту назначается приложение AAD. Но это приложение только защищает доступ из канала к боту.
Вам потребуются дополнительные приложения AAD, по одному для каждого приложения, в которых бот должен выполнять проверку подлинности от имени пользователя.

В этой статье описан пример бота, который подключается к Microsoft Graph с помощью маркера AAD версии 1 или 2. Здесь также рассматривается, как создать и зарегистрировать связанное с ним приложение AAD. При этом будет использоваться код из репозитория GitHub [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples). В этой статье описаны все эти процессы.

- **Создание ресурса бота**.
- **Создание приложения Azure AD**.
- **Регистрация в боте приложения Azure AD**.
- **Подготовка примера кода бота**

Завершив этот процесс, вы получите локально выполняемый бот, который может отвечать на несколько простых действий с использованием приложения AAD, например проверять и отправлять электронную почту или отображать сведения о пользователе и его руководителе. Чтобы это сделать, бот будет использовать токен приложения Azure AD библиотеки Microsoft.Graph. Вам не обязательно публиковать бота, чтобы протестировать возможности OAuth для проверки подлинности, но боту потребуется допустимый идентификатор приложения Azure и пароль к нему.

Эти функции проверки подлинности работают и с другими типами ботов. Но в этом руководстве рассматривается только бот регистрации.

### <a name="web-chat-and-direct-line-considerations"></a>Рекомендации по веб-чатам и Direct Line

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

Если вы намерены применить проверку подлинности службы Azure Bot для веб-чата, важно учитывать несколько вопросов, связанных с безопасностью.

1. Предотвращайте возможность олицетворения, когда злоумышленник выдает себя перед ботом за другое лицо. В веб-чате злоумышленник может олицетворять кого-то, изменив идентификатор пользователя в экземпляре веб-чата.

    Чтобы избежать этого, идентификатор пользователя не должен быть угадываемым. Если вы включите в канале Direct Line расширенную проверку подлинности, служба Azure Bot сможет обнаруживать и отклонять любые изменения идентификатора пользователя. Идентификатор пользователя в сообщениях, отправляемых боту из Direct Line, всегда будет тем же, с которым вы инициализировали веб-чат. Обратите внимание, что для использования этой функции идентификатор пользователя должен начинаться с `dl_`.

1. Следите за тем, чтобы вход выполняли только допустимые пользователи. У пользователя есть два удостоверения: учетные данные в канале и учетные данные от поставщика удостоверений. В веб-чате служба Azure Bot может гарантировать, что процесс входа выполняется в том же сеансе браузера, что и сам веб-чат.

    Чтобы включить такую защиту, запустите веб-чат с маркером Direct Line, который содержит список доверенных доменов, которым разрешено размещать клиент веб-чата для бота. Затем настройте статический список доверенных доменов (источников) на странице настройки Direct Line.

С помощью конечной точки REST `/v3/directline/tokens/generate` Direct Line создайте маркер для беседы и укажите идентификатор пользователя в полезных данных запроса. Пример кода вы найдете в записи блога о [функции расширенной проверки подлинности Direct Line](https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/).

<!-- The eventual article about this should talk about the tokens/generate endpoint and its parameters: user, trustedOrigins, and [maybe] eTag.
Sample payload
{
  "user": {
    "id": "string",
    "name": "string",
    "aadObjectId": "string",
    "role": "string"
  },
  "trustedOrigins": [
    "string"
  ],
  "eTag": "string"
}
 -->

## <a name="prerequisites"></a>Предварительные требования

- Понимание [основ работы ботов][concept-basics] и [управления состоянием][concept-state].
- Понимание процессов разработки для Azure и OAuth 2.0.
- Visual Studio 2017 или более поздней версии, Node.js, npm и GIT.
- Один из этих примеров.

| Образец | Версия Bot Builder | Что демонстрирует |
|:---|:---:|:---|
| **Проверка подлинности бота** на [**CSharp**][cs-auth-sample] или [**JavaScript**][js-auth-sample] | версия 4 | Поддержка OAuthCard |
| **Проверка подлинности бота в MSGraph** на [**CSharp**][cs-msgraph-sample] или [**JavaScript**][js-msgraph-sample] | версия 4 |  Поддержка API Microsoft Graph с использованием OAuth 2 |

## <a name="create-your-bot-resource-on-azure"></a>Создание ресурса бота в Azure

Создайте **Bot Channels Registration** (Бот регистрации каналов) с помощью [портала Azure](https://portal.azure.com/).

## <a name="create-and-register-an-azure-ad-application"></a>Создание и регистрация приложения AAD

Вам потребуется приложение AAD, которое бот сможет использовать для подключения к API Microsoft Graph.

Для данного бота можно использовать конечные точки Azure AD версии 1 или 2.
Дополнительные сведения о разнице между конечными точками версии 1 и 2 см. в статьях [What's different about the v2.0 endpoint?](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare) (Что нового в конечной точке версии 2.0 ) и [Настройка входа пользователей с помощью учетной записи Майкрософт и Azure Active Directory в одном приложении](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview).

### <a name="create-your-azure-ad-application"></a>Создание приложения AAD

Эта процедура позволяет создать приложение AAD. Созданное приложение можно использовать с конечными точками версии 1 или 2.

> [!TIP]
> Вам нужно создать приложение AAD и зарегистрировать его в арендаторе, для работы с которым у вас есть права администратора.

1. Откройте панель [Azure Active Directory][azure-aad-blade] на портале Azure.
    Если вы не попадете сразу в нужный арендатор, щелкните действие **Переключить каталог**. (См. инструкции по [созданию арендатора с помощью портала](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-access-create-new-tenant).)
1. Откройте панель **Регистрация приложений**.
1. На панели **Регистрация приложений** щелкните **Регистрация нового приложения**.
1. Заполните обязательные поля и создайте регистрацию приложения.

   1. Присвойте имя приложению.
   1. В качестве **типа приложения** установите **Веб-приложение или API**.
   1. В качестве значения **URL-адрес входа** установите `https://token.botframework.com/.auth/web/redirect`.
   1. Нажмите кнопку **Создать**.

      - Когда приложение будет создано, оно появится в области **Зарегистрированное приложение**.
      - Запишите значение **идентификатора приложения**. Это значение вам нужно будет ввести позднее в поле _Идентификатор клиента_ при регистрации приложения AAD в боте.

1. Чтобы настроить приложение, щелкните **Параметры**.
1. Чтобы открыть панель **Ключи**, щелкните **Ключи**.

   1. В разделе **Пароли** создайте ключ `BotLogin`.
   1. Установите его **Продолжительность** на **Never expires** (Нелимитированный срок действия).
   1. Нажмите кнопку **Сохранить** и запишите значение ключа. Это значение вам нужно будет ввести позднее в поле _Секрет клиента_ при регистрации приложения AAD в боте.
   1. Закройте панель **Ключи**.

1. Чтобы открыть панель **Требуемые разрешения**, щелкните **Требуемые разрешения**.

   1. Щелкните **Добавить**.
   1. Щелкните **Выбор API**, выберите **Microsoft Graph**, а затем щелкните **Выбор**.
   1. Щелкните **Выбор разрешений**. Выберите делегированные разрешения, которые будут использоваться в приложении.

      > [!NOTE]
      > Для любого разрешения, отмеченного как **Требуются права администратора**, требуется, чтобы пользователь и администратор клиента вошли в систему. Поэтому для бота требуется использовать поменьше разрешений такого типа.

      Выберите следующие делегированные разрешения Microsoft Graph.
      - Чтение базовых профилей всех пользователей.
      - Чтение почты пользователей
      - отправка сообщений в качестве пользователя.
      - Вход в систему и чтение профиля пользователя.
      - Просмотр базовых профилей всех пользователей.
      - Просмотр пользовательских адресов электронной почты.

   1. Щелкните **Выбрать**, а затем нажмите **Готово**.
   1. Закройте панель **Требуемые разрешения**.

Теперь приложение Azure AD версии 1 настроено.

### <a name="register-your-azure-ad-application-with-your-bot"></a>Регистрация в боте приложения Azure AD

Следующим шагом является регистрация приложения AAD в созданном боте.

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD версии 1](#tab/aadv1)

1. На [портале Azure](http://portal.azure.com/) перейдите к странице ресурса бота.
1. Щелкните **Параметры**.
1. В разделе **OAuth Connection Settings** (Параметры подключения OAuth), который находится в нижней части страницы, щелкните **Добавить настройку**.
1. Заполните форму следующим образом.

    1. В поле **Имя** введите имя подключения, которое будет затем использовано в коде бота.
    1. В разделе **Поставщик службы** выберите **Azure Active Directory**. После выбора этого параметра появятся отдельные поля Azure AD.
    1. В поле **Идентификатор клиента** введите идентификатор приложения, который был записан для приложения Azure AD версии 1.
    1. В поле **Секрет клиента** введите ключ, который был записан для ключа приложения `BotLogin`.
    1. В поле **Тип предоставления разрешения** введите `authorization_code`.
    1. В поле **URL-адрес входа** введите `https://login.microsoftonline.com`.
    1. В поле **Идентификатор клиента** введите идентификатор клиента Azure Active Directory, например `microsoft.com` или `common`.

       Данный клиент будет связан с пользователями, которые могут пройти проверку подлинности. Чтобы позволить кому-либо пройти проверку подлинности с помощью бота, используйте клиент `common`.

    1. В поле **URL-адрес ресурса** введите `https://graph.microsoft.com/`.
    1. Оставите поле **Области** пустым.

1. Выберите команду **Сохранить**.

> [!NOTE]
> Используя API Microsoft Graph, эти значения позволяют приложению получать доступ к данным Office 365.

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD версии 2](#tab/aadv2)

1. Перейдите к странице регистрации каналов бота на [портале Azure](http://portal.azure.com/).
1. Щелкните **Параметры**.
1. В разделе **OAuth Connection Settings** (Параметры подключения OAuth), который находится в нижней части страницы, щелкните **Добавить настройку**.
1. Заполните форму следующим образом.

    1. В поле **Имя** введите имя подключения, которое было использовано в коде бота.
    1. В разделе **Поставщик службы** выберите **Azure Active Directory v2**. После выбора этого параметра появятся отдельные поля Azure AD.
    1. В поле **Идентификатор клиента** введите идентификатор приложения Azure AD версии 2, который был использован при регистрации приложения.
    1. В поле **Секрет клиента** введите пароль приложения Azure AD версии 2, который был использован при регистрации приложения.
    1. В поле **Идентификатор клиента** введите идентификатор клиента Azure Active Directory, например `microsoft.com` или `common`.

        Данный клиент будет связан с пользователями, которые могут пройти проверку подлинности. Чтобы позволить кому-либо пройти проверку подлинности с помощью бота, используйте клиент `common`.

    1. В поле **Области** введите имя разрешения, которое было выбрано при регистрации приложения `Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`.

        > [!NOTE]
        > Для приложения Azure AD версии 2 в поле **Области** можно вносить списки значений, которые являются чувствительными к регистру и разделенные пробелами.

1. Выберите команду **Сохранить**.

> [!NOTE]
> Используя API Microsoft Graph, эти значения позволяют приложению получать доступ к данным Office 365.

---

### <a name="test-your-connection"></a>Тестирование подключения

1. Щелкните запись созданного подключения, чтобы открыть его.
1. Щелкните **Проверка подключения** в верхней части области **Service Provider Connection Setting** (Параметры подключения поставщика службы).
1. При выполнении данного действия в первый раз должна открыться новая вкладка браузера с перечисленными разрешениями, которые запрашивает приложение, и предложение их принять.
1. Нажмите кнопку **Принимаю**.
1. Вы перейдете на страницу **Проверка подключения к \<имя_подключения> успешно выполнена**.

Для получения токенов пользователя можно использовать имя этого подключения в коде бота.

## <a name="prepare-the-bot-sample-code"></a>Подготовка примера кода бота

В зависимости от выбранного примера вы будете работать с C# или Node.

| Образец | Версия Bot Builder | Что демонстрирует |
|:---|:---:|:---|
| **Проверка подлинности бота** на [**CSharp**][cs-auth-sample] или [**JavaScript**][js-auth-sample] | версия 4 | Поддержка OAuthCard |
| **Проверка подлинности бота в MSGraph** на [**CSharp**][cs-msgraph-sample] или [**JavaScript**][js-msgraph-sample] | версия 4 |  Поддержка API Microsoft Graph с использованием OAuth 2 |

1. Щелкните ссылку для одного из примеров выше и клонируйте репозиторий github.
1. Следуйте инструкциям на странице readme для репозитория GitHub, где описан запуск конкретного бота (для C# или Node).
1. Если вы используете пример бота с аутентификацией для C#, выполните следующее.

    1. Сохраните значение, указанное при настройке параметра подключения бота OAuth 2.0, в переменную `ConnectionName` в файле `AuthenticationBot.cs`.
    1. Сохраните идентификатор приложения бота в параметр `appId` в файле `BotConfiguration.bot`.
    1. Сохраните секрет бота в параметр `appPassword` в файле `BotConfiguration.bot`.

1. Если вы используете пример для Node/JS, выполните следующее.

    1. Сохраните значение, указанное при настройке параметра подключения бота OAuth 2.0, в переменную `CONNECTION_NAME` в файле `bot.js`.
    1. Сохраните идентификатор приложения бота в параметр `appId` в файле `bot-authentication.bot`.
    1. Сохраните секрет бота в параметр `appPassword` в файле `bot-authentication.bot`.

    > [!IMPORTANT]
    > В зависимости от символов, находящихся в секрете, может потребоваться сменить пароль для XML. Например, символ амперсанда (&) потребуется кодировать как `&amp;`.

    ```json
    {
        "name": "BotAuthentication",
        "secretKey": "",
        "services": [
            {
            "appId": "",
            "id": "http://localhost:3978/api/messages",
            "type": "endpoint",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages",
            "name": "BotAuthentication"
            }
        ]
    }
    ```

Если вы не знаете, как получить **идентификатор приложения Майкрософт** и **пароль приложения Майкрософт**, можете создать новый пароль, как описано здесь:

[Пароль службы "Регистрация каналов бота"](../bot-service-quickstart-registration.md#bot-channels-registration-password)
  
Или получите **идентификатор приложения Майкрософт** и **пароль приложения Майкрософт**, предоставляемые при **регистрации каналов бота**, из развертывания, как описано в статье [Find Your Azure Bot’s AppID and AppSecret](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret) (Поиск идентификатора и секрета приложения для бота Azure).

    > [!NOTE]
    > You could now publish this bot code to your Azure subscription (right-click on the project and choose **Publish**), but it is not necessary for this tutorial. You would need to set up a publishing configuration that uses the application and hosting plan that you used when configuration the bot in the Azure Portal.

## <a name="use-the-emulator-to-test-your-bot"></a>Подключение эмулятора Bot Framework для проверки бота

Для локальной проверки бота требуется установить [Bot Emulator](https://github.com/Microsoft/BotFramework-Emulator). Можно использовать Bot Framework Emulator версии 3 или 4.

1. Запустите бот (с отладкой или без).
1. Запишите номер порта localhost страницы. Данная информация потребуется для взаимодействия с ботом.
1. Запустите эмулятор Bot Framework.
1. Подключитесь к боту. Если вы используете аутентификацию, в конфигурации бота следует использовать **идентификатор приложения Microsoft** и **пароль приложения Microsoft**.
1. Убедитесь, что включен параметр эмулятора **Use a sign-in verification code for OAuthCards** (Использовать код проверки входа в систему для OAuthCards) и включен **ngrok**, чтобы служба Azure Bot могла вернуть эмулятору маркер, когда он станет доступен.

   Если подключение еще не было настроено, необходимо предоставить адрес, а также идентификатор приложения Майкрософт и пароль для бота. К каждому из URL-адресов требуется добавить `/api/messages`. URL-адрес должен выглядеть примерно так: `http://localhost:portNumber/api/messages`.

1. Чтобы просмотреть список доступных команд для бота и проверить функции проверки подлинности, введите `help`.
1. После выполнения входа и до момента выхода не требуется повторно предоставлять учетные данные.
1. Чтобы выйти и отменить проверку подлинности, введите `signout`.

<!--To restart completely from scratch you also need to:
1. Navigate to the **AppData** folder for your account.
1. Go to the **Roaming/botframework-emulator** subfolder.
1. Delete the **Cookies** and **Coolies-journal** files.
-->

> [!NOTE]
> Для использования проверки подлинности бота требуется служба Bot Connector. Служба получает доступ к сведениям о боте регистрации каналов, поэтому на портале требуется установить конечную точку обмена сообщениями бота. Для проверки подлинности также требуется использовать протокол HTTPS. Поэтому для бота, запущенного локально, необходимо создать адрес перенаправления HTTPS.

<!--The following is necessary for WebChat:
1. Use the **ngrok** command-line tool to get a forwarding HTTPS address for your bot.
   - For information on how to do this, see [Debug any Channel locally using ngrok](https://blog.botframework.com/2017/10/19/debug-channel-locally-using-ngrok/).
   - Any time you exit **ngrok**, you will need to redo this and the following step before starting the Emulator.
1. On the Azure Portal, go to the **Settings** blade for your bot.
   1. In the **Configuration** section, change the **Messaging endpoint** to the HTTPS forwarding address generated by **ngrok**.
   1. Click **Save** to save your change.
-->

## <a name="notes-on-the-token-retrieval-flow"></a>Примечания о потоке извлечения токена

Когда пользователь приказывает боту выполнить некоторые действия, которые требуют пользовательского входа, бот может с помощью `OAuthPrompt` инициировать извлечение токена для данного соединения. `OAuthPrompt` создает поток получения маркера, который выполняет следующие действия.

1. Проверяет, имеет ли служба Azure Bot маркер для текущего сочетания пользователя и подключения. Если маркер есть, он возвращается.
1. Если служба Azure Bot не имеет нужного маркера в кэше, создается `OAuthCard` с кнопкой входа, на которую может нажать пользователь.
1. Когда пользователь нажимает на кнопку входа `OAuthCard`, служба Azure Bot выполняет одно из двух действий: напрямую отправляет боту маркер пользователя или предоставляет пользователю шестизначный код аутентификации для ввода в окно чата.
1. Если пользователю предоставлен код аутентификации, бот позднее обменивает этот код на маркер пользователя.

Следующие фрагменты кода взяты из `OAuthPrompt`, где демонстрируется работа этих шагов в командной строке.

### <a name="check-for-a-cached-token"></a>Проверка кэшированного токена

Как видно из этого кода, сначала бот выполняет быструю проверку, чтобы определить, имеет ли служба Azure Bot токен пользователя (который идентифицирован текущим отправителем действия) и полученное ConnectionName (которое является именем подключения, используемое в конфигурации). К этому моменту служба Azure Bot уже будет обладать кэшированным токеном (или не будет). Эту быструю проверку можно выполнить, вызвав метод GetUserTokenAsync. Если служба Azure Bot имеет токен и возвращает его, он может быть использован мгновенно. Если служба Azure Bot не имеет токена, результатом выполнения этого метода будет NULL. В таком случае бот может отправить пользователю для входа настраиваемый OAuthCard.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// First ask Bot Service if it already has a token for this user
var token = await adapter.GetUserTokenAsync(turnContext, connectionName, null, cancellationToken).ConfigureAwait(false);
if (token != null)
{
    // use the token to do exciting things!
}
else
{
    // If Bot Service does not have a token, send an OAuth card to sign in
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
public async getUserToken(context: TurnContext, code?: string): Promise<TokenResponse|undefined> {
    // Get the token and call validator
    const adapter: any = context.adapter as any; // cast to BotFrameworkAdapter
    return await adapter.getUserToken(context, this.settings.connectionName, code);
}
```

---

### <a name="send-an-oauthcard-to-the-user"></a>Отправка OAuthCard пользователю

Для настраивания OAuthCard можно использовать любой текст и текст кнопки. Ниже приведены важные действия, которые необходимо выполнить.

- Измените `ContentType` на `OAuthCard.ContentType`.
- Замените значение свойства `ConnectionName` на имя требуемого подключения.
- Включите одну кнопку с действием `CardAction` `Type` `ActionTypes.Signin`. Обратите внимание, что для ссылки входа не требуется указывать какие-либо значения.

В конце вызова боту требуется подождать на "возвращение токена". Ожидание происходит в основной ленте активности, потому что для входа в систему может потребоваться много времени.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private async Task SendOAuthCardAsync(ITurnContext turnContext, IMessageActivity message, CancellationToken cancellationToken = default(CancellationToken))
{
    if (message.Attachments == null)
    {
        message.Attachments = new List<Attachment>();
    }

    message.Attachments.Add(new Attachment
    {
        ContentType = OAuthCard.ContentType,
        Content = new OAuthCard
        {
            Text = "Please sign in",
            ConnectionName = connectionName,
            Buttons = new[]
            {
                new CardAction
                {
                    Title = "Sign In",
                    Text = "Sign In",
                    Type = ActionTypes.Signin,
                },
            },
        },
    });

    await turnContext.SendActivityAsync(message, cancellationToken).ConfigureAwait(false);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
private async sendOAuthCardAsync(context: TurnContext, prompt?: string|Partial<Activity>): Promise<void> {
    // Initialize outgoing message
    const msg: Partial<Activity> =
        typeof prompt === 'object' ? {...prompt} : MessageFactory.text(prompt, undefined, InputHints.ExpectingInput);
    if (!Array.isArray(msg.attachments)) { msg.attachments = []; }

    const cards: Attachment[] = msg.attachments.filter((a: Attachment) => a.contentType === CardFactory.contentTypes.oauthCard);
    if (cards.length === 0) {
        // Append oauth card
        msg.attachments.push(CardFactory.oauthCard(
            this.settings.connectionName,
            this.settings.title,
            this.settings.text
        ));
    }

    // Send prompt
    await context.sendActivity(msg);
}
```

---

### <a name="wait-for-a-tokenresponseevent"></a>Ожидание метода TokenResponseEvent

В этом коде Bot ожидает `TokenResponseEvent` (дополнительные сведения о том, как этот метод перенаправляется в стек Dialog, приведены ниже). Метод `WaitForToken` сначала определяет, было ли это отправлено. Если да, то оно может быть использовано ботом. Если нет, метод `RecognizeTokenAsync` принимает любой текст, отправленный боту, и передает его `GetUserTokenAsync`. Данное действие выполняется, потому что клиенту (например WebChat) не требуется код проверки подлинности шифра, и он может напрямую отсылать маркер в `TokenResponseEvent`. Другие клиенты (например Facebook или Slack) все еще требуют шифр. Служба Azure Bot представит этим клиентам шестизначный шифр и попросит пользователя ввести его в окно чата. Это не оптимальное решение, но оно применяется в качестве резервного. Таким образом, если `RecognizeTokenAsync` получает код, бот может отправить его в службу Azure Bot и получить маркер. Если данный вызов также потерпит неудачу, пользователь может принять решение об отправке отчета об ошибке, или выполнить другое действие. В большинстве случаев бот получит токен пользователя.

Изучив код каждого из примеров бота, вы увидите, что действия `Event` и `Invoke` также направляются в стек диалогов.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// This can be called when the bot receives an Activity after sending an OAuthCard
private async Task<TokenResponse> RecognizeTokenAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    if (IsTokenResponseEvent(turnContext))
    {
        // The bot received the token directly
        var tokenResponseObject = turnContext.Activity.Value as JObject;
        var token = tokenResponseObject?.ToObject<TokenResponse>();
        return token;
    }
    else if (IsTeamsVerificationInvoke(turnContext))
    {
        var magicCodeObject = turnContext.Activity.Value as JObject;
        var magicCode = magicCodeObject.GetValue("state")?.ToString();

        var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, magicCode, cancellationToken).ConfigureAwait(false);
        return token;
    }
    else if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // make sure it's a 6-digit code
        var matched = _magicCodeRegex.Match(turnContext.Activity.Text);
        if (matched.Success)
        {
            var token = await adapter.GetUserTokenAsync(turnContext, _settings.ConnectionName, matched.Value, cancellationToken).ConfigureAwait(false);
            return token;
        }
    }

    return null;
}

private bool IsTokenResponseEvent(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Event && activity.Name == "tokens/response";
}

private bool IsTeamsVerificationInvoke(ITurnContext turnContext)
{
    var activity = turnContext.Activity;
    return activity.Type == ActivityTypes.Invoke && activity.Name == "signin/verifyState";
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
private async recognizeToken(context: TurnContext): Promise<PromptRecognizerResult<TokenResponse>> {
    let token: TokenResponse|undefined;
    if (this.isTokenResponseEvent(context)) {
        token = context.activity.value as TokenResponse;
    } else if (this.isTeamsVerificationInvoke(context)) {
        const code: any = context.activity.value.state;
        await context.sendActivity({ type: 'invokeResponse', value: { status: 200 }});
        token = await this.getUserToken(context, code);
    } else if (context.activity.type === ActivityTypes.Message) {
        const matched: RegExpExecArray = /(\d{6})/.exec(context.activity.text);
        if (matched && matched.length > 1) {
            token = await this.getUserToken(context, matched[1]);
        }
    }

    return token !== undefined ? { succeeded: true, value: token } : { succeeded: false };
}

private isTokenResponseEvent(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Event && activity.name === 'tokens/response';
}

private isTeamsVerificationInvoke(context: TurnContext): boolean {
    const activity: Activity = context.activity;
    return activity.type === ActivityTypes.Invoke && activity.name === 'signin/verifyState';
}
```

---

### <a name="message-controller"></a>Контроллер сообщений

При последующих вызовах бота обратите внимание, что этот токен никогда не кэшируется ботом из этого примера. Это связано с тем, что бот всегда может запросить токен в службе Azure Bot. Это позволяет избежать того, что боту необходимо управлять жизненным циклом токена, обновлять токен и т. д., поскольку служба Azure Bot делает все это сама.

### <a name="further-reading"></a>Дополнительные материалы

- [Дополнительные ресурсы Bot Framework](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help) содержат ряд ссылок с сопроводительной информацией.
- Репозиторий с [пакетом SDK Bot Framework ](https://github.com/microsoft/botbuilder) содержит дополнительные сведения о репозиториях, примерах, средствах и спецификациях для пакета SDK Bot Builder.

<!-- Footnote-style links -->

[Azure portal]: https://ms.portal.azure.com
[azure-aad-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
[aad-registration-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-auth-sample]: https://aka.ms/v4cs-bot-auth-sample
[js-auth-sample]: https://aka.ms/v4js-bot-auth-sample
[cs-msgraph-sample]: https://aka.ms/v4cs-auth-msgraph-sample
[js-msgraph-sample]: https://aka.ms/v4js-auth-msgraph-sample

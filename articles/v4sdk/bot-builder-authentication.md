---
title: Добавление проверки подлинности к боту с помощью службы Azure Bot | Документация Майкрософт
description: Сведения об использовании функции проверки подлинности службы Azure Bot для добавления боту функции единого входа.
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f41409b534b997d486a94dc206ecc85bf6f0ca9e
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215556"
---
<!-- Related TODO:
- Check code in [Web Chat channel](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0)
- Check guidance in [DirectLine authentication](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)
-->

<!-- General TODO: (Feedback from CSE (Nafis))
- Add note that: token management is based on user ID
- Explain why/how to share existing website authentication with a bot.
- Risk: Even people who use a DirectLine token can be vulnerable to user ID impersonation.
    Docs/samples that show exchange of secret for token don't specify a user ID, so an attacker can impersonate a different user by modifying the ID client side. There's a [blog post](https://nam06.safelinks.protection.outlook.com/?url=https%3A%2F%2Fblog.botframework.com%2F2018%2F09%2F01%2Fusing-webchat-with-azure-bot-services-authentication%2F&data=02%7C01%7Cv-jofing%40microsoft.com%7Cee005e1c9d2c4f4e7ea508d6b231b422%7C72f988bf86f141af91ab2d7cd011db47%7C1%7C0%7C636892323874079713&sdata=b0DWMxHzmwQvg5EJtlqKFDzR7fYKmg10fXma%2B8zGqEI%3D&reserved=0) that shows how to do this properly.
"Major issues":
- This doc is a sample walkthrough, but there's no deeper documentation explaining how the Azure Bot Service is handling tokens. How does the OAuth flow work? Where is it storing my users' access tokens? What's the security and best practices around using it?

"Minor issues":
- AAD v2 steps tell you to add delegated permission scopes during registration, but this shouldn't be necessary in AAD v2 due to dynamic scopes. (Ming, "This is currently necessary because scopes are not exposed through our runtime API. We don’t currently have a way for the developer to specify which scope he wants at runtime.")

- "The scope of the connection setting needs to have both openid and a resource in the Azure AD graph, such as Mail.Read." Unclear if I need to take some action at this point to make happen. Kind of out of context. I'm registering an AAD application in the portal, there's no connection setting
- Does the bot need all of these scopes for the samples? (e.g. "Read all users' basic profiles")
-->

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

## <a name="prerequisites"></a>Предварительные требования

- Понимание [основных принципов работы ботов][concept-basics], [управления состоянием][concept-state], [библиотек диалогов][concept-dialogs], [реализации последовательного потока беседы][simple-dialog], [получения данных от пользователя с помощью диалога запросов][dialog-prompts] и [повторного использования диалогов][component-dialogs].
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

# <a name="azure-ad-v1tabaadv1"></a>[Azure AD версии 1](#tab/aadv1)

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

# <a name="azure-ad-v2tabaadv2"></a>[Azure AD версии 2](#tab/aadv2)

1. Перейдите на [портал регистрации приложений Майкрософт](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade).
1. Нажмите кнопку **Добавить приложение**.
1. Назовите приложение Azure AD и щелкните **Создать**.

    Запишите GUID **идентификатора приложения**. Эти данные понадобятся позже в качестве идентификатора клиента для настройки подключения.

1. В разделе **Application Secrets** (Секреты приложения) щелкните **Generate New Password** (Создать новый пароль).

    Запишите пароль из всплывающего окна. Эти данные понадобятся позже в качестве секрета клиента для настройки подключения.

1. В разделе **Платформы** щелкните **Добавление платформы**.
1. Во всплывающем окне **Добавление платформы** щелкните **Web**.

    1. Установите флажок **Разрешить неявный поток**.
    1. В поле **URL-адрес перенаправления** введите `https://token.botframework.com/.auth/web/redirect`.
    1. Оставьте поле **URL-адрес выхода** пустым.

1. В разделе **Разрешения Microsoft Graph** можно добавить дополнительные делегированные разрешения.

    - Для выполнения задач из этой статьи добавьте разрешения **Mail.Read**, **Mail.Send**, **openid**, **profile**, **User.Read** и **User.ReadBasic.All**.
      В области настройки подключения должно находиться как разрешение **openid**, так и ресурс Azure AD Graph, например **Mail.Read**.
    - Запишите выбранное разрешение. Это разрешение понадобится позже в качестве области для настройки подключения.

1. В нижней части страницы нажмите кнопку **Сохранить** .

Теперь приложение Azure AD версии 2 настроено.

---

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

## <a name="prepare-the-bot-code"></a>Подготовка кода бота

Вам потребуются идентификатор приложения бота и пароль, чтобы выполнить этот процесс. Чтобы получить идентификатор приложения бота и пароль, сделайте следующее:

1. На [портал Azure][] перейдите к группе ресурсов, в которой вы создали бот регистрации каналов.
1. Откройте панель **Развертывания**, затем откройте развертывание нужного бота.
1. Откройте панель **Входные данные** и скопируйте значения **appId** и **appSecret** для этого бота.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- TODO: Add guidance (once we have it) on how not to hard-code IDs and ABS auth. -->

1. Клонируйте нужный пример из репозитория GitHub: [**Аутентификация бота**][cs-auth-sample] или [**Аутентификация бота MSGraph**][cs-msgraph-sample].
1. Следуйте инструкциям на странице readme для репозитория GitHub, где описан запуск конкретного бота. <!--TODO: Can we remove this step and still have the instructions make sense? What is the minimum we need to say in its place? -->
1. Обновите файл **appsettings.json**:

    - параметру `ConnectionName` присвойте значение имени подключения OAuth, которое вы добавили в бот;
    - параметрам `MicrosoftAppId` и `MicrosoftAppPassword` присвойте значения идентификатора приложения бота и секрета приложения.

      В зависимости от символов, из которых состоит секрет бота, может потребоваться XML-экранирование пароля. Например, символ амперсанда (&) потребуется кодировать как `&amp;`.

[!code-json[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/appsettings.json)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

1. Клонируйте нужный пример из репозитория GitHub: [**Аутентификация бота**][js-auth-sample] или [**Аутентификация бота MSGraph**][js-msgraph-sample].
1. Следуйте инструкциям на странице readme для репозитория GitHub, где описан запуск конкретного бота. <!--TODO: Can we remove this step and still have the instructions make sense? What is the minimum we need to say in its place? -->
1. Обновите файл с расширением **.env**:

    - параметру `connectionName` присвойте значение имени подключения OAuth, которое вы добавили в бот;
    - параметрам `MicrosoftAppId` и `MicrosoftAppPassword` присвойте значения идентификатора приложения бота и секрета приложения.

      В зависимости от символов, из которых состоит секрет бота, может потребоваться XML-экранирование пароля. Например, символ амперсанда (&) потребуется кодировать как `&amp;`.

    [!code-txt[.env](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/.env)]

---

Если вы не знаете, как получить **идентификатор приложения Майкрософт** и **пароль приложения Майкрософт**, можно выбрать любой из следующих вариантов:

- создайте новый пароль, [как описано здесь](../bot-service-quickstart-registration.md#bot-channels-registration-password);
- получите **идентификатор приложения Майкрософт** и **пароль приложения Майкрософт**, предоставляемые при **регистрации каналов бота**, из развертывания, как описано [здесь](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret).

> [!NOTE]
> Теперь код бота можно опубликовать в подписке Azure (щелкните проект правой кнопкой мыши и выберите **Опубликовать**), но для этой статьи это не требуется. Необходимо будет настроить конфигурацию публикации, которая использует план приложения и размещения, который использовался при настройке бота на портале Azure.

## <a name="test-the-bot"></a>Тестирование бота

1. Установите [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme), если вы этого еще не сделали.
1. Выполните этот пример на локальном компьютере.
1. Запустите эмулятор, подключитесь к боту и отправьте несколько сообщений.

    - При подключении к боту необходимо указать идентификатор приложения бота и пароль.
    - Чтобы просмотреть список доступных команд для бота и проверить функции проверки подлинности, введите `help`.
    - После выполнения входа и до момента выхода не требуется повторно предоставлять учетные данные.
    - Чтобы выйти и отменить проверку подлинности, введите `logout`.

> [!NOTE]
> Для использования проверки подлинности бота требуется служба Bot Connector. Эта служба использует сведения о регистрации каналов бота.

# <a name="bot-authenticationtabbot-oauth"></a>[Аутентификация бота](#tab/bot-oauth)

В примере **Аутентификация бота** диалог получает маркер пользователя после входа пользователя в систему.

![Пример выходных данных](media/how-to-auth/auth-bot-test.png)

# <a name="bot-authentication-msgraphtabbot-msgraph-auth"></a>[Аутентификация бота MSGraph](#tab/bot-msgraph-auth)

В примере **Аутентификация бота MSGraph** диалог принимает ограниченный набор команд после входа пользователя в систему.

![Пример выходных данных](media/how-to-auth/msgraph-bot-test.png)

---

## <a name="additional-information"></a>Дополнительная информация

Когда пользователь приказывает боту выполнить некоторые действия, которые требуют пользовательского входа, бот может с помощью `OAuthPrompt` инициировать извлечение токена для данного соединения. `OAuthPrompt` создает поток получения маркера, который выполняет следующие действия.

1. Проверяет, имеет ли служба Azure Bot маркер для текущего сочетания пользователя и подключения. Если маркер есть, он возвращается.
1. Если служба Azure Bot не имеет нужного маркера в кэше, создается `OAuthCard` с кнопкой входа, на которую может нажать пользователь.
1. Когда пользователь нажимает на кнопку входа `OAuthCard`, служба Azure Bot выполняет одно из двух действий: напрямую отправляет боту маркер пользователя или предоставляет пользователю шестизначный код аутентификации для ввода в окно чата.
1. Если пользователю предоставлен код аутентификации, бот позднее обменивает этот код на маркер пользователя.

В следующих разделах объясняется, как этот пример реализует некоторые распространенные задачи проверки подлинности.

### <a name="use-an-oauth-prompt-to-sign-the-user-in-and-get-a-token"></a>Использование запроса OAuth для входа пользователя и получения маркера

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![Архитектура бота](media/how-to-auth/architecture.png)

<!-- The two authentication samples have nearly identical architecture. Using 18.bot-authentication for the sample code. -->

**Dialogs\MainDialog.cs**

Добавьте запрос OAuth в **MainDialog** с помощью конструктора. Здесь мы получаем значение имени подключения из файла **appsettings.json**.

[!code-csharp[Add OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=23-31)]

В шаге диалога укажите `BeginDialogAsync` для запуска командной строки OAuth, в которой пользователю будет предложено войти в систему.

- Если пользователь уже выполнил вход, будет создано событие ответа маркера без подтверждения пользователя.
- В противном случае пользователю будет предложено войти в систему. Служба Azure Bot отправляет событие ответа маркера после попытки входа пользователя.

[!code-csharp[Use the OAuthPrompt](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=49)]

В следующем шаге диалога проверьте наличие маркера в результате, полученном от предыдущего шага. Если он не равен NULL, значит пользователь успешно выполнил вход.

[!code-csharp[Get the OAuthPrompt result](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=54-58)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![Архитектура бота](media/how-to-auth/architecture-js.png)

**dialogs/mainDialog.js**

Добавьте запрос OAuth в **MainDialog** с помощью конструктора. В этом примере значение имени подключения получено из файла с расширением **.env**.

[!code-javascript[Add OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=23-28)]

В шаге диалога укажите `beginDialog` для запуска командной строки OAuth, в которой пользователю будет предложено войти в систему.

- Если пользователь уже выполнил вход, будет создано событие ответа маркера без подтверждения пользователя.
- В противном случае пользователю будет предложено войти в систему. Служба Azure Bot отправляет событие ответа маркера после попытки входа пользователя.

[!code-javascript[Use OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=57)]

В следующем шаге диалога проверьте наличие маркера в результате, полученном от предыдущего шага. Если он не равен NULL, значит пользователь успешно выполнил вход.

[!code-javascript[Get OAuthPrompt result](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=61-64)]

---

### <a name="wait-for-a-tokenresponseevent"></a>Ожидание метода TokenResponseEvent

Когда вы запускаете командную строку OAuth, она ожидает события получения маркера, из которого затем извлекает маркер пользователя.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots\AuthBot.cs**

**AuthBot** наследуется от `ActivityHandler` и явным образом обрабатывает действия ответа маркера. Здесь мы продолжаем активный диалог, что позволяет командной строке OAuth обработать событие и получить маркер.

[!code-csharp[OnTokenResponseEventAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Bots/AuthBot.cs?range=32-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/authBot.js**

**AuthBot** наследуется от `ActivityHandler` и явным образом обрабатывает действия ответа маркера. Здесь мы продолжаем активный диалог, что позволяет командной строке OAuth обработать событие и получить маркер.

[!code-javascript[onTokenResponseEvent](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/bots/authBot.js?range=28-33)]

---

### <a name="log-the-user-out"></a>Выход пользователя

Мы рекомендуем предоставлять пользователям возможность явно выйти из системы, не полагаясь на автоматический разрыв подключения при превышении времени ожидания.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\LogoutDialog.cs**

[!code-csharp[Allow logout](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/LogoutDialog.cs?range=20-61&highlight=35)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/logoutDialog.js**

[!code-javascript[Allow logout](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/logoutDialog.js?range=13-42&highlight=25)]

---

### <a name="further-reading"></a>Дополнительные материалы

- [Дополнительные ресурсы Bot Framework](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help) содержат ряд ссылок с сопроводительной информацией.
- Репозиторий с [пакетом SDK Bot Framework ](https://github.com/microsoft/botbuilder) содержит дополнительные сведения о репозиториях, примерах, средствах и спецификациях для пакета SDK Bot Builder.

<!-- Footnote-style links -->

[портал Azure]: https://ms.portal.azure.com
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

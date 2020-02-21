---
title: Включение проверки подлинности для бота с помощью службы Azure Bot — Служба Azure Bot
description: Сведения об использовании функции проверки подлинности службы Azure Bot для добавления боту функции единого входа.
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/7/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e1cdfbb96e14175f764b481f9634c2d7c93f0c29
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2020
ms.locfileid: "77441640"
---
<!-- 

Related TODO:
- Check code in [Web Chat channel](https://docs.microsoft.com/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0)
- Check guidance in [DirectLine authentication](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0)
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

См. подробнее о том, как [служба Azure Bot выполняет проверку подлинности пользователей в диалоге](bot-builder-concept-authentication.md).

Чтобы добавить к существующему боту следующие функции, можно экстраполировать шаги, описанные в этой статье. Эти примеры ботов демонстрируют новые функции проверки подлинности.

> [!NOTE]
> Функции аутентификации также работают с Bot Builder версии 3. Но в этой статье рассматривается только пример кода версии 4.

### <a name="about-this-sample"></a>Об этом примере

Чтобы создать ресурс Azure Bot, вам потребуется следующее:

1. Регистрация приложения Azure AD, чтобы ваш бот мог получить доступ к внешним ресурсам, например Office 365.
1. Отдельный ресурс бота. Ресурс бота регистрирует учетные данные бота, которые нужны для тестирования функций проверки подлинности даже при локальном выполнении кода.

> [!IMPORTANT]
> Каждому зарегистрированному в Azure боту назначается приложение AAD. Но это приложение только защищает доступ из канала к боту. Вам потребуются дополнительные приложения AAD, по одному для каждого приложения, в которых бот должен выполнять проверку подлинности от имени пользователя.

В этой статье описан пример бота, который подключается к Microsoft Graph с помощью маркера AAD версии 1 или 2. Здесь также рассматривается, как создать и зарегистрировать связанное с ним приложение AAD. При этом будет использоваться код из репозитория GitHub [Microsoft/BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples). В этой статье описаны все эти процессы.

- **Создание ресурса бота**.
- **Создание приложения Azure AD**.
- **Регистрация в боте приложения Azure AD**.
- **Подготовка примера кода бота**

Завершив этот процесс, вы получите локально выполняемый бот, который может отвечать на несколько простых действий с использованием приложения AAD, например проверять и отправлять электронную почту или отображать сведения о пользователе и его руководителе. Чтобы это сделать, бот будет использовать токен приложения Azure AD библиотеки Microsoft.Graph. Вам не обязательно публиковать бота, чтобы протестировать возможности OAuth для проверки подлинности, но боту потребуется допустимый идентификатор приложения Azure и пароль к нему.

### <a name="web-chat-and-direct-line-considerations"></a>Рекомендации по веб-чатам и Direct Line

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

> [!IMPORTANT]
> Учитывайте эти важные [вопросы, связанные с безопасностью](../rest-api/bot-framework-rest-direct-line-3-0-authentication.md#security-considerations).

## <a name="prerequisites"></a>Предварительные требования

- Понимание [основных принципов работы ботов][concept-basics], [управления состоянием][concept-state], [библиотек диалогов][concept-dialogs], [реализации последовательного процесса общения][simple-dialog], а также [повторного использования диалогов][component-dialogs].
- Понимание процессов разработки для Azure и OAuth 2.0.
- Visual Studio 2017 или более поздней версии, Node.js, npm и GIT.
- Один из этих примеров.

| Образец | Версия Bot Builder | Что демонстрирует |
|:---|:---:|:---|
| **Пример с аутентификацией бота** для [**C#** ][cs-auth-sample], [**JavaScript**][js-auth-sample] или [**Python**][python-auth-sample] | версия 4 | Поддержка OAuthCard |
| **Пример с аутентификацией бота (MSGraph)** для [**C#** ][cs-msgraph-sample], [**JavaScript**][js-msgraph-sample] или [**Python**](https://aka.ms/bot-auth-msgraph-python-sample-code)| версия 4 |  Поддержка API Microsoft Graph с использованием OAuth 2 |

## <a name="create-your-bot-resource-on-azure"></a>Создание ресурса бота в Azure

Создайте **ресурс бота** с помощью [портала Azure](https://portal.azure.com/).

См. также о [создании бота с помощью службы Azure Bot](./abs-quickstart.md).

## <a name="create-and-register-an-azure-ad-application"></a>Создание и регистрация приложения AAD

Вам потребуется приложение AAD, которое бот сможет использовать для подключения к API Microsoft Graph.

Для данного бота можно использовать конечные точки Azure AD версии 1 или 2.
Дополнительные сведения о разнице между конечными точками версии 1 и 2 см. в статьях [What's different about the v2.0 endpoint?](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare) (Что нового в конечной точке версии 2.0 ) и [Настройка входа пользователей с помощью учетной записи Майкрософт и Azure Active Directory в одном приложении](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview).

### <a name="create-your-azure-ad-application"></a>Создание приложения AAD

Эта процедура позволяет создать приложение AAD. Созданное приложение можно использовать с конечными точками версии 1 или 2.

> [!TIP]
> Вам понадобится создать и зарегистрировать приложение Azure AD в клиенте, в котором можно дать согласие на делегирование разрешений, запрошенных приложением.

1. Откройте панель [Azure Active Directory][azure-aad-blade] на портале Azure.
    Если вы не попадете сразу в нужный арендатор, щелкните действие **Переключить каталог**. (См. инструкции по [созданию арендатора с помощью портала](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant).)
1. Откройте панель **Регистрация приложений**.
1. На панели **Регистрация приложений** щелкните **Новая регистрация**.
1. Заполните обязательные поля и создайте регистрацию приложения.

   1. Присвойте имя приложению.
   1. Выберите **Поддерживаемые типы учетных записей** для приложения. (Все эти параметры будут работать с этим примером.)
   1. Для **URI перенаправления**:
       1. Выберите **Интернет**.
       1. Укажите для URL-адреса значение `https://token.botframework.com/.auth/web/redirect`.
   1. Щелкните **Зарегистрировать**.

      - После создания в Azure отображается страница **Обзор** для приложения.
      - Запишите **идентификатор приложения (клиента)** . Это значение вам нужно будет ввести позднее в поле _Идентификатор клиента_ при регистрации приложения AAD в боте.
      - Также запишите **идентификатор каталога (арендатора)** . Вы будете использовать эти значения для регистрации приложения с ботом.

1. В области навигации щелкните **Сертификаты и секреты**, чтобы создать секрет для приложения.

   1. В разделе **Секреты клиента** щелкните **New client secret** (Новый секрет клиента).
   1. Добавьте описание, чтобы отличать этот секрет от других секретов, которые могут вам понадобиться для создания этого приложения, включая `bot login`.
   1. Задайте для параметра **Срок действия истекает** значение **Никогда**.
   1. Нажмите кнопку **Добавить**.
   1. Перед закрытием этой страницы запишите секрет. Это значение вам нужно будет ввести позднее в поле _Секрет клиента_ при регистрации приложения AAD в боте.

1. В области навигации щелкните **Разрешения API**, чтобы открыть панель **Разрешения API**. Рекомендуется явно задать разрешения API для приложения.

   1. Щелкните **Добавить разрешение**, чтобы отобразить область **Запрос разрешений API**.
   1. Для этого примера выберите **API Microsoft** и **Microsoft Graph**.
   1. Выберите **Делегированные разрешения** и убедитесь, что выбраны все разрешения, требуемые для этого примера.

      > [!NOTE]
      > Для любого разрешения, отмеченного как **Требуется согласие администратора**, необходимо, чтобы пользователь и администратор клиента вошли в систему. Поэтому для бота лучше использовать поменьше разрешений такого типа.

      - **openid**
      - **profile**
      - **Mail.Read**
      - **Mail.Send**
      - **User.Read**
      - **User.ReadBasic.All**

   1. Щелкните **Добавить разрешения**. (При первом обращении пользователя к этому приложению через бота требуется предоставить согласие.)

Теперь приложение Azure AD настроено.

### <a name="register-your-azure-ad-application-with-your-bot"></a>Регистрация в боте приложения Azure AD

Следующим шагом является регистрация приложения Azure AD в созданном боте.

#### <a name="azure-ad-v1"></a>AAD версии 1

1. На [портале Azure](https://portal.azure.com/) перейдите к странице ресурса бота.
1. Щелкните **Параметры**.
1. В разделе **OAuth Connection Settings** (Параметры подключения OAuth), который находится в нижней части страницы, щелкните **Добавить настройку**.
1. Заполните форму следующим образом.

    1. В поле **Имя** введите имя подключения, которое будет затем использовано в коде бота.
    1. В разделе **Поставщик службы** выберите **Azure Active Directory**. После выбора этого параметра появятся отдельные поля Azure AD.
    1. В поле **Идентификатор клиента** введите идентификатор приложения (клиента), который вы записали для приложения Azure AD версии 1.
    1. В поле **Секрет клиента**, введите секрет, который вы создали для предоставления боту доступа к приложению Azure AD.
    1. В поле **Тип предоставления разрешения** введите `authorization_code`.
    1. В поле **URL-адрес входа** введите `https://login.microsoftonline.com`.
    1. В поле **Идентификатор клиента** введите **идентификатор каталога (клиент)** , записанный ранее для приложения AAD, или задайте значение **Общие** в зависимости от поддерживаемых типов учетных записей, выбранных при создании приложения ADD. Чтобы решить, какое значение следует задать, руководствуйтесь следующими критериями:

        - Если при создании приложения AAD вы выбрали параметр *Учетные записи только в этом каталоге организации* (только Майкрософт — один клиент) или *Учетные записи в любом каталоге организации* (каталог Microsoft AAD — несколько клиентов), введите **идентификатор клиента**, который вы записали ранее для приложения AAD.

        - Если же вы выбрали *Учетные записи в любом каталоге организации (любой каталог AAD — несколько клиентов) и личные учетные записи Майкрософт (например, Skype, Xbox, Outlook.com)* , введите слово **общие** вместо идентификатора клиента. В противном случае приложение AAD будет проверять клиент по выбранному для него идентификатору и исключать личные учетные записи Майкрософт.

       Данный клиент будет связан с пользователями, которые могут пройти проверку подлинности.

    1. В поле **URL-адрес ресурса** введите `https://graph.microsoft.com/`.
    1. Оставите поле **Области** пустым.

1. Выберите команду **Сохранить**.

> [!NOTE]
> Используя API Microsoft Graph, эти значения позволяют приложению получать доступ к данным Office 365.

#### <a name="azure-ad-v2"></a>AAD версии 2

1. Перейдите к странице регистрации каналов бота на [портале Azure](https://portal.azure.com/).
1. Щелкните **Параметры**.
1. В разделе **OAuth Connection Settings** (Параметры подключения OAuth), который находится в нижней части страницы, щелкните **Добавить настройку**.
1. Заполните форму следующим образом.

    1. В поле **Имя** введите имя подключения, которое было использовано в коде бота.
    1. В разделе **Поставщик службы** выберите **Azure Active Directory v2**. После выбора этого параметра появятся отдельные поля Azure AD.
    1. В поле **Идентификатор клиента** введите идентификатор приложения (клиента), который вы записали для приложения Azure AD версии 1.
    1. В поле **Секрет клиента**, введите секрет, который вы создали для предоставления боту доступа к приложению Azure AD.
    1. В поле **Идентификатор клиента** введите **идентификатор каталога (клиент)** , записанный ранее для приложения AAD, или задайте значение **Общие** в зависимости от поддерживаемых типов учетных записей, выбранных при создании приложения ADD. Чтобы решить, какое значение следует задать, руководствуйтесь следующими критериями:

        - Если при создании приложения AAD вы выбрали параметр *Учетные записи только в этом каталоге организации* (только Майкрософт — один клиент) или *Учетные записи в любом каталоге организации* (каталог Microsoft AAD — несколько клиентов), введите **идентификатор клиента**, который вы записали ранее для приложения AAD.

        - Если же вы выбрали *Учетные записи в любом каталоге организации (любой каталог AAD — несколько клиентов) и личные учетные записи Майкрософт (например, Skype, Xbox, Outlook.com)* , введите слово **общие** вместо идентификатора клиента. В противном случае приложение AAD будет проверять клиент по выбранному для него идентификатору и исключать личные учетные записи Майкрософт.

       Данный клиент будет связан с пользователями, которые могут пройти проверку подлинности.

    1. В поле **Области** введите имя разрешения, которое было выбрано при регистрации приложения `Mail.Read Mail.Send openid profile User.Read User.ReadBasic.All`.

        > [!NOTE]
        > Для приложения Azure AD версии 2 в поле **Области** можно вносить списки значений, которые являются чувствительными к регистру и разделенные пробелами.

1. Выберите команду **Сохранить**.

> [!NOTE]
> Используя API Microsoft Graph, эти значения позволяют приложению получать доступ к данным Office 365.

### <a name="test-your-connection"></a>Тестирование подключения

1. Щелкните запись созданного подключения, чтобы открыть его.
1. Щелкните **Проверка подключения** в верхней части области **Service Provider Connection Setting** (Параметры подключения поставщика службы).
1. При выполнении данного действия в первый раз должна открыться новая вкладка браузера с перечисленными разрешениями, которые запрашивает приложение, и предложение их принять.
1. Нажмите кнопку **Принимаю**.
1. Вы перейдете на страницу **Проверка подключения к \<имя_подключения> успешно выполнена**.

Для получения токенов пользователя можно использовать имя этого подключения в коде бота.

## <a name="prepare-the-bot-code"></a>Подготовка кода бота

Вам потребуются идентификатор приложения бота и пароль, чтобы выполнить этот процесс.

# <a name="c"></a>[C#](#tab/csharp)

<!-- TODO: Add guidance (once we have it) on how not to hard-code IDs and ABS auth. -->

1. Клонируйте нужный пример из репозитория GitHub: [**Аутентификация бота**][cs-auth-sample] или [**Аутентификация бота (MSGraph)** ][cs-msgraph-sample].
1. Обновите файл **appsettings.json**:

    - параметру `ConnectionName` присвойте значение имени подключения OAuth, которое вы добавили в бот;
    - параметрам `MicrosoftAppId` и `MicrosoftAppPassword` присвойте значения идентификатора приложения бота и секрета приложения.

      В зависимости от символов, из которых состоит секрет бота, может потребоваться XML-экранирование пароля. Например, символ амперсанда (&) потребуется кодировать как `&amp;`.

    [!code-json[appsettings](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/appsettings.json)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

1. Клонируйте нужный пример из репозитория GitHub: [**Аутентификация бота**][js-auth-sample] или [**Аутентификация бота (MSGraph)** ][js-msgraph-sample].
1. Обновите файл с расширением **.env**:

    - параметру `connectionName` присвойте значение имени подключения OAuth, которое вы добавили в бот;
    - параметрам `MicrosoftAppId` и `MicrosoftAppPassword` присвойте значения идентификатора приложения бота и секрета приложения.

      В зависимости от символов, из которых состоит секрет бота, может потребоваться XML-экранирование пароля. Например, символ амперсанда (&) потребуется кодировать как `&amp;`.

    [!code-txt[.env](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/.env)]

# <a name="python"></a>[Python](#tab/python)

1. Клонируйте пример [**с аутентификацией бота**][python-auth-sample] из репозитория GitHub.
1. Обновите **config.py**:

    - параметру `ConnectionName` присвойте значение имени подключения OAuth, которое вы добавили в бот;
    - параметрам `MicrosoftAppId` и `MicrosoftAppPassword` присвойте значения идентификатора приложения бота и секрета приложения.

      В зависимости от символов, из которых состоит секрет бота, может потребоваться XML-экранирование пароля. Например, символ амперсанда (&) потребуется кодировать как `&amp;`.

    [!code-python[config](~/../botbuilder-samples/samples/python/18.bot-authentication/config.py)]

---

Если вы не знаете, как получить **идентификатор приложения (Майкрософт)** и **пароль приложения (Майкрософт)** , [создайте новый пароль](../bot-service-quickstart-registration.md#get-registration-password).

> [!NOTE]
> Теперь код бота можно опубликовать в подписке Azure (щелкните проект правой кнопкой мыши и выберите **Опубликовать**), но для этой статьи это не требуется. Необходимо будет настроить конфигурацию публикации, которая использует план приложения и размещения, который использовался при настройке бота на портале Azure.

## <a name="test-the-bot-using-the-emulator"></a>Тестирование бота с помощью эмулятора

Установите [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme), если вы этого еще не сделали. Также см. статью об [отладке ботов с помощью эмулятора](../bot-service-debug-emulator.md).

<!-- auth config steps -->
Чтобы функция входа работала в примере бота, настройте эмулятор для аутентификации, как показано в [этом разделе](../bot-service-debug-emulator.md#configure-the-emulator-for-authentication).

### <a name="testing"></a>Тестирование

Когда вы настроите механизм аутентификации, можно протестировать пример бота.  

1. Запустите пример бота на локальном компьютере.
1. Запустите эмулятор.
1. При подключении к боту необходимо указать идентификатор приложения бота и пароль.
    - Идентификатор приложения и пароль можно получить в области регистрации приложений Azure. Это те же значения, которые вы назначили боту в файле `appsettings.json` или `.env`. В эмуляторе эти значения присваиваются в файле конфигурации или при первом подключении к боту.
    - Вы также можете экранировать пароль (XML) в коде бота.
1. Чтобы просмотреть список доступных команд для бота и проверить функции проверки подлинности, введите `help`.
1. После выполнения входа и до момента выхода не требуется повторно предоставлять учетные данные.
1. Чтобы выйти и отменить проверку подлинности, введите `logout`.

> [!NOTE]
> Для использования проверки подлинности бота требуется служба Bot Connector. Эта служба использует сведения о регистрации каналов бота.

## <a name="bot-authentication-example"></a>Пример аутентификация бота

В примере **Аутентификация бота** диалог получает маркер пользователя после входа пользователя в систему.

![Пример выходных данных](media/how-to-auth/auth-bot-test.png)

## <a name="bot-authentication-msgraph-example"></a>Пример MSGraph для аутентификации бота

В примере **Аутентификация бота MSGraph** диалог принимает ограниченный набор команд после входа пользователя в систему.

![Пример выходных данных](media/how-to-auth/msgraph-bot-test.png)

---

## <a name="additional-information"></a>Дополнительные сведения

Когда пользователь приказывает боту выполнить некоторые действия, которые требуют пользовательского входа, бот может с помощью `OAuthPrompt` инициировать извлечение токена для данного соединения. `OAuthPrompt` создает поток получения маркера, который выполняет следующие действия.

1. Проверяет, имеет ли служба Azure Bot маркер для текущего сочетания пользователя и подключения. Если маркер есть, он возвращается.
1. Если служба Azure Bot не имеет нужного маркера в кэше, создается `OAuthCard` с кнопкой входа, на которую может нажать пользователь.
1. Когда пользователь нажимает на кнопку входа `OAuthCard`, служба Azure Bot выполняет одно из двух действий: напрямую отправляет боту маркер пользователя или предоставляет пользователю шестизначный код аутентификации для ввода в окно чата.
1. Если пользователю предоставлен код аутентификации, бот позднее обменивает этот код на маркер пользователя.

В следующих разделах объясняется, как этот пример реализует некоторые распространенные задачи проверки подлинности.

### <a name="use-an-oauth-prompt-to-sign-the-user-in-and-get-a-token"></a>Использование запроса OAuth для входа пользователя и получения маркера

# <a name="c"></a>[C#](#tab/csharp)

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

[!code-csharp[Get the OAuthPrompt result](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/MainDialog.cs?range=54-56)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

![Архитектура бота](media/how-to-auth/architecture-js.png)

**dialogs/mainDialog.js**

Добавьте запрос OAuth в **MainDialog** с помощью конструктора. В этом примере значение имени подключения получено из файла с расширением **.env**.

[!code-javascript[Add OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=24-29)]

В шаге диалога укажите `beginDialog` для запуска командной строки OAuth, в которой пользователю будет предложено войти в систему.

- Если пользователь уже выполнил вход, будет создано событие ответа маркера без подтверждения пользователя.
- В противном случае пользователю будет предложено войти в систему. Служба Azure Bot отправляет событие ответа маркера после попытки входа пользователя.

[!code-javascript[Use OAuthPrompt](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=57)]

В следующем шаге диалога проверьте наличие маркера в результате, полученном от предыдущего шага. Если он не равен NULL, значит пользователь успешно выполнил вход.

[!code-javascript[Get OAuthPrompt result](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/mainDialog.js?range=62-63)]

# <a name="python"></a>[Python](#tab/python)

![Архитектура бота](media/how-to-auth/architecture-python.png)

**dialogs/main_dialog.py**

Добавьте запрос OAuth в **MainDialog** с помощью конструктора. В этом примере значение имени подключения получено из файла **config.py**.

[!code-python[Add OAuthPrompt](~/../botbuilder-samples/samples/python/18.bot-authentication/dialogs/main_dialog.py?range=34-44)]

В шаге диалога укажите `begin_dialog` для запуска командной строки OAuth, в которой пользователю будет предложено войти в систему.

- Если пользователь уже выполнил вход, будет создано событие ответа маркера без подтверждения пользователя.
- В противном случае пользователю будет предложено войти в систему. Служба Azure Bot отправляет событие ответа маркера после попытки входа пользователя.

[!code-python[Add OAuthPrompt](~/../botbuilder-samples/samples/python/18.bot-authentication/dialogs/main_dialog.py?range=49)]

В следующем шаге диалога проверьте наличие маркера в результате, полученном от предыдущего шага. Если он не равен NULL, значит пользователь успешно выполнил вход.

[!code-python[Add OAuthPrompt](~/../botbuilder-samples/samples/python/18.bot-authentication/dialogs/main_dialog.py?range=54-65)]

---

### <a name="wait-for-a-tokenresponseevent"></a>Ожидание метода TokenResponseEvent

Когда вы запускаете командную строку OAuth, она ожидает события получения маркера, из которого затем извлекает маркер пользователя.

# <a name="c"></a>[C#](#tab/csharp)

**Bots\AuthBot.cs**

**AuthBot** наследуется от `ActivityHandler` и явным образом обрабатывает действия ответа маркера. Здесь мы продолжаем активный диалог, что позволяет командной строке OAuth обработать событие и получить маркер.

[!code-csharp[OnTokenResponseEventAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Bots/AuthBot.cs?range=32-38)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/authBot.js**

**AuthBot** наследуется от `ActivityHandler` и явным образом обрабатывает действия ответа маркера. Здесь мы продолжаем активный диалог, что позволяет командной строке OAuth обработать событие и получить маркер.

[!code-javascript[onTokenResponseEvent](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/bots/authBot.js?range=29-31)]

# <a name="python"></a>[Python](#tab/python)

**bots/auth_bot.py**

**AuthBot** явным образом обрабатывает действия ответа маркера. Здесь мы продолжаем активный диалог, что позволяет командной строке OAuth обработать событие и получить маркер.

[!code-python[on_token_response_event](~/../botbuilder-samples/samples/python/18.bot-authentication/bots/auth_bot.py?range=38-44)]

---

### <a name="log-the-user-out"></a>Выход пользователя

Мы рекомендуем предоставить пользователям возможность явно выходить из системы, не полагаясь на автоматический разрыв подключения при превышении времени ожидания.

# <a name="c"></a>[C#](#tab/csharp)

**Dialogs\LogoutDialog.cs** [!code-csharp[Allow logout](~/../botbuilder-samples/samples/csharp_dotnetcore/18.bot-authentication/Dialogs/LogoutDialog.cs?range=44-61&highlight=11)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**dialogs/logoutDialog.js** [!code-javascript[Allow logout](~/../botbuilder-samples/samples/javascript_nodejs/18.bot-authentication/dialogs/logoutDialog.js?range=31-42&highlight=7)]

# <a name="python"></a>[Python](#tab/python)

**dialogs/logout_dialog.py** [!code-python[allow logout](~/../botbuilder-samples/samples/python/18.bot-authentication/dialogs/logout_dialog.py?range=27-34&highlight=6)]

---

### <a name="adding-teams-authentication"></a>Добавление аутентификации Teams

В контексте OAuth Teams работает несколько иначе, чем другие каналы. Чтобы правильно реализовать аутентификацию, нужно внести несколько изменений. Мы добавим код из примера бота аутентификации Teams ([C#][cs-teams-auth-sample]/[JavaScript][js-teams-auth-sample]).

Одно из различий между другими каналами и Teams заключается в том, что Teams отправляет в бот действие *invoke*, а не действие *event*.

# <a name="c"></a>[C#](#tab/csharp)

**Bots/TeamsBot.cs**  
[!code-csharp[Invoke Activity](~/../botbuilder-samples/samples/csharp_dotnetcore/46.teams-auth/Bots/TeamsBot.cs?range=34-42&highlight=1)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/teamsBot.js**  
[!code-javascript[Invoke Activity](~/../botbuilder-samples/samples/javascript_nodejs/46.teams-auth/bots/teamsBot.js?range=16-25&highlight=1)]

# <a name="python"></a>[Python](#tab/python)

Сейчас способ интеграции аутентификации с ботом в Microsoft Teams немного отличается. См. соответствующую документацию по [Teams](https://aka.ms/teams-docs).

---

При использовании *запроса OAuth* это действие invoke должно быть переадресовано в диалог. Это будет сделано в `TeamsActivityHandler`. Добавьте в файл основного диалога следующий код.

# <a name="c"></a>[C#](#tab/csharp)

**Bots/DialogBot.cs**  
[!code-csharp[Dialogs Handler](~/../botbuilder-samples/samples/csharp_dotnetcore/46.teams-auth/Bots/DialogBot.cs?range=19)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**Bots/dialogBot.js**  
[!code-javascript[Dialogs Handler](~/../botbuilder-samples/samples/javascript_nodejs/46.teams-auth/bots/dialogBot.js?range=6)]

# <a name="python"></a>[Python](#tab/python)

Сейчас способ интеграции аутентификации с ботом в Microsoft Teams немного отличается. См. соответствующую документацию по [Teams](https://aka.ms/teams-docs).

---

Наконец, добавьте соответствующий файл `TeamsActivityHandler` (`TeamsActivityHandler.cs` для ботов C# `teamsActivityHandler.js` для ботов JavaScript) в папку бота самого верхнего уровня.

`TeamsActivityHandler` также отправляет действия *message reaction*. Действие message reaction ссылается на исходное действие с помощью поля *reply to ID*. Это действие также должно отображаться в [веб-канале активности ][teams-activity-feed] в Microsoft Teams.

> [!NOTE]
> Вам нужно создать манифест и включить `token.botframework.com` в раздел `validDomains`. В противном случае при нажатии кнопки **входа** в OAuthCard окно проверки подлинности не будет открываться. Создайте манифест с помощью [App Studio](https://docs.microsoft.com/microsoftteams/platform/get-started/get-started-app-studio).

### <a name="further-reading"></a>Дополнительные материалы

- [Дополнительные ресурсы Bot Framework](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help) содержат ряд ссылок с сопроводительной информацией.
- Репозиторий с [пакетом SDK Bot Framework ](https://github.com/microsoft/botbuilder) содержит дополнительные сведения о репозиториях, примерах, средствах и спецификациях для пакета SDK Bot Builder.

<!-- Footnote-style links -->

[Azure portal]: https://ms.portal.azure.com
[azure-aad-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview
[aad-registration-blade]: https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredAppsPreview

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-auth-sample]: https://aka.ms/v4cs-bot-auth-sample
[js-auth-sample]: https://aka.ms/v4js-bot-auth-sample
[python-auth-sample]: https://aka.ms/bot-auth-python-sample-code

[cs-msgraph-sample]: https://aka.ms/v4cs-auth-msgraph-sample
[js-msgraph-sample]: https://aka.ms/v4js-auth-msgraph-sample
[cs-teams-auth-sample]:https://aka.ms/cs-teams-auth-sample
[js-teams-auth-sample]:https://aka.ms/js-teams-auth-sample
[teams-activity-feed]:[https://aka.ms/teams-activity-feed

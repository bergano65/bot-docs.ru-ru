---
title: Добавление проверки подлинности к боту с помощью службы Azure Bot | Документация Майкрософт
description: Сведения об использовании функции проверки подлинности службы Azure Bot для добавления боту функции единого входа.
author: JonathanFingold
ms.author: JonathanFingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 09/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 93d32d5d0ac35dead8e9f1c48b526058449fabad
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998791"
---
# <a name="add-authentication-to-your-bot-via-azure-bot-service"></a>Добавление проверки подлинности к боту с помощью службы Azure Bot

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

В этом руководстве используются функции бота проверки подлинности службы Azure Bot. Они упрощают разработку бота, который выполняет проверку подлинности пользователей в различных поставщиках удостоверений, например Azure AD (Azure Active Directory), GitHub, Uber и т. д. Также эти обновления помогают улучшить взаимодействие с пользователем путем устранения _проверки шифра_ для некоторых клиентов.

До этого бот должен включать контроллеры OAuth и ссылки для входа, хранить целевые идентификаторы и секреты клиента и выполнять управление токенами пользователей.
<!--
These capabilities were bundled in the BotAuth and AuthBot samples that are on GitHub.
-->

Теперь разработчикам ботов больше не нужно размещать контроллеры OAuth или управлять жизненным циклом токена, так как все это может выполнять служба Azure Bot.

Доступны следующие функции:

- Усовершенствования каналов поддержки новых функций проверки подлинности, таких как новые библиотеки WebChat и DirectLineJS, позволяют устранить необходимость в шестизначной проверке шифра.
- Усовершенствования портала Azure по добавлению, удалению и настройке параметров подключения к различным поставщикам удостоверений OAuth.
- Поддержка множества встроенных поставщиков удостоверений, включая Azure AD (конечные точки версий 1 и 2), GitHub и другие.
- Обновления пакетов SDK Bot Builder для C# и Node.js, которые дают возможность извлекать токены, создавать OAuthCards и обрабатывать события TokenResponse.
- Примеры создания бота с аутентификацией в Azure AD.

Чтобы добавить к существующему боту следующие функции, можно экстраполировать шаги, описанные в этой статье. Далее приведены примеры ботов, которые демонстрируют новые функции проверки подлинности.

| Образец | Версия Bot Builder | ОПИСАНИЕ |
|:---|:---:|:---|
| [C# Auth](http://aka.ms/v4csharpauth) | версия 4 | Реализована поддержка OAuthCard в пакете SDK версии 4 для C# |
| [C# Auth Graph](http://aka.ms/v4csharpauthgraph) | версия 4 |  Реализована поддержка OAuthCard в пакете SDK версии 4 для C# с использованием AAD и Microsoft Graph API |
| [Node Auth](http://aka.ms/v4cnodeauth) | версия 4 |  Реализована поддержка OAuthCard в пакете SDK версии 4 для Node/JavaScript |

> [!NOTE]
> Функции аутентификации также работают с Bot Builder версии 3. Но в этой статье мы рассматриваем только пример кода для версии 4.

Дополнительные сведения и поддержку можно найти в статье [Bot Framework additional resources](https://docs.microsoft.com/azure/bot-service/bot-service-resources-links-help) (Дополнительные ресурсы Bot Framework).

## <a name="overview"></a>Обзор

В этом руководстве создается пример бота, который подключается к Microsoft Graph, с помощью токена Azure AD версии 1 или 2. Для этого процесса используется код из репозитория GitHub. В этом руководстве есть инструкции по его настройке, включая настройку приложения бота.

- **Создание бота и приложения проверки подлинности**
- **Подготовка примера кода бота**
- **Подключение эмулятора Bot Framework для проверки бота**

Для выполнения этих действий потребуются установленные Visual Studio 2017, Npm, Node.js и Git. Также необходимо обладать некоторым опытом работы с Azure, OAuth 2.0 и разработки ботов.

После завершения появится бот, который может выполнить несколько простых задач в приложении Azure AD, например он может проверять и отправлять электронную почту или отображать пользователя и менеджера. Чтобы это сделать, бот будет использовать токен приложения Azure AD библиотеки Microsoft.Graph.

В заключительной части статьи рассматриваются некоторые примеры кодов бота.

- **Примечания о потоке извлечения токена**

## <a name="create-your-bot-and-an-authentication-application"></a>Создание бота и приложения проверки подлинности

Потребуется создать бот регистрации, которому будет отправляться код бота, а также создать приложение Azure AD (версии 1 или 2), чтобы позволить боту получить доступ к Office 365.

> [!NOTE]
> Данные функции проверки подлинности работают с другими типами ботов. Тем не менее в этом руководстве используется только бот регистрации.

### <a name="register-an-application-in-azure-ad"></a>Регистрация приложение в Azure AD

Вам потребуется приложение Azure AD, которое может использовать бот при подключении к API Microsoft Graph, защищенный Azure AD ресурс и т. д.

Для данного бота можно использовать конечные точки Azure AD версии 1 или 2.
Дополнительные сведения о разнице между конечными точками версии 1 и 2 см. в статьях [What's different about the v2.0 endpoint?](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-compare) (Что нового в конечной точке версии 2.0 ) и [Настройка входа пользователей с помощью учетной записи Майкрософт и Azure Active Directory в одном приложении](https://docs.microsoft.com/azure/active-directory/develop/active-directory-appmodel-v2-overview).

#### <a name="to-create-an-azure-ad-v1-application"></a>Создание приложения Azure AD версии 1

1. Перейдите к [Azure AD на портале Azure](https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Overview).
1. Щелкните **Регистрация приложений**.
1. На панели **Регистрация приложений** щелкните **Регистрация нового приложения**.
1. Заполните обязательные поля и создайте регистрацию приложения.
   1. Присвойте имя приложению.
   1. В качестве **типа приложения** установите **Веб-приложение или API**.
   1. В качестве значения **URL-адрес входа** установите `https://token.botframework.com/.auth/web/redirect`.
   1. Нажмите кнопку **Создать**.
      - Когда приложение будет создано, оно появится в области **Зарегистрированное приложение**.
      - Запишите значение **идентификатора приложения**. Это значение понадобится позже в качестве _идентификатора клиента_.
1. Чтобы настроить приложение, щелкните **Параметры**.
1. Чтобы открыть панель **Ключи**, щелкните **Ключи**.
   1. В разделе **Пароли** создайте ключ `BotLogin`.
   1. Установите его **Продолжительность** на **Never expires** (Нелимитированный срок действия).
   1. Нажмите кнопку **Сохранить** и запишите значение ключа. Это значение понадобится позже в качестве _секрета приложения_.
   1. Закройте панель **Ключи**.
1. Чтобы открыть панель **Требуемые разрешения**, щелкните **Требуемые разрешения**.
   1. Щелкните **Добавить**.
   1. Щелкните **Выбор API**, выберите **Microsoft Graph**, а затем щелкните **Выбор**.
   1. Щелкните **Выбор разрешений**. Выберите разрешения, которые будут использоваться в приложении.

      > [!NOTE]
      > Для любого разрешения, отмеченного как **Требуются права администратора**, требуется, чтобы пользователь и администратор клиента вошли в систему. Поэтому для бота требуется использовать поменьше разрешений такого типа.

      Выберите следующие делегированные разрешения Microsoft Graph.
      - Чтение базовых профилей всех пользователей.
      - Чтение почты пользователей
      - Вход в систему и чтение профиля пользователя.
      - отправка сообщений в качестве пользователя.
      - Просмотр базовых профилей всех пользователей.
      - Просмотр пользовательских адресов электронной почты.

   1. Щелкните **Выбрать**, а затем нажмите **Готово**.
   1. Закройте панель **Требуемые разрешения**.

Теперь приложение Azure AD версии 1 настроено.

#### <a name="to-create-an-azure-ad-v2-application"></a>Для создания приложения Azure AD версии 2 выполните следующие действия.

1. Перейдите на [портал регистрации приложений Майкрософт](https://apps.dev.microsoft.com).
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
    - В рамках этого руководства добавьте разрешения **Mail.Read**, **Mail.Send**, **openid**, **profile**, **User.Read** и **User.ReadBasic.All**.
      В области настройки подключения должно находиться как разрешение **openid**, так и ресурс Azure AD Graph, например **Mail.Read**.
    - Запишите выбранное разрешение. Это разрешение понадобится позже в качестве области для настройки подключения.

1. В нижней части страницы нажмите кнопку **Сохранить** .

### <a name="create-your-bot-on-azure"></a>Создание бота в Azure

Создайте **Bot Channels Registration** (Бот регистрации каналов) с помощью [портала Azure](https://portal.azure.com/).

### <a name="register-your-azure-ad-application-with-your-bot"></a>Регистрация в боте приложения Azure AD

Следующим шагом является регистрация приложения Azure AD в созданном боте.

#### <a name="to-register-an-azure-ad-v1-application"></a>Регистрация приложения Azure AD версии 1

1. На [портале Azure](http://portal.azure.com/) перейдите к странице ресурса бота.
1. Щелкните **Параметры**.
1. В разделе **OAuth Connection Settings** (Параметры подключения OAuth), который находится в нижней части страницы, щелкните **Добавить настройку**.
1. Заполните форму следующим образом.
    1. В поле **Имя** введите имя подключения, которое было использовано в коде бота.
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

Для получения токенов пользователя можно использовать имя этого подключения в коде бота.

#### <a name="to-register-an-azure-ad-v2-application"></a>Регистрация приложения Azure AD версии 2

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

Для получения токенов пользователя можно использовать имя этого подключения в коде бота.

#### <a name="to-test-your-connection"></a>Тестирование подключения

1. Откройте созданное подключение.
1. Щелкните **Проверка подключения** в верхней части области **Service Provider Connection Setting** (Параметры подключения поставщика службы).
1. При выполнении данного действия в первый раз должна открыться новая вкладка браузера с перечисленными разрешениями, которые запрашивает приложение, и предложение их принять.
1. Нажмите кнопку **Принимаю**.
1. Данное действие перенаправляет пользователя на страницу **Проверка подключения к <your-connection-name> успешно выполнена** .

## <a name="prepare-the-bot-sample-code"></a>Подготовка примера кода бота

В зависимости от выбранного примера вы будете работать с C# или Node.

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

    ```xml
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

    Если вы не знаете, как получить значения **Идентификатор приложения Майкрософт** и **Пароль приложения Майкрософт**, посмотрите параметр **ApplicationSettings** службы приложений Azure, который был подготовлен для бота на портале Azure.

    > [!NOTE]
    > Теперь код бота можно опубликовать в подписке Azure (щелкните проект правой кнопкой мыши и выберите **Опубликовать**), но данное действие не является обязательным для текущего руководства. Необходимо будет настроить конфигурацию публикации, которая использует план приложения и размещения, который использовался при настройке бота на портале Azure.

## <a name="use-the-emulator-to-test-your-bot"></a>Подключение эмулятора Bot Framework для проверки бота

Для локальной проверки бота требуется установить [Bot Emulator](https://github.com/Microsoft/BotFramework-Emulator). Можно использовать Bot Framework Emulator версии 3 или 4.

1. Запустите бот (с отладкой или без).
1. Запишите номер порта localhost страницы. Данная информация потребуется для взаимодействия с ботом.
1. Запустите эмулятор Bot Framework.
1. Подключитесь к боту.

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

Как видно из этого кода, сначала бот выполняет быструю проверку, чтобы определить, имеет ли служба Azure Bot токен пользователя (который идентифицирован текущим отправителем действия) и полученное ConnectionName (которое является именем подключения, используемое в конфигурации). К этому моменту служба Azure Bot уже будет обладать кэшированным токеном (или не будет). Эту быструю проверку можно выполнить с помощью вызова метода GetUserTokenAsync. Если служба Azure Bot имеет токен и возвращает его, он может быть использован мгновенно. Если служба Azure Bot не имеет токена, результатом выполнения этого метода будет NULL. В таком случае бот может отправить пользователю для входа настраиваемый OAuthCard.

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

## <a name="additional-resources"></a>Дополнительные ресурсы
[botbuilder-core package](https://github.com/microsoft/botbuilder)
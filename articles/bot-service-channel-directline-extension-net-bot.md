---
title: Бот .NET с расширением Службы приложений Direct Line
titleSuffix: Bot Service
description: Настройка работы с расширением Службы приложений Direct Line в боте .NET
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 01/16/2020
ms.openlocfilehash: 7f46d5f2b5012db914568e5ae613c18405dfa113
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117637"
---
# <a name="configure-net-bot-for-extension"></a>Настройка бота .NET для использования расширения

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

В этой статье описывается, как включить в бота поддержку работы с **именованными каналами** и включить расширение службы Direct Line в ресурсе **Службы приложений Azure**, в котором размещен этот бот. См. также о [создании клиента .NET для подключения к расширению Службы приложений Direct Line](bot-service-channel-directline-extension-net-client.md).


## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить описанные далее действия, вам потребуется ресурс **Службы приложений Azure** и связанная с ним **Служба приложений** в Azure.

## <a name="enable-direct-line-app-service-extension"></a>Включение расширения Службы приложений Direct Line

В этом разделе описывается, как включить расширение службы Direct Line с использованием ключей из конфигурации канала бота и ресурса **Службы приложений Azure**, в котором размещен ваш бот.

## <a name="update-net-bot-to-use-direct-line-app-service-extension"></a>Настройка бота .NET для использования расширения Службы приложений Direct Line

> [!NOTE]
> Пакеты `Microsoft.Bot.Builder.StreamingExtensions` находятся в предварительной версии и не будут обновляться. Пакет SDK версии 4.7 уже содержит [код потоковой передачи](https://github.com/microsoft/botbuilder-dotnet/tree/master/libraries/Microsoft.Bot.Builder/Streaming), и вам не нужно отдельно устанавливать пакеты потоковой передачи.

1. Откройте проект бота в Visual Studio.
2. Добавьте в проект пакет NuGet **расширения потоковой передачи**.
    1. В проекте щелкните правой кнопкой мыши элемент **Зависимости** и выберите **Управление пакетами NuGet**.
    2. На вкладке *Обзор* щелкните **Включить предварительные выпуски**, чтобы просмотреть пакеты предварительных версий.
    3. Выберите пакет **Microsoft.Bot.Builder.StreamingExtensions**.
    4. Нажмите кнопку **Установить**, чтобы установить пакет, прочитайте и примите условия лицензионного соглашения.
3. Разрешите приложению использовать **NamedPipe из Bot Framework**.
    - Откройте файл `Startup.cs` .
    - В методе ``Configure`` добавьте код в ``UseBotFrameworkNamedPipe``.

    ```csharp

    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }
        else
        {
            app.UseHsts();
        }

        app.UseDefaultFiles();
        app.UseStaticFiles();

        // Allow the bot to use named pipes.
        app.UseNamedPipes();

        app.UseMvc();
    }
    ```

4. Сохраните файл `Startup.cs`.
5. Откройте файл `appsettings.json` и укажите следующие значения.
    1. `"MicrosoftAppId": "<secret Id>"`
    2. `"MicrosoftAppPassword": "<secret password>"`

    Это значения **appid** и **appSecret**, связанные с группой регистрации службы.

6. **Опубликуйте** бота в Службе приложений Azure.

### <a name="gather-your-direct-line-extension-keys"></a>Сбор ключей расширения Direct Line

1. В браузере перейдите на [портал Azure](https://portal.azure.com/).
1. На портале Azure найдите ресурс **службы Azure Bot**.
1. Щелкните элемент **Каналы**, чтобы настроить каналы для бота.
1. Если канал **Direct Line** еще не включен, щелкните его, чтобы включить.
1. Если он уже включен, в таблице "Подключение к каналам" щелкните ссылку **Изменить** в строке Direct Line.
1. Прокрутите вниз до раздела "Ключи расширения Службы приложений".
1. Щелкните **Показать ссылку**, чтобы отобразить один из ключей, а затем скопируйте и сохраните его значение. Это значение потребуется в следующем разделе.

![Ключи расширения Службы приложений](./media/channels/direct-line-extension-extension-keys.png)

### <a name="enable-the-direct-line-app-service-extension"></a>Включение расширения Службы приложений Direct Line

1. В браузере перейдите на [портал Azure](https://portal.azure.com/).
1. На портале Azure найдите страницу ресурсов **Службы приложений Azure** для веб-приложения, в котором размещен или будет размещен ваш бот.
1. Щелкните элемент **Конфигурация**. Добавьте следующие параметры приложения в разделе *Параметры приложения*.

    |Имя|Значение|
    |---|---|
    |DirectLineExtensionKey|<App_Service_Extension_Key>|
    |DIRECTLINE_EXTENSION_VERSION|последняя|

    Где *App_Service_Extension_Key* — это сохраненное ранее значение.

1. В разделе *Конфигурация* щелкните раздел **Общие параметры** и включите **Websocket**.
1. Нажмите кнопку **Сохранить**, чтобы сохранить параметры. Эта команда перезапускает Службу приложений Azure.

## <a name="confirm-direct-line-app-extension-and-the-bot-are-initialized"></a>Подтвердите, что расширение Direct Line Службы приложений и бот инициализированы.

В браузере откройте страницу https://<ваша_служба_приложений>.azurewebsites.net/.bot.
Если все настроено правильно, вы увидите такое содержимое JSON: `{"k":true,"ib":true,"ob":true,"initialized":true}`. В этих данных, которые указывают на то, что **все работает правильно**:

- **k** определяет, может ли расширение Direct Line Службы приложений (ASE) считывать соответствующий ключ расширения из своей конфигурации.
- **initialized** определяет, может ли ASE Direct Line с помощью соответствующего ключа расширения скачивать метаданные бота из службы Azure Bot.
- **ib** определяет, может ли ASE Direct Line устанавливать входящее соединение с ботом.
- **ob** определяет, может ли ASE Direct Line устанавливать исходящее соединение с ботом.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Создание клиента .NET](./bot-service-channel-directline-extension-net-client.md)

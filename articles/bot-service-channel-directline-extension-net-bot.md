---
title: Бот .NET с расширением Службы приложений Direct Line
titleSuffix: Bot Service
description: Настройка работы с расширением Службы приложений Direct Line в боте .NET
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: 6b6e841e49ab800b239961cf5ae54dc2d5feab23
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757776"
---
## <a name="net-bot-with-direct-line-app-service-extension"></a>Бот .NET с расширением Службы приложений Direct Line

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

В этой статье описывается, как включить в бота поддержку работы с **именованными каналами** и включить расширение службы Direct Line в ресурсе **Службы приложений Azure**, в котором размещен этот бот.  

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить описанные далее действия, вам потребуется ресурс **Службы приложений Azure** и связанная с ним **Служба приложений** в Azure.

## <a name="update-net-bot-to-use-direct-line-app-service-extension"></a>Настройка бота .NET для использования расширения Службы приложений Direct Line

1. Откройте проект бота в Visual Studio.
1. Добавьте в проект пакет NuGet **расширения потоковой передачи**.
    1. В проекте щелкните правой кнопкой мыши элемент **Зависимости** и выберите **Управление пакетами NuGet**.
    1. На вкладке *Обзор* щелкните **Включить предварительные выпуски**, чтобы просмотреть пакеты предварительных версий.
    1. Выберите пакет **Microsoft.Bot.Builder.StreamingExtensions**.
    1. Нажмите кнопку **Установить**, чтобы установить пакет, прочитайте и примите условия лицензионного соглашения.
1. Разрешите приложению использовать **NamedPipe из Bot Framework**.
    - Откройте файл `Startup.cs` .
    - В методе ``Configure`` добавьте код в ``UseBotFrameworkNamedPipe``.

    ```csharp

    using Microsoft.Bot.Builder.StreamingExtensions;

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

        // Allow bot to use named pipes.
        app.UseBotFrameworkNamedPipe();

        app.UseMvc();
    }
    ```

1. Сохраните файл `Startup.cs`.
1. Откройте файл `appsettings.json` и укажите следующие значения.
    1. `"MicrosoftAppId": "<secret Id>"`
    1. `"MicrosoftAppPassword": "<secret password>"`

    Это значения **appid** и **appSecret**, связанные с группой регистрации службы.

1. **Опубликуйте** бота в Службе приложений Azure.
1. В браузере откройте страницу https://<ваша_служба_приложений>.azurewebsites.net/.bot. Если все настроено правильно, вы увидите такое содержимое JSON: `{"k":true,"ib":true,"ob":true,"initialized":true}`.

## <a name="enable-direct-line-app-service-extension"></a>Включение расширения Службы приложений Direct Line

В этом разделе описывается, как включить расширение службы Direct Line с использованием ключей из конфигурации канала бота и ресурса **Службы приложений Azure**, в котором размещен ваш бот.

### <a name="gather-your-direct-line-extension-keys"></a>Сбор ключей расширения Direct Line

1. В браузере перейдите на [портал Azure](https://portal.azure.com/).
1. На портале Azure найдите ресурс **службы Azure Bot**.
1. Щелкните элемент **Каналы**, чтобы настроить каналы для бота.
1. Если канал **Direct Line** еще не включен, щелкните его, чтобы включить. 
1. Если он уже включен, в таблице "Подключение к каналам" щелкните ссылку **Изменить** в строке Direct Line.
1. Прокрутите вниз до раздела "Ключи расширения Службы приложений". 
1. Щелкните **Показать ссылку**, чтобы отобразить один из ключей, и скопируйте его значение.

![Ключи расширения Службы приложений](./media/channels/direct-line-extension-extension-keys.png)

### <a name="enable-the-direct-line-app-service-extension"></a>Включение расширения Службы приложений Direct Line

1. В браузере перейдите на [портал Azure](https://portal.azure.com/).
1. На портале Azure найдите страницу ресурсов **Службы приложений Azure** для веб-приложения, в котором размещен или будет размещен ваш бот.
1. Щелкните элемент **Конфигурация**. Добавьте следующие параметры приложения в разделе *Параметры приложения*.

    |ИМЯ|Значение|
    |---|---|
    |DirectLineExtensionKey|<Ключ_расширения_Службы_приложений_из_раздела_1>|
    |DIRECTLINE_EXTENSION_VERSION|последняя|

1. В разделе *Конфигурация* щелкните раздел **Общие параметры** и включите **Websocket**.
1. Нажмите кнопку **Сохранить**, чтобы сохранить параметры. Эта команда перезапускает Службу приложений Azure.
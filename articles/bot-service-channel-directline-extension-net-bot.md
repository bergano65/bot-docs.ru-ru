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
ms.openlocfilehash: d49ec4b742d644371458cc732fe60c605878ff27
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791721"
---
# <a name="configure-net-bot-for-extension"></a>Настройка бота .NET для использования расширения

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

В этой статье описывается, как включить в бота поддержку работы с **именованными каналами** и включить расширение службы Direct Line в ресурсе **Службы приложений Azure**, в котором размещен этот бот.  

## <a name="prerequisites"></a>предварительные требования

Чтобы выполнить описанные далее действия, вам потребуется ресурс **Службы приложений Azure** и связанная с ним **Служба приложений** в Azure.

## <a name="enable-direct-line-app-service-extension"></a>Включение расширения Службы приложений Direct Line

В этом разделе описывается, как включить расширение службы Direct Line с использованием ключей из конфигурации канала бота и ресурса **Службы приложений Azure**, в котором размещен ваш бот.

## <a name="update-net-bot-to-use-direct-line-app-service-extension"></a>Настройка бота .NET для использования расширения Службы приложений Direct Line

>Примечание. Пакеты `Microsoft.Bot.Builder.StreamingExtensions` находятся в режиме предварительной версии и не будут обновляться. Пакет SDK версии 4.7 уже содержит [код потоковой передачи](https://github.com/microsoft/botbuilder-dotnet/tree/master/libraries/Microsoft.Bot.Builder/Streaming), и вам не нужно отдельно устанавливать пакеты потоковой передачи. Если вы выполнили обновление до пакета SDK версии 4.7, ознакомьтесь с разделом [дополнительных сведений](bot-service-channel-directline-extension-net-bot.md#additional-information), где описаны изменения, которые нужно внести в этот раздел для включения этой функции. 

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

4. Сохраните файл `Startup.cs`.
5. Откройте файл `appsettings.json` и укажите следующие значения.
    1. `"MicrosoftAppId": "<secret Id>"`
    2. `"MicrosoftAppPassword": "<secret password>"`

    Это значения **appid** и **appSecret**, связанные с группой регистрации службы.

6. **Опубликуйте** бота в Службе приложений Azure.
7. В браузере откройте страницу https://<ваша_служба_приложений>.azurewebsites.net/.bot. Если все настроено правильно, вы увидите такое содержимое JSON: `{"k":true,"ib":true,"ob":true,"initialized":true}`. В этих данных, которые указывают на то, что **все работает правильно**:

    - **k** определяет, может ли расширение Службы приложений (ASE) Direct Line считывать ключ расширения из своей конфигурации. 
    - **initialized** определяет, может ли ASE Direct Line с помощью ключа расширения скачивать метаданные бота из службы Azure Bot.
    - **ib** определяет, может ли ASE Direct Line устанавливать входящее соединение с ботом.
    - **ob** определяет, может ли ASE Direct Line устанавливать исходящее соединение с ботом. 


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

    |Имя|Значение|
    |---|---|
    |DirectLineExtensionKey|<Ключ_расширения_Службы_приложений_из_раздела_1>|
    |DIRECTLINE_EXTENSION_VERSION|последняя|

1. В разделе *Конфигурация* щелкните раздел **Общие параметры** и включите **Websocket**.
1. Нажмите кнопку **Сохранить**, чтобы сохранить параметры. Эта команда перезапускает Службу приложений Azure.

## <a name="additional-information"></a>Дополнительные сведения 

Если вы выполнили обновление до пакета SDK версии 4.7, немного измените инструкции в разделе "Настройка бота .NET для использования расширения Службы приложений Direct Line", как показано ниже. 
- Пропустите **шаг 2**, так как вам не нужны пакеты в предварительной версии. 
- На **шаге 3** выполните следующие действия.  

Разрешите приложению **использовать именованные каналы (UseNamedPipes)** :
- Откройте файл `Startup.cs` .
- В методе ``Configure`` добавьте код в ``UseNamedPipes``.

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
        app.UseNamedPiped();

        app.UseMvc();
    }

    ```


---
title: Использование бота Node.js с расширением Direct Line Службы приложений
titleSuffix: Bot Service
description: Настройка бота Node.js для работы с расширением Direct Line Службы приложений
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: dev
ms.date: 01/15/2020
ms.openlocfilehash: 025b3db49f82643d0942518ff87810714389b54c
ms.sourcegitcommit: df2b8d4e29ebfbb9e8a10091bb580389fe4c34cc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2020
ms.locfileid: "76256445"
---
# <a name="configure-nodejs-bot-for-extension"></a>Настройка бота Node.js для работы с расширением

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

В этой статье описывается, как включить в бота поддержку работы с **именованными каналами** и включить расширение службы Direct Line в ресурсе **Службы приложений Azure**, в котором размещен этот бот.  

## <a name="prerequisites"></a>Предварительные требования

Чтобы выполнить описанные далее действия, вам потребуется ресурс **Службы приложений Azure** и связанная с ним **Служба приложений** в Azure.

## <a name="enable-direct-line-app-service-extension"></a>Включение расширения Службы приложений Direct Line

В этом разделе описывается, как включить расширение службы Direct Line с использованием ключей из конфигурации канала бота и ресурса **Службы приложений Azure**, в котором размещен ваш бот.

## <a name="update-nodejs-bot-to-use-direct-line-app-service-extension"></a>Обновление бота Node.js для работы с расширением Direct Line Службы приложений

1. Для работы BotBuilder версии 4.7.0 и выше требуется использовать бот Node.js с расширениями Direct Line Службы приложений.
1. Обновите существующего бота с использованием пакета SDK 4.х.
    1. В корневом каталоге бота выполните `npm install --save botbuilder`, чтобы обновить пакеты до последних версий.
1. Разрешите приложению использовать **именованный канал расширения Direct Line Службы приложений**.
    - Обновите файл index.js бота (под назначением адаптера и бота), чтобы включить следующее:
    
    ```Node.js
    
    adapter.useNamedPipe(async (context) => {
        await myBot.run(context);
    });
    ```

1. Сохраните файл `index.js`.
1. Обновите файл `Web.Config` по умолчанию, чтобы добавить обработчик `AspNetCore`, требуемый для обслуживания запросов расширением Direct Line Службы приложений:
    - Укажите файл `Web.Config` в каталоге `wwwroot` бота и замените содержимое по умолчанию следующим:
    
    ```XML
    
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
      <system.webServer>
        <handlers>      
          <add name="aspNetCore" path="*/.bot/*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
          <add name="iisnode" path="*" verb="*" modules="iisnode" />
        </handlers>
       </system.webServer>
    </configuration>
    ```
    
1. Откройте файл `appsettings.json` и укажите следующие значения.
    1. `"MicrosoftAppId": "<secret Id>"`
    1. `"MicrosoftAppPassword": "<secret password>"`

    Это значения **appid** и **appSecret**, связанные с группой регистрации службы.

1. **Опубликуйте** бота в Службе приложений Azure.

### <a name="gather-your-direct-line-app-service-extension-keys"></a>Сбор ключей расширения Direct Line Службы приложений

1. В браузере перейдите на [портал Azure](https://portal.azure.com/).
1. На портале Azure найдите ресурс **службы Azure Bot**.
1. Щелкните элемент **Каналы**, чтобы настроить каналы для бота.
1. Если канал **Direct Line** еще не включен, щелкните его, чтобы включить. 
1. Если он уже включен, в таблице "Подключение к каналам" щелкните ссылку **Изменить** в строке Direct Line.
1. Прокрутите вниз до раздела **Ключи расширений Службы приложений**. 
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

### <a name="confirm-direct-line-app-extension-and-the-bot-are-initialized"></a>Подтвердите, что расширение Direct Line Службы приложений и бот инициализированы.

1. В браузере откройте страницу https://<ваша_служба_приложений>.azurewebsites.net/.bot. Если все настроено правильно, вы увидите такое содержимое JSON: `{"k":true,"ib":true,"ob":true,"initialized":true}`. В этих данных, которые указывают на то, что **все работает правильно**:

    - **k** определяет, может ли расширение Direct Line Службы приложений (ASE) считывать соответствующий ключ расширения из своей конфигурации. 
    - **initialized** определяет, может ли ASE Direct Line с помощью соответствующего ключа расширения скачивать метаданные бота из службы Azure Bot.
    - **ib** определяет, может ли ASE Direct Line устанавливать входящее соединение с ботом.
    - **ob** определяет, может ли ASE Direct Line устанавливать исходящее соединение с ботом. 

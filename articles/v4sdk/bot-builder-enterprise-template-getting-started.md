---
title: Развертывание шаблона Enterprise Bot | Документация Майкрософт
description: Узнайте, как развернуть все вспомогательные ресурсы Azure для бота Enterprise Bot.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: e888b2473269cf576fd9edda0d99a30aa6212f7b
ms.sourcegitcommit: b2245df2f0a18c5a66a836ab24a573fd70c7d272
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/07/2019
ms.locfileid: "57571881"
---
# <a name="enterprise-bot-template---getting-started"></a>Enterprise Bot Template — начало работы

> [!NOTE]
> Приведенные здесь сведения относятся к пакету SDK версии 4. 

## <a name="prerequisites"></a>Предварительные требования

Установите следующие компоненты:
- [модуль VSIX для Enterprise Bot Template](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4enterprise);
- [пакет SDK для .NET Core](https://www.microsoft.com/net/download) (последней версии);
- [диспетчер пакетов узла](https://nodejs.org/en/);
- [Bot Framework Emulator](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0) (последней версии);
- [Интерфейс командной строки Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
- [средства CLI службы Azure Bot](https://github.com/Microsoft/botbuilder-tools) (последних версий);
    ```shell
    npm install -g ludown luis-apis qnamaker botdispatch msbot chatdown
    ```
- [LuisGen](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/LUISGen/src/npm/readme.md).
    ```shell
    dotnet tool install -g luisgen
    ```

## <a name="create-your-bot-project"></a>Создание проекта бота
1. В Visual Studio щелкните **Файл > Создать > Проект**.
1. В разделе **Бот**выберите **Enterprise Bot Template**.

![Шаблон нового проекта](media/enterprise-template/new_project.jpg)

1. Присвойте проекту имя и нажмите кнопку **Создать**.
1. Щелкните проект правой кнопкой мыши и выберите **Выполнить сборку**, чтобы восстановить пакеты NuGet.

## <a name="deploy-your-bot"></a>Развертывание бота

Для выполнения сквозных операций ботам на основе Enterprise Template Bot необходимы такие службы Azure:
- Веб-приложение Azure
- учетная запись хранения Azure (расшифровки);
- Azure Application Insights (телеметрия);
- Azure CosmosDB (хранилище состояния беседы);
- Azure Cognitive Services — Интеллектуальная служба распознавания речи;
- Azure Cognitive Services — QnA Maker (включает Поиск Azure и Веб-приложения Azure).

Следующие действия помогут вам развернуть эти службы с помощью предоставленных скриптов развертывания.

1. Получите ключ разработки LUIS.
   - Ознакомьтесь с [этой статьей](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions), чтобы выбрать портал LUIS, соответствующий вашему региону развертывания. Обратите внимание, что адрес www.luis.ai относится к региону США, и код разработки, полученный с этого портала, не будет работать в европейском развертывании.
   - Выполнив вход, щелкните свое имя в правом верхнем углу.
   - Выберите пункт **Параметры** и запишите ключ разработки, который понадобится позднее.
    
    ![Снимок экрана ключа разработки LUIS](./media/enterprise-template/luis_authoring_key.jpg)

1. Откройте окно командной строки.
1. Войдите в учетную запись Azure с помощью Azure CLI. Список доступных вам подписок вы можете найти на странице [Подписки](https://ms.portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) портала Azure.
    ```shell
    az login
    az account set --subscription "YOUR_SUBSCRIPTION_NAME"
    ```

1. Выполните команду msbot clone services, чтобы развернуть свои службы и настроить файл .bot в проекте. **ПРИМЕЧАНИЕ. После завершения развертывания вам нужно записать секрет файла бота, который отображается в окне командной строки, так как он понадобится позднее.**

    ```shell
    msbot clone services --name "YOUR-BOT-NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\LOCALE_FOLDER" --location "REGION"
    ```

    > **Примечания**
    >- Параметр **YOUR-BOT-NAME** должен быть **глобально уникальным** и содержать только буквы в нижем регистре, числа и дефисы (-).
    >- Убедитесь, что регион развертывания, указанный вами на этом шаге, соответствует региону для портала ключа разработки LUIS (например, westus для luis.ai или westeurope для eu.luis.ai).
    >- Некоторые пользователи могут столкнуться с известной проблемой при подготовке идентификатора приложения MSA и пароля. Если вы получите ошибку `ERROR: Unable to provision MSA id automatically. Please pass them in as parameters and try again`, перейдите на страницу [https://apps.dev.microsoft.com](https://apps.dev.microsoft.com), вручную создайте приложение и запишите идентификатор приложения и его пароль или секрет. Выполните указанную выше команду msbot clone services еще раз, указав два новых аргумента — `--appId` и `--appSecret` — с полученными значениями. Возможно, вам понадобится экранировать специальные символы в пароле, который может интерпретироваться оболочкой как команда:
    >   - Для *командной строки Windows* заключите appSecret в двойные кавычки. Например, `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret "YOUR_APP_SECRET"`
    >   - Для *Windows PowerShell укажите appSecret после аргумента --%. Например, `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --% --appSecret "YOUR_APP_SECRET"`
    >   - Для *MacOS или Linux* заключите appSecret в одинарные кавычки. Например, `msbot clone services --name xxxx --luisAuthoringKey xxxx --location xxxx --folder bot.recipe --appSecret 'YOUR_APP_SECRET'`

1. По завершении развертывания сохраните секрет файла бота в файле **appsettings.json**. 
    
    ```
    "botFilePath": "./YOUR_BOT_FILE.bot",
    "botFileSecret": "YOUR_BOT_SECRET",
    ```
1. Выполните приведенную ниже команду и получите ключ инструментирования (InstrumentationKey) для экземпляра Application Insights.
    ```
    msbot list --bot YOUR_BOT_FILE.bot --secret "YOUR_BOT_SECRET"
    ```

1. Сохраните ключ инструментирования в файле **appsettings.json**:

    ```
    "ApplicationInsights": {
        "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
    }
    ```

## <a name="test-your-bot"></a>Тестирование бота

После завершения запустите проект бота в среде разработки и откройте эмулятор **Bot Framework Emulator**. В эмуляторе выберите последовательно **File > Open Bot** (Файл > Открыть бота) и перейдите к файлу .bot в каталоге.

![Снимок экрана конечной точки разработки эмулятора](./media/enterprise-template/development_endpoint.jpg)

Вы должны получить приветственное сообщение при начале беседы.

Введите ```hi```, чтобы убедиться в правильной работе бота.

## <a name="publish-your-bot"></a>Публикация бота

Тестирование можно провести комплексно и локально. Когда вы будете готовы развернуть бота в Azure для дополнительного тестирования, можно использовать следующую команду для публикации исходного кода:

```shell
az bot publish -g YOUR-BOT-NAME -n YOUR-BOT-NAME --proj-name YOUR-BOT-NAME.csproj --version v4
```

## <a name="view-your-bot-analytics"></a>Просмотр аналитики бота
Enterprise Bot Template включает предварительно настроенную панель мониторинга Power BI, которая подключается к службе Application Insights для предоставления аналитических сведений о беседах. После локального тестирования бота вы можете открыть эту панель мониторинга для просмотра данных. 

1. Скачайте панель мониторинга Power BI (файл .pbit) [здесь](https://github.com/Microsoft/AI/blob/master/solutions/analytics/ConversationalAnalyticsSample_02132019.pbit).
1. Откройте панель мониторинга в [Power BI Desktop](https://powerbi.microsoft.com/en-us/desktop/).
1. Введите идентификатор приложения Application Insights (указан в файле .bot).

    ![Где найти идентификатор приложения AppInsights в файле бота](./media/enterprise-template/appInsights_appId.jpg)

1. Дополнительные сведения о возможностях панели мониторинга Power BI можно найти [здесь](https://github.com/Microsoft/AI/tree/master/solutions/analytics).

# <a name="next-steps"></a>Дальнейшие действия

После успешного развертывания бота настройте его в соответствии со своими потребностями. Сведения о настройке бота см. [здесь](bot-builder-enterprise-template-customize.md).

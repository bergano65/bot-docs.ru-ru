---
title: Развертывание бота | Документация Майкрософт
description: Развертывание бота в облаке Azure.
keywords: deploy bot, azure deploy bot, publish bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 02/13/2019
ms.openlocfilehash: 53889703d58983a87a7a2d16622f1298d56c87db
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591032"
---
# <a name="deploy-your-bot"></a>Развертывание бота

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Завершив создание и локальную проверку бота, его можно развернуть в Azure, чтобы сделать доступным из любого расположения. Развертывание бота в Azure подразумевает оплату за используемые службы. Руководство по [управлению счетами и расходами](https://docs.microsoft.com/en-us/azure/billing/) поможет вам понять, как расшифровывать счета Azure, отслеживать использование и расходы, а также управлять учетными записями и подписками.

В этой статье описано, как развернуть в Azure боты, написанные на C# или JavaScript. Эту статью полезно изучить до выполнения описанных действий, чтобы ознакомиться со всеми процессами, связанными с развертыванием бота.

## <a name="prerequisites"></a>Предварительные требования

- Установите последнюю версию средства [msbot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
- Бот на [C#](./dotnet/bot-builder-dotnet-sdk-quickstart.md) или [JavaScript](./javascript/bot-builder-javascript-quickstart.md), разработанный на локальном компьютере.

## <a name="1-prepare-for-deployment"></a>1. Подготовка к развертыванию
Для процесса развертывания требуется целевой бот веб-приложения в Azure, в котором нужно развернуть локальный бот. Целевой бот веб-приложения и подготовленные для него ресурсы в Azure используются локальным ботом для развертывания. Эта необходимость вызвана тем, что у локального бота нет необходимых ресурсов Azure. При создании целевого бота веб-приложения для вас автоматически подготавливаются следующие ресурсы:
-   Бот веб-приложения будет использоваться для развертывания в нем локального бота.
-   План Службы приложений Azure предоставляет необходимые ресурсы для выполнения приложения Службы приложений.
-   Служба приложений для размещения веб-приложений.
-   Учетная запись хранения содержит все объекты данных службы хранилища Azure: большие двоичные объекты, файлы, очереди, таблицы и диски.

Во время создания целевого бота веб-приложения для вас будут созданы идентификатор приложения и пароль. В Azure идентификатор и пароль приложения используются для [проверки подлинности и авторизации в службе](https://docs.microsoft.com/azure/app-service/overview-authentication-authorization). Часть этой информации вы получите для использования в коде локального бота. 

> [!IMPORTANT]
> Язык шаблона бота для службы должен совпадать с языком, на котором написан сам бот.

Если вы уже создали в Azure нужный бот, новый бот веб-приложения можно не создавать.

1. Войдите на [портал Azure](https://portal.azure.com).
1. Щелкните ссылку **Создание ресурса** в верхнем левом углу окна портала Azure, затем выберите **Искусственный интеллект и машинное обучение > Web App bot** (Бот веб-приложения).
1. Откроется новая колонка с информацией о боте веб-приложения. 
1. В колонке **Служба программ-роботов**  введите необходимые сведения о своем боте.
1. Нажмите кнопку **Создать** для создания службы и развертывания бота в облаке. Это может занять несколько минут.

### <a name="download-the-source-code"></a>Скачивание исходного кода
После создания целевого бота веб-приложения следует скачать код бота с портала Azure на локальный компьютер. Так вы сможете получить ссылки на службы, которые содержатся в [файле .bot](./v4sdk/bot-file-basics.md). Это ссылки на службы бота веб-приложения, план Службы приложений, Службу приложений и учетную запись хранения. 

1. В разделе **Bot Management** (Управление ботами) щелкните **Build** (Выполнить сборку).
1. Щелкните ссылку **Download Bot source code** (Скачивание исходного кода бота) справа.
1. Следуйте инструкциям на экране, чтобы скачать код, а затем распакуйте папку.

### <a name="decrypt-the-bot-file"></a>Расшифровка файла .bot

Исходный код, скачанный с портала Azure, содержит зашифрованный файл .bot. Вам потребуется расшифровать его, чтобы скопировать значения в локальный файл .bot. Этот шаг является обязательным, чтобы скопировать фактические ссылки на службы, а не зашифрованные.  

1. Откройте ресурс "Бот веб-приложения" для своего бота на портале Azure.
1. Откройте **параметры приложения** для бота.
1. В окне **Параметры приложения** перейдите к разделу **Параметры приложения**.
1. Найдите параметр **botFileSecret** и скопируйте его значение.

Воспользуйтесь `msbot cli` для расшифровки файла.

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

### <a name="update-your-local-bot-file"></a>Обновите локальный файл .bot.

Откройте расшифрованный файл .bot. Скопируйте **все** записи, перечисленные в разделе `services`, и добавьте их в локальный файл .bot. Устраните все повторяющиеся записи или идентификаторы служб. Сохраните все дополнительные ссылки на службы, от которых зависит ваш бот. Например: 

```json
"services": [
    {
        "type": "abs",
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group-name>",
        "serviceName": "<bot-service-name>",
        "name": "<friendly-service-name>",
        "id": "1",
        "appId": "<app-id>"
    },
    {
        "type": "blob",
        "connectionString": "<connection-string>",
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group-name>",
        "serviceName": "<blob-service-name>",
        "id": "2"
    },
    {
        "type": "endpoint",
        "appId": "",
        "appPassword": "",
        "endpoint": "<local-endpoint-url>",
        "name": "development",
        "id": "3"
    },
    {
        "type": "endpoint",
        "appId": "<app-id>",
        "appPassword": "<app-password>",
        "endpoint": "<hosted-endpoint-url>",
        "name": "production",
        "id": "4"
    },
    {
        "type": "appInsights",
        "instrumentationKey": "<instrumentation-key>",
        "applicationId": "<appinsights-app-id>",
        "apiKeys": {},
        "tenantId": "<tenant-id>",
        "subscriptionId": "<subscription-id>",
        "resourceGroup": "<resource-group>",
        "serviceName": "<appinsights-service-name>",
        "id": "5"
    }
],
```

Сохраните файл.

### <a name="setup-a-repository"></a>Настройка репозитория

Чтобы реализовать непрерывное развертывание, создайте репозиторий Git с помощью любого поставщика системы управления версиями Git. Зафиксируйте свой код в репозитории. 

Убедитесь, что корень репозитория содержит нужные файлы, которые описаны в разделе [Подготовка репозитория](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment#prepare-your-repository).

### <a name="update-app-settings-in-azure"></a>Обновление параметров приложения в Azure
Локальный бот не использует зашифрованный файл .bot. 
1. Откройте ресурс **Бот веб-приложения** для своего бота на портале Azure.
1. Откройте **параметры приложения** для бота.
1. В окне **Параметры приложения** перейдите к разделу **Параметры приложения**.
1. Найдите параметр **botFileSecret** и удалите его.
1. Обновите имя файла бота, чтобы оно совпадало с именем измененного файла в репозитории.
1. Сохраните изменения.

## <a name="2-deploy-using-azure-deployment-center"></a>2. Развертывание с помощью центра развертывания Azure

Теперь следует передать код бота в Azure. Следуйте инструкциям в статье [Непрерывное развертывание в Службе приложений Azure](https://docs.microsoft.com/azure/app-service/deploy-continuous-deployment).

Учтите, что сборку рекомендуется выполнять с помощью `App Service Kudu build server`.

После настройки непрерывного развертывания автоматически публикуются все изменения, которые вы фиксируете в репозитории. Но если вы добавите в бот новые службы, придется добавить для них записи в файл .bot.

## <a name="3-test-your-deployment"></a>3. Выполните тестирование развертывания

Подождите несколько секунд после успешного завершения развертывания. Можно также перезапустить веб-приложение, чтобы очистить все кэши. Вернитесь в колонку "Бот веб-приложения" и выполните тестирование с помощью веб-чата на портале Azure.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Изучение распространенных проблем с непрерывным развертыванием](https://github.com/projectkudu/kudu/wiki/Investigating-continuous-deployment)

<!--

## Prerequisites

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## Deploy JavaScript and C# bots using az cli

You've already created and tested a bot locally, and now you want to deploy it to Azure. These steps assume that you have created the required Azure resources.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### Create a Web App Bot

If you don't already have a resource group to which to publish your bot, create one:

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

Before proceeding, read the instructions that apply to you based on the type of email account you use to log in to Azure.

#### MSA email account

If you are using an [MSA](https://en.wikipedia.org/wiki/Microsoft_account) email account, you will need to create the app ID and app password on the Application Registration Portal to use with `az bot create` command.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### Business or school account

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### Download the bot from Azure

Next, download the bot you just created. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

### Decrypt the downloaded .bot file and use in your project

The sensitive information in the .bot file is encrypted.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### Update the .bot file

If your bot uses LUIS, QnA Maker, or Dispatch services, you will need to add references to them to your .bot file. Otherwise, you can skip this step.

1. Open your bot in the BotFramework Emulator, using the new .bot file. The bot does not need to be running locally.
1. In the **BOT EXPLORER** panel, expand the **SERVICES** section.
1. To add references to LUIS apps, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Language Understanding (LUIS)**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of LUIS applications you have access to. Select the ones for your bot.
1. To add references to a QnA Maker knowledge base, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add QnA Maker**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of knowledge bases you have access to. Select the ones for your bot.
1. To add references to Dispatch models, click the plus-sign (+) to the right of **SERVICES**.
   1. Select **Add Dispatch**.
   1. If it prompts you to log into your Azure account, do so.
   1. It presents a list of Dispatch models you have access to. Select the ones for your bot.

### Test your bot locally

At this point, your bot should work the same way it did with the old .bot file. Make sure that it works as expected with the new .bot file.

### Publish your bot to Azure

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]


[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## Additional resources

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## Next steps
> [!div class="nextstepaction"]
> [Set up continous deployment](bot-service-build-continuous-deployment.md)

-->

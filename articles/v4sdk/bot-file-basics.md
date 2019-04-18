---
title: Управление ресурсами с помощью файла .bot | Документация Майкрософт
description: Описание назначения и применения файла бота.
keywords: bot file, .bot, .bot file, bot resources, manage bot resources
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/30/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 14552c55da4b1f9b581b81917496de179e92762b
ms.sourcegitcommit: 721bb09f10524b0cb3961d7131966f57501734b8
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/18/2019
ms.locfileid: "58811506"
---
# <a name="manage-bot-resources"></a>Управление ресурсами бота

Боты часто используют разные службы, например [LUIS.ai](https://luis.ai) или [QnaMaker.ai](https://qnamaker.ai). При разработке ботов вам необходимо иметь возможность отслеживать их все. Вы можете использовать разные средства, например файлы appsettings.json, web.config или .env. 

> [!IMPORTANT]
> До выпуска для Bot Framework пакета SDK версии 4.3 мы предлагали файл .bot в качестве средства управления ресурсами. Но в будущем мы рекомендуем вам использовать для этого файлы appsettings.json или .env. Боты, которые используют файл .bot, пока будут работать и дальше, хотя файл .bot был объявлен **_нерекомендуемым_**. Если вы используете файл .bot для управления ресурсами, выполните соответствующие шаги по переносу параметров. 

## <a name="migrating-settings-from-bot-file"></a>Перенос параметров из файла .bot
В разделах ниже описано, как перенести параметры из файла .bot. Следуйте подходящему для вас сценарию.

**Сценарий 1. Локальный бот с файлом .bot**

В этом сценарии вам доступен локальный бот, который использует файл .bot, но _бот не был перенесен_ на портал Azure. Выполните следующие шаги, чтобы перенести параметры из файла .bot в файл appsettings.json или .env.

- Если файл .bot зашифрован, расшифруйте его с помощью следующей команды:

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear` command.
```

- Откройте расшифрованный файл .bot, скопируйте значения и добавьте их в файл appsettings.json или .env.
- Обновите код, чтобы считать параметры из файла appsettings.json или .env.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В методе `ConfigureServices` воспользуйтесь предоставляемым ASP.NET Core объектом конфигурации, например: 

**Startup.cs.**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В JavaScript укажите для файла .env ссылки на объект `process.env`, например:
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

При необходимости подготовьте ресурсы и подключите их к своему боту с помощью файла appsettings.json или .env.

**Сценарий 2. Бот, развернутый в Azure с помощью файла .bot**

В этом сценарии вы уже развернули бот на портале Azure с помощью файла .bot и теперь хотите перенести параметры из файла .bot в файл appsettings.json или .env.

- Скачайте код бота с портала Azure. При скачивании кода вам будет предложено включить файл appsettings.json или .env, который будет содержать MicrosoftAppId и MicrosoftAppPassword, а также другие дополнительные параметры. 
- Откройте _скачанный_ файл appsettings.json или .env и скопируйте из него параметры в _локальный_ файл appsettings.json или .env. Не забудьте удалить записи botSecret и botFilePath из локального файла appsettings.json или .env.
- Обновите код, чтобы считать параметры из файла appsettings.json или .env.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
В методе `ConfigureServices` воспользуйтесь предоставляемым ASP.NET Core объектом конфигурации, например: 

**Startup.cs.**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```
# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
В JavaScript укажите для файла .env ссылки на объект `process.env`, например:
   
**index.js**

```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```
---

Вам также потребуется удалить записи `botFilePath` и `botFileSecret` из раздела **Параметры приложения** на **портале Azure**.

_При необходимости_ подготовьте ресурсы и подключите их к своему боту с помощью файла appsettings.json или .env.

**Сценарий 3. Боты, которые используют файл appsettings.json или .env**

В этом сценарии рассматривается случай, когда вы с нуля разрабатываете боты с помощью пакета SDK 4.3 и у вас нет файлов .bot для переноса. Все параметры, которые вы хотите использовать в своем боте, доступны в файле appsettings.json или .env, как показано ниже:

```JSON
{
  "MicrosoftAppId": "<your-AppId>",
  "MicrosoftAppPassword": "<your-AppPwd>"
}
```

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы считать указанные выше параметры в коде C#, вам нужно воспользоваться предоставляемым ASP.NET Core объектом конфигурации, например: **Startup.cs.**
```csharp
var appId = Configuration.GetSection("MicrosoftAppId").Value;
var appPassword = Configuration.GetSection("MicrosoftAppPassword").Value;
options.CredentialProvider = new SimpleCredentialProvider(appId, appPassword);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)
В JavaScript укажите для файла .env ссылки на объект `process.env`, например: **index.js**
```js
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});
```

---

При необходимости подготовьте ресурсы и подключите их к своему боту с помощью файла appsettings.json или .env.


## <a name="faq"></a>Часто задаваемые вопросы
**Вопрос.** Я хочу создать нового бота версии 4 на портале Azure. Как изменилась работа с порталом Azure после того, как файл .bot больше не используется?

**Ответ.** При создании бота на портале Azure файл .bot не будет создан. Вы можете перейти в раздел **Параметры приложения** на **портале Azure**, чтобы найти идентификаторы и ключи. При скачивании кода эти параметры будут сохранены в файле appsettings.json или .env. Вы можете обновить код своего бота для считывания параметров до вызова отдельной службы. Обновив код бота, вы можете воспользоваться командой az bot publish, чтобы развернуть своего бота.

**Вопрос.** Что будет с ботами версии 3?

**Ответ.** Сценарий для бота версии 3 похож на сценарий для ботов версии 4, но без использования файла .bot. Развертывание при этом будет выполняться в обычном режиме. 

## <a name="additional-resources"></a>Дополнительные ресурсы
- Подробнее см. в [инструкциях по развертыванию бота](../bot-builder-deploy-az-cli.md).
- Для защиты ключей и секретов мы рекомендуем использовать Azure Key Vault. Azure Key Vault — это средство для безопасного хранения секретов, например конечных точек вашего бота и ключей разработки, а также доступа к ним. Оно предоставляет решение для [управления ключами](https://docs.microsoft.com/en-us/azure/key-vault/key-vault-whatis) и упрощает создание и администрирование ключей шифрования, позволяя вам полностью контролировать секреты.


<!--

# Manage resources with a .bot file

Bots usually consume lots of different services, such as [LUIS.ai](https://luis.ai) or [QnaMaker.ai](https://qnamaker.ai). When you are developing a bot, there is no uniform place to store the metadata about the services that are in use.  This prevents us from building tooling that looks at a bot as a whole.

To address this problem, we have created a **.bot file** to act as the place to bring all service references together in one place to 
enable tooling.  For example, the Bot Framework Emulator ([V4](https://aka.ms/Emulator-wiki-getting-started)) uses a  .bot file to create a unified view over the connected services your bot consumes.  

With a .bot file, you can register services like:

* **Localhost** local debugger endpoints
* [**Azure Bot Service**](https://azure.microsoft.com/en-us/services/bot-service/) Azure Bot Service registrations.
* [**LUIS.AI**](https://www.luis.ai/) LUIS gives your bot the ability to communicate with people using natural language.. 
* [**QnA Maker**](https://qnamaker.ai/) Build, train and publish a simple question and answer bot based on FAQ URLs, structured documents or editorial content in minutes.
* [**Dispatch**](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch) models for dispatching across multiple services.
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/) for insights and bot analytics.
* [**Azure Blob Storage**](https://azure.microsoft.com/en-us/services/storage/blobs/) for bot state persistence. 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) - globally distributed, multi-model database service to persist bot state.

Apart from these, your bot might rely on other custom services. You can leverage the [generic service](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md) capability to connect a generic service configuration.

## When is a .bot file created? 
- If you create a bot using [Azure Bot Service](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D), a .bot file is automatically created for you with list of connected services provisioned. The .bot is encrypted by default.
- If you create a bot using Bot Framework V4 SDK [Template](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) for Visual Studio or using Bot Builder [Yeoman Generator](https://www.npmjs.com/package/generator-botbuilder), a .bot file is automatically created. No connected services are provisioned in this flow and the bot file is not encrypted.
- If you are starting with [BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples), every sample for Bot Framework V4 SDK includes a .bot file and the .bot file is not encrypted. 
- You can also create a bot file using the [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) tool.

## What does a bot file look like? 
Take a look at a sample [.bot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) file.
To learn about encrypting and decrypting the .bot file, see [Bot Secrets](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md).

## Why do I need a .bot file?

A .bot file is **not** a requirement to build bots with Bot Framework SDK. You can continue to use appsettings.json, web.config, env, 
keyvault or any mechanism you see fit to keep track of service references and keys that your bot depends on. However, to test
the bot using the Emulator, you'll need a .bot file. The good news is that Emulator can create a .bot file for testing. To do that, 
start the Emulator, click on the **create a new bot configuration** link on the Welcome page. In the dialog box that appears, type a **Bot name** and an **Endpoint URL**. Then connect.

The advantages of using .bot file are:
- Provides a standard way of storing resources regardless of the language/platform you use.   
- Bot Framework Emulator and CLI tools rely on and work great with tracking connected services in a consistent format (in a .bot file) 
- Elegant tooling solutions around services creation and management is harder without a well defined schema (.bot file).  


## Using .bot file in your Bot Framework SDK bot

You can use the .bot file to get service configuration information in your bot's code. The BotFramework-Configuration library available 
for [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) and [JS](https://www.npmjs.com/package/botframework-config) helps you load a bot file and supports several methods to query and get the appropriate service configuration information.

## Additional resources
Refer to [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md) readme file for more information on using a bot file.

-->


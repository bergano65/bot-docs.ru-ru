---
title: Управление ресурсами бота | Документация Майкрософт
description: Описание назначения и применения файла бота.
keywords: bot file, .bot, .bot file, msbot, bot resources, manage bot resources
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 14c6b9c180d02acbb7c8f3df7843bf90bc0e400a
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299135"
---
# <a name="manage-bot-resources"></a>Управление ресурсами бота

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

Боты часто используют разные службы, например [LUIS.ai](https://luis.ai) или [QnaMaker.ai](https://qnamaker.ai). При разработке ботов вам необходимо иметь возможность отслеживать их все. Вы можете использовать разные средства, например файлы appsettings.json, web.config или .env. 

> [!IMPORTANT]
> До выпуска для Bot Framework пакета SDK версии 4.3 мы предлагали файл .bot в качестве средства управления ресурсами. Но в будущем мы рекомендуем вам использовать для этого файлы appsettings.json или .env. Боты, которые используют файл .bot, пока будут работать и дальше, хотя файл .bot был объявлен **_нерекомендуемым_** . Если вы используете файл .bot для управления ресурсами, выполните соответствующие шаги по переносу параметров. 

## <a name="migrating-settings-from-bot-file"></a>Перенос параметров из файла .bot
В разделах ниже описано, как перенести параметры из файла .bot. Следуйте подходящему для вас сценарию.

**Сценарий 1. Локальный бот с файлом .bot**

В этом сценарии вам доступен локальный бот, который использует файл .bot, но _бот не был перенесен_ на портал Azure. Выполните следующие шаги, чтобы перенести параметры из файла .bot в файл appsettings.json или .env.

- Если файл .bot зашифрован, расшифруйте его с помощью следующей команды:

```cli
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
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

*При необходимости* подготовьте ресурсы и подключите их к своему боту с помощью файла appsettings.json или .env.

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

*При необходимости* подготовьте ресурсы и подключите их к своему боту с помощью файла appsettings.json или .env.

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

## <a name="additional-resources"></a>Дополнительные ресурсы
- Подробнее см. в [инструкциях по развертыванию бота](../bot-builder-deploy-az-cli.md).
- Узнайте, как использовать [Azure Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview) для защиты и администрирования криптографических ключей и секретов, используемых облачными приложениями и службами.

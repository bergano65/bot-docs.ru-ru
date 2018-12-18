---
title: Развертывание бота с помощью Azure CLI | Документация Майкрософт
description: Развертывание бота в облаке Azure.
keywords: deploy bot, azure deploy, publish bot, az deploy bot, visual studio deploy bot, msbot publish, msbot clone
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/11/2018
ms.openlocfilehash: a7cb9cb1e3df14f2f46bc5a4c3a633f5212b5dfd
ms.sourcegitcommit: 0b421ff71617f03faf55ea175fb91d1f9e348523
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/12/2018
ms.locfileid: "53286620"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Развертывание бота с помощью Azure CLI

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Завершив создание и локальную проверку бота, его можно развернуть в Azure, чтобы сделать доступным из любого расположения. Развертывание бота в Azure подразумевает оплату за используемые службы. Руководство по [управлению счетами и расходами](https://docs.microsoft.com/en-us/azure/billing/) поможет вам понять, как расшифровывать счета Azure, отслеживать использование и расходы, а также управлять учетными записями и подписками.

В этой статье описано, как развернуть в Azure боты, написанные на C# или JavaScript, с помощью средства `msbot`. Эту статью полезно изучить до выполнения описанных действий, чтобы ознакомиться со всеми процессами, связанными с развертыванием бота.


## <a name="prerequisites"></a>Предварительные требования
- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.
- Установите [пакет SDK для .NET Core](https://dotnet.microsoft.com/download) с версией не ниже 2.2. 
- Установите последнюю версию [средства Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
- Установите последнюю версию расширения `botservice` для средства `az`. 
  - Во-первых, удалите старую версию с помощью команды `az extension remove -n botservice`. Затем выполните команду `az extension add -n botservice`, чтобы установить последнюю версию.
- Установите последнюю версию средства [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
  - Вам потребуется [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation), если операция клонирования затрагивает ресурсы LUIS или диспетчера ресурсов.
  - Вам потребуется [QnA Maker CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli), если операция клонирования затрагивает ресурсы QnA Maker.
- Установленное приложение [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Установленное и настроенное средство [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Понимание структуры файла с расширением [.bot](v4sdk/bot-file-basics.md).

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Развертывание ботов, написанных на JavaScript и C#, с помощью az cli
Итак, вы уже создали бот и хотите развернуть его в Azure. Предполагается, что вы уже создали все необходимые ресурсы Azure и обновили ссылки на службы в файле с расширением .bot с помощью команды `msbot connect` или Bot Framework Emulator. Если вы не обновите файл .bot, процесс развертывания может завершиться без ошибок или предупреждений, но развернутый бот не будет работать.

Откройте командную строку, чтобы войти на портал Azure.

```cmd
az login
```
Она откроет окно браузера с интерфейсом для входа. 

### <a name="set-the-subscription"></a>Настройка подписки 
Выберите подписку с помощью следующей команды:

```cmd
az account set --subscription "<azure-subscription>"
``` 

Если вы не уверены, какую подписку выбрать для развертывания бота, просмотрите список `subscriptions` для учетной записи с помощью команды `az account list`.

Перейдите в папку бота. 
```cmd 
cd <local-bot-folder>
```

### <a name="azure-subscription-account"></a>Учетная запись подписки Azure
Прежде чем продолжить работу, ознакомьтесь с описанными ниже инструкциями, выбрав нужный раздел в зависимости от типа учетной записи электронной почты, с которой вы входите в Azure.

**Учетная запись электронной почты MSA**

Если вы используете учетную запись электронной почты [MSA](https://en.wikipedia.org/wiki/Microsoft_account), подготовьте значения appId и appSecret для использования в команде `msbot clone services`. 

- Перейдите на [портал регистрации приложений](https://apps.dev.microsoft.com/). Щелкните **Add an app** (Добавить приложение), чтобы зарегистрировать приложение, затем создайте **Application Id** (Идентификатор приложения) и щелкните **Generate New Password** (Создать пароль). 
- Сохраните идентификатор приложения и созданный пароль, чтобы применить их в команде `msbot clone services`. 
- Для развертывания выберите правильную команду в зависимости от типа бота.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appid "xxxxxxxx" --password "xxxxxxx" --verbose`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

**Рабочая или учебная учетная запись**

Если вы используете для входа в Azure учетную запись электронной почты, которую вам предоставила компания или учебное заведение, идентификатор приложения и пароль создавать не нужно. Для развертывания выберите правильную команду в зависимости от типа бота.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>"`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>"`


[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="save-the-secret-used-to-decrypt-bot-file"></a>Сохранение секрета для расшифровки файла .bot
Здесь важно отметить, что в ходе развертывания создается _файл .bot, который шифруется с помощью секрета_. При развертывании этого бота вы увидите в командной строке следующее сообщение с предложением сохранить секрет для файла .bot. 

`The secret used to decrypt myAzBot.bot is:`
`hT6U1SIPQeXlebNgmhHYxcdseXWBZlmIc815PpK0WWA=`

`NOTE: This secret is not recoverable and you should save it in a safe place according to best security practices.
      Copy this secret and use it to open the <file.bot> the first time.`
      
Сохраните этот секрет файла .bot для дальнейшего использования. Новый зашифрованный файл .bot используется на портале Azure совместно с botFileSecret. Если позже потребуется изменить имя файла бота или секрет, перейдите на страницу **Параметры Службы приложений > Параметры приложения** на портале. Учтите, что имя файла бота в файле с расширением appsettings.json или .env всегда заменяется именем файла для последнего созданного бота.  

### <a name="test-your-bot"></a>Тестирование бота
В эмуляторе выберите рабочую конечную точку для тестирования приложения. Если вы хотите выполнить тестирование локально, убедитесь, что бот выполняется на локальном компьютере. 

### <a name="to-update-your-bot-code-in-azure"></a>Обновление кода бота в Azure
НЕ ИСПОЛЬЗУЙТЕ команду `msbot clone services` для обновления кода бота в Azure. Следует использовать команду `az bot publish`, как показано ниже:

```cmd
az bot publish --name "<your-azure-bot-name>" --proj-file "<your-proj-file>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| Аргументы        | ОПИСАНИЕ |
|----------------  |-------------|
| `name`      | Имя, указанное при первом развертывании бота в Azure.|
| `proj-file` | Для бота на C# используется файл с расширением .csproj. Для бота на JS/TS используется имя файла запуска (например index.js или index.ts) из локального проекта бота.|
| `resource-group` | Группа ресурсов Azure, которая используется командой `msbot clone services`.|
| `code-dir`  | Указывает на локальную папку бота.|



## <a name="additional-resources"></a>Дополнительные ресурсы

При развертывании бота, как правило, на портале Azure создаются следующие ресурсы:

| Ресурсы      | ОПИСАНИЕ |
|----------------|-------------|
| Бот веб-приложения | Бот в службе Azure Bot, который развернут в Службе приложений Azure.|
| [Служба приложений](https://docs.microsoft.com/en-us/azure/app-service/)| Позволяет создавать и размещать веб-приложения.|
| [План обслуживания приложения](https://docs.microsoft.com/en-us/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Определяет набор вычислительных ресурсов, на которых выполняется веб-приложение.|
| [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-overview)| Предоставляет средства для сбора и анализа данных телеметрии.|
| [Учетная запись хранения](https://docs.microsoft.com/en-us/azure/storage/common/storage-introduction)| Предоставляет облачную службу с высоким уровнем доступности, безопасности, надежности, масштабируемости и избыточности.|

Для просмотра документации по команде `az bot` см. этот раздел [справки](https://docs.microsoft.com/en-us/cli/azure/bot?view=azure-cli-latest).

Если вы еще не знакомы с концепцией группы ресурсов в Azure, см. раздел со списком [терминов](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview#terminology).

## <a name="next-steps"></a>Дополнительная информация
> [!div class="nextstepaction"]
> [Настройка непрерывного развертывания](bot-service-build-continuous-deployment.md)

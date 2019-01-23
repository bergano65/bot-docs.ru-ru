---
title: Развертывание ботов из репозитория botbuilder-samples | Документация Майкрософт
description: Развертывание бота в облаке Azure.
keywords: deploy bot, azure deploy, publish bot, az deploy bot, visual studio deploy bot, msbot publish, msbot clone
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78e960357d6c4dc1c9751a9921a2338f552738b0
ms.sourcegitcommit: b94361234816e6b95459f142add936732fc40344
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/15/2019
ms.locfileid: "54317620"
---
# <a name="deploy-bots-from-botbuilder-samples-repo"></a>Развертывание ботов из репозитория botbuilder-samples

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

В этой статье мы продемонстрируем развертывание в Azure примеров на C# и JavaScript, размещенных в репозитории [botbuilder-samples](https://github.com/Microsoft/BotBuilder-Samples).

Инструкции по развертыванию тестовых ботов _отличаются_ от инструкций [по развертыванию ботов, которые вы можете создать, когда все ресурсы уже подготовлены в Azure](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-deploy-az-cli?view=azure-bot-service-4.0&tabs=csharp).

> [!IMPORTANT]
> Развертывание в Azure бота, опубликованного в репозитории [botbuilder-samples](https://github.com/Microsoft/BotBuilder-Samples), включает подготовку служб Azure и подразумевает оплату используемых служб.
> Руководство по [управлению счетами и расходами](https://docs.microsoft.com/en-us/azure/billing/) поможет вам понять, как расшифровывать счета Azure, отслеживать использование и расходы, а также управлять учетными записями и подписками.

Эту статью лучше прочитать полностью до того, как вы начнете выполнять описанные действия, чтобы хорошо понимать все процессы, связанные с развертыванием бота.

## <a name="prerequisites"></a>Предварительные требования

- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.
- Установите [пакет SDK для .NET Core](https://dotnet.microsoft.com/download) версии не ниже 2.2. Используемый номер версии можно проверить с помощью `dotnet --version`.
- Установите последнюю версию [средства Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest). Используемый номер версии можно проверить с помощью `az --version`.
- Установите последнюю версию средства [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
  - Вам потребуется [LUIS CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#installation), если операция клонирования затрагивает ресурсы LUIS или диспетчера ресурсов.
  - Вам потребуется [QnA Maker CLI](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#as-a-cli), если операция клонирования затрагивает ресурсы QnA Maker.
- Установленное приложение [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Установленное и настроенное средство [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Понимание структуры файлов с расширением [.bot](v4sdk/bot-file-basics.md).

Для работы с msbot 4.3.2 и более поздними версиями вам понадобится Azure CLI версии 2.0.54 или более поздней. Если вы установили расширение botservice, удалите его с помощью следующей команды.

```cmd
az extension remove --name botservice
```

### <a name="c"></a>C\#

 `msbot clone services` не отправляет в Azure файлы кода, а только библиотеку DLL и еще несколько других файлов. Тестовый пример следует скомпилировать перед выполнением этой команды.

### <a name="service-names"></a>Имена служб

Помимо развертывания бота, команда `msbot clone services`подготовит все связанные службы в выбранной подписке.

Если какие-то сочетания имен и служб уже существуют в этой подписке, команда вернет ошибку. Перед новой попыткой не забудьте удалить частично развернутые ресурсы. Сюда относятся приложения LUIS, базы знаний QnA Maker и модели диспетчеризации.

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Развертывание ботов, написанных на JavaScript и C#, с помощью az cli

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

### <a name="clone-the-sample"></a>Клонирования репозитория

**Учетная запись подписки Azure**. Прежде чем продолжить работу, ознакомьтесь с описанными ниже инструкциями, выбрав нужный раздел в зависимости от типа учетной записи электронной почты, с которой вы входите в Azure.

**Создание служб**. Команда `msbot clone services` создает для бота нужные службы Azure.

1. Она перечисляет службы, которые будут созданы, и предлагает подтвердить согласие на продолжение работы. Если вы откажетесь, команда завершает работу без создания каких-либо служб.
1. Для продолжения работы потребуется пройти проверку подлинности в Azure.

**Службы LUIS**. Если бот использует LUIS или службу диспетчеризации, включите в команду `msbot clone services` ключ разработки LUIS.

#### <a name="msa-email-account"></a>Учетная запись электронной почты MSA

Если вы используете учетную запись электронной почты [MSA](https://en.wikipedia.org/wiki/Microsoft_account), подготовьте значения appId и appSecret для использования в команде `msbot clone services`.

- Перейдите на [портал регистрации приложений](https://apps.dev.microsoft.com/). Щелкните **Add an app** (Добавить приложение), чтобы зарегистрировать приложение, затем создайте **Application Id** (Идентификатор приложения) и щелкните **Generate New Password** (Создать пароль).
- Сохраните идентификатор приложения и созданный пароль, чтобы применить их в команде `msbot clone services`.
- Для развертывания выберите правильную команду в зависимости от типа бота.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --proj-file "<your.csproj>" --name "<bot-name>" --appId "xxxxxxxx" --appSecret "xxxxxxx" --verbose --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>"   --code-dir . --name "<bot-name>" --appId "xxxxxxxx" --appSecret "xxxxxxx" --verbose --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

---

#### <a name="business-or-school-account"></a>Рабочая или учебная учетная запись

Если вы используете для входа в Azure учетную запись электронной почты, которую вам предоставила компания или учебное заведение, идентификатор приложения и пароль создавать не нужно. Для развертывания выберите правильную команду в зависимости от типа бота.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --proj-file "<your-project-file>" --name "<bot-name>" --luisAuthoringKey <luis-authoring-key>`

[!INCLUDE [deployment note](./includes/deployment-note-cli.md)]

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

`msbot clone services --folder deploymentScripts/msbotClone --location "<geographic-location>" --verbose --code-dir . --name "<bot-name>" --luisAuthoringKey <luis-authoring-key>`

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
az bot publish --name "<your-azure-bot-name>" --proj-name "<your-proj-name>" --resource-group "<azure-resource-group>" --code-dir "<folder>" --verbose --version v4
```

| Аргументы        | ОПИСАНИЕ |
|----------------  |-------------|
| `name`      | Имя, указанное при первом развертывании бота в Azure.|
| `proj-name` | Для C# введите имя начального файла (без расширения .csproj) в публикуемом проекте. Например, `EnterpriseBot`. Для Node.js используйте главную точку входа бота. Например, `index.js`. |
| `resource-group` | Группа ресурсов Azure, которая используется командой `msbot clone services`.|
| `code-dir`  | Указывает на локальную папку бота.|

## <a name="additional-resources"></a>Дополнительные ресурсы

При развертывании бота на портале Azure обычно создаются следующие ресурсы.

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

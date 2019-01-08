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
ms.date: 12/14/2018
ms.openlocfilehash: 19960940a40fa291534bc1f88290bc6a7da109e0
ms.sourcegitcommit: 8c10aa7372754596a3aa7303a3a893dd4939f7e9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/20/2018
ms.locfileid: "53654339"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Развертывание бота с помощью Azure CLI

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Завершив создание и локальную проверку бота, его можно развернуть в Azure, чтобы сделать доступным из любого расположения. Развертывание бота в Azure подразумевает оплату за используемые службы. Руководство по [управлению счетами и расходами](https://docs.microsoft.com/en-us/azure/billing/) поможет вам понять, как расшифровывать счета Azure, отслеживать использование и расходы, а также управлять учетными записями и подписками.

В этой статье объясняется, как развернуть в Azure боты, написанные на C# или JavaScript, с помощью средств командной строки `az` и `msbot`. Эту статью полезно изучить до выполнения описанных действий, чтобы ознакомиться со всеми процессами, связанными с развертыванием бота.


## <a name="prerequisites"></a>Предварительные требования
- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.
- Установите последнюю версию [средства Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
- Установите последнюю версию расширения `botservice` для средства `az`.
  - Во-первых, удалите старую версию с помощью команды `az extension remove -n botservice`. Затем выполните команду `az extension add -n botservice`, чтобы установить последнюю версию.
- Установите последнюю версию средства [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
- Установите последнюю версию [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Установленное и настроенное средство [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Понимание структуры файла с расширением [.bot](v4sdk/bot-file-basics.md).

## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Развертывание ботов, написанных на JavaScript и C#, с помощью az cli

Итак, вы уже создали бот и протестировали его локально, а теперь хотите развернуть его в Azure. В этих шагах предполагается, что вы уже создали необходимые ресурсы Azure.

Откройте командную строку, чтобы войти на портал Azure.

```cmd
az login
```

Она откроет окно браузера с интерфейсом для входа.

### <a name="set-the-subscription"></a>Настройка подписки

Укажите подписку, которая будет использоваться по умолчанию.

```cmd
az account set --subscription "<azure-subscription>"
```

Если вы не уверены, какую подписку выбрать для развертывания бота, просмотрите список `subscriptions` для учетной записи с помощью команды `az account list`.

Перейдите в папку бота.

```cmd
cd <local-bot-folder>
```

### <a name="create-a-web-app-bot"></a>Создание бота веб-приложения

Создайте ресурс бота, в котором вы намерены опубликовать этот бот.

Прежде чем продолжить работу, ознакомьтесь с описанными ниже инструкциями, выбрав нужный раздел в зависимости от типа учетной записи электронной почты, с которой вы входите в Azure.

#### <a name="msa-email-account"></a>Учетная запись электронной почты MSA

Если вы используете учетную запись электронной почты [MSA](https://en.wikipedia.org/wiki/Microsoft_account), вам потребуется создать идентификатор и пароль приложения на портале регистрации приложений для использования в команде `az bot create`.

1. Перейдите на [**портал регистрации приложений**](https://apps.dev.microsoft.com/).
1. Щелкните **Add an app** (Добавить приложение), чтобы зарегистрировать приложение, затем создайте **Application Id** (Идентификатор приложения) и щелкните **Generate New Password** (Создать пароль). Если у вас уже есть приложение и пароль для него, но вы не помните этот пароль, следует создать новый пароль в разделе секретов приложения.
1. Сохраните идентификатор приложения и созданный пароль, чтобы применить их в команде `az bot create`.  

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --name | Уникальное имя, которое используется для развертывания бота в Azure. Это имя может совпадать с именем локального бота. НЕ ИСПОЛЬЗУЙТЕ пробелы и подчеркивания в этом имени. |
| --location | Географическое расположение для создания ресурсов службы бота. Например, `eastus`, `westus`, `westus2` и т. д. |
| --lang | Язык, используемый для создания бота: `Csharp` или `Node` (по умолчанию `Csharp`). |
| --resource-group | Имя группы ресурсов, в которой будет создан бот. Вы можете настроить расположение по умолчанию с помощью `az configure --defaults group=<name>`. |
| --appid | Идентификатор учетной записи Майкрософт (MSA), который будет использоваться с этим ботом. |
| --password | Пароль учетной записи Майкрософт (MSA) для бота. |

#### <a name="business-or-school-account"></a>Рабочая или учебная учетная запись

```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --name | Уникальное имя, которое используется для развертывания бота в Azure. Это имя может совпадать с именем локального бота. НЕ ИСПОЛЬЗУЙТЕ пробелы и подчеркивания в этом имени. |
| --location | Географическое расположение для создания ресурсов службы бота. Например, `eastus`, `westus`, `westus2` и т. д. |
| --lang | Язык, используемый для создания бота: `Csharp` или `Node` (по умолчанию `Csharp`). |
| --resource-group | Имя группы ресурсов, в которой будет создан бот. Вы можете настроить расположение по умолчанию с помощью `az configure --defaults group=<name>`. |

### <a name="download-the-bot-from-azure"></a>Скачивание бота из Azure

Теперь скачайте бот, который вы только что создали. Эта команда создает подкаталог в каталоге save-path, но сам этот каталог уже должен существовать.

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --name | Имя бота в Azure. |
| --resource-group | Имя группы ресурсов, в которой размещен бот. |
| --save-path | Существующий каталог, в который будет скачиваться код бота. |

### <a name="decrypt-the-downloaded-bot-file"></a>Расшифровка скачанного файла .bot

Все конфиденциальные сведения в файле .bot шифруются.

Получите ключ шифрования.

1. Войдите на [портал Azure](http://portal.azure.com/).
1. Откройте ресурс Web App Bot (Бот веб-приложения) для своего бота.
1. Откройте **параметры приложения** для бота.
1. В окне **Параметры приложения** прокрутите страницу вниз до раздела **Параметры приложения**.
1. Найдите параметр **botFileSecret** и скопируйте его значение.

Расшифруйте файл .bot.

```cmd
msbot secret --bot <name-of-bot-file> --secret "<bot-file-secret>" --clear
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --bot | Относительный путь к скачанному файлу .bot. |
| --secret | Ключ шифрования. |

### <a name="use-the-downloaded-bot-file-in-your-project"></a>Применение скачанного файла .bot в проекте

Скопируйте расшифрованный файл .bot в каталог, где хранится локальный проект бота.

Обновите бот, чтобы он использовал новый файл .bot.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В файле **appsettings.json** обновите свойство **botFilePath**, чтобы оно указывало на новый файл .bot.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В файле **.env** обновите свойство **botFilePath**, чтобы оно указывало на новый файл .bot.

---

### <a name="update-the-bot-file"></a>Обновление файла .bot

Если ваш бот использует службы LUIS, QnA Maker или службу диспетчеризации, следует добавить ссылки на них в файл .bot. В противном случае этот шаг можно пропустить.

1. Откройте бот в эмуляторе BotFramework, используя новый файл .bot. Бот не нужно запускать локально.
1. На панели **обозревателя ботов** разверните раздел **СЛУЖБЫ**.
1. Чтобы добавить ссылки на приложения LUIS, щелкните значок плюс (+) справа от раздела **СЛУЖБЫ**.
   1. Выберите **Add Language Understanding (LUIS)** (Добавить "Распознавание речи" (LUIS)).
   1. Если появится запрос на вход в учетную запись Azure, выполните его.
   1. Здесь вы увидите список приложений LUIS, к которым у вас есть доступ. Выберите те, которые нужны для вашего бота.
1. Чтобы добавить ссылки на базу знаний QnA Maker, щелкните значок плюс (+) справа от раздела **СЛУЖБЫ**.
   1. Выберите **Add QnA Maker** (Добавить QnA Maker).
   1. Если появится запрос на вход в учетную запись Azure, выполните его.
   1. Здесь вы увидите список баз знаний, к которым у вас есть доступ. Выберите те, которые нужны для вашего бота.
1. Чтобы добавить ссылки на модели диспетчеризации, щелкните значок плюс (+) справа от раздела **СЛУЖБЫ**.
   1. Выберите **Add Dispatch** (Добавить диспетчеризацию).
   1. Если появится запрос на вход в учетную запись Azure, выполните его.
   1. Здесь вы увидите список моделей диспетчеризации, к которым у вас есть доступ. Выберите те, которые нужны для вашего бота.

### <a name="test-your-bot-locally"></a>Локальное тестирование бота

На этом этапе бот должен работать точно так же, как со старым файлом .bot. Убедитесь, что с новым файлом .bot он работает правильно.

### <a name="publish-your-bot-to-azure"></a>Публикация бота в Azure

<!-- TODO: re-encrypt your .bot file? -->

Опубликуйте бот из локальной среды в Azure. Этот процесс может занять некоторое время.

```cmd
az bot publish --name <bot-resource-name> --proj-file "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

<!-- Question: What should --proj-file be for a Node project? -->

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --name | Имя ресурса для бота в Azure. |
| --proj-file | Имя начального файла (без расширения .csproj) в публикуемом проекте. Например:  EnterpriseBot. |
| --resource-group | Имя группы ресурсов. |
| --code-dir | Каталог, из которого следует передать код бота. |

Когда этот процесс завершится, вы увидите сообщение "Deployment successful!" (Развертывание успешно выполнено). Теперь ваш бот развернут в Azure.

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

Очистите параметр ключа шифрования.

1. Войдите на [портал Azure](http://portal.azure.com/).
1. Откройте ресурс Web App Bot (Бот веб-приложения) для своего бота.
1. Откройте **параметры приложения** для бота.
1. В окне **Параметры приложения** прокрутите страницу вниз до раздела **Параметры приложения**.
1. Найдите параметр **botFileSecret** и удалите его.

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

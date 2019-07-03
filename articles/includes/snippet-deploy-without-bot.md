---
ms.openlocfilehash: 117f95799df0abbe957000d4979b10f05baf262c
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405536"
---
До начала развертывания убедитесь, что у вас установлена последняя версия [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) и [.Net CLI](https://dotnet.microsoft.com/download). Если у вас нет .Net CLI, установите программу с помощью среды выполнения .Net Core по приведенной выше ссылке. 

### <a name="login-to-azure-cli-and-set-your-subscription"></a>Вход в Azure CLI и настройка подписки.
Итак, вы уже создали бот и протестировали его локально, а теперь хотите развернуть его в Azure. Откройте командную строку, чтобы войти на портал Azure.

```cmd
az login
```
### <a name="set-the-subscription"></a>Настройка подписки

Укажите подписку, которая будет использоваться по умолчанию.

```cmd
az account set --subscription "<azure-subscription>"
```

Если вы не уверены, какую подписку выбрать для развертывания бота, просмотрите список подписок в учетной записи с помощью команды `az account list`.

Перейдите в папку бота.
`cd <local-bot-folder>`

### <a name="create-a-web-app-bot-in-azure"></a>Создание бота веб-приложения в Azure 

Если у вас еще нет группы ресурсов для публикации бота, создайте ее.

```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| Параметр     | ОПИСАНИЕ |
|:-----------|:---|
| name     | Уникальное имя группы ресурсов. НЕ ИСПОЛЬЗУЙТЕ пробелы и подчеркивания в этом имени. |
| location | Географическое расположение, в котором будут создана группа ресурсов. Например, `eastus`, `westus`, `westus2` и т. д. Используйте `az account list-locations` для получения списка расположений. |

Затем создайте ресурс бота, в котором вы будете публиковать этот бот. Это подготовит в Azure необходимые ресурсы и создаст приложение бота, вместо которого вы запишете локальную версию бота. 

Прежде чем продолжить работу, ознакомьтесь с описанными ниже инструкциями, выбрав нужный раздел в зависимости от типа учетной записи электронной почты, с которой вы входите в Azure.

#### <a name="msa-email-account"></a>Учетная запись электронной почты MSA
Если вы используете учетную запись электронной почты MSA, вам потребуется создать идентификатор и пароль приложения на портале регистрации приложений для использования в команде `az bot create`.
1. Перейдите на [**портал регистрации приложений**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade).
1. Щелкните **Add an app** (Добавить приложение), чтобы зарегистрировать приложение, затем создайте **Application Id** (Идентификатор приложения) и щелкните **Generate New Password** (Создать пароль). Если у вас уже есть приложение и пароль для него, но вы не помните этот пароль, следует создать новый пароль в разделе секретов приложения.
1. Сохраните идентификатор приложения и созданный пароль, чтобы применить их в команде `az bot create`.  

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name> --appid "<application-id>" --password "<application-password>" --verbose
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| name | Уникальное имя, которое используется для развертывания бота в Azure. Это имя может совпадать с именем локального бота. НЕ ИСПОЛЬЗУЙТЕ пробелы и подчеркивания в этом имени. |
| location | Географическое расположение для создания ресурсов службы бота. Например, `eastus`, `westus`, `westus2` и т. д. |
| resource-group | Имя группы ресурсов, в которой будет создан бот. Вы можете настроить расположение по умолчанию с помощью `az configure --defaults group=<name>`. |
| appid | Идентификатор учетной записи Майкрософт (MSA), который будет использоваться с этим ботом. |
| password | Пароль учетной записи Майкрософт (MSA) для бота. |

#### <a name="business-or-school-account"></a>Рабочая или учебная учетная запись

```cmd
az bot create --kind webapp --name <bot-name-in-azure> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```
| Параметр | ОПИСАНИЕ |
|:---|:---|
| name | Уникальное имя, которое используется для развертывания бота в Azure. Это имя может совпадать с именем локального бота. НЕ ИСПОЛЬЗУЙТЕ пробелы и подчеркивания в этом имени. |
| location | Географическое расположение для создания ресурсов службы бота. Например, `eastus`, `westus`, `westus2` и т. д. |
| lang | Язык, используемый для создания бота: `Csharp` или `Node` (по умолчанию `Csharp`). |
| resource-group | Имя группы ресурсов, в которой будет создан бот. Вы можете настроить расположение по умолчанию с помощью `az configure --defaults group=<name>`. |

#### <a name="update-appsettingsjson-or-env-file"></a>Обновление локальных файлов appsettings.json или .env
После создания бота вы увидите следующие сведения в окне консоли: 

```JSON
{
  "appId": "as234-345b-4def-9047-a8a44b4s",
  "appPassword": "34$#w%^$%23@334343",
  "endpoint": "https://mybot.azurewebsites.net/api/messages",
  "id": "mybot",
  "name": "mybot",
  "resourceGroup": "botresourcegroup",
  "serviceName": "mybot",
  "subscriptionId": "234532-8720-5632-a3e2-a1qw234",
  "tenantId": "32f955bf-33f1-43af-3ab-23d009defs47",
  "type": "abs"
}
```

Скопируйте значения `appId` и `appPassword` и вставьте их в файл appsettings.json или .env. Например:

```JSON
{
  MicrosoftAppId: "as234-345b-4def-9047-a8a44b4s",
  MicrosoftAppPassword: "34$#w%^$%23@334343"
}
```
Обратите внимание, что из файла appsettings.json или .env не нужно удалять дополнительные ключи для других служб, которые вы подготовили для бота.

Сохраните файл.

Далее выполните описанные ниже действия в зависимости от языка программирования (**C#** или **JS**), который вы выбрали для создания бота.

**Бот на C#** . 

Откройте командную строку и перейдите к папке проекта. Выполните следующую команду из командной строки.

| Задача | Команда |
|:-----|:--------|
| 1. Восстановление зависимостей проектов | `dotnet restore`|
| 2. Сборка проекта     | `dotnet build` |
| 3. Упаковка файлов проекта | С помощью любой служебной программы упакуйте файлы проекта. Перейдите в папку с файлом .csproj и выберите все файлы и папки на этом уровне, чтобы создать сжатую папку. |
| 4. Установка параметров развертывания для сборки | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|
| 5. Установка аргументов генератора скриптов | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_SCRIPT_GENERATOR_ARGS="--aspNetCore mybot.csproj"`|

**Бот на JS**.
1. Скачайте файл web.config [отсюда](https://github.com/projectkudu/kudu/wiki/Using-a-custom-web.config-for-Node-apps) и сохраните его в папку проекта. 
1. Откройте файл в редакторе и замените все вхождения server.js на index.js. 
1. Сохраните файл.

Откройте командную строку и перейдите к папке проекта. Выполните следующую команду из командной строки.

| Задача | Команда |
|:-----|:--------|
| 1. Установка модулей Node | `npm install` |
| 2. Упаковка файлов проекта | С помощью любой служебной программы упакуйте файлы проекта. Перейдите в папку с файлом .csproj и выберите все файлы и папки на этом уровне, чтобы создать сжатую папку. |
| 3. Установка параметров развертывания для сборки | `az webapp config appsettings set --resource-group <resource-group-name> --name <bot-name> --settings SCM_DO_BUILD_DEPLOYMENT=false`|

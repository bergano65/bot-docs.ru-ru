---
title: Развертывание бота | Документация Майкрософт
description: Развертывание бота в облаке Azure.
keywords: deploy bot, azure deploy bot, publish bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.subservice: abs
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ed7c9d7a883a1d1807237b636bbb59d25df60e08
ms.sourcegitcommit: f3fda6791f48ab178721b72d4f4a77c373573e38
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/31/2019
ms.locfileid: "68671385"
---
# <a name="deploy-your-bot"></a>Развертывание бота

[!INCLUDE [applies-to](./includes/applies-to.md)]

В этой статье описано, как развернуть бота в Azure. Эту статью полезно изучить до выполнения описанных действий, чтобы ознакомиться со всеми процессами, связанными с развертыванием бота.

## <a name="prerequisites"></a>Предварительные требования
- Если у вас еще нет подписки Azure, создайте [учетную запись](https://azure.microsoft.com/free/), прежде чем начинать работу.
- Бот на C#, JavaScript или TypeScript, который вы разработали на локальном компьютере.
- Последняя версия [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest).
- Знакомство с [Azure CLI и шаблонами ARM](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview).

## <a name="1-prepare-for-deployment"></a>1. Подготовка к развертыванию
Если бот создается на основе шаблонов Visual Studio или Yeoman, его исходный код содержит папку `deploymentTemplates` с шаблонами ARM. В описанном здесь процессе развертывания используется шаблон ARM для подготовки необходимых для бота ресурсов Azure с помощью Azure CLI. 

> [!IMPORTANT]
> С появлением пакета SDK Bot Framework 4.3 _не рекомендуется_ использовать файл .bot для управления ресурсами. Вместо него следует использовать файл appsettings.json или .env. Сведения о переносе параметров из файла .bot в файл appsettings.json или .env см в статье [об управлении ресурсами бота](v4sdk/bot-file-basics.md).

### <a name="login-to-azure"></a>Вход в Azure

Итак, вы уже создали бот и протестировали его локально, а теперь хотите развернуть его в Azure. Откройте командную строку, чтобы войти на портал Azure.

```cmd
az login
```
Она откроет окно браузера с интерфейсом для входа.

### <a name="set-the-subscription"></a>Настройка подписки
Укажите подписку, которая будет использоваться по умолчанию.

```cmd
az account set --subscription "<azure-subscription>"
```

Если вы не уверены, какую подписку выбрать для развертывания бота, просмотрите список подписок в учетной записи с помощью команды `az account list`. Перейдите в папку бота.

### <a name="create-an-app-registration"></a>Регистрация приложения
Регистрация приложения означает, что вы сможете использовать Azure AD для аутентификации пользователей и для запроса доступа к ресурсам пользователей. Боту нужно зарегистрированное в Azure приложение, которое предоставит боту доступ к службе Bot Framework для отправки и получения сообщений с проверкой подлинности. Чтобы зарегистрировать приложение с помощью Azure CLI, выполните следующую команду:

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| Параметр   | ОПИСАНИЕ |
|:---------|:------------|
| display-name | Отображаемое имя приложения. |
| password | Пароль приложения, также называемый "секрет клиента". Пароль должен содержать не менее 16 символов с минимум одной буквой в верхнем или нижнем регистре и минимум одним специальным символом.|
| available-to-other-tenants| Приложение может использоваться из любого клиента Azure AD. Наличие `true` позволяет боту работать с каналами службы Azure Bot.|

Приведенная выше команда выводит код JSON с ключом `appId`. Сохраните значение этого ключа для развертывания с помощью ARM, где этот ключ нужно указать в параметре `appId`. Предоставленный пароль будет использоваться в параметре `appSecret`.

Вы можете развернуть бота в новой группе ресурсов или использовать существующую. Выберите любой вариант, который вам подходит.

# <a name="deploy-via-arm-template-with-new-resource-grouptabnewrg"></a>[Развертывание с помощью шаблона ARM (в **новой** группе ресурсов)](#tab/newrg)

### <a name="create-azure-resources"></a>Создание ресурсов Azure

Вы создадите новую группу ресурсов в Azure, а затем с помощью шаблона ARM создадите указанные в нем ресурсы. В нашем примере это план Службы приложений, веб-приложение и регистрация каналов бота.

```cmd
az deployment create --name "<name-of-deployment>" --template-file "template-with-new-rg.json" --location "location-name" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" botSku=F0 newAppServicePlanName="<name-of-app-service-plan>" newWebAppName="<name-of-web-app>" groupName="<new-group-name>" groupLocation="<location>" newAppServicePlanLocation="<location>"
```

| Параметр   | ОПИСАНИЕ |
|:---------|:------------|
| name | Понятное имя развертывания. |
| template-file | Путь к шаблону ARM. Вы можете использовать файл `template-with-new-rg.json` из папки проекта `deploymentTemplates`. |
| location |Расположение. Значения из `az account list-locations`. Расположение по умолчанию можно настроить с помощью `az configure --defaults location=<location>`. |
| parameters | Укажите значения параметров развертывания. Значение `appId`, полученное при выполнении команды `az ad app create`. `appSecret` — это пароль, который вы ввели на предыдущем шаге. Параметр `botId` должен быть глобально уникальным. Он используется как неизменяемый идентификатор бота. Он также используется для настройки отображаемого имени бота, которое допускает изменения. `botSku` обозначает ценовую категорию — F0 (Бесплатный) или S1 (Стандартный). `newAppServicePlanName` — имя плана Службы приложений. `newWebAppName` — имя веб-приложения, которое вы создаете. `groupName` — имя группы ресурсов Azure, которую вы создаете. `groupLocation` — расположение группы ресурсов Azure. `newAppServicePlanLocation` — расположение плана Службы приложений. |

# <a name="deploy-via-arm-template-with-existing--resource-grouptaberg"></a>[Развертывание с помощью шаблона ARM (в **существующей** группе ресурсов)](#tab/erg)

### <a name="create-azure-resources"></a>Создание ресурсов Azure

Если вы используете существующую группу ресурсов, можно выбрать существующий план Службы приложений или создать новый. Ниже описаны процедуры для обоих вариантов. 

**Вариант 1. Существующий план Службы приложений** 

В этом варианте мы используем существующий план Службы приложений, но создаем новое веб-приложение и новую регистрацию каналов бота. 

_Примечание. Параметр botId должен быть глобально уникальным. Он используется как неизменяемый идентификатор бота. Он также используется для настройки отображаемого имени бота (displayName), которое допускает изменения._

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" existingAppServicePlan="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

**Вариант 2. Новый план Службы приложений** 

В этом варианте мы создаем план Службы приложений, веб-приложение и регистрацию каналов бота. 

```cmd
az group deployment create --name "<name-of-deployment>" --resource-group "<name-of-resource-group>" --template-file "template-with-preexisting-rg.json" --parameters appId="<msa-app-guid>" appSecret="<msa-app-password>" botId="<id-or-name-of-bot>" newWebAppName="<name-of-web-app>" newAppServicePlanName="<name-of-app-service-plan>" appServicePlanLocation="<location>"
```

| Параметр   | ОПИСАНИЕ |
|:---------|:------------|
| name | Понятное имя развертывания. |
| resource-group | Имя группы ресурсов Azure. |
| template-file | Путь к шаблону ARM. Вы можете использовать файл `template-with-preexisting-rg.json` из папки проекта `deploymentTemplates`. |
| location |Расположение. Значения из `az account list-locations`. Расположение по умолчанию можно настроить с помощью `az configure --defaults location=<location>`. |
| parameters | Укажите значения параметров развертывания. Значение `appId`, полученное при выполнении команды `az ad app create`. `appSecret` — это пароль, который вы ввели на предыдущем шаге. Параметр `botId` должен быть глобально уникальным. Он используется как неизменяемый идентификатор бота. Он также используется для настройки отображаемого имени бота, которое допускает изменения. `newWebAppName` — имя веб-приложения, которое вы создаете. `newAppServicePlanName` — имя плана Службы приложений. `newAppServicePlanLocation` — расположение плана Службы приложений. |

---

### <a name="retrieve-or-create-necessary-iiskudu-files"></a>Получение или создание файлов, необходимых для IIS либо Kudu

**Для ботов на C#**

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

Необходимо указать путь к CSPROJ-файлу относительно папки --code-dir. Для этого можно применить аргумент --proj-file-path. Эта команда разрешит аргументы --code-dir и --proj-file-path в значение ./MyBot.csproj.

**Для ботов на JavaScript**

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

Эта команда получает файл web.config, который необходим для работы приложений Node.js со службами IIS в Службе приложений Azure. Убедитесь, что файл web.config сохранен в корневой каталог бота.

**Для ботов на TypeScript**

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

Эта команда работает так же, как предложенный выше код JavaScript, но применяется для бота на Typescript.

### <a name="zip-up-the-code-directory-manually"></a>Архивация каталога кода вручную

Если для развертывания кода бота используются ненастроенные интерфейсы [API развертывания ZIP-файлов](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url), Web App или Kudu демонстрирует следующее поведение.

_Kudu по умолчанию предполагает, что развертывание из ZIP-файлов готово к выполнению и не требует дополнительных шагов сборки, таких как npm install или dotnet restore/dotnet publish._

Поэтому важно включить весь код сборки и все необходимые зависимости в ZIP-файл, предназначенный для развертывания веб-приложения, иначе бот не будет работать должным образом.

> [!IMPORTANT]
> Прежде чем запаковать файлы проекта, убедитесь, что вы зашли _в_ нужную папку. 
> - Для ботов на C# это папка, в которой расположен CSPROJ-файл. 
> - Для ботов на JS это папка, в которой расположен файл app.js или index.js. 
>
> Выберите все файлы и заархивируйте их, а затем выполните команду, **оставаясь в той же папке**.
>
> Если расположение корневой папки выбрано неверно, **бот не сможет запуститься на портале Azure**.

## <a name="2-deploy-code-to-azure"></a>2. Развертывание кода в Azure
Теперь мы готовы развернуть код в виде веб-приложения Azure. Запустите следующую команду из командной строки, чтобы выполнить развертывание с помощью принудительного развертывания в Kudu из ZIP-файла веб-приложения.

```cmd
az webapp deployment source config-zip --resource-group "<new-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| Параметр   | ОПИСАНИЕ |
|:---------|:------------|
| resource-group | Имя созданной ранее группы ресурсов Azure. |
| name | Имя веб-приложения, которое вы использовали ранее. |
| src  | Путь к созданному ранее ZIP-файлу. |

## <a name="3-test-in-web-chat"></a>3. Тестирование в веб-чате

1. В браузере перейдите на [портал Azure](https://ms.portal.azure.com).
2. На панели слева щелкните **Группы ресурсов**.
3. На панели справа найдите свою группу.
4. Щелкните имя группы.
5. Щелкните ссылку регистрации канала бота.
6. В колонке *Регистрация канала бота* щелкните **Тестировать в веб-чате**.
Или на панели справа щелкните поле "Тест".

См. также о регистрации канала в руководстве по [регистрации бота с помощью службы Azure Bot](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration?view=azure-bot-service-3.0).

> [!NOTE]
> Колонка — это область, в которой отображаются функции службы или элементы навигации.

## <a name="additional-information"></a>Дополнительная информация
Развертывание бота в Azure подразумевает оплату за используемые службы. Руководство по [управлению счетами и расходами](https://docs.microsoft.com/azure/billing/) поможет вам понять, как расшифровывать счета Azure, отслеживать использование и расходы, а также управлять учетными записями и подписками.

## <a name="next-steps"></a>Дополнительная информация
> [!div class="nextstepaction"]
> [Настройка непрерывного развертывания](bot-service-build-continuous-deployment.md)

---
title: Развертывание бота | Документация Майкрософт
description: Развертывание бота в облаке Azure
keywords: deploy bot, azure deploy bot, publish bot
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 08/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d44b1339b367c7243cfb6d2311a553becb6245ff
ms.sourcegitcommit: c200cc2db62dbb46c2a089fb76017cc55bdf26b0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/27/2019
ms.locfileid: "70037507"
---
# <a name="deploy-your-bot"></a>Развертывание бота

[!INCLUDE [applies-to](./includes/applies-to.md)]

В этой статье описано, как развернуть простого бота в Azure. Мы объясним, как подготовить бота к развертыванию, развернуть его в Azure и протестировать в Web Chat. Эту статью полезно изучить до выполнения описанных действий, чтобы ознакомиться со всеми процессами, связанными с развертыванием бота.

<!-- create your Azure Application, 2) prepare your source code for deployment, and 3) deploy your code to your Azure Application.  -->

## <a name="prerequisites"></a>Предварительные требования
- подписка на [Microsoft Azure](https://azure.microsoft.com/free/).
- Бот на C#, JavaScript или TypeScript, который вы разработали на локальном компьютере.
- Последняя версия [Azure CLI](https://docs.microsoft.com/cli/azure/?view=azure-cli-latest).
- Навыки работы с [Azure CLI и шаблонами ARM](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview).

## <a name="prepare-for-deployment"></a>Подготовка к развертыванию
Если бот создается на основе [шаблона Visual Studio](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0) или [шаблона Yeoman](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0), его исходный код содержит папку `deploymentTemplates` с шаблонами ARM. В описанном здесь процессе развертывания используется шаблон ARM для подготовки необходимых для бота ресурсов Azure с помощью Azure CLI. 

> [!NOTE]
> С появлением пакета SDK Bot Framework 4.3 _не рекомендуется_ использовать файл .bot для управления ресурсами. Вместо него следует использовать файл appsettings.json или .env. Сведения о переносе параметров из файла .bot в файл appsettings.json или .env см в статье [об управлении ресурсами бота](v4sdk/bot-file-basics.md).

### <a name="1-login-to-azure"></a>1. Вход в Azure

Итак, вы уже создали бот и протестировали его локально, а теперь хотите развернуть его в Azure. Откройте командную строку, чтобы войти на портал Azure.

```cmd
az login
```
Она откроет окно браузера с интерфейсом для входа.

> [!NOTE]
> При развертывании бота в облаке, отличном от Azure, например US Gov, необходимо выполнить `az cloud set --name <name-of-cloud>` перед `az login`, где &lt;name-of-cloud> обозначает имя зарегистрированного облака, например `AzureUSGovernment`. Если вы хотите вернуться в общедоступное облако, можно запустить `az cloud set --name AzureCloud`. 

### <a name="2-set-the-subscription"></a>2. Настройка подписки

Укажите подписку, которая будет использоваться по умолчанию.

```cmd
az account set --subscription "<azure-subscription>"
```

Если вы не уверены, какую подписку выбрать для развертывания бота, просмотрите список подписок в учетной записи с помощью команды `az account list`. Перейдите в папку бота.

### <a name="3-create-an-app-registration"></a>3. Регистрация приложения

Регистрация приложения означает, что вы сможете использовать Azure AD для аутентификации пользователей и для запроса доступа к ресурсам пользователей. Боту нужно зарегистрированное в Azure приложение, которое предоставит боту доступ к службе Bot Framework для отправки и получения сообщений с проверкой подлинности. Чтобы зарегистрировать приложение с помощью Azure CLI, выполните следующую команду:

```cmd
az ad app create --display-name "displayName" --password "AtLeastSixteenCharacters_0" --available-to-other-tenants
```

| Параметр   | ОПИСАНИЕ |
|:---------|:------------|
| display-name | Отображаемое имя приложения. |
| password | Пароль приложения, также называемый "секрет клиента". Пароль должен содержать не менее 16 символов, среди которых есть хотя бы по одной букве в верхнем и нижнем регистрах и хотя бы один специальный символ.|
| available-to-other-tenants| Приложение может использоваться из любого клиента Azure AD. Наличие `true` позволяет боту работать с каналами службы Azure Bot.|

Приведенная выше команда выводит код JSON с ключом `appId`. Сохраните значение этого ключа для развертывания с помощью ARM, где этот ключ нужно указать в параметре `appId`. Предоставленный пароль будет использоваться в параметре `appSecret`.

> [!NOTE] 
> Если вы хотите использовать существующую регистрацию приложения, можно использовать эту команду:
> ``` cmd
> az bot create --kind webapp --resource-group "<name-of-resource-group>" --name "<name-of-web-app>" --appid "<existing-app-id>" --password "<existing-app-password>" --lang <Javascript|Csharp>
> ```

### <a name="4-deploy-via-arm-template"></a>4. Развертывание с использованием шаблона ARM
Вы можете развернуть бот в новой группе ресурсов или использовать имеющуюся. Выберите любой вариант, который вам подходит. 
<!--
## [Deploy via ARM template (with **new**  Resource Group)](#tab/nerg)
-->
#### <a name="deploy-via-arm-template-with-new-resource-group"></a>**Развертывание с помощью шаблона ARM (в **новой** группе ресурсов)**
<!-- ##### Create Azure resources -->
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

#### <a name="deploy-via-arm-template-with-existing--resource-group"></a>**Развертывание с помощью шаблона ARM (в **существующей** группе ресурсов)**
<!--
## [Deploy via ARM template (with **existing**  Resource Group)](#tab/erg)
##### Create Azure resources
-->
Если вы используете существующую группу ресурсов, можно выбрать существующий план Службы приложений или создать новый. Ниже описаны процедуры для обоих вариантов. 

**Вариант 1. Существующий план Службы приложений** 

В этом варианте мы используем существующий план Службы приложений, но создаем новое веб-приложение и новую регистрацию каналов бота. 

> [!NOTE]
> Параметр botId должен быть глобально уникальным. Он используется как неизменяемый идентификатор бота. Он также используется для настройки отображаемого имени бота (displayName), которое допускает изменения.

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

### <a name="5-prepare-your-code-for-deployment"></a>5. Подготовка кода к развертыванию
#### <a name="51-retrieve-or-create-necessary-iiskudu-files"></a>5.1 Получение или создание файлов, необходимых для IIS либо Kudu

<!-- **C# bots** -->
##### <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
az bot prepare-deploy --lang Csharp --code-dir "." --proj-file-path "MyBot.csproj"
```

Необходимо указать путь к CSPROJ-файлу относительно папки --code-dir. Для этого можно применить аргумент --proj-file-path. Эта команда разрешит аргументы --code-dir и --proj-file-path в значение ./MyBot.csproj.

<!-- **JavaScript bots** -->
##### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
az bot prepare-deploy --code-dir "." --lang Javascript
```

Эта команда получает файл web.config, который необходим для работы приложений Node.js со службами IIS в Службе приложений Azure. Убедитесь, что файл web.config сохранен в корневой каталог бота.

<!-- **TypeScript bots** -->
##### <a name="typescripttabtypescript"></a>[TypeScript](#tab/typescript)

```cmd
az bot prepare-deploy --code-dir "." --lang Typescript
```

Эта команда работает так же, как предложенный выше код JavaScript, но применяется для бота на Typescript.

---

> [!NOTE]
> После выполнения указанной выше команды вы увидите файл с расширением `.deployment` в папке проекта бота.

#### <a name="52-zip-up-the-code-directory-manually"></a>5.2 Архивация каталога кода вручную

Если для развертывания кода бота используются ненастроенные интерфейсы [API развертывания ZIP-файлов](https://github.com/projectkudu/kudu/wiki/Deploying-from-a-zip-file-or-url), Web App или Kudu демонстрирует следующее поведение.

_Kudu по умолчанию предполагает, что развертывание из ZIP-файлов готово к выполнению и не требует дополнительных шагов сборки, таких как npm install или dotnet restore/dotnet publish._

Поэтому важно включить весь код сборки и все необходимые зависимости в ZIP-файл, предназначенный для развертывания веб-приложения, иначе бот не будет работать должным образом.

> [!IMPORTANT]
> Прежде чем запаковать файлы проекта, убедитесь, что вы зашли _в_ нужную папку. 
> - Для ботов на C# это папка, в которой расположен CSPROJ-файл. 
> - Для ботов на JS это папка, в которой расположен файл app.js или index.js. 
>
>**В папке проекта**, выберите все файлы и заархивируйте их, а затем выполните команду, оставаясь в той же папке. 
>
> Если расположение корневой папки выбрано неверно, **бот не сможет запуститься на портале Azure**.

## <a name="deploy-code-to-azure"></a>Развертывание кода в Azure
Теперь мы готовы развернуть код в виде веб-приложения Azure. Запустите следующую команду из командной строки, чтобы выполнить развертывание с помощью принудительного развертывания в Kudu из ZIP-файла веб-приложения.

```cmd
az webapp deployment source config-zip --resource-group "<new-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| Параметр   | ОПИСАНИЕ |
|:---------|:------------|
| resource-group | Имя созданной ранее группы ресурсов Azure. |
| name | Имя веб-приложения, которое вы использовали ранее. |
| src  | Путь к созданному ранее ZIP-файлу. |

## <a name="test-in-web-chat"></a>Тестирование в веб-чате

1. В браузере перейдите на [портал Azure](https://ms.portal.azure.com).
2. На панели слева щелкните **Группы ресурсов**.
3. На панели справа найдите свою группу.
4. Щелкните имя группы.
5. Щелкните ссылку регистрации канала бота.
6. В колонке *Регистрация канала бота* щелкните **Тестировать в веб-чате**.
Или на панели справа щелкните поле "Тест".

См. также о регистрации канала в руководстве по [регистрации бота с помощью службы Azure Bot](https://docs.microsoft.com/azure/bot-service/bot-service-quickstart-registration?view=azure-bot-service-3.0).

> [!NOTE]
> Колонка — это область, в которой отображаются функции службы или элементы навигации.

## <a name="additional-information"></a>Дополнительная информация
Развертывание бота в Azure подразумевает оплату за используемые службы. Руководство по [управлению счетами и расходами](https://docs.microsoft.com/azure/billing/) поможет вам понять, как расшифровывать счета Azure, отслеживать использование и расходы, а также управлять учетными записями и подписками.

## <a name="next-steps"></a>Дополнительная информация
> [!div class="nextstepaction"]
> [Set up continuous deployment](bot-service-build-continuous-deployment.md) (Настройка непрерывного развертывания)

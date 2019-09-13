---
ms.openlocfilehash: 029275eae6f7b0b4448613ede898575eb491a56f
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386046"
---
Если вы используете существующую группу ресурсов, можно выбрать существующий план Службы приложений или создать новый. Ниже описаны процедуры для обоих вариантов. 

**Вариант 1. Существующий план Службы приложений** 

В этом варианте мы используем существующий план Службы приложений, но при этом создаем новое веб-приложение и новую регистрацию каналов бота. 

> [!NOTE]
> Эта команда определяет идентификатор и отображаемое имя бота. Параметр `botId` должен быть глобально уникальным. Он используется как неизменяемый идентификатор бота. Отображаемое имя бота является изменяемым.

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

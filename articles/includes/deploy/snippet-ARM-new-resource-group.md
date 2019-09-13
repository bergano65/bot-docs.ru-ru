---
ms.openlocfilehash: fd279af81e0c94ac22f54429cb8ae330e5d60661
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386039"
---
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

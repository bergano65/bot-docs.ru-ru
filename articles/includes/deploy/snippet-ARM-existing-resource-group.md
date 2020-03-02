---
ms.openlocfilehash: 2559f424e9f50a760837494f87e4f33731ffb358
ms.sourcegitcommit: 4ddee4f90a07813ce570fdd04c8c354b048e22f3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/20/2020
ms.locfileid: "77479312"
---
На этом шаге вы создадите службу приложения для бота, то есть определите этап развертывания для бота. Если вы используете существующую группу ресурсов, можно выбрать существующий план службы приложений или создать новый. Ниже описаны процедуры для обоих вариантов.

Из выходных данных в формате JSON скопируйте значение поля **id**, которое на следующем шаге будет использоваться в качестве значения для **идентификатора подписки регистрации**.

> [!NOTE]
> Этот шаг может занять несколько минут.

**Вариант 1. Существующий план Службы приложений**

В этом варианте мы используем существующий план Службы приложений, но при этом создаем новое веб-приложение и новую регистрацию каналов бота.

> [!NOTE]
> Эта команда определяет идентификатор и отображаемое имя бота. Параметр `botId` должен быть глобально уникальным. Он используется как неизменяемый идентификатор бота. Отображаемое имя бота является изменяемым.

```cmd
az group deployment create --resource-group "<name-of-resource-group>" --template-file "<path-to-template-with-preexisting-rg.json>" --parameters appId="<app-id-from-previous-step>" appSecret="<password-from-previous-step>" botId="<id or bot-app-service-name>" newWebAppName="<bot-app-service-name>" existingAppServicePlan="<name-of-app-service-plan>" appServicePlanLocation="<region-location-name>" --name "<bot-app-service-name>"
```

**Вариант 2. Новый план Службы приложений**

В этом варианте мы создаем план Службы приложений, веб-приложение и регистрацию каналов бота.

```cmd
az group deployment create --resource-group "<name-of-resource-group>" --template-file "<path-to-template-with-preexisting-rg.json>" --parameters appId="<app-id-from-previous-step>" appSecret="<password-from-previous-step>" botId="<id or bot-app-service-name>" newWebAppName="<bot-app-service-name>" newAppServicePlanName="<name-of-app-service-plan>" appServicePlanLocation="<region-location-name>" --name "<bot-app-service-name>"
```

| Параметр   | Описание |
|:---------|:------------|
| name | Отображаемое имя, которое используется для регистрации каналов бота. По умолчанию здесь используется значение параметра `botId`.|
| resource-group | Имя группы ресурсов Azure. |
| template-file | Путь к шаблону ARM. Обычно файл `template-with-preexisting-rg.json` размещается в папке `deploymentTemplates` проекта. Это путь к существующему файлу шаблона. Можно указать абсолютный или относительный путь к текущему каталогу. Все шаблоны бота создают файлы шаблонов ARM.|
| location |Расположение. Значения из `az account list-locations`. Расположение по умолчанию можно настроить с помощью `az configure --defaults location=<location>`. |
| параметры | Параметры развертывания в формате списка пар "ключ — значение". Введите следующие значения параметров:

- `appId` — значение *идентификатора приложения*, созданное на предыдущем шаге.
- `appSecret` — пароль, который вы ввели на предыдущем шаге.
- `botId` — имя создаваемого ресурса регистрации канала бота. Оно должно быть глобально уникальным. Оно используется как неизменяемый идентификатор бота и сохраняется в качестве отображаемого имени бота, но это значение вы можете изменить.
- `newWebAppName` — имя службы приложений для бота.
- `newAppServicePlanName` — имя создаваемого ресурса плана службы приложений.
- `newAppServicePlanLocation` — расположение плана службы приложений.
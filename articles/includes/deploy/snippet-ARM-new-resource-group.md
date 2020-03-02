---
ms.openlocfilehash: 4cee4c59fc7baa4d9aa18b573f7aebce8e99d1bb
ms.sourcegitcommit: 4ddee4f90a07813ce570fdd04c8c354b048e22f3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/20/2020
ms.locfileid: "77479308"
---
На этом шаге вы создадите службу приложения для бота, то есть определите этап развертывания для бота. Вы примените для этого шаблон ARM, создадите план обслуживания и группу ресурсов.

Из выходных данных в формате JSON скопируйте числовое значение поля **id**, которое на следующем шаге будет использоваться в качестве значения для **идентификатора подписки регистрации**.

> [!NOTE]
> Этот шаг может занять несколько минут.

```cmd
az deployment create --template-file "<path-to-template-with-new-rg.json" --location <region-location-name> --parameters appId="<app-id-from-previous-step>" appSecret="<password-from-previous-step>" botId="<id or bot-app-service-name>" botSku=F0 newAppServicePlanName="<new-service-plan-name>" newWebAppName="<bot-app-service-name>" groupName="<new-group-name>" groupLocation="<region-location-name>" newAppServicePlanLocation="<region-location-name>" --name "<bot-app-service-name>"
```

| Параметр   | Описание |
|:---------|:------------|
| name | Отображаемое имя, которое используется для регистрации каналов бота. По умолчанию здесь используется значение параметра `botId`.|
| template-file | Путь к шаблону ARM. Обычно файл `template-with-new-rg.json` размещается в папке `deploymentTemplates` проекта бота. Это путь к существующему файлу шаблона. Можно указать абсолютный или относительный путь к текущему каталогу. Все шаблоны бота создают файлы шаблонов ARM.|
| location |Расположение. Значения из `az account list-locations`. Расположение по умолчанию можно настроить с помощью `az configure --defaults location=<location>`. |
| параметры | Параметры развертывания в формате списка пар "ключ — значение". Введите следующие значения параметров:

- `appId` — значение *идентификатора приложения*, созданное на предыдущем шаге.
- `appSecret` — пароль, который вы ввели на предыдущем шаге.
- `botId` — имя создаваемого ресурса регистрации канала бота. Оно должно быть глобально уникальным. Оно используется как неизменяемый идентификатор бота и сохраняется в качестве отображаемого имени бота, но это значение вы можете изменить.
- `botSku` — ценовая категория, где возможны значения F0 (Бесплатный) или S1 (Стандартный).
- `newAppServicePlanName` — имя нового плана службы приложений.
- `newWebAppName` — имя службы приложений для бота.
- `groupName` — имя новой группы ресурсов.
- `groupLocation` — расположение группы ресурсов Azure.
- `newAppServicePlanLocation` — расположение плана службы приложений.
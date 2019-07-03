---
ms.openlocfilehash: 14d9632ad578014a36b5f13e6dee883e2a6e1722
ms.sourcegitcommit: a47183f5d1c2b2454c4a06c0f292d7c075612cdd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/19/2019
ms.locfileid: "67252652"
---
1. Перейдите на [**портал регистрации приложений**](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade).
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

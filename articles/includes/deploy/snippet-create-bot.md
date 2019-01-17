---
ms.openlocfilehash: f95c8a37b1207a26dab0a714b86412a9dba2dcb4
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360910"
---
```cmd
az bot create --kind webapp --name <bot-resource-name> --location <geographic-location> --version v4 --lang <language> --verbose --resource-group <resource-group-name>
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --name | Уникальное имя, которое используется для развертывания бота в Azure. Это имя может совпадать с именем локального бота. НЕ ИСПОЛЬЗУЙТЕ пробелы и подчеркивания в этом имени. |
| --location | Географическое расположение для создания ресурсов службы бота. Например, `eastus`, `westus`, `westus2` и т. д. |
| --lang | Язык, используемый для создания бота: `Csharp` или `Node` (по умолчанию `Csharp`). |
| --resource-group | Имя группы ресурсов, в которой будет создан бот. Вы можете настроить расположение по умолчанию с помощью `az configure --defaults group=<name>`. |
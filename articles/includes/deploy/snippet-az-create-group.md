---
ms.openlocfilehash: 53db401b00e53b964027f08c32ca7a38a4915f60
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035699"
---
```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --name | Уникальное имя группы ресурсов. НЕ ИСПОЛЬЗУЙТЕ пробелы и подчеркивания в этом имени. |
| --location | Географическое расположение, в котором будут создана группа ресурсов. Например, `eastus`, `westus`, `westus2` и т. д. Используйте `az account list-locations` для получения списка расположений. |
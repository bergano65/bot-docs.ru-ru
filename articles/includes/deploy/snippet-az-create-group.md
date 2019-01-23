---
ms.openlocfilehash: 53db401b00e53b964027f08c32ca7a38a4915f60
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360916"
---
```cmd
az group create --name <resource-group-name> --location <geographic-location> --verbose
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --name | Уникальное имя группы ресурсов. НЕ ИСПОЛЬЗУЙТЕ пробелы и подчеркивания в этом имени. |
| --location | Географическое расположение, в котором будут создана группа ресурсов. Например, `eastus`, `westus`, `westus2` и т. д. Используйте `az account list-locations` для получения списка расположений. |
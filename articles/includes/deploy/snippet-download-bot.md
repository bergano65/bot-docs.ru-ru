---
ms.openlocfilehash: 867e65b25878f810e3247eb3cace95f4d31e11db
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035730"
---
Используйте временный каталог вне каталога текущего проекта. 

Эта команда создает подкаталог в каталоге save-path, но сам этот каталог уже должен существовать.

```cmd
az bot download --name <bot-resource-name> --resource-group <resource-group-name> --save-path "<path>"
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --name | Имя бота в Azure. |
| --resource-group | Имя группы ресурсов, в которой размещен бот. |
| --save-path | Существующий каталог, в который будет скачиваться код бота. |
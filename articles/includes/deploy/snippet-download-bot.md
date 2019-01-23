---
ms.openlocfilehash: 867e65b25878f810e3247eb3cace95f4d31e11db
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360907"
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
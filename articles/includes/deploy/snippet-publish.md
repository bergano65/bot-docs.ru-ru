---
ms.openlocfilehash: bb7520e8e99ad6326d7d00d8190dae306bf11afa
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035753"
---
Опубликуйте бот из локальной среды в Azure. Этот процесс может занять некоторое время.

```cmd
az bot publish --name <bot-resource-name> --proj-file-path "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --name | Имя ресурса для бота в Azure. |
| --proj-file-path | Для C# введите имя начального файла (без расширения .csproj) в публикуемом проекте. Например, `EnterpriseBot`. Для Node.js используйте главную точку входа бота. Например, `index.js`. |
| --resource-group | Имя группы ресурсов. |
| --code-dir | Каталог, из которого следует передать код бота. |

Когда этот процесс завершится, вы увидите сообщение "Deployment successful!" (Развертывание успешно выполнено). Теперь ваш бот развернут в Azure.

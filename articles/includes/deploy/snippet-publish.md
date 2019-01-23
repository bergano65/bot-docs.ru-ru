---
ms.openlocfilehash: 88732d2d5490962d7a899d936767e7dd148e94c5
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360898"
---
Опубликуйте бот из локальной среды в Azure. Этот процесс может занять некоторое время.

```cmd
az bot publish --name <bot-resource-name> --proj-name "<project-file-name>" --resource-group <resource-group-name> --code-dir <directory-path> --verbose --version v4
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --name | Имя ресурса для бота в Azure. |
| --proj-name | Для C# введите имя начального файла (без расширения .csproj) в публикуемом проекте. Например, `EnterpriseBot`. Для Node.js используйте главную точку входа бота. Например, `index.js`. |
| --resource-group | Имя группы ресурсов. |
| --code-dir | Каталог, из которого следует передать код бота. |

Когда этот процесс завершится, вы увидите сообщение "Deployment successful!" (Развертывание успешно выполнено). Теперь ваш бот развернут в Azure.
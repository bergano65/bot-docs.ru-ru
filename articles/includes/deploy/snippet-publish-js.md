---
ms.openlocfilehash: 0891b9652154f8ed086cc45ce6018aa0be1a67b8
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905176"
---
Чтобы опубликовать локальный бот JavaScript снова в Azure, необходимо сначала вручную создать единый ZIP-файл, содержащий все файлы, использовавшиеся для локального создания и запуска бота. К ним относятся все библиотеки npm, скачанные в папку `node_modules`. При создании этого ZIP-файла _убедитесь, что используется тот корневой каталог, в котором находится файл index.js_.

После создания ZIP-файла со всем исходным кодом бота откройте окно командной строки и выполните команду _Az cli_. 

Этот процесс может занять некоторое время.

```cmd
az webapp deployment source config-zip --resource-group <resource-group-name> --name <bot-resource-name> --src <directory-path>
```

| Параметр | ОПИСАНИЕ |
|:---|:---|
| --resource-group | Имя группы ресурсов в Azure. |
| --name | Имя ресурса для бота в Azure. |
| --src | Полный путь к каталогу для отправки кода бота в виде ZIP-файла. Например `c:\my-local-repository\this-app-folder\my-zipped-code.zip` |

После успешного завершения ваш бот будет развернут в Azure.

---
ms.openlocfilehash: 0c28cf506f2701acfc705fdcc67ec6a1e7224ad1
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386067"
---
Теперь мы готовы развернуть код в виде веб-приложения Azure. Запустите следующую команду из командной строки, чтобы выполнить развертывание с помощью принудительного развертывания в Kudu из ZIP-файла веб-приложения.

```cmd
az webapp deployment source config-zip --resource-group "<resource-group-name>" --name "<name-of-web-app>" --src "code.zip" 
```

| Параметр   | ОПИСАНИЕ |
|:---------|:------------|
| resource-group | Имя группы ресурсов Azure, которая содержит бота. (Это будет группа ресурсов, которую вы использовали или создали при регистрации приложения для бота.) |
| name | Имя веб-приложения, которое вы использовали ранее. |
| src  | Путь к созданному ранее ZIP-файлу. |

---
ms.openlocfilehash: c664749bc6ad63d1b0f60e001b2603898e58a9ea
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386016"
---
Созданный и протестированный локально бот можно развернуть в Azure. Откройте командную строку, чтобы войти на портал Azure.

```cmd
az login
```
Она откроет окно браузера с интерфейсом для входа.

> [!NOTE]
> При развертывании бота в облаке, отличном от Azure, например US Gov, необходимо выполнить `az cloud set --name <name-of-cloud>` перед `az login`, где &lt;name-of-cloud> обозначает имя зарегистрированного облака, например `AzureUSGovernment`. Если вы хотите вернуться в общедоступное облако, можно запустить `az cloud set --name AzureCloud`. 

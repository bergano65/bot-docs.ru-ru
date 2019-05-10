---
ms.openlocfilehash: f8aad539a2d1e415833609f66cd5b398c88206f1
ms.sourcegitcommit: 4ff7a8772124a567f43e2c3e13aded368c4002e3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/03/2019
ms.locfileid: "65035774"
---
Откройте командную строку, чтобы войти на портал Azure.

```cmd
az login
```

Она откроет окно браузера с интерфейсом для входа.

### <a name="set-the-subscription"></a>Настройка подписки

Укажите подписку, которая будет использоваться по умолчанию.

```cmd
az account set --subscription "<azure-subscription>"
```

Если вы не уверены, какую подписку выбрать для развертывания бота, просмотрите список `subscriptions` для учетной записи с помощью команды `az account list`.

Перейдите в папку бота.

```cmd
cd <local-bot-folder>
```
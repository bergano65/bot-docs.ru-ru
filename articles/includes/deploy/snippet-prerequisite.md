---
ms.openlocfilehash: 266cdc2bbeeb140e4b601c1bf0fb3fa8eb085dda
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360923"
---
- Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу.
- Установите последнюю версию [средства Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
- Установите последнюю версию средства [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/MSBot).
- Установите последнюю версию [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Установленное и настроенное средство [ngrok](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-%28ngrok%29).
- Понимание структуры файла с расширением [.bot](~/v4sdk/bot-file-basics.md).

Для работы с msbot 4.3.2 и более поздними версиями вам понадобится Azure CLI версии 2.0.54 или более поздней. Если вы установили расширение botservice, удалите его с помощью следующей команды.

```cmd
az extension remove --name botservice
```
---
ms.openlocfilehash: db7c4b447c5a27fe2047cfa41e65fac0d3e3a86c
ms.sourcegitcommit: dd12ddf408c010182b09da88e2aac0de124cef22
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/05/2019
ms.locfileid: "70386009"
---
Если бот создается на основе [шаблона Visual Studio](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0) или [шаблона Yeoman](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0), его исходный код содержит папку `deploymentTemplates` с шаблонами ARM. В описанном здесь процессе развертывания используется один из шаблонов ARM для подготовки необходимых для бота ресурсов Azure с помощью Azure CLI. 

> [!NOTE]
> С появлением пакета SDK Bot Framework 4.3 _не рекомендуется_ использовать файл .bot. Вместо него следует использовать файл appsettings.json или .env. Сведения о переносе параметров из файла .bot в файл appsettings.json или .env см в статье [об управлении ресурсами бота](https://docs.microsoft.com/azure/bot-service/bot-file-basics?view=azure-bot-service-4.0).

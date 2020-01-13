---
ms.openlocfilehash: 3cb29b395019e720d5bc2ef7475c48f6c5757206
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491151"
---
Если бот создается на основе шаблона [Visual Studio](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0), [Yeoman](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0) или [Cookiecutter](https://docs.microsoft.com/azure/bot-service/python/bot-builder-python-quickstart?view=azure-bot-service-4.0), его исходный код содержит папку `deploymentTemplates` с шаблонами ARM. В описанном здесь процессе развертывания используется один из шаблонов ARM для подготовки необходимых для бота ресурсов Azure с помощью Azure CLI. 

> [!NOTE]
> С появлением пакета SDK Bot Framework 4.3 _не рекомендуется_ использовать файл .bot. Вместо него следует использовать файл appsettings.json или .env. Сведения о переносе параметров из файла .bot в файл appsettings.json или .env см в статье [об управлении ресурсами бота](https://docs.microsoft.com/azure/bot-service/bot-file-basics?view=azure-bot-service-4.0).

---
ms.openlocfilehash: 83ed1e7a28eb3cf417f5f404a1b33f0620e855ea
ms.sourcegitcommit: 4ddee4f90a07813ce570fdd04c8c354b048e22f3
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/20/2020
ms.locfileid: "77479306"
---
Если бот создается на основе шаблона [Visual Studio](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0), [Yeoman](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0) или [Cookiecutter](https://docs.microsoft.com/azure/bot-service/python/bot-builder-python-quickstart?view=azure-bot-service-4.0), его исходный код содержит папку `deploymentTemplates` с шаблонами ARM. В описанном здесь процессе развертывания используется один из шаблонов ARM для подготовки необходимых для бота ресурсов Azure с помощью Azure CLI.

> [!NOTE]
> С появлением пакета SDK Bot Framework 4.3 _не рекомендуется_ использовать файл .bot. Вместо него следует использовать файл appsettings.json или .env. Сведения о переносе параметров из файла .bot в файл appsettings.json или .env см в статье [об управлении ресурсами бота](https://docs.microsoft.com/azure/bot-service/bot-file-basics?view=azure-bot-service-4.0).

### <a name="bot-ready-to-deploy"></a>Бот, готовый к развертыванию

В этой статье предполагается, что у вас есть бот, готовый к развертыванию, и **путь** к связанному проекту. Вам потребуется путь для доступа к шаблонам развертывания, а также для создания *ZIP-файла* для развертывания.

Сведения о том, как создать простой бот Echo Bot, см. в кратком руководстве с примерами для[CSharp](~/dotnet/bot-builder-dotnet-sdk-quickstart.md), [JavaScript](~/javascript/bot-builder-javascript-quickstart.md) или [Python](~/python/bot-builder-python-quickstart.md).

Вы также можете использовать один из примеров, приведенных в соответствующем репозитории [Bot Framework](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md).
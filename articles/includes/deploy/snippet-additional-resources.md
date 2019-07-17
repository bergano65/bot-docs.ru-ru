---
ms.openlocfilehash: 2c06c67099f44fe1df2eb0099a514a697ef0d1c9
ms.sourcegitcommit: fa6e775dcf95a4253ad854796f5906f33af05a42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/16/2019
ms.locfileid: "68230820"
---
При развертывании бота на портале Azure обычно создаются следующие ресурсы.

| Ресурсы      | ОПИСАНИЕ |
|----------------|-------------|
| Бот веб-приложения | Бот в службе Azure Bot, который развернут в Службе приложений Azure.|
| [Служба приложений](https://docs.microsoft.com/azure/app-service/)| Позволяет создавать и размещать веб-приложения.|
| [План обслуживания приложения](https://docs.microsoft.com/azure/app-service/azure-web-sites-web-hosting-plans-in-depth-overview)| Определяет набор вычислительных ресурсов, на которых выполняется веб-приложение.|

При создании бота через портал Azure вы сможете подготовить дополнительные ресурсы, например [Application Insights для телеметрии](~/v4sdk/bot-builder-telemetry.md).

Для просмотра документации по команде `az bot` см. этот раздел [справки](https://docs.microsoft.com/cli/azure/bot?view=azure-cli-latest).

Если вы еще не знакомы с концепцией группы ресурсов в Azure, см. раздел со списком [терминов](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview#terminology).
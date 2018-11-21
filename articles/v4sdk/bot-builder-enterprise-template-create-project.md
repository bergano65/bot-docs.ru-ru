---
title: Создание проекта с помощью шаблона Enterprise Bot | Документация Майкрософт
description: Как создать проект бота на основе шаблона Enterprise Bot
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4b85b0e2d3c1ae8c30ea9d5d9fa62783c2968744
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645491"
---
# <a name="enterprise-bot-template---creating-a-new-project"></a>Шаблон Enterprise Bot — создание проекта

> [!NOTE]
> Приведенные здесь сведения относятся к пакету SDK версии 4. 

Шаблон Enterprise Bot объединяет в себе все рекомендации и вспомогательные компоненты, которые мы определили в процессе разработки решений для ведения диалога. Шаблон доступен на таких платформах пакета SDK Bot Builder:

- .NET
- Node.js (ожидается в ближайшее время).

## <a name="net"></a>.NET

Шаблон Enterprise Bot доступен для .NET (версии **4** пакета SDK). Он реализован в виде пакета [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package). Чтобы скачать решение, щелкните следующую ссылку:

- [Шаблон Enterprise Bot в пакете SDK Bot Builder версии 4](https://aka.ms/GetEnterpriseBotTemplate).

#### <a name="prerequisites"></a>Предварительные требования

- [Visual Studio 2017 или более поздней версии](https://www.visualstudio.com/downloads/);
- [Учетная запись Azure](https://azure.microsoft.com/en-us/free/)
- [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/overview?view=azurermps-6.8.1)

### <a name="install-the-template"></a>Установка шаблона

Просто откройте пакет VSIX из сохраненного каталога, и шаблон Bot Builder установится в Visual Studio и будет доступным при следующем открытии.

Чтобы создать проект бота на основе шаблона, просто откройте Visual Studio и последовательно выберите **Файл** > **новый** > **Проект**. В Visual C# выберите **Bot Framework** > Enterprise Bot Template (Шаблон Enterprise Bot). При этом будет локально создан проект бота, который можно изменить по своему усмотрению. 

![Шаблон нового проекта](media/enterprise-template/EnterpriseBot-NewProject.png)

## <a name="deploy-your-bot"></a>Развертывание бота

Теперь проект создан, и можно приступать к созданию инфраструктуры поддержки Azure, а также выполнить настройку и развертывание, чтобы бот был сразу же готов к работе. Сведения о развертывании бота см. [здесь](bot-builder-enterprise-template-deployment.md).

> Это действие необходимо выполнить, иначе инициализация бота (AppInsights) и зависимости LUIS не будут доступны.
## <a name="customize-your-bot"></a>Настройка бота

Убедившись, что развернутый бот готов к работе, настройте его в соответствии со своими потребностями. Сведения о настройке бота см. [здесь](bot-builder-enterprise-template-customize.md).

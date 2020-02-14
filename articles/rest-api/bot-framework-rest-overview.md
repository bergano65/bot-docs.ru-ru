---
title: REST API для Bot Framework — Служба Azure Bot
description: Сведения о том, как приступить к работе с интерфейсами REST API Bot Framework, которые можно использовать для создания ботов и клиентов, подключающихся к ботам.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 2c42b39f838177dcc0e07b53357ab1395500e010
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071802"
---
# <a name="bot-framework-rest-apis"></a>REST API для Bot Framework
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Интерфейсы REST API для Bot Framework позволяют создавать боты, которые обмениваются сообщениями с каналами, настроенными на [портале Azure](https://portal.azure.com), сохраняют и получают данные о состоянии и подключают ваши клиентские приложения к ботам. Все службы Bot Framework используют стандартные отраслевые REST и JSON по протоколу HTTPS.

## <a name="build-a-bot"></a>Создание бота

Создать бот можно с помощью любого языка программирования. В основе бота лежит служба Bot Connector, используемая для обмена сообщениями с каналами, настроенными на портале Bot Framework. 

> [!TIP]
> Bot Framework предоставляет клиентские библиотеки, которые могут использоваться для разработки ботов на C# или Node.js. Чтобы создать бот на языке C#, используйте [пакет SDK Bot Framework для C#](../dotnet/bot-builder-dotnet-overview.md). Чтобы создать бот на языке Node.js, используйте [пакет SDK Bot Framework для Node.js](../nodejs/index.md). 

Дополнительные сведения о создании ботов с использованием службы Bot Connector см. в разделе [Основные понятия](bot-framework-rest-connector-concepts.md).

## <a name="build-a-client"></a>Создание клиента

C помощью API Direct Line вы сможете взаимодействовать с ботом в своем клиентском приложении. API Direct Line реализует механизм проверки подлинности, который использует стандартные шаблоны секрета или маркера и предоставляет стабильную схему проверки подлинности, которая продолжит работать даже при изменении версии протокола бота. Дополнительные сведения о взаимодействии клиента и бота с использованием API Direct Line см. в разделе [Основные понятия](bot-framework-rest-direct-line-3-0-concepts.md). 
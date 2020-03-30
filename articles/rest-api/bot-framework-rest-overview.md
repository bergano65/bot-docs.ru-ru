---
title: REST API для Bot Framework — Служба Azure Bot
description: Сведения о том, как приступить к работе с интерфейсами REST API Bot Framework, которые можно использовать для создания ботов и клиентов, подключающихся к ботам.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/20/2020
ms.openlocfilehash: 60a5e08caf11977eed72bfba0dfeba3472194822
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117456"
---
# <a name="bot-framework-rest-apis"></a>REST API для Bot Framework

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Большинство ботов Bot Framework создаются с помощью пакета SDK для Bot Framework, который организует бота и управляет диалогами. Вместо работы с пакетом SDK вы можете отправлять сообщения боту напрямую через REST API.

## <a name="build-a-bot"></a>Создание бота

Включив в код обращения к API REST из Bot Framework, вы сможете отправлять сообщения пользователям и получать от них сообщения через любой канал, настроенный в регистрации службы Azure Bot для вашего бота.

> [!TIP]
> Bot Framework предоставляет клиентские библиотеки, которые могут использоваться для разработки ботов на C# или Node.js.
> Чтобы создать бот на языке C#, используйте [пакет SDK Bot Framework для C#](../dotnet/bot-builder-dotnet-overview.md).
> Чтобы создать бот на языке Node.js, используйте [пакет SDK Bot Framework для Node.js](../nodejs/index.md).

См. сведения о [создании ботов и использовании службы Azure Bot](../bot-service-overview-introduction.md).

## <a name="build-a-direct-line-or-web-chat-client"></a>Создание клиента Direct Line или Web Chat

Большинство каналов, таких как Facebook, Teams и Slack, предоставляют собственные клиенты, но Direct Line и Web Chat позволяют использовать для взаимодействия с ботом собственное клиентское приложение. API Direct Line реализует механизм проверки подлинности, который использует стандартные шаблоны секрета или маркера и предоставляет стабильную схему проверки подлинности, которая продолжит работать даже при изменении версии протокола бота. Дополнительные сведения о взаимодействии клиента и бота с использованием API Direct Line см. в разделе [Основные понятия](bot-framework-rest-direct-line-3-0-concepts.md).

Клиенты Direct Line и Web Chat можно создавать на разных языках и размещать в разных типах развертываний (например, автономное приложение вместо службы). См. сведения о [Direct Line](https://docs.microsoft.com/azure/bot-service/bot-service-channel-directline?view=azure-bot-service-4.0).

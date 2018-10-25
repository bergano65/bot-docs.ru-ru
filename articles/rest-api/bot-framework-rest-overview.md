---
title: REST API Bot Builder | Документы Майкрософт
description: Сведения о том, как приступить к работе с интерфейсами REST API Bot Framework, которые можно использовать для создания ботов и клиентов, подключающихся к ботам.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 632d0b1e603445954eb4894fcfc256d5ac738539
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998441"
---
# <a name="bot-framework-rest-apis"></a>REST API для Bot Framework
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Интерфейсы API REST Bot Framework позволяют создавать боты, которые обмениваются сообщениями с каналами, настроенными на <a href="https://dev.botframework.com/" target="_blank">портале Bot Framework</a>, сохраняют и получают данные о состоянии и подключают ваши собственные клиентские приложения к вашим ботам. Все службы Bot Framework используют стандартные отраслевые REST и JSON по протоколу HTTPS.

## <a name="build-a-bot"></a>Создание бота

Создать бот можно с помощью любого языка программирования. В основе бота лежит служба Bot Connector, используемая для обмена сообщениями с каналами, настроенными на портале Bot Framework. 

> [!TIP]
> Bot Framework предоставляет клиентские библиотеки, которые могут использоваться для разработки ботов на C# или Node.js. Чтобы создать бот на языке C#, используйте [пакет SDK Bot Builder для C#](../dotnet/bot-builder-dotnet-overview.md). Чтобы создать бот на Node.js, используйте [пакет SDK Bot Builder для Node.js](../nodejs/index.md). 

Дополнительные сведения о создании ботов с использованием службы Bot Connector см. в разделе [Основные понятия](bot-framework-rest-connector-concepts.md).

## <a name="build-a-client"></a>Создание клиента

C помощью API Direct Line вы сможете взаимодействовать с ботом в своем клиентском приложении. API Direct Line реализует механизм проверки подлинности, который использует стандартные шаблоны секрета или маркера и предоставляет стабильную схему проверки подлинности, которая продолжит работать даже при изменении версии протокола бота. Дополнительные сведения о взаимодействии клиента и бота с использованием API Direct Line см. в разделе [Основные понятия](bot-framework-rest-direct-line-3-0-concepts.md). 
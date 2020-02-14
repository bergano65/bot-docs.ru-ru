---
title: Ключевые понятия служб Bot Connector и "Состояние бота" — Служба Azure Bot
description: Понимание ключевых понятий службы Bot Connector и службы Состояние бота платформы Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/16/2019
ms.openlocfilehash: 15f30ff5d4db24d222a1a9df3ac98e24e576b238
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071892"
---
# <a name="key-concepts"></a>Основные понятия

Службу Bot Connector используют для обмена данными с пользователями через различные каналы связи, такие как Skype, электронная почта, Slack и другие. В этой статье рассматриваются ключевые понятия службы Bot Connector.

## <a name="bot-connector-service"></a>Служба Bot Connector

Служба Bot Connector позволяет боту обмениваться сообщениями с каналами, настроенными на [портале Azure](https://portal.azure.com). В нем используются стандартные отраслевые форматы REST и JSON через HTTPS и обеспечивается проверка подлинности с помощью токенов носителя JWT. Подробные сведения о том, как использовать службу Bot Connector, см. в статье [Проверка подлинности](bot-framework-rest-connector-authentication.md) и в остальных статьях этого раздела.

### <a name="activity"></a>Действие

С помощью службы Bot Connector происходит обмен данными между ботом и каналом (пользователем) путем передачи объекта [Activity][Activity]. Наиболее распространенным типом действия является **сообщение**, однако существуют другие типы действий, которые можно использовать для передачи данных различных типов в бот или канал. Дополнительные сведения о действиях в службе Bot Connector см. в статье [Общие сведения о действиях](https://aka.ms/botSpecs-activitySchema).

## <a name="bot-state-service"></a>Служба Состояние бота

Служба состояний Microsoft Bot Framework устарела с 30 марта 2018 г. Ранее боты, созданные на основе службы Azure Bot или пакета SDK для Bot Builder, по умолчанию использовали подключение к этой службе, размещенной корпорацией Майкрософт, для хранения данных о состоянии бота. Теперь эти боты нужно обновить, чтобы они использовали собственное хранилище состояний.

## <a name="authentication"></a>Аутентификация

Служба Bot Connector обеспечивает проверку подлинности с помощью токенов носителя JWT. Подробные сведения о способах проверки подлинности исходящих запросов, отправляемых ботом в Bot Framework, о способах проверки подлинности входящих запросов, которые ваш бот получает от Bot Framework, и многое другое см. в статье [Проверка подлинности](bot-framework-rest-connector-authentication.md). 

## <a name="client-libraries"></a>Клиентские библиотеки

Bot Framework предоставляет клиентские библиотеки, которые могут использоваться для разработки ботов на C# или Node.js. 

- Чтобы создать бот на языке C#, используйте [пакет SDK Bot Framework для C#](../dotnet/bot-builder-dotnet-overview.md). 
- Чтобы создать бот на языке Node.js, используйте [пакет SDK Bot Framework для Node.js](../nodejs/index.md). 

Помимо моделирования службы Bot Connector, каждый пакет SDK Bot Framework также предоставляет мощную систему создания диалоговых окон с поддержкой логики беседы, встроенными запросами для простых сведений типа "Да или Нет", строк, чисел и перечислений, встроенной поддержкой мощных платформ ИИ (например, <a href="https://www.luis.ai/" target="_blank">LUIS</a>) и многими другими возможностями. 

> [!NOTE]
> Вместо пакета SDK для C# или пакета SDK для Node.js вы можете создать собственную клиентскую библиотеку на любом языке, используя <a href="https://aka.ms/connector-swagger-file" target="_blank">файл Swagger для службы Bot Connector</a>.

## <a name="additional-resources"></a>Дополнительные ресурсы

Узнайте больше о создании ботов с помощью службы Bot Connector, ознакомившись со статьями в этом разделе, начиная со статьи [Проверка подлинности](bot-framework-rest-connector-authentication.md). Если у вас возникли проблемы или есть предложения относительно службы Bot Connector, список доступных ресурсов находится в статье [Поддержка](../bot-service-resources-links-help.md). 

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

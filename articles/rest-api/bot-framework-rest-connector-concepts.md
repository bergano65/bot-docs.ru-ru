---
title: Ключевые понятия служб Bot Connector и "Состояние бота" — Служба Azure Bot
description: Понимание ключевых понятий службы Bot Connector и службы Состояние бота платформы Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/20/2020
ms.openlocfilehash: f68451b8bf336a8b4f3bfc026c682338c561aed0
ms.sourcegitcommit: 308e6df385b9bac9c8d60f8b75eabc813b823c38
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/20/2020
ms.locfileid: "77519903"
---
# <a name="key-concepts"></a>Основные понятия

Bot Framework и служба Azure Bot позволяют боту взаимодействовать с пользователями через Teams, Кортану, Facebook и т. п. Каналы предоставляются в двух видах: как служба в составе службы Azure Bot и как библиотеки адаптеров для использования с пакетом SDK для Bot Framework. В этой статье рассматриваются стандартизированные каналы, включенные в службу Azure Bot.

## <a name="bot-framework-channels"></a>Каналы Bot Framework

Каналы Bot Framework позволяют боту обмениваться сообщениями с каналами, настроенными на [портале Azure](https://portal.azure.com). В нем используются стандартные отраслевые форматы REST и JSON через HTTPS и обеспечивается проверка подлинности с помощью токенов носителя JWT. Подробные сведения о том, как использовать службу Bot Connector, см. в статье [Проверка подлинности](bot-framework-rest-connector-authentication.md) и в остальных статьях этого раздела.

### <a name="activity"></a>Действие

С помощью службы Bot Connector происходит обмен данными между ботом и каналом (пользователем) путем передачи объекта [Activity][Activity]. Наиболее распространенным типом действия является **сообщение**, однако существуют другие типы действий, которые можно использовать для передачи данных различных типов в бот или канал. Дополнительные сведения о действиях в службе Bot Connector см. в статье [Общие сведения о действиях](https://aka.ms/botSpecs-activitySchema).

## <a name="authentication"></a>Аутентификация

Службы Bot Framework используют для проверки подлинности токены носителя JWT. Подробные сведения о способах проверки подлинности исходящих запросов, отправляемых ботом в Bot Framework, о способах проверки подлинности входящих запросов, которые ваш бот получает от Bot Framework, и многое другое см. в статье [Проверка подлинности](bot-framework-rest-connector-authentication.md). 

## <a name="client-libraries"></a>Клиентские библиотеки

Bot Framework предоставляет клиентские библиотеки, которые могут использоваться для разработки ботов на C#, JavaScript или Python.

- Чтобы создать бот на языке C#, используйте [пакет SDK Bot Framework для C#](../dotnet/bot-builder-dotnet-overview.md). 
- Чтобы создать бот на языке Node.js, используйте [пакет SDK Bot Framework для Node.js](../nodejs/index.md). 

Помимо упрощенных вызовов API REST в Bot Framework, каждый пакет SDK Bot Framework также предоставляет полнофункциональную систему для создания диалогов с поддержкой логики беседы, встроенными запросами простых сведений (ответы "Да" или "Нет", строки, числа и возможность выбора из списка), со встроенной поддержкой мощных платформ ИИ (например, <a href="https://www.luis.ai/" target="_blank">LUIS</a>) и многими другими функциями. 

> [!NOTE]
> Вместо пакета SDK вы можете создать собственную клиентскую библиотеку на любом языке, используя <a href="https://aka.ms/connector-swagger-file" target="_blank">файл Swagger для службы Bot Connector</a> или прямой вызов API REST из кода приложения.

## <a name="bot-state-service"></a>Служба Состояние бота

Служба состояний Microsoft Bot Framework устарела с 30 марта 2018 г. Ранее боты, созданные на основе службы Azure Bot или пакета SDK для Bot Builder, по умолчанию использовали подключение к этой службе, размещенной корпорацией Майкрософт, для хранения данных о состоянии бота. Теперь эти боты нужно обновить, чтобы они использовали собственное хранилище состояний.

## <a name="additional-resources"></a>Дополнительные ресурсы

Узнайте больше о создании ботов с помощью службы Bot Connector, ознакомившись со статьями в этом разделе, начиная со статьи [Проверка подлинности](bot-framework-rest-connector-authentication.md). Если у вас возникли проблемы или есть предложения относительно службы Bot Connector, список доступных ресурсов находится в статье [Поддержка](../bot-service-resources-links-help.md). 

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object

---
title: Ключевые понятия службы Bot Connector и службы Состояние бота | Документация Майкрософт
description: Понимание ключевых понятий службы Bot Connector и службы Состояние бота платформы Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: a3d6bd957b835a0b8d86e47595ce28506c32d636
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224559"
---
# <a name="key-concepts"></a>Основные понятия

Службу Bot Connector и службу Состояние бота используют для обмена данными с пользователями через различные каналы связи, такие как Skype, электронная почта, Slack и другие. В этой статье рассматриваются ключевые понятия службы Bot Connector и службы Состояние бота.

> [!IMPORTANT]
> API службы состояния Bot Framework не рекомендуется использовать для рабочей среды. Он может считаться устаревшим в будущем выпуске. Рекомендуется обновить код бота для использования хранилища в памяти в целях тестирования или использовать одно из **расширений Azure** для ботов в рабочей среде. Дополнительные сведения см. в разделе **Управление данными состояния** для реализации [.NET](~/dotnet/bot-builder-dotnet-state.md) или [Node.js](~/nodejs/bot-builder-nodejs-state.md).

## <a name="bot-connector-service"></a>Служба Bot Connector

Служба Bot Connector позволяет боту обмениваться сообщениями на каналах, настроенных на <a href="https://dev.botframework.com/" target="_blank">портале Bot Framework</a>. В нем используются стандартные отраслевые форматы REST и JSON через HTTPS и обеспечивается проверка подлинности с помощью токенов носителя JWT. Подробные сведения о том, как использовать службу Bot Connector, см. в статье [Проверка подлинности](bot-framework-rest-connector-authentication.md) и в остальных статьях этого раздела.

### <a name="activity"></a>Действие

С помощью службы Bot Connector происходит обмен данными между ботом и каналом (пользователем) путем передачи объекта [Действие][Activity]. Наиболее распространенным типом действия является **сообщение**, однако существуют другие типы действий, которые можно использовать для передачи данных различных типов в бот или канал. Дополнительные сведения о действиях в службе Bot Connector см. в статье [Общие сведения о действиях](bot-framework-rest-connector-activities.md).

## <a name="bot-state-service"></a>Служба Состояние бота

Служба Состояние бота позволяет боту хранить и получать данные состояния, связанные с пользователем, общением или определенным пользователем в контексте определенного общения. В нем используются стандартные отраслевые форматы REST и JSON через HTTPS и обеспечивается проверка подлинности с помощью токенов носителя JWT. Все данные, которые вы сохраните с помощью службы Состояние бота, будут зашифрованы и сохранены в Azure.

Служба Состояние бота полезна только в сочетании со службой Bot Connector. То есть ее можно использовать только для хранения данных состояния, связанных с общением, которое ваш бот проводит с помощью службы Bot Connector. Подробные сведения о том, как использовать службу Состояние бота, см. в статьях [Проверка подлинности](bot-framework-rest-connector-authentication.md) и [Управление данными состояния](bot-framework-rest-state.md).

## <a name="authentication"></a>Authentication

Служба Bot Connector, как и служба Состояние бота, обеспечивают проверку подлинности с помощью токенов носителя JWT. Подробные сведения о способах проверки подлинности исходящих запросов, отправляемых ботом в Bot Framework, о способах проверки подлинности входящих запросов, которые ваш бот получает от Bot Framework, и многое другое см. в статье [Проверка подлинности](bot-framework-rest-connector-authentication.md). 

## <a name="client-libraries"></a>Клиентские библиотеки

Bot Framework предоставляет клиентские библиотеки, которые могут использоваться для разработки ботов на C# или Node.js. 

- Чтобы создать бот на языке C#, используйте [пакет SDK Bot Framework для C#](../dotnet/bot-builder-dotnet-overview.md). 
- Чтобы создать бот на языке Node.js, используйте [пакет SDK Bot Framework для Node.js](../nodejs/index.md). 

Помимо моделирования службы Bot Connector и службы Состояние бота, каждый пакет SDK Bot Framework также предоставляет мощную систему создания диалоговых окон с поддержкой логики беседы, встроенными запросами для простых сведений типа "Да или Нет", строк, чисел и перечислений, встроенной поддержкой мощных платформ ИИ (например, <a href="https://www.luis.ai/" target="_blank">LUIS</a>) и многими другими возможностями. 

> [!NOTE]
> Вместо пакета SDK для C# или пакета SDK для Node.js вы можете создать собственную клиентскую библиотеку на любом языке, используя <a href="https://raw.githubusercontent.com/Microsoft/BotBuilder/master/CSharp/Library/Microsoft.Bot.Connector.Shared/Swagger/ConnectorAPI.json" target="_blank">файл Swagger для службы Bot Connector</a> и <a href="https://raw.githubusercontent.com/Microsoft/BotBuilder/master/CSharp/Library/Microsoft.Bot.Connector.Shared/Swagger/StateAPI.json" target="_blank">файл Swagger для службы Состояние бота</a>.

## <a name="additional-resources"></a>Дополнительные ресурсы

Узнайте больше о создании ботов с помощью службы Bot Connector и службы Состояние бота, ознакомившись со статьями в этом разделе, начиная со статьи [Проверка подлинности](bot-framework-rest-connector-authentication.md). Если у вас возникли проблемы или есть предложения относительно службы Bot Connector и службы Состояние бота, список доступных ресурсов находится в статье [Поддержка](../bot-service-resources-links-help.md). 

[Activity]: bot-framework-rest-connector-api-reference.md#activity-object
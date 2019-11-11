---
title: Новые возможности | Документация Майкрософт
description: Узнайте о новых возможностях Bot Framework.
keywords: bot framework, azure bot service
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 11/04/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8cb03814a9fbb7fdbbfb457400eca3cf6a67ec17
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592242"
---
# <a name="whats-new-november-2019-ignite"></a>Отчет о новых возможностях за ноябрь 2019 г. (Ignite)

[!INCLUDE[applies-to](includes/applies-to.md)]

Пакет SDK Bot Framework версии 4 — это [пакет SDK с открытым кодом](https://github.com/microsoft/botframework-sdk/#readme), позволяющий разработчикам моделировать и создавать сложные диалоги с использованием популярных языков программирования.

В этой статье перечислены новые ключевые возможности и улучшения Bot Framework и службы Azure Bot.


|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|Release |[4.6 (общедоступная версия)][1] | [4.6 (общедоступная версия)][2] | [4 (бета-версия)][3] | [3 (предварительная версия)][3a]|
|Docs | [docs][5] |[docs][5] |  | |
|Примеры |[.NET Core][6], [WebAPI][10] |[Node.js][7], [TypeScript][8], [ES6][9]  | | | 


[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/typescript_nodejs
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi

#### <a name="bot-framework-sdk-for-microsoft-teams-ga"></a>Пакет SDK Bot Framework для Microsoft Teams (общедоступная версия)
В пакете SDK для Bot Framework версии 4.6 реализована функция создания ботов для Teams, что позволяет использовать их в каналах и групповых беседах. Если вы добавите бота в группу или чат, функции бота будут доступны всем пользователям этой беседы непосредственно в ходе беседы.  [[Документация](https://docs.microsoft.com/azure/bot-service/bot-builder-basics-teams)]

#### <a name="bot-framework-for-power-virtual-agent-preview"></a>Bot Framework для Power Virtual Agent (предварительная версия)

Служба Power Virtual Agent позволяет создавать ботов с помощью пользовательского интерфейса в формате SaaS без необходимости создания кода и управления отдельными службами искусственного интеллекта. Power Virtual Agent можно расширить с помощью Microsoft Bot Framework, что позволит разработчикам и пользователям совместно разрабатывать ботов для своих организаций. [[Документация](https://docs.microsoft.com/dynamics365/ai/customer-service-virtual-agent/overview)]


#### <a name="bot-framework-sdk-for-skills-preview"></a>Пакет SDK Bot Framework для Skills (предварительная версия)

- **Skills для ботов**. Создавайте повторно используемые навыки ведения беседы, чтобы расширить возможности бота. Используйте готовые навыки, такие как календарь, электронная почта, задача, точка интереса, транспорт, погода и новости. Навыки — это языковые модели, диалоги, вопросы и ответы, а также код интеграции. Их можно настраивать и расширять под определенные потребности. [[Документация](https://microsoft.github.io/botframework-solutions/overview/skills/)]

- **Навыки для Power Virtual Agent (выход ожидается)** . Для ботов с поддержкой Power Virtual Agent вы можете создавать новые навыки на основе Bot Framework и Azure Cognitive Services, не разрабатывая бота с нуля. 

#### <a name="adaptive-dialogs-preview"></a>Адаптивные диалоги (предварительная версия)
Адаптивные диалоги позволяют разработчикам динамически обновлять поток беседы на основе контекста и событий. Это особенно удобно при переключениях контекста беседы и паузах в ней. [[Документация][48] | [Примеры для C#][49]] 

#### <a name="language-generation-preview"></a>Создание текста (предварительная версия)
Функция создания текста позволяет разработчикам выделить логику, используемую для создания ответов бота, включая возможность определять несколько вариантов фраз, выполнять простые выражения на основе контекста и обращаться к сохраненным данным беседы. [[Документация][44] | [Примеры для C#][45]]

#### <a name="common-expression-language-preview"></a>Язык общих выражений (предварительная версия)
Язык общих выражений позволяет оценивать работу логики на основе условий во время выполнения. Общий язык можно использовать в пакетах SDK для Bot Framework и в компонентах искусственного интеллекта для бесед, таких как адаптивные диалоговые окна и создание текста. [[документация][40] | [API][41]]


[40]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/common-expression-language#readme
[41]:https://github.com/Microsoft/BotBuilder-Samples/blob/master/experimental/common-expression-language/api-reference.md
[43]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation#readme
[44]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/docs
[45]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/csharp_dotnetcore
[46]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/language-generation/javascript_nodejs/13.core-bot
[47]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme
[48]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs
[49]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore
[50]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/declarative

## <a name="additional-information"></a>Дополнительная информация
- См. [предыдущие объявления о новых возможностях](what-is-new-archive.md).

---
title: Новые возможности — Служба Azure Bot
description: Узнайте о новых возможностях Bot Framework.
keywords: bot framework, azure bot service
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 12/10/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f223766f0dcc4145fa058dcd447d4e3967db93d3
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791006"
---
# <a name="whats-new-december-2019"></a>Новые возможности (декабрь 2019 г.)

[!INCLUDE[applies-to](includes/applies-to.md)]

Пакет SDK Bot Framework версии 4 — это [пакет SDK с открытым кодом](https://github.com/microsoft/botframework-sdk/#readme), позволяющий разработчикам моделировать и создавать сложные диалоги с использованием популярных языков программирования.

В этой статье перечислены новые ключевые возможности и улучшения Bot Framework и службы Azure Bot.

|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|Release |[GA][1] | [GA][2] | [GA][3] | [3 (предварительная версия)][3a]|
|Docs | [docs][5] |[docs][5] |[docs][5]  | |
|Примеры |[.NET Core][6], [WebAPI][10] |[Node.js][7], [TypeScript][8], [ES6][9]  | [Python][11a] | | 

#### <a name="python-sdk-ga"></a>Пакет SDK для Python (общедоступная версия)
Мы рады сообщить о выпуске общедоступной версии пакета SDK для Python. Этот пакет SDK поддерживает основную среду выполнения ботов, соединители, ПО промежуточного слоя, диалоги, приглашения, Распознавание речи (LUIS) и QnA Maker. 

#### <a name="bot-framework-sdk-for-skills-ga"></a>Пакет SDK Bot Framework для Skills (общедоступная версия)

- **Skills для ботов**. Создавайте повторно используемые навыки ведения беседы, чтобы расширить возможности бота. Используйте встроенные средства, такие как календарь, электронная почта, задача, точка интереса, транспорт, погода и новости. Навыки — это языковые модели, диалоги, вопросы и ответы, а также код интеграции. Их можно настраивать и расширять под определенные потребности. [[Документация](https://aka.ms/skills-docs)]

- **Навыки для Power Virtual Agent (выход ожидается)** . Для ботов с поддержкой Power Virtual Agent вы можете создавать новые навыки на основе Bot Framework и Azure Cognitive Services, не разрабатывая бота с нуля. 

#### <a name="bot-framework-for-power-virtual-agent-ga"></a>Bot Framework для Power Virtual Agent (общедоступная версия)

Служба Power Virtual Agent позволяет создавать ботов с помощью пользовательского интерфейса в формате SaaS без необходимости создания кода и управления отдельными службами искусственного интеллекта. Power Virtual Agent можно расширить с помощью Microsoft Bot Framework, что позволит разработчикам и пользователям совместно разрабатывать ботов для своих организаций. [[Документация](https://docs.microsoft.com/dynamics365/ai/customer-service-virtual-agent/overview)]

#### <a name="adaptive-dialogs-preview"></a>Адаптивные диалоги (предварительная версия)
Адаптивные диалоги позволяют разработчикам динамически обновлять поток беседы на основе контекста и событий. Это особенно удобно при переключениях контекста беседы и паузах в ней. [[Документация][48] | [Примеры для C#][49]] 

#### <a name="language-generation-preview"></a>Создание текста (предварительная версия)
Функция создания текста позволяет разработчикам выделить логику, используемую для создания ответов бота, включая возможность определять несколько вариантов фраз, выполнять простые выражения на основе контекста и обращаться к сохраненным данным беседы. [[Документация][44] | [Примеры для C#][45]]

#### <a name="common-expression-language-preview"></a>Язык общих выражений (предварительная версия)
Язык общих выражений позволяет оценивать работу логики на основе условий во время выполнения. Общий язык можно использовать в пакетах SDK для Bot Framework и в компонентах искусственного интеллекта для бесед, таких как адаптивные диалоговые окна и создание текста. [[документация][40] | [API][41]]

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
[11a]:https://aka.ms/python-sample-repo


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

## <a name="additional-information"></a>Дополнительные сведения
- См. [предыдущие объявления о новых возможностях](what-is-new-archive.md).

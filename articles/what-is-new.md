---
title: Новые возможности | Документация Майкрософт
description: Узнайте о новых возможностях Bot Framework.
keywords: bot framework, azure bot service
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 07/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 708ad1fac166f312ad6ccf14a024f821f19eaaf2
ms.sourcegitcommit: e573c586472c5328ce875114308d9d1b73651e62
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/03/2019
ms.locfileid: "70224390"
---
# <a name="whats-new-in-bot-framework-july-2019"></a>Новые возможности Bot Framework (июль 2019 г.)

[!INCLUDE[applies-to](includes/applies-to.md)]

Пакет SDK Bot Framework версии 4 — это [пакет SDK с открытым кодом][1a], позволяющий разработчикам моделировать и создавать сложные диалоги с использованием популярных языков программирования.

В этой статье перечислены новые ключевые возможности и улучшения Bot Framework и службы Azure Bot.

|   | C#  | JS  | Python |   
|---|:---:|:---:|:------:|
|SDK |[4.5][1] | [4.5][2] | [4.4.0b2 (предварительная версия)][3] | 
|Docs | [docs][5] |[docs][5] |  | |
|Примеры |[.NET Core][6], [WebAPI][10] |[Node.js][7] , [TypeScript][8], [es6][9]  | [Python][111] | | 

[1a]:https://github.com/microsoft/botframework-sdk/#readme
[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_typescript
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[111]:https://github.com/Microsoft/botbuilder-python/tree/master/samples


## <a name="bot-framework-channels"></a>Каналы Bot Framework
- [Direct Line Speech (общедоступная предварительная версия)](https://aka.ms/streaming-extensions) | [Документация](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0). Bot Framework и Службы речи Майкрософт предоставляют канал, который поддерживает двунаправленную потоковую передачу речи и текста между клиентом и приложением бота по протоколам WebSocket.  

- [Расширение Службы приложений Direct Line (общедоступная предварительная версия)](https://portal.azure.com) | [Документация](https://aka.ms/directline-ase). Версия Direct Line, которая позволяет клиентам напрямую подключаться к ботам с помощью Direct Line API. Это обеспечивает множество преимуществ, в том числе повышение производительности и большую степень изоляции. Расширение Службы приложений Direct Line доступно во всех Службах приложений Azure, включая размещенные в Среде службы приложений Azure. Среда службы приложений Azure обеспечивает изоляцию и идеально подходит для работы в виртуальной сети. Виртуальная сеть позволяет создать личное пространство в Azure и является критически важной для вашей облачной сети, так как она обеспечивает изоляцию, сегментацию и другие ключевые преимущества. 

## <a name="bot-framework-sdk"></a>Пакет SDK Bot Framework
- [Adaptive Dialog (пакет SDK 4.6, предварительная версия)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme) | [Документация](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs) | [Примеры C#](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore). Средство Adaptive Dialog теперь позволяет разработчикам динамически обновлять поток беседы на основе контекста и событий. Это особенно полезно при переключениях контекста беседы и паузах в ней. 
  
- [Пакет SDK Bot Framework для Python (предварительная версия 2)](https://github.com/microsoft/botbuilder-python) | [Примеры](https://github.com/Microsoft/botbuilder-python/tree/master/samples). Пакет SDK для Python теперь поддерживает OAuth, запросы, Cosmos DB, а также включает все основные функции пакета SDK 4.5. Также доступны примеры, которые познакомят вас с новыми возможностями пакета SDK.

## <a name="bot-framework-testing"></a>Тестирование в Bot Framework
- [Документация](https://aka.ms/testing-framework) | Пакеты модульного тестирования ([C#](https://aka.ms/nuget-botbuilder-testing)/ [JavaScript](https://aka.ms/npm-botbuilder-testing)) | [Пример C#](https://aka.ms/cs-core-test-sample) | [Пример JS](https://aka.ms/js-core-test-sample): Учитывая пожелания клиентов и разработчиков касательно более совершенных средств тестирования, в июльскую версию пакета SDK мы добавили новую функцию модульного тестирования. Пакет Microsoft.Bot.Builder.testing упрощает процесс модульного тестирования диалогов в боте.  

- [Тестирование канала](https://github.com/Microsoft/BotFramework-Emulator/releases) | [Документация](https://aka.ms/channel-testing). 

Bot Inspector — это новая функция Bot Framework Emulator, презентация которой состоялась на конференции Microsoft Build 2019. Она позволяет отлаживать и тестировать ботов в таких каналах, как Microsoft Teams, Slack, Кортана и т. д. При использовании бота в определенном канале полученные ботом сообщения будут дублироваться в Bot Framework Emulator, где вы можете их просматривать. Кроме того, можно отобразить моментальный снимок состояния бота на любом шаге обмена данными между каналом и ботом.

## <a name="web-chat"></a>Веб-чат.
- Учитывая пожелания корпоративных клиентов, мы добавили [пример веб-чата](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps#single-sign-on-demo-for-enterprise-apps-using-oauth), демонстрирующий, как обеспечить пользователю доступ к ресурсам корпоративного приложения с помощью бота. Взаимодействие OAuth с Microsoft Graph и API GitHub демонстрируется с использованием ресурсов двух типов.

## <a name="solutions"></a>Решения
- [Акселератор решений виртуального помощника](https://github.com/Microsoft/botframework-solutions#readme). Доступен набор шаблонов, акселераторы решений и навыки для создания сложных диалогов. Новый клиент приложений Android для виртуального помощника, который интегрируется с Direct Line Speech и виртуальным помощником, демонстрирует, как клиент на устройстве может взаимодействовать с вашим виртуальным помощником и отображать адаптивные карточки. Обновления также включают поддержку Direct Line Speech и Microsoft Teams.
  
- [Виртуальный агент Dynamics 365 для обслуживания клиентов (общедоступная предварительная версия)](https://dynamics.microsoft.com/en-us/ai/virtual-agent-for-customer-service/). Общедоступная предварительная версия позволяет качественно обслуживать клиентов с помощью интеллектуальных и адаптируемых виртуальных агентов. Специалисты по обслуживанию клиентов могут легко создавать и улучшать ботов, используя аналитические сведения на основе ИИ.
  
- [Chat for Dynamics 365](https://www.powerobjects.com/powerpacks/powerchat/). Решение Chat for Dynamics 365 предлагает несколько возможностей, обеспечивающих эффективное взаимодействие агентов поддержки с пользователями и их высокую производительность. Доступны возможности интерактивного чата и отслеживания диалогов с посетителями вашего веб-сайта, размещенного в Microsoft Dynamics 365.

## <a name="additional-information"></a>Дополнительная информация
- См. [предыдущие объявления о новых возможностях](what-is-new-archive.md).

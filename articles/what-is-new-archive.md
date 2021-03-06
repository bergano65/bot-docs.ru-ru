---
title: Новые возможности (архив) — Служба Azure Bot
description: Узнайте о новых возможностях Bot Framework.
keywords: bot framework, azure bot service
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 11/28/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 95ce084b68da41d9d9f434348c4539c9f67a2c78
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2020
ms.locfileid: "77441528"
---
# <a name="whats-new-november-2019"></a>Отчет о новых возможностях за ноябрь 2019 г.

[!INCLUDE[applies-to](includes/applies-to.md)]

Пакет SDK Bot Framework версии 4 — это [пакет SDK с открытым кодом](https://github.com/microsoft/botframework-sdk/#readme), позволяющий разработчикам моделировать и создавать сложные диалоги с использованием популярных языков программирования.

В этой статье перечислены новые ключевые возможности и улучшения Bot Framework и службы Azure Bot.


|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|Release |[4.6 (общедоступная версия)][1] | [4.6 (общедоступная версия)][2] | [4 (бета-версия)][3] | [3 (предварительная версия)][3a]|
|Docs | [docs][5] |[docs][5] |  | |
|Примеры |[.NET Core][6], [WebAPI][10] |[Node.js][7], [TypeScript][8], [ES6][9]  | | | 

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

## <a name="whats-new-july-2019"></a>Что нового (июль 2019 г.)

[!INCLUDE[applies-to](includes/applies-to.md)]

Пакет SDK Bot Framework версии 4 — это [пакет SDK с открытым кодом][1a], позволяющий разработчикам моделировать и создавать сложные диалоги с использованием популярных языков программирования.

В этой статье перечислены новые ключевые возможности и улучшения Bot Framework и службы Azure Bot.

|   | C#  | JS  | Python |   
|---|:---:|:---:|:------:|
|SDK |[4.5][1] | [4.5][2] | [4.4.0b2 (предварительная версия)][3] | 
|Docs | [docs][5] |[docs][5] |  | |
|Примеры |[.NET Core][6], [WebAPI][10] |[Node.js][7] , [TypeScript][8], [es6][9]  | [Python][11] | | 


### <a name="bot-framework-channels"></a>Каналы Bot Framework
- [Direct Line Speech (общедоступная предварительная версия)](https://aka.ms/streaming-extensions) | [Документация](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0). Bot Framework и Службы речи Майкрософт предоставляют канал, который поддерживает двунаправленную потоковую передачу речи и текста между клиентом и приложением бота по протоколам WebSocket.  

- [Расширение Службы приложений Direct Line (общедоступная предварительная версия)](https://portal.azure.com) | [Документация](https://aka.ms/directline-ase). Версия Direct Line, которая позволяет клиентам напрямую подключаться к ботам с помощью Direct Line API. Это обеспечивает множество преимуществ, в том числе повышение производительности и большую степень изоляции. Расширение Службы приложений Direct Line доступно во всех Службах приложений Azure, включая размещенные в Среде службы приложений Azure. Среда службы приложений Azure обеспечивает изоляцию и идеально подходит для работы в виртуальной сети. Виртуальная сеть позволяет создать личное пространство в Azure и является критически важной для вашей облачной сети, так как она обеспечивает изоляцию, сегментацию и другие ключевые преимущества. 

### <a name="bot-framework-sdk"></a>Пакет SDK Bot Framework
- [Adaptive Dialog (пакет SDK 4.6, предварительная версия)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog#readme) | [Документация](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/docs) | [Примеры C#](https://github.com/Microsoft/BotBuilder-Samples/tree/master/experimental/adaptive-dialog/csharp_dotnetcore). Средство Adaptive Dialog теперь позволяет разработчикам динамически обновлять поток беседы на основе контекста и событий. Это особенно полезно при переключениях контекста беседы и паузах в ней. 
  
- [Пакет SDK Bot Framework для Python (предварительная версия 2)](https://github.com/microsoft/botbuilder-python) | [Примеры](https://github.com/Microsoft/botbuilder-python/tree/master/samples). Пакет SDK для Python теперь поддерживает OAuth, запросы, Cosmos DB, а также включает все основные функции пакета SDK 4.5. Также доступны примеры, которые познакомят вас с новыми возможностями пакета SDK.

### <a name="bot-framework-testing"></a>Тестирование в Bot Framework
- [Документация](https://aka.ms/testing-framework) | Пакеты модульного тестирования ([C#](https://aka.ms/nuget-botbuilder-testing)/ [JavaScript](https://aka.ms/npm-botbuilder-testing)) | [Пример C#](https://aka.ms/cs-core-test-sample) | [Пример JS](https://aka.ms/js-core-test-sample): Учитывая пожелания клиентов и разработчиков касательно более совершенных средств тестирования, в июльскую версию пакета SDK мы добавили новую функцию модульного тестирования. Пакет Microsoft.Bot.Builder.testing упрощает процесс модульного тестирования диалогов в боте.  

- [Тестирование канала](https://github.com/Microsoft/BotFramework-Emulator/releases) | [Документация](https://aka.ms/channel-testing). 

Bot Inspector — это новая функция Bot Framework Emulator, презентация которой состоялась на конференции Microsoft Build 2019. Она позволяет отлаживать и тестировать ботов в таких каналах, как Microsoft Teams, Slack, Кортана и т. д. При использовании бота в определенном канале полученные ботом сообщения будут дублироваться в Bot Framework Emulator, где вы можете их просматривать. Кроме того, можно отобразить моментальный снимок состояния бота на любом шаге обмена данными между каналом и ботом.

### <a name="web-chat"></a>Веб-чат.
- Учитывая пожелания корпоративных клиентов, мы добавили [пример веб-чата](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/19.a.single-sign-on-for-enterprise-apps#single-sign-on-demo-for-enterprise-apps-using-oauth), демонстрирующий, как обеспечить пользователю доступ к ресурсам корпоративного приложения с помощью бота. Взаимодействие OAuth с Microsoft Graph и API GitHub демонстрируется с использованием ресурсов двух типов.

### <a name="solutions"></a>Решения
- [Акселератор решений виртуального помощника](https://github.com/Microsoft/botframework-solutions#readme). Доступен набор шаблонов, акселераторы решений и навыки для создания сложных диалогов. Новый клиент приложений Android для виртуального помощника, который интегрируется с Direct Line Speech и виртуальным помощником, демонстрирует, как клиент на устройстве может взаимодействовать с вашим виртуальным помощником и отображать адаптивные карточки. Обновления также включают поддержку Direct Line Speech и Microsoft Teams.
  
- [Виртуальный агент Dynamics 365 для обслуживания клиентов (общедоступная предварительная версия)](https://dynamics.microsoft.com/en-us/ai/virtual-agent-for-customer-service/). Общедоступная предварительная версия позволяет качественно обслуживать клиентов с помощью интеллектуальных и адаптируемых виртуальных агентов. Специалисты по обслуживанию клиентов могут легко создавать и улучшать ботов, используя аналитические сведения на основе ИИ.
  
- [Chat for Dynamics 365](https://www.powerobjects.com/powerpacks/powerchat/). Решение Chat for Dynamics 365 предлагает несколько возможностей, обеспечивающих эффективное взаимодействие агентов поддержки с пользователями и их высокую производительность. Доступны возможности интерактивного чата и отслеживания диалогов с посетителями вашего веб-сайта, размещенного в Microsoft Dynamics 365.

## <a name="whats-new-may-2019"></a>Что нового (май 2019 г.)

|   | C#  | JS  | Python |  Java | 
|---|:---:|:---:|:------:|:-----:|
|SDK |[4.4.3][1] | [4.4.0][2] | [4.4.0b1 (предварительная версия)][3] | [4.0.0a6 (предварительная версия)][3a]|
|Docs | [docs][5] |[docs][5] |  | |
|Примеры |[.NET Core][6], [WebAPI][10] |[Node.js][7] , [TypeScript][8], [es6][9]  | [Python][11] | | 

<a name="V4-whats-new"></a>
## <a name="bot-framework-sdk-new-in-preview"></a>Пакет SDK Bot Framework (новая предварительная версия)

- [Адаптивные диалоги][47] | [Документация][48] | [Примеры C#][49]. Адаптивные диалоги позволяют разработчикам создавать диалоги, которые могут изменяться динамически в ходе беседы.  Как правило, разработчики планируют весь поток беседы наперед, что ограничивает возможности общения.  Адаптивные диалоги предоставляют больше свободы, позволяя реагировать на изменения в контексте и вставлять новые шаги или целые вложенные диалоги по мере развития беседы. 

- [Создание языка][43] | [Документация][44] | [Примеры C#][45]. Генерирование текста позволяет разработчикам извлекать внедренные строки из файлов ресурсов и кода, а также управлять ими, используя соответствующие среду выполнения и формат файла.  Генерирование текста позволяет клиентам определять разные варианты фраз, выполнять простые выражения на основе контекста и обращаться к сохраненным данным беседы. Со временем мы сможем включить дополнительные возможности, которые помогут сделать процесс общения более естественным.

- [Язык общих выражений][40] | [API][41]. В основе адаптивных диалогов и генерирования текста лежит язык общих выражений, который используется для управления общения с ботами.

## <a name="botkit"></a>Botkit
[Botkit][100] — это средство разработки и пакет SDK для создания ботов, приложений и специализированных решений для интеграции, используемых с основными платформами обмена сообщениями. Боты Botkit используют триггеры `hear()`, вопросы `ask()` и ответы `say()`. Разработчики могут использовать этот синтаксис для создания диалогов, совместимых с последней версией пакета SDK Bot Framework. 

Кроме того, Botkit предоставляет шесть адаптеров платформы, позволяя приложениям ботов JavaScript непосредственно взаимодействовать со следующими платформами обмена сообщениями: [Slack][102], [Webex Teams][103], [Google Hangouts][104], [Facebook Messenger][105], [Twilio][106] и [WebChat][107].

Botkit является компонентом Microsoft Bot Framework, который выпускается под [лицензией открытого программного обеспечения MIT][101].

## <a name="bot-framework-solutions-new-in-preview"></a>Решения Bot Framework (новая предварительная версия)

[Репозиторий решений Bot Framework](https://github.com/Microsoft/AI#readme) — это хранилище наборов шаблонов, акселераторов решений и навыков для создания сложных решений для общения, работающих по принципу помощника.

| Имя | Описание |  
|:------------|:------------| 
|[**Виртуальный помощник**](https://github.com/Microsoft/AI/tree/master/docs#virtual-assistant) | Многие клиенты хотели бы иметь помощника для ведения диалога, который бы представлял конкретную торговую марку, учитывал особенности целевой аудитории и был доступным для широкого ряда устройств и холстов. <br/><br/> Корпоративный шаблон значительно упрощает создание проекта ботов, включая такие компоненты и функции, как основные намерения для реализации общения, интеграция со средством Dispatch, QnA Maker, Application Insights и автоматизированное развертывание.|
|[**Навыки**](https://github.com/Microsoft/AI/blob/master/docs/overview/skills.md)| Разработчики могут создавать решения для общения, комбинируя доступные для повторного использования возможности ведения беседы, называемые навыками. Навыки представляют собой удаленно вызываемые боты. Для упрощения создания навыков предоставляется шаблон разработчика навыков (.NET, TS). 
|[**Аналитика в Application Insights**](https://github.com/Microsoft/AI/blob/master/docs/readme.md#analytics)| Анализируйте сведения о работоспособности и поведении бота с помощью решений аналитики ИИ для общения. Просматривайте доступные данные телеметрии, примеры запросов Application Insights и панели мониторинга Power BI, чтобы оценить общение бота с пользователями. |

## <a name="azure-bot-service"></a>Служба Azure Bot
Служба Azure Bot позволяет размещать интеллектуальных ботов корпоративного уровня, предоставляя права полного владения данными и управления ими. Разработчики могут регистрировать и подключать своих ботов для пользователей таких решений, как Skype, Microsoft Teams, Cortana, WebChat и др. [Azure][27]  |  [Документация][28] | [Подключение к каналам][29]. 

* **Использование клиента Direct Line JS** Если вы хотите использовать канал Direct Line в службе Azure Bot и не используете клиент WebChat, вы можете использовать в своем приложении клиент Direct Line JS. Подробнее см. на [GitHub][30].

<a name="ABS-whats-new"></a>

* **Новые функции! Канал Direct Line Speech**. Мы объединили Bot Framework и Службы речи Майкрософт, предоставляя канал, который поддерживает двунаправленную потоковую передачу речи и текста между клиентом и приложением бота.  См. подробнее о том, как [добавить канал речи для бота](https://docs.microsoft.com/azure/bot-service/directline-speech-bot?view=azure-bot-service-4.0).


## <a name="bot-framework-emulator"></a>Bot Framework Emulator
[Bot Framework Emulator][60] — это кроссплатформенное классическое приложение, которое позволяет разработчикам ботов выполнять тестирование и отладку ботов, созданных с помощью пакета SDK Bot Framework. Bot Framework Emulator можно использовать для тестирования ботов локально на компьютере или удаленно.

- [Скачивание последнего решения][61] | [Документация][62]

<a name="Emulator-whats-new"></a>
### <a name="bot-inspector-new-in-preview"></a>Bot Inspector (новая предварительная версия)

Bot Framework Emulator включает бета-версию нового компонента — Bot Inspector. Это решение для отладки и тестирования ботов, созданных с помощью пакета SDK Bot Framework версии 4 в таких каналах, как Microsoft Teams, Slack, Cortana, Facebook Messenger, Skype и т. д. По мере ведения диалога сообщения будут отображаться в Bot Framework Emulator, где вы можете проверить данные сообщения, полученные ботом. Также отображается моментальный снимок состояния бота на любом шаге обмена данными между каналом и ботом. См. подробнее о [Bot Inspector](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md).


## <a name="related-services"></a>Связанные службы

### <a name="language-understanding"></a>Распознавание речи 
Служба машинного обучения для создания решений с поддержкой естественного языка. Вы можете быстро создавать пользовательские модели корпоративного уровня, которые постоянно совершенствуются. [Служба "Распознавание речи" (LUIS)][30] позволяет приложению распознавать желания людей на основе их собственных слов.

<a name="LUIS-whats-new"></a>

- **Новые функции! Роли, внешние сущности и динамические сущности**. В LUIS добавлено несколько функций, которые позволяют разработчикам извлекать подробные сведения из текста, чтобы пользователи могли создавать интеллектуальные решения с меньшими усилиями. В LUIS также расширены роли для всех типов сущностей, что позволяет классифицировать одни и те же сущности в разные подтипы с учетом контекста. Теперь разработчикам предоставляются расширенные возможности использования LUIS, включая возможность определять и обновлять модели во время выполнения с использованием динамических списков и внешних сущностей. Динамические списки добавляются в сущности списков во время прогноза, обеспечивая точное соответствие сведений о пользователе. Отдельные дополнительные средства извлечения сущностей выполняются с внешними объектами, и эти сведения могут добавляться в LUIS в качестве надежных сигналов для других моделей.

- **Новые функции! Панель мониторинга аналитики**. LUIS предоставляет подробные наглядные панели мониторинга с комплексным представлением аналитики. Удобный интерфейс описывает распространенные проблемы, с которыми сталкивается большинство пользователей при разработке приложений, предоставляя простые рекомендации по их устранению. Так пользователи могут получить больше сведений о качестве моделей, потенциальных проблемах с данными и способах реализации рекомендаций.

[Документация][31] | [Добавление в бота функции распознавания естественного языка][32] 

### <a name="qna-maker"></a>QnA Maker
[QnA Maker][33] — это облачная служба API, которая позволяет работать с данными в форме вопросов и ответов. QnA Maker позволяет за несколько минут создать, обучить и опубликовать простого бота вопросов и ответов, основанного на веб-страницах часто задаваемых вопросов, структурированных документах, руководствах по продуктам и редакторских материалах.

<a name="QnA-whats-new"></a>

- **Новые функции! Конвейер извлечения**. Теперь вы можете извлекать иерархические сведения из URL-адреса, файлов и данных SharePoint.
- **Новые функции! Аналитика**. Модели контекстного ранжирования, предложения для активного обучения.
- **Новые функции! Общение**. Ведение диалогов с несколькими шагами в QnA Maker.

[Документация][34]  | [Добавление в бота QnA Maker][35] 

### <a name="speech-services"></a>Службы "Речь"
[Службы речи](https://docs.microsoft.com/azure/cognitive-services/speech-service/) выполняют преобразование звука в текст, перевод речи и преобразование текста в речь с помощью унифицированных решений. С помощью служб речи можно интегрировать речь в бота, создавать пользовательские слова для активации или создавать содержимое на нескольких языках.

### <a name="adaptive-cards"></a>Адаптивные карточки
[Адаптивные карточки](https://adaptivecards.io) — это открытый стандарт для разработчиков, который позволяет согласованно обмениваться содержимым карточек. Он используются разработчиками Bot Framework для создания удобных решений для общения с поддержкой нескольких каналов.

## <a name="additional-information"></a>Дополнительные сведения
- См. подробнее на [странице GitHub](https://github.com/Microsoft/botframework/blob/master/whats-new.md#whats-new).

[1]:https://github.com/Microsoft/botbuilder-dotnet/#packages
[1a]:https://github.com/microsoft/botframework-sdk/#readme
[2]:https://github.com/Microsoft/botbuilder-js#packages
[3]:https://github.com/Microsoft/botbuilder-python#packages
[3a]:https://github.com/Microsoft/botbuilder-java#packages
[5]:https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0
[6]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore
[7]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs
[8]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/typescript_nodejs
[9]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/javascript_es6
[10]:https://github.com/Microsoft/BotBuilder-Samples/tree/master/samples/csharp_webapi
[11]:https://github.com/Microsoft/botbuilder-python/tree/master/samples

[18]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS#readme
[19]:https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker#readme

[27]:https://azure.microsoft.com/services/bot-service/
[28]:https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[29]:https://docs.microsoft.com/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0

[30]:https://www.luis.ai
[31]:https://docs.microsoft.com/azure/cognitive-services/LUIS/Home
[32]:https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-luis?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=csharp
[33]:https://www.qnamaker.ai/
[34]:https://aka.ms/what-is-qnamaker
[35]:https://docs.microsoft.com/azure/bot-service/bot-builder-howto-qna?view=azure-bot-service-4.0&branch=pr-en-us-1325&tabs=cs

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

[60]:https://github.com/Microsoft/BotFramework-Emulator#readme
[61]:https://github.com/Microsoft/BotFramework-Emulator/releases/latest
[62]:https://docs.microsoft.com/azure/bot-service/bot-service-debug-emulator?view=azure-bot-service-4.0

[100]:https://github.com/howdyai/botkit#readme
[101]:https://github.com/howdyai/botkit/blob/master/LICENSE.md
[102]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-slack#readme
[103]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-webex#readme
[104]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-hangouts#readme
[105]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-facebook#readme
[106]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-twilio-sms#readme
[107]:https://github.com/howdyai/botkit/tree/master/packages/botbuilder-adapter-web#readme

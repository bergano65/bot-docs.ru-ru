---
title: Управление ресурсами бота с помощью файла бота | Документация Майкрософт
description: Описание назначения и применения файла бота.
keywords: bot file, .bot, .bot file, msbot, bot resources, manage bot resources
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/13/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0ff3f0e68d58a8768bb785a88ee7664ab430e453
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645634"
---
# <a name="manage-bot-resources-with-a-bot-file"></a>Управление ресурсами бота с помощью файла бота

Боты часто используют множество разных служб, например [LUIS.ai](https://luis.ai) или [QnaMaker.ai](https://qnamaker.ai). При разработке бота нет универсального расположения для хранения метаданных об используемых службах.  Это мешает нам создать хороший инструментарий, позволяющий работать с ботом как с единым целым.

Чтобы решить эту проблему, мы создали **BOT-файл** для централизованного хранения всех ссылок на службы и поддержки работы с инструментами.  Например, Bot Framework Emulator ([версии 4](https://aka.ms/Emulator-wiki-getting-started)) использует BOT-файл для создания единого представления обо всех подключенных к боту службах.  

В BOT-файле вы можете зарегистрировать такие службы, как:

* **Localhost** — конечные точки локального отладчика;
* [**Служба Azure Bot**](https://azure.microsoft.com/en-us/services/bot-service/) — регистрации службы Azure Bot;
* [**LUIS. AI**](https://www.luis.ai/) — служба LUIS предоставляет боту возможность взаимодействовать с другими пользователями на естественном языке; 
* [**QnA Maker**](https://qnamaker.ai/) — позволяет за несколько минут создать, обучить и опубликовать простой бот вопросов и ответов, основанный на веб-страницах часто задаваемых вопросов, структурированных документах и (или) редакторских материалах;
* [**Dispatch**](https://github.com/Microsoft/botbuilder-tools/tree/master/Dispatch) — организует диспетчеризацию между несколькими службами;
* [**Azure Application Insights**](https://azure.microsoft.com/en-us/services/application-insights/) — предоставляет полезные сведения и аналитическую информацию по работе бота;
* [**Хранилище BLOB-объектов Azure**](https://azure.microsoft.com/en-us/services/storage/blobs/) — длительное сохранение состояний бота; 
* [**Azure Cosmos DB**](https://azure.microsoft.com/en-us/services/cosmos-db/) — глобально распространенная многомодельная служба базы данных для сохранения состояний бота.

Помимо перечисленных, бот может использовать любые пользовательские службы. Концепция [Универсальная служба](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/add-services.md) позволяет подключить конфигурацию любой стандартной службы.

## <a name="when-is-a-bot-file-created"></a>Когда создается BOT-файл 
- Если вы создаете бот с помощью [службы Azure Bot](https://ms.portal.azure.com/#blade/Microsoft_Azure_Marketplace/GalleryResultsListBlade/selectedSubMenuItemId/%7B%22menuItemId%22%3A%22gallery%2FCognitiveServices_MP%2FBotService%22%2C%22resourceGroupId%22%3A%22%22%2C%22resourceGroupLocation%22%3A%22%22%2C%22dontDiscardJourney%22%3Afalse%2C%22launchingContext%22%3A%7B%22source%22%3A%5B%22GalleryFeaturedMenuItemPart%22%5D%2C%22menuItemId%22%3A%22CognitiveServices_MP%22%2C%22subMenuItemId%22%3A%22BotService%22%7D%7D), для вас автоматически создается BOT-файл с готовым списком подключенных служб. По умолчанию BOT-файл шифруется.
- Если вы создаете бот с помощью [шаблона](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4) для Visual Studio из пакета SDK Bot Builder версии 4 или с помощью [генератора Yeoman](https://www.npmjs.com/package/generator-botbuilder) из Bot Builder, BOT-файл создается автоматически. В этом потоке подключенные службы не подготавливаются и файл бота не шифруется.
- Если вы начинаете работу с [образцами BotBuilder-samples](https://github.com/Microsoft/botbuilder-samples), каждый образец для пакета SDK Bot Builder версии 4 содержит готовый незашифрованный BOT-файл. 
- Также файл бота можно создать с помощью средства [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md).

## <a name="what-does-a-bot-file-look-like"></a>Что представляет собой файл бота 
Давайте рассмотрим пример [BOT-файла](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json).
Сведения о шифровании и расшифровке содержимого BOT-файла вы найдете в статье о [секретах ботов](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/bot-file-encryption.md).
## <a name="why-do-i-need-a-bot-file"></a>Зачем нужен BOT-файл

BOT-файл **не** является обязательным при создании бота с помощью пакета SDK Bot Builder. Вы можете по-прежнему использовать файлы appsettings.json, web.config, env, хранилище ключей или любой другой механизм, который считаете подходящим для хранения ссылок на службы и ключей, которыми пользуется ваш бот. Но BOT-файл является обязательным для тестирования бота с помощью эмулятора. Рады сообщить, что эмулятор может самостоятельно создавать BOT-файл для тестирования. Для этого запустите эмулятор, щелкните ссылку **Create a new bot configuration** (Создать новую конфигурацию бота) на странице приветствия. В появившемся диалоговом окне введите **имя бота** и **URL-адрес конечной точки**. Теперь запустите подключение.

Ниже перечислены преимущества использования BOT-файла.
- Стандартный способ для хранения ресурсов, не зависящий от используемых языков и платформ.   
- Bot Framework Emulator и средства CLI используют только этот стандартизированный формат (BOT-файл), благодаря чему отлично отслеживают подключенные службы. 
- Элегантное решение инструментирования для создания служб и управления ими достаточно сложно создать без хорошо определенной (в BOT-файле) схемы.  


## <a name="using-bot-file-in-your-bot-builder-sdk-bot"></a>Применение BOT-файла в боте, созданном на основе SDK Bot Builder
BOT-файл позволяет получить сведения о конфигурации служб прямо из кода бота. Библиотека BotFramework-Configuration, которая предоставляется для [C#](https://www.nuget.org/packages/Microsoft.Bot.Configuration) и [JS](https://www.npmjs.com/package/botframework-config), поможет вам загрузить BOT-файл и поддерживает несколько методов поиска и получения сведений о конфигурации нужных служб.

## <a name="additional-resources"></a>Дополнительные ресурсы
Воспользуйтесь файлом сведений об [MSBot](https://github.com/Microsoft/botbuilder-tools/blob/master/packages/MSBot/README.md), чтобы больше узнать об использовании файла бота.

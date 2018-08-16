---
title: Приступая к созданию ботов с помощью Bot Builder | Документация Майкрософт
description: Начните создавать мощные боты с помощью Bot Builder.
author: robstand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 21186c5d3b0769311e4703ca1dab2f48a0a0081a
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574890"
---
# <a name="develop-bots-with-bot-builder"></a>Разработка ботов с помощью Bot Builder

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Bot Builder предоставляет пакет SDK, библиотеки, примеры, а также инструменты для создания и отладки ботов. Пакет SDK Bot Builder используется при создании бота с помощью службы Bot. Кроме того, этот пакет SDK позволяет с нуля создать бот на C# или Node.js. Bot Builder содержит Bot Framework Emulator для тестирования ботов и инспектор каналов для предварительного просмотра интерфейса вашего бота на различных каналах.

С помощью REST API можно создать бот на любом языке.

## <a name="bot-builder-sdk-for-net"></a>Пакет SDK Bot Builder для .NET
В пакете SDK Bot Builder для .NET используется C#, что позволяет разработчикам .NET создавать боты привычным способом. Это мощная платформа для создания ботов, которые могут взаимодействовать с пользователем в свободной форме и обеспечивают более интерактивную беседу, где пользователю предлагается выбрать одно из возможных значений. 

Пошаговые инструкции по созданию ботов с помощью пакета SDK Bot Builder для .NET описаны в [кратком руководстве по .NET](~/dotnet/bot-builder-dotnet-quickstart.md).

Пакет SDK Bot Builder для .NET доступен в виде [пакета NuGet](https://www.nuget.org/packages/Microsoft.Bot.Builder/).

> [!IMPORTANT]
> Для пакета SDK Bot Builder для .NET требуется .NET Framework 4.6 или более поздней версии. Если вы добавляете пакет SDK в существующий проект, предназначенный для более ранней версии .NET Framework, нужно сначала обновить проект, чтобы он предназначался для .NET Framework 4.6.

Чтобы установить пакет SDK в существующий проект Visual Studio, выполните следующие действия:

1. В **обозревателе решений** щелкните правой кнопкой мыши имя проекта и выберите пункт **Управление пакетами NuGet…**.
2. На вкладке **Обзор** введите Microsoft.Bot.Builder в поле поиска.
3. Выберите **Microsoft.Bot.Builder** в списке результатов, щелкните **Установить** и примените изменения.

Также можно скачать [исходный код](https://github.com/Microsoft/BotBuilder/tree/master/CSharp) пакета SDK Bot Builder для .NET с сайта GitHub.

### <a name="visual-studio-project-templates"></a>Шаблоны проектов Visual Studio
Скачайте шаблоны проектов Visual Studio, чтобы ускорить разработку ботов:

* [Шаблон бота для Visual Studio][bot-template], который позволяет разрабатывать боты с помощью C#.
* [Шаблон навыка Кортаны для Visual Studio][cortana-template], который позволяет разрабатывать навыки Кортаны с помощью C#.

> [!TIP]
> <a href="/visualstudio/ide/how-to-locate-and-organize-project-and-item-templates" target="_blank">Узнайте больше</a> о том, как установить шаблоны проектов Visual Studio 2017.

## <a name="bot-builder-sdk-for-nodejs"></a>Пакет SDK Bot Builder для Node.js
Пакет SDK Bot Builder для Node.js позволяет разработчикам Node.js создавать боты привычным способом. С помощью этого пакета можно создавать различные пользовательские интерфейсы диалогов — от простых запросов до диалогов в свободной форме. Для создания ботов веб-сервера в пакете SDK Bot Builder для Node.js используется Restify — популярная платформа, которая позволяет создавать веб-службы. Пакет SDK также совместим с Express. 

Пошаговые инструкции по созданию ботов с помощью пакета SDK Bot Builder для Node.js приведены в [кратком руководстве по Node.js](~/nodejs/bot-builder-nodejs-quickstart.md). 

Пакет SDK Bot Builder для Node.js доступен в виде пакета npm. Чтобы установить пакет SDK Bot Builder для Node.js и его зависимости, сначала создайте папку для вашего бота, перейдите к ней и выполните следующую команду **npm**:

```nodejs
npm init
```

Затем установите пакет SDK Bot Builder для Node.js и модули <a href="http://restify.com/" target="_blank">Restify</a>, выполнив следующие команды **npm**:

```nodejs
npm install --save botbuilder
npm install --save restify
```

Также можно скачать [исходный код](https://github.com/Microsoft/BotBuilder/tree/master/Node) пакета SDK Bot Builder для Node.js с сайта GitHub.

## <a name="rest-api"></a>REST API

Вы можете создать бот на любом языке с помощью REST API для Bot Framework. Пошаговые инструкции по созданию ботов с помощью REST описаны в [кратком руководстве по REST](rest-api/bot-framework-rest-connector-quickstart.md).

В Bot Framework существует три интерфейса REST API:

 - [REST API для Bot Connector][connectorAPI] позволяет боту отправлять и получать сообщения по каналам, настроенным на [портале Bot Framework](https://dev.botframework.com/). 

- [REST API для службы "Состояние бота"][stateAPI] дает боту возможность сохранять и извлекать данные о состоянии, связанные с диалогами, которые выполняются через REST API для Bot Connector.

- [REST API для Direct Line][directLineAPI] позволяет напрямую подключить к одному боту свое клиентское приложение, например элемент управления "Веб-чат" или мобильное приложение.

## <a name="bot-framework-emulator"></a>Bot Framework Emulator
Bot Framework Emulator — это классическое приложение, которое дает возможность локально или удаленно тестировать и отлаживать боты. С помощью эмулятора вы можете общаться с ботом и проверять сообщения, которые он отправляет и получает. [Узнайте больше о Bot Framework Emulator](~/bot-service-debug-emulator.md) и [скачайте это приложение](http://emulator.botframework.com) для своей платформы.

## <a name="bot-framework-channel-inspector"></a>Инспектор каналов Bot Framework
С помощью [инспектора каналов](bot-service-channel-inspector.md) Bot Framework вы можете быстро просмотреть, как будет функционировать бот с различными каналами.

[bot-template]: http://aka.ms/bf-bc-vstemplate
[cortana-template]: https://aka.ms/bf-cortanaskill-template


[connectorAPI]: https://docs.botframework.com/en-us/restapi/connector/#navtitle
 
[stateAPI]: https://docs.botframework.com/en-us/restapi/state/#navtitle

[directLineAPI]: https://docs.botframework.com/en-us/restapi/directline3/#navtitle

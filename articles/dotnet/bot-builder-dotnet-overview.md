---
title: Пакет SDK Bot Framework для .NET — Служба Azure Bot
description: Начало работы с пакетом SDK Bot Framework для .NET, который предоставляет простую и мощную платформу для создания ботов.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ffd8b9381ba0e7624706a020f654ac6cc9d5b6eb
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75797297"
---
# <a name="bot-framework-sdk-for-net"></a>Пакет SDK Bot Framework для .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Пакет SDK Bot Framework для .NET — это мощная платформа для создания ботов, которые могут взаимодействовать с пользователем в свободной форме и с подсказками, позволяя пользователю выбирать возможные значения. Эта платформа является простой и использует C#, что позволяет разработчикам .NET создавать боты привычным способом.

С помощью пакета SDK вы можете создавать боты и пользоваться преимуществами следующих компонентов этого пакета: 

- мощная система диалогов, которые являются изолированными и поддерживают объединение;
- встроенные запросы на ввод таких простых элементов, как ответы "Да" или "Нет", строки, числа и перечисления;
- встроенные диалоги, которые используют мощные системы искусственного интеллекта, например <a href="http://luis.ai" target="_blank">LUIS</a>;
- FormFlow для автоматического создания бота (на основе класса C#), который ведет диалог с пользователем для предоставления справки, навигации, уточнений и получения подтверждения, если это необходимо.

> [!IMPORTANT]
> 31 июля 2017 г. в протокол безопасности Bot Framework внесены критические изменения. Чтобы эти изменения не повлияли на ваш бот негативно, убедитесь, что приложение использует пакет SDK Bot Framework 3.5 или более поздней версии. Если для создания бота вы использовали пакет SDK, полученный до 5 января 2017 г. (дата выпуска пакета SDK Bot Framework версии 3.5), обязательно обновите этот пакет SDK до версии 3.5 или выше.

## <a name="get-the-sdk"></a>Получение пакета SDK

Пакет SDK доступен в виде пакета NuGet и в качестве открытого кода на сайте <a href="https://github.com/Microsoft/BotBuilder" target="_blank">GitHub</a>.

> [!IMPORTANT]
> Для использования пакета SDK Bot Framework для .NET требуется .NET Framework 4.6 или более поздней версии. Если вы добавляете пакет SDK в существующий проект, предназначенный для более ранней версии .NET Framework, нужно сначала обновить проект, чтобы он предназначался для .NET Framework 4.6.

Чтобы установить пакет SDK в проект Visual Studio, выполните следующие действия:

1. В **обозревателе решений** щелкните правой кнопкой мыши имя проекта и выберите пункт **Управление пакетами NuGet…**
2. На вкладке **Обзор** введите Microsoft.Bot.Builder в поле поиска.
3. Выберите **Microsoft.Bot.Builder** в списке результатов, щелкните **Установить** и примените изменения.

## <a name="get-code-samples"></a>Получение примеров кода

Этот пакет SDK содержит [пример исходного кода](bot-builder-dotnet-samples.md), в котором используются компоненты пакета SDK Bot Framework для .NET.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о создании ботов с помощью пакета SDK Bot Framework для .NET, ознакомившись со статьями в этом разделе в следующем порядке:

- [Быстрый запуск](bot-builder-dotnet-quickstart.md): Пошаговые инструкции по быстрому созданию и тестированию простого бота.
- [Основные понятия](bot-builder-dotnet-concepts.md). Сведения о важнейших концепциях, которые используются в пакете SDK Bot Framework для .NET.

Если у вас возникли проблемы или есть предложения по пакету SDK Bot Framework для .NET, см. [список доступных ресурсов](../bot-service-resources-links-help.md). 

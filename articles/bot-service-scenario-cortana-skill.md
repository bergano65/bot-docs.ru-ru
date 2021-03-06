---
title: Сценарий бота навыков Кортаны — Служба Azure Bot
description: Изучите сценарий бота навыков Кортаны с помощью Bot Framework.
author: BrianRandell
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2b7d960514970c0d04d9ff37550874dc9de69cde
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2020
ms.locfileid: "77441235"
---
# <a name="cortana-skills-bot-scenario"></a>Сценарий бота навыков Кортаны

[!INCLUDE [pre-release-label](includes/pre-release-label-v3.md)]

Бот навыков Кортаны расширяет возможности Кортаны, чтобы упростить заказ обслуживания автомобиля с мобильного устройства с помощью голоса с применением контекста из календаря.

Кортана — это личный помощник. Используя естественный интерфейс голоса и пользовательский бот навыков Кортаны, вы можете попросить Кортану обменяться данными с организацией, например с магазином автозапчастей, чтобы назначить встречу. Служба может предоставить список услуг, данные о доступном времени и длительности. Кортана может просмотреть календарь, чтобы определить, не назначены ли события на то же время. Если таких событий нет, Кортана создаст встречу и добавит ее в календарь.

![Схема бота навыков Кортаны](~/media/scenarios/bot-service-scenario-cortana-skill.png)

Ниже приведен логический поток действий бота навыков Кортаны для магазина автозапчастей:

1. Пользователь открывает Кортану на компьютере или мобильном устройстве.
2. С помощью текстовых и голосовых команд пользователь запрашивает встречу со специалистом по обслуживанию автомобилей.
3. Так как бот интегрирован с Кортаной, у него есть доступ к календарю пользователя, и он применяет логику к запросу.
4. Используя эту информацию, бот может отправить в службу ремонта автомобилей запрос относительно допустимым встреч.
5. С учетом различных контекстных вариантов пользователь может назначить встречу.
6. Служба Application Insights собирает телеметрию среды выполнения, чтобы предоставить разработчикам сведения о производительности бота и его использовании.

## <a name="sample-bot"></a>Пример бота
Для бота навыков Кортаны важен личный контекст. Используя Кортану, вы можете произнести вслух запрос на обслуживание вашего автомобиля с мобильного устройства у Боба в зависимости от своего расположения. С помощью личных сведений, предоставляемых в Кортане, бот может подтвердить ваше расположение в зависимости от того, где вы находитесь, произнося запрос для бота.

Можно скачать или клонировать исходный код для этого примера бота из репозитория [Примеры для распространенных сценариев Bot Framework](https://aka.ms/abs-scenarios).

## <a name="components-youll-use"></a>Используемые компоненты
Бот Кортаны использует следующие компоненты:
-   Кортана
-   Application Insights

### <a name="cortana"></a>Кортана
Теперь вы можете добавить поддержку для своего бота, создав навык Кортаны. Чтобы создать функции (называемые навыками) для помощника Кортана, используется набор навыков Кортаны. Навык — это конструкция, расширяющая возможности Кортаны. Вы создадите навыки для интеграции с ботом, позволяя Кортане выполнять задачи. В рамках процесса вызова Кортана может (с согласия пользователя) передавать сведения о пользователе в навык во время выполнения, чтобы навык смог соответствующим образом настроить свою работу. Контекстные знания Кортаны позволяют обеспечить пригодность бота и даже его интеллектуальные возможности. После вызова определенные типы навыков могут управлять интерфейсом Кортаны для обмена данными между навыком и пользователем. После публикации пользователи могут просмотреть и использовать навык в Кортане для Windows 10 Anniversary Update + (Desktop и Mobile), iOS и Android.

### <a name="application-insights"></a>Application Insights
Application Insights помогает получать полезные аналитические сведения с помощью средств управления производительностью приложений и мгновенной аналитики. Готовые широкие возможности мониторинга производительности, усовершенствованные предупреждения и удобные в использовании панели мониторинга помогут обеспечить доступность и надежную работу бота. Можно быстро определить наличие проблемы и выполнить анализ основных причин для ее поиска и устранения.

## <a name="next-steps"></a>Дальнейшие действия
Узнайте о сценарии бота для повышения производительности на предприятии.

> [!div class="nextstepaction"]
> [Сценарий бота для повышения производительности на предприятии](bot-service-scenario-enterprise-productivity.md)

---
title: Анализ возможностей общения с помощью Power BI | Документация Майкрософт
description: Сведения о том, как с помощью шаблона Enterprise Bot использовать Application Insights, чтобы получать аналитические сведения из Power BI.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 88208a2f5b0eb88d3b2964e63a21585484166d73
ms.sourcegitcommit: 2d84d5d290359ac3cfb8c8f977164f799666f1ab
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2019
ms.locfileid: "54152177"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>Шаблон Enterprise Bot — анализ возможностей общения с помощью панели мониторинга Power BI и Application Insights

> [!NOTE]
> Приведенные здесь сведения относятся к пакету SDK версии 4. 

Когда бот будет развернут и начнет обрабатывать сообщения, вы увидите данные телеметрии, поступающие в экземпляр Application Insights в группе ресурсов. 

Эти данные телеметрии можно просмотреть в колонке Application Insights на портале Azure и с помощью Log Analytics. Кроме того, эти данные телеметрии можно использовать в Power BI, чтобы получить более общие аналитические бизнес-сведения об использовании бота.

См. пример [панели мониторинга PowerBI](https://aka.ms/botPowerBiTemplate). 

Пример предоставлен для ознакомления. В нем показано, как можно приступить к созданию собственных аналитических сведений. Со временем мы усовершенствуем эти визуализации. 


## <a name="middleware-processing"></a>Обработка ПО промежуточного слоя

Оболочки телеметрии для классов QnAMaker и LuisRecognizer обеспечивают согласованность выходных данных телеметрии независимо от сценария и работу стандартной панели мониторинга в каждом проекте.

```TelemetryLuisRecognizer``` и ```TelemetryQnAMaker``` предоставляют свойства конструктора, которые позволяют разработчику отключить запись имен пользователей и исходных сообщений в журнал. Это позволит сократить объем доступных аналитических сведений.

## <a name="telemetry-captured"></a>Сбор данных телеметрии

С помощью ```TelemetryLuisRecognizer``` и ```TelemetryQnAMaker```, которые включены по умолчанию в шаблоне Enterprise, записываются 4 отдельных события телеметрии. 

К каждому намерению LUIS, которое используется в проекте, будет добавлен префикс LuisIntent. Это упростит распознавание намерений для панели мониторинга.

```
-BotMessageReceived
    - ActivityId
    - Channel
    - FromId
    - FromName
    - ConversationId
    - ConversationName
    - Locale
    - Text
    - RecipientId
    - RecipientName
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - ConversationId
    - ConversationName
    - Locale
    - RecipientId
    - RecipientName
    - ReplyToId
    - Text
```

```
- LuisIntent.*
    - ActivityId
    - Intent
    - IntentScore
    - SentimentLabel
    - SentimentScore
    - ConversationId
    - Question
    - DialogId
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - FromName
    - ArticleFound
    - Question
    - Answer
    - Score
```
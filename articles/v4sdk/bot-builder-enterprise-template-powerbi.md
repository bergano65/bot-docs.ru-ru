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
ms.openlocfilehash: 9c5a156ff949c4014b69638e9cd5abb4b0ede149
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999395"
---
# <a name="enterprise-bot-template---conversational-analytics-using-powerbi-dashboard-and-application-insights"></a>Шаблон Enterprise Bot — анализ возможностей общения с помощью панели мониторинга Power BI и Application Insights

> [!NOTE]
> Приведенные здесь сведения относятся к пакету SDK версии 4. 

Когда бот будет развернут и начнет обрабатывать сообщения, вы увидите данные телеметрии, поступающие в экземпляр Application Insights в группе ресурсов. 

Эти данные телеметрии можно просмотреть в колонке Application Insights на портале Azure и с помощью Log Analytics. Кроме того, эти данные телеметрии можно использовать в Power BI, чтобы получить более общие аналитические бизнес-сведения об использовании бота.

Пример панели мониторинга Power BI можно найти в папке Power BI созданного проекта. Пример предоставлен для ознакомления. В нем показано, как можно приступить к созданию собственных аналитических сведений. Со временем мы усовершенствуем эти визуализации. 

## <a name="getting-started"></a>Приступая к работе

- Скачайте Power BI Desktop [по этой ссылке](https://powerbi.microsoft.com/en-us/desktop/).
 
- Получите значение ```Application Id``` для ресурса Application Insights, который используется ботом. Для этого перейдите на страницу "Доступ через API" из раздела "Настройка" в колонке Application Insights на портале Azure.

Дважды щелкните предоставленный файл шаблона Power BI, расположенный в папке Power BI вашего решения. Вам будет предложено ввести значение для ```Application Id```, полученное на предыдущем шаге. Пройдите аутентификацию при появлении соответствующего запроса, используя свои учетные данные подписки Azure. Возможно, для входа вам потребуется выбрать параметр "Учетная запись организации".

Панель мониторинга теперь связана с экземпляром Application Insights. На ней вы увидите начальные аналитические сведения, если бот отправлял и получал сообщения.

>Обратите внимание, что визуализация тональности не будет отображаться, так как сейчас скрипт развертывания не поддерживает такую возможность при публикации модели LUIS. Если вы [повторно опубликуете](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-how-to-publish-app) модель LUIS и включите анализ тональности, функция будет работать.

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
    - Conversationid
    - ConversationName
    - Locale
    - UserName
    - Text
```
  
```
-BotMessageSent
    - ActivityId,
    - Channel
    - RecipientId
    - Conversationid
    - ConversationName
    - Locale
    - ReceipientName
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
```

```
- QnAMaker
    - ActivityId
    - ConversationId
    - OriginalQuestion
    - UserName
    - QnAItemFound
    - Question
    - Answer
    - Score
```
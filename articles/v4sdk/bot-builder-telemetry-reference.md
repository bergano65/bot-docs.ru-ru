---
title: События, создаваемые телеметрией службы Bot Framework | Документация Майкрософт
description: Узнайте, какие события активируются в новых функциях телеметрии.
keywords: telemetry, appinsights, monitor bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 929432d51a7bdfbcbd328ae7ea4e0df8873e7392
ms.sourcegitcommit: 23a1808e18176f1704f2f6f2763ace872b1388ae
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/25/2019
ms.locfileid: "68484361"
---
# <a name="events-generated-by-the-bot-framework-service-telemetry"></a>События, создаваемые телеметрией службы Bot Framework

## <a name="channel-service-events"></a>События службы канала

Наряду с классом `WaterfallDialog`, который обсуждается в разделе, посвященном [телеметрии](bot-builder-telemetry.md) и который создает события в коде бота, служба канала Bot Framework также регистрирует в журнале события.  Это помогает диагностировать проблемы с работой каналов или общие сбои бота.

### <a name="customevent-activity"></a>CustomEvent: "Activity"
**Откуда регистрируется:** Служба канала. Регистрируется службой канала при получении сообщения.

### <a name="exception-bot-errors"></a>Исключение: "Bot Errors"
**Откуда регистрируется:** Служба канала. Регистрируется каналом, когда вызов к боту возвращает ответ с кодом HTTP, отличным от 2XX.

## <a name="customevent-waterfallstart"></a>CustomEvent: "WaterfallStart" 

В начале WaterfallDialog регистрируется событие `WaterfallStart`.

- `user_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `session_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Это идентификатор dialogId в строковом формате, переданный в каскад.  Его можно рассматривать как тип каскада)
- `customDimensions.InstanceID` (уникальное значение для каждого экземпляра диалога)

## <a name="customevent-waterfallstep"></a>CustomEvent: "WaterfallStep" 

Регистрирует отдельные шаги из каскадного диалога.

- `user_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `session_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Это идентификатор dialogId в строковом формате, переданный в каскад.  Его можно рассматривать как тип каскада)
- `customDimensions.StepName` (содержит имя метода или `StepXofY`, если это лямбда-выражение)
- `customDimensions.InstanceID` (уникальное значение для каждого экземпляра диалога)

## <a name="customevent-waterfalldialogcomplete"></a>CustomEvent: "WaterfallDialogComplete"

Регистрирует завершение каскадного диалога.

- `user_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `session_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Это идентификатор dialogId в строковом формате, переданный в каскад.  Его можно рассматривать как тип каскада)
- `customDimensions.InstanceID` (уникальное значение для каждого экземпляра диалога)

## <a name="customevent-waterfalldialogcancel"></a>CustomEvent: "WaterfallDialogCancel" 

Регистрирует отмену каскадного диалога

- `user_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `session_id` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.activityType` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.channelId` ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- `customDimensions.DialogId` (Это идентификатор dialogId в строковом формате, переданный в каскад.  Его можно рассматривать как тип каскада)
- `customDimensions.StepName` (содержит имя метода или `StepXofY`, если это лямбда-выражение)
- `customDimensions.InstanceID` (уникальное значение для каждого экземпляра диалога)

## <a name="customevent-botmessagereceived"></a>CustomEvent: BotMessageReceived 
Запись в журнал выполняется, когда бот получает новое сообщение от пользователя.

Если переопределение не выполняется, это событие записывается в журнал из `Microsoft.Bot.Builder.TelemetryLoggerMiddleware` с помощью метода `Microsoft.Bot.Builder.IBotTelemetry.TrackEvent()`.

- Идентификатор сеанса  
  - Если вы используете Application Insights, запись в журнал выполняется из `TelemetryBotIdInitializer` в качестве идентификатора **сеанса** (*Temeletry.Context.Session.Id*) для Application Insights.  
  - Соответствует [идентификатору беседы](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `session_id`.

- Идентификатор пользователя
  - Если вы используете Application Insights, запись в журнал выполняется из `TelemetryBotIdInitializer` в качестве идентификатора **пользователя** (*Temeletry.Context.User.Id*) для Application Insights.  
  - Значение этого свойства представляет собой объединенные значения [идентификатора канала](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#channel-id) и [идентификатора пользователя](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `user_id`.

- Идентификатор действия 
  - Если вы используете Application Insights, запись в журнал выполняется из `TelemetryBotIdInitializer` в качестве свойства, добавляемого в событие.
  - Соответствует [идентификатору действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#Id), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `activityId`.

- Идентификатор канала
  - Если вы используете Application Insights, запись в журнал выполняется из `TelemetryBotIdInitializer` в качестве свойства, добавляемого в событие.  
  - Соответствует [идентификатору канала](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#id), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `channelId`.

- ActivityType 
  - Если вы используете Application Insights, запись в журнал выполняется из `TelemetryBotIdInitializer` в качестве свойства, добавляемого в событие.  
  - Соответствует [типу действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#type), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `activityType`.

- текст
  - Запись в журнал выполняется **при необходимости**, если свойству `logPersonalInformation` задано значение `true`.
  - Соответствует [способу описания действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#text), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `text`.

- Speak

  - Запись в журнал выполняется **при необходимости**, если свойству `logPersonalInformation` задано значение `true`.
  - Соответствует [способу озвучивания действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#speak), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `speak`.

  - 

- FromId
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromId`.

- FromName
  - Запись в журнал выполняется **при необходимости**, если свойству `logPersonalInformation` задано значение `true`.
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

- RecipientId
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

- RecipientName
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

- ConversationId
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

- ConversationName
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

- Языковой стандарт
  - Соответствует [источнику действия](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#from), как определено протоколом Bot Framework.
  - В журнал записывается имя свойства `fromName`.

## <a name="customevent-botmessagesend"></a>CustomEvent: BotMessageSend 
**Откуда регистрируется:** TelemetryLoggerMiddleware 

Регистрируется, когда бот отправляет сообщение.

- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- SessionID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ReplyToID
- RecipientId
- ConversationName
- Языковой стандарт
- RecipientName (необязательно для персональных данных)
- Text (необязательно для персональных данных)
- Speak (необязательно для персональных данных)


## <a name="customevent-botmessageupdate"></a>CustomEvent: BotMessageUpdate
**Откуда регистрируется:** TelemetryLoggerMiddleware. Регистрируется, когда бот обновляет сообщение (редкий случай).
- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- SessionID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName
- Языковой стандарт
- Text (необязательно для персональных данных)


## <a name="customevent-botmessagedelete"></a>CustomEvent: BotMessageDelete
**Откуда регистрируется:** TelemetryLoggerMiddleware. Регистрируется, когда бот удаляет сообщение (редкий случай).
- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- SessionID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- RecipientId
- ConversationId
- ConversationName

## <a name="customevent-luisevent"></a>CustomEvent: LuisEvent
**Откуда регистрируется:** LuisRecognizer

Регистрирует результаты, полученные от службы LUIS.

- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- SessionID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ApplicationId
- Намерение
- IntentScore
- Intent2 
- IntentScore2 
- FromId
- SentimentLabel
- SentimentScore
- Сущности (в виде JSON)
- Question (необязательно для персональных данных)

## <a name="customevent-qnamessage"></a>CustomEvent: QnAMessage
**Откуда регистрируется:** QnA Maker

Регистрирует результаты, полученные от службы QnA Maker.

- UserID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- SessionID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityID ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Channel ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- ActivityType ([из инициализатора телеметрии](https://aka.ms/telemetry-initializer))
- Username (необязательно для персональных данных)
- Question (необязательно для персональных данных)
- MatchedQuestion
- QuestionId
- Ответ
- Оценка
- ArticleFound


---
title: Аналитические сведения бота | Документация Майкрософт
description: Узнайте, как использовать сбор и анализ данных для оптимизации бота с помощью аналитики в Bot Framework.
keywords: bot analytics, application insights, traffic, latency, integrations, AppInsights
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 12/04/2018
ms.openlocfilehash: 2f7474500af4305f4c51193a2a5af264d419569b
ms.sourcegitcommit: 91156d0866316eda8d68454a0c4cd74be5060144
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/07/2018
ms.locfileid: "53010519"
---
# <a name="bot-analytics"></a>Аналитика бота

Analytics — это расширение [Application Insights](/azure/application-insights/app-insights-analytics). Application Insights предоставляет данные **уровня службы** и данные инструментирования, например трафик, задержку и интеграции. Analytics поддерживает отчеты **уровня общения** для данных пользователя, сообщений и каналов.

## <a name="view-analytics-for-a-bot"></a>Просмотр аналитики для бота

Чтобы получить доступ к Analytics, откройте бот на портале Azure и щелкните **Analytics**.

Слишком много данных? Вы можете [включить и настроить выборку](/azure/application-insights/app-insights-sampling) для службы Application Insights, связанной с ботом. Это позволяет сократить объем трафика телеметрии и хранимых данных, сохраняя статистически верный анализ.

### <a name="specify-channel"></a>Указание канала

Выберите, какие каналы отображаются на схемах ниже. Обратите внимание, что если бот не включен на канале, данные этого канала будут отсутствовать.

![Выбор канала](~/media/analytics-channels.png)

* Установите флажок, чтобы включить канал в диаграмму.
* Снимите флажок, чтобы удалить канал из диаграммы.

### <a name="specify-time-period"></a>Указание периода времени

Анализ доступен только за последние 90 дней. Сбор данных начинается после включения Application Insights.

![Выбор периода времени](~/media/analytics-timepick.png)

Щелкните стрелку раскрывающегося меню и выберите количество времени отображения диаграммы.
Обратите внимание, что изменение общего интервала времени приведет к соответствующему изменению шага приращения времени (ось X) на диаграммах.

### <a name="grand-totals"></a>Общий итог

Общее число активных пользователей и действий, отправленных и полученных в течение заданного интервала времени.
Дефисы `--` указывают на отсутствие действий.

### <a name="retention"></a>Сохранение

При хранении отслеживается, сколько пользователей, отправивших одно сообщение, вернулись позже и отправили еще одно.
Диаграмма — это скользящее 10-дневное окно; результаты не изменяются при изменении интервала времени.

![Диаграмма хранения](~/media/analytics-retention.png)

Обратите внимание, что последняя возможная дата — два дня назад. Пользователь отправил сообщение позавчера и *вернулся* вчера.

### <a name="user"></a>Пользователь

На диаграмме пользователей отслеживается, сколько пользователей получили доступ к боту с помощью каждого канала за указанный интервал времени.

![Диаграмма пользователей](~/media/analytics-users.png)

* На процентной диаграмме указано процентное соотношение пользователей, использовавших каждый канал.
* На линейном графике показано, сколько пользователей получали доступ к боту за определенное время.
* В условных обозначениях линейного графика указано цветовое представление каждого канала и общее число пользователей во время заданного периода времени.

### <a name="activities"></a>Действия

На диаграмме действий отслеживается количество действий, отправленных и полученных с помощью каждого канала за указанный интервал времени.

![Диаграмма действий](~/media/analytics-activities.png)

* На процентной диаграмме показано процентное соотношение сообщений, переданных по каждому каналу.
* На линейном графике показано количество действий, полученных и отправленных за указанный интервал времени.
* В условных обозначениях линейного графика указано цветовое линейное представление каждого канала и общее число действий, отправленных и полученных на этом канале во время заданного периода времени.

## <a name="enable-analytics"></a>Включение аналитики

Чтобы расширение Analytics стало доступно, нужно включить и настроить Application Insights. После включения Application Insights начнет собирать данные. Например, если служба Application Insights включена неделю назад для бота, созданного шесть месяцев назад, он соберет данные за неделю.

> [!NOTE]
> Analytics требуется [ресурс](/azure/application-insights/app-insights-create-new-resource) подписки Azure и Application Insights.
Чтобы получить доступ к Application Insights, откройте бот на [портале Azure](https://portal.azure.com/) и щелкните **Параметры**.

Application Insights можно добавить при создании ресурса бота.

Также ресурс Application Insights можно создать и добавить к боту позднее.

1. Создайте ресурс [Application Insights](/azure/application-insights/app-insights-create-new-resource).
2. Откройте бот на панели мониторинга. Щелкните **Параметры** и прокрутите вниз до раздела **Analytics**.
3. Введите сведения для подключения бота к Application Insights. Все поля обязательные.

![Подключение Insights](~/media/analytics-enable.png)

<!--Snip: As of 12/04/2018, parts of this appear to be out of date. However, ~/bot-service-resources-app-insights-keys.md appears to be up to date.

### AppInsights Instrumentation Key

To find this value, open the Application Insights resource for your bot and navigate to **Configure** > **Properties**.

### AppInsights API key

Provide an Azure App Insights API key. Learn how to [generate a new API key](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID). Only **Read** permission is required.

### AppInsights Application ID

To find this value, open Application Insights and navigate to **Configure** > **API Access**.

/Snip-->

Дополнительные сведения о том, как найти эти значения, см. в статье [Ключи Application Insights](~/bot-service-resources-app-insights-keys.md).

## <a name="additional-resources"></a>Дополнительные ресурсы
* [Ключи Application Insights](~/bot-service-resources-app-insights-keys.md)
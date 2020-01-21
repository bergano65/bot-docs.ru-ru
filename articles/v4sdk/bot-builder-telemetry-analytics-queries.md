---
title: Анализ данных телеметрии бота (служба Bot)
description: Сведения о том, как анализировать поведение бота с помощью запросов Kusto.
keywords: телеметрия, appinsights, мониторинг бота, Kusto, запросы
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/10/2020
ms.openlocfilehash: a1762e79ab1524f05818d546d04c2e8a1df5fcdd
ms.sourcegitcommit: caaf394017dbdb1cfaba32e2d0a1e32c5ab71792
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/10/2020
ms.locfileid: "75869834"
---
# <a name="analyze-your-bots-telemetry-data"></a>Анализ данных телеметрии бота 

## <a name="analyzing-bot-behavior"></a>Анализ поведения бота

Анализировать поведение бота можно с помощью указанной ниже коллекции запросов. Эта коллекция позволяет создавать пользовательские запросы в [Azure Monitor Log Analytics](https://aka.ms/log-analytics-azure-monitor) и (или) панели мониторинга и визуализации [Power BI](https://aka.ms/power-bi-overview).

## <a name="prerequisites"></a>предварительные требования
Будет полезным иметь хотя бы общее представление о следующих понятиях:

* [запросы Kusto](https://aka.ms/Kusto-query-overview);

* создание запросов к журналам Azure Monitor с помощью [Log Analytics](https://aka.ms/azure-monitor-log-queries-get-started) на портале Azure;

* основные принципы [запросов к журналам](https://aka.ms/azure-monitor-log-queries-get-started) в Azure Monitor.

## <a name="dashboards"></a>Панели мониторинга
Панели мониторинга Azure позволяют с удобством просматривать и предоставлять другим сведения, полученные на основе ваших запросов.  Вы можете создавать настраиваемые панели мониторинга, чтобы отслеживать работу ботов, сопоставляя запросы с плитками на панели мониторинга. Дополнительные сведения о панелях мониторинга и сопоставлении запросов с ними см. в статье о [создании панелей мониторинга данных Log Analytics и предоставлении к ним общего доступа](https://aka.ms/log-analytics-create-share-dashboards). Далее в этой статье представлены примеры запросов, которые будут полезны для мониторинга поведения бота.  

## <a name="example-kusto-queries"></a>Примеры запросов Kusto

> [!NOTE]
> Мы рекомендуем создать сводки по разным измерениям, таким как периоды, каналы и языковые стандарты, для всех запросов в этой статье.

### <a name="number-of-users-per-period"></a>Число пользователей за период
В этом примере создается график по данным о том, сколько уникальных пользователей обращалось к вашему боту за последние 14 дней.  Период можно легко изменить, присвоив другие значения переменным `queryStartDate`, `queryEndDate` и `interval`.

> [!IMPORTANT]
> Правильное число уникальных пользователей в этом запросе вы получите только в том случае, если они проходили проверку подлинности. Также результаты могут изменяться в зависимости от возможностей канала.

```Kusto
// number of users per period
let queryStartDate = ago(14d);
let queryEndDate = now();
let groupByInterval = 1d;
customEvents 
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| summarize uc=dcount(user_Id) by bin(timestamp, groupByInterval)
| render timechart
```

> [!TIP]
> [Оператор summarize](https://aka.ms/kusto-summarize-operator) в Kusto позволяет создать таблицу со статистическими сведениями по содержимому входной таблицы.
>
> Скалярная функция [Bin](https://docs.microsoft.com/azure/kusto/query/binfunction) в Kusto, когда она используется в сочетании с `summarize operator`, позволяет сгруппировать результаты запроса по указанному значению. В приведенном выше примере группировка выполнена по дням. Кроме этого, Kusto принимает значение h = часы, m = минуты, s = секунды, ms = миллисекунды, microsecond = микросекунды.
>
> Оператор [Render](https://aka.ms/kusto-query-render-operator?pivots=Kusto) позволяет легко визуализировать диаграммы, например график _timechart_, в котором ось x представляет значение DateTime, а для оси y можно указать любой другой числовой столбец. Он автоматически поддерживает эффективное распределение значений по оси x, даже если данные есть не за все периоды.  Если оператор render не указан, по умолчанию используется `table`.

#### <a name="sample-query-results"></a>Результаты выполнения примера запроса

<!-- 

OPEN ISSUE: Is it possible to define a more descriptive title in the legend when drawing the graph (vs. “Count”)?  Or overall title on top or underneath?  

Use the render PropertyName parameter:  title, xtitle, ytitle,, legend

-->

![Число пользователей за период](./media/userscount.png)

<!--  

OPEN ISSUE 1:

    "Days with little interaction may indicate service health issues"

    This is an interesting statement – I’m not sure if there’s a way to overlay service health metrics over these metrics.  It should all be in App Insights.

OPEN ISSUE 2:

    Gary Pretty: Overlap for activity per period - Looks like we have a large overlap between ‘Activity per period’ and ‘Activity per user per period’ – given that activity per user is only really useful in some scenarios, such as authenticated, I think it might simplify things to update the ‘Activity per period’ query / instructions to include a note as to how to filter down per user?   Not sure about this one myself, but thought it was worth bringing up.

    < I agree with you.  One approach might be to include both in the same section, with the ‘Activity per period’ as the primary with a note about what might be considered more of a ‘special case’ (I will need input on more details of where this is most applicable – as I’ve commented in a previous email). >


QUESTION: What changes are required?

-->

### <a name="activity-per-period"></a>Активность по периодам
В этом примере показано, как оценить объем действий по нужному измерению, например количество бесед, диалогов или сообщений в день за последние 14 дней. Период можно легко изменить, присвоив другие значения переменным `querystartdate`, `queryEndDate` и `interval`. Выбор измерения определяется предложением `extend`. В следующем примере `metric` может принимать значение _InstanceId_, _DialogId_ или _ActivityId_.

Присвойте параметру *metric* значение того измерения, которое вы намерены отображать.
  * *InstanceId* позволяет измерить количество [бесед](https://aka.ms/bot-builder-conversations);
  * *DialogId* позволяет измерить количество [диалогов](https://aka.ms/bot-builder-concept-dialog);
  * *ActivityId* позволяет измерить количество [сообщений](https://aka.ms/bot-rest-create-messages).

```Kusto
// measures the number of activity's (conversations, dialogs, messages) per period 
let queryStartDate = ago(14d);
let queryEndDate = now();
let groupByInterval = 1d;
customEvents 
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend InstanceId = tostring(customDimensions['<InstanceId>'])
| extend DialogId = tostring(customDimensions['<DialogId>'])
| extend ActivityId = tostring(customDimensions['<activityId>'])
| where DialogId != '' and  InstanceId != '' and user_Id != ''
| extend metric = InstanceId // DialogId or ActivityId
| summarize Count=dcount(metric) by  bin(timestamp, groupByInterval)
| order by Count desc nulls last 
| render timechart
```

> [!TIP]
> Оператор Kusto [Extend](https://aka.ms/kusto-extend-operator) позволяет создавать вычисляемые столбцы и добавлять их к результирующему набору.


#### <a name="sample-query-results"></a>Результаты выполнения примера запроса

![Активность по периодам](./media/convscount.PNG)


### <a name="activity-per-user-per-period"></a>Активность по пользователям по периодам
В этом примере показано, как подсчитать количество действий на одного пользователя за период. Это удобно, когда мы детализируем запрос _активности по периодам_ до следующего уровня: активности по пользователям по периодам. К действиям относятся диалоги, беседы или сообщения.  Запрос помогает оценить взаимодействие пользователей с ботом и выявить потенциальные проблемы, например: 

- дни с большим количеством действий одного пользователя могут означать атаку или тестирование;
- дни с низком уровнем взаимодействия могут указывать на проблемы с работоспособностью службы.

> [!TIP]
> Удалив строку _by user_Id_, вы получите общий объем действий бота, который можно сводить по времени и диалогам, сообщениям или беседам.

```Kusto
// number of users per period per dialogs
let queryStartDate = ago(14d);
let queryEndDate = now();
let interval = 6h;
customEvents 
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend InstanceId = tostring(customDimensions['InstanceId'])
| extend DialogId = tostring(customDimensions['DialogId'])
| extend ActivityId = tostring(customDimensions['activityId'])
| where DialogId != '' and InstanceId != '' and user_Id != ''
| extend metric = ActivityId // InstanceId // DialogId // or InstanceId for conversation count
| summarize Count=dcount(metric) by user_Id, bin(timestamp, groupByInterval)
| order by Count desc nulls last 
```

#### <a name="sample-query-results"></a>Результаты выполнения примера запроса 

| **user_Id**   | **timestamp**        | **Count** |
| ------------- | -------------------- | :------:  |
| User-8107ffd2 | 2019-09-03T00:00:00Z |    14     |
| User-75f2cc8f | 2019-08-30T00:00:00Z |    13     |
| User-75f2cc8d | 2019-09-03T00:00:00Z |    13     |
| User-3060aada | 2019-09-03T00:00:00Z |    10     |


### <a name="dialog-completion"></a>Завершение диалогов
Когда вы настроите клиент телеметрии для некоторого диалога, этот диалог и его дочерние элементы будут передавать некоторые данные телеметрии по умолчанию, например _started_ и _completed_. Этот пример можно использовать для сравнения количества завершенных (*completed*) и начатых (*started*) диалогов.  Если число начатых диалогов превышает число завершенных, значит, некоторые из пользователей не завершают настроенную последовательность. Это может стать отправной точкой для выявления и устранения ошибок в логике диалога.  Также это можно использовать для выявления более популярных и редко используемых диалогов.

```Kusto
// % Completed Waterfall Dialog: shows completes relative to starts 
let queryStartDate = ago(14d);
let queryEndDate = now();
customEvents
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| where name=="WaterfallStart"
| extend DialogId = customDimensions['DialogId']
| extend InstanceId = tostring(customDimensions['InstanceId'])
| join kind=leftouter (
    customEvents 
    | where name=="WaterfallComplete" 
    | extend InstanceId = tostring(customDimensions['InstanceId'])
  ) on InstanceId    
| summarize started=countif(name=='WaterfallStart'), completed=countif(name1=='WaterfallComplete') by tostring(DialogId)
| where started > 100  // filter for sample
// Show starts vs. completes
| project tostring(DialogId), started, completed
| order by started desc, completed asc  nulls last 
| render barchart  with (kind=unstacked, xcolumn=DialogId, ycolumns=completed, started, ysplit=axes) 
```

> [!TIP]
> [Оператор join](https://aka.ms/kusto-join-operator) в Kusto позволяет объединить строки двух таблиц для формирования новой таблицы путем сопоставления значений указанных столбцов каждой таблицы.
>
> [Оператор project](https://aka.ms/kusto-project-operator) используется для выбора полей, которые должны отображаться в выходных данных. Как и `extend operator`, который отвечает за добавление нового поля, оператор `project operator` позволяет выбрать поле из набора существующих или добавить новое.

#### <a name="sample-query-results"></a>Результаты выполнения примера запроса

![Завершение диалогов](./media/dialogwfratio.PNG)


### <a name="dialog-incompletion"></a>Незавершение диалога
Этот пример можно использовать для подсчета запущенных в течение указанного периода потоков диалогов, которые не были завершены из-за отмены или прекращения действий. С его помощью можно просмотреть незавершенные диалоги и выяснить, была ли отмена активным действием из-за сомнений или связана с потерей внимания либо интереса пользователя.

<!--  

OPEN ISSUE:

“If number of started dialogs is much greater than number of completed, users do not complete the dialog flow. Troubleshoot dialog logic.”

you can use the funnel view to understand where step dropoff is.  If there is a doc describing how to do that in our docs, point to it... 

QUESTION: Who can I talk to about a a doc describing this? 

ALSO: I removed what was line 6 in the example because it was a duplicate where statement: | where timestamp > period    // change timespan accordingly

-->

```Kusto
// show incomplete dialogs
let queryStartDate = ago(14d);
let queryEndDate = now();
customEvents 
| where timestamp > queryStartDate 
| where timestamp < queryEndDate
| where name == "WaterfallStart" 
| extend DialogId = customDimensions['DialogId']
| extend instanceId = tostring(customDimensions['InstanceId'])
| join kind=leftanti (
  customEvents
  | where name == "WaterfallComplete" 
  | extend instanceId = tostring(customDimensions['InstanceId'])
  ) on instanceId
| summarize cnt=count() by  tostring(DialogId)
| order by cnt
| render barchart
```

> [!TIP]
> [Оператор order](https://aka.ms/kusto-query-order-operator) в Kusto (аналогичен `sort operator`) сортирует строки входной таблицы в порядке значений одного или нескольких столбцов.  Примечание. Если вам нужно исключить значения NULL из результатов любого запроса, их можно отфильтровать в операторе WHERE (например, добавив строку "and isnotnull(Timestamp)") или вернуть значения NULL в начале или в конце списка (добавив `nulls first` или `nulls first` в конец инструкции order). 
>

#### <a name="sample-query-results"></a>Результаты выполнения примера запроса

![](./media/cancelleddialogs.PNG)





### <a name="dialog-sequence-drill-down"></a>Детализация последовательности диалога 

#### <a name="waterfall-startstepcomplete-for-dialog-in-conversation"></a>Просмотр начала, шагов и завершения для каскадных диалогов в беседе
Этот пример отображает последовательность шагов диалога, сгруппированных по беседе (instanceId). Это полезно при выявлении шагов, которые приводят к прерыванию диалога. 

Выполните этот запрос, указав нужное значение `DialogId` вместо заполнителя \<SampleDialogId>. 

```Kusto
// Drill down: Show waterfall start/step/complete for specific dialog
let queryStartDate = ago(14d);
let queryEndDate = now();
let DialogActivity=(dlgid:string) {
customEvents
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend DialogId = customDimensions['DialogId']
| extend StepName = customDimensions['StepName']
| extend InstanceId = customDimensions['InstanceId']
| where DialogId == dlgid
| project timestamp, name, StepName, InstanceId 
| order by tostring(InstanceId), timestamp asc
};
// For example see SampleDialogId behavior
DialogActivity("<SampleDialogId>")
```

> [!TIP]
> Запрос был написан с использованием [определяемой запросом функции](https://aka.ms/kusto-user-functions). Это функция, которая определяется пользователем с помощью оператора let и используется в пределах одного запроса. Этот пример можно переписать и без `query-defined function`, вот так:
>
> ```Kusto
> let queryStartDate = ago(14d);
> let queryEndDate = now();
> customEvents
> | where timestamp > queryStartDate
> | where timestamp < queryEndDate
> | extend DialogId = customDimensions['DialogId']
> | extend StepName = customDimensions['StepName']
> | extend InstanceId = customDimensions['InstanceId']
> | where DialogId == "<SampleDialogId>"
> | project timestamp, name, StepName, InstanceId 
> | order by tostring(InstanceId), timestamp asc
> ```

##### <a name="sample-query-results"></a>Результаты выполнения примера запроса

| **timestamp**       | **name**                   | **StepName**                    | **InstanceId**  |
| ------------------- | -------------------------- | ------------------------------- | --------------- |
| 2019-08-23T20:04... | WaterfallStart             | null                            | ...79c0f03d8701 |
| 2019-08-23T20:04... | WaterfallStep              | GetPointOfInterestLocations     | ...79c0f03d8701 |
| 2019-08-23T20:04... | WaterfallStep              | ProcessPointOfInterestSelection | ...79c0f03d8701 |
| 2019-08-23T20:04... | WaterfallStep              | GetRoutesToDestination          | ...79c0f03d8701 |
| 2019-08-23T20:05... | WaterfallStep              | ResponseToStartRoutePrompt      | ...79c0f03d8701 |
| 2019-08-23T20:05... | WaterfallComplete _<sup>1_ | null                            | ...79c0f03d8701 |
| 2019-08-28T23:35... | WaterfallStart             | null                            | ...6ac8b3211b99 |
| 2019-08-28T23:35... | WaterfallStep _<sup>2_     | GetPointOfInterestLocations     | ...6ac8b3211b99 |
| 2019-08-28T19:41... | WaterfallStart             | null                            | ...8137d76a5cbb |
| 2019-08-28T19:41... | WaterfallStep _<sup>2_     | GetPointOfInterestLocations     | ...8137d76a5cbb |
| 2019-08-28T19:41... | WaterfallStart             | null                            | ...8137d76a5cbb |

<sub>1</sup> _Завершено_ 

<sub>2</sup> _Прервано_

_Интерпретация полученных данных. Похоже, что пользователи часто прерывают беседу на шаге GetPointOfInterestLocations._ 

> [!NOTE] 
> Каскадные диалоги выполняют последовательность (начало, несколько шагов, завершение). Если для последовательности отображается начало, но нет завершения, значит, диалог был прерван из-за отмены пользователем или прекращения действий. В этих подробных результатах анализа вы заметите такое поведение (сравнивая количество завершенных и прерванных шагов).

#### <a name="waterfall-startstepcompletecancel-steps-aggregate-totals"></a>Агрегированные результаты для шагов каскадного диалога (начало, шаг, завершение, отмена)
Этот пример демонстрирует статистические итоги: общее количество запусков последовательности диалога, общее количество каскадных шагов, количество успешно выполненных и количество отмен. Разность значения _WaterfallStart_ и суммы значений _WaterfallComplete_ и _WaterfallCancel_ обозначает общее число отмененных диалогов.

```Kusto
// Drill down: Aggregate view of waterfall start/step/complete/cancel steps totals for specific dialog
let queryStartDate = ago(14d);
let queryEndDate = now();
let DialogSteps=(dlgid:string) {
customEvents
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend DialogId = customDimensions['DialogId']
| where DialogId == dlgid
| project name
| summarize count() by name
};
// For example see SampleDialogId behavior
DialogSteps("<SampleDialogId>")
```

##### <a name="sample-query-results"></a>Результаты выполнения примера запроса

| **name**          | **count** |
| ----------------- | --------: |
| WaterfallStart    | 21        |
| WaterfallStep     | 47        |
| WaterfallComplete | 11        |
| WaterfallCancel   | 1         |

_Интерпретация полученных данных. Из 21 вызванных последовательностей этого диалога завершены только 11. Еще 9 оставлены без внимания и один отменен пользователем_.



### <a name="average-duration-in-dialog"></a>Средняя продолжительность диалога
В этом примере оценивается среднее время, затрачиваемое пользователями на определенный диалог. Длительное время нахождения в диалоге может означать, что можно его упростить.

 ```Kusto
// Average dialog duration
let queryStartDate = ago(14d);
let queryEndDate = now();
customEvents 
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| where name=="WaterfallStart"
| extend DialogId = customDimensions['DialogId']
| extend instanceId = tostring(customDimensions['InstanceId'])
| join kind=leftouter (customEvents | where name=="WaterfallCancel" | extend instanceId = tostring(customDimensions['InstanceId'])) on instanceId 
| join kind=leftouter (customEvents | where name=="WaterfallComplete" | extend instanceId = tostring(customDimensions['InstanceId'])) on instanceId 
| extend duration = case(not(isnull(timestamp1)), timestamp1 - timestamp, 
not(isnull(timestamp2)), timestamp2 - timestamp, 0s) // Abandoned are not counted. Alternate: now()-timestamp)
| extend seconds = round(duration / 1s)
| summarize AvgSeconds=avg(seconds) by tostring(DialogId)
| order by AvgSeconds desc nulls last 
| render barchart with (title="Duration in Dialog")
 ```

#### <a name="sample-query-results"></a>Результаты выполнения примера запроса

![dialogduration](./media/dialogduration.PNG)


### <a name="average-steps-in-dialog"></a>Среднее число шагов в диалоге
В этом примере отображается "длина" для каждого выполненного диалога, которая определяется по среднему, минимальному и максимальному количеству шагов, а также стандартному отклонению. Это помогает анализировать качество диалогов. Пример:

- Диалоги с большим количеством шагов следует проверить на возможность упрощения.
- Большая разница между минимальным, максимальным и (или) средним числом шагов может означать, что пользователи путаются или теряются при выполнении задач. Возможно, для таких задач стоит сократить длину путей или снизить сложность диалога.
- Диалоги с большим стандартным отклонением могут содержать слишком сложные пути или ошибки в логике (частая отмена или прекращение).
- Малое число шагов в диалоге может означать, что они не завершены. Сравнение долей завершенных и прекращенных диалогов поможет понять, так ли это.  

```Kusto
// min/max/std/avg steps per dialog
let queryStartDate = ago(14d);
let queryEndDate = now();
customEvents
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend DialogId = tostring(customDimensions['DialogId'])
| extend StepName = tostring(customDimensions['StepName'])
| extend InstanceId = tostring(customDimensions['InstanceId'])
| where name == "WaterfallStart" or  name == "WaterfallStep" or  name == "WaterfallComplete" 
| order by InstanceId, timestamp asc
| project timestamp, DialogId, name, InstanceId, StepName 
| summarize cnt=count() by InstanceId, DialogId
| summarize avg=avg(cnt), minsteps=min(cnt),maxsteps=max(cnt), std=stdev(cnt) by DialogId
| extend avgsteps = round(avg, 1)
| extend avgshortbysteps=maxsteps-avgsteps
| extend avgshortbypercent=round((1.0 - avgsteps/maxsteps)*100.0, 1)
| project DialogId, avgsteps, minsteps, maxsteps, std, avgshortbysteps, avgshortbypercent
| order by std desc nulls last 
```

#### <a name="sample-query-results"></a>Результаты выполнения примера запроса

| ИД диалога               | среднее число шагов | минимальное число шагов | максимальное число шагов | std  | среднее сокращение по шагам | средняя сокращение в процентах |
| ----------------------- | --------: | :-------: | :-------: | ---: | :----------------: | -------------------: |
| FindArticlesDialog      | 6.2       | 2         | 7         | 2,04 | 0,8                | 11,4 %                |
| CreateTicket            | 4.3       | 2         | 5         | 1.5  | 0,7                | 14 %                  |
| CheckForCurrentLocation | 3.9       | 2         | 5         | 1,41 % | 1,1                | 22 %                  |
| BaseAuth                | 3.3       | 2         | 4         | 1,03 % | 0,7                | 17,5 %                |
| Подключение              | 2.7       | 2         | 4         | 0,94 | 1,3                | 32,5 %                |

Интерпретация полученных данных. Например, у FindArticlesDialog большая разница между минимальным и максимальным числом шагов, а значит, его нужно проверить и, возможно, переработать и (или) оптимизировать.



### <a name="channel-activity-by-activity-metric"></a>Активность канала по метрике действия
В этом примере оценивается количество действий, которые бот получает от каждого канала за определенный период. Для этого он подсчитывает любую из следующих метрик: входящие сообщений, пользователи, беседы или диалоги. Это может быть полезно для анализа работоспособности служб и (или) оценки популярности каналов.


```Kusto
// number of metric: messages, users, conversations, dialogs by channel
let queryStartDate = ago(14d);
let queryEndDate = now();
let groupByInterval = 1d;
customEvents 
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| extend InstanceId = tostring(customDimensions['InstanceId'])
| extend DialogId = tostring(customDimensions['DialogId'])
| extend ActivityId = tostring(customDimensions['activityId'])
| extend ChannelId = tostring(customDimensions['channelId'])
| where DialogId != '' and  InstanceId != '' and user_Id != ''
| extend metric = user_Id // InstanceId or ActivityId or user_Id
| summarize Count=count(metric) by  ChannelId, bin(timestamp, groupByInterval)
| order by Count desc nulls last 
| render barchart with (title="Users", kind=stacked) // or Incoming Messages or Conversations or Users
```

> [!TIP]
> Вы можете попробовать следующие варианты:
> - Выполните запрос без сегментирования по меткам времени: `bin(timestamp, groupByInterval)`.
> - Также сравните `dcount` (для отдельных пользователей) и `count` (для всех действий с событиями пользователя).  Этот запрос удобен и для повторных пользователей.
> 

#### <a name="sample-query-results"></a>Результаты выполнения примера запроса

![ChannelsUsage](./media/ChannelsUsage.PNG)

 _Интерпретация полученных данных. Ранее широко использовалось тестирование в эмуляторе, но с момента публикации службы в канале DirectLineSpeech он стал самым популярным._

<!--  

Open Issue: More interesting than the “certainty” score would be linking intent to dialog completion %. That infers “certainty” by user’s actions. 

QUESTION: What changes are required?

-->
### <a name="total-intents-by-popularity"></a>Общее количество намерений по популярности
Этот пример применяется к ботам с поддержкой LUIS. Он отображает сводку по всем [намерениям](https://aka.ms/botbuilder-luis-concept#recognize-intent) с сортировкой по популярности и оценками уверенности в определении этих намерений.

* На практике это представление лучше просматривать отдельно для каждой метрики.
* Более популярные пути намерений можно оптимизировать для улучшения взаимодействия с пользователем.
* Низкие средние показатели могут означать плохое определение намерений или низкую востребованность.

```Kusto
// show total intents
let queryStartDate = ago(14d);
let queryEndDate = now();
customEvents
| where timestamp > queryStartDate
| where timestamp < queryEndDate
| where name startswith "LuisResult" 
| extend intentName = tostring(customDimensions['intent'])
| extend intentScore = todouble(customDimensions['intentScore'])
| summarize ic=count(), ac=avg(intentScore)*100 by intentName
| project intentName, ic, ac
| order by ic desc nulls last 
| render barchart with (kind=unstacked, xcolumn=intentName, ycolumns=ic,ac, title="Intents Popularity")
```

#### <a name="sample-query-results"></a>Результаты выполнения примера запроса

![IntentPopularity](./media/Telemetry/IntentPopularity.PNG)

_Интерпретация полученных данных. Например, самое популярное намерение (подтверждение) обнаруживается со средней точностью лишь 23 %._


> [!TIP]
> Линейчатые диаграммы (barchart) — один из нескольких десятков вариантов, доступных для запросов Kusto.  Вы также можете использовать anomalychart (диаграмма аномалий), areachart (диаграмма зона), columnchart (диаграмма столбцов), linechart (график), scatterchart (точечная диаграмма). Дополнительные сведения см. в документации по [оператору render](https://aka.ms/kusto-query-render-operator?pivots=Kusto).


## <a name="schema-of-bot-analytics-instrumentation"></a>Схема инструментирования аналитики бота
В следующих таблицах показаны наиболее распространенные поля, в которые бот может записывать данные телеметрии.

### <a name="general-envelope"></a>Общая огибающая

Типичные поля анализа журналов при инструментировании Application Insights.

| **Поле**        | **Описание**        | **Примеры значений**                                            |
| ---------------- | ---------------------- | ------------------------------------------------------------ |
| name             | тип сообщений;           | BotMessageSend, BotMessageReceived, LuisResult, WaterfallStep, WaterfallStart, SkillWebSocketProcessRequestLatency, SkillWebSocketOpenCloseLatency, WaterfallComplete, QnaMessage, WaterfallCancel, SkillWebSocketTurnLatency, AuthPromptValidatorAsyncFailure |
| customDimensions | Пакет средств разработки аналитики бота      | activityId=\<id>,  activityType=message,  channelId=emulator,  fromId=\<id>,  fromName=User,  locale=en-us,  recipientId=\<id>,  recipientName=Bot,  text=find  a coffee shop |
| TIMESTAMP        | Время события          | 2019-09-05T18:32:45.287082Z                                  |
| instance_Id      | Идентификатор беседы        | f7b2c416-a680-4b2c-b4cc-79c0f03d8711                         |
| operation_Id     | ИД шага                | 084b2856947e3844a5a18a8476d99aaa                             |
| user_Id          | Уникальный идентификатор пользователя в канале | emulator7c259c8e-2f47...                                     |
| client_IP        | IP-адрес клиента      | 127.0.0.1 (может отсутствовать в связи с блокировкой конфиденциальности)               |
| client_City      | Город клиента            | Редмонд (если обнаружен, но может отсутствовать)                         |

### <a name="custom-dimensions"></a>Пользовательские измерения

Большинство данных о действиях для конкретного бота хранится в поле _customDimensions_.

| **Поле**     | **Описание**      | **Примеры значений**                                 |
| ------------- | -------------------- | ------------------------------------------------- |
| activityId    | Идентификатор сообщения           | \<id>: 8da6d750-d00b-11e9-80e0-c14234b3bc2a       |
| activityType  | Тип сообщения      | message, conversationUpdate,  event, invoke       |
| channelId     | Идентификатор канала   | emulator, directline, msteams,  webchat           |
| fromId        | Идентификатор источника      | \<id>                                             |
| fromName      | Имя пользователя, полученное от клиента | John Bonham, Keith Moon, Steve Smith,  Steve Gadd |
| локаль        | Исходный язык клиента | en-us, zh-cn, en-GB,  de-de, zh-CN                |
| recipientId   | Идентификатор получателя | \<id>                                             |
| recipientName | Имя получателя       | John Bonham, Keith Moon, Steve Smith,  Steve Gadd |
| text          | Текст сообщения      | find a coffee shop                                |

### <a name="custom-dimensions-luis"></a>Пользовательские измерения: LUIS

При инструментировании LUIS данные сохраняются в следующих полях пользовательских измерений.

| **Поле**      | **Описание**         | **Примеры значений**                                       |
| -------------- | ----------------------- | ------------------------------------------------------- |
| intent         | Обнаруженное намерение LUIS    | pointOfInterestSkill                                    |
| intentScore    | Оценка распознавания LUIS  | 0,98                                                    |
| Сущности       | Обнаруженные сущности LUIS  | FoodOfGrocery =  [["coffee"]], KEYWORD= ["coffee shop"] |
| Вопрос       | Обнаруженный вопрос LUIS  | find a coffee shop                                      |
| sentimentLabel | Обнаруженная тональность LUIS | Позитивная                                                |

### <a name="custom-dimensions-qnamaker"></a>Пользовательские измерения: QnA Maker

При инструментировании QnAMaker данные сохраняются в следующих полях пользовательских измерений.

| **Поле**       | **Описание**            | **Примеры значений**                                            |
| --------------- | -------------------------- | ------------------------------------------------------------ |
| question        | Обнаруженный вопрос QnA      | what can you do?                                             |
| Ответ          | Ответ QnA                 | You have questions, I may have  answers.                     |
| articleFound    | QnA                        | Да                                                         |
| questionId      | Идентификатор вопроса QnA            | 488                                                          |
| knowledgeBaseId | Идентификатор базы знаний QnA                  | 2a4936f3-b2c8-44ff-b21f-67bc413b9727                         |
| matchedQuestion | Массив подходящих вопросов | ["Can you explain to me what your  role is?","Can  you tell me a bit about yourself?","Can  you tell me about you?","could  you help me","hmmm so  what can you do?","how  can you help me","How  can you help me?","How  can you help?","so  how can i  use you in my projects?","Talk to  me about your capability","What  are you capable of?",… |

 

## <a name="see-also"></a>См. также:

* Инструкции по написанию запросов к журналам Azure Monitor см. в [этой статье](https://aka.ms/azure-monitor-log-queries-get-started).
* Прочтите статью о [визуализации данных из Azure Monitor](https://aka.ms/azure-monitor-visualize-data).
* Узнайте, как [добавить телеметрию в бота](https://aka.ms/AddBotTelemetry).
* Ознакомьтесь с дополнительными сведениями о [запросах по журналам в Azure Monitor](https://aka.ms/azure-monitor-log-queries).
* Узнайте, как [создавать панели мониторинга данных Log Analytics и предоставлять к ним общий доступ](https://aka.ms/log-analytics-create-share-dashboards).

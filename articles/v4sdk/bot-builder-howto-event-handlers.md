---
title: Использование обработчиков событий | Документация Майкрософт
description: Узнайте, как использовать обработчики событий.
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 06/14/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 36bd2638c599de1662a37dd85790b6126184d51b
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42905693"
---
# <a name="using-event-handlers"></a>Использование обработчиков событий

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Обработчики событий — это функции, которые можно добавить к событиям будущего действия в [Defining a turn](bot-builder-basics.md#defining-a-turn) (Определение активности). Эти действия являются `SendActivity`, `UpdateActivity` и `DeleteActivity`, и каждое из них имеет собственный обработчик. Эти обработчики полезны, когда нужно что-то делать для каждого будущего действия этого типа для текущего объекта контекста.

Можно добавить несколько обработчиков событий к каждому событию действия, и они будут обрабатывать каждое событие **после** их добавления, поэтому важно, где они будут добавлены в коде.

> [!NOTE]
> *Обработчики событий* в этой статье не соответствуют традиционным языковым определениям *событий* и *обработчиков*. Вместо этого они ссылаются на идею концептуального программирования *событий* и *обработчиков*.

## <a name="using-the-next-delegate"></a>Использование делегата *next*

Вызывая делегат *next*, каждый обработчик передает выполнение следующему обработчику в том порядке, в котором они были добавлены. Последний делегат *next* представляет собой вызов адаптера для фактической отправки, обновления или удаления действия.

Если не предусмотрено иное, важно вызывать *next()* в обработчике, иначе возникнет [короткое замыкание](bot-builder-create-middleware.md#short-circuit-routing) действия. Разница между коротким замыканием действия и ПО промежуточного слоя заключается в том, что короткое замыкание полностью отменяет действие, тогда как ПО промежуточного слоя изменяет код разрешения для запуска, но выполнение действия продолжается.

Для операций отправки в случае успеха делегат *next* возвращает идентификаторы, назначенные отправленным сообщениям.

## <a name="expected-return-value"></a>Ожидаемое возвращаемое значение

Обработчик события ожидает, что возвращаемое значение будет результатом следующего делегата. При необходимости этот результат можно просмотреть и сохранить перед возвратом, либо он может быть возвращен непосредственно, как показано ниже.

Возвращаемое значение обработчика `SendActivity` — это возвращаемое значение, которое передается обратно через цепочку в качестве возвращаемого значения соответствующего следующего делегата.

# <a name="ctabcseventhandler"></a>[C#](#tab/cseventhandler)

Существует три типа обработчиков событий: `OnSendActivities()`, `OnUpdateActivity()` и `OnDeleteActivity()`. В этом примере обработчик `OnSendActivities()` добавляется в код бота, однако обработчики также можно добавлять в код ПО промежуточного слоя.

Обработчики часто добавляются в виде лямбда-выражений, которые можно увидеть в этом примере. Здесь можно будет прослушивать действие ввода пользователя при написании **help**.

```cs
public Task OnTurn(ITurnContext context)
{
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        
        context.OnSendActivities(async (handlerContext, activities, handlerNext) =>
        {
            if (handlerContext.Activity.Text == "help")
            {
                Console.WriteLine("help!");
                // Do whatever logging you want to do for this help message
            }

            return await handlerNext();
        });
        // ...
    }
}
```

# <a name="javascripttabjseventhandler"></a>[JavaScript](#tab/jseventhandler)

Существует три типа обработчиков событий: `onSendActivities()`, `onUpdateActivity()` и `onDeleteActivity()`. В этом примере обработчик `onSendActivities()` добавляется в код бота, однако обработчики также можно добавлять в код ПО промежуточного слоя.

В этом примере будет прослушиваться действие ввода пользователя при написании **help**.

```js
adapter.processActivity(req, res, async (context) => {

    if (context.activity.type === 'message') {

        context.onSendActivities(async (handlerContext, activities, handlerNext) => { 
            
            if(handlerContext.activity.text === 'help'){
                console.log('help!')
                // Do whatever logging you want to do for this help message
            }
            // Add handler logic here
        
            await handlerNext(); 
        });
        await context.sendActivity(`you said ${context.activity.text}`);
    }
});
```

---

Важно различать события *отправить действие* и *обновить или удалить действие*, так как первое создает совершенно новое событие действия, а второе работает с прошлым действием. Также не все каналы поддерживают действие *обновления* или *удаления*. Рекомендуется добавить соответствующую обработку исключений для вызовов этих действий и их обработчиков, чтобы учесть такую возможность.


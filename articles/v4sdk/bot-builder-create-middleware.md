---
title: Создание собственного ПО промежуточного слоя | Документация Майкрософт
description: Узнайте, как написать собственное ПО промежуточного слоя.
keywords: ПО промежуточного слоя, пользовательское ПО промежуточного слоя, короткое замыкание, откат, обработчики действий
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 78c0afd6095cf9a10e04e1b0131d66ce6964f369
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000201"
---
# <a name="create-your-own-middleware"></a>Создание собственного ПО промежуточного слоя

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

ПО промежуточного слоя позволяет писать полнофункциональные подключаемые модули для ботов, которые также могут использоваться и другими пользователями. Мы расскажем, как добавить и реализовать базовое ПО промежуточного слоя, и покажем, как оно работает. Пакет SDK версии 4 предоставляет некоторые ПО промежуточного слоя для управления состоянием, служб LUIS, QnAMaker и для перевода. Для дополнительных сведений просмотрите пакет SDK Bot Builder для платформ [.NET](https://github.com/Microsoft/botbuilder-dotnet) или [JavaScript](https://github.com/Microsoft/botbuilder-js).

## <a name="adding-middleware"></a>Добавление ПО промежуточного слоя

Как показано в следующем примере, основанном на простом примере бота, созданном по инструкциям из статьи [по началу работы](~/bot-service-quickstart.md), к нашим службам добавляются два разных фрагмента ПО промежуточного слоя с новым экземпляром каждого из классов.

> [!IMPORTANT]
> Помните, что порядок, в котором они добавляются к параметрам, определяет порядок их выполнения. Учитывайте это при использовании более одного фрагмента ПО промежуточного слоя.

**Startup.cs.**

# <a name="ctabcsaddmiddleware"></a>[C#](#tab/csaddmiddleware)

Добавьте вызовы метода `options.Middleware.Add(new MyMiddleware());` к параметрам службы бота для каждого фрагмента ПО промежуточного слоя, который вы хотите добавить.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<HelloBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        options.Middleware.Add(new MyMiddleware());
        options.Middleware.Add(new MyOtherMiddleware());
    });
}
```
# <a name="javascripttabjsaddmiddleware"></a>[JavaScript](#tab/jsaddmiddleware)

Добавьте `adapter.use(MyMiddleware());` к адаптеру для каждого фрагмента ПО промежуточного слоя, который вы хотите добавить.

```javascript
// Create adapter
const adapter = new botbuilder.BotFrameworkAdapter({
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

adapter.use(MyMiddleware());
adapter.use(MyOtherMiddleware());
```

---


## <a name="implementing-your-middleware"></a>Реализация ПО промежуточного слоя

Каждый фрагмент ПО промежуточного слоя наследуется из интерфейса ПО промежуточного слоя и всегда реализует обработчик процессов, который выполняется для каждого действия, отправленного боту. Для каждого добавляемого фрагмента ПО промежуточного слоя обработчик процессов может изменить объект контекста или выполнить задачу, такую как ведение журнала, прежде чем разрешить другому ПО промежуточного слоя или логике бота взаимодействовать с объектом контекста по мере его продвижения по конвейеру.

# <a name="ctabcsetagoverwrite"></a>[C#](#tab/csetagoverwrite)

Каждый фрагмент ПО промежуточного слоя наследуется из `IMiddleware` и всегда реализует `OnTurn()`.

**ExampleMiddleware.cs**
```csharp
public class MyMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {            
        // This simple middleware reports the request type and if we responded
        await context.SendActivity($"Request type: {context.Activity.Type}");
        
        await next();            

        // Report if any responses were recorded
        string response = context.Responded ? "yes" : "no";
        await context.SendActivity($"Responded?  {response}");
    }
}

public class MyOtherMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        // simple middleware to add an additional send activity
        await context.SendActivity($"My other middleware just saying Hi before the bot logic");

        await next();
    }
}

```

# <a name="javascripttabjsimplementmiddleware"></a>[JavaScript](#tab/jsimplementmiddleware)

Каждый фрагмент ПО промежуточного слоя наследуется из `MiddlewareSet` и всегда реализует `onTurn()`.

**ExampleMiddleware.js**
```js
adapter.use({onTurn: async (context, next) =>{

    // This simple middleware reports the activity type and if we responded
    await context.sendActivity(`Activity type: ${context.activity.type}`); 
    await next();            

    // Report if any responses were recorded
    const response = context.activity.text ? "yes" : "no";
    await context.sendActivity(`Responded?  ${response}`);

}}, {onTurn: async (context, next) => {

     // simple middleware to add an additional send activity
     await context.sendActivity("My other middleware just saying Hi before the bot logic");

     await next();
}})
```

---

Вызов команды `next()` приводит к продолжению выполнения следующего фрагмента ПО промежуточного слоя. Возможность выбора того, когда выполнение продолжится, позволяет написать код, который выполняется **после** выполнения остальной части стека ПО промежуточного слоя. Можно предпринять действия на "заднем фронте" обработчика процессов после завершения выполнения логики бота и другого ПО промежуточного слоя.  В примере выше именно это совершило первое реализованное ПО промежуточного слоя, сообщив о наличии ответа этому объекту контекста перед возвратом выполнения на конвейер.

## <a name="short-circuit-routing"></a>Маршрутизация короткого замыкания

В некоторых случаях вам может понадобиться остановка обработки полученного действия, которая называется коротким замыканием. Это удобно, когда ПО промежуточного слоя полностью выполняет запрос, обеспечивает легкий ответ для определенных команд или иным образом обрабатывает входящий запрос без необходимости использования логики бота.

Давайте создадим фрагмент ПО промежуточного слоя, который отправляет ответ и предотвращает дальнейшую маршрутизацию запроса каждый раз, когда пользователь говорит "ping".

# <a name="ctabcsmiddlewareshortcircuit"></a>[C#](#tab/csmiddlewareshortcircuit)
```cs
public class ExampleMiddleware : IMiddleware
{
    public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
    {
        var utterance = context.Activity?.AsMessageActivity()?.Text.Trim().ToLower();

        if (utterance == "ping") 
        {
            context.SendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }
    }
}
```
# <a name="javascripttabjsmiddlewareshortcircuit"></a>[JavaScript](#tab/jsmiddlewareshortcircuit)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    const utterance = (context.activity.text || '').trim().toLowerCase();
        if (utterance == "ping") 
        {
            await context.sendActivity("pong");
            return;
        } 
        else 
        {
            await next();
        }

}})

```

---

## <a name="fallback-processing"></a>Обработка отката

Вам также может понадобиться отвечать на еще не обработанные запросы. Это легко выполнить с помощью заднего фронта обработчика процессов, проверив свойство `context.Responded`. Давайте создадим простой фрагмент ПО промежуточного слоя, который автоматически отвечает "I didn't understand", если бот не справляется с запросом.

# <a name="ctabcsfallback"></a>[C#](#tab/csfallback)
```cs
public async Task OnTurn(ITurnContext context, MiddlewareSet.NextDelegate next)
{
    await next();

    if (!context.Responded) 
    {
        context.SendActivity("I didn't understand.");
    }
}
```
# <a name="javascripttabjsfallback"></a>[JavaScript](#tab/jsfallback)
```JavaScript
adapter.use({onTurn: async (context, next) =>{
    await next();

    if (!context.responded) 
    {
       await context.sendActivity("I didn't understand.");
    }

}})
```

---

> [!NOTE] 
> Это может не сработать в некоторых случаях, например, когда отвечать пользователю может другое ПО промежуточного слоя или когда бот принимает сообщение, но не отвечает. В этом случае ответ "I don't understand" введет пользователя в заблуждение.



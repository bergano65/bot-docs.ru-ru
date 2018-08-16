---
title: Управление ходом общения с помощью диалогов | Документы Майкрософт
description: Узнайте, как моделировать общение и управлять ходом общения с помощью диалогов и пакета SDK Bot Builder для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 81f403e9affa38f61ccb42c4c32a47683bbcbf33
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574810"
---
# <a name="manage-conversation-flow-with-dialogs"></a>Управление ходом общения с помощью диалогов

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-manage-conversation-flow.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-manage-conversation-flow.md)

[!INCLUDE [Dialog flow example](../includes/snippet-dotnet-manage-conversation-flow-intro.md)]

В этой статье описывается, как моделировать ход общения с помощью [диалогов](bot-builder-dotnet-dialogs.md) и пакета SDK Bot Builder для .NET. 

## <a name="invoke-the-root-dialog"></a>Вызов корневого диалога

Сначала бот вызывает "корневой диалог". В приведенном ниже примере показано, как привязать простой вызов HTTP POST к контроллеру и затем вызвать корневой диалог. 

```cs
public class MessagesController : ApiController
{
    public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
    {
            // Redirect to the root dialog.
            await Conversation.SendAsync(activity, () => new RootDialog()); 
            ...
    }
}
```

## <a name="invoke-the-new-order-dialog"></a>Вызов диалога "New Order"

Затем корневой диалог вызывает диалог "New Order". 

```cs
[Serializable]
public class RootDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        // Root dialog initiates and waits for the next message from the user. 
        // When a message arrives, call MessageReceivedAsync.
        context.Wait(this.MessageReceivedAsync); 
    }

    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result; // We've got a message!
        if (message.Text.ToLower().Contains("order"))
        {
            // User said 'order', so invoke the New Order Dialog and wait for it to finish.
            // Then, call ResumeAfterNewOrderDialog.
            await context.Forward(new NewOrderDialog(), this.ResumeAfterNewOrderDialog, message, CancellationToken.None);
        }
        // User typed something else; for simplicity, ignore this input and wait for the next message.
        context.Wait(this.MessageReceivedAsync);
    }

    private async Task ResumeAfterNewOrderDialog(IDialogContext context, IAwaitable<string> result)
    {
        // Store the value that NewOrderDialog returned. 
        // (At this point, new order dialog has finished and returned some value to use within the root dialog.)
        var resultFromNewOrder = await result;

        await context.PostAsync($"New order dialog just told me this: {resultFromNewOrder}");

        // Again, wait for the next message from the user.
        context.Wait(this.MessageReceivedAsync);
    }
}
```

## <a id="dialog-lifecycle"></a> Жизненный цикл диалога

При вызове диалог берет на себя управление ходом общения. Каждое новое сообщение будет обрабатываться этим диалогом, пока он не будет закрыт или перенаправлен к другому диалогу. 

В C# можно использовать `context.Wait()`, чтобы указать обратный вызов, выполняемый при следующей отправке сообщения пользователем. Чтобы закрыть диалог и удалить его из стека (и тем самым вернуть пользователя к предыдущему диалогу в стеке), используйте `context.Done()`. Необходимо завершать каждый метод диалога `context.Wait()`, `context.Fail()`, `context.Done()` или какой-либо директивой перенаправления, такой как `context.Forward()` или `context.Call()`. Метод диалога, который не заканчивается подобным образом, приведет к ошибке (так как платформа не будет знать, какое действие следует предпринять при следующей отправке сообщения пользователем).

## <a name="passing-state-between-dialogs"></a>Передача состояния между диалогами

Помимо сохранения состояния в боте, можно также передавать эти данные между разными диалогами, перегружая конструктор класса диалога.

```cs
[Serializable]
public class AgeDialog : IDialog<int>
{
    private string name;

    public AgeDialog(string name)
    {
        this.name = name;
    }
}
 ```

Вызовите код диалога, передав значение имени от пользователя.

```cs
private async Task NameDialogResumeAfter(IDialogContext context, IAwaitable<string> result)
{
    try
    {
        this.name = await result;
        context.Call(new AgeDialog(this.name), this.AgeDialogResumeAfter);
    }
    catch (TooManyAttemptsException)
    {
        await context.PostAsync("I'm sorry, I'm having issues understanding you. Let's try again.");
        await this.SendWelcomeMessageAsync(context);
    }
}
```

## <a name="sample-code"></a>Пример кода 

Полный пример, в котором показано, как управлять общением с помощью диалогов в пакете SDK Bot Builder для .NET, приведен в <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-BasicMultiDialog" target="_blank">примере BasicMultiDialog</a> на сайте GitHub.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Диалоги](bot-builder-dotnet-dialogs.md)
- [Проектирование потока беседы и управление им](../bot-service-design-conversation-flow.md)
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-BasicMultiDialog" target="_blank">Простой пример с несколькими диалогами (GitHub)</a>
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочник по пакету SDK Bot Builder для .NET</a>

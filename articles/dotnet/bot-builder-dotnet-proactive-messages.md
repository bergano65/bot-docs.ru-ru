---
title: Отправка упреждающих сообщений | Документация Майкрософт
description: Узнайте, как отправлять упреждающие сообщения с помощью пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c601db171f253b83bfa2d354f79018f03287bcf6
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297344"
---
# <a name="send-proactive-messages"></a>Отправка упреждающих сообщений

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-proactive-messages.md)
> - [Node.js](../nodejs/bot-builder-nodejs-proactive-messages.md)

[!INCLUDE [Introduction to proactive messages - part 1](../includes/snippet-proactive-messages-intro-1.md)]

## <a name="types-of-proactive-messages"></a>Типы упреждающих сообщений 

[!INCLUDE [Introduction to proactive messages - part 2](../includes/snippet-proactive-messages-intro-2.md)]

## <a name="send-an-ad-hoc-proactive-message"></a>Отправка нерегламентированного упреждающего сообщения

В следующих примерах кода показано, как отправить нерегламентированное упреждающее сообщение с помощью пакета SDK Bot Framework для .NET.

Чтобы иметь возможность отправить нерегламентированное сообщение пользователю, боту сначала необходимо собрать и сохранить сведения о пользователе из текущего сеанса общения. 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Extract data from the user's message that the bot will need later to send an ad hoc message to the user. 
    // Store the extracted data in a custom class "ConversationStarter" (not shown here).

    ConversationStarter.toId = message.From.Id;
    ConversationStarter.toName = message.From.Name;
    ConversationStarter.fromId = message.Recipient.Id;
    ConversationStarter.fromName = message.Recipient.Name;
    ConversationStarter.serviceUrl = message.ServiceUrl;
    ConversationStarter.channelId = message.ChannelId;
    ConversationStarter.conversationId = message.Conversation.Id;

    // (Save this information somewhere that it can be accessed later, such as in a database.)

    await context.PostAsync("Hello user, good to meet you! I now know your address and can send you notifications in the future.");
    context.Wait(MessageReceivedAsync);
}
```
> [!NOTE]
> Для простоты в этом примере не указан способ хранения данных пользователя. Неважно, каким образом хранятся данные, пока они доступны для последующего использования ботом.

Теперь, когда данные сохранены, бот может просто получить данные, создать нерегламентированное упреждающее сообщение и отправить его. 

```cs
// Use the data stored previously to create the required objects.
var userAccount = new ChannelAccount(toId,toName);
var botAccount = new ChannelAccount(fromId, fromName);
var connector = new ConnectorClient(new Uri(serviceUrl));

// Create a new message.
IMessageActivity message = Activity.CreateMessageActivity();
if (!string.IsNullOrEmpty(conversationId) && !string.IsNullOrEmpty(channelId))  
{
    // If conversation ID and channel ID was stored previously, use it.
    message.ChannelId = channelId;
}
else
{
    // Conversation ID was not stored previously, so create a conversation. 
    // Note: If the user has an existing conversation in a channel, this will likely create a new conversation window.
    conversationId = (await connector.Conversations.CreateDirectConversationAsync( botAccount, userAccount)).Id;
}

// Set the address-related properties in the message and send the message.
message.From = botAccount;
message.Recipient = userAccount;
message.Conversation = new ConversationAccount(id: conversationId);
message.Text = "Hello, this is a notification";
message.Locale = "en-us";
await connector.Conversations.SendToConversationAsync((Activity)message);
```

> [!NOTE]
> Если бот указывает идентификатор сеанса общения, который был сохранен ранее, скорее всего, сообщение будет доставлено пользователю в существующем окне сеанса общения в клиенте. Если бот создает новый идентификатор сеанса общения, то сообщение будет доставлено пользователю в новом окне сеанса общения в клиенте, при условии, что клиент поддерживает несколько окон сеансов общения. 

## <a name="send-a-dialog-based-proactive-message"></a>Отправка упреждающего сообщения на основе диалога

В следующих примерах кода показано, как отправить упреждающее сообщение на основе диалога с помощью пакета SDK Bot Framework для .NET.

Чтобы иметь возможность отправить упреждающее сообщение на основе диалога пользователю, боту сначала необходимо собрать (и сохранить) сведения из текущего сеанса общения. 

```cs
public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
{
    var message = await result;
    
    // Store information about this specific point the conversation, so that the bot can resume this conversation later.
    var conversationReference = message.ToConversationReference();
    ConversationStarter.conversationReference = JsonConvert.SerializeObject(conversationReference);

    await context.PostAsync("Greetings, user! I now know how to start a proactive message to you."); 
    context.Wait(MessageReceivedAsync);
}
```

Когда необходимо отправить сообщение, бот создает диалог и добавляет его в верхнюю часть стека диалогов. Новый диалог берет на себя управление общением, передает упреждающее сообщение, закрывается и возвращает управление предыдущему диалогу в стеке. 

```cs
// This will interrupt the conversation and send the user to SurveyDialog, then wait until that's done 
public static async Task Resume() 
{
    // Recreate the message from the conversation reference that was saved previously.
    var message = JsonConvert.DeserializeObject<ConversationReference>(conversationReference).GetPostToBotMessage(); 
    var client = new ConnectorClient(new Uri(message.ServiceUrl));

    // Create a scope that can be used to work with state from bot framework.
    using (var scope = DialogModule.BeginLifetimeScope(Conversation.Container, message))
    {
        var botData = scope.Resolve<IBotData>();
        await botData.LoadAsync(CancellationToken.None);

        // This is our dialog stack.
        var task = scope.Resolve<IDialogTask>();

        // Create the new dialog and add it to the stack.
        var dialog = new SurveyDialog();
        // interrupt the stack. This means that we're stopping whatever conversation that is currently happening with the user
        // Then adding this stack to run and once it's finished, we will be back to the original conversation
        task.Call(dialog.Void<object, IMessageActivity>(), null);
        
        await task.PollAsync(CancellationToken.None);

        // Flush the dialog stack back to its state store.
        await botData.FlushAsync(CancellationToken.None);        
    }
}
```
`SurveyDialog` управляет сеансом общения, пока он не завершается. После завершения своей задачи он вызывает `context.Done` и закрывается, возвращая управление предыдущему диалогу. 

```cs
[Serializable]
public class SurveyDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("Hello, I'm the survey dialog. I'm interrupting your conversation to ask you a question. Type \"done\" to resume");

        context.Wait(this.MessageReceivedAsync);
    }
    public virtual async Task MessageReceivedAsync(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        if ((await result).Text == "done")
        {
            await context.PostAsync("Great, back to the original conversation!");
            context.Done(String.Empty); // Finish this dialog.
        }
        else
        {
            await context.PostAsync("I'm still on the survey until you type \"done\"");
            context.Wait(MessageReceivedAsync); // Not done yet.
        }
    }
}
```

## <a name="sample-code"></a>Пример кода

Полный пример, в котором показано, как отправлять упреждающие сообщения с помощью пакета SDK Bot Framework для .NET, приведен в <a href="https://aka.ms/proactive-messaging-cs-v3 " target="_blank">примере Proactive Messages</a> на сайте GitHub. В примере proactiveMessages <a href="https://aka.ms/proactive-sendmessage-cs-v3 " target="_blank">simpleSendMessage</a> показывает, как отправить нерегламентированное упреждающее сообщение, а <a href="https://aka.ms/proactive-newdialog-cs-v3 " target="_blank">startNewDialog</a> показывает, как отправить упреждающее сообщение на основе диалога. 

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Проектирование потока диалога и управление им](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочная информация по пакету SDK Bot Framework для .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages" target="_blank">Пример proactiveMessages (GitHub)</a>


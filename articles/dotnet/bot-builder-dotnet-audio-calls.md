---
title: Осуществление голосовых вызовов через Skype | Документы Майкрософт
description: Узнайте, как совершать голосовые вызовы в Skype с помощью пакета SDK для построителя ботов для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 9e555de909275a991294cf093c4a76b7035b4a13
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305558"
---
# <a name="conduct-audio-calls-with-skype"></a>Осуществление голосовых вызовов через Skype

[!INCLUDE [Introduction to conducting audio calls](../includes/snippet-audio-call-intro.md)]

Архитектура бота, поддерживающего голосовые вызовы, очень похожа на архитектуру обычного бота. В следующих примерах кода показано, как включить поддержку голосовых вызовов через Skype с помощью пакета SDK для построителя ботов для .NET. 

## <a name="enable-support-for-audio-calls"></a>Включение поддержки голосовых вызовов

Чтобы включить поддержку голосовых вызовов в боте, определите `CallingController`.

```cs
[BotAuthentication]
[RoutePrefix("api/calling")]
public class CallingController : ApiController
{
    public CallingController() : base()
    {
        CallingConversation.RegisterCallingBot(callingBotService => new IVRBot(callingBotService));
    }

    [Route("callback")]
    public async Task<HttpResponseMessage> ProcessCallingEventAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.CallingEvent);
    }

    [Route("call")]
    public async Task<HttpResponseMessage> ProcessIncomingCallAsync()
    {
        return await CallingConversation.SendAsync(this.Request, CallRequestType.IncomingCall);
    }
}
```

> [!NOTE]
> В дополнение к `CallingController`, который поддерживает голосовые вызовы, бот также может содержать `MessagesController` для поддержки сообщений. Если вы предоставите оба варианта, пользователи сами смогут выбирать способ общения с ботом. <!-- docs on MessagesController are where? -->

##  <a name="answer-the-call"></a>Ответить на звонок

Задача `ProcessIncomingCallAsync` будет выполняться каждый раз, когда пользователь будет звонить боту через Skype.
Конструктор регистрирует класс `IVRBot`, который содержит предопределенный обработчик для `incomingCallEvent`.

Первое действие в рабочем процессе должно определять, следует боту ответить на вызов или отклонить его. Этот рабочий процесс сообщает боту, что нужно ответить на вызов и воспроизвести приветственное сообщение. 

```cs
private Task OnIncomingCallReceived(IncomingCallEvent incomingCallEvent)
{
    this.callStateMap[incomingCallEvent.IncomingCall.Id] = new CallState(incomingCallEvent.IncomingCall.Participants);

    incomingCallEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        new Answer { OperationId = Guid.NewGuid().ToString() },
        GetPromptForText(WelcomeMessage)
    };

    return Task.FromResult(true);
}
```

## <a name="after-the-bot-answers"></a>После ответа бота

Если бот отвечает на вызов, последующие действия, указанные в рабочем процессе, сообщают **платформе бота Skype для вызовов**, что нужно воспроизвести запрос, записать аудио, распознать речь или записать цифры с клавиатуры. Последним действием рабочего процесса может быть завершение вызова. 

В этом примере кода определяется обработчик, который настраивает меню после воспроизведения приветственного сообщения.

```cs
private Task OnPlayPromptCompleted(PlayPromptOutcomeEvent playPromptOutcomeEvent)
{
    var callState = this.callStateMap[playPromptOutcomeEvent.ConversationResult.Id];
    SetupInitialMenu(playPromptOutcomeEvent.ResultingWorkflow);
    return Task.FromResult(true);
}
```

Метод `CreateIvrOptions` определяет, какое меню будет предложено пользователю.

```cs
private static Recognize CreateIvrOptions(string textToBeRead, int numberOfOptions, bool includeBack)
{
    if (numberOfOptions > 9)
    {
        throw new Exception("too many options specified");
    }

    var choices = new List<RecognitionOption>();

    for (int i = 1; i <= numberOfOptions; i++)
    {
        choices.Add(new RecognitionOption { Name = Convert.ToString(i), DtmfVariation = (char)('0' + i) });
    }

    if (includeBack)
    {
        choices.Add(new RecognitionOption { Name = "#", DtmfVariation = '#' });
    }

    var recognize = new Recognize
    {
        OperationId = Guid.NewGuid().ToString(),
        PlayPrompt = GetPromptForText(textToBeRead),
        BargeInAllowed = true,
        Choices = choices
    };

    return recognize;
}
```

Класс `RecognitionOption` определяет устный ответ, а также соответствующий двухтональный многочастотный набор (DTMF). DTMF позволяет пользователю дать ответ с помощью соответствующих цифр на клавиатуре, а не устно.

Метод `OnRecognizeCompleted` обрабатывает выбор пользователя, а входной параметр `recognizeOutcomeEvent` содержит значение выбора пользователя.

```cs
private Task OnRecognizeCompleted(RecognizeOutcomeEvent recognizeOutcomeEvent)
{
    var callState = this.callStateMap[recognizeOutcomeEvent.ConversationResult.Id];
    ProcessMainMenuSelection(recognizeOutcomeEvent, callState);
    return Task.FromResult(true);
}
```

## <a name="support-natural-language"></a>Поддержка естественного языка
Бот может распознавать ответы на естественном языке. **API распознавания речи Bing** позволяет боту понимать слова из устного ответа пользователя.

```cs
private async Task OnRecordCompleted(RecordOutcomeEvent recordOutcomeEvent)
{
    recordOutcomeEvent.ResultingWorkflow.Actions = new List<ActionBase>
    {
        GetPromptForText(EndingMessage),
        new Hangup { OperationId = Guid.NewGuid().ToString() }
    };

    // Convert the audio to text
    if (recordOutcomeEvent.RecordOutcome.Outcome == Outcome.Success)
    {
        var record = await recordOutcomeEvent.RecordedContent;
        string text = await this.GetTextFromAudioAsync(record);

        var callState = this.callStateMap[recordOutcomeEvent.ConversationResult.Id];

        await this.SendSTTResultToUser("We detected the following audio: " + text, callState.Participants);
    }

    recordOutcomeEvent.ResultingWorkflow.Links = null;
    this.callStateMap.Remove(recordOutcomeEvent.ConversationResult.Id);
}
```

## <a name="sample-code"></a>Пример кода

Полный пример реализации поддержки голосовых вызовов через Skype с помощью пакета SDK для построителя ботов для .NET см. в разделе <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Пример бота Skype для вызовов</a> в GitHub.

## <a name="additional-resources"></a>Дополнительные ресурсы

- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочник по пакету SDK построителя ботов для .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/skype-CallingBot" target="_blank">Пример бота Skype для вызовов (GitHub)</a>

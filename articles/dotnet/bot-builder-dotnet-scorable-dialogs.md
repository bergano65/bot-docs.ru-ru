---
title: Глобальные обработчики сообщений, использующие элементы с возможностью оценки
description: Создавайте более гибкие диалоги, используя элементы с возможностью оценки в пакете SDK Bot Builder для .NET.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a6a69bd1001c51b44ad42824a0da6a079c5ac5a0
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999521"
---
# <a name="global-message-handlers-using-scorables"></a>Глобальные обработчики сообщений, использующие элементы с возможностью оценки

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Пользователи пытаются получить доступ к определенной функции бота, используя такие слова, как "help" (справка), "cancel" (отмена) или "start over" (начать сначала) посреди диалога, когда бот ожидает другой ответ. Используя диалоги с возможностью оценки, можно разработать бот, который будет корректно обрабатывать такие запросы.

Диалоги с возможностью оценки отслеживают все входящие сообщения и определяют, требуется ли какое-либо ответное действие. Каждый диалог с возможностью оценки присваивает оцениваемым сообщениям определенную оценку в диапазоне [0–1]. Диалог с возможностью оценки, определяющий высшую оценку, добавляется в верхнюю часть стека диалога, а затем передает ответ пользователю. По завершении выполнения диалога с возможностью оценки беседа продолжается с того места, на котором она остановилась.

Элементы с возможностью оценки помогают создавать более гибкие беседы, позволяя пользователям прерывать обычный ход беседы, который присутствует в стандартных диалогах.

## <a name="create-a-scorable-dialog"></a>Создание диалога с возможностью оценки

Во-первых, определите новый [диалог](bot-builder-dotnet-dialogs.md). В следующем коде используется диалог, который является производным от интерфейса `IDialog`.

```cs
public class SampleDialog : IDialog<object>
{
    public async Task StartAsync(IDialogContext context)
    {
        await context.PostAsync("This is a Sample Dialog which is Scorable. Reply with anything to return to the prior prior dialog.");

        context.Wait(this.MessageReceived);
    }

    private async Task MessageReceived(IDialogContext context, IAwaitable<IMessageActivity> result)
    {
        var message = await result;

        if ((message.Text != null) && (message.Text.Trim().Length > 0))
        {
            context.Done<object>(null);
        }
        else
        {
            context.Fail(new Exception("Message was not a string or was an empty string."));
        }
    }
}
```
Чтобы создать диалог с возможностью оценки, создайте класс, который наследуется от абстрактного класса `ScorableBase`. В приведенном ниже коде используется класс `SampleScorable`.

```cs
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.Internals.Fibers;
using Microsoft.Bot.Builder.Scorables.Internals;

public class SampleScorable : ScorableBase<IActivity, string, double>
{
    private readonly IDialogTask task;

    public SampleScorable(IDialogTask task)
    {
        SetField.NotNull(out this.task, nameof(task), task);
    }
}
```
Абстрактный класс `ScorableBase` наследуется от интерфейса `IScorable`. В классе нужно реализовать следующие методы `IScorable`:

- `PrepareAsync` — первый метод, который вызывается в экземпляре элемента с возможностью оценки. Он принимает действие из входящего сообщения, анализирует его и устанавливает состояние диалога, которое передается во все остальные методы интерфейса `IScorable`.

```cs
protected override async Task<string> PrepareAsync(IActivity item, CancellationToken token)
{
        // TODO: insert your code here
}
```

- Метод `HasScore` проверяет свойство состояния, чтобы определить, должен ли диалог с возможностью оценки оценивать сообщение. Если возвращается значение false, диалог с возможностью оценки проигнорирует сообщение.

```cs
protected override bool HasScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```

- `GetScore` активируется только в том случае, если `HasScore` возвращает значение true. В этом методе вы подготовите логику, чтобы определить для сообщения оценку в диапазоне 0–1.

```cs
protected override double GetScore(IActivity item, string state)
{
        // TODO: insert your code here
}
```
- В методе `PostAsync` определите основные действия, которые необходимо выполнить в классе с возможностью оценки. Все диалоги с возможностью оценки будут отслеживать входящие сообщения и назначать допустимым сообщениям оценки на основании собственного метода GetScore. Класс с возможностью оценки, который определяет наивысшую оценку (в диапазоне 0–1.0), затем активирует метод `PostAsync` этого диалога.

```cs
protected override Task PostAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

- `DoneAsync` вызывается по завершении процесса оценивания. С помощью этого метода можно удалить все ресурсы с областью действия.

```cs
protected override Task DoneAsync(IActivity item, string state, CancellationToken token)
{
        //TODO: insert your code here
}
```

## <a name="create-a-module-to-register-the-iscorable-service"></a>Создание модуля для регистрации службы IScorable

На следующем этапе определите модуль `Module`, который зарегистрирует класс `SampleScorable` в качестве компонента. Это позволит подготовить службу `IScorable`.

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SampleScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```
## <a name="register-the-module"></a>Регистрация модуля  

Последним шагом процесса является применение диалога с возможностью оценки `SampleScorable` к контейнеру диалога бота. Это позволит зарегистрировать службу с возможностью оценки в конвейере обработки сообщений Bot Framework. В следующем коде показано обновление `Conversation.Container` в процессе инициализации приложения бота в **Global.asax.cs**:

```cs
public class WebApiApplication : System.Web.HttpApplication
{
    protected void Application_Start()
    {
        this.RegisterBotModules();
        GlobalConfiguration.Configure(WebApiConfig.Register);
    }

    private void RegisterBotModules()
    {
        var builder = new ContainerBuilder();
        builder.RegisterModule(new ReflectionSurrogateModule());

        //Register the module within the Conversation container
        builder.RegisterModule<GlobalMessageHandlersBotModule>();

        builder.Update(Conversation.Container);
    }
}
```

## <a name="additional-resources"></a>Дополнительные ресурсы
* [Global Message Handlers Sample (GitHub)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers) (Пример глобальных обработчиков сообщений (GitHub))
* [Simple Scorable Bot sample](https://github.com/Microsoft/BotFramework-Samples/tree/master/blog-samples/CSharp/ScorableBotSample) (Пример простого бота с возможностью оценки)
* [Диалоги в пакете SDK построителя ботов для .NET](bot-builder-dotnet-dialogs.md)
* [Autofac](https://autofac.org/)

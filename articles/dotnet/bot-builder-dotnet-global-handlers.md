---
title: Реализация глобальных обработчиков сообщений | Документация Майкрософт
description: Сведения о том, как включить бот для прослушивания и обработки введенных пользователем данных, содержащих определенные ключевые слова, с помощью пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 86964bc39a95a23f397af649cfac6e2784dd588a
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224369"
---
# <a name="implement-global-message-handlers"></a>Реализация глобальных обработчиков сообщений

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

[!INCLUDE [Introduction to global message handlers](../includes/snippet-global-handlers-intro.md)]

## <a name="listen-for-keywords-in-user-input"></a>Прослушивание ключевых слов во входных данных пользователя

В следующем пошаговом руководстве показано, как реализовать глобальные обработчики сообщений с помощью пакета SDK Bot Framework для .NET.

Во-первых, `Global.asax.cs` регистрирует модуль `GlobalMessageHandlersBotModule`, реализованный, как показано ниже. В этом примере модуль регистрирует два элемента с возможностью оценки: один для управления запросом на изменение параметров (`SettingsScorable`), другой — для управления запросом на отмену (`CancelScoreable`).

```cs
public class GlobalMessageHandlersBotModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        base.Load(builder);

        builder
            .Register(c => new SettingsScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();

        builder
            .Register(c => new CancelScorable(c.Resolve<IDialogTask>()))
            .As<IScorable<IActivity, double>>()
            .InstancePerLifetimeScope();
    }
}
```

`CancelScorable` содержит метод `PrepareAsync`, который определяет триггер: при получении сообщения с текстом Cancel (Отмена) будет запущен этот элемент с возможностью оценки.

```cs
protected override async Task<string> PrepareAsync(IActivity activity, CancellationToken token)
{
    var message = activity as IMessageActivity;
    if (message != null && !string.IsNullOrWhiteSpace(message.Text))
    {
        if (message.Text.Equals("cancel", StringComparison.InvariantCultureIgnoreCase))
        {
            return message.Text;
        }
    }
    return null;
}
```

В случае получения запроса с текстом Cancel (Отмена) метод `PostAsync` в `CancelScoreable` сбрасывает стек диалога. 

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    this.task.Reset();
}
```

При получении запроса Change Settings (Изменить параметры) метод `PostAsync` в `SettingsScorable` вызывает `SettingsDialog` (передавая запрос в этот диалог), тем самым добавляя `SettingsDialog` в верхнюю часть стека диалога и передавая его под управление диалога.

```cs
protected override async Task PostAsync(IActivity item, string state, CancellationToken token)
{
    var message = item as IMessageActivity;
    if (message != null)
    {
        var settingsDialog = new SettingsDialog();
        var interruption = settingsDialog.Void<object, IMessageActivity>();
        this.task.Call(interruption, null);
        await this.task.PollAsync(token);
    }
}
```

## <a name="sample-code"></a>Пример кода

Полный пример, в котором показано, как реализовывать глобальные обработчики сообщений с помощью пакета SDK Bot Framework для .NET, см. в примере <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">Global Message Handlers Sample</a> (Пример глобальных обработчиков сообщений) на портале GitHub.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Проектирование потока диалога и управление им](../bot-service-design-conversation-flow.md)
- <a href="/dotnet/api/?view=botbuilder-3.12.2.4" target="_blank">Справочная информация по пакету SDK Bot Framework для .NET</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GlobalMessageHandlers" target="_blank">Global Message Handlers Sample (GitHub)</a> (Пример глобальных обработчиков сообщений (GitHub))

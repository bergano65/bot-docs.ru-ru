---
title: Как запустить бот, созданный с помощью пакета SDK версии 3 для .NET, с пакетом SDK версии 4 | Документация Майкрософт
description: Узнайте, как преобразовать бот из версии 3.x в 4.0, используя классический пакет NuGet.
keywords: migration, classic bot, convert v3, v3 to v4
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/25/18
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a808076e4865a181802b85cfc24ce342dbf23cba
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301493"
---
# <a name="how-to-run-net-sdk-v3-bots-in-sdk-40"></a>Как запускать боты, созданные с помощью пакета SDK версии 3 для .NET, с пакетом SDK версии 4.0

Пакет NuGet **Microsoft.Bot.Builder.Classic** упрощает перенос ботов из Microsoft Bot Framework версии 3.x в версию 4.0.

**Примечание.** Этот процесс можно выполнить только с ботами **веб-приложения ASP.NET (.NET Framework)**. Он не поддерживается для ботов **веб-приложения ASP.NET Core**.

## <a name="the-process"></a>Процесс

Процесс относительно прост:

- Добавьте в проект пакет NuGet **Microsoft.Bot.Builder.Classic**.
    - Возможно, вам также потребуется обновить пакет NuGet **Autofac**.
- Обновите пространство имен **Microsoft.Bot.Builder.Classic**.
- Вызовите диалоги, используя **Conversation.SendAsync()** на основе **ITurnContext** в версии 4.0.

### <a name="add-the-microsoftbotbuilderclassic-nuget-package"></a>Добавление пакета NuGet Microsoft.Bot.Builder.Classic

Чтобы добавить пакет NuGet **Microsoft.Bot.Builder.Classic**, используйте элемент **Управление пакетами NuGet**.

### <a name="update-the-namespaces"></a>Обновление пространства имен

Чтобы обновить пространство имен, удалите все найденные следующие инструкции `using`:

```csharp
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
using Microsoft.Bot.Builder.FormFlow;
using Microsoft.Bot.Builder.Scorables;
```

Вместо них добавьте такие инструкции `using`:

```csharp
using Microsoft.Bot.Builder.Adapters;
using Microsoft.Bot.Builder.Classic.Dialogs;
using Microsoft.Bot.Builder.Classic.Dialogs.Internals;
using Microsoft.Bot.Builder.Classic.FormFlow;
using Microsoft.Bot.Builder.Classic.Scorables;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Schema;
```

Если есть строка `[BotAuthentication]`, удалите или закомментируйте ее.

### <a name="invoke-your-3x-dialog"></a>Вызов диалога, созданного в версии 3.x

Чтобы вызвать диалог, созданный в версии 3.x, также используется `Conversation.SendAsync`. Только теперь вместо объекта **Activity** применяется **ITurnContext** из версии 4.0.

```csharp
// invoke a Classic V3 IDialog 
await Conversation.SendAsync(turnContext, () => new EchoDialog());
```

Если объект **ITurnContext** отсутствует, но есть объект **Activity**, **ITurnContext** можно получить следующими образом:

```csharp
BotFrameworkAdapter adapter = new BotFrameworkAdapter("", "");

await adapter.ProcessActivity(this.Request.Headers.Authorization?.Parameter,
        activity,
        async (context) =>
        {
            // Do something with context here. For example, the body of your Post() method may go here.
        });
```

## <a name="fix-assembly-conflicts"></a>Устранение конфликтов сборок

В ботах, созданных с помощью классического пакета NuGet, могут возникнуть конфликты сборок. Сведения об этих конфликтах появятся в окне "Список ошибок" в Visual Studio после компиляции.

### <a name="if-you-see-warning-found-conflicts-between-different-versions-of-the-same-dependent-assembly"></a>Действия при получении предупреждения о том, что обнаружены конфликты между различными версиями одной и той же зависимой сборки

Если поступит предупреждение с текстом **Found conflicts between different versions of the same dependent assembly** (Обнаружены конфликты между различными версиями одной и той же зависимой сборки), сделайте следующее:

- Дважды щелкните предупреждающее сообщение. Появится диалоговое окно с вопросом "Do you want to fix these conflicts by adding binding redirect records in the application configuration file?" (Вы хотите устранить эти конфликты при помощи добавления связывающих перенаправляющих записей в файл конфигурации приложения?).
- Щелкните "Да".
- Скомпилируйте проект еще раз.

### <a name="if-you-see-error-missing-method-exception-on-startup"></a>Действия при получении сообщения о том, что отсутствует исключение метода при запуске

Эта ошибка с .NET Standard происходит, когда вы обновляете старый проект .NET 4.6 до версии 4.6.1, а затем пытаетесь использовать с ним библиотеку .NET Standard. Причина в том, что выполняется попытка динамически выгрузить 2 разные сборки System.Net.Http. Эту проблему можно решить, создав для System.Net.Http связывающее перенаправление на Web.config. 

Если возникнет такая ошибка, добавьте в файл Web.config следующий код:

```xml
<dependentAssembly>
    <assemblyIdentity name="System.Net.Http" publicKeyToken="B03F5F7F11D50A3A" culture="neutral" />
    <bindingRedirect oldVersion="0.0.0.0-4.2.0.0" newVersion="4.2.0.0" />
</dependentAssembly>
```

Дополнительные сведения об этой проблеме см. в разделе [System.Net.Http v4.2.0.0 being copied/loaded from MSBuild tooling #25773](https://github.com/dotnet/corefx/issues/25773) (Копирование или загрузка System.Net.Http версии 4.2.0.0 из инструмента MSBuild #25773).

## <a name="sample-of-a-converted-bot"></a>Пример преобразованного бота

Вы можете просмотреть пример [EchoBot-Classic](https://github.com/Microsoft/botbuilder-dotnet/tree/master/samples/Microsoft.Bot.Samples.EchoBot-Classic), в котором показан бот версии 3.x, преобразованный для работы в версии 4.0.

## <a name="limitations"></a>Ограничения
Библиотека Microsoft.Bot.Builder.Classic поддерживается только для .NET 4.6.1. Она не предназначена для работы в .NET Core.

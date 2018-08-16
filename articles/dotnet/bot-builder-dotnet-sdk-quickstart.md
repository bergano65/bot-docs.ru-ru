---
title: Создание бота с помощью пакета SDK Bot Builder для .NET | Документация Майкрософт
description: Создание бота с помощью пакета SDK Bot Builder для .NET — мощной платформы для создания ботов.
keywords: Пакет SDK для Bot Builder, создание бота, краткое руководство, .NET, приступая к работе
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 98c6103f0cd38bf6d3f477735dafcf01952304d5
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39306235"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-for-net"></a>Создание бота с помощью пакета SDK Bot Builder версии 4 для .NET
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

В этом кратком руководстве описывается создание бота с помощью шаблона приложения бота и пакета SDK Bot Builder для .NET, а также его тестирование с помощью Bot Framework Emulator. Это руководство основано на [пакете SDK для Microsoft Bot Builder версии 4](https://github.com/Microsoft/botbuilder-dotnet).

## <a name="prerequisites"></a>Предварительные требования
- Visual Studio [2017](https://www.visualstudio.com/downloads).
- Шаблон пакета SDK для Bot Builder версии 4 для [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.botbuilderv4).
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases).
- Навыки разработки для [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) и асинхронного программирования в [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index).

## <a name="create-a-bot"></a>Создание бота
Установите шаблон BotBuilderVSIX.vsix, скачанный при выполнении предварительных требований. 

В Visual Studio создайте проект бота.

![Проект Visual Studio](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> При необходимости обновите [пакеты NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio).

## <a name="explore-code"></a>Изучение кода
Откройте файл Startup.cs, чтобы просмотреть код метода `ConfigureServices(IServiceCollection services)`. ПО промежуточного слоя `CatchExceptionMiddleware` добавлено в конвейер обмена сообщениями. Оно обрабатывает все исключения, порождаемые другим ПО промежуточного слоя или методом OnTurn. 

```cs
options.Middleware.Add(new CatchExceptionMiddleware<Exception>(async (context, exception) =>
{
    await context.TraceActivity("EchoBot Exception", exception);
    await context.SendActivity("Sorry, it looks like something went wrong!");
}));
```

ПО промежуточного слоя состояния общения использует хранилище в памяти. Оно считывает и записывает значение EchoState в хранилище.  Счетчик шагов в классе EchoState отслеживает количество сообщений, отправленных боту. Аналогичный прием можно использовать для поддержания состояния между шагами.

```cs
 IStorage dataStore = new MemoryStorage();
 options.Middleware.Add(new ConversationState<EchoState>(dataStore));
```

Метод `Configure(IApplicationBuilder app, IHostingEnvironment env)` вызывает `.UseBotFramework` для передачи входящих действий в адаптер бота. 

Метод `OnTurn(ITurnContext context)` в файле EchoBot.cs используется для проверки типа входящего действия и отправки ответа пользователю. 

```cs
public async Task OnTurn(ITurnContext context)
{
    // This bot is only handling Messages
    if (context.Activity.Type == ActivityTypes.Message)
    {
        // Get the conversation state from the turn context
        var state = context.GetConversationState<EchoState>();

        // Bump the turn count. 
        state.TurnCount++;

        // Echo back to the user whatever they typed.
        await context.SendActivity($"Turn {state.TurnCount}: You sent '{context.Activity.Text}'");
    }
}
```
## <a name="start-your-bot"></a>Запуск бота

- Ваша страница `default.html` будет отображена в браузере.
- Запишите номер порта localhost страницы. Данная информация потребуется для взаимодействия с ботом.

### <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота

На этом этапе бот выполняется локально.
После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Create a new bot configuration** (Создать конфигурацию бота) на вкладке "Welcome" (Приветствие) эмулятора. 

2. Введите **имя бота** и укажите путь к каталогу, в котором расположен код бота. Файл конфигурации бота будет сохранен по этому пути.

3. Введите `http://localhost:port-number/api/messages` в поле **Endpoint URL** (URL-адрес конечной точки), где *port-number* соответствует номеру порта в браузере, в котором запущено приложение.

4. Щелкните **Connect** (Подключиться) для подключения к боту. Не нужно указывать **идентификатор приложения Майкрософт** и **пароль приложения Майкрософт**. Эти поля пока можно оставить пустыми. Вы получите эти сведения позднее, когда зарегистрируете бот.

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом

Отправьте сообщение "Hi" своему боту и, бот ответит на это сообщение: "Turn 1: You sent Hi".

## <a name="next-steps"></a>Дополнительная информация

Затем [разверните бот в Azure](../bot-builder-howto-deploy-azure.md) или перейдите к базовым понятиям, описывающим боты и принципы их работы.

> [!div class="nextstepaction"]
> [Базовые понятия о ботах](../v4sdk/bot-builder-basics.md)

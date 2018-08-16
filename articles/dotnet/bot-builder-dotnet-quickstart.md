---
title: Создание бота с помощью пакета SDK Bot Builder для .NET | Документация Майкрософт
description: Создание бота с помощью пакета SDK Bot Builder для .NET — мощной платформы для создания ботов.
keywords: Пакет SDK для Bot Builder, создание бота, краткое руководство, начало работы
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 04/27/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fcb21fa38750c09f110a3c71f763a941c4437979
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39306198"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>Создание бота с помощью пакета SDK Bot Builder для .NET
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Служба Azure Bot](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

<a href="https://github.com/Microsoft/BotBuilder" target="_blank">Пакет SDK Bot Builder для .NET</a> — это простая в использовании платформа для разработки ботов с помощью Visual Studio и Windows. Пакет SDK использует C#, позволяя разработчикам .NET создавать мощные боты.


В этом кратком руководстве описывается создание бота с помощью шаблона приложения бота и пакета SDK Bot Builder для .NET, а также тестирование его с помощью Bot Framework Emulator.

## <a name="prerequisites"></a>Предварительные требования
1. Visual Studio [2017](https://www.visualstudio.com/).

2. Обновите все [расширения](https://docs.microsoft.com/en-us/visualstudio/extensibility/how-to-update-a-visual-studio-extension) Visual Studio до последних версий.

3. Шаблон бота для [C#](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3).

> [!TIP]
> Каталог шаблонов проекта Visual Studio 2017 обычно находится в `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ProjectTemplates\Visual C#\`, а каталог шаблонов элемента — в `%USERPROFILE%\Documents\Visual Studio 2017\Templates\ItemTemplates\Visual C#\`

## <a name="create-your-bot"></a>Создание бота

Откройте Visual Studio и создайте новый проект C#. Выберите шаблон **Simple Echo Bot Application** для нового проекта.

![Создание проекта в Visual Studio](../media/connector-getstarted-create-project.png)

> [!NOTE]
> Visual Studio может потребовать загрузки и установки [IIS Express](https://www.microsoft.com/en-us/download/details.aspx?id=48264). 

Благодаря шаблону приложения бота проект содержит весь код, необходимый для создания бота в этом руководстве. Теперь нет необходимости писать дополнительный код. Тем не менее, прежде чем перейти к тестированию бота, быстро просмотрите код, предоставленный шаблоном приложения бота.

> [!TIP] 
> При необходимости обновите [пакеты NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio).

## <a name="explore-the-code"></a>Обзор кода

Во-первых, метод `Post` в **Controllers\MessagesController.cs** получает сообщение от пользователя и вызывает корневое диалоговое окно.

```csharp
public async Task<HttpResponseMessage> Post([FromBody]Activity activity)
{
    if (activity.GetActivityType() == ActivityTypes.Message)
    {
        await Conversation.SendAsync(activity, () => new Dialogs.RootDialog());
    }
    else
    {
        HandleSystemMessage(activity);
    }
    var response = Request.CreateResponse(HttpStatusCode.OK);
    return response;
}

```

Корневое диалоговое окно обрабатывает сообщение и генерирует ответ. Метод `MessageReceivedAsync` в **Dialogs\RootDialog.cs** отправляет ответ, который возвращает сообщение пользователя, которое начинается с текста "You sent" и заканчивается текстом "which was *##* characters", где *##* представляет количество символов в сообщении пользователя.

```csharp
public class RootDialog : IDialog<object>
{
    public Task StartAsync(IDialogContext context)
    {
        context.Wait(MessageReceivedAsync);

        return Task.CompletedTask;
    }

    private async Task MessageReceivedAsync(IDialogContext context, IAwaitable<object> result)
    {
        var activity = await result as Activity;

        // Calculate something for us to return
        int length = (activity.Text ?? string.Empty).Length;

        // Return our reply to the user
        await context.PostAsync($"You sent {activity.Text} which was {length} characters");

        context.Wait(MessageReceivedAsync);
    }
}
```

## <a name="test-your-bot"></a>Тестирование бота

[!INCLUDE [Get started test your bot](../includes/snippet-getstarted-test-bot.md)]

### <a name="start-your-bot"></a>Запуск бота

После установки эмулятора запустите бот в Visual Studio, используя браузер в качестве узла приложения.
Этот снимок экрана Visual Studio показывает, что бот будет запускаться в Microsoft Edge при нажатии кнопки запуска.

![Проект запуска Visual Studio](../media/connector-getstarted-start-bot-locally.png)

При нажатии кнопки запуска Visual Studio будет создавать приложение, развертывать его в узле localhost и запускать веб-браузер для отображения страницы приложения **default.htm**.
Например, представление страницы приложения **default.htm**, показанной в Microsoft Edge.

![Узел localhost запуска бота в Visual Studio](../media/connector-getstarted-bot-running-localhost.png)

> [!NOTE]
> Можно изменить файл **default.htm** проекта, чтобы указать имя и описание приложения бота.

### <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота

На этом этапе бот запускается локально.
После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Создайте новую конфигурацию бота. Введите `http://localhost:port-number/api/messages` в адресной строке, где значение *port-number* должно соответствовать номеру порта, указанному в браузере, где запускается приложение.

2. Щелкните **Сохранить и подключиться**. Не нужно указывать **идентификатор приложения Майкрософт** и **пароль приложения Майкрософт**. Эти поля пока можно оставить пустыми. Вы получите эти сведения позже, когда выполните [регистрацию бота](~/bot-service-quickstart-registration.md).

### <a name="test-your-bot"></a>Тестирование бота

Теперь, когда бот работает локально и подключен к эмулятору, проверьте бот, введя несколько сообщений в эмуляторе.
Будет видно, что бот отвечает на каждое отправленное сообщение, повторяя ваше сообщение, начинающееся с текста "You sent" и заканчивая текстом "which was *##* characters", где *##* — общее количество символов в отправляемом сообщении.


> [!TIP]
> В эмуляторе нажмите на любое облачко с текстом общения. Сведения о сообщении появятся на панели сведений в формате JSON.

Вы успешно создали бота с помощью шаблона приложения бота пакета SDK Bot Builder для .NET!

## <a name="next-steps"></a>Дополнительная информация

В этом кратком руководстве был создан простой бот с помощью шаблона приложения бота и пакета SDK Bot Builder для .NET и проверена функциональность бота с помощью Bot Framework Emulator.

Ознакомьтесь с основными понятиями пакета SDK Bot Builder для .NET.

> [!div class="nextstepaction"]
> [Key concepts in the Bot Builder SDK for .NET](bot-builder-dotnet-concepts.md) (Основные понятия пакета SDK Bot Builder для .NET)

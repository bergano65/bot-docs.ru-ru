---
title: Создание клиента .NET для подключения к расширению Службы приложений Direct Line
titleSuffix: Bot Service
description: Клиент .NET на C# для подключения к расширению Службы приложений Direct Line
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: 1b3b11723cc93c32aef92250518d2ffc16435460
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757786"
---
## <a name="create-net-client-to-connect-to-direct-line-app-service-extension"></a>Создание клиента .NET для подключения к расширению Службы приложений Direct Line

В этой статье описано, как создать клиент .NET на языке C#, который подключается к расширению Службы приложений Direct Line.

### <a name="gather-your-direct-line-extension-keys"></a>Сбор ключей расширения Direct Line

1. В браузере перейдите на [портал Azure](https://portal.azure.com/).
1. На портале Azure найдите ресурс **службы Azure Bot**.
1. Щелкните элемент **Каналы**, чтобы настроить каналы для бота.
1. Если канал **Direct Line** еще не включен, щелкните его, чтобы включить. 
1. Если он уже включен, в таблице "Подключение к каналам" щелкните ссылку **Изменить** в строке Direct Line.
1. Прокрутите вниз до раздела "Сайты". Обычно здесь отображается сайт по умолчанию, если вы его не удалили и не переименовали.
1. Щелкните **Показать ссылку**, чтобы отобразить один из ключей, и скопируйте его значение.

![Ключи расширения Службы приложений](./media/channels/direct-line-extension-extension-keys-net-client.png)

> [!NOTE]
> Клиент Direct Line использует это значение секрета для подключения к расширению Службы приложений Direct Line. При желании вы можете создать дополнительные сайты и указать здесь их секреты.

### <a name="add-the-preview-nuget-package-source"></a>Добавление источника предварительных версий пакета NuGet

Пакеты NuGet предварительной версии, необходимые для создания клиента Direct Line на C#, можно получить из веб-канала NuGet.

1. В Visual Studio перейдите к пункту меню **Сервис -> Параметры**.
1. Выберите **Диспетчер пакетов NuGet -> Источники пакетов**.
1. Нажмите кнопку "+", чтобы добавить новый источник пакета со следующими значениями.
    - Имя: DL ASE Preview
    - Источник: https://botbuilder.myget.org/F/experimental/api/v3/index.json
1. Чтобы сохранить новые значения, нажмите кнопку **Обновить**.
1. Щелкните **ОК**, чтобы выйти из режима настройки источника пакетов.

### <a name="create-a-c-direct-line-client"></a>Создание клиента Direct Line на C#

Взаимодействие с расширением службы приложений Direct существенно отличается от обычной работы с Direct Line, так как значительная часть взаимодействия выполняется через *WebSocket*. Обновленный клиент Direct Line содержит вспомогательные классы для открытия и закрытия *WebSocket*, отправки команд через WebSocket и получения действий от бота. В этом разделе описывается, как создать простой клиент C# для взаимодействия с ботом.

1. Создайте проект консольного приложения .NET Core 2.2 в Visual Studio.
1. Добавьте в этот проект **клиент NuGet Direct Line** .
    - Щелкните "Зависимости" в дереве решения.
    - Выберите **Manage NuGet Packages...** (Управление пакетами NuGet...).
    - Укажите тот источник пакета, который вы определили выше (DL ASE Preview).
    - Найдите пакет *Microsoft.Bot.Connector.Directline* версии v3.0.3-Preview1 или более поздней.
    - Щелкните **Установить пакет**.
1. Создайте клиент и маркер для него с помощью секрета. Этот шаг будет одинаковым для любого клиента Direct Line на C#, не считая значения конечной точки, которое нужно указать в боте с добавлением пути **.bot/** , как показано далее. Не забудьте **/** в конце.

    ```csharp
    string endpoint = "https://<YOUR_BOT_HOST>.azurewebsites.net/.bot/";
    string secret = "<YOUR_BOT_SECRET>";

    var tokenClient = new DirectLineClient(
        new Uri(endpoint),
        new DirectLineClientCredentials(secret));
    var conversation = await tokenClient.Tokens.GenerateTokenForNewConversationAsync();
    ```

1. Получив ссылку на беседу после создания маркера, вы сможете с помощью идентификатора этой беседы открыть WebSocket с новым свойством `StreamingConversations` для `DirectLineClient`. Для этого следует создать обратный вызов, который будет использоваться в боте для отправки клиенту `ActivitySets`:

    ```csharp
    public static void ReceiveActivities(ActivitySet activitySet)
    {
        if (activitySet != null)
        {
            foreach (var a in activitySet.Activities)
            {
                if (a.Type == ActivityTypes.Message && a.From.Id.Contains("bot"))
                {
                    Console.WriteLine($"<Bot>: {a.Text}");
                }
            }
        }
    }
    ```

1. Теперь все готово к тому, чтобы открыть WebSocket в свойстве `StreamingConversations` с использованием маркера диалога `conversationId` и только что созданного обратного вызова `ReceiveActivities`:

    ```csharp
    var client = new DirectLineClient(
        new Uri(endpoint),
        new DirectLineClientCredentials(conversation.Token));

    await client.StreamingConversations.ConnectAsync(
        conversation.ConversationId,
        ReceiveActivities);
    ```

1. Теперь этот клиент можно использовать для запуска диалога и отправки `Activities` боту:

    ```csharp

    var startConversation = await client.StreamingConversations.StartConversationAsync();
    var from = new ChannelAccount() { Id = "123", Name = "Fred" };
    var message = Console.ReadLine();

    while (message != "end")
    {
        try
        {
            var response = await client.StreamingConversations.PostActivityAsync(
                startConversation.ConversationId,
                new Activity()
                {
                    Type = "message",
                    Text = message,
                    From = from
                });
        }
        catch (OperationException ex)
        {
            Console.WriteLine(
                $"OperationException when calling PostActivityAsync: ({ex.StatusCode})");
        }
        message = Console.ReadLine();
    }
    ```

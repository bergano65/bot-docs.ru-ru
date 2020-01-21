---
title: Использование пользовательского состояния .NET версии 3 в боте версии 4 — Служба Azure Bot
description: Использование данных о состоянии пользователя версии 3 в боте версии 4
keywords: Csharp, миграция бота, бот версии 3
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/21/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 51aef13bee896feeb901040cef641f7b1011dbd5
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75798087"
---
# <a name="using-net-v3-user-state-in-a-v4-bot"></a>Использование данных .NET о состоянии пользователя версии 3 в боте версии 4

В этой статье описано, как бот версии 4 может выполнять с информацией о состоянии пользователя версии 3 операции чтения, записи и удаления.
Бот поддерживает состояние беседы с помощью `MemoryStorage` для ее отслеживания, управления ее ходом и отправки вопросов пользователю.  Он сохраняет **состояние пользователя** в формате версии 3 для отслеживания ответов пользователя с помощью настраиваемого класса `IStorage` с именем `V3V4Storage`.  Одним из аргументов этого класса является `IBotDataStore`. База кода пакета SDK версии 3 была скопирована в `Bot.Builder.Azure.V3V4` и содержит всех три поставщика хранилищ из пакета SDK версии 3 (Azure SQL, таблицы Azure и Cosmos DB).  Это нужно, чтобы разрешить внедрение существующего **состояния пользователя** версии 3 в перенесенного бота версии 4.

Пример кода можно найти [здесь](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/V4StateBotFromV3Providers).

## <a name="prerequisites"></a>предварительные требования

- [Пакет SDK для .NET Core](https://dotnet.microsoft.com/download) версии 2.1.

    ```bash
    # determine dotnet version
    dotnet --version
    ```

## <a name="setup"></a>Настройка

1. Клонирование репозитория

    ```bash
    git clone https://github.com/microsoft/botbuilder-samples.git
    ```

1. В окне терминала перейдите в папку `MigrationV3V4/CSharp/V4StateBotFromV3Providers`.

    ```bash
    cd BotBuilder-Samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers
    ```

1. Запустите бота из окна терминала или из Visual Studio и выберите вариант A или B.

    - В окне терминала

        ```bash
        # run the bot
        dotnet run
        ```

    - Или из Visual Studio

        - Откройте Visual Studio и выберите "Файл" -> "Открыть" -> "Проект или решение".
        - Перейдите в папку `BotBuilder-Samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers`.
        - Выберите файл `V4StateBot.sln`.
        - Нажмите клавишу F5, чтобы запустить проект.


## <a name="storage-provider-setup"></a>Настройка поставщика хранилища

Предполагается, что вам доступно настроенное и используемое существующее хранилище состояний версии 3. В таком случае для подготовки этого примера с использованием существующего хранилища состояний просто нужно добавить сведения о подключениях поставщика хранилища в файл `web.config`, как показано далее.

- Cosmos DB

```json
  "v3CosmosEndpoint": "https://yourcosmosdb.documents.azure.com:443/",
  "v3CosmosKey": "YourCosmosDbKey",
  "v3CosmosDatataseName": "v3botdb",
  "v3CosmosCollectionName": "v3botcollection",
```

- Azure SQL

```json
 "ConnectionStrings": {
    "SqlBotData": "Server=YourServer;Initial Catalog=BotData;Persist Security Info=False;User ID=YourUserName;Password=YourUserPassword;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=True;Connection Timeout=30;"
  },
```

- таблице Azure

```json
 "ConnectionStrings": {
    "AzureTable": "DefaultEndpointsProtocol=https;AccountName=YourAccountName;AccountKey=YourAccountKey;EndpointSuffix=core.windows.net"
  },
```

- Укажите поставщика хранилища для бота.

    Откройте файл `Startup.cs` в корне проекта `V4V3StateBot`. В середине файла (приблизительно строки 52–76) хранятся настройки для каждого поставщика хранилищ. Они считывают значения конфигурации из файла web.config. 

    [!code-csharp[Storage configuration](~/../botbuilder-samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers/V4V3StateBot/Startup.cs?range=52-76)]

    Укажите поставщика хранилищ, который будет использоваться ботом, раскомментировав соответствующие строки для выбранного экземпляра. После настройки поставщика убедитесь, что его класс передается `V3V4Storage` (приблизительно строки 72–75). 

    [!code-csharp[Storage provider](~/../botbuilder-samples/MigrationV3V4/CSharp/V4StateBotFromV3Providers/V4V3StateBot/Startup.cs?range=72-75)]

    По умолчанию задается Cosmos DB (ранее — Document DB). Вы можете выбрать

    ```bash
    documentDbBotDataStore
    tableBotDataStore
    tableBotDataStore2
    sqlBotDataStore
    ```

- Запустите приложение. 

## <a name="v3v4-storage-and-state-classes"></a>V3V4 Storage и классы состояния

### <a name="v3v4storage"></a>V3V4Storage

Класс `V3V4Storage` содержит основные функции сопоставления хранилищ. Он реализует интерфейс `IStorage` версии 4 и сопоставляет методы поставщика хранилищ (чтение, запись и удаление) с классами поставщика хранилищ версии 3, чтобы состояние пользователя в формате версии 3 можно было использовать в боте версии 4.

### <a name="v3v4state"></a>V3V4State

Этот класс наследуется от класса `BotState` версии 4 и использует ключ в стиле версии 3 (`IAddress`). Это позволяет выполнять операции чтения, записи и удаления с хранилищем версии 3 аналогично тому, как они реализованы для хранилища состояний версии 3.


## <a name="testing-the-bot-using-bot-framework-emulator"></a>Тестирование бота с помощью Bot Framework Emulator

[Bot Framework Emulator][5] — это классическое приложение, которое позволяет разработчикам бота локально или удаленно (через туннель) тестировать боты или выполнять их отладку.

- Установите Bot Framework Emulator версии 4.3.0 или выше, скачав ее [здесь][6].


### <a name="connect-to-the-bot-using-bot-framework-emulator"></a>Подключение к боту с помощью Bot Framework Emulator

- Запустите Bot Framework Emulator.
- Выберите "Файл" -> "Открыть бота".
- Введите URL-адрес бота `http://localhost:3978/api/messages`.


## <a name="further-reading"></a>Дополнительные материалы

- [Общие сведения о службе Azure Bot][21]
- [Состояние бота][7]
- [Непосредственная запись в хранилище][8]
- [Управление состоянием беседы и пользователя][9]

[3]: https://aka.ms/botframework-emulator
[5]: https://github.com/microsoft/botframework-emulator
[6]: https://github.com/Microsoft/BotFramework-Emulator/releases
[7]: https://docs.microsoft.com/azure/bot-service/bot-builder-storage-concept
[8]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-storage?tabs=csharp
[9]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-state?tabs=csharp
[21]: https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[40]: https://aka.ms/azuredeployment

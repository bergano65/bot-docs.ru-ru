---
title: Использование данных JavaScript о состоянии пользователя версии 3 в боте версии 4 — Служба Azure Bot
description: Использование данных о состоянии пользователя версии 3 в боте версии 4
keywords: JavaScript, миграция бота, бот версии 3
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/14/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ad3f9a1cc9ce3f06bad71615e244a791025904d9
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791066"
---
<!-- This article is on hold -->

# <a name="using-javascript-v3-user-state-in-a-v4-bot"></a>Использование данных JavaScript о состоянии пользователя версии 3 в боте версии 4

В этой статье описано, как бот версии 4 может выполнять с информацией о состоянии пользователя версии 3 операции чтения, записи и удаления.

Пример кода можно найти [здесь](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot).

> [!NOTE]
> Бот поддерживает **состояние беседы** для ее отслеживания, управления ее ходом и отправки вопросов пользователю. Он поддерживает **состояние пользователя** для отслеживания ответов пользователя.

## <a name="prerequisites"></a>предварительные требования

- Средство npm версии 6.9.0 или выше (поддерживает присвоение псевдонимов пакетам).

- Node.js версии 10.14.1 или выше.

    Чтобы проверить, какая версия установлена на компьютере, выполните следующую команду в окне терминала:

    ```bash
    # determine node version
    node --version
    ```

## <a name="setup"></a>Настройка

1. Клонирование репозитория

    ```bash
    git clone https://github.com/microsoft/botbuilder-samples.git
    ```

1. В окне терминала перейдите в папку `BotBuilder-Samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot`.

    ```bash
    cd BotBuilder-Samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot
    ```

1. Запустите команду `npm install` в следующих расположениях:

    ```bash
    root
    /V4V3StorageMapper
    /V4V3UserState
    ```

1. Выполните команду ``npm run build`` или ``tsc``, чтобы скомпилировать модули `StorageMapper` и `UserState` в следующих расположениях:

    ```bash
    /V4V3StorageMapper
    /V4V3UserState
    ```

1. Настройте базу данных.

    1. Скопируйте содержимое файла `.env.example`.
    1. Создайте новый файл с именем `.env` и вставьте в него предыдущее содержимое. 
    1. Укажите значения для своих поставщиков хранилищ.
        Обратите внимание, что *имя пользователя*, *пароль* и *сведения об узле* можно найти на портале Azure в разделе для конкретного поставщика хранилища, например *Cosmos DB*, *Хранилище таблиц* или *База данных SQL*. Имена таблиц и коллекций определяются пользователем.
  
1. Укажите поставщика хранилища для бота.

    1. Откройте файл `index.js` в корне проекта. В начале файла (приблизительно строки 38–98) хранятся настройки для каждого поставщика хранилища (указаны в комментариях). Они считывают значения конфигурации из файла `.env` через `process.env` узла. В приведенном ниже фрагменте кода показано, как настроить Базу данных SQL.

        [!code-javascript[Storage configuration](~/../botbuilder-samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot/index.js?range=77-92)]

    1. Укажите поставщика хранилища, который будет использоваться ботом, передав выбранный вами экземпляр клиента хранилища адаптеру `StorageMapper` (приблизительно строка 107).  

        [!code-javascript[StorageMapper](~/../botbuilder-samples/MigrationV3V4/Node/V4V3-user-state-adapter-sample-bot/index.js?range=105-107)]

        Значение по умолчанию: *Cosmos DB*. Вы можете выбрать

        ```bash
            cosmosStorageClient
            tableStorage
            sqlStorage
        ```

1. Запустите приложение. В корне проекта выполите следующую команду:

    ```bash
    npm run start
    ```

## <a name="adapter-classes"></a>Классы адаптера

### <a name="v4v3storagemapper"></a>V4V3StorageMapper

Класс `StorageMapper` содержит основную функциональность адаптера. Он реализует интерфейс хранилища версии 4 и сопоставляет методы поставщика хранилища (чтение, запись и удаление) с классами поставщика хранилища версии 3, чтобы состояние пользователя в формате версии 3 можно было использовать в боте версии 4.

### <a name="v4v3userstate"></a>V4V3UserState

Этот класс расширяет класс `BotState` версии 4 (`botbuilder-core`), чтобы он использовал ключ в стиле версии 3. Это позволяет выполнять операции чтения, записи и удаления с хранилищем версии 3.

## <a name="testing-the-bot-using-bot-framework-emulator"></a>Тестирование бота с помощью Bot Framework Emulator

[Bot Framework Emulator][5] — это классическое приложение, которое позволяет тестировать ботов локально или удаленно (через туннель) и выполнять их отладку.

- Установите Bot Framework Emulator версии 4.3.0 или выше, скачав ее [здесь][6].

### <a name="connect-to-the-bot-using-bot-framework-emulator"></a>Подключение к боту с помощью Bot Framework Emulator

1. Запустите Bot Framework Emulator.
1. Введите следующий URL-адрес (конечная точка): `http://localhost:3978/api/messages`.

### <a name="testing-steps"></a>Этапы проверки

1. Откройте бота в эмуляторе и отправьте сообщение. При появлении запроса укажите свое имя.
1. После окончания шага отправьте боту еще одно сообщение.
1. Убедитесь, что запрос на ввод имени не отображается повторно. Бот должен считать его из хранилища и распознать, что он уже отправлял вам такой запрос.
1. Бот должен вернуть полученное от вас сообщение.
1. Перейдите к поставщику хранилища в Azure и убедитесь, что ваше имя хранится в виде пользовательских данных в базе данных.

## <a name="deploy-the-bot-to-azure"></a>Развертывание бота в Azure

Полный список инструкций для развертывания бота в Azure см. в статье [Развертывание бота][40].

## <a name="further-reading"></a>Дополнительные материалы

- [Общие сведения о службе Azure Bot][21]
- [Состояние бота][7]
- [Непосредственная запись в хранилище][8]
- [Управление состоянием беседы и пользователя][9]
- [Restify][30]
- [dotenv][31]

[3]: https://aka.ms/botframework-emulator
[5]: https://github.com/microsoft/botframework-emulator
[6]: https://github.com/Microsoft/BotFramework-Emulator/releases
[7]: https://docs.microsoft.com/azure/bot-service/bot-builder-storage-concept
[8]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-storage?tabs=javascript
[9]: https://docs.microsoft.com/azure/bot-service/bot-builder-howto-v4-state?tabs=javascript
[21]: https://docs.microsoft.com/azure/bot-service/bot-service-overview-introduction?view=azure-bot-service-4.0
[30]: https://www.npmjs.com/package/restify
[31]: https://www.npmjs.com/package/dotenv
[40]: https://aka.ms/azuredeployment

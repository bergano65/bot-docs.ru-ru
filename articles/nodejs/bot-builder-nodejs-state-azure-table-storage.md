---
title: Управление данными о состоянии пользователя с помощью Хранилища таблиц Azure (JS версии 3) — Служба Azure Bot
description: Узнайте, как сохранять и извлекать данные о состоянии в хранилище таблиц Azure с помощью пакета SDK Bot Framework для Node.js.
author: DucVo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6eb4645437210f98cf43270a50b1654faf30ef25
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790349"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-nodejs"></a>Управление данными пользовательских состояний с помощью хранилища таблиц Azure для Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

В этой статье вы реализуете Хранилище таблиц Azure для хранения данных о состоянии вашего бота и управления ими. Служба Connector State Service, используемая ботами по умолчанию, не предназначена для рабочей среды. Следует или использовать [расширения Azure](https://www.npmjs.com/package/botbuilder-azure) с веб-сайта GitHub, или реализовывать клиент состояния пользователя с помощью любой платформы хранилища данных. Ниже приведены причины, по которым следует использовать хранилище состояния пользователя:

- более высокая пропускная способность API состояния (больше контроля над производительностью);
- уменьшенная задержка для географического распределения;
- управление местом хранения данных (например, западная часть США или восточная часть США);
- доступ к данным о фактическом состоянии;
- базы данных с данными о состоянии не используются совместно с другими ботами;
- возможность хранения более 32 КБ.

## <a name="prerequisites"></a>предварительные требования

- [Node.js](https://nodejs.org/en/).
- [Эмулятор Bot Framework](~/bot-service-debug-emulator.md).
- Необходимо иметь бот на Node.js. Если у вас его нет, см. раздел [Create a bot with the Bot Builder SDK for Node.js](bot-builder-nodejs-quickstart.md) (Создание бота с помощью пакета SDK Bot Builder для Node.js). 
- [Обозреватель хранилища](http://storageexplorer.com/).

## <a name="create-azure-account"></a>Создание учетной записи Azure
Если у вас нет учетной записи Azure, нажмите [здесь](https://azure.microsoft.com/free/), чтобы зарегистрировать бесплатную учетную запись.

## <a name="set-up-the-azure-table-storage-service"></a>Настройка службы хранилища таблиц Azure
1. После входа на портал Azure нажмите **Создать**, чтобы создать новую службу хранилища таблиц Azure. 
2. Найдите **учетную запись хранения**, реализующую таблицу Azure. Нажмите кнопку **Создать**, чтобы начать создание учетной записи хранения. 
3. Заполните поля и нажмите кнопку **Создать** в нижней части экрана, чтобы развернуть новую службу хранилища. 
4. После развертывания новой службы хранилища перейдите в учетную запись хранения, которую вы только что создали. Ее можно найти в колонке **Учетные записи хранения**.
4. Выберите **Ключи доступа**и скопируйте ключ для последующего использования. Ваш бот будет использовать **Имя учетной записи хранения** и **Ключ** для вызова службы хранилища и сохранения данных состояния.

## <a name="install-botbuilder-azure-module"></a>Установка модуля botbuilder-azure

Чтобы установить модуль `botbuilder-azure` из командной строки, перейдите в каталог бота и выполните следующую команду npm.

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>Изменение кода бота

Чтобы использовать хранилище **таблиц Azure**, добавьте следующие строки кода в файл **app.js** вашего бота.

1. Требуется только что установленный модуль.

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. Настройте параметры подключения для подключения к Azure.
   ```javascript
   // Table storage
   var tableName = "Table-Name"; // You define
   var storageName = "Table-Storage-Name"; // Obtain from Azure Portal
   var storageKey = "Azure-Table-Key"; // Obtain from Azure Portal
   ```
   Значения `storageName` и `storageKay` можно найти в меню **Ключи доступа** в таблице Azure. Если значение `tableName` не существует в таблице Azure, оно будет создано автоматически.

3. С помощью модуля `botbuilder-azure` создайте два новых объекта для подключения к таблице Azure. Во-первых, создайте экземпляр `AzureTableClient`, который передается в параметры конфигурации подключения. Затем создайте экземпляр `AzureBotStorage`, который передается в объект `AzureTableClient`. Пример:

   ```javascript
   var azureTableClient = new azure.AzureTableClient(tableName, storageName, storageKey);

   var tableStorage = new azure.AzureBotStorage({gzipData: false}, azureTableClient);
   ```

4. Укажите, что вы хотите использовать пользовательскую базу данных вместо хранения в памяти и добавьте сведения о сеансе в базу данных. Пример:

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...

        // capture session user information
        session.userData = {"userId": session.message.user.id, "jobTitle": "Senior Developer"};

        // capture conversation information  
        session.conversationData[timestamp.toISOString().replace(/:/g,"-")] = session.message.text;

        // save data
        session.save();
   })
   .set('storage', tableStorage);
   ```
Теперь все готово к тестированию бота в эмуляторе.

## <a name="run-your-bot-app"></a>Запуск приложения бота

Из командной строки перейдите к каталогу бота и запустите бот, выполнив следующую команду.

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>Подключение бота к эмулятору

На этом этапе бот выполняется локально. Запустите эмулятор, а затем подключитесь к боту из эмулятора.

1. Введите <strong>http://localhost:port-number/api/messages</strong> в адресную строку эмулятора. Номер порта должен соответствовать номеру, указанному в браузере, где выполняется приложение. Поля <strong>Идентификатор приложения Microsoft</strong> и <strong>Пароль приложения Microsoft</strong> пока можно не заполнять. Вы получите эти сведения позже, когда выполните [регистрацию бота](~/bot-service-quickstart-registration.md).
2. Нажмите кнопку **Соединить**.
3. Протестируйте бот, отправив ему сообщение. Общайтесь с ботом обычным образом. Когда закончите, перейдите к **Обозревателю службы хранилища** и просмотрите сохраненные данные о состоянии.

## <a name="view-data-in-storage-explorer"></a>Просмотр данных в обозревателе хранилища

Чтобы просмотреть данные о состоянии, откройте **обозреватель хранилища** и подключитесь к Azure с помощью ваших учетных данных портала Azure или подключитесь напрямую к таблице с помощью `storageName` и `storageKey` и перейдите к `tableName`. 

![Снимок экрана — обозреватель хранилища со строками данных бота](~/media/bot-builder-nodejs-state-azure-table-storage/bot-builder-nodejs-state-azure-table-storage-query.png)

Одна запись диалога в столбце **данных** выглядит следующим образом:

```JSON
{
    "2018-05-15T18-23-48.780Z": "I'm the second user",
    "2018-05-15T18-23-55.120Z": "Do you know what time it is?",
    "2018-05-15T18-24-12.214Z": "I'm looking for information about the new process."
}
```

## <a name="next-step"></a>Следующий шаг

Теперь, когда имеется полный контроль над данными о состоянии бота, узнайте, как можно использовать их для более эффективного управления потоком общения.

> [!div class="nextstepaction"]
> [Управление потоком общения с помощью диалогов](bot-builder-nodejs-dialog-manage-conversation-flow.md)

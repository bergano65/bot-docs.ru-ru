---
title: Управление данными о состоянии пользователя с помощью Azure Cosmos DB | Документация Майкрософт
description: Узнайте, как сохранять и извлекать данные о состоянии в Azure Cosmos DB с помощью пакета SDK Bot Framework для Node.js.
author: DucVo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: dcf4b4cbeb9e0827f0baa4d1a799355bd422a9de
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299692"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-nodejs"></a>Управление данными о состоянии пользователя с помощью Azure Cosmos DB для Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

В этой статье вы реализуете хранилище Cosmos DB для хранения данных о состоянии бота и управления ими. Служба Connector State Service, используемая ботами по умолчанию, не предназначена для рабочей среды. Следует или использовать [расширения Azure](https://www.npmjs.com/package/botbuilder-azure) с веб-сайта GitHub, или реализовывать клиент состояния пользователя с помощью любой платформы хранилища данных. Ниже приведены причины, по которым следует использовать хранилище состояния пользователя:

- более высокая пропускная способность API состояния (больше контроля над производительностью);
- уменьшенная задержка для географического распределения;
- управление местом хранения данных (например, западная часть США или восточная часть США);
- доступ к данным о фактическом состоянии;
- базы данных с данными о состоянии не используются совместно с другими ботами;
- возможность хранения более 32 КБ.

## <a name="prerequisites"></a>Предварительные требования

- [Node.js](https://nodejs.org/en/).
- [Bot Framework Emulator](~/bot-service-debug-emulator.md).
- Необходимо иметь бот на Node.js. Если у вас его нет, см. раздел [Create a bot with the Bot Builder SDK for Node.js](bot-builder-nodejs-quickstart.md) (Создание бота с помощью пакета SDK Bot Builder для Node.js). 

## <a name="create-azure-account"></a>Создание учетной записи Azure
Если у вас нет учетной записи Azure, нажмите [здесь](https://azure.microsoft.com/free/), чтобы зарегистрировать бесплатную учетную запись.

## <a name="set-up-the-azure-cosmos-db-database"></a>Настройка базы данных Azure Cosmos DB
1. После входа на портал Azure щелкните **Создать**, чтобы создать базу данных *Azure Cosmos DB*. 
2. Щелкните **Базы данных**. 
3. Найдите **Azure Cosmos DB** и щелкните **Создать**.
4. Заполните поля. Для поля **API** выберите значение **SQL (DocumentDB)** . Заполните все поля и нажмите кнопку **Создать** в нижней части экрана, чтобы развернуть новую базу данных. 
5. После развертывания новой базы данных перейдите к ней. Щелкните **Ключи доступа**, чтобы найти ключи и строки подключения. Ваш бот будет использовать эту информацию для вызова службы хранилища и сохранения данных о состоянии.

## <a name="install-botbuilder-azure-module"></a>Установка модуля botbuilder-azure

Чтобы установить модуль `botbuilder-azure` из командной строки, перейдите в каталог бота и выполните следующую команду npm.

```nodejs
npm install --save botbuilder-azure
```

## <a name="modify-your-bot-code"></a>Изменение кода бота

Чтобы использовать базу данных **Azure Cosmos DB**, добавьте следующие строки кода в файл **app.js** бота.

1. Требуется только что установленный модуль.

   ```javascript
   var azure = require('botbuilder-azure'); 
   ```

2. Настройте параметры подключения для подключения к Azure.
   ```javascript
   var documentDbOptions = {
       host: 'Your-Azure-DocumentDB-URI', 
       masterKey: 'Your-Azure-DocumentDB-Key', 
       database: 'botdocs',   
       collection: 'botdata'
   };
   ```
   Значения `host` и `masterKey` можно найти в меню **Ключи** базы данных. Если записи `database` и `collection` в базе данных Azure не существуют, они будут созданы автоматически.

3. С помощью модуля `botbuilder-azure` создайте два новых объекта для подключения к базе данных Azure. Во-первых, создайте экземпляр `DocumentDBClient`, который передается в параметры конфигурации подключения (определяется как `documentDbOptions` сверху). Затем создайте экземпляр `AzureBotStorage`, который передается в объект `DocumentDBClient`. Например:
   ```javascript
   var docDbClient = new azure.DocumentDbClient(documentDbOptions);

   var cosmosStorage = new azure.AzureBotStorage({ gzipData: false }, docDbClient);
   ```

4. Укажите, что вы хотите использовать пользовательскую базу данных вместо хранилища в памяти. Например:

   ```javascript
   var bot = new builder.UniversalBot(connector, function (session) {
        // ... Bot code ...
   })
   .set('storage', cosmosStorage);
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
2. Щелкните **Подключить**.
3. Протестируйте бот, отправив ему сообщение. Общайтесь с ботом обычным образом. Когда закончите, перейдите к **Обозревателю службы хранилища** и просмотрите сохраненные данные о состоянии.

## <a name="view-state-data-on-azure-portal"></a>Просмотр данных о состоянии на портале Azure

Чтобы просмотреть данные о состоянии, войдите на портал Azure и перейдите к базе данных. Щелкните **Обозреватель данных (предварительная версия)** , чтобы убедиться, что сохраняются сведения о состоянии от бота.

## <a name="next-step"></a>Дальнейшие действия

Теперь, когда имеется полный контроль над данными о состоянии бота, узнайте, как можно использовать их для более эффективного управления потоком общения.

> [!div class="nextstepaction"]
> [Управление потоком общения с помощью диалогов](bot-builder-nodejs-dialog-manage-conversation-flow.md)

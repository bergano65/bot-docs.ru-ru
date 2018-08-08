---
title: Управление данными пользовательских состояний с помощью хранилища таблиц Azure | Документы Майкрософт
description: Узнайте, как сохранять и извлекать данные о состоянии с помощью хранилища таблиц Azure с пакетом SDK для построителя ботов для Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: c77b07801b8eb0168ac3e09d7b271ddfb17a04ac
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305302"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-nodejs"></a>Управление данными пользовательских состояний с помощью хранилища таблиц Azure для Node.js

В этой статье вы реализуете хранилище таблиц Azure для хранения данных о состоянии вашего бота и управления ими. Служба состояния соединителя, используемая ботами по умолчанию, не предназначена для рабочей среды. Следует использовать [расширения Azure](https://www.npmjs.com/package/botbuilder-azure) с сайта GitHub или реализовать пользовательское состояние клиента с помощью любой платформы хранения данных. Ниже приведены причины, по которым следует использовать хранилище пользовательских состояний:

- более высокая пропускная способность API состояния (больше контроля над производительностью);
- меньше задержка для географического распределения;
- управление местом хранения данных (например, западная часть США или восточная часть США);
- доступ к данным о фактическом состоянии;
- базы данных с данными о состоянии не используются совместно с другими ботами;
- возможность хранения более 32 КБ.

## <a name="prerequisites"></a>Предварительные требования

- [Node.js](https://nodejs.org/en/).
- [Эмулятор Bot Framework](~/bot-service-debug-emulator.md).
- Необходимо иметь бот на Node.js. Если у вас его нет, [создайте его](bot-builder-nodejs-quickstart.md). 
- [Обозреватель хранилища](http://storageexplorer.com/).

## <a name="create-azure-account"></a>Создание учетной записи Azure
Если у вас нет учетной записи Azure, нажмите [здесь](https://azure.microsoft.com/en-us/free/), чтобы зарегистрировать бесплатную учетную запись.

## <a name="set-up-the-azure-table-storage-service"></a>Настройка службы хранилища таблиц Azure
1. После входа на портал Azure нажмите **Создать**, чтобы создать новую службу хранилища таблиц Azure. 
2. Найдите **учетную запись хранения**, реализующую таблицу Azure. Нажмите кнопку **Создать**, чтобы начать создание учетной записи хранения. 
3. Заполните поля и нажмите кнопку **Создать** в нижней части экрана, чтобы развернуть новую службу хранилища. 
4. После развертывания новой службы хранилища перейдите в учетную запись хранения, которую вы только что создали. Ее можно найти в колонке **Учетные записи хранения**.
4. Выберите **Ключи доступа**и скопируйте ключ для последующего использования. Ваш бот будет использовать **Имя учетной записи хранения** и **Ключ** для вызова службы хранилища и сохранения данных состояния.

## <a name="install-botbuilder-azure-module"></a>Установка модуля botbuilder-azure

Чтобы установить модуль `botbuilder-azure` из командной строки, перейдите в каталог бота и запустите следующую команду npm:

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

3. С помощью модуля `botbuilder-azure` создайте два новых объекта для подключения к таблице Azure. Во-первых, создайте экземпляр `AzureTableClient`, который передается в параметры конфигурации подключения. Затем создайте экземпляр `AzureBotStorage`, который передается в объект `AzureTableClient`. Например: 

   ```javascript
   var azureTableClient = new azure.AzureTableClient(tableName, storageName, storageKey);

   var tableStorage = new azure.AzureBotStorage({gzipData: false}, azureTableClient);
   ```

4. Укажите, что вы хотите использовать пользовательскую базу данных вместо хранения в памяти и добавьте сведения о сеансе в базу данных. Например: 

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

Из командной строки перейдите к каталогу бота и запустите бот, выполнив следующую команду:

```nodejs
node app.js
```

## <a name="connect-your-bot-to-the-emulator"></a>Подключение бота к эмулятору

На этом этапе бот выполняется локально. Запустите эмулятор, а затем подключитесь к боту из эмулятора:

1. Введите <strong>http://localhost:port-number/api/messages</strong> в адресную строку эмулятора. Номер порта должен соответствовать номеру, указанному в браузере, где выполняется приложение. Поля <strong>Идентификатор приложения Microsoft</strong> и <strong>Пароль приложения Microsoft</strong> пока можно не заполнять. Вы получите эти сведения позднее, когда [зарегистрируете бот](~/bot-service-quickstart-registration.md).
2. Щелкните **Подключить**.
3. Протестируйте бот, отправив ему сообщение. Общайтесь с ботом обычным образом. Когда закончите, перейдите к **обозревателю хранилища** и просмотрите сохраненные данные о состоянии.

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

## <a name="next-step"></a>Дальнейшие действия

Теперь, когда у вас есть полный контроль над данными о состоянии бота, давайте узнаем, как можно использовать их для более эффективного управления диалогом.

> [!div class="nextstepaction"]
> [Управление потоком беседы](bot-builder-nodejs-dialog-manage-conversation-flow.md)
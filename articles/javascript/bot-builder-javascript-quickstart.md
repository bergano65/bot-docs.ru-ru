---
title: Создание бота с помощью пакета SDK Bot Builder для JavaScript | Документация Майкрософт
description: Быстрое создание бота с помощью пакета SDK Bot Builder для JavaScript.
keywords: Краткое руководство, пакет SDK Bot Builder, приступая к работе
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 07/12/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 337b6a0b8739b5e5de4d2d1b2b87dcad55f2c854
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352933"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-v4-preview-for-javascript"></a>Создание бота с помощью пакета SDK Bot Builder версии 4 (предварительная версия) для JavaScript
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

В этом кратком руководстве описывается создание бота с помощью генератора Yeoman Bot Builder и пакета SDK Bot Builder для JavaScript, а также его тестирование с помощью Bot Framework Emulator. Это руководство основано на [пакете SDK для Microsoft Bot Builder версии 4](https://github.com/Microsoft/botbuilder-js).

## <a name="pre-requisites"></a>Предварительные требования
- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/en/)
- Компонент [Yeoman](http://yeoman.io/), который позволяет использовать генератор для автоматического создания бота.
- [Bot Emulator](https://github.com/Microsoft/BotFramework-Emulator).
- Навыки разработки для [Restify](http://restify.com/) и асинхронного программирования в Java.

> [!NOTE]
> Для некоторых конфигураций при установке Restify возникает ошибка, связанная с node-gyp.
> В этом случае попробуйте выполнить команду `npm install -g windows-build-tools`.


Пакет SDK Bot Builder для JavaScript состоит из ряда [пакетов](https://github.com/Microsoft/botbuilder-js/tree/master/libraries), которые можно установить из NPM с помощью специального тега `@preview`.

# <a name="create-a-bot"></a>Создание бота

Откройте командную строку с повышенными привилегиями, создайте каталог и инициализируйте пакет для бота.

```bash
md myJsBots
cd myJsBots
```

Затем установите Yeoman и генератор для JavaScript.

```bash
npm install -g yo
npm install -g generator-botbuilder@preview
```

Используйте генератор для создания бота Echo.

```bash
yo botbuilder
```

Yeoman запросит некоторые сведения для создания бота.
-   Введите имя бота.
-   Введите описание.
-   Выберите язык для бота, `JavaScript` или `TypeScript`.
-   Выберите шаблон для использования. В настоящее время доступен только шаблон `Echo`, но в скором времени будут добавлены и другие.

Yeoman создаст бот в новой папке.

## <a name="explore-code"></a>Изучение кода

Открыв только что созданную папку бота, вы увидите файл `app.js`. Этот файл `app.js` будет содержать весь код, необходимый для запуска приложения бота. Данный файл содержит бот Echo, который повторяет все, что вы вводите, и увеличивает значение счетчика. 

В приведенном ниже коде ПО промежуточного слоя состояния общения использует хранилище в памяти. Оно считывает и записывает объект state в хранилище. В переменную count записывается число сообщений, отправленных боту. Аналогичный прием можно использовать для поддержания состояния между шагами. 

**app.js**
```javascript
// Packages are installed for you
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);
```

Следующий код ожидает передачи входящего запроса и проверяет тип входящего действия перед отправкой ответа пользователю.

```javascript
// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        // This bot is only handling Messages
        if (context.activity.type === 'message') {
        
            // Get the conversation state
            const state = conversationState.get(context);
            
            // If state.count is undefined set it to 0, otherwise increment it by 1
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            
            // Echo back to the user whatever they typed.
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            // Echo back the type of activity the bot detected if not of type message
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

## <a name="start-your-bot"></a>Запуск бота

Перейдите в каталог, созданный для бота, и запустите его.

```bash
cd <bot directory>
node app.js
```

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота
На этом этапе бот выполняется локально. После этого запустите эмулятор и подключитесь к боту в эмуляторе.
1. Щелкните ссылку **Create a new bot configuration** (Создать конфигурацию бота) на вкладке "Welcome" (Приветствие) эмулятора. 

2. Введите **имя бота** и укажите путь к каталогу, в котором расположен код бота. Файл конфигурации бота будет сохранен по этому пути.

3. Введите `http://localhost:port-number/api/messages` в поле **Endpoint URL** (URL-адрес конечной точки), где *port-number* соответствует номеру порта в браузере, в котором запущено приложение.

4. Щелкните **Connect** (Подключиться) для подключения к боту. Не нужно указывать **идентификатор приложения Майкрософт** и **пароль приложения Майкрософт**. Эти поля пока можно оставить пустыми. Вы получите эти сведения позднее, когда зарегистрируете бот.

Отправьте сообщение "Hi" своему боту и, бот ответит на это сообщение: "0: You said "Hi"".

## <a name="next-steps"></a>Дополнительная информация

Теперь перейдите к основным понятиям, описывающим боты и принципы их работы.

> [!div class="nextstepaction"]
> [Базовые понятия о ботах](../v4sdk/bot-builder-basics.md)

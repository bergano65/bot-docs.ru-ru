---
title: Создание бота с помощью пакета SDK Bot Builder для Node.js | Документация Майкрософт
description: Создание бота с помощью пакета SDK Bot Builder для Node.js — мощной платформы для создания ботов.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6159997ec5ea3dbd3188ba2ea4b6207b5d9db08f
ms.sourcegitcommit: 97bb24f15041caccef4ca5736aa14f144881e0c6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/06/2018
ms.locfileid: "39567553"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-nodejs"></a>Создание бота с помощью пакета SDK Bot Builder для Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-quickstart.md)
> - [Node.js](../nodejs/bot-builder-nodejs-quickstart.md)
> - [Служба Azure Bot](../bot-service-quickstart.md)
> - [REST](../rest-api/bot-framework-rest-connector-quickstart.md)

Пакет SDK Bot Builder для Node.js — это платформа для разработки ботов. С помощью таких платформ, как Express и Restify, поддерживающих моделирование, легко предоставить разработчикам для JavaScript привычную среду для создания ботов.

В этом руководстве рассматривается создание бота с помощью пакета SDK Bot Builder для Node.js. Этот бот можно протестировать в окне консоли и с помощью Bot Framework Emulator.

## <a name="prerequisites"></a>Предварительные требования
Приступите к работе, выполнив следующие предварительные задачи.

1. Установите [Node.js](https://nodejs.org).
2. Создайте папку для своего бота.
3. Из командной строки или терминала перейдите в эту папку.
4. Выполните следующую команду **npm**.

```nodejs
npm init
```

Следуйте указаниям на экране, чтобы ввести сведения о боте, и npm создаст файл **package.json**, содержащий сведения, которые вы указали. 

## <a name="install-the-sdk"></a>Установка пакета SDK
Затем установите пакет SDK Bot Builder для Node.js, выполнив следующую команду **npm**.

```nodejs
npm install --save botbuilder
```

После установки пакета SDK все будет готово к созданию первого бота.

Ваш первый бот будет просто повторять все, что вводит пользователь. Выполните следующие действия, чтобы создать бот.

1. В папке, созданной ранее для бота, создайте файл **app.js**.
2. Откройте файл **app.js** в текстовом редакторе или интегрированной среде разработки на свой выбор. Добавьте в него указанный ниже код. 

   [!code-javascript[consolebot code sample Node.js](../includes/code/node-getstarted.js#consolebot)]

3. Сохраните файл. Теперь все готово для запуска и тестирования бота.

### <a name="start-your-bot"></a>Запуск бота

Перейдите в каталог бота в окне консоли и запустите бот.

```nodejs
node app.js
```

Теперь бот работает в локальной среде. Протестируйте бот, введя несколько сообщений в окне консоли.
Вы увидите, что бот отвечает на каждое сообщение, повторяя их и добавляя перед каждым сообщением текст *"You said:"*.

## <a name="install-restify"></a>Установка Restify

Консольные боты являются хорошими текстовыми клиентами, но чтобы использовать любой из каналов Bot Framework (или запустить бот в эмуляторе), ваш бот должен работать на конечной точке API. Установите <a href="http://restify.com/" target="_blank">Restify</a>, выполнив следующую команду **npm**.

```nodejs
npm install --save restify
```

Установив Restify, вы сможете внести некоторые изменения в бот.

## <a name="edit-your-bot"></a>Изменение бота

Необходимо будет внести некоторые изменения в файл **app.js**. 

1. Добавьте строку, чтобы требовать использование модуля `restify`.
2. Измените `ConsoleConnector` на `ChatConnector`.
3. Добавьте идентификатор и пароль приложения Майкрософт.
4. Настройте соединитель для ожидания передачи данных на конечной точке API.

   [!code-javascript[echobot code sample Node.js](../includes/code/node-getstarted.js#echobot)]

5. Сохраните файл. Теперь все готово для запуска и тестирования бота в эмуляторе.

> [!NOTE] 
> Для запуска бота в Bot Framework Emulator нет нужен **идентификатор приложения Майкрософт** или **пароль приложения Майкрософт**.

## <a name="test-your-bot"></a>Тестирование бота
Проверьте работу бота с помощью [Bot Framework Emulator](../bot-service-debug-emulator.md). Этот эмулятор — классическое приложение, которое позволяет выполнять тестирование и отладку бота на локальном компьютере или удаленно через туннель.

Для начала необходимо [скачать](https://emulator.botframework.com) и установить эмулятор. После завершения скачивания запустите исполняемый файл и выполните установку.

### <a name="start-your-bot"></a>Запуск бота

После установки эмулятора перейдите в каталог бота в окне консоли и запустите бот.

```nodejs
node app.js
```
   
Теперь бот работает в локальной среде.

### <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота
После запуска бота запустите эмулятор и подключитесь к боту.

1. Щелкните ссылку **Create a new bot configuration** (Создать конфигурацию бота) в окне эмулятора. 

2. Введите `http://localhost:port-number/api/messages` в адресной строке, где значение *port-number* должно соответствовать номеру порта, указанному в браузере, где запускается приложение.

3. Щелкните "Сохранить и подключиться". Идентификатор и пароль приложения Майкрософт указывать не нужно. Эти поля пока можно оставить пустыми. Вы получите эти сведения позднее, когда зарегистрируете бот.

### <a name="try-out-your-bot"></a>Проверка бота

Теперь, когда бот работает локально и подключен к эмулятору, проверьте бот, введя несколько сообщений в эмуляторе.
Вы увидите, что бот отвечает на каждое сообщение, повторяя их и добавляя перед каждым сообщением текст *"You said:"*.

Вы успешно создали первый бот с помощью пакета SDK Bot Builder для Node.js.

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Пакет SDK Bot Builder для Node.js](bot-builder-nodejs-overview.md)

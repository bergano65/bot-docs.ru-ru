---
title: Создание бота с помощью пакета SDK Bot Builder для JavaScript | Документация Майкрософт
description: Быстрое создание бота с помощью пакета SDK Bot Builder для JavaScript.
keywords: Краткое руководство, пакет SDK Bot Builder, приступая к работе
author: jonathanfingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/30/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1b111125ea240bf89f506106c948c6b5d3be7649
ms.sourcegitcommit: 15f7fa40b7e0a05507cdc66adf75bcfc9533e781
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/02/2018
ms.locfileid: "50916821"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-javascript"></a>Создание бота с помощью пакета SDK Bot Builder для JavaScript

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

В этом кратком руководстве описано создание одного бота с помощью генератора Yeoman Bot Builder и пакета SDK Bot Builder для JavaScript, а также его тестирование с помощью Bot Framework Emulator.

## <a name="prerequisites"></a>Предварительные требования

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- Компонент [Yeoman](http://yeoman.io/), который использует генератор для автоматического создания бота.
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator).
- навыки разработки для [Restify](http://restify.com/) и асинхронного программирования в JavaScript.

> [!NOTE]
> Для некоторых конфигураций при установке Restify возникает ошибка, связанная с node-gyp.
> В таком случае попробуйте выполнить следующую команду с дополнительными разрешениями:
> ```bash
> npm install -g windows-build-tools
> ```

## <a name="create-a-bot"></a>Создание бота

Создание бота и инициализация пакетов для него

1. Откройте терминал или командную строку с повышенными привилегиями.
1. Если у вас еще нет каталога для ботов JavaScript, создайте и откройте его. (Мы создаем общий каталог для ботов JavaScript в, хотя в этом руководстве потребуется только один бот.)

   ```bash
   md myJsBots
   cd myJsBots
   ```

1. Убедитесь, что обновление npm не требуется.

   ```bash
   npm install -g npm
   ```

1. Затем установите Yeoman и генератор для JavaScript.

   ```bash
   npm install -g yo generator-botbuilder
   ```

1. Используйте генератор для создания бота Echo.

   ```bash
   yo botbuilder
   ```

Yeoman запросит некоторые сведения для создания бота. Для задач в этом руководстве используйте значение по умолчанию.

- Введите имя бота. (myChatBot)
- Введите описание. (Демонстрация базовых возможностей Microsoft Bot Framework.)
- Выберите язык для бота. JavaScript
- Выберите шаблон для использования. (Echo)

Благодаря шаблону проект содержит весь код, необходимый для создания бота в рамках этого краткого руководства. Теперь нет необходимости писать дополнительный код.

> [!NOTE]
> Чтобы создать бот `Basic`, вам потребуется языковая модель LUIS. Вы можете создать ее, перейдя по адресу [luis.ai](https://www.luis.ai). После создания модели обновите файл .bot. Файл бота должен быть примерно [таким](../v4sdk/bot-builder-service-file.md).

## <a name="start-your-bot"></a>Запуск бота

В терминале или командной строке перейдите к каталогу, созданному для бота, и запустите бот с помощью `npm start`. На этом этапе бот выполняется локально.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение к боту

1. Установите Bot Framework Emulator.
2. Щелкните ссылку **Open Bot** (Открыть бот) на вкладке приветствия в эмуляторе.
3. Выберите файл с расширением .bot, расположенный в каталоге созданного проекта.

Отправьте сообщение боту и получите от него сообщение в ответ.
![Работающий эмулятор](../media/emulator-v4/js-quickstart.png)

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Принципы работы бота](../v4sdk/bot-builder-basics.md)

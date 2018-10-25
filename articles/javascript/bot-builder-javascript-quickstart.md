---
title: Создание бота с помощью пакета SDK Bot Builder для JavaScript | Документация Майкрософт
description: Быстрое создание бота с помощью пакета SDK Bot Builder для JavaScript.
keywords: Краткое руководство, пакет SDK Bot Builder, приступая к работе
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/15/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: aa13889cea2a26bf094a919f5d05905d65f7661f
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49998863"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-javascript"></a>Создание бота с помощью пакета SDK Bot Builder для JavaScript

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

В этом кратком руководстве описывается создание бота с помощью генератора Yeoman Bot Builder и пакета SDK Bot Builder для JavaScript, а также его тестирование с помощью Bot Framework Emulator. 

## <a name="prerequisites"></a>Предварительные требования

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- Компонент [Yeoman](http://yeoman.io/), который позволяет использовать генератор для автоматического создания бота.
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator).
- навыки разработки для [Restify](http://restify.com/) и асинхронного программирования в JavaScript.

> [!NOTE]
> Для некоторых конфигураций при установке Restify возникает ошибка, связанная с node-gyp.
> В этом случае попробуйте выполнить команду `npm install -g windows-build-tools`.

## <a name="create-a-bot"></a>Создание бота

Откройте командную строку с повышенными привилегиями, создайте каталог и инициализируйте пакет для бота.

```bash
md myJsBots
cd myJsBots
```

Убедитесь, что обновление npm не требуется.
```bash
npm i npm
```

Затем установите Yeoman и генератор для JavaScript.

```bash
npm install -g yo
npm install -g generator-botbuilder
```

Используйте генератор для создания бота Echo.

```bash
yo botbuilder
```

Yeoman запросит некоторые сведения для создания бота.

- Введите имя бота.
- Введите описание.
- Выберите язык для бота, `JavaScript` или `TypeScript`.
- Выберите шаблон `Echo`.

Благодаря шаблону проект содержит весь код, необходимый для создания бота в рамках этого краткого руководства. Теперь нет необходимости писать дополнительный код.

> [!NOTE]
> Чтобы создать базовый бот, вам потребуется языковая модель LUIS. Вы можете создать ее, перейдя по адресу [luis.ai](https://www.luis.ai). После создания модели обновите файл .bot. Файл бота должен быть примерно [таким](../v4sdk/bot-builder-service-file.md). 

## <a name="start-your-bot"></a>Запуск бота

В Powershell или Bash перейдите в каталог, созданный для бота, и запустите бота с помощью `npm start`. На этом этапе бот выполняется локально.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота
1. Запустите эмулятор.
2. Щелкните ссылку **Open Bot** (Открыть бот) на вкладке приветствия в эмуляторе.
3. Выберите файл с расширением .bot, расположенный в каталоге созданного проекта.

Отправьте сообщение боту и получите от него сообщение в ответ.
![Работающий эмулятор](../media/emulator-v4/emulator-running.png)

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Принципы работы бота](../v4sdk/bot-builder-basics.md) 

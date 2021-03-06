---
ms.openlocfilehash: 77635aab0e1535a2d44ef24c3e33094b13f4a15a
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117829"
---
## <a name="prerequisites"></a>Предварительные требования

- [Visual Studio Code](https://www.visualstudio.com/downloads)
- [Node.js](https://nodejs.org/)
- Компонент [Yeoman](http://yeoman.io/), который использует генератор для автоматического создания бота.
- [git](https://git-scm.com/)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).
- навыки разработки для [Restify](http://restify.com/) и асинхронного программирования в JavaScript.

> [!NOTE]
> Установка Microsoft Build Tools, указанная ниже, требуется только в том случае, если Windows используется как операционная система для разработки.
> Для некоторых конфигураций при установке Restify возникает ошибка, связанная с node-gyp.
> В таком случае попробуйте выполнить следующую команду с дополнительными разрешениями.
> Этот вызов также может зависнуть без завершения, если на компьютере уже установлена среда Python.

> ```bash
> # only run this command if you are on Windows. Read the above note. 
> npm install -g windows-build-tools
> ```

## <a name="create-a-bot"></a>Создание бота

Создание бота и инициализация пакетов для него

1. Откройте терминал или командную строку с повышенными привилегиями.
1. Если у вас еще нет каталога для ботов JavaScript, создайте и откройте его. (Мы создаем общий каталог для ботов JavaScript в, хотя в этом руководстве потребуется только один бот.)

   ```bash
   mkdir myJsBots
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

```text
? What's the name of your bot? my-chat-bot
? What will your bot do? Demonstrate the core capabilities of the Microsoft Bot Framework
? What programming language do you want to use? JavaScript
? Which template would you like to start with? Echo Bot - https://aka.ms/bot-template-echo
? Looking good.  Shall I go ahead and create your new bot? (Y/n)
```

Благодаря шаблону проект содержит весь код, необходимый для создания бота в рамках этого краткого руководства. Теперь нет необходимости писать дополнительный код.

> [!NOTE]
> Чтобы создать бот `Core`, вам потребуется языковая модель LUIS. Вы можете создать ее, перейдя по адресу [luis.ai](https://www.luis.ai). Создав модель, обновите файл конфигурации.

## <a name="start-your-bot"></a>Запуск бота

В терминале или командной строке перейдите к каталогу, созданному для бота, и запустите бот с помощью `npm start`. На этом этапе бот выполняется локально.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение к боту

1. Установите Bot Framework Emulator.
2. Щелкните ссылку **Create a new bot configuration** (Создать конфигурацию бота) на вкладке Welcome (Приветствие) эмулятора. 
3. Заполните поля для своего бота. Используйте адрес страницы приветствия своего бота (обычно http://localhost:3978) и добавьте к нему сведения маршрутизации "/api/messages".
4. Щелкните **Сохранить и подключиться**.

Отправьте сообщение боту и получите от него сообщение в ответ.
![Работающий эмулятор](../media/emulator-v4/js-quickstart.png)

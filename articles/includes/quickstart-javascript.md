---
ms.openlocfilehash: 04b015963b8ea991b87f085dd5d6aa0110c50a18
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360814"
---
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
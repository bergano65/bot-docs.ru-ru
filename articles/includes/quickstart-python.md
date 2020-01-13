---
ms.openlocfilehash: e2a43cf5880da0036415d80d19e59db4c84d2f73
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491748"
---
## <a name="prerequisites"></a>предварительные требования
- Python [3.6](https://www.python.org/downloads/release/python-369/) или [3.7](https://www.python.org/downloads/release/python-375/)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).
- [git](https://git-scm.com/)
- Знание асинхронного программирования на Python

## <a name="create-a-bot"></a>Создание бота
1. Откройте окно терминала и перейдите в папку, в которой вы сохраняете бот локально. Установите требуемые пакеты, выполнив следующие команды:
- `pip install botbuilder-core`
- `pip install asyncio`
- `pip install -r requirements.txt`
- `pip install cookiecutter`

Последний пакет (cookiecutter) будет использоваться для создания бота. Убедитесь, что пакет cookiecutter установлен правильно, выполнив команду `cookiecutter --help`.

2. Чтобы создать бот, сделайте следующее:

```cmd
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

Эта команда создает бот Echo на основе [этого шаблона](https://github.com/microsoft/BotBuilder-Samples/tree/master/generators/python/app/templates/echo/%7B%7Bcookiecutter.bot_name%7D%7D) на Python.

3. После этого вам будет предложено ввести *имя* и *описание* бота. Присвойте боту имя `echo-bot` и задайте для него описание `A bot that echoes back user response.`, как показано ниже:

![указание имени и описания](~/media/python/quickstart/set-name-description.png)

Скопируйте последние четыре цифры в адресе в последней строке (обычно это 3978) для использования на следующем шаге. Теперь вы готовы запустить бот.

## <a name="start-you-bot"></a>Запуск бота
1. В окне терминала перейдите в папку `echo-bot`, в которой вы сохранили бот. Выполните команду `pip install -r requirements.txt`, чтобы установить пакеты, требуемые для запуска бота.

2. После установки пакетов выполните `python app.py` для запуска бота. Бот будет готов к тестированию, когда появится последняя строка, показанная на снимке экрана ниже:

![локальное выполнение бота](~/media/python/quickstart/bot-running-locally.png)
<!---
Alternatively, you can set the file in an environment variable with set `FLASK_APP=app.py` in Windows and `export FLASK_APP=app.py` in Mac OS/Linux and then run `flask run --host=127.0.0.1 --port=3978`.-->

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение к боту
1. Запустите эмулятор и щелкните **Open Bot** (Открыть бот).

2. После нажатия кнопки откроется окно, где можно указать значения, требуемые для запуска бота. Используйте сохраненное ранее число и задайте `http://localhost:<saved number>/api/messages` в качестве **URL-адреса бота**, как показано ниже:

![открытие экрана бота](~/media/python/quickstart/open-bot.png)

3. Нажмите кнопку **Подключить**. Бот должен запуститься. Попробуйте проверить бот, введя любую команду и нажав кнопку *ВВОД*, как показано ниже:

![подключение и тестирование](~/media/python/quickstart/connect-and-start.png)

---
title: Создание бота с помощью пакета SDK Bot Framework для Python | Документация Майкрософт
description: Быстрое создание бота с помощью пакета SDK Bot Framework для Python.
keywords: краткое руководство, пакет SDK Bot Framework, приступая к работе
author: emgrol
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/10/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3df1e44b5643f0dae98e659d76b8e30224081782
ms.sourcegitcommit: 86495b597e55c94309a0c73fc1945a3393ddcbbf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/08/2020
ms.locfileid: "75756372"
---
# <a name="create-a-bot-with-the-bot-framework-sdk-for-python"></a>Создание бота с помощью пакета SDK Bot Framework для Python

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

В этом кратком руководстве описывается, как с помощью шаблона Echo Bot на Python создать бот и протестировать его, используя Bot Framework Emulator.

## <a name="prerequisites"></a>предварительные требования
- Python [3.6](https://www.python.org/downloads/release/python-369/) или [3.7](https://www.python.org/downloads/release/python-375/)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).
- [git](https://git-scm.com/)
- Навыки асинхронного программирования на Python

## <a name="create-a-bot"></a>Создание бота
1. Откройте окно терминала и перейдите в папку, в которой вы бот хранится локально. Установите требуемые пакеты, выполнив следующие команды:
- `pip install botbuilder-core`
- `pip install asyncio`
- `pip install cookiecutter`

Последний пакет (cookiecutter) будет использоваться для создания бота. Убедитесь, что пакет cookiecutter установлен правильно, выполнив команду `cookiecutter --help`.

2. Чтобы создать бот, сделайте следующее:

```cmd
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

Эта команда создает бот Echo на основе [этого шаблона](https://github.com/microsoft/botbuilder-python/tree/master/generators/app/templates/echo) на Python.

3. После этого вам будет предложено ввести *имя* и *описание* бота. Присвойте боту имя `echo-bot` и задайте для него описание `A bot that echoes back user response.`, как показано ниже:

![указание имени и описания](../media/python/quickstart/set-name-description.png)

Скопируйте последние четыре цифры в адресе в последней строке (обычно это 3978) для использования на следующем шаге. Теперь вы готовы запустить бот.

## <a name="start-you-bot"></a>Запуск бота
1. В окне терминала перейдите в папку `echo-bot`, в которой вы сохранили бот. Выполните команду `pip install -r requirements.txt`, чтобы установить пакеты, требуемые для запуска бота.

2. После установки пакетов выполните `python app.py` для запуска бота. Вы поймете, что бот готов к тестированию, когда появится последняя строка, показанная на снимке экрана ниже:

![Локальное выполнение бота](../media/python/quickstart/bot-running-locally.png)

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение к боту
1. Запустите эмулятор и щелкните **Open Bot** (Открыть бот).

2. После нажатия кнопки откроется окно, где можно указать значения, требуемые для запуска бота. Используйте сохраненное ранее число и задайте `http://localhost:<saved number>/api/messages` в качестве **URL-адреса бота**, как показано ниже:

![Открытие экрана бота](../media/python/quickstart/open-bot.png)

3. Нажмите кнопку **Подключить**. Бот должен запуститься. Попробуйте проверить бот, введя любую команду и нажав кнопку *ВВОД*, как показано ниже:

![Подключение и тестирование](../media/python/quickstart/connect-and-start.png)

## <a name="additional-resources"></a>Дополнительные ресурсы
Подключение к удаленно размещенном боту описано в документации по [туннелированию (ngrok)](https://github.com/Microsoft/BotFramework-Emulator/wiki/Tunneling-(ngrok)).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Развертывание бота в Azure](../bot-builder-deploy-az-cli.md)


---
ms.openlocfilehash: 04f9101d0cf29618fb7d50e126c008190064a831
ms.sourcegitcommit: 3e3c9986b95532197e187b9cc562e6a1452cbd95
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/06/2019
ms.locfileid: "65199392"
---
## <a name="prerequisites"></a>Предварительные требования
- Visual Studio [2017 или 2017](https://www.visualstudio.com/downloads).
- Шаблон пакета SDK Bot Framework версии 4 для [C#](https://aka.ms/bot-vsix).
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).
- Опыт работы с [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) и асинхронного программирования в [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index).

## <a name="create-a-bot"></a>Создание бота
Установите шаблон BotBuilderVSIX.vsix, скачанный при выполнении предварительных требований.

В Visual Studio создайте проект бота с использованием шаблона **эхо-бота для Bot Framework версии 4**.

![Проект Visual Studio](~/media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> При необходимости укажите для проекта тип сборки ``.Net Core 2.1``. Также, если потребуется, обновите [пакеты NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio) `Microsoft.Bot.Builder`.

Благодаря шаблону проект содержит весь код, необходимый для создания бота в рамках этого краткого руководства. Теперь нет необходимости писать дополнительный код.

## <a name="start-your-bot-in-visual-studio"></a>Запуск бота в Visual Studio

Когда вы нажмете кнопку запуска, Visual Studio создаст приложение, развернет его в узле localhost и запустит веб-браузер для отображения страницы приложения `default.htm`. На этом этапе бот выполняется локально.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота

После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Create a new bot configuration** (Создать конфигурацию бота) на вкладке Welcome (Приветствие) эмулятора. 
2. Заполните поля для бота, а затем щелкните **Save and connect** (Сохранить и подключить).

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом

Отправьте сообщение боту и получите от него сообщение в ответ.

![Работающий эмулятор](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> Если вы видите, что сообщение не отправляется, вам может потребоваться перезагрузить компьютер, так как ngrok еще не получил необходимые привилегии в вашей системе (это нужно выполнить только один раз).

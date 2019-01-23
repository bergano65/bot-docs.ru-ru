---
ms.openlocfilehash: 2096aa342fe954b9f9f1d128bc080c0e0e6efdce
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360804"
---
## <a name="prerequisites"></a>Предварительные требования
- Visual Studio [2017](https://www.visualstudio.com/downloads).
- Шаблон для пакета SDK Bot Framework версии 4 для [C#](https://aka.ms/bot-vsix).
- Bot Framework Channel [Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Навыки разработки для [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) и асинхронного программирования в [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index).

## <a name="create-a-bot"></a>Создание бота
Установите шаблон BotBuilderVSIX.vsix, скачанный при выполнении предварительных требований.

В Visual Studio создайте новый проект бота с использованием шаблона бота Echo для Bot Framework версии 4.

![Проект Visual Studio](~/media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> При необходимости укажите для проекта тип сборки ``.Net Core 2.1``. Также, если потребуется, обновите [пакеты NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio) `Microsoft.Bot.Builder`.

Благодаря шаблону проект содержит весь код, необходимый для создания бота в рамках этого краткого руководства. Теперь нет необходимости писать дополнительный код.

## <a name="start-your-bot-in-visual-studio"></a>Запуск бота в Visual Studio

Когда вы нажмете кнопку запуска, Visual Studio создаст приложение, развернет его в узле localhost и запустит веб-браузер для отображения страницы приложения `default.htm`. На этом этапе бот выполняется локально.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота

После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Open Bot** (Открыть бот) на вкладке приветствия в эмуляторе. 
2. Выберите BOT-файл, расположенный в каталоге с созданным решением Visual Studio.

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом

Отправьте сообщение боту и получите от него сообщение в ответ.

![Работающий эмулятор](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> Если вы видите, что сообщение не отправляется, вам может потребоваться перезагрузить компьютер, так как ngrok еще не получил необходимые привилегии в вашей системе (это нужно выполнить только один раз).

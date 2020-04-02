---
ms.openlocfilehash: 9c86001a51914f359163e7d755aa57e1c54127f8
ms.sourcegitcommit: dcacda776c927bcc7c76d00ff3cc6b00b062bd6b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/23/2019
ms.locfileid: "74414477"
---
## <a name="prerequisites"></a>Предварительные требования
- [Visual Studio 2017 или более поздней версии](https://www.visualstudio.com/downloads)
- [Шаблон пакета SDK Bot Framework версии 4 для C#](https://aka.ms/bot-vsix)
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).
- Опыт работы с [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) и асинхронного программирования в [C#](https://docs.microsoft.com/dotnet/csharp/programming-guide/concepts/async/index).

## <a name="create-a-bot"></a>Создание бота
Установите [шаблон BotBuilderVSIX.vsix](https://aka.ms/bot-vsix), который вы скачали при выполнении предварительных требований.

В Visual Studio создайте проект бота с использованием шаблона **Эхо-бота для Bot Framework версии 4**. Введите в поле поиска _Bot Framework v4_, чтобы отображались только шаблоны ботов.

![В Visual Studio создайте новый диалог проекта.](../media/azure-bot-quickstarts/bot-builder-dotnet-project-vs2019.png)

> [!TIP] 
> Если вы используете Visual Studio 2017, проект должен иметь тип сборки ``.Net Core 2.1`` или более поздней версии. Также, если потребуется, обновите [пакеты NuGet](https://docs.microsoft.com/nuget/quickstart/install-and-use-a-package-in-visual-studio) `Microsoft.Bot.Builder`.

Благодаря шаблону проект содержит весь код, необходимый для создания бота в рамках этого краткого руководства. Теперь нет необходимости писать дополнительный код.

## <a name="start-your-bot-in-visual-studio"></a>Запуск бота в Visual Studio

Когда вы нажмете кнопку запуска, Visual Studio создаст приложение, развернет его в узле localhost и запустит веб-браузер для отображения страницы приложения `default.htm`. На этом этапе бот выполняется локально.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота

После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Создать конфигурацию бота** на вкладке **Приветствие** эмулятора. 
2. Заполните поля для своего бота. Используйте адрес страницы приветствия своего бота (обычно http://localhost:3978) и добавьте к нему сведения маршрутизации "/api/messages".
3. Щелкните **Сохранить и подключиться**.

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом

Отправьте сообщение боту и получите от него сообщение в ответ.

![Работающий эмулятор](~/media/emulator-v4/emulator-running.png)

> [!NOTE]
> Если вы видите, что сообщение не отправляется, вам может потребоваться перезагрузить компьютер, так как ngrok еще не получил необходимые привилегии в вашей системе (это нужно выполнить только один раз).

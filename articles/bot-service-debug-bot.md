---
title: Отладка бота | Документация Майкрософт
description: Отладка ботов, построенных с помощью службы Azure Bot.
author: v-ducvo
ms.author: v-ducvo
keywords: Пакет SDK Bot Framework, отладка бота, тестирование бота, эмулятор бота, эмулятор
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/13/2018
ms.openlocfilehash: b445ce7796c3f7f3180b15fd6dfac1ef82b808ae
ms.sourcegitcommit: d385ec5fe61c469ab17e6f21b4a0d50e5110d0fd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/15/2019
ms.locfileid: "54298301"
---
# <a name="debug-a-bot"></a>Отладка бота

В этой статье описывается отладка ботов с помощью интегрированной среды разработки (IDE), такой как Visual Studio или Visual Studio Code, и Bot Framework Emulator. Эти методы можно использовать для локальной отладки любого бота. Но для работы с этой статьей используются боты [C#](~/dotnet/bot-builder-dotnet-sdk-quickstart.md) и [JS](~/javascript/bot-builder-javascript-quickstart.md), созданные в рамках краткого руководства.

## <a name="prerequisites"></a>Предварительные требования 
- Скачайте и установите [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Скачайте и установите [Visual Studio Code](https://code.visualstudio.com) или [Visual Studio](https://www.visualstudio.com/downloads) (Community Edition или более поздней версии).

### <a name="debug-a-javascript-bot-using-command-line-and-emulator"></a>Отладка ботов JavaScript с помощью командной строки и эмулятора

Для запуска бота JavaScript при помощи командной строки и тестирования ботов эмулятором, выполните следующие действия.
1. В командной строке измените каталог на каталог проекта ботов.
1. Запустите бота с помощью команды **node app.js**.
1. Запустите эмулятор и подключитесь к конечной точке бота (например, **http://localhost:3978/api/messages**). Если бот запускается впервые, щелкните **Файл > Новая программа-робот** и следуйте инструкциям на экране. В противном случае, чтобы открыть существующий бот, щелкните **Файл > Открыть программу-робота**. Поскольку этот бот работает локально на компьютере, вы можете оставить строки **ИД приложения MSA** и **Пароль приложения MSA** пустыми. Дополнительные сведения см. в статье [Отладка ботов с помощью Bot Framework Emulator](bot-service-debug-emulator.md).
1. Из эмулятора отправьте боту сообщение (например, "Hi"). 
1. Для отладки бота используйте панели **Inspector** (Инспектор) и **Log** (Журнал) в правой части окна эмулятора. Например, щелкните любой из пузырьков сообщений (например, пузырек сообщения "Hi", показанный на снимке экрана ниже), чтобы открыть сведения об этом сообщении на панели **Inspector** (Инспектор). Эту панель можно использовать для просмотра запросов и ответов при обмене сообщениями между эмулятором и ботом. Или вы можете щелкнуть любую ссылку на панели **Журнал** для просмотра сведений на панели **Инспектор**.


   ![Панель Inspector (Инспектор) в Bot Framework Emulator](~/media/bot-service-debug-bot/emulator_inspector.png)

### <a name="debug-a-javascript-bot-using-breakpoints-in-visual-studio-code"></a>Отладка бота JavaScript с использованием точек останова в Visual Studio Code

В Visual Studio Code можно установить точки останова и запустить бот в режиме отладки для пошагового выполнения кода. Чтобы установить точки останова в VS Code, выполните следующие действия.

1. Запустите VS Code и откройте папку проекта бота.
2. В строке меню щелкните **Отладка**, а затем — **Начать отладку**. Если будет предложено выбрать механизм среды выполнения для запуска кода, выберите **Node.js**. На этом этапе бот функционирует локально. 
<!--
   > [!NOTE]
   > If you get the "Value cannot be null" error, check to make sure your **Table Storage** setting is valid.
   > The **EchoBot** is default to using **Table Storage**. To use Table Storage in your bot, you need the table *name* and *key*. If you do not have a Table Storage instance ready, you can create one or for testing purposes, you can comment out the code that uses **TableBotDataStore** and uncomment the line of code that uses **InMemoryDataStore**. The **InMemoryDataStore** is intended for testing and prototyping only.
-->
3. Задайте необходимые точки останова. В VS Code можно установить точки останова, наведя указатель мыши над столбцом слева от номеров строк. Появится красная точка. Если щелкнуть на нее, установятся точки останова. Если щелкнуть на нее снова, точки останова будут удалены.

   ![Задание точек останова в VS Code](~/media/bot-service-debug-bot/breakpoint-set.png)

4. Запустите Bot Framework Emulator и подключитесь к боту, как описано выше. 
5. Из эмулятора отправьте боту сообщение (например, "Hi"). Выполнение останавливается на строке, в которой установлена точка останова.

   ![Отладка в VS Code](~/media/bot-service-debug-bot/breakpoint-caught.png)

### <a name="debug-a-c-bot-using-breakpoints-in-visual-studio"></a>Отладка бота C# с использованием точек останова в Visual Studio

В Visual Studio (VS) можно установить точки останова и запустить бот в режиме отладки для пошагового выполнения кода. Чтобы установить точки останова в VS, выполните следующие действия.

1. Перейдите к папке ботов и откройте файл **SLN**. Откроется решение в Visual Studio.
2. В строке меню щелкните **Построить**, затем **Построить решение**.
3. В **обозревателе решений** щелкните **EchoWithCounterBot.cs**. Этот файл при необходимости определяет основную точку останова logic.Set для вашего бота. В VS можно установить точки останова, наведя указатель мыши на столбец слева от номеров строк. Появится красная точка. Если щелкнуть на нее, установятся точки останова. Если щелкнуть на нее снова, точки останова будут удалены.
5. В строке меню щелкните **Отладка**, а затем — **Начать отладку**. На этом этапе бот функционирует локально. 

<!--
   > [!NOTE]
   > If you get the "Value cannot be null" error, check to make sure your **Table Storage** setting is valid.
   > The **EchoBot** is default to using **Table Storage**. To use Table Storage in your bot, you need the table *name* and *key*. If you do not have a Table Storage instance ready, you can create one or for testing purposes, you can comment out the code that uses **TableBotDataStore** and uncomment the line of code that uses **InMemoryDataStore**. The **InMemoryDataStore** is intended for testing and prototyping only.
-->

   ![Задание точки останова в VS](~/media/bot-service-debug-bot/breakpoint-set-vs.png)

7. Запустите Bot Framework Emulator и подключитесь к боту, как описано выше. 
8. Из эмулятора отправьте боту сообщение (например, "Hi"). Выполнение останавливается на строке, в которой установлена точка останова.

   ![Отладка в VS](~/media/bot-service-debug-bot/breakpoint-caught-vs.png)

::: moniker range="azure-bot-service-3.0" 

## <a id="debug-csharp-serverless"></a> Отладка ботов Плана потребления функций C\#

План потребления бессерверной среды C\# в службе ботов имеет больше общего с Node.js, чем обычное приложение C\#, поскольку для него требуется хост-приложение среды выполнения, так же, как и для подсистемы Node. В Azure среда выполнения является частью среды размещения в облаке, но нужно реплицировать эту среду локально на рабочий стол. 

### <a name="prerequisites"></a>Предварительные требования

Перед отладкой плана потребления ботов C# необходимо выполнить следующие задачи.

- Загрузите исходный код для вашего бота (из Azure), как описано в разделе [Настройка непрерывного развертывания для бота в плане службы приложений](bot-service-continuous-deployment.md).
- Скачайте и установите [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Установите <a href="https://www.npmjs.com/package/azure-functions-cli" target="_blank">CLI Функций Azure</a>.
- Установите <a href="https://github.com/dotnet/cli" target="_blank">DotNet CLI</a>.
  
Чтобы иметь возможность отлаживать код с помощью точек останова в Visual Studio 2017, необходимо также выполнить следующие задачи.
  
- Загрузите и установите <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017</a> (Community Edition или более поздней версии).
- Скачайте и установите <a href="https://visualstudiogallery.msdn.microsoft.com/e6bf6a3d-7411-4494-8a1e-28c1a8c4ce99" target="_blank">расширение Command Task Runner для Visual Studio</a>.

> [!NOTE]
> Visual Studio Code в настоящее время не поддерживается.

### <a name="debug-a-consumption-plan-c-functions-bot-using-the-emulator"></a>Отладка плана потребления ботов функций C# с помощью эмулятора

Самый простой способ отладки вашего бота локально — запустить бот, а затем подключиться к нему, используя Bot Framework Emulator. 
Сначала откройте командную строку и перейдите к папке, где в репозитории находится файл **project.json**. Далее выполните команду `dotnet restore`, чтобы восстановить различные пакеты, которые упоминаются в боте.

> [!NOTE]
> Visual Studio 2017 изменяет то, каким образом Visual Studio обрабатывает зависимости. Visual Studio 2015 использует **project.json** для обработки зависимостей, а Visual Studio 2017 использует модель формата **CSPROJ** при загрузке в Visual Studio. Если вы используете Visual Studio 2017, <a href="https://aka.ms/bf-debug-project">загрузите этот файл **CSPROJ**</a> в папку **/messages** в репозитории, прежде чем запускать команду `dotnet restore`.

![Окно командной строки](~/media/bot-service-debug-bot/csharp-azureservice-debug-envconfig.png)

Затем выполните `debughost.cmd`, чтобы загрузить и запустить бота. 

![В командной строке запустите debughost.cmd](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughost.png)

На этом этапе бот функционирует локально. В окне консоли скопируйте конечную точку, от которой debughost ожидает передачи данных (в примере это `http://localhost:3978`). Затем запустите Bot Framework Emulator и вставьте конечную точку в адресную строку эмулятора. В этом примере к конечной точке необходимо также добавить `/api/messages`. Поскольку безопасность для локальной отладки не нужна, можно оставить поля **ИД приложения Microsoft** и **Пароль приложения Microsoft** пустыми. Щелкните **Connect**, чтобы установить подключение к боту с помощью указанной конечной точки.

![Настройка эмулятора](~/media/bot-service-debug-bot/mac-azureservice-emulator-config.png)

После подключения эмулятора к боту отправьте боту сообщение, введя текст в поле, расположенном в нижней части окна эмулятора (т. е., в нижнем левом углу, где появляется надпись **Введите сообщение...**). С помощью панелей **Журнал** и **Инспектор** в правой части окна эмулятора можно просмотреть запросы и ответы при обмене сообщениями между эмулятором и ботом.

![Тестирование с помощью эмулятора](~/media/bot-service-debug-bot/mac-azureservice-debug-emulator.png)

Кроме того, можно просмотреть подробные сведения журнала в окне консоли.

![Окно консоли](~/media/bot-service-debug-bot/csharp-azureservice-debug-debughostlogging.png)

::: moniker-end

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Отладка бота с помощью файлов записей разговоров](~/v4sdk/bot-builder-debug-transcript.md)

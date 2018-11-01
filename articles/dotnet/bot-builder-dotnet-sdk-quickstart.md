---
title: Создание бота с помощью пакета SDK Bot Builder для .NET | Документация Майкрософт
description: Создание бота с помощью пакета SDK Bot Builder для .NET — мощной платформы для создания ботов.
keywords: Bot Builder SDK, create a bot, quickstart, .NET, getting started, C# bot
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: sdk
ms.date: 09/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 32743e895f2390fe018dc3801ed5b8a67b32a8cc
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999381"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-net"></a>Создание бота с помощью пакета SDK Bot Builder для .NET
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

В этом кратком руководстве описывается, как с помощью шаблона C# создать бот и протестировать его, используя Bot Framework Channel Emulator. 

## <a name="prerequisites"></a>Предварительные требования
- Visual Studio [2017](https://www.visualstudio.com/downloads).
- Шаблон пакета SDK для Bot Builder версии 4 для [C#](https://botbuilder.myget.org/feed/aitemplates/package/vsix/BotBuilderV4.fbe0fc50-a6f1-4500-82a2-189314b7bea2).
- Bot Framework Channel [Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases).
- Навыки разработки для [ASP.Net Core](https://docs.microsoft.com/aspnet/core/) и асинхронного программирования в [C#](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/index).

## <a name="create-a-bot"></a>Создание бота
Установите шаблон BotBuilderVSIX.vsix, скачанный при выполнении предварительных требований. 

В Visual Studio создайте новый проект бота с использованием шаблона Bot Builder Echo Bot версии 4.

![Проект Visual Studio](../media/azure-bot-quickstarts/bot-builder-dotnet-project.png)

> [!TIP] 
> При необходимости измените тип сборки проекта на ``.Net Core 2.1``. Также при необходимости обновите [пакеты NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/install-and-use-a-package-in-visual-studio).

Благодаря шаблону проект содержит весь код, необходимый для создания бота в рамках этого краткого руководства. Теперь нет необходимости писать дополнительный код.

## <a name="start-your-bot-in-visual-studio"></a>Запуск бота в Visual Studio

Когда вы нажмете кнопку запуска, Visual Studio создаст приложение, развернет его в узле localhost и запустит веб-браузер для отображения страницы приложения `default.htm`. На этом этапе бот выполняется локально.

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота

После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Open Bot** (Открыть бот) на вкладке приветствия в эмуляторе. 
2. Выберите BOT-файл, расположенный в каталоге с созданным решением Visual Studio.

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом

Отправьте сообщение боту и получите от него сообщение в ответ.

![Работающий эмулятор](../media/emulator-v4/emulator-running.png)

> [!NOTE]
> Если вы видите, что сообщение не отправляется, вам может потребоваться перезагрузить компьютер, так как ngrok еще не получил необходимые привилегии в вашей системе (это нужно выполнить только один раз).

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Принципы работы бота](../v4sdk/bot-builder-basics.md) 

---
title: Публикация службы ботов из системы управления версиями или Visual Studio | Документы Майкрософт
description: Сведения об однократной публикации службы ботов из Visual Studio и постоянной публикации службы ботов из системы управления версиями.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 980ee1c6e4e54ff3f74ccfa618b0aeff7b507815
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305686"
---
# <a name="publish-a-bot-to-bot-service"></a>Публикация бота в службе ботов

После обновления исходного кода бота на C# вы можете опубликовать бот, запустив его в службе ботов с помощью Visual Studio. Вы также можете автоматически публиковать исходный код бота на C# или Node.js каждый раз при отправке файла исходного кода в службу управления версиями в любой интегрированной среде разработки.


## <a name="publish-a-bot-on-app-service-plan-from-the-online-code-editor"></a>Публикация бота в плане службы приложений из интерактивного редактора кода

Если вы не настроили непрерывное развертывание, то можете изменять исходные файлы в интерактивном редакторе кода. Чтобы развернуть измененный исходный файл, выполните следующие действия.

4. Щелкните значок "Открыть консоль".  
    ![Значок консоли](~/media/azure-bot-service-console-icon.png)
2. В окне консоли введите **build.cmd** и нажмите клавишу ВВОД.


## <a name="publish-c-bot-on-app-service-plan-from-visual-studio"></a>Публикация бота C# в плане службы приложений из Visual Studio 

Чтобы настроить публикацию из Visual Studio с помощью файла `.PublishSettings`, выполните следующие действия:

1. На портале Azure выберите свою службу ботов, перейдите на вкладку **Сборка** и щелкните **Скачать ZIP-файл**.
3. Извлеките содержимое загруженного ZIP-файла в локальную папку.
4. В проводнике найдите файл решения Visual Studio (SLN-файл) для вашего бота и дважды щелкните его.
4. В Visual Studio выберите **Вид** > **Обозреватель решений**.
5. На панели обозревателя щелкните проект правой кнопкой мыши и выберите **Опубликовать...**. Откроется окно "Публикация". 
6. В окне "Публикация" выберите **Создать профиль**, щелкните **Импортировать профиль** и нажмите **ОК**.
7. Перейдите в папку проекта, затем перейдите в папку **PostDeployScripts**, выберите файл, имя которого заканчивается на `.PublishSettings`, и нажмите кнопку **Открыть**.

Теперь вы настроили публикацию для этого проекта. Чтобы опубликовать локальный исходный код в службе ботов, щелкните проект правой кнопкой мыши, выберите **Опубликовать...**  и нажмите кнопку **Опубликовать...**. 

## <a name="set-up-continuous-deployment"></a>Непрерывное развертывание с использованием GIT в службе приложений Azure

По умолчанию служба ботов позволяет разрабатывать боты непосредственно в браузере с помощью редактора Azure без применения локального редактора и систем управления версиями. Однако редактор Azure не позволяет управлять файлами в приложении (например, добавлять, переименовывать или удалять файлы). Если вам необходима возможность управлять файлами в приложении, можно настроить непрерывное развертывание и использовать интегрированную среду разработки (ITE) и выбранную систему управления версиями (например, Visual Studio Team, GitHub, Bitbucket). После настройки непрерывного развертывания все изменения кода, которые фиксируются в системе управления версиями, автоматически развертываются в Azure. После настройки непрерывного развертывания вы можете [отлаживать бот локально](bot-service-debug-bot.md).

> [!NOTE]
> При включении непрерывного развертывания для своего бота необходимо отправить изменения кода в вашу систему управления версиями. Если вы хотите снова редактировать код в редакторе Azure, необходимо [отключить непрерывное развертывание](#disable-continuous-deployment).

Чтобы включить непрерывное развертывание для приложения-бота, выполните следующие действия.

## <a name="set-up-continuous-deployment-for-a-bot-on-an-app-service-plan"></a>Настройка непрерывного развертывания для бота в плане службы приложений

В этом разделе описано, как включить непрерывное развертывание для бота, созданного с помощью службы ботов с планом размещения службы приложений.

1. На портале Azure найдите бот Azure, перейдите на вкладку **Сборка** и найдите раздел **Непрерывное развертывание из системы управления версиями**.
2. Для Visual Studio Online или GitHub укажите маркер доступа, полученный на этих сайтах. Ваш исходный код будет загружен из Azure в репозиторий исходного кода.
3. Для других систем управления версиями выберите **Другие** и следуйте инструкциям. 
3. Нажмите **Включить**.  

### <a name="create-an-empty-repository-and-download-bot-source-code"></a>Создайте пустой репозиторий и скачайте исходный код бота.

Если вы хотите использовать систему управления версиями, *отличную от* Visual Studio Online или Github, выполните следующие действия. Visual Studio Online и Github извлекают исходный код вашего бота из Azure, чтобы пользователи этих двух служб могли пропустить эти действия.

3. Найдите страницу бота для плана службы приложений в Azure, перейдите на вкладку **Сборка**, найдите раздел **Скачивание исходного кода** и щелкните **Скачать ZIP-файл**.
1. Создайте пустой репозиторий в одной из систем управления версиями, которые поддерживает Azure.

    ![Система управления версиями](~/media/continuous-integration-sourcecontrolsystem.png)

3. Извлеките содержимое скачанного ZIP-файла в локальную папку, с которой вы хотите синхронизировать источник развертывания.
4. Нажмите кнопку **Настроить** и следуйте инструкциям на экране. 

## <a name="set-up-continuous-deployment-for-a-bot-on-a-consumption-plan"></a>Настройка непрерывного развертывания для бота в плане потребления 

Выберите источник развертывания для вашего бота и подключите репозиторий. 

1. На портале Azure для бота Azure перейдите на вкладку **Параметры** и нажмите кнопку **Настроить**, чтобы развернуть раздел **Непрерывное развертывание**.  
2. Выполните приведенные инструкции и установите флажок, чтобы подтвердить, что вы готовы. 
3. Нажмите кнопку **Настройка**, выберите источник развертывания, соответствующий системе управления версиями, в которой вы создали пустой репозиторий, и выполните необходимые действия для подключения.   


## <a name="disable-continuous-deployment"></a>Отключение непрерывного развертывания 

При отключении непрерывного развертывания служба управления версиями продолжает работать, но ваши изменения не публикуются в Azure автоматически. Чтобы отключить непрерывное развертывание, выполните следующие действия:

1. Если у вашего бота есть план размещения службы приложений на портале Azure, найдите бот Azure, перейдите на вкладку **Сборка** и найдите раздел **Непрерывное развертывание из системы управления версиями**, *или...* 
2. Если у вашего бота есть план потребления, перейдите на вкладку **Параметры**, разверните раздел **Непрерывное развертывание** и щелкните **Настроить**.
3. В области **Развертывания** выберите службу управления версиями, в которой включено непрерывное развертывание, и щелкните **Отключиться**.  


## <a name="additional-resources"></a>Дополнительные ресурсы

Чтобы узнать, как выполнить локальную отладку бота после настройки непрерывного развертывания, см. раздел [Отладка бота службы ботов](bot-service-debug-bot.md).

В этой статье будут выделены конкретные характеристики непрерывного развертывания службы ботов. Сведения о непрерывном развертывании в отношении службы приложений Azure см. в разделе <a href="https://azure.microsoft.com/en-us/documentation/articles/app-service-continuous-deployment/" target="_blank">Непрерывное развертывание в службе приложений Azure</a>.
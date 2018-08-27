---
title: Перенос бота C# из плана потребления в план службы приложений | Документация Майкрософт
description: Перенесите бот C# службы Bot из плана потребления в план размещения службы приложений.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 1ba574de619e482b6248d33bcb805080c0ea72d0
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305103"
---
# <a name="change-the-hosting-plan-for-your-bot-service"></a>Изменение плана размещения для службы ботов

В этом разделе объясняется, как можно перенести бот скрипта C# с помощью плана потребления в бот C# с помощью плана службы приложений. 

## <a name="advantages-of-a-bot-on-an-app-service-plan"></a>Преимущества бота, работающего в рамках плана службы приложений

Боты в рамках плана службы приложений выполняются как веб-приложения Azure. Боты веб-приложений могут выполнять функции, недоступные ботам плана потребления:

- Бот веб-приложения может добавить определения пользовательских маршрутов.
- Бот веб-приложения может включить сервер Websocket. 
- Бот, использующий план потребления, имеет те же ограничения, что и весь код, выполняемый в службе "Функции Azure". Дополнительные сведения см. в статье <a target='_blank' href='/azure/azure-functions/functions-scale'>Масштабирование и размещение Функций Azure</a>.

## <a name="download-your-existing-bot-source"></a>Загрузка имеющегося источника бота

Выполните следующие действия, чтобы загрузить исходный код имеющегося бота.

1. В боте Azure щелкните вкладку **Параметры** и раскройте раздел **Непрерывное развертывание**.  
2. Щелкните синюю кнопку, чтобы скачать ZIP-файл, содержащий исходный код для вашего бота.  
    ![Загрузка ZIP-файла бота](~/media/continuous-deployment-consumption-download.png)
3. Извлеките содержимое загруженного ZIP-файла в локальную папку. 


## <a name="create-a-bot-template"></a>Создание шаблона бота

В службе Bot предусмотрены одни и те же шаблоны для ботов плана потребления и службы приложений. Чтобы выполнить миграцию из бота плана потребления, создайте бот плана службы приложений в службе Bot на основе одного шаблона. Базовый код может различаться между двумя типами размещения для одного шаблона, но новое веб-приложение имеет похожую структуру и возможности настройки, используемые имеющимся ботом.

## <a name="download-the-new-bot-source"></a>Загрузка источника нового бота

Выполните следующие действия, чтобы загрузить исходный код нового бота.

1. В боте Azure щелкните вкладку **Build** (Сборка), найдите раздел **Download source code** (Загрузка исходного кода) и щелкните **Download zip file** (Загрузить ZIP-файл). 
2. Извлеките содержимое загруженного ZIP-файла в локальную папку.

## <a name="add-source-files-to-new-solution"></a>Добавление исходных файлов в новое решение

Некоторые CSX-файлы могут компилироваться и запускаться как CS-файлы в новом решении. Создайте CS-файл для каждого CSX-файла в решении, за исключением `run.csx`. Вы перенесете логику `run.csx` вручную. В CS-файлах может понадобиться добавить объявление класса и необязательное объявление пространства имен.

## <a name="migrate-runcsx-logic-into-your-project"></a>Перенос логики run.csx в проект

Проекты скриптов C# содержат метод `Run`, в котором обрабатываются разные значения `ActivityTypes`. Импортируйте логику обработки действий в метод `MessageController.Post`, в `MessageController.cs`.

## <a name="remove-compiler-keywords"></a>Удаления ключевых слов компилятора

Файлы скриптов C# могут включать ссылочный модуль с помощью ключевого слова `#r`. Удалите следующие строки, а также добавьте их в качестве ссылки в проект Visual Studio. Кроме того, удалите ключевые слова `#load`. При этом в компиляцию файла вставляются другие файлы исходного кода. Вместо этого добавьте все файлы `.csx` в проект в качестве исходного кода `.cs`.

## <a name="add-references-from-projectjson"></a>Добавление ссылок из файла project.json

Если ваш бот плана потребления добавляет ссылки на NuGet в файл `project.json`, добавьте эти ссылки в новое решение Visual Studio, щелкнув правой кнопкой мыши проект на панели обозревателя решений и выбрав пункт **Добавить ссылку**.

### <a name="add-references-that-were-implicit"></a>Добавление неявных ссылок

Бот службы Bot, работающий в рамках плана потребления, неявно включает эти ссылки во все исходные CSX-файлы. Для источника, перенесенного в эти исходные CS-файлы, может потребоваться добавить явные ссылки на эти классы:

- `TraceWriter` в пакете NuGet `Microsoft.Azure.WebJobs`, предоставляющем типы пространств имен `Microsoft.Azure.WebJobs.Host`. 
- Триггеры таймера в пакете NuGet `Microsoft.Azure.WebJobs.Extensions`.
- `Newtonsoft.Json`, `Microsoft.ServiceBus`и другие сборки с автоматическими ссылками.
- `System.Threading.Tasks` и другие автоматически импортированные пространства имен.

Дополнительные сведения см. в разделе *Преобразование в файлы классов* в статье <a target='_blank' href='https://blogs.msdn.microsoft.com/appserviceteam/2017/03/16/publishing-a-net-class-library-as-a-function-app/'>Publishing a .NET class library as a Function App</a> (Публикация библиотеки классов .NET в качестве приложения-функции).

## <a name="debug-your-new-bot"></a>Отладка нового бота

Локальная отладка бота, выполняющегося в рамках плана службы приложений, гораздо проще, чем локальная отладка бота, выполняющегося в рамках плана потребления. Для локальной отладки перенесенного кода можно воспользоваться [эмулятором](bot-service-debug-emulator.md).

## <a name="publish-from-visual-studio-or-set-up-continuous-deployment"></a>Публикация из Visual Studio или настройка непрерывного развертывания

Наконец, опубликуйте перенесенный исходный код в службе Bot. Для этого импортируйте его файл `.PublishSettings` и щелкните **Опубликовать** или [настройте непрерывное развертывание](bot-service-debug-bot.md).
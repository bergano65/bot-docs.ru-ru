---
title: Перенос имеющегося чат-бота в новый проект .NET Core — Служба Azure Bot
description: Узнайте, как перенести имеющийся бот .NET версии 3 в пакет SDK версии 4 для нового проекта .NET Core.
keywords: bot migration, formflow, dialogs, v3 bot
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 06/17/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 776ba2013e7f66abc36cbf4810a7520a8da8e496
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791106"
---
# <a name="migrate-a-net-v3-bot-to-a-net-core-v4-bot"></a>Перенос бота .NET версии 3 в бот .NET Core версии 4

В этой статье описывается, как преобразовать бот [ContosoHelpdeskChatBot версии 3](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V3) в бот версии 4 _в новом проекте .NET Core_.
Преобразование включает в себя следующие действия:

1. создание проекта на основе шаблона;
1. установка дополнительных пакетов NuGet при необходимости;
1. настройка бота согласно требованиям, обновление файла Startup.cs, а затем класса контроллера;
1. обновление класса бота;
1. копирование и обновление диалогов и моделей;
1. и наконец, перенос данных.

В результате этого преобразования вы получите [бот ContosoHelpdeskChatBot в .NET Core версии 4](https://github.com/microsoft/BotBuilder-Samples/tree/master/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore).
Дополнительные сведения о том, как перенести бот в .NET Framework версии 4 _без преобразования типа проекта_ см. [здесь](conversion-framework.md).

Пакет SDK версии 4 для Bot Framework использует тот же базовый REST API, что и пакет SDK версии 3. Но в пакете SDK версии 4 выполнен рефакторинг кода предыдущей версии пакета SDK для повышения гибкости и улучшения контроля над ботами. Основные изменения в пакете SDK:

- Управление состоянием осуществляется через соответствующие объекты и методы доступа к свойствам.
- Также изменились процессы настройки обработчика шагов и передачи ему действий.
- Элементов с возможностью оценки больше нет. Вы можете проверить наличие глобальных команд в обработчике шагов, прежде чем передавать средства управления в диалоги.
- Новая библиотека Dialogs, которая сильно отличается от предыдущей версии. Вам потребуется преобразовать все старые диалоги в новую систему, используя компонентные, каскадные диалоги и созданную сообществом реализацию диалогов Formflow для версии 4.

См. дополнительные сведения об [отличиях между версиями 3 и 4 пакета SDK для .NET](migration-about.md).

## <a name="create-the-new-project-using-a-template"></a>Создание проекта на основе шаблона

Чтобы создать проект для бота, сделайте следующее:

1. Установите [шаблон для C#](https://aka.ms/bot-vsix) пакета SDK Bot Framework версии 4, если вы этого еще не сделали.
1. Откройте Visual Studio и создайте на основе шаблона проект для бота Echo. Назовите проект `ContosoHelpdeskChatBot`.

## <a name="install-additional-nuget-packages"></a>Установка дополнительных пакетов NuGet

С шаблоном устанавливаются большинство необходимых пакетов, в том числе **Microsoft.Bot.Builder** и **Microsoft.Bot.Connector**.

1. Добавьте **Bot.Builder.Community.Dialogs.Formflow**.

    Это библиотека сообщества для сборки диалогов версии 4 из файлов определения Formflow версии 3. Одно из зависимостей для нее является **Microsoft.Bot.Builder.Dialogs**, поэтому она тоже автоматически устанавливается.

1. Добавьте **log4net**, чтобы обеспечить поддержку ведения журналов.

## <a name="personalize-your-bot"></a>Настройка бота согласно требованиям

1. Переименуйте файл бота **Bots\EchoBot.cs** на **Bots\DialogBot.cs**, а класс `EchoBot` — на `DialogBot`.
1. Переименуйте контроллер **Controllers\BotController.cs** на **Controllers\MessagesController.cs**, а класс `BotController` — на `MessagesController`.

## <a name="update-your-startupcs-file"></a>Обновление файла Startup.cs

Для бота изменились способ управления состоянием и получения входящих действий. Теперь для версии 4 нужно самостоятельно настроить элементы инфраструктуры [управления состоянием](../bot-builder-concept-state.md). К примеру, в версии 4 применяется адаптер ботов для проверки подлинности и передачи действий в код бота, а это означает, что все свойства состояния нужно объявить заранее.

Мы создадим свойство состояния для `DialogState`, который потребуется в версии 4 для поддержки диалогов. Мы будем использовать внедрение зависимостей, чтобы получить информацию, которая нужна контроллеру и боту.

В **Startup.cs**:

1. Обновите инструкции `using`.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Startup.cs?range=4-13)]

1. Удалите этот конструктор:
    ```csharp
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }
    ```

1. Удалите свойство `Configuration`.

1. Вместо метода `ConfigureServices` сохраните следующий код:  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Startup.cs?range=19-41)]

Возможно, на этом этапе возникнут ошибки времени компиляции. Мы исправим их на следующих шагах. 

## <a name="messagescontroller-class"></a>Класс MessagesController
Этот класс обрабатывает запрос. Внедрение зависимостей обеспечивает получение данных адаптера и реализацию IBot в среде выполнения. Этот класс шаблона останется без изменений. 

В этой части кода в версии 4 бот запускает шаг, что существенно отличается от того, как это делает контроллер сообщений версии 3. Все остальное, кроме обработчика шагов бота, можно считать стандартным кодом.

Обработчик шагов бота будет определен в файле **Bots\DialogBot.cs**.

### <a name="ignore-the-cancelscorable-and-globalmessagehandlersbotmodule-classes"></a>Игнорирование классов CancelScorable и GlobalMessageHandlersBotModul

Так как в версии 4 больше нет элементов с возможностью оценки, эти классы следует просто игнорировать. Мы обновим обработчик шагов так, чтобы бот реагировал на сообщение `cancel`.

## <a name="update-your-bot-class"></a>Обновление класса бота

В версии 4 логика цикла обработки сообщений или обработчика шагов находится преимущественно в файле бота. Мы получаем данные от `ActivityHandler`, который определяет обработчики для распространенных типов действий.

1. Обновите файл **Bots\DialogBots.cs**.

1. Обновите инструкции `using`.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=4-8)]

1. Обновите `DialogBot`, добавив универсальный параметр для диалога.
    [!code-csharp[Class definition](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=19)]

1. Добавьте эти поля и конструктор для инициализации. В ASP.NET также используется внедрение зависимостей, чтобы получить значения параметров.
    [!code-csharp[Fields and constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=21-28)]

1. Измените внедрение `OnMessageActivityAsync` так, чтобы оно вызывало главный диалог. (Мы определим метод расширения `Run` чуть позже.)

[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=38-47)]

1. Обновите `OnTurnAsync`, чтобы сохранять состояние беседы в конце шага. В версии 4 нам нужно сделать это явным образом, чтобы записать состояние на уровне сохраняемости. Метод `ActivityHandler.OnTurnAsync` вызывает методы конкретного обработчика действий с учетом типа полученного действия. Поэтому мы сохраним состояние после вызова базового метода.
    [!code-csharp[OnTurnAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Bots/DialogBot.cs?range=30-36)]

### <a name="create-the-run-extension-method"></a>Создание метода расширения запуска

Мы создаем метод расширения, чтобы объединить код для запуска простого компонентного диалога из бота.

Создайте файл **DialogExtensions.cs** и реализуйте метод расширения `Run`.
[!code-csharp[The extension](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/DialogExtensions.cs?range=4-26)]

## <a name="copy-over-and-convert-your-dialogs"></a>Копирование и преобразование диалогов

Мы будем вносить изменения в исходные диалоги версии 3, что позволит использовать их с пакетом SDK версии 4. Пока не беспокойтесь об ошибках компилятора. Они исчезнут, когда преобразование завершится.
Мы не будем изменять исходный код больше, чем необходимо, поэтому некоторые предупреждения компилятора сохранятся после завершения переноса.

Все диалоги теперь будут не реализацией интерфейса `IDialog<object>` из версии 3, а станут производными от `ComponentDialog`.

Этот бот использует четыре диалога, и все их нужно преобразовать:

| | |
|---|---|
| [RootDialog](#update-the-root-dialog) | Представляет варианты на выбор и запускает другие диалоги. |
| [InstallAppDialog](#update-the-install-app-dialog) | Обрабатывает запросы на установку приложения на компьютер. |
| [LocalAdminDialog](#update-the-local-admin-dialog) | Обрабатывает запросы на получение прав локального администратора на компьютере. |
| [ResetPasswordDialog](#update-the-reset-password-dialog) | Обрабатывает запросы на сброс пароля. |

Мы не будем копировать класс `CancelScorable`, так как больше нет элементов с возможностью оценки. Вы можете проверить наличие _глобальных_ команд в обработчике шагов, прежде чем передавать средства управления в диалоги.

Все они собирают данные, но не выполняют никаких операций на компьютере.

1. Создайте папку **Dialogs** (Диалоги) в проекте.
1. Скопируйте эти файлы из каталога с диалогами проекта версии 3 в новый каталог:
    - **InstallAppDialog.cs**;
    - **LocalAdminDialog.cs**;
    - **ResetPasswordDialog.cs**;
    - **RootDialog.cs**.

### <a name="make-solution-wide-dialog-changes"></a>Внесение изменений в диалоги на уровне решения

1. Для всего решения измените все вхождения `IDialog<object>` на `ComponentDialog`.
1. Для всего решения измените все вхождения `IDialogContext` на `DialogContext`.
1. Для каждого класса диалогов удалите атрибут `[Serializable]`.

Поток управления и обмен сообщениями в диалогах теперь обрабатываются другими способами, и это нужно учитывать при преобразовании каждого диалога.

| Операция | Код для версии 3 | Код для версии 4 |
| :--- | :--- | :--- |
| Обработка запуска диалога | Реализуйте `IDialog.StartAsync` | Сделайте этот шаг первым в каскадном диалоге или реализуйте `Dialog.BeginDialogAsync` |
| Обработка продолжения диалога | Вызовите `IDialogContext.Wait` | Добавьте в каскадный диалог дополнительные шаги или реализуйте `Dialog.ContinueDialogAsync` |
| Отправка сообщения пользователю | Вызовите `IDialogContext.PostAsync` | Вызовите `ITurnContext.SendActivityAsync` |
| Запуск дочернего диалога | Вызовите `IDialogContext.Call` | Вызовите `DialogContext.BeginDialogAsync` |
| Информирование о завершении текущего диалога | Вызовите `IDialogContext.Done` | Вызовите `DialogContext.EndDialogAsync` |
| Получение введенных пользователем данных | Используйте параметр `IAwaitable<IMessageActivity>` | Используйте приглашение в каскадном диалоге или `ITurnContext.Activity` |

Примечания о коде для версии 4:

- В коде диалога используйте свойство `DialogContext.Context`, чтобы получить контекст текущего шага.
- Каскадные шаги имеют параметр `WaterfallStepContext`, который является производным от `DialogContext`.
- Все конкретные классы диалогов и приглашений наследуют от абстрактного класса `Dialog`.
- Идентификатор присваивается при создании компонентного диалога. Каждый диалог в наборе диалогов должен иметь уникальный в пределах этого набора идентификатор.

### <a name="update-the-root-dialog"></a>Обновление корневого диалога

В этом боте корневой диалог предлагает пользователю выбрать один вариант из набора, а затем на основе этого выбора запускает дочерний диалог. Этот цикл повторяется неограниченно долго, пока существует беседа.

- Основной поток можно создать в формате каскадного диалога — это новая концепция в пакете SDK версии 4. Он выполняет фиксированный набор шагов в заданном порядке. Подробнее см. статью [Реализация последовательной беседы](~/v4sdk/bot-builder-dialog-manage-conversation-flow.md).
- Запросы теперь обрабатываются через классы prompt. Эти короткие дочерние диалоги предлагают ввести данные, выполняют минимальную обработку и проверку этих данных и возвращают полученное значение. Дополнительные сведения о сборе данных от пользователя с помощью запросов диалога см. в [этой статье](~/v4sdk/bot-builder-prompts.md).

В файле **Dialogs/RootDialog.cs** сделайте следующее:

1. Обновите инструкции `using`.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=4-10)]

1. Нам нужно преобразовать варианты `HelpdeskOptions` из списка строк в список вариантов. Они будут применяться в строке запроса, которая в качестве допустимых входных данных принимает номер выбранного варианта в списке, значение любого из вариантов или его синоним.
    [!code-csharp[HelpDeskOptions](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=28-33)]

1. Добавьте конструктор. Этот код делает следующее:
   - Каждому экземпляру диалога при создании присваивается идентификатор. Идентификатор диалога входит в набор диалогов, к которому добавляется этот диалог. Не забывайте, что бот уже инициализирован с использованием объекта диалога в классе `MessageController`. Для всех `ComponentDialog` есть свой внутренний набор диалогов с отдельным набором идентификаторов диалогов.
   - Включает остальные диалоги, в том числе запросы выбора, в качестве дочерних диалогов. Здесь мы используем для каждого диалога в качестве идентификатора соответствующее имя класса.
   - Определяет каскадный диалог с тремя шагами. Их мы реализуем чуть позже.
     - Сначала диалог предлагает пользователю выбрать задание для выполнения.
     - Затем он запускает дочерний диалог, связанный с выбранным вариантом.
     - И уже потом он перезапускает сам себя.
   - Каждый шаг каскадного диалога является делегатом. Мы реализуем их далее, по возможности сохраняя существующий код каждого исходного диалога.
   - При запуске компонентного диалога будет запущен _начальный диалог_. По умолчанию это первый дочерний диалог, добавленный в компонентный диалог. Мы явным образом определим свойство `InitialDialogId`. Это значит, что нам не нужно добавлять основной каскадный диалог первым в набор диалогов. Например, вы можете сначала добавить запросы, и это не вызовет никаких проблем во время выполнения.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=35-49)]

1. Метод `StartAsync` можно удалить. При запуске компонентный диалог автоматически выполняет свой _начальный_ диалог. В нашем примере это тот каскадный диалог, который мы определяем в конструкторе. Он автоматически начинает выполнение с первого действия.

1. Мы удалим методы `MessageReceivedAsync` и `ShowOptions`, а затем заменим их первым шагом нашего каскадного диалога. Эти два метода ранее приветствовали пользователя и предлагали выбор из доступных вариантов.
   - Здесь вы видите список значений, приветствия и сообщения об ошибках, которые передаются в вызов запроса на выбор в качестве параметров.
   - Нам не нужно указывать следующий метод, который будет вызван в диалоге, так как каскадный диалог автоматически перейдет к следующему шагу после завершения запроса на выбор.
   - Запрос выбора повторяется в цикле, пока не получит допустимые входные данные или весь стек диалогов не будет отменен.
    [!code-csharp[PromptForOptionsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=51-65)]

1. `OnOptionSelected` можно заменить вторым шагом каскадного диалога. Здесь мы также запускаем дочерний диалог в зависимости от ввода пользователя.
   - Запрос выбора возвращает значение `FoundChoice`. Оно передается в свойстве `Result` этого шага. Стек диалогов обрабатывает все возвращаемые значения как объекты. Если это значение возвращается из вашего диалога, вы получите тип значения для этого объекта. Список значений, возвращаемых разными типами, вы можете изучить [здесь](../bot-builder-concept-dialog.md#prompt-types).
   - Так как запрос выбора не создает исключение, мы можем удалить блок try-catch.
   - Но нам нужно добавить механизм передачи управления, чтобы этот метод всегда возвращал допустимое значение. Этот код никогда не должен выполняться, но если это произойдет из-за сбоя, он корректно завершит работу диалога.
    [!code-csharp[ShowChildDialogAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=67-102)]

1. Наконец, замените старый метод `ResumeAfterOptionDialog` последним шагом каскадного диалога.
    - В исходном диалоге мы завершали работу и возвращали номер билета. Теперь же мы перезапускаем каскадный диалог, заменяя его старый экземпляр в стеке диалогов новым экземпляром того же диалога. Это стало возможным, так как исходное приложение всегда игнорирует возвращаемое значение (номер билета) и перезапускает корневой диалог.
    [!code-csharp[ResumeAfterAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/RootDialog.cs?range=104-138)]

### <a name="update-the-install-app-dialog"></a>Обновление диалога установки приложения

Диалог установки приложения выполняет несколько логических задач, которые мы сейчас настроим в четырех шагах каскадного диалога. Перенос существующего кода в каскадные шаги будет отдельной логической задачей для каждого из диалогов. Для каждого шага указан исходный метод, из которого взят его код.

1. Запрос на ввод строки поиска пользователем.
1. Запрос возможных совпадений в базе данных.
   - Если есть одно совпадение, выбор этого значения и продолжение работы.
   - Если совпадений несколько, запрос на выбор одного варианта пользователем.
   - Если совпадений нет, выход из диалога.
1. Запрос на выбор компьютера, где пользователем будет установлено приложение.
1. Сохранение сведений в базе данных и отправка сообщения с подтверждением.

В файле **Dialogs/InstallAppDialog.cs** сделайте следующее:

1. Обновите инструкции `using`.  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=4-11)]

1. Определите константу для ключа, чтобы отслеживать собранные сведения.
    [!code-csharp[Key ID](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=17-18)]

1. Добавьте конструктор и инициализируйте набор диалогов для компонентного диалога.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=20-33)]

1. `StartAsync` можно заменить первым шагом каскадного диалога.
    - Но нам придется самостоятельно управлять состоянием, для которого мы применим объект установки приложения в состоянии диалога.
    - Сообщение с запросом пользовательских данных становится необязательным параметром в вызове запроса.
    [!code-csharp[GetSearchTermAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=35-50)]

1. `appNameAsync` и `multipleAppsAsync` можно заменить вторым шагом каскадного диалога.
    - Теперь мы получаем результат запроса, а не просто проверяем последнее сообщение пользователя.
    - Запрос к базе данных и инструкции if организованы здесь так же, как в `appNameAsync`. Код в каждом блоке инструкции if обновлен для поддержки диалогов версии 4.
        - Если обнаружится только одно совпадение, мы обновляем состояние диалога и переходим к следующему шагу.
        - Если совпадений несколько, мы применяем запрос выбора и предлагаем пользователю выбрать вариант из списка. Это означает, что можно просто удалить `multipleAppsAsync`.
        - Если совпадений нет, мы завершаем диалог и возвращаем значение NULL в корневой диалог.
    [!code-csharp[ResolveAppNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=52-91)]

1. Метод `appNameAsync` также запрашивает у пользователя имя компьютера после обработки запроса. Мы реализуем эту часть логики в следующем шаге каскадного диалога.
    - Не забывайте, что в версии 4 нам нужно самостоятельно управлять состоянием. Единственной сложностью будет то, что на этот шаг с предыдущего мы можем попасть через две разные ветви логики.
    - Для запроса имени компьютера у пользователя мы применим тот же запрос текста, что и ранее, но с другими вариантами ответов.
    [!code-csharp[GetMachineNameAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=93-114)]

1. Логика из метода `machineNameAsync` попадает в заключительный шаг каскадного диалога.
    - Мы извлекаем имя компьютера из результата, возвращенного запросом текста, и обновляем состояние диалога.
    - Вызов для обновления базы данных мы удалим, так как вспомогательный код размещается в другом проекте.
    - Затем мы отправляем пользователю сообщение об успешном выполнении и заканчиваем диалог.
    [!code-csharp[SubmitRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=116-135)]

1. Чтобы сымитировать вызов базы данных, мы создадим вызов с использованием `getAppsAsync`, что позволит обратиться к статическому списку, а не реальной базе данных.
    [!code-csharp[GetAppsAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/InstallAppDialog.cs?range=137-200)]

### <a name="update-the-local-admin-dialog"></a>Обновление диалога локального администратора

В версии 3 этот диалог приветствовал пользователя, запускал диалог Formflow и сохранял результат в базе данных. Эти действия легко преобразовать в каскадный диалог с двумя шагами.

1. Обновите инструкции `using`. Обратите внимание на то, что этот диалог включает диалог Formflow из версии 3. В версии 4 мы применим библиотеку Formflow, созданную сообществом.
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=4-8)]

1. Мы можем удалить свойство экземпляра для `LocalAdmin`, так как результат будет доступен в состоянии диалога.

1. Добавьте конструктор и инициализируйте набор диалогов для компонентного диалога. Диалог Formflow создается аналогичным образом. Мы просто добавляем его в набор диалогов для компонентного диалога в конструкторе.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=14-23)]

1. `StartAsync` можно заменить первым шагом каскадного диалога. Мы уже создали Formflow в конструкторе, а остальные две инструкции будут преобразованы соответствующим образом. Обратите внимание, что `FormBuilder` присваивает имя типа модели в качестве идентификатора созданного диалога (`LocalAdminPrompt` для этой модели).
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=25-35)]

1. `ResumeAfterLocalAdminFormDialog` можно заменить вторым шагом каскадного диалога. Нам нужно получать возвращаемое значение из контекста шага, а не свойства экземпляра.
    [!code-csharp[SaveResultAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=37-50)]

1. `BuildLocalAdminForm` почти не изменяется, за исключением того, что теперь FormFlow не будет обновлять свойство экземпляра.
    [!code-csharp[BuildLocalAdminForm](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/LocalAdminDialog.cs?range=52-76)]

### <a name="update-the-reset-password-dialog"></a>Обновление диалога смены пароля

В версии 3 этот диалог приветствует пользователя, авторизует пользователя по коду доступа, возвращает ошибку авторизации или запускает диалог Formflow и сбрасывает значение пароля. Все это неплохо преобразуется в формат каскада.

1. Обновите инструкции `using`. Обратите внимание на то, что этот диалог включает диалог Formflow из версии 3. В версии 4 мы применим библиотеку Formflow, созданную сообществом.
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=4-9)]

1. Добавьте конструктор и инициализируйте набор диалогов для компонентного диалога. Диалог Formflow создается аналогичным образом. Мы просто добавляем его в набор диалогов для компонентного диалога в конструкторе.
    [!code-csharp[Constructor](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=15-25)]

1. `StartAsync` можно заменить первым шагом каскадного диалога. Мы уже создали Formflow в конструкторе. Для всего остального мы сохраняем прежнюю логику, а вызовы версии 3 просто преобразуем в эквиваленты для версии 4.
    [!code-csharp[BeginFormflowAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=27-45)]

1. `sendPassCode` используется в основном для примера. Мы закомментировали весь исходный код, и теперь этот метод просто возвращает значение true. Также мы можем удалить адрес электронной почты, так как он не использовался в старой версии бота.
    [!code-csharp[SendPassCode](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=47-81)]

1. `BuildResetPasswordForm` сохраняется в неизменном виде.

1. Мы можем заменить `ResumeAfterResetPasswordFormDialog` вторым шагом каскадного диалога, а возвращаемое значение теперь будем получать из контекста шага. Мы удалили адрес электронной почты, с которым исходный диалог не выполнял никаких действий, и вместо запроса к базе данных возвращаем фиктивный результат. Мы полностью сохраняем прежнюю логику, а вызовы версии 3 преобразуем в эквиваленты для версии 4.
    [!code-csharp[ProcessRequestAsync](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Dialogs/ResetPasswordDialog.cs?range=90-113)]

## <a name="copy-over-and-update-models-as-necessary"></a>Копирование и обновление моделей при необходимости
Вы можете использовать модели версии 3 в библиотеке сообщества FormFlow версии 4. 

1. Создайте папку **Models** (Модели) в проекте.
1. Скопируйте эти файлы из каталога с моделями проекта версии 3 в новый каталог:
    - **InstallApp.cs**;
    - **LocalAdmin.cs**;
    - **LocalAdminPrompt.cs**;
    - **ResetPassword.cs**;
    - **ResetPasswordPrompt.cs**.

### <a name="update-using-statements"></a>Обновление инструкций using

В классах моделей необходимо обновить инструкции `using` как показано ниже.

1. В **InstallApps.cs** измените их так:  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/InstallApp.cs?range=4-5)]

1. В **LocalAdmin.cs** измените их так:  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/LocalAdmin.cs?range=4-5)]

1. В **LocalAdminPrompt.cs** измените их так:  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/LocalAdminPrompt.cs?range=4)]

1. В **ResetPassword.cs** измените их так:

[!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/ResetPassword.cs?range=4-5)]
    Кроме того, удалите инструкции `using` в пространстве имен.

1. В **ResetPasswordPrompt.cs** измените их так:  
    [!code-csharp[Using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/ResetPasswordPrompt.cs?range=4-5)]

### <a name="additional-changes"></a>Дополнительные изменения

В **ResetPassword.cs** измените тип возвращаемого значения `MobileNumber` так:

[!code-csharp[MobileNumber](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/Models/ResetPassword.cs?range=17)]

## <a name="final-porting-steps"></a>Конечный перенос данных 
Чтобы завершить процесс переноса, сделайте следующее:

1. Создайте класс `AdapterWithErrorHandler`, чтобы определить адаптер, что предусматривает обработчик ошибок, перехватывающий исключения в программном обеспечении промежуточного слоя и приложениях. Адаптер обрабатывает и направляет входящие действия через конвейер ПО промежуточного слоя бота в логику бота, а затем обратно. Используйте код ниже, чтобы создать класс:

 [!code-csharp[MobileNumber](~/../botbuilder-samples/MigrationV3V4/CSharp/ContosoHelpdeskChatBot-V4NetCore/ContosoHelpdeskChatBot/AdapterWithErrorHandler.cs?range=4-46)]
1. Измените страницу **wwwroot\default.htm** согласно требованиям.

## <a name="run-and-test-your-bot-in-the-emulator"></a>Запуск и тестирование бота в эмуляторе

На этом этапе мы уже можем запустить бота локально в IIS и подключить его к эмулятору.

1. Запустите бот в IIS.
1. Запустите эмулятор и подключитесь к конечной точке бота (например, **http://localhost:3978/api/messages** ).
    - Если бот запускается впервые, щелкните **Файл > Новая программа-робот** и следуйте инструкциям на экране. В противном случае, чтобы открыть существующий бот, щелкните **Файл > Открыть программу-робота**.
    - Внимательно проверьте параметры порта в конфигурации. Например, если в браузере бот открывается по адресу `http://localhost:3979/`, укажите в эмуляторе для бота конечную точку `http://localhost:3979/api/messages`.
1. Все четыре диалога должны работать. Вы можете установить точки останова в нужных шагах каскада, чтобы проверить контекст и состояние диалога в этих точках.

## <a name="additional-resources"></a>Дополнительные ресурсы

Тематические статьи по версии 4:

- [Принципы работы бота](../bot-builder-basics.md)
- [Управление состоянием](../bot-builder-concept-state.md)
- [Библиотека диалогов](../bot-builder-concept-dialog.md)

Пошаговые инструкции для версии 4:

- [Отправка и получение текстовых сообщений](../bot-builder-howto-send-messages.md)
- [Сохранение данных пользователя и диалога](../bot-builder-howto-v4-state.md)
- [Реализация процесса общения](../bot-builder-dialog-manage-conversation-flow.md)
- [Отладка с помощью эмулятора](../../bot-service-debug-emulator.md)
- [Добавление данных телеметрии в бот](../bot-builder-telemetry.md)

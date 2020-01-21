---
title: Создание сложной последовательной беседы с использованием ветвей и циклов — Служба Azure Bot
description: Сведения о том, как управлять сложным потоком беседы с помощью диалогов из пакета SDK Bot Framework.
keywords: complex conversation flow, repeat, loop, menu, dialogs, prompts, waterfalls, dialog set
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 701eea560d46acc9d3917716366c509e1032e30f
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75798560"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>Создание сложного потока беседы с использованием ветвления и циклов

[!INCLUDE[applies-to](../includes/applies-to.md)]

С помощью библиотеки диалогов можно управлять простыми и сложными процессами общения.
В этой статье мы покажем, как управлять сложными беседами с ветвлениями и циклами.
Мы также продемонстрируем передачу аргументов между разными частями диалога.

## <a name="prerequisites"></a>предварительные требования

- Понимание [основных принципов работы ботов][concept-basics], [управления состоянием][concept-state], [библиотек диалогов][concept-dialogs] и [реализации последовательного процесса общения][simple-dialog].
- Копия примера сложного диалога для [**C#** ][cs-sample], [**JavaScript**][js-sample] и [**Python**][python-sample].

## <a name="about-this-sample"></a>Об этом примере

Этот пример содержит бот, который может выполнять регистрацию пользователей для оценки одной или двух компаний из заданного списка.

`DialogAndWelcomeBot` расширяет `DialogBot`, который определяет обработчики для разных действий и обработчик реплик бота. `DialogBot` запускает диалоги.

- Метод _run_ используется в `DialogBot` для запуска диалога.
- `MainDialog` является родительским для двух других диалогов, которые вызываются в определенные моменты. В этой статье приведены сведения об этих диалогах.

Диалоги здесь разделены на компоненты `MainDialog`, `TopLevelDialog` и `ReviewSelectionDialog`, которые вместе выполняют следующие действия.

- В них запрашивается имя и возраст пользователя, и в зависимости от возраста пользователя выполняется _ветвление_.
  - Если пользователь слишком молод, ему не предлагают оценивать компании.
  - Если пользователь достиг нужного возраста, начинается сбор предпочтений пользователя.
    - В диалогах предлагается пользователю выбрать компанию для оценки.
    - Если пользователь выбирает компанию, диалоги _в цикле_ переходят к выбору второй компании.
- И в завершение в диалогах выражается благодарность пользователю за участие.

Для управления сложной беседой в них используется два каскадных диалога и несколько запросов.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

![Сложный процесс общения в боте](./media/complex-conversation-flow.png)

Чтобы использовать диалоги, необходимо установить в проект пакет NuGet **Microsoft.Bot.Builder.Dialogs**.

**Startup.cs.**

В `Startup` мы регистрируем службы для бота. Эти службы доступны в других частях кода через механизм внедрения зависимостей.

- Основные службы бота: поставщик учетных данных, адаптер и реализация бота.
- Службы для управления состоянием: хранилище, состояние пользователя и состояние беседы.
- Диалог, который будет использовать бот.

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Startup.cs?range=22-36)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![Сложный процесс общения в боте](./media/complex-conversation-flow-js.png)

Чтобы использовать диалоги, в проект следует установить пакет npm **botbuilder-dialogs**.

**index.js**

Мы создаем следующие требуемые службы для бота.

- Основные службы: адаптер и реализация бота.
- Управление состоянием: хранилище, состояние пользователя и состояние беседы.
- Диалоги: робот использует их для управления беседами.

[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=26-65)]

# <a name="pythontabpython"></a>[Python](#tab/python)

![Сложный процесс общения в боте](./media/complex-conversation-flow-python.png)

Чтобы использовать диалоги, в проект следует установить пакет pypi **botbuilder-dialogs**, выполнив `pip install botbuilder-dialogs`.

**app.py**

Мы создаем службы для бота, которые нужны в других частях кода.

- Основные службы бота: адаптер и реализация бота.
- Службы для управления состоянием: хранилище, состояние пользователя и состояние беседы.
- Диалог, который будет использовать бот.

[!code-python[ConfigureServices](~/../botbuilder-python/samples/python/43.complex-dialog/app.py?range=28-75)]

---

> [!NOTE]
> Хранилище в памяти используется только для тестирования и не предназначено для рабочей среды.
> Для ботов в рабочей среде обязательно используйте постоянное хранилище любого типа.

## <a name="define-a-class-in-which-to-store-the-collected-information"></a>Определение класса для хранения собранных данных

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**UserProfile.cs**

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/UserProfile.cs?range=8-16)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**userProfile.js**

[!code-javascript[UserProfile class](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/userProfile.js?range=4-12)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**data_models/user_profile.py**

[!code-python[UserProfile class](~/../botbuilder-python/samples/python/43.complex-dialog/data_models/user_profile.py?range=7-13)]


---

## <a name="create-the-dialogs-to-use"></a>Создание диалогов для использования

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

Мы определили компонентный диалог `MainDialog`, который содержит несколько основных шагов и управляет другими диалогами и запросами. На первом шаге здесь вызывается метод `TopLevelDialog`, который описан ниже.

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/MainDialog.cs?range=31-50&highlight=3)]

**Dialogs\TopLevelDialog.cs**

Исходный диалог верхнего уровня состоит из четырех шагов:

1. запрос имени пользователя;
1. запрос возраста пользователя;
1. ветвление в зависимости от возраста пользователя;
1. отправка пользователю благодарности за участие и возвращение собранных сведений.

На первом шаге мы очищаем профиль пользователя, чтобы диалог каждый раз запускался с пустым профилем. Так как последний шаг при завершении будет возвращать сведения, `AcknowledgementStepAsync` следует завершить с сохранением состояния пользователя и передачей этих сведений в главный диалог для использования на последнем шаге.

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=39-96&highlight=3-4,47-49,56-57)]

**Dialogs\ReviewSelectionDialog.cs**

Диалог review-selection запускается из диалога верхнего уровня `StartSelectionStepAsync`, и состоит из двух шагов:

1. предложение пользователю выбрать компанию для оценки или ввести `done`, чтобы завершить процесс;
1. повтор того же диалога или выход, в зависимости от некоторых условий.

В такой конфигурации диалог верхнего уровня всегда находится в стеке выше, чем диалог review-selection, то есть его можно считать дочерним элементом диалога верхнего уровня.

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=42-106)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

Мы определили компонентный диалог `MainDialog`, который содержит несколько основных шагов и управляет другими диалогами и запросами. На первом шаге здесь вызывается метод `TopLevelDialog`, который описан ниже.

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=43-55&highlight=2)]

**dialogs/topLevelDialog.js**

Исходный диалог верхнего уровня состоит из четырех шагов:

1. запрос имени пользователя;
1. запрос возраста пользователя;
1. ветвление в зависимости от возраста пользователя;
1. отправка пользователю благодарности за участие и возвращение собранных сведений.

На первом шаге мы очищаем профиль пользователя, чтобы диалог каждый раз запускался с пустым профилем. Так как последний шаг при завершении будет возвращать сведения, `acknowledgementStep` следует завершить с сохранением состояния пользователя и передачей этих сведений в главный диалог для использования на последнем шаге.

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=32-76&highlight=2-3,37-39,43-44)]

**dialogs/reviewSelectionDialog.js**

Диалог review-selection запускается из диалога верхнего уровня `startSelectionStep`, и состоит из двух шагов:

1. предложение пользователю выбрать компанию для оценки или ввести `done`, чтобы завершить процесс;
1. повтор того же диалога или выход, в зависимости от некоторых условий.

В такой конфигурации диалог верхнего уровня всегда находится в стеке выше, чем диалог review-selection, то есть его можно считать дочерним элементом диалога верхнего уровня.

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=33-78)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**dialogs\main_dialog.py**

Мы определили компонентный диалог `MainDialog`, который содержит несколько основных шагов и управляет другими диалогами и запросами. На первом шаге здесь вызывается метод `TopLevelDialog`, который описан ниже.

[!code-python[step implementations](~/../botbuilder-python/samples/python/43.complex-dialog/dialogs/main_dialog.py?range=29-50&highlight=4)]

**dialogs\top_level_dialog.py**

Исходный диалог верхнего уровня состоит из четырех шагов:

1. запрос имени пользователя;
1. запрос возраста пользователя;
1. ветвление в зависимости от возраста пользователя;
1. отправка пользователю благодарности за участие и возвращение собранных сведений.

На первом шаге мы очищаем профиль пользователя, чтобы диалог каждый раз запускался с пустым профилем. Так как последний шаг при завершении будет возвращать сведения, `acknowledgementStep` следует завершить с сохранением состояния пользователя и передачей этих сведений в главный диалог для использования на последнем шаге.

[!code-python[step implementations](~/../botbuilder-python/samples/python/43.complex-dialog/dialogs/top_level_dialog.py?range=43-95&highlight=2-3,43-44,52)]

**dialogs/review_selection_dialog.py**

Диалог review-selection запускается из диалога верхнего уровня `startSelectionStep`, и состоит из двух шагов:

1. предложение пользователю выбрать компанию для оценки или ввести `done`, чтобы завершить процесс;
1. повтор того же диалога или выход, в зависимости от некоторых условий.

В такой конфигурации диалог верхнего уровня всегда находится в стеке выше, чем диалог review-selection, то есть его можно считать дочерним элементом диалога верхнего уровня.

[!code-python[step implementations](~/../botbuilder-python/samples/python/43.complex-dialog/dialogs/review_selection_dialog.py?range=42-99)]

---

## <a name="implement-the-code-to-manage-the-dialog"></a>Реализация кода для управления диалогом

Обработчик шагов бота повторяет один и тот же поток общения, который определен в этих диалогах.
При получении сообщения от пользователя применяется следующий алгоритм.

1. Продолжить активный диалог, если таковой имеется.
   - Если активного диалога нет, очистить профиль пользователя и запустить диалог верхнего уровня.
   - Если активный диалог завершается, собрать и сохранить полученные данные и отобразить информационное сообщение.
   - Все остальные ситуации означают, что мы находимся в середине процесса активного диалога, и дополнительные действия в данный момент не требуются.
1. Сохранить состояние диалога, чтобы не потерять обновленные сведения о состоянии диалога.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

<!-- **DialogExtensions.cs**

In this sample, we've defined a `Run` helper method that we will use to create and access the dialog context.
Since component dialog defines an inner dialog set, we have to create an outer dialog set that's visible to the message handler code, and use that to create a dialog context.

- `dialog` is the main component dialog for the bot.
- `turnContext` is the current turn context for the bot.

[!code-csharp[Run method](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/DialogExtensions.cs?range=13-24)]

-->

**Bots\DialogBot.cs**

Обработчик сообщений вызывает метод `RunAsync` для управления диалогом. Мы переопределили обработчик шагов так, чтобы он сохранял любые изменения в состоянии беседы и пользователя, происходящие на определенном шаге. Базовый метод `OnTurnAsync` вызовет метод `OnMessageActivityAsync`, чтобы гарантировать вызов сохранения данных в конце этой реплики.

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogBot.cs?range=33-48&highlight=5-7)]

**Bots\DialogAndWelcome.cs**

`DialogAndWelcomeBot` расширяет описанный выше `DialogBot`, предоставляя приветственное сообщение при присоединении пользователя к беседе. Именно он создается в `Startup.cs`.

[!code-csharp[On members added](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogAndWelcome.cs?range=21-38)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

В этом примере мы определили метод `run`, который будет использоваться для создания контекста диалога и доступа к нему.
Так как компонентный диалог определяет набор внутренних диалогов, нам следует создать внешний набор диалогов, доступный для кода обработчика сообщений, чтобы использовать его для создания контекста диалога.

- `turnContext` содержит контекст текущей реплики бота.
- Мы создали метод доступа `accessor` для управления состоянием диалога.

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=32-41)]

**bots/dialogBot.js**

Обработчик сообщений вызывает вспомогательный метод `run` для управления диалогом, и мы реализовали обработчик реплик так, чтобы он сохранял любые изменения в состоянии беседы и пользователя, выполненные в рамках текущей реплики. Вызов `next` позволяет базовой реализации вызвать метод `onDialog`, чтобы гарантировать вызов сохранения данных в конце этой реплики.

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=24-41)]

**bots/dialogAndWelcomeBot.js**

`DialogAndWelcomeBot` расширяет описанный выше `DialogBot`, предоставляя приветственное сообщение при присоединении пользователя к беседе. Именно он создается в `index.js`.

[!code-javascript[On members added](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogAndWelcomeBot.js?range=10-21)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**bots/dialog_bot.py**

Обработчик сообщений вызывает метод `run_dialog` для управления диалогом. Мы переопределили обработчик шагов так, чтобы он сохранял любые изменения в состоянии беседы и пользователя, происходящие на определенном шаге. Базовый метод `on_turn` вызовет метод `on_message_activity`, чтобы гарантировать вызов сохранения данных в конце этой реплики.

[!code-python[Overrides](~/../botbuilder-python/samples/python/43.complex-dialog/bots/dialog_bot.py?range=29-41&highlight=32-34)]

**bots/dialog_and_welcome_bot.py**

`DialogAndWelcomeBot` расширяет описанный выше `DialogBot`, предоставляя приветственное сообщение при присоединении пользователя к беседе. Именно он создается в `config.py`.

[!code-python[on_members_added](~/../botbuilder-python/samples/python/43.complex-dialog/bots/dialog_and_welcome_bot.py?range=28-39)]

---

## <a name="branch-and-loop"></a>Ветвление и цикл

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\TopLevelDialog.cs**

Ниже приведен пример логики ветвления для шага в диалоге _верхнего уровня_.

[!code-csharp[branching logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=68-80)]

**Dialogs\ReviewSelectionDialog.cs**

Ниже приведен пример логики ветвления для шага в диалоге _просмотра и выбора_.

[!code-csharp[looping logic](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=96-105)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/topLevelDialog.js**

Ниже приведен пример логики ветвления для шага в диалоге _верхнего уровня_.

[!code-javascript[branching logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=56-64)]

**dialogs/reviewSelectionDialog.js**

Ниже приведен пример логики ветвления для шага в диалоге _просмотра и выбора_.

[!code-javascript[looping logic](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=71-77)]

# <a name="pythontabpython"></a>[Python](#tab/python)

**dialogs/top_level_dialog.py**

Ниже приведен пример логики ветвления для шага в диалоге _верхнего уровня_.

[!code-python[branching logic](~/../botbuilder-python/samples/python/43.complex-dialog/dialogs/top_level_dialog.py?range=71-80)]

**dialogs/review_selection_dialog.py**

Ниже приведен пример логики ветвления для шага в диалоге _просмотра и выбора_.

[!code-python[looping logic](~/../botbuilder-python/samples/python/43.complex-dialog/dialogs/review_selection_dialog.py?range=93-98)]

---

## <a name="to-test-the-bot"></a>Тестирование бота

1. Установите [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme), если вы этого еще не сделали.
1. Выполните этот пример на локальном компьютере.
1. Запустите эмулятор, подключитесь к боту и отправьте несколько сообщений, как показано ниже.

![Пример тестирования сложного диалога](~/media/emulator-v4/test-complex-dialog.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

Общие сведения о создании диалогов можно получить в статье [о реализации последовательного потока диалога][simple-dialog], в котором на основе одного каскадного диалога и нескольких запросов создается простой пример взаимодействия, который задает пользователю набор вопросов.

Библиотека диалогов выполняет простую проверку запросов. Вы также можете добавить любую собственную проверку. Дополнительные сведения о сборе данных от пользователя с помощью запросов диалога см. в [этой статье][dialog-prompts].

Чтобы упростить код диалога и использовать его повторно в нескольких ботах, следует определить компоненты набора диалогов в отдельном классе.
Подробнее об этом см. в статье [о повторном использовании диалогов][component-dialogs].

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Повторное использование диалогов](bot-builder-compositcontrol.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-dialog]: bot-builder-dialog-manage-conversation-flow.md
[dialog-prompts]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-complex-dialog-sample
[js-sample]: https://aka.ms/js-complex-dialog-sample
[python-sample]: https://aka.ms/python-complex-dialog-sample

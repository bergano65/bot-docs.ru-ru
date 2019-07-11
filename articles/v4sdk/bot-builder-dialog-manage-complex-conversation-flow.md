---
title: Создание сложного потока беседы с использованием ветвления и циклов | Документация Майкрософт
description: Сведения о том, как управлять сложным потоком беседы с помощью диалогов из пакета SDK Bot Framework.
keywords: complex conversation flow, repeat, loop, menu, dialogs, prompts, waterfalls, dialog set
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b7ffa16c2f0a00043b12faec1d31bbfe5bfa250f
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/05/2019
ms.locfileid: "67587476"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>Создание сложного потока беседы с использованием ветвления и циклов

[!INCLUDE[applies-to](../includes/applies-to.md)]

С помощью библиотеки диалогов можно управлять простыми и сложными процессами общения.
В этой статье мы покажем, как управлять сложными беседами с ветвлениями и циклами.
Мы также продемонстрируем передачу аргументов между разными частями диалога.

## <a name="prerequisites"></a>Предварительные требования

- Понимание [основных принципов работы ботов][concept-basics], управления состоянием, , [managing state][concept-state][библиотек диалогов][concept-dialogs] и [реализации последовательного процесса общения][simple-dialog].
- Копия примера сложного диалога на [**C#** ][cs-sample] or [**JavaScript**][js-sample].

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

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Startup.cs?range=22-39)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![Сложный процесс общения в боте](./media/complex-conversation-flow-js.png)

Чтобы использовать диалоги, в проект следует установить пакет npm **botbuilder-dialogs**.

**index.js**

Мы создаем службы для бота, которые нужны в других частях кода.

- Основные службы бота: адаптер и реализация бота.
- Службы для управления состоянием: хранилище, состояние пользователя и состояние беседы.
- Диалог, который будет использовать бот.

[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=25-38)]
[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=43-45)]

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

`DialogAndWelcomeBot` расширяет описанный выше `DialogBot`, предоставляя приветственное сообщение при присоединении пользователя к диалогу. Именно он вызывается из `Startup.cs`.

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

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=30-47)]

**bots/dialogAndWelcomeBot.js**

`DialogAndWelcomeBot` расширяет описанный выше `DialogBot`, предоставляя приветственное сообщение при присоединении пользователя к диалогу. Именно он вызывается из `Startup.cs`.

[!code-javascript[On members added](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogAndWelcomeBot.js?range=10-21)]

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

## <a name="next-steps"></a>Дополнительная информация

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

---
title: Создание сложной последовательной беседы с использованием ветвей и циклов — Служба Azure Bot
description: Сведения о том, как управлять сложным потоком беседы с помощью диалогов из пакета SDK Bot Framework.
keywords: complex conversation flow, repeat, loop, menu, dialogs, prompts, waterfalls, dialog set
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/30/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f8854469b131b11d53c90047d438af4e26b06f97
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2020
ms.locfileid: "77441635"
---
# <a name="create-advanced-conversation-flow-using-branches-and-loops"></a>Создание сложного потока беседы с использованием ветвления и циклов

[!INCLUDE[applies-to](../includes/applies-to.md)]

С помощью библиотеки диалогов можно создавать сложные потоки беседы.
В этой статье объясняется, как управлять сложными беседами с ответвлениями и циклами, а также как передавать аргументы между разными частями диалога.

## <a name="prerequisites"></a>Предварительные требования

- Понимание [основных принципов работы ботов][concept-basics], [управления состоянием][concept-state], [библиотек диалогов][concept-dialogs] и [реализации последовательного процесса общения][simple-dialog].
- Копия примера сложного диалога для [**C#** ][cs-sample], [**JavaScript**][js-sample] или [**Python**][python-sample].

## <a name="about-this-sample"></a>Об этом примере

Этот пример содержит бот, который может выполнять регистрацию пользователей для оценки одной или двух компаний из заданного списка.
Для управления потоком беседы в боте используются три диалога-компонента.
Каждый диалог-компонент содержит каскадный диалог и все запросы, необходимые для сбора данных, вводимых пользователем.
Эти диалоги подробно описаны в следующих разделах.
Бот использует состояние беседы для управления диалогами и состояние пользователя для сохранения сведений о нем и тех компаниях, которые пользователь хочет оценить.

Бот разработан на основе обработчика действий. Как и во многих примерах ботов, этот бот отправляет пользователю приветствие, использует диалоги для обработки сообщений от пользователя, а также сохраняет состояние пользователя и диалога до окончания реплики.

### <a name="c"></a>[C#](#tab/csharp)

Чтобы использовать диалоги, установите пакет NuGet **Microsoft.Bot.Builder.Dialogs**.

![Сложный процесс общения в боте](./media/complex-conversation-flow.png)

### <a name="javascript"></a>[JavaScript](#tab/javascript)

Чтобы использовать диалоги, в проект следует установить пакет npm **botbuilder-dialogs**.

![Сложный процесс общения в боте](./media/complex-conversation-flow-js.png)

### <a name="python"></a>[Python](#tab/python)

Чтобы использовать диалоги, установить в проект пакет PyPI **botbuilder-dialogs**. Для этого выполните `pip install botbuilder-dialogs`.

![Сложный процесс общения в боте](./media/complex-conversation-flow-python.png)

---

## <a name="define-the-user-profile"></a>Определение профиля пользователя

Профиль пользователя будет содержать такие собранные в диалогах сведения, как имя пользователя, возраст и выбранные для оценки компании.

### <a name="c"></a>[C#](#tab/csharp)

**UserProfile.cs**

[!code-csharp[UserProfile class](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/UserProfile.cs?range=8-16)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**userProfile.js**

[!code-javascript[UserProfile class](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/userProfile.js?range=4-12)]

### <a name="python"></a>[Python](#tab/python)

**data_models/user_profile.py**

[!code-python[UserProfile class](~/../botbuilder-samples/samples/python/43.complex-dialog/data_models/user_profile.py?range=7-13)]

---

## <a name="create-the-dialogs"></a>Создание диалогов

Этот бот содержит три диалога:

- В основном диалоге запускается общий процесс общения и создается сводка собранных сведений.
- В диалоге верхнего уровня происходит сбор сведений пользователя. Здесь включена логика ветвления на основе возраста пользователя.
- В диалоге оценки и выбора пользователь может последовательно выбрать компании для оценки. Для этого используется логика цикла.

### <a name="the-main-dialog"></a>Основной диалог

Основной диалог состоит из двух шагов:

1. Запуск диалога верхнего уровня.
1. Получение и формирование сводки данных в профиле пользователя, собранных в диалоге верхнего уровня, сохранение данных о состоянии пользователя и информирование о завершении основного диалога.

#### <a name="c"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/MainDialog.cs?range=31-50)]

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/mainDialog.js?range=43-55)]

#### <a name="python"></a>[Python](#tab/python)

**dialogs\main_dialog.py**

[!code-python[step implementations](~/../botbuilder-samples/samples/python/43.complex-dialog/dialogs/main_dialog.py?range=29-50)]

---

### <a name="the-top-level-dialog"></a>Диалог верхнего уровня

Диалог верхнего уровня состоит из четырех шагов:

1. запрос имени пользователя;
1. запрос возраста пользователя;
1. Запуск диалога оценки и выбора или переход к следующему шагу в зависимости от возраста пользователя.
1. отправка пользователю благодарности за участие и возвращение собранных сведений.

На первом шаге в рамках сохранения состояния диалога создается пустой профиль пользователя. В начале диалога создается пустой профиль, который заполняется в ходе ведения беседы. В конце диалога возвращаются собранные сведения.

На третьем шаге (начало выбора) в потоке беседы создается ветвь на основе возраста пользователя.

#### <a name="c"></a>[C#](#tab/csharp)

**Dialogs\TopLevelDialog.cs**

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/TopLevelDialog.cs?range=39-96&highlight=30-42)]

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

**dialogs/topLevelDialog.js**

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/topLevelDialog.js?range=32-76&highlight=25-33)]

#### <a name="python"></a>[Python](#tab/python)

**dialogs\top_level_dialog.py**

[!code-python[step implementations](~/../botbuilder-samples/samples/python/43.complex-dialog/dialogs/top_level_dialog.py?range=43-95&highlight=29-38)]

---

### <a name="the-review-selection-dialog"></a>Диалог выбора элементов для обзора

Диалог оценки и выбора состоит из двух шагов:

1. Предложение пользователю выбрать компанию для оценки или ввести `done`, чтобы завершить процесс.
   - Если в начале диалога были известны какие-либо сведения, их можно получить с помощью свойства _options_ контекста каскадного шага. Диалог оценки и выбора поддерживает автоматический перезапуск. Благодаря этому пользователь может выбрать несколько компаний для оценки.
   - Если пользователь уже выбрал компанию для оценки, ее название удаляется из доступных вариантов.
   - Вариант `done` добавлен, чтобы пользователь мог завершить цикл на раннем этапе.
1. повтор того же диалога или выход, в зависимости от некоторых условий.
   - Если пользователь выбрал компанию для оценки, ее нужно добавить в список.
   - Если пользователь выбрал две компании или решил выйти, нужно завершить диалог и вывести список собранных сведений.
   - В противном случае перезапустите диалог, инициализируя его с содержимым списка.

#### <a name="c"></a>[C#](#tab/csharp)

**Dialogs\ReviewSelectionDialog.cs**

[!code-csharp[step implementations](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Dialogs/ReviewSelectionDialog.cs?range=42-106&highlight=55-64)]

#### <a name="javascript"></a>[JavaScript](#tab/javascript)

**dialogs/reviewSelectionDialog.js**

[!code-javascript[step implementations](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/dialogs/reviewSelectionDialog.js?range=33-78&highlight=39-45)]

#### <a name="python"></a>[Python](#tab/python)

**dialogs/review_selection_dialog.py**

[!code-python[step implementations](~/../botbuilder-samples/samples/python/43.complex-dialog/dialogs/review_selection_dialog.py?range=42-99&highlight=51-58)]

---

## <a name="run-the-dialogs"></a>Запуск диалогов

Класс _dialog bot_ позволяет расширить возможности обработчика действия. Этот класс содержит логику выполнения диалогов.
Класс _dialog and welcome bot_ позволяет расширить возможности чат-бота, а также добавить приветствие пользователя, когда он присоединяется к беседе.

Обработчик реплик бота повторяет поток беседы, который определен в трех диалогах.
При получении сообщения от пользователя происходит следующее:

1. Запускается основной диалог.
   - Если стек диалога пуст, начнется основной диалог.
   - Если это не так, диалог все еще выполняется, а значит, активный диалог будет продолжен.
1. Сохраняется состояние пользователя, беседы и диалога, чтобы не потерять новые сведения о состоянии.

### <a name="c"></a>[C#](#tab/csharp)

**Bots\DialogBot.cs**

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Bots/DialogBot.cs?range=33-48&highlight=5-7,14-15)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/dialogBot.js**

[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=24-32&highlight=4-5)]
[!code-javascript[run](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/bots/dialogBot.js?range=35-44&highlight=7-9)]

### <a name="python"></a>[Python](#tab/python)

**bots/dialog_bot.py**

[!code-python[Overrides](~/../botbuilder-samples/samples/python/43.complex-dialog/bots/dialog_bot.py?range=29-41&highlight=4-6,9-13)]

---

## <a name="register-services-for-the-bot"></a>Регистрация служб для бота

При необходимости создайте и зарегистрируйте службы:

- Основные службы бота: адаптер и реализация бота.
- Службы для управления состоянием: хранилище, состояние пользователя и состояние беседы.
- Корневой диалог, который будет использоваться бот.

### <a name="c"></a>[C#](#tab/csharp)

**Startup.cs.**

[!code-csharp[ConfigureServices](~/../botbuilder-samples/samples/csharp_dotnetcore/43.complex-dialog/Startup.cs?range=18-37)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**index.js**

[!code-javascript[ConfigureServices](~/../botbuilder-samples/samples/javascript_nodejs/43.complex-dialog/index.js?range=26-43)]

### <a name="python"></a>[Python](#tab/python)

**app.py**

[!code-python[ConfigureServices](~/../botbuilder-samples/samples/python/43.complex-dialog/app.py?range=29-32)]
[!code-python[ConfigureServices](~/../botbuilder-samples/samples/python/43.complex-dialog/app.py?range=70-77)]

---

> [!NOTE]
> Хранилище в памяти используется только для тестирования и не предназначено для рабочей среды.
> Для ботов в рабочей среде обязательно используйте постоянное хранилище любого типа.

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

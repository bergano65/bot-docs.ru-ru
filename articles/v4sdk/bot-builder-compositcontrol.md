---
title: Повторное использование диалогов | Документация Майкрософт
description: Сведения о разделении логики бота с использованием компонентных диалогов в пакете SDK Bot Framework.
keywords: составной элемент управления, модульная логика бота
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/16/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6840305684eff7dc325a541a0b545396e6c30fa8
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/03/2019
ms.locfileid: "65033466"
---
# <a name="reuse-dialogs"></a>Повторное использование диалогов

[!INCLUDE[applies-to](../includes/applies-to.md)]

Компонентные диалоги позволяют создавать независимые диалоги для обработки определенных сценариев, разбивая большие наборы диалогов на более управляемые фрагменты. Каждый из этих компонентов имеет отдельный набор диалогов, что позволяет избежать конфликтов имен с внешними наборами диалогов.

## <a name="prerequisites"></a>Предварительные требования

- Опыт работы с [ботами][concept-basics], [библиотеками диалогов][concept-dialogs] и [управлением сообщениями][simple-flow].
- Копия примера диалога с несколькими запросами для [**C#**][cs-sample] или [**JavaScript**][js-sample].

## <a name="about-the-sample"></a>Сведения о примере

В примере диалога с несколькими запросами мы применим каскадный диалог, несколько запросов и компонентный диалог для реализации простого взаимодействия, в рамках которого пользователю предлагается несколько вопросов. Код диалога циклически перебирает следующие действия:

| Действия        | Тип запроса  |
|:-------------|:-------------|
| Запрос к пользователю о режиме транспортировки | Запрос выбора |
| Запрос имени пользователя | Запрос текста |
| Запрос к пользователю, готов ли он указать свой возраст | Запрос подтверждения |
| Если получен положительный ответ, запрос возраста пользователя  | Запрос числа с проверкой, при которой принимается возраст только в диапазоне от 0 до 150. |
| Запрос на подтверждение собранной информации | Повторный запрос подтверждения |

И наконец, если получен положительный ответ, отображается вся собранная информация. В противном случае пользователь получает сообщение о том, что данные не будут сохранены.

## <a name="implement-the-component-dialog"></a>Реализация компонентного диалога

В примере диалога с несколькими запросами мы применим _каскадный диалог_, несколько _запросов_ и _компонентный диалог_ для создания простого взаимодействия, в рамках которого пользователю предлагается несколько вопросов.

Компонентный диалог включает один или несколько диалогов. Компонентный диалог содержит внутренний набор диалогов, в котором все добавляемые к диалоги и запросы имеют собственные идентификаторы, доступные только для компонентного диалога.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы использовать диалоги, установите пакет NuGet **Microsoft.Bot.Builder.Dialogs**.

**Dialogs\UserProfileDialog.cs**

Класс `UserProfileDialog` является производным от класса `ComponentDialog`.

[!code-csharp[Class](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=13)]

В конструкторе метод `AddDialog` добавляет диалоги и запросы в компонентный диалог. Первый элемент, добавленный с помощью этого метода, настраивается как начальный диалог. Но это можно изменить, явно задав свойство `InitialDialogId`. При запуске компонентного диалога будет запущен _начальный диалог_.

[!code-csharp[Constructor](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=17-42)]

Это реализация первого шага каскадного диалога.

[!code-csharp[First step](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Dialogs/UserProfileDialog.cs?range=44-54)]

См. подробнее о [реализации последовательного потока диалога](bot-builder-dialog-manage-complex-conversation-flow.md).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы использовать диалоги, в проект следует установить пакет npm **botbuilder-dialogs**.

**dialogs/userProfileDialog.js**

Класс `UserProfileDialog` является расширением `ComponentDialog`.

[!code-javascript[Class](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=24)]

В конструкторе метод `AddDialog` добавляет диалоги и запросы в компонентный диалог. Первый элемент, добавленный с помощью этого метода, настраивается как начальный диалог. Но это можно изменить, явно задав свойство `InitialDialogId`. При запуске компонентного диалога будет запущен _начальный диалог_.

[!code-javascript[Constructor](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=25-47)]

Это реализация первого шага каскадного диалога.

[!code-javascript[First step](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=66-73)]

См. подробнее о [реализации последовательного потока диалога](bot-builder-dialog-manage-complex-conversation-flow.md).

---

Во время выполнения компонентный диалог поддерживает собственный стек диалогов. При запуске компонентного диалога происходит следующее:

- Создается экземпляр, который добавляется во внешний стек диалогов.
- Создается внутренний стек диалогов, который добавляется в свое состояние.
- Запускается начальный диалог, который добавляется во внутренний стек диалогов.

Из родительского контекста активным диалогом будет считаться компонентный диалог. Внутри него активным диалогом будет считаться начальный диалог.

### <a name="implement-the-rest-of-the-dialog-and-add-it-to-the-bot"></a>Реализация остальной части диалога и добавление кода в бот

Во внешнем наборе диалогов, в который вы добавили компонентный диалог, такому диалогу присвоен идентификатор, с которым он был создан. Во внешнем наборе компонентный диалог выглядят как один диалог, похожий на обычные запросы.

Чтобы использовать компонентный диалог, добавьте его экземпляр в набор диалогов бота, то есть во внешний набор диалогов.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**DialogExtensions.cs**

В нашем примере для этого используется метод расширения `Run`, как показано ниже.

[!code-csharp[Run method](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/DialogExtensions.cs?range=13-24)]

**Bots\DialogBot.cs**

Метод `Run` вызывается из метода `OnMessageActivityAsync` бота.

[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/05.multi-turn-prompt/Bots/DialogBot.cs?range=42-48)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/userProfileDialog.js**

В этом примере мы добавили метод `run` в диалог профиля пользователя.

[!code-javascript[run method](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/dialogs/userProfileDialog.js?range=55-64)]

**bots/dialogBot.js**

Метод `run` вызывается из метода `onMessage` бота.

[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/05.multi-turn-prompt/bots/dialogBot.js?range=30-37)]

---

## <a name="to-test-the-bot"></a>Тестирование бота

1. Установите [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme), если вы этого еще не сделали.
1. Выполните этот пример на локальном компьютере.
1. Запустите эмулятор, подключитесь к боту и отправьте несколько сообщений, как показано ниже.

![Тестовый запуск диалога с несколькими запросами](../media/emulator-v4/multi-turn-prompt.png)

## <a name="additional-information"></a>Дополнительная информация

### <a name="how-cancellation-works-for-component-dialogs"></a>Как работает отмена для компонентных диалогов

Если вы вызовете метод _отмены всех диалогов_ из контекста компонентного диалога, этот метод отменит все диалоги во внутреннем стеке и завершит работу, передав управление следующему диалогу во внешнем стеке.

Если вы вызовете метод _отмены всех диалогов_ из внешнего контекста, компонентный диалог будет отменен вместе со всеми остальными диалогами во внешнем контексте.

Не забывайте об этом, организуя управление вложенными компонентными диалогами в боте.

## <a name="next-steps"></a>Дополнительная информация

Вы можете добавить в бот реакцию на другие входящие запросы, которая будет прерывать обычный поток беседы, например для получения справки или отмены действия.

> [!div class="nextstepaction"]
> [Обработка прерываний диалога пользователем](bot-builder-howto-handle-user-interrupt.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-multi-prompts-sample
[js-sample]: https://aka.ms/js-multi-prompts-sample

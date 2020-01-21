---
title: Обработка прерываний диалога пользователем — Служба Azure Bot
description: Узнайте, как обрабатывать прерывания со стороны пользователя и прямой поток беседы.
keywords: interrupt, interruptions, switching topic, break
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/05/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fca68b09632c89cd027d012a92fe99e186324f70
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75798378"
---
# <a name="handle-user-interruptions"></a>Обработка прерываний со стороны пользователя

[!INCLUDE[applies-to](../includes/applies-to.md)]

Обработка прерываний является важным аспектом для создания надежного бота. Пользователи не всегда будут строго соблюдать определенный вами процесс общения. Они могут задать вопрос в середине процесса или отменить процесс вместо завершения. В этом разделе рассматривается несколько распространенных способов обработки прерываний со стороны пользователя.

## <a name="prerequisites"></a>предварительные требования

- Понимание [основных принципов работы ботов][concept-basics], [управления состоянием][concept-state], [библиотек диалогов][concept-dialogs] и [повторного использования диалогов][component-dialogs].
- Копия примера бота на языке [**CSharp**][cs-sample], [**JavaScript**][js-sample] или [**Python**][python-sample].

## <a name="about-this-sample"></a>Об этом примере

Пример в этой статье моделирует работу бота для бронирования авиабилетов, который использует диалоги для получения от пользователя информации о нужном рейсе. В любой момент общения с ботом пользователь может выдать команду _help_ (помощь) или _cancel_ (отмена), что должно вызвать прерывание. Здесь мы будем использовать два вида прерываний.

- **На уровне шага**. Обработка на уровне шага отменяется, но сам диалог сохраняется в стеке вместе с предоставленной информацией. Следующий шаг будет продолжен с того же места, где мы остановились. 
- **На уровне диалога**. Полная отмена обработки, позволяющая боту начать работу сначала.

## <a name="define-and-implement-the-interruption-logic"></a>Определение и реализация логики прерывания

Сначала нам нужно определить и реализовать прерывания по командам _help_ и _help_.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы использовать диалоги, установите пакет NuGet **Microsoft.Bot.Builder.Dialogs**.

**Dialogs\CancelAndHelpDialog.cs**

Первым делом мы реализуем класс `CancelAndHelpDialog` для обработки прерываний со стороны пользователя.

[!code-csharp[Class signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=12)]

В классе `CancelAndHelpDialog` метод `OnContinueDialogAsync` вызывает метод `InerruptAsync`, чтобы проверить наличие прерываний со стороны пользователя. Если процесс прерывается, вызываются методы базового класса. В противном случае возвращается значение, полученное из `InterruptAsync`.

[!code-csharp[Overrides](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=22-31)]

Если пользователь вводит слово help, метод `InterrupAsync` отправляет сообщение и вызывает `DialogTurnResult (DialogTurnStatus.Waiting)`, чтобы обозначить ожидание ответа от пользователя в диалоге верхнего уровня. В этом случае процесс общения прерывается только на один шаг, а на следующем шаге продолжается с того места, где мы остановились.

Если пользователь вводит cancel, то вызывается метод `CancelAllDialogsAsync` в контексте внутреннего диалога. Это действие очищает стек диалогов и приводит к выходу из диалога с состоянием отмены и без результирующего значения. С точки зрения `MainDialog` (см. далее) все будет выглядеть так, как будто диалог бронирования завершился и вернул значение NULL. Этот равнозначно ситуации, когда пользователь отказался подтвердить бронирование.

[!code-csharp[Interrupt](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/CancelAndHelpDialog.cs?range=33-56)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы использовать диалоги, установите пакет npm **botbuilder-dialogs**.

**dialogs/cancelAndHelpDialog.js**

Первым делом мы реализуем класс `CancelAndHelpDialog` для обработки прерываний со стороны пользователя.

[!code-javascript[Class signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=11)]

В классе `CancelAndHelpDialog` метод `onContinueDialog` вызывает метод `interrupt`, чтобы проверить наличие прерываний со стороны пользователя. Если процесс прерывается, вызываются методы базового класса. В противном случае возвращается значение, полученное из `interrupt`.

[!code-javascript[Overrides](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js?range=12-18)]

Если пользователь вводит слово help, метод `interrupt` отправляет сообщение и возвращает объект `{ status: DialogTurnStatus.waiting }`, чтобы обозначить ожидание ответа от пользователя в диалоге верхнего уровня. В этом случае процесс общения прерывается только на один шаг, а на следующем шаге продолжается с того места, где мы остановились.

Если пользователь вводит cancel, то вызывается метод `cancelAllDialogs` в контексте внутреннего диалога. Это действие очищает стек диалогов и приводит к выходу из диалога с состоянием отмены и без результирующего значения. С точки зрения `MainDialog` (см. далее) все будет выглядеть так, как будто диалог бронирования завершился и вернул значение NULL. Этот равнозначно ситуации, когда пользователь отказался подтвердить бронирование.

[!code-javascript[Interrupt](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/cancelAndHelpDialog.js??range=20-39)]


## <a name="pythontabpython"></a>[Python](#tab/python)

Чтобы использовать диалоги, установите пакет `botbuilder-dialogs` и убедитесь, что пример файла `requirements.txt` содержит соответствующую ссылку, например `botbuilder-dialogs>=4.5.0`. Дополнительные сведения об установке пакетов см. в [файле сведений](https://github.com/microsoft/botbuilder-python) в репозитории примеров.
> [!NOTE]
> При выполнении команды `pip install botbuilder-dialogs` также установятся пакеты `botbuilder-core`, `botbulder-connector` и `botbuilder-schema`.

**dialogs/cancel-and-help-dialog.py**

Первым делом мы реализуем класс `CancelAndHelpDialog` для обработки прерываний со стороны пользователя.

[!code-python[class signature](~/../botbuilder-python/samples/python/13.core-bot/dialogs/cancel_and_help_dialog.py?range=14)]

В классе `CancelAndHelpDialog` метод `on_continue_dialog` вызывает метод `interrupt`, чтобы проверить наличие прерываний со стороны пользователя. Если процесс прерывается, вызываются методы базового класса. В противном случае возвращается значение, полученное из `InterruptAsync`.

[!code-python[dialog](~/../botbuilder-python/samples/python/13.core-bot/dialogs/cancel_and_help_dialog.py?range=18-23)]

Если пользователь вводит слово *help* или знак *?* , метод `interrupt` отправляет сообщение и вызывает `DialogTurnResult(DialogTurnStatus.Waiting)`, чтобы обозначить ожидание ответа от пользователя в диалоге в верхней части стека. В этом случае процесс общения прерывается только на один шаг, а на следующем шаге продолжается с того места, где мы остановились.

Если пользователь вводит *cancel* или *quit*, то вызывается метод `cancel_all_dialogs()` в контексте внутреннего диалога. Это действие очищает стек диалогов и приводит к выходу из диалога с состоянием отмены и без результирующего значения. С точки зрения `MainDialog` (см. далее) все будет выглядеть так, как будто диалог бронирования завершился и вернул значение NULL. Этот равнозначно ситуации, когда пользователь отказался подтвердить бронирование.

[!code-python[interrupt](~/../botbuilder-python/samples/python/13.core-bot/dialogs/cancel_and_help_dialog.py?range=25-47)]

---

## <a name="check-for-interruptions-each-turn"></a>Проверка прерываний на каждом шаге

Теперь мы знаем, как работает класс обработки прерываний. Давайте вернемся немного назад и рассмотрим, что происходит при получении ботом нового сообщения от пользователя.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Dialogs\MainDialog.cs**

При поступлении действия с новым сообщением бот выполняет `MainDialog`. В `MainDialog` у пользователя спрашивается, какая помощь тому требуется. Затем запускается `BookingDialog` в методе `MainDialog.ActStepAsync` с помощью вызова `BeginDialogAsync`, как показано ниже.

[!code-csharp[ActStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=58-101&highlight=6,26)]

После этого в методе `FinalStepAsync` класса `MainDialog` завершается диалог бронирования — оно считается завершенным или отмененным.

[!code-csharp[FinalStepAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=130-150)]

Код в `BookingDialog` здесь не показан, так как он не имеет прямого отношения к обработке прерывания. Он используется, чтобы запросить у пользователя сведения о бронировании. Вы можете найти этот код в файле **Dialogs\BookingDialogs.cs**.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**dialogs/mainDialog.js**

При поступлении действия с новым сообщением бот выполняет `MainDialog`. В `MainDialog` у пользователя спрашивается, какая помощь тому требуется. Затем запускается `bookingDialog` в методе `MainDialog.actStep` с помощью вызова `beginDialog`, как показано ниже.

[!code-javascript[Act step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=71-115&highlight=6,27)]

После этого в методе `finalStep` класса `MainDialog` завершается диалог бронирования — оно считается завершенным или отмененным.

[!code-javascript[Final step](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=142-159)]

Код в `BookingDialog` здесь не показан, так как он не имеет прямого отношения к обработке прерывания. Он используется, чтобы запросить у пользователя сведения о бронировании. Вы можете найти этот код в файле **dialogs/bookingDialogs.js**.

## <a name="pythontabpython"></a>[Python](#tab/python)

**dialogs/main_dialog.py**

При поступлении действия с новым сообщением бот выполняет `MainDialog`. В `MainDialog` у пользователя спрашивается, какая помощь тому требуется. Затем запускается `bookingDialog` в методе `MainDialog.act_step` с помощью вызова `begin_dialog`, как показано ниже.

[!code-python[act step](~/../botbuilder-python/samples/python/13.core-bot/dialogs/main_dialog.py?range=63-100&highlight=4-5,20)]

После этого в методе `final_step` класса `MainDialog` завершается диалог бронирования — оно считается завершенным или отмененным.

[!code-python[final step](~/../botbuilder-python/samples/python/13.core-bot/dialogs/main_dialog.py?range=102-118)]

---

## <a name="handle-unexpected-errors"></a>Обработка непредвиденных ошибок

Теперь мы разберемся с необработанными исключениями, которые могут возникнуть при работе.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**AdapterWithErrorHandler.cs**

В нашем примере обработчик `OnTurnError` в адаптере получает все исключения, создаваемые в соответствии с логикой шага в боте. Если создано исключение, обработчик удаляет состояние текущей беседы, чтобы бот не застрял в цикле ошибки из-за неправильного состояния.

[!code-csharp[AdapterWithErrorHandler](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/AdapterWithErrorHandler.cs?range=19-50)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

В нашем примере обработчик `onTurnError` в адаптере получает все исключения, создаваемые в соответствии с логикой шага в боте. Если создано исключение, обработчик удаляет состояние текущей беседы, чтобы бот не застрял в цикле ошибки из-за неправильного состояния.

[!code-javascript[AdapterWithErrorHandler](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=35-57)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**adapter_with_error_handler.py**

В нашем примере обработчик `on_error` в адаптере получает все исключения, создаваемые в соответствии с логикой шага в боте. Если создано исключение, обработчик удаляет состояние текущей беседы, чтобы бот не застрял в цикле ошибки из-за неправильного состояния.

[!code-python[adapter_with_error_handler](~/../botbuilder-python/samples/python/13.core-bot/adapter_with_error_handler.py?range=15-54)]

---

## <a name="register-services"></a>Регистрация служб

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Startup.cs.**

Наконец, в `Startup.cs` создается временный бот, то есть на каждом шаге создается новый экземпляр бота.

[!code-csharp[Add transient bot](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Startup.cs?range=43-44)]

Для справки ниже приведены определения классов, которые используются в описанном выше вызове для создания бота.

[!code-csharp[MainDialog signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Dialogs/MainDialog.cs?range=17)]
[!code-csharp[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogAndWelcomeBot.cs?range=16)]
[!code-csharp[DialogBot signature](~/../botbuilder-samples/samples/csharp_dotnetcore/13.core-bot/Bots/DialogBot.cs?range=18)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**index.js**

Наконец, в `index.js` создается бот.

[!code-javascript[Create bot](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/index.js?range=75-78)]

Для справки ниже приведены определения классов, которые используются в описанном выше вызове для создания бота.

[!code-javascript[MainDialog signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/dialogs/mainDialog.js?range=11)]
[!code-javascript[DialogAndWelcomeBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogAndWelcomeBot.js?range=8)]
[!code-javascript[DialogBot signature](~/../botbuilder-samples/samples/javascript_nodejs/13.core-bot/bots/dialogBot.js?range=6)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**app.py** Наконец, в `app.py` создается бот.

[!code-python[create bot](~/../botbuilder-python/samples/python/13.core-bot/app.py?range=44-48)]

Для справки ниже приведены определения классов, которые используются в вызове для создания бота.

[!code-python[main dialog](~/../botbuilder-python/samples/python/13.core-bot/dialogs/main_dialog.py?range=20)]

[!code-python[dialog and welcome](~/../botbuilder-python/samples/python/13.core-bot/bots/dialog_and_welcome_bot.py?range=21)]

[!code-python[dialog](~/../botbuilder-python/samples/python/13.core-bot/bots/dialog_bot.py?range=9)]

---

## <a name="to-test-the-bot"></a>Тестирование бота

1. Установите [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme), если вы этого еще не сделали.
1. Выполните этот пример на локальном компьютере.
1. Запустите эмулятор, подключитесь к боту и отправьте несколько сообщений, как показано ниже.

<!--![test dialog prompt sample](~/media/emulator-v4/test-dialog-prompt.png)-->

## <a name="additional-information"></a>Дополнительные сведения

- В [примере проверки подлинности](https://aka.ms/logout) показана обработка выхода пользователя. Там применен тот же шаблон, что и для обработки прерываний в этом примере.

- Вместо бездействия следует отправить ответ по умолчанию, чтобы пользователь не оставался в недоумении от молчания бота. Ответ по умолчанию должен сообщать, какие команды понимает бот, чтобы пользователь мог вернуться в нужное русло.

- В любой точке шага свойство контекста _responded_ указывает, отправлял ли бот пользователю сообщение на этом шаге. Следите за тем, чтобы бот обязательно отправил пользователю хоть что-то до завершения шага, хотя бы просто подтверждение ввода.

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[using-luis]: bot-builder-howto-v4-luis.md
[using-qna]: bot-builder-howto-qna.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: https://aka.ms/cs-core-sample
[js-sample]: https://aka.ms/js-core-sample
[python-sample]: https://aka.ms/bot-core-python-sample-code

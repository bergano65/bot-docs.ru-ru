---
title: Использование библиотеки диалогов для сбора данных, вводимых пользователем | Документация Майкрософт
description: Узнайте, как запросить у пользователя входные данные, используя библиотеку диалогов из пакета SDK Bot Builder.
keywords: запросы, диалоговые окна, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, повторный запрос, проверка
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/02/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 150d5f0a68d897ac278026a7cf36609aca05bb80
ms.sourcegitcommit: 984705927561cc8d6a84f811ff24c8c71b71c76b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/02/2018
ms.locfileid: "50965722"
---
# <a name="use-dialog-library-to-gather-user-input"></a>Использование библиотеки диалогов для сбора данных, вводимых пользователем

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Сбор данных путем размещения вопросов — это один из основных способов взаимодействия между ботом и пользователями. Для его непосредственного выполнения можно использовать метод _отправки действия_ объекта в [контексте реплики](~/v4sdk/bot-builder-basics.md#defining-a-turn), а затем обработать следующее входящее сообщение как ответ. Но пакет SDK Bot Builder предоставляет библиотеку [диалогов](bot-builder-concept-dialog.md), в которой содержатся методы, которые упрощают постановку вопросов и гарантируют, что ответы соответствуют определенному типу данных или правилам настраиваемой проверки. В этой статье объясняется, как достичь этого с помощью объектов запроса для получения входных данных от пользователя.

Из этой статьи вы узнаете, как создавать запросы и вызывать их из диалога.
Инструкции по получению входных данных от пользователя без использования диалогов см. в [этой статье](bot-builder-primitive-prompts.md).
Общие сведения об использовании диалогов см. в статье [Управление простым процессом общения с помощью диалогов](bot-builder-dialog-manage-conversation-flow.md).

## <a name="prompt-types"></a>Типы запросов

Запросы данных, по сути, являются диалогами из двух этапов. На первом запрашиваются входные данные, а на втором возвращается допустимое значение или цикл запроса запускается заново.

Библиотека диалогов содержит несколько простых запросов, каждый из которых возвращает разные типы ответов.

| prompt | ОПИСАНИЕ | Результаты |
|:----|:----|:----|
| _Запрос вложений_ | Предложение передать одно или несколько вложений, например документов или изображений. | Коллекция объектов _вложений_. |
| _Запрос выбора_ | Предложение выбрать один параметр из набора. | Объект _найденного выбора_. |
| _Запрос подтверждения_ | Запрашивается подтверждение. | Логическое значение. |
| _Запрос даты и времени_ | Предложение ввести дату и (или) время. | Коллекция объектов _разрешения даты и времени_. |
| _Запрос числа_ | Предложение ввести число. | Числовое значение. |
| _Запрос текста_ | Предложение ввести входные данные в простом текстовом виде. | Строка. |

Библиотека также включает _запрос OAuth_ для получения _маркера OAuth_ для доступа к другому приложению от имени пользователя. Дополнительные сведения об аутентификации см. в статье [о добавлении аутентификации в код бота](bot-builder-authentication.md).

Базовые запросы могут интерпретировать входные данные на естественном языке, например числа "ten" (десять) или "a dozen" (дюжина) и указания времени "tomorrow" (завтра) или "Friday at 10am" (в 10 вечера в пятницу).

## <a name="using-prompts"></a>Использование запросов

Диалог может использовать запрос только в том случае, если они находятся в одном наборе диалогов.

1. Определите для диалога метод доступа к свойству состояния.
1. Создайте набор диалогов.
1. Создайте собственные запросы и добавьте их в набор диалогов.
1. Создайте диалог, в котором пользователю будут предлагаться ваши запросы, и добавьте его к набору диалогов.
1. Добавьте в этот диалог вызовы запросов и получение результатов.

В этой статье объясняется, как создавать запросы и вызывать их из каскадного диалога.
Дополнительные сведения о диалогах в целом см. в статье [о библиотеке диалогов](bot-builder-concept-dialog.md).
Статья [Управление простым процессом общения с помощью диалогов](bot-builder-dialog-manage-conversation-flow.md) содержит полное описание реализации бота с диалогами и запросами.

Вы можете использовать один запрос в нескольких этапах одного диалога и (или) в нескольких разных диалогах из одного набора диалогов.
Пользовательская проверка связывается в конкретным запросом во время инициализации.
Это означает, что если для одного типа запросов нужны разные проверки, придется создать несколько экземпляров этого типа запросов со своим кодом проверок.

### <a name="create-a-prompt"></a>Создание запроса

Чтобы запросить ввод данных от пользователя, определите запрос на основе одного из встроенных классов, например _текстового запроса_, и добавьте этот запрос в набор диалогов.

* Запрос имеет фиксированный идентификатор. (Все идентификаторы должны быть уникальными в пределах набора диалогов.)
* В запросе может применяться пользовательский проверяющий элемент управления. (См. статью о [пользовательской проверке](#custom-validation).)
* Для некоторых запросов можно указать _языковой стандарт по умолчанию_.

Как правило, запросы и диалоги создаются и добавляются в набор диалогов на этапе инициализации бота. Это позволяет набору диалогов разрешать идентификаторы запросов при поступлении входных данных от пользователя в бот.

Например, следующий код позволяет создать два текстовых запроса и добавить их к существующему набору диалогов. Второй текстовый запрос ссылается на метод проверки, который здесь не показан.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Здесь `_dialogs` содержит существующий набор диалогов, а `NameValidator` представляет собой метод проверки.

```csharp
_dialogs.Add(new TextPrompt("nickNamePrompt"));
_dialogs.Add(new TextPrompt("namePrompt", NameValidator));
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Здесь `this.dialogs` содержит существующий набор диалогов, а `NameValidator` представляет собой функцию проверки.

```javascript
this.dialogs.add(new TextPrompt('nickNamePrompt'));
this.dialogs.add(new TextPrompt('namePrompt', NameValidator));
```

---

#### <a name="locales"></a>Locales

Языковой стандарт используется для определения поведения, зависящего от языка, в запросах **выбора**, **подтверждения**, **даты и времени** и **числа**. В отношении любых входных данных, поступающих от пользователя, действуют указанные ниже правила.

* Если канал передает _языковой стандарт_ в сообщении от пользователя, то используется именно он.
* Если для запроса задан _языковой стандарт по умолчанию_ при вызове конструктора строки или позднее, то используется именно он.
* В противном случае в качестве языкового стандарта используется английский язык ("en-us").

> [!NOTE]
> Языковой стандарт определяется кодом ISO 639 из 2, 3 или 4 символов, который указывает определенный язык или языковую группу.

### <a name="call-a-prompt-from-a-waterfall-dialog"></a>Вызов запроса из каскадного диалога

Добавив запрос, вызовите его на одном из этапов каскадного диалога и получите результат запроса на следующем этапе.
Чтобы вызвать запрос на этапе каскадного диалога, вызовите метод _prompt_ из объекта _контекста этапа каскадного диалога_. Первым параметром является идентификатор используемого запроса, а второй параметр содержит аргументы запроса, например текст обращения к пользователю для ввода данных.

Предположим, что пользователь взаимодействует с ботом, в котором активен каскадный диалог с запросом на следующем этапе.

1. Когда пользователь отправляет боту сообщение, выполняются описанные ниже действия.
   1. Обработчик реплик бота создает контекст диалога и вызывает его метод _continue_.
   1. Контроль переходит к следующему этапу в активном диалоге, то есть к каскадному диалогу в нашем примере.
   1. На этом этапе вызывается метод _prompt_ из контекста этапа каскадного диалога, чтобы запросить входные данные от пользователя.
   1. Контекст этапа каскадного диалога помещает запрос в стек и запускает его.
   1. Пользователю отправляется действие запроса на входные данные.
1. Когда пользователь отправляет в бот следующее сообщение, выполняются описанные ниже действия.
   1. Обработчик реплик бота создает контекст диалога и вызывает его метод _continue_.
   1. Контроль переходит к следующему этапу в активном диалоге, то есть ко второй реплике запроса в нашем примере.
   1. Запрос проверяет введенные пользователем данные.
      * Если входные данные имеют недопустимый формат, запрос повторяется и пользователю снова предлагается ввести данные. В этом случае при следующей реплике выполняется тот же набор действий.
      * В противном случае запрос завершается и возвращает объект _результата реплики диалога_ в родительский диалог. Управление передается следующему этапу каскадного диалога. При этом результат запроса помещается в свойство _result_ контекста этапа каскадного диалога.

<!--
> [!NOTE]
> A waterfall step delegate takes a _waterfall step context_ parameter and returns a _dialog turn result_.
> A prompt's result is contained within the prompt's return value (a dialog turn result object) when it ends.
> The waterfall dialog provides the return value in the waterfall step context parameter when it calls the next waterfall step.
-->

Когда выполнение запроса завершается, свойство _result_ контекста этапа каскадного диалога принимает значение, возвращенное запросом.

В этом примере показаны элементы двух последовательных этапов каскадного диалога. На первом пользователю отправляется запрос на его имя. На втором этапе извлекается возвращаемое значение этого запроса.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Здесь `name` содержит идентификатор текстового запроса, а `NameStepAsync` и `GreetingStepAsync` представляют собой делегаты двух последовательных этапов в каскадном диалоге.

```csharp
private static async Task<DialogTurnResult> NameStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    // Prompt for the user's name.
    return await stepContext.PromptAsync(
        "name",
         new PromptOptions { Prompt = MessageFactory.Text("Please enter your name.") },
         cancellationToken);
}

private static async Task<DialogTurnResult> GreetingStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Get the user's name from the prompt result.
    string name = (string)stepContext.Result;
    await stepContext.Context.SendActivityAsync(
        MessageFactory.Text($"Pleased to meet you, {name}."),
         cancellationToken);

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Здесь `name` содержит идентификатор текстового запроса, а `nameStep` и `greetingStep` представляют собой функции двух последовательных этапов в каскадном диалоге.

```javascript
async nameStep(step) {
    // ...

    return await step.prompt('name', 'Please enter your name.');
}

async greetingStep(step) {
    // Get the user's name from the prompt result.
    const name = step.result;
    await step.context.sendActivity(`Pleased to meet you, ${name}.`);

    // ...
}
```

---

### <a name="call-a-prompt-from-the-bots-turn-handler"></a>Вызов запроса из обработчика реплик бота

Вы можете напрямую вызвать запрос из обработчика реплик, используя метод _prompt_ из контекста диалога.
После этого потребуется вызвать для следующей реплики метод _continue dialog_ из контекста диалога и проверить его возвращаемое значение, которое содержит объект _результата реплики запроса_. Вариант такой реализации вы найдете в примере с проверкой запроса ([C#](https://aka.ms/cs-prompt-validation-sample) | [JS](https://aka.ms/js-prompt-validation-sample)), а другой возможный подход описан в статье [Создание собственных запросов на сбор данных, вводимых пользователем](bot-builder-primitive-prompts.md).

## <a name="prompt-options"></a>Параметры запроса

Второй параметр для метода _prompt_ принимает объект _аргументов запроса_ с указанными ниже свойствами.

| Свойство | ОПИСАНИЕ |
| :--- | :--- |
| _prompt_ | Исходное действие, отправляемое пользователю для получения входных данных. |
| _retry prompt_ | Действие, отправляемое пользователю при несоблюдении формата для первого введенного значения. |
| _choices_ | Список вариантов, из которых может выбрать пользователь при использовании запроса выбора. |

Как правило, аргументы запроса и повторной попытки являются действиями, но в разных языках программирования они обрабатываются немного по-разному.

Обязательно укажите исходное действие запроса, которое будет отправляться пользователю.

Указывать строку повторного запроса рекомендуется в тех случаях, когда высока вероятность того, что проверка входных данных не будет пройдена из-за невозможности проанализировать значение (например, строка "tomorrow" для числового параметра) или несоответствия значения критериям проверки. Если строка повторного запроса не указана, запрос в таких случаях будет повторно предоставлять пользователю исходное действие запроса для получения входных данных.

Для запроса выбора обязательно нужно указывать список доступных вариантов.

Этот пример демонстрирует запрос выбора со всеми тремя аргументами. Метод _favorite color_ (любимый цвет) используется в качестве этапа в каскадном диалоге, а набор диалогов из нашего примера содержит и каскадный диалог, и запрос выбора с идентификатором `colorChoice`.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task<DialogTurnResult> FavoriteColorAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // ...

    return await stepContext.PromptAsync(
        "colorChoice",
        new PromptOptions {
            Prompt = MessageFactory.Text("Please choose a color."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a color from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "blue", "green", "red" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Пакет SDK для JavaScript позволяет ввести строковые значения для свойств `prompt` и `retryPrompt`. Запрос автоматически преобразует их в действия сообщений.

```javascript
async favoriteColor(step) {
    // ...

    return await step.prompt('colorChoice', {
        prompt: 'Please choose a color:',
        retryPrompt: 'Sorry, please choose a color from the list.',
        choices: [ 'red', 'green', 'blue' ]
    });
}
```

---

## <a name="custom-validation"></a>Пользовательская проверка

Можно проверить ответ на запрос, прежде чем значение будет возвращено на следующем этапе **каскада**. Функция проверяющего элемента управления принимает параметр _контекста проверяющего элемента управления запроса_ и возвращает логическое значение, которое подтверждает успешное прохождение проверки для входных данных.

Контекст проверяющего элемента управления запроса поддерживает указанные ниже свойства.

| Свойство | ОПИСАНИЕ |
| :--- | :--- |
| _Контекст_ | Текущий контекст реплики для бота. |
| _Recognized_ | _Результат распознавателя запроса_, который содержит сведения о входных данных после обработки распознавателем. |

Результат распознавателя запроса имеет описанные ниже свойства.

| Свойство | ОПИСАНИЕ |
| :--- | :--- |
| _Успешно_ | Указывает, смог ли распознаватель успешно выполнить синтаксический анализ входных данных. |
| _Значение_ | Возвращаемое распознавателем значение. При необходимости это значение можно изменять в коде проверки. |

### <a name="setup"></a>Настройка

Перед добавлением кода проверки нужно выполнить некоторую настройку.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В файле **.cs** для вашего бота определите внутренний класс для сведений о резервировании.

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Date { get; set; }
}
```

В файл **BotAccessors.cs** добавьте метод доступа к свойству состояния для данных о резервировании.

```csharp
public class BotAccessors
{
    public BotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "BotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "BotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<ReservationBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

В файле **Startup.cs** измените `ConfigureServices` для настройки методов доступа.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        // ...

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(BotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<ReservationBot.Reservation>(BotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Для JavaScript нет необходимости изменять код службы HTTP. Поэтому файл **index.js** мы сохраняем как есть.

В файле **bot.js** обновите инструкции require и добавьте идентификаторы для методов доступа к свойству состояния.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, DialogTurnStatus } = require('botbuilder-dialogs');

// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';
```

---

В файле бота добавьте идентификаторы нужных диалогов и запросов.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="define-the-prompts-and-dialogs"></a>Определение запросов и диалогов

В коде конструктора бота создайте набор диалогов, добавьте запросы и диалог резервирования.
При создании запросов добавляется пользовательская проверка. Теперь мы реализуем функции для этой проверки.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public ReservationBot(BotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...
    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };
    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, partySizeValidator));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-validation-code"></a>Реализация кода проверки

Создайте проверяющий элемент управления для размера компании. Установим возможность резервирования столиков только для компаний от 6 до 20 человек.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>Validates whether the party size is appropriate to make a reservation.</summary>
/// <param name="promptContext">The validation context.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>Reservations can be made for groups of 6 to 20 people.
/// If the task is successful, the result indicates whether the input was valid.</remarks>
private async Task<bool> PartySizeValidatorAsync(
    PromptValidatorContext<int> promptContext,
    CancellationToken cancellationToken)
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the number of people in your party.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether the party size is appropriate.
    int size = promptContext.Recognized.Value;
    if (size < 6 || size > 20)
    {
        await promptContext.Context.SendActivityAsync(
            "Sorry, we can only take reservations for parties of 6 to 20.",
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async partySizeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }
    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < 6 || size > 20) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of 6 to 20.');
        return false;
    }

    return true;
}
```

---

Запрос даты и времени возвращает список или массив всех возможных _разрешений даты и времени_, которые соответствуют входным данным пользователя. Например, значение 9:00 может означать 9 AM или 9 PM, а текст Sunday (воскресенье) также является неоднозначным. Кроме того, разрешение даты и времени может содержать дату, время, дату и время или диапазон времен. Запрос даты и времени использует для синтаксического анализа входных данных пользователя [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text).

Создайте проверяющий элемент управления для даты резервирования. Для времени резервирования мы установим ограничение — не менее чем через час от текущего времени. Теперь мы сохраняем первое разрешение, которое соответствуют заданным критериям, и удаляем все остальные.

Этот код проверки не является исчерпывающим. Он лучше всего подходит для входных данных, которые преобразуются в дату и время. Он демонстрирует некоторые возможности проверки для даты и времени, а код для конкретной реализации зависит от характера запрашиваемой у пользователя информации.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>Validates whether the reservation date is appropriate.</summary>
/// <param name="promptContext">The validation context.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>Reservations must be made at least an hour in advance.
/// If the task is successful, the result indicates whether the input was valid.</remarks>
private async Task<bool> DateValidatorAsync(
    PromptValidatorContext<IList<DateTimeResolution>> promptContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Check whether the input could be recognized as an integer.
    if (!promptContext.Recognized.Succeeded)
    {
        await promptContext.Context.SendActivityAsync(
            "I'm sorry, I do not understand. Please enter the date or time for your reservation.",
            cancellationToken: cancellationToken);
        return false;
    }

    // Check whether any of the recognized date-times are appropriate,
    // and if so, return the first appropriate date-time.
    DateTime earliest = DateTime.Now.AddHours(1.0);
    DateTimeResolution value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out DateTime time) && DateTime.Compare(earliest,time) <= 0);
    if (value != null)
    {
        promptContext.Recognized.Value.Clear();
        promptContext.Recognized.Value.Add(value);
        return true;
    }

    await promptContext.Context.SendActivityAsync(
            "I'm sorry, we can't take reservations earlier than an hour from now.",
            cancellationToken: cancellationToken);
    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async dateValidator(promptContext) {
// Check whether the input could be recognized as an integer.
if (!promptContext.recognized.succeeded) {
    await promptContext.context.sendActivity(
        "I'm sorry, I do not understand. Please enter the date or time for your reservation.");
    return false;
}

// Check whether any of the recognized date-times are appropriate,
// and if so, return the first appropriate date-time.
const earliest = Date.now() + (60 * 60 * 1000);
let value = null;
promptContext.recognized.value.forEach(candidate => {
    // TODO: update validation to account for time vs date vs date-time vs range.
    const time = new Date(candidate.value || candidate.start);
    if (earliest < time.getTime()) {
        value = candidate;
    }
});
if (value) {
    promptContext.recognized.value = [value];
    return true;
}

await promptContext.context.sendActivity(
    "I'm sorry, we can't take reservations earlier than an hour from now.");
return false;
}
```

---

### <a name="implement-the-dialog-steps"></a>Реализация этапов диалога

Примените запросы, которые мы добавили к набору диалогов. При создании запросов в конструкторе ботов мы добавили к ним некоторые проверки. При первом запросе входных данных пользователю отправляется действие _prompt_ из указанных аргументов запроса. Если проверка не пройдена, бот отправляет действие _retry prompt_, которое предлагает пользователю ввести другие входные данные.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
/// <summary>First step of the main dialog: prompt for party size.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

/// <summary>Second step of the main dialog: record the party size and prompt for the
/// reservation date.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> PromptForReservationDateAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        ReservationDatePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Great. When will the reservation be for?"),
            RetryPrompt = MessageFactory.Text("What time should we make your reservation for?"),
        },
        cancellationToken);
}

/// <summary>Third step of the main dialog: return the collected party size and reservation date.</summary>
/// <param name="stepContext">The context for the waterfall step.</param>
/// <param name="cancellationToken">A cancellation token that can be used by other objects
/// or threads to receive notice of cancellation.</param>
/// <returns>A task that represents the work queued to execute.</returns>
/// <remarks>If the task is successful, the result contains information from this step.</remarks>
private async Task<DialogTurnResult> AcknowledgeReservationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Retrieve the reservation date.
    DateTimeResolution resolution = (stepContext.Result as IList<DateTimeResolution>).First();
    string time = resolution.Value ?? resolution.Start;

    // Send an acknowledgement to the user.
    await stepContext.Context.SendActivityAsync(
        "Thank you. We will confirm your reservation shortly.",
        cancellationToken: cancellationToken);

    // Return the collected information to the parent context.
    Reservation reservation = new Reservation
    {
        Date = time,
        Size = (int)stepContext.Values["size"],
    };
    return await stepContext.EndDialogAsync(reservation, cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForReservationDate(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for the reservation date. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        RESERVATION_DATE_PROMPT, {
            prompt: 'Great. When will the reservation be for?',
            retryPrompt: 'What time should we make your reservation for?'
        });
}

async acknowledgeReservation(stepContext) {
    // Retrieve the reservation date.
    const resolution = stepContext.result[0];
    const time = resolution.value || resolution.start;

    // Send an acknowledgement to the user.
    await stepContext.context.sendActivity(
        'Thank you. We will confirm your reservation shortly.');

    // Return the collected information to the parent context.
    return await stepContext.endDialog({ date: time, size: stepContext.values.size });
}
```

---

### <a name="update-the-turn-handler"></a>Обновление обработчика реплик

Обновите обработчик реплик бота, чтобы он запускал диалог и принимал значение, которое возвращает по завершении.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            Reservation reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext, () => null, cancellationToken);

            // Generate a dialog context for our dialog set.
            DialogContext dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

            if (dc.ActiveDialog is null)
            {
                // If there is no active dialog, check whether we have a reservation yet.
                if (reservation is null)
                {
                    // If not, start the dialog.
                    await dc.BeginDialogAsync(ReservationDialog, null, cancellationToken);
                }
                else
                {
                    // Otherwise, send a status message.
                    await turnContext.SendActivityAsync(
                        $"We'll see you {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                DialogTurnResult dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.Status is DialogTurnStatus.Complete)
                {
                    reservation = (Reservation)dialogTurnResult.Result;
                    await _accessors.ReservationAccessor.SetAsync(
                        turnContext,
                        reservation,
                        cancellationToken);

                    // Send a confirmation message to the user.
                    await turnContext.SendActivityAsync(
                        $"Your party of {reservation.Size} is confirmed for {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(turnContext, false, cancellationToken);
            break;

        // Handle other incoming activity types as appropriate to your bot.
        default:
            await turnContext.SendActivityAsync($"{turnContext.Activity.Type} event detected");
            break;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async onTurn(turnContext) {
    switch (turnContext.activity.type) {
        case ActivityTypes.Message:
            // Get the current reservation info from state.
            const reservation = await this.reservationAccessor.get(turnContext, null);

            // Generate a dialog context for our dialog set.
            const dc = await this.dialogSet.createContext(turnContext);

            if (!dc.activeDialog) {
                // If there is no active dialog, check whether we have a reservation yet.
                if (!reservation) {
                    // If not, start the dialog.
                    await dc.beginDialog(RESERVATION_DIALOG);
                }
                else {
                    // Otherwise, send a status message.
                    await turnContext.sendActivity(
                        `We'll see you ${reservation.date}.`);
                }
            }
            else {
                // Continue the dialog.
                const dialogTurnResult = await dc.continueDialog();

                // If the dialog completed this turn, record the reservation info.
                if (dialogTurnResult.status === DialogTurnStatus.complete) {
                    await this.reservationAccessor.set(
                        turnContext,
                        dialogTurnResult.result);

                    // Send a confirmation message to the user.
                    await turnContext.sendActivity(
                        `Your party of ${dialogTurnResult.result.size} is ` +
                        `confirmed for ${dialogTurnResult.result.date}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        default:
            break;
    }
}
```

---

Дополнительные примеры можно найти в [примерах репозитория](https://aka.ms/bot-samples-readme).

Можно использовать аналогичные методы для проверки ответов на запросы любого типа.

## <a name="handling-prompt-results"></a>Обработка результатов запроса

Действия, выполняемые для результата запроса, зависят от целей получения этой информации от пользователя. Доступные параметры: 

* Используйте сведения для управления потоком диалога с учетом подтверждения или выбора пользователя.
* Кэшируйте информацию в состоянии диалога, например установите свойство _values_ для контекста этапа каскадного диалога, а затем возвращайте собранные сведения при завершении диалога.
* Сохраните информацию в сведениях о состоянии бота. Чтобы использовать этот вариант, диалог должен иметь доступ к методам доступа к свойству состояния бота.

Статьи и примеры по этим сценариям перечислены в разделе дополнительных ресурсов.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление простым процессом общения](bot-builder-dialog-manage-conversation-flow.md)
* [Управление сложным процессом общения](bot-builder-dialog-manage-complex-conversation-flow.md)
* [Создание встроенного набора диалогов](bot-builder-compositcontrol.md)
* [Хранение данных в диалогах](bot-builder-tutorial-persist-user-inputs.md)
* Пример **запроса с несколькими репликами** (для [C#](https://aka.ms/cs-multi-prompts-sample) | [JS](https://aka.ms/js-multi-prompts-sample))

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знаете, как запрашивать у пользователя входные данные, улучшите код бота и пользовательский интерфейс, управляя разнообразными последовательностями общения с помощью диалоговых окон.

> [!div class="nextstepaction"]
> [Управление сложным процессом общения](bot-builder-dialog-manage-complex-conversation-flow.md)

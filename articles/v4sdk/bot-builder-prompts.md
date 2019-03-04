---
title: Сбор данных, которые вводит пользователь, с помощью диалогового окна | Документация Майкрософт
description: Узнайте, как запросить у пользователя входные данные, используя библиотеку диалогов из пакета SDK Bot Framework.
keywords: prompts, prompt, user input, dialogs, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, reprompt, validation
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/19/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 68c01b0f12790393fe0ee7ae0bd28addf2d26ae7
ms.sourcegitcommit: 05ddade244874b7d6e2fc91745131b99cc58b0d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/21/2019
ms.locfileid: "56591125"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>Сбор данных, которые вводит пользователь, с помощью диалогового окна

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Сбор данных путем размещения вопросов — это один из основных способов взаимодействия между ботом и пользователями. Библиотека *Dialogs* упрощает отправку вопросов и проверку ответов, чтобы убедиться в правильности типов данных и (или) выполнении пользовательских правил проверки. В этой статье подробно описывается создание и вызов запросов из каскадного диалога.

## <a name="prerequisites"></a>Предварительные требования

- Код в этой статье основан на примере **DialogPromptBot**. Вам потребуется копия примера для [C#](https://aka.ms/dialog-prompt-cs) или [JS](https://aka.ms/dialog-prompt-js).
- Необходимо базовое понимание [библиотеки Dialogs](bot-builder-concept-dialog.md) и [управления беседами](bot-builder-dialog-manage-conversation-flow.md).
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator) для целей тестирования.

## <a name="using-prompts"></a>Использование запросов

Диалог может использовать запрос только в том случае, если они находятся в одном наборе диалогов. Вы можете использовать один запрос в нескольких этапах одного диалога и (или) в нескольких разных диалогах из одного набора диалогов. Пользовательская проверка связывается в конкретным запросом во время инициализации. Чтобы использовать для одного типа запросов разные правила проверки, придется создать несколько экземпляров этого типа запросов со своим кодом проверки.

### <a name="define-a-state-property-accessor-for-the-dialog-state"></a>Определите для диалога метод доступа к свойству состояния.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Пример запроса для диалога, который используется в этой статье, запрашивает у пользователя сведения о резервировании. Чтобы отслеживать количество посетителей и дату, мы используем внутренний класс резервирования, определенный в файле DialogPromptBot.cs.

```csharp
public class Reservation
{
    public int Size { get; set; }

    public string Location { get; set; }

    public string Date { get; set; }
}
```

Далее мы добавим метод доступа к свойству состояния для данных о резервировании.

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState
            ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; }
        = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; }
        = "DialogPromptBotAccessors.Reservation";

    public IStatePropertyAccessor<DialogState> DialogStateAccessor { get; set; }
    public IStatePropertyAccessor<DialogPromptBot.Reservation> ReservationAccessor { get; set; }

    public ConversationState ConversationState { get; }
}
```

В файле Startup.cs мы изменяем метод `ConfigureServices` для настройки методов доступа.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    IStorage dataStore = new MemoryStorage();
    var conversationState = new ConversationState(dataStore);

    // Create and register state accesssors.
    services.AddSingleton<DialogPromptBotAccessors>(sp =>
    {
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new DialogPromptBotAccessors(conversationState)
        {
            DialogStateAccessor =
                conversationState.CreateProperty<DialogState>(
                    DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor =
                conversationState.CreateProperty<DialogPromptBot.Reservation>(
                    DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });

    // ...
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Для JavaScript нет необходимости изменять код службы HTTP, поэтому файл index.js мы оставим без изменений.

В файл bot.js, мы добавляем инструкции `require`, которые потребуются для работы бота с запросом для диалога.

```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus }
    = require('botbuilder-dialogs');
```

Добавьте идентификаторы для методов доступа к свойству состояния, диалогов и запросов.

```javascript
// Define identifiers for our state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for our dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const SIZE_RANGE_PROMPT = 'rangePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>Создание набора диалогов и запросов

Как правило, запросы и диалоги создаются и добавляются в набор диалогов на этапе инициализации бота. Это позволяет набору диалогов разрешать идентификаторы запросов при поступлении входных данных от пользователя в бот.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В классе `DialogPromptBot` определите идентификаторы диалогов, запросы и значения для отслеживания в диалогах.

```csharp
// Define identifiers for our dialogs and prompts.
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partySizePrompt";
private const string SizeRangePrompt = "sizeRangePrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

// Define keys for tracked values within the dialog.
private const string LocationKey = "location";
private const string PartySizeKey = "partySize";
```

В конструкторе бота создайте набор диалогов, добавьте запросы и диалог резервирования. На этапе создания запросов мы определим пользовательские проверки, а функции для них реализуем позже.

```csharp
private readonly DialogSet _dialogSet;
private readonly DialogPromptBotAccessors _accessors;

// ...

// Initializes a new instance of the <see cref="DialogPromptBot"/> class.
public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
    // ...

    _accessors = accessors ?? throw new System.ArgumentNullException(nameof(accessors));

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);

    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
    _dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
    _dialogSet.Add(new ChoicePrompt(LocationPrompt));
    _dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));

    // Define the steps of the waterfall dialog and add it to the set.
    WaterfallStep[] steps = new WaterfallStep[]
    {
        PromptForPartySizeAsync,
        PromptForLocationAsync,
        PromptForReservationDateAsync,
        AcknowledgeReservationAsync,
    };

    _dialogSet.Add(new WaterfallDialog(ReservationDialog, steps));
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Создайте свойства для доступа к состоянию в конструкторе.
Затем создайте набор диалогов и добавьте запросы, в том числе пользовательскую проверку.
Затем определите шаги каскадного диалога и добавьте его в набор.

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    // See https://aka.ms/about-bot-state-accessors to learn more about the bot state and state accessors.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;

    // Create the dialog set and add the prompts, including custom validation.
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
    this.dialogSet.add(new ChoicePrompt(LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));

    // Define the steps of the waterfall dialog and add it to the set.
    this.dialogSet.add(new WaterfallDialog(RESERVATION_DIALOG, [
        this.promptForPartySize.bind(this),
        this.promptForLocation.bind(this),
        this.promptForReservationDate.bind(this),
        this.acknowledgeReservation.bind(this),
    ]));
}
```

---

### <a name="implement-dialog-steps"></a>Реализация шагов диалога

В главном файле бота мы реализуем каждый из шагов каскадного диалога. Добавив запрос, мы вызовем его на одном из шагов каскадного диалога и получим результат запроса на следующем шаге. Чтобы вызвать запрос на этапе каскадного диалога, вызовите метод _prompt_ из объекта _контекста этапа каскадного диалога_. Первым параметром является идентификатор используемого запроса, а второй параметр содержит аргументы запроса, например текст обращения к пользователю для ввода данных.

Эти методы демонстрируют следующие действия:

- вызов запроса из каскадного шага, в том числе с передачей _вариантов выбора для запроса_;
- предоставление дополнительных параметров в пользовательский проверяющий элемент управления через свойство _validations_;
- предоставление вариантов выбора для запроса при помощи свойства _choices_.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В файле **DialogPromptBot.cs** мы реализуем шаги каскадного диалога.

Здесь представлены только первые два шага каскада: `PromptForPartySizeAsync` и `PromptForLocationAsync`.

```csharp
// First step of the main dialog: prompt for party size.
private async Task<DialogTurnResult> PromptForPartySizeAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.PromptAsync(
        SizeRangePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
            Validations = new Range { Min = 3, Max = 8 },
        },
        cancellationToken);
}

// Second step of the main dialog: prompt for location.
private async Task<DialogTurnResult> PromptForLocationAsync(
    WaterfallStepContext stepContext,
    CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    var size = (int)stepContext.Result;
    stepContext.Values[PartySizeKey] = size;

    // Prompt for the location.
    return await stepContext.PromptAsync(
        LocationPrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В файле **bot.js** мы реализуем шаги каскадного диалога.

Здесь представлены только первые два шага каскада: `promptForPartySize` и `promptForLocation`.

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        SIZE_RANGE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?',
            validations: { min: 3, max: 8 },
        });
}

async promptForLocation(stepContext) {
    // Record the party size information in the current dialog state.
    stepContext.values.size = stepContext.result;

    // Prompt for location.
    return await stepContext.prompt(LOCATION_PROMPT, {
        prompt: 'Please choose a location.',
        retryPrompt: 'Sorry, please choose a location from the list.',
        choices: ['Redmond', 'Bellevue', 'Seattle'],
    });
}
```

---

Второй параметр для метода _prompt_ принимает объект _аргументов запроса_ с указанными ниже свойствами.

| Свойство | ОПИСАНИЕ |
| :--- | :--- |
| _prompt_ | Исходное действие, отправляемое пользователю для получения входных данных. |
| _retry prompt_ | Действие, отправляемое пользователю при несоблюдении формата для первого введенного значения. |
| _choices_ | Список вариантов, из которых может выбрать пользователь при использовании запроса выбора. |
| _validations_ | Дополнительные параметры для пользовательского проверяющего элемента управления. |

Как правило, аргументы запроса и повторной попытки являются действиями, но в разных языках программирования они обрабатываются немного по-разному.

Обязательно укажите исходное действие запроса, которое будет отправляться пользователю.

Указывать строку повторного запроса полезно в тех случаях, когда высока вероятность того, что проверка входных данных не будет пройдена из-за невозможности проанализировать значение (например, строка tomorrow для числового параметра) или несоответствия введенного значения критериям проверки. Если строка повторного запроса не указана, запрос в таких случаях будет повторно предоставлять пользователю исходное действие запроса для получения входных данных.

Для запроса выбора обязательно нужно указывать список доступных вариантов.

## <a name="custom-validation"></a>Пользовательская проверка

Можно проверить ответ на запрос, прежде чем значение будет возвращено на следующем этапе **каскада**. Функция проверяющего элемента управления принимает параметр _контекста проверяющего элемента управления запроса_ и возвращает логическое значение, которое подтверждает успешное прохождение проверки для входных данных.

Контекст проверяющего элемента управления запроса поддерживает указанные ниже свойства.

| Свойство | ОПИСАНИЕ |
| :--- | :--- |
| _Контекст_ | Текущий контекст реплики для бота. |
| _Recognized_ | _Результат распознавателя запроса_, который содержит сведения о входных данных после обработки распознавателем. |
| _Варианты_ | Содержит _варианты выбора для запроса_, предоставленные в вызове, который запускал этот запрос. |

Результат распознавателя запроса имеет описанные ниже свойства.

| Свойство | ОПИСАНИЕ |
| :--- | :--- |
| _Успешно_ | Указывает, смог ли распознаватель успешно выполнить синтаксический анализ входных данных. |
| _Значение_ | Возвращаемое распознавателем значение. При необходимости это значение можно изменять в коде проверки. |

### <a name="implement-validation-code"></a>Реализация кода проверки

Пользовательская проверка связывается в конкретным запросом в конструкторе бота во время инициализации.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// ...
_dialogSet.Add(new NumberPrompt<int>(SizeRangePrompt, RangeValidatorAsync));
// ...
_dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));
// ...
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// ...
this.dialogSet.add(new NumberPrompt(SIZE_RANGE_PROMPT, this.rangeValidator));
// ...
this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
// ...
```

---

**Проверяющий элемент управления для размера группы**

Мы ограничиваем размер групп, для которых можно зарезервировать столик. Допустимый размер определяется свойством _validations_, которое мы использовали для вызова запроса о размере группы.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the party size is appropriate to make a reservation.
private async Task<bool> RangeValidatorAsync(
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
    var size = promptContext.Recognized.Value;
    var validRange = promptContext.Options.Validations as Range;
    if (size < validRange.Min || size > validRange.Max)
    {
        await promptContext.Context.SendActivitiesAsync(
            new Activity[]
            {
                MessageFactory.Text($"Sorry, we can only take reservations for parties " +
                    $"of {validRange.Min} to {validRange.Max}."),
                promptContext.Options.RetryPrompt,
            },
            cancellationToken: cancellationToken);
        return false;
    }

    return true;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
async rangeValidator(promptContext) {
    // Check whether the input could be recognized as an integer.
    if (!promptContext.recognized.succeeded) {
        await promptContext.context.sendActivity(
            "I'm sorry, I do not understand. Please enter the number of people in your party.");
        return false;
    }
    else if (promptContext.recognized.value % 1 != 0) {
        await promptContext.context.sendActivity(
            "I'm sorry, I don't understand fractional people.");
        return false;
    }

    // Check whether the party size is appropriate.
    var size = promptContext.recognized.value;
    if (size < promptContext.options.validations.min
        || size > promptContext.options.validations.max) {
        await promptContext.context.sendActivity(
            'Sorry, we can only take reservations for parties of '
            + `${promptContext.options.validations.min} to `
            + `${promptContext.options.validations.max}.`);
        await promptContext.context.sendActivity(promptContext.options.retryPrompt);
        return false;
    }

    return true;
}
```

---

**Проверка даты и времени**

В проверяющем элементе управления для даты резервирования мы установим ограничение — не менее чем через час от текущего времени. Теперь мы сохраняем первое разрешение, которое соответствуют заданным критериям, и удаляем все остальные.

Представленный код проверки не является исчерпывающим и хорошо работает только для тех входных данных, которые успешно преобразуются в дату и время. Он лишь демонстрирует некоторые возможности для проверки даты и времени, а код для конкретной реализации зависит от характера запрашиваемой у пользователя информации.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
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
    var earliest = DateTime.Now.AddHours(1.0);
    var value = promptContext.Recognized.Value.FirstOrDefault(v =>
        DateTime.TryParse(v.Value ?? v.Start, out var time) && DateTime.Compare(earliest, time) <= 0);

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

Запрос даты и времени возвращает список или массив всех возможных _разрешений даты и времени_, которые соответствуют входным данным пользователя. Например, значение 9:00 может означать 9 AM или 9 PM, а текст Sunday (воскресенье) также является неоднозначным. Кроме того, разрешение даты и времени может содержать дату, время, дату и время или диапазон времен. Запрос даты и времени использует для синтаксического анализа входных данных пользователя [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text).

## <a name="update-the-turn-handler"></a>Обновление обработчика реплик

Обновите обработчик реплик бота, чтобы он запускал диалог и принимал значение, которое возвращает по завершении. Здесь мы предполагаем, что пользователь взаимодействует с ботом, для которого активен каскадный диалог с запросом на следующем шаге.

Когда пользователь отправляет боту сообщение, выполняются описанные ниже действия.

1. Бот получает сведения о состоянии.
1. Бот создает контекст диалога
    - Если активный диалог отсутствует, бот начинает новый диалог, если пользователь еще не выполнял резервирование.
    - Если есть активный диалог, бот продолжает его. При завершении диалога сведения о резервировании записываются в кэш состояния.
1. Бот сохраняет любые изменения состояния.

Когда шаг диалога вызывает метод _prompt_ из контекста шага, выполняется следующее:

1. Создается экземпляр запроса, который затем помещается в стек диалогов и запускается. (Главный диалог ожидает, пока завершится этот запрос, прежде чем продолжить свою работу.)
1. Запрос отправляет пользователю действие с предложением ввести данные.

При вводе данных в ответ на запрос происходит следующее:

1. Запрос пытается обработать эти входные данные с учетом типа запроса (числовые данные, выбор варианта и т. п.).
1. Если запрос содержит пользовательскую проверку, выполняется ее код.
1. Если входные данные соответствуют всем проверкам, запрос завершается и возвращает обработанный ввод, в противном случае он запускает сам себя заново.

**Обработка результатов запроса**

Действия, выполняемые для результата запроса, зависят от целей получения этой информации от пользователя. Доступные параметры: 

- Вы можете управлять потоком диалога, например когда пользователь отвечает на запрос подтверждения или выбора.
- Кэшируйте информацию в состоянии диалога, например установите свойство _values_ для контекста этапа каскадного диалога, а затем возвращайте собранные сведения при завершении диалога.
- Сохраните информацию в сведениях о состоянии бота. Чтобы использовать этот вариант, диалог должен иметь доступ к методам доступа к свойству состояния бота.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(
    ITurnContext turnContext,
    CancellationToken cancellationToken = default(CancellationToken))
{
    switch (turnContext.Activity.Type)
    {
        // On a message from the user:
        case ActivityTypes.Message:

            // Get the current reservation info from state.
            var reservation = await _accessors.ReservationAccessor.GetAsync(
                turnContext,
                () => null,
                cancellationToken);

            // Generate a dialog context for our dialog set.
            var dc = await _dialogSet.CreateContextAsync(turnContext, cancellationToken);

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
                        $"We'll see you on {reservation.Date}.",
                        cancellationToken: cancellationToken);
                }
            }
            else
            {
                // Continue the dialog.
                var dialogTurnResult = await dc.ContinueDialogAsync(cancellationToken);

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
                        $"Your party of {reservation.Size} is confirmed for " +
                        $"{reservation.Date} in {reservation.Location}.",
                        cancellationToken: cancellationToken);
                }
            }

            // Save the updated dialog state into the conversation state.
            await _accessors.ConversationState.SaveChangesAsync(
                turnContext, false, cancellationToken);
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
                        `We'll see you on ${reservation.date}.`);
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
                        `confirmed for ${dialogTurnResult.result.date} in ` +
                        `${dialogTurnResult.result.location}.`);
                }
            }

            // Save the updated dialog state into the conversation state.
            await this.conversationState.saveChanges(turnContext, false);
            break;
        case ActivityTypes.EndOfConversation:
        case ActivityTypes.DeleteUserData:
            break;
        default:
            break;
    }
}
```

---

Можно использовать аналогичные методы для проверки ответов на запросы любого типа.

## <a name="test-your-bot"></a>Тестирование бота

1. Выполните этот пример на локальном компьютере. Если потребуются дополнительные инструкции, изучите файл README для [C#](https://aka.ms/dialog-prompt-cs) или [JS](https://aka.ms/dialog-prompt-js).
2. Запустите эмулятор и отправьте представленные ниже сообщения для тестирования бота.

![Тестирование примера с запросом для диалога](~/media/emulator-v4/test-dialog-prompt.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

Чтобы вызвать запрос непосредственно из обработчика шагов, примените пример _prompt-validations_ на [C#](https://aka.ms/cs-prompt-validation-sample) или [JS](https://aka.ms/js-prompt-validation-sample).

Библиотека диалогов также включает _запрос OAuth_ для получения _токена OAuth_ для доступа к другому приложению от имени пользователя. Дополнительные сведения об аутентификации см. в статье [о добавлении аутентификации в код бота](bot-builder-authentication.md).

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Создание сложной последовательной беседы с использованием ветвей и циклов](bot-builder-dialog-manage-complex-conversation-flow.md)

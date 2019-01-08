---
title: Сбор данных, которые вводит пользователь, с помощью диалогового окна | Документация Майкрософт
description: Узнайте, как запросить у пользователя входные данные, используя библиотеку диалогов из пакета SDK Bot Builder.
keywords: prompts, prompt, user input, dialogs, AttachmentPrompt, ChoicePrompt, ConfirmPrompt, DatetimePrompt, NumberPrompt, TextPrompt, reprompt, validation
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/21/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4acb12a5e06032db898a651c6c8bf1dae06765ef
ms.sourcegitcommit: f7a8f05fc05ff4a7212a437d540485bf68831604
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/21/2018
ms.locfileid: "53735974"
---
# <a name="gather-user-input-using-a-dialog-prompt"></a>Сбор данных, которые вводит пользователь, с помощью диалогового окна

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Сбор данных путем размещения вопросов — это один из основных способов взаимодействия между ботом и пользователями. Библиотека *Dialogs* упрощает отправку вопросов и проверку ответов, чтобы убедиться в правильности типов данных и (или) выполнении пользовательских правил проверки. В этой статье подробно описывается создание и вызов запросов из каскадного диалога.

## <a name="prerequisites"></a>Предварительные требования

- Код в этой статье основан на примере **DialogPromptBot**. Вам потребуется копия этого примера на языке [C#](https://aka.ms/dialog-prompt-cs) или [JS](https://aka.ms/dialog-prompt-js).
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

    public string Date { get; set; }
}
```

Далее мы добавим метод доступа к свойству состояния для данных о резервировании.

```csharp
public class DialogPromptBotAccessors
{
    public DialogPromptBotAccessors(ConversationState conversationState)
    {
        ConversationState = conversationState ?? throw new ArgumentNullException(nameof(conversationState));
    }

    public static string DialogStateAccessorKey { get; } = "DialogPromptBotAccessors.DialogState";
    public static string ReservationAccessorKey { get; } = "DialogPromptBotAccessors.Reservation";

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

    // Create and register state accesssors.
    // Acessors created here are passed into the IBot-derived class on every turn.
    services.AddSingleton<BotAccessors>(sp =>
    {
        // ...

        // Create the custom state accessor.
        // State accessors enable other components to read and write individual properties of state.
        var accessors = new BotAccessors(conversationState)
        {
            DialogStateAccessor = conversationState.CreateProperty<DialogState>(DialogPromptBotAccessors.DialogStateAccessorKey),
            ReservationAccessor = conversationState.CreateProperty<DialogPromptBot.Reservation>(DialogPromptBotAccessors.ReservationAccessorKey),
        };

        return accessors;
    });
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Для JavaScript нет необходимости изменять код службы HTTP, поэтому файл index.js мы оставим без изменений.

В файл bot.js, мы добавляем инструкции `require`, которые потребуются для работы бота с запросом для диалога. 
```javascript
const { ActivityTypes } = require('botbuilder');
const { DialogSet, WaterfallDialog, NumberPrompt, DateTimePrompt, ChoicePrompt, DialogTurnStatus } = require('botbuilder-dialogs');
```

Добавьте идентификаторы для методов доступа к свойству состояния, диалогов и запросов.

```javascript
// Define identifiers for state property accessors.
const DIALOG_STATE_ACCESSOR = 'dialogStateAccessor';
const RESERVATION_ACCESSOR = 'reservationAccessor';

// Define identifiers for dialogs and prompts.
const RESERVATION_DIALOG = 'reservationDialog';
const PARTY_SIZE_PROMPT = 'partySizePrompt';
const LOCATION_PROMPT = 'locationPrompt';
const RESERVATION_DATE_PROMPT = 'reservationDatePrompt';
```

---

### <a name="create-a-dialog-set-and-prompts"></a>Создание набора диалогов и запросов

Как правило, запросы и диалоги создаются и добавляются в набор диалогов на этапе инициализации бота. Это позволяет набору диалогов разрешать идентификаторы запросов при поступлении входных данных от пользователя в бот.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В классе `DialogPromptBot` определите идентификаторы диалогов, запросов и наборов диалогов.
```csharp
private const string ReservationDialog = "reservationDialog";
private const string PartySizePrompt = "partyPrompt";
private const string LocationPrompt = "locationPrompt";
private const string ReservationDatePrompt = "reservationDatePrompt";

private readonly DialogSet _dialogSet;
```

В конструкторе бота создайте набор диалогов, добавьте запросы и диалог резервирования. На этапе создания запросов мы определим пользовательские проверки, а функции для них реализуем позже.

```csharp
// The following code creates prompts and adds them to an existing dialog set. The DialogSet contains all the dialogs that can 
// be used at runtime. The prompts also references a validation method is not shown here.

public DialogPromptBot(DialogPromptBotAccessors accessors, ILoggerFactory loggerFactory)
{
   // ...

    // Create the dialog set and add the prompts, including custom validation.
    _dialogSet = new DialogSet(_accessors.DialogStateAccessor);
    _dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
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

```javascript
constructor(conversationState) {
    // Creates our state accessor properties.
    this.dialogStateAccessor = conversationState.createProperty(DIALOG_STATE_ACCESSOR);
    this.reservationAccessor = conversationState.createProperty(RESERVATION_ACCESSOR);
    this.conversationState = conversationState;
    // ...
    }
```

Затем создайте набор диалогов и добавьте запросы, в том числе пользовательскую проверку.

```javascript
    // ...
    this.dialogSet = new DialogSet(this.dialogStateAccessor);
    this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, this.partySizeValidator));
    this.dialogSet.add(new ChoicePrompt (LOCATION_PROMPT));
    this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
    // ...
```

Затем определите шаги каскадного диалога и добавьте его в набор.

```javascript
    // ...
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

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В файле DialogPromptBot.cs мы реализуем шаги каскадного диалога `PromptForPartySizeAsync`, `PromptForLocationAsync`, `PromptForReservationDateAsync` и `AcknowledgeReservationAsync`.

Здесь мы представили только `PromptForPartySizeAsync` и `PromptForLocationAsync` — это делегаты двух последовательных шагов каскадного диалога.

```csharp
private async Task<DialogTurnResult> PromptForPartySizeAsync(WaterfallStepContext stepContext)
{
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    // If the input is not valid, the prompt is restarted, causing it to reprompt for input
    // and this set of steps is repeated next turn. Otherwise, the prompt ends and returns a _dialog turn result_ object 
    // to the parent dialog. Control passes to the next step of your waterfall dialog, with the result of the prompt 
    // available in the waterfall step context's _result_ property.
    return await stepContext.PromptAsync(
        PartySizePrompt,
        new PromptOptions
        {
            Prompt = MessageFactory.Text("How many people is the reservation for?"),
            RetryPrompt = MessageFactory.Text("How large is your party?"),
        },
        cancellationToken);
}

private async Task<DialogTurnResult> PromptForLocationAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
{
    // Record the party size information in the current dialog state.
    int size = (int)stepContext.Result;
    stepContext.Values["size"] = size;

    return await stepContext.PromptAsync(
        "locationPrompt",
        new PromptOptions
        {
            Prompt = MessageFactory.Text("Please choose a location."),
            RetryPrompt = MessageFactory.Text("Sorry, please choose a location from the list."),
            Choices = ChoiceFactory.ToChoices(new List<string> { "Redmond", "Bellevue", "Seattle" }),
        },
        cancellationToken);
}
```

Пример выше использует запрос выбора, предоставляя для него все три аргумента. Метод `PromptForLocationAsync` используется в качестве шага в каскадном диалоге, а набор диалогов из нашего примера содержит и каскадный диалог, и запрос выбора с идентификатором `locationPrompt`.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Здесь `PARTY_SIZE_PROMPT` и `LOCATION_PROMPT` являются идентификаторами запросов, а `promptForPartySize` и `promptForLocation` представляют собой функции двух последовательных шагов в каскадном диалоге.

```javascript
async promptForPartySize(stepContext) {
    // Prompt for the party size. The result of the prompt is returned to the next step of the waterfall.
    return await stepContext.prompt(
        PARTY_SIZE_PROMPT, {
            prompt: 'How many people is the reservation for?',
            retryPrompt: 'How large is your party?'
        });
}

async promptForLocation(stepContext) {
    // Prompt for location
    return await stepContext.prompt(
        LOCATION_PROMPT, 'Select a location.', ['Redmond', 'Bellevue', 'Seattle']
    );
}
```

---

Второй параметр для метода _prompt_ принимает объект _аргументов запроса_ с указанными ниже свойствами.

| Свойство | ОПИСАНИЕ |
| :--- | :--- |
| _prompt_ | Исходное действие, отправляемое пользователю для получения входных данных. |
| _retry prompt_ | Действие, отправляемое пользователю при несоблюдении формата для первого введенного значения. |
| _choices_ | Список вариантов, из которых может выбрать пользователь при использовании запроса выбора. |

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
_dialogSet = new DialogSet(_accessors.DialogStateAccessor);
_dialogSet.Add(new NumberPrompt<int>(PartySizePrompt, PartySizeValidatorAsync));
_dialogSet.Add(new ChoicePrompt(LocationPrompt));
_dialogSet.Add(new DateTimePrompt(ReservationDatePrompt, DateValidatorAsync));
// ...
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// ...
this.dialogSet = new DialogSet(this.dialogStateAccessor);
this.dialogSet.add(new NumberPrompt(PARTY_SIZE_PROMPT, this.partySizeValidator));
this.dialogSet.add(new ChoicePrompt (LOCATION_PROMPT));
this.dialogSet.add(new DateTimePrompt(RESERVATION_DATE_PROMPT, this.dateValidator));
// ...
```

---

**Проверяющий элемент управления для размера группы**

Мы предоставляем возможность резервирования только для групп от 6 до 20 человек.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
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

**Проверка даты и времени**

В проверяющем элементе управления для даты резервирования мы установим ограничение — не менее чем через час от текущего времени. Теперь мы сохраняем первое разрешение, которое соответствуют заданным критериям, и удаляем все остальные. Приведенный ниже код проверки не является исчерпывающим и хорошо работает только для тех входных данных, которые успешно преобразуются в дату и время. Он лишь демонстрирует некоторые возможности для проверки даты и времени, а код для конкретной реализации зависит от характера запрашиваемой у пользователя информации.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Validates whether the reservation date is appropriate.
// Reservations must be made at least an hour in advance.
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

Запрос даты и времени возвращает список или массив всех возможных _разрешений даты и времени_, которые соответствуют входным данным пользователя. Например, значение 9:00 может означать 9 AM или 9 PM, а текст Sunday (воскресенье) также является неоднозначным. Кроме того, разрешение даты и времени может содержать дату, время, дату и время или диапазон времен. Запрос даты и времени использует для синтаксического анализа входных данных пользователя [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text).

### <a name="update-the-turn-handler"></a>Обновление обработчика реплик

Обновите обработчик реплик бота, чтобы он запускал диалог и принимал значение, которое возвращает по завершении. Здесь мы предполагаем, что пользователь взаимодействует с ботом, для которого активен каскадный диалог с запросом на следующем шаге.

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

**Обработка результатов запроса**

Действия, выполняемые для результата запроса, зависят от целей получения этой информации от пользователя. Доступные параметры: 

- Вы можете управлять последовательностью диалога, например когда пользователь отвечает на запрос подтверждения или выбора.
- Кэшируйте информацию в состоянии диалога, например установите свойство _values_ для контекста этапа каскадного диалога, а затем возвращайте собранные сведения при завершении диалога.
- Сохраните информацию в сведениях о состоянии бота. Чтобы использовать этот вариант, диалог должен иметь доступ к методам доступа к свойству состояния бота.

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

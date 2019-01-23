---
title: Создание собственных запросов на сбор данных, вводимых пользователем | Документация Майкрософт
description: Сведения о том, как управлять потоком общения с помощью простых запросов из пакета SDK Bot Framework.
keywords: conversation flow, prompts, conversation state, user state, custom prompts
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 11/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2e591f19f7df8fa6281573c0ac7f1330d95f4c53
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54225439"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>Создание собственных запросов на сбор данных, вводимых пользователем

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Общение между ботом и пользователем часто подразумевает запрашивание у пользователя информации, анализ его ответа и выполнение действий с учетом этой информации.

Бот должен отслеживать контекст общения, чтобы управлять его ходом и запоминать ответы на предыдущие вопросы. *Состояние* бота — это информация, которую программа отслеживает для правильных ответов на входящие сообщения.

## <a name="prerequisites"></a>Предварительные требования

- Код в этой статье основан на примере **запроса на ввод данных пользователем**. Вам потребуется копия этого примера на языке [C#](https://aka.ms/cs-primitive-prompt-sample) или [JS](https://aka.ms/js-primitive-prompt-sample).
- Понимание принципов [управления состоянием](bot-builder-concept-state.md) и [сохранения данных пользователя и диалога](bot-builder-howto-v4-state.md).
- [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started) для локального тестирования бота.

## <a name="about-the-sample-code"></a>Сведения о примере кода

В этой статье мы зададим пользователю несколько вопросов, а затем проверим и сохраним введенные данные.
Мы будем управлять потоком общения и процессом сбора данных, используя обработчик шагов бота и свойства состояния пользователя и общения.

1. Определение и настройка состояния
1. Использование свойств состояния для управления общением
   1. Обновите обработчик шагов бота.
   1. Реализуйте вспомогательный метод для управления сбором данных от пользователя.
   1. Реализуйте методы для проверки вводимых данных.

## <a name="define-and-configure-state"></a>Определение и настройка состояния

Нам нужно настроить для бота отслеживание следующих сведений:

- имя пользователя, возраст и произвольная дата, которые мы определим в состоянии пользователя;
- последний заданный пользователю вопрос, который мы определим в состоянии беседы.

Мы не планируем развертывать этот бот, поэтому состояние беседы и пользователя будем хранить во _временной памяти_. Ниже описан ряд ключевых аспектов для кода конфигурации.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Мы определяем следующие типы.

- Класс `UserProfile` для хранения собранных ботом сведений о пользователе.
- Класс `ConversationFlow` для хранения информации о текущем положении в беседе.
- Внутреннее перечисление `ConversationFlow.Question` для отслеживания текущего положения в беседе.
- Класс `CustomPromptBotAccessors` для хранения набора сведений об управлении состояниями.

Созданные нами объекты управления состоянием и методов доступа к свойству состояния содержатся в классе методов доступа бота, который передается в бот путем внедрения зависимостей в ASP.NET Core. В боте мы сохраняем сведения о методах доступа к свойству состояния, которую вы будете получать при создании бота на каждом очередном шаге.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Мы создаем объекты управления состоянием и передаем их при создании бота.
В боте мы определяем идентификаторы для свойств состояния и для отслеживания текущего положения в беседе, а затем сохраняем объекты управления состоянием и создаем методы доступа к свойству состояния в конструкторе бота.

---

## <a name="use-state-properties-to-direct-the-conversation"></a>Использование свойств состояния для управления общением

После завершения настройки свойств состояния мы можем использовать их в боте.

- Определите [обработчик шагов](#the-bots-turn-handler) для доступа к состоянию и вызова вспомогательного метода.
- Реализуйте [вспомогательный метод](#filling-out-the-user-profile), чтобы управлять сбором данных для профиля пользователя.
- Реализуйте [методы проверки](#parse-and-validate-input) для синтаксического анализа и проверки данных, вводимых пользователем.

### <a name="the-bots-turn-handler"></a>Обработчик шагов бота

Мы применим методы доступа к свойству состояния, чтобы получить свойства состояния из контекста шага.
Когда потребуется заполнить профиль пользователя, мы вызовем вспомогательный метод и затем сохраним изменения состояния.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
public async Task OnTurnAsync(ITurnContext turnContext, CancellationToken cancellationToken = default(CancellationToken))
{
   if (turnContext.Activity.Type == ActivityTypes.Message)
    {
        // Get the state properties from the turn context.
        ConversationFlow flow = await _accessors.ConversationFlowAccessor.GetAsync(turnContext, () => new ConversationFlow());
        UserProfile profile = await _accessors.UserProfileAccessor.GetAsync(turnContext, () => new UserProfile());

        await FillOutUserProfileAsync(flow, profile, turnContext);

        // Update state and save changes.
        await _accessors.ConversationFlowAccessor.SetAsync(turnContext, flow);
        await _accessors.ConversationState.SaveChangesAsync(turnContext);

        await _accessors.UserProfileAccessor.SetAsync(turnContext, profile);
        await _accessors.UserState.SaveChangesAsync(turnContext);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// The bot's turn handler.
async onTurn(turnContext) {
    // This bot listens for message activities.
    if (turnContext.activity.type === ActivityTypes.Message) {
        // Get the state properties from the turn context.
        const flow = await this.conversationFlow.get(turnContext, { lastQuestionAsked: question.none });
        const profile = await this.userProfile.get(turnContext, {});

        await MyBot.fillOutUserProfile(flow, profile, turnContext);

        // Update state and save changes.
        await this.conversationFlow.set(turnContext, flow);
        await this.conversationState.saveChanges(turnContext);

        await this.userProfile.set(turnContext, profile);
        await this.userState.saveChanges(turnContext);
    }
}
```

---

### <a name="filling-out-the-user-profile"></a>Заполнение профиля пользователя

Мы начнем со сбора сведений. Интерфейсы для всех данных будут схожими.

- Возвращаемое значение позволяет узнать, содержат ли входные данные допустимый ответ на этот вопрос.
- Если проверка проходит успешно, она возвращает извлеченное и нормализованное значение для сохранения.
- Если процедура проверки завершается ошибкой, она создает сообщение, которое бот может отправить для повторного запроса той же информации.

 В [следующем разделе](#parse-and-validate-input) мы определим вспомогательные методы для анализа и проверки пользовательского ввода.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
private static async Task FillOutUserProfileAsync(ConversationFlow flow, UserProfile profile, ITurnContext turnContext)
{
    string input = turnContext.Activity.Text?.Trim();
    string message;
    switch (flow.LastQuestionAsked)
    {
        case ConversationFlow.Question.None:
            await turnContext.SendActivityAsync("Let's get started. What is your name?");
            flow.LastQuestionAsked = ConversationFlow.Question.Name;
            break;
        case ConversationFlow.Question.Name:
            if (ValidateName(input, out string name, out message))
            {
                profile.Name = name;
                await turnContext.SendActivityAsync($"Hi {profile.Name}.");
                await turnContext.SendActivityAsync("How old are you?");
                flow.LastQuestionAsked = ConversationFlow.Question.Age;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Age:
            if (ValidateAge(input, out int age, out message))
            {
                profile.Age = age;
                await turnContext.SendActivityAsync($"I have your age as {profile.Age}.");
                await turnContext.SendActivityAsync("When is your flight?");
                flow.LastQuestionAsked = ConversationFlow.Question.Date;
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
        case ConversationFlow.Question.Date:
            if (ValidateDate(input, out string date, out message))
            {
                profile.Date = date;
                await turnContext.SendActivityAsync($"Your cab ride to the airport is scheduled for {profile.Date}.");
                await turnContext.SendActivityAsync($"Thanks for completing the booking {profile.Name}.");
                await turnContext.SendActivityAsync($"Type anything to run the bot again.");
                flow.LastQuestionAsked = ConversationFlow.Question.None;
                profile = new UserProfile();
                break;
            }
            else
            {
                await turnContext.SendActivityAsync(message ?? "I'm sorry, I didn't understand that.");
                break;
            }
    }
}


```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```javascript
// Manages the conversation flow for filling out the user's profile.
static async fillOutUserProfile(flow, profile, turnContext) {
    const input = turnContext.activity.text;
    let result;
    switch (flow.lastQuestionAsked) {
        // If we're just starting off, we haven't asked the user for any information yet.
        // Ask the user for their name and update the conversation flag.
        case question.none:
            await turnContext.sendActivity("Let's get started. What is your name?");
            flow.lastQuestionAsked = question.name;
            break;

        // If we last asked for their name, record their response, confirm that we got it.
        // Ask them for their age and update the conversation flag.
        case question.name:
            result = this.validateName(input);
            if (result.success) {
                profile.name = result.name;
                await turnContext.sendActivity(`I have your name as ${profile.name}.`);
                await turnContext.sendActivity('How old are you?');
                flow.lastQuestionAsked = question.age;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for their age, record their response, confirm that we got it.
        // Ask them for their date preference and update the conversation flag.
        case question.age:
            result = this.validateAge(input);
            if (result.success) {
                profile.age = result.age;
                await turnContext.sendActivity(`I have your age as ${profile.age}.`);
                await turnContext.sendActivity('When is your flight?');
                flow.lastQuestionAsked = question.date;
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }

        // If we last asked for a date, record their response, confirm that we got it,
        // let them know the process is complete, and update the conversation flag.
        case question.date:
            result = this.validateDate(input);
            if (result.success) {
                profile.date = result.date;
                await turnContext.sendActivity(`Your cab ride to the airport is scheduled for ${profile.date}.`);
                await turnContext.sendActivity(`Thanks for completing the booking ${profile.name}.`);
                await turnContext.sendActivity('Type anything to run the bot again.');
                flow.lastQuestionAsked = question.none;
                profile = {};
                break;
            } else {
                // If we couldn't interpret their input, ask them for it again.
                // Don't update the conversation flag, so that we repeat this step.
                await turnContext.sendActivity(
                    result.message || "I'm sorry, I didn't understand that.");
                break;
            }
    }
}
```

---

### <a name="parse-and-validate-input"></a>Синтаксический анализ и проверка входных данных

Мы будем использовать для проверки входных данных следующие критерии.

- **Name** (имя) не может быть пустой строкой. Для нормализации выполняется усечение пробелов.
- Значение **age** (возраст) должно находиться в диапазоне от 18 до 120. Для нормализации оно округляется до целого числа.
- **Date** (дата) может быть любой датой или временем в будущем, если разница между указанным значением и текущим временем составляет не менее одного часа.
  Для нормализации возвращается только дата, содержащаяся во входных данных.

> [!NOTE]
> При вводе даты и возраста мы используем библиотеку [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) для начальной обработки.
> Мы предоставляем здесь пример кода, но не описываем работу библиотек средств распознавания текста. Можно использовать и другие варианты синтаксического анализа входных данных.
> Дополнительные сведения об этих библиотеках можно найти в репозитории **README**.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Добавьте в бот следующие методы проверки.

```csharp
private static bool ValidateName(string input, out string name, out string message)
{
    name = null;
    message = null;

    if (string.IsNullOrWhiteSpace(input))
    {
        message = "Please enter a name that contains at least one character.";
    }
    else
    {
        name = input.Trim();
    }

    return message is null;
}

private static bool ValidateAge(string input, out int age, out string message)
{
    age = 0;
    message = null;

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try
    {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        List<ModelResult> results = NumberRecognizer.RecognizeNumber(input, Culture.English);
        foreach (var result in results)
        {
            // result.Resolution is a dictionary, where the "value" entry contains the processed string.
            if (result.Resolution.TryGetValue("value", out object value))
            {
                age = Convert.ToInt32(value);
                if (age >= 18 && age <= 120)
                {
                    return true;
                }
            }
        }

        message = "Please enter an age between 18 and 120.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120.";
    }

    return message is null;
}

private static bool ValidateDate(string input, out string date, out string message)
{
    date = null;
    message = null;

    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try
    {
        List<ModelResult> results = DateTimeRecognizer.RecognizeDateTime(input, Culture.English);

        // Check whether any of the recognized date-times are appropriate,
        // and if so, return the first appropriate date-time. We're checking for a value at least an hour in the future.
        DateTime earliest = DateTime.Now.AddHours(1.0);
        foreach (ModelResult result in results)
        {
            // result.Resolution is a dictionary, where the "values" entry contains the processed input.
            var resolutions = result.Resolution["values"] as List<Dictionary<string, string>>;
            foreach (var resolution in resolutions)
            {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                if (resolution.TryGetValue("value", out string dateString)
                    || resolution.TryGetValue("start", out dateString))
                {
                    if (DateTime.TryParse(dateString, out var candidate)
                        && earliest < candidate)
                    {
                        date = candidate.ToShortDateString();
                        return true;
                    }
                }
            }
        }

        message = "I'm sorry, please enter a date at least an hour out.";
    }
    catch
    {
        message = "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out.";
    }

    return false;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте в бот следующие методы проверки.

```javascript
// Validates name input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateName(input) {
    const name = input && input.trim();
    return name != undefined
        ? { success: true, name: name }
        : { success: false, message: 'Please enter a name that contains at least one character.' };
};

// Validates age input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateAge(input) {

    // Try to recognize the input as a number. This works for responses such as "twelve" as well as "12".
    try {
        // Attempt to convert the Recognizer result to an integer. This works for "a dozen", "twelve", "12", and so on.
        // The recognizer returns a list of potential recognition results, if any.
        const results = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "value" entry contains the processed string.
            const value = result.resolution['value'];
            if (value) {
                const age = parseInt(value);
                if (!isNaN(age) && age >= 18 && age <= 120) {
                    output = { success: true, age: age };
                    return;
                }
            }
        });
        return output || { success: false, message: 'Please enter an age between 18 and 120.' };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an age. Please enter an age between 18 and 120."
        };
    }
}

// Validates date input. Returns whether validation succeeded and either the parsed and normalized
// value or a message the bot can use to ask the user again.
static validateDate(input) {
    // Try to recognize the input as a date-time. This works for responses such as "11/14/2018", "today at 9pm", "tomorrow", "Sunday at 5pm", and so on.
    // The recognizer returns a list of potential recognition results, if any.
    try {
        const results = Recognizers.recognizeDateTime(input, Recognizers.Culture.English);
        const now = new Date();
        const earliest = now.getTime() + (60 * 60 * 1000);
        let output;
        results.forEach(function (result) {
            // result.resolution is a dictionary, where the "values" entry contains the processed input.
            result.resolution['values'].forEach(function (resolution) {
                // The processed input contains a "value" entry if it is a date-time value, or "start" and
                // "end" entries if it is a date-time range.
                const datevalue = resolution['value'] || resolution['start'];
                // If only time is given, assume it's for today.
                const datetime = resolution['type'] === 'time'
                    ? new Date(`${now.toLocaleDateString()} ${datevalue}`)
                    : new Date(datevalue);
                if (datetime && earliest < datetime.getTime()) {
                    output = { success: true, date: datetime.toLocaleDateString() };
                    return;
                }
            });
        });
        return output || { success: false, message: "I'm sorry, please enter a date at least an hour out." };
    } catch (error) {
        return {
            success: false,
            message: "I'm sorry, I could not interpret that as an appropriate date. Please enter a date at least an hour out."
        };
    }
}
```

---

## <a name="test-the-bot-locally"></a>Локальная проверка бота
1. Выполните этот пример на локальном компьютере. Если потребуются дополнительные инструкции, изучите файл README для примера на [C#](https://aka.ms/cs-primitive-prompt-sample) или [JS](https://aka.ms/js-primitive-prompt-sample).
1. Выполните тестирование в эмуляторе, как показано ниже.

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

[Библиотека Dialogs](bot-builder-concept-dialog.md) предоставляет классы, которые автоматизируют многие аспекты управления беседами. 

## <a name="next-step"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Реализация последовательной беседы](bot-builder-dialog-manage-conversation-flow.md)

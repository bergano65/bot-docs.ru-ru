---
title: Управление процессом общения с помощью простых запросов | Документация Майкрософт
description: Сведения о том, как управлять процессом общения с помощью простых запросов с использованием пакета SDK Bot Builder.
keywords: conversation flow, prompts
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 7/20/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 1514e1bcafc87be9e8449382bca7aa156e512ed9
ms.sourcegitcommit: e8c513d3af5f0c514cadcbcd0a737a7393405afa
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/20/2018
ms.locfileid: "40228353"
---
# <a name="prompt-users-for-input-using-your-own-prompts"></a>Создание собственных запросов на ввод данных пользователем
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Общение между ботом и пользователем часто подразумевает запрашивание у пользователя информации, анализ его ответа и выполнение действий с учетом этой информации. [В этом разделе](bot-builder-prompts.md) подробно описано, как запросить у пользователя входные данные с помощью библиотеки **диалогов**. Помимо прочего, библиотека **диалогов** отвечает за отслеживание текущего диалога и текущего вопроса. Она также предоставляет методы для запрашивания разных типов данных, таких как текст, числа, дата и время и т. д. 

В некоторых случаях встроенная библиотека **диалогов** может не подойти для вашего бота. Использование библиотеки **диалогов** может привести к большим затратам на обслуживание простых ботов. Также она может быть слишком негибкой или же неэффективной для реализации задач бота. В таком случае можно не использовать эту библиотеку, а реализовать собственную логику запросов. В этом разделе объясняется, как настроить базовый бот *EchoBot*, чтобы управлять диалогом с использованием собственных запросов.

## <a name="track-prompt-states"></a>Отслеживание состояний запросов

В диалоге на основе запросов необходимо отслеживать, в какой точке диалога вы сейчас находитесь и какой вопрос был только что задан. В коде это реализуется с помощью управления несколькими флагами. 

Например, можно создать несколько свойств, которые нужно отслеживать. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

В этом примере класс профиля пользователя вложен в блок сведений о запросе, позволяя хранить все эти сведения в данных о [состоянии](bot-builder-howto-v4-state.md) бота.

```csharp
// Where user information will be stored
public class UserProfile
{
    public string userName = null;
    public string workPlace = null;
    public int age = 0;
}

// Flags to keep track of our prompts, and the 
// user profile object for this conversation
public class PromptInformation
{
    public string topicTitle = null;
    public string prompt = null;
    public UserProfile userProfile = null;
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В этих данных о состоянии хранятся сведения о текущей теме и текущем запросе. Чтобы эти флаги функционировали должным образом после развертывания в облако, мы сохраняем их в данных о [состояния диалога](bot-builder-howto-v4-state.md). Реализуйте этот код в основной логике бота.

**app.js**
```javascript
// Define a topicStates object if it doesn't exist in the convoState.
if(!convo.topicStates){
    convo.topicStates = {
        "topicTitle": undefined, // Current conversation topic in progress
        "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
    };
}
```

---

## <a name="manage-a-topic-of-conversation"></a>Управление темой диалога

При последовательном общении, в ходе которого, например, собирается информация от пользователя, необходимо знать, в какой точке диалога пользователь присоединился и в какой находится сейчас. Это можно отслеживать в основном обработчике этапа бота. Для этого нужно задать и проверить флаги запросов, определенные выше, а затем выполнить соответствующие действия. В следующем примере показано, как использовать эти флаги для сбора данных профиля пользователя в процессе общения.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
// Get our current state, as defined above
var convoState = context.GetConversationState<PromptInformation>();

if (convoState.userProfile == null)
{
    await context.SendActivity("Welcome new user, please fill out your profile information.");
    convoState.topicTitle = "profileTopic"; // Start the userProfile topic
    convoState.userProfile = new UserProfile();
}

// Start or continue a conversation if we are in one
if (convoState.topicTitle == "profileTopic")
{
    if (convoState.userProfile.userName == null && convoState.prompt == null)
    {
        convoState.prompt = "askName";
        await context.SendActivity("What is your name?");
    }
    else if (convoState.prompt == "askName")
    {
        // Save the user's response
        convoState.userProfile.userName = context.Activity.Text;

        // Ask next question
        convoState.prompt = "askAge";
        await context.SendActivity("How old are you?");
    }
    else if (convoState.prompt == "askAge")
    {
        // Save user's response
        if (!Int32.TryParse(context.Activity.Text, out convoState.userProfile.age))
        {
            // Failed to convert to int
            await context.SendActivity("Failed to convert string to int");
        }
        else
        {
            // Ask next question
            convoState.prompt = "workPlace";
            await context.SendActivity("Where do you work?");
        }

    }
    else if (convoState.prompt == "workPlace")
    {
        // Save user's response
        convoState.userProfile.workPlace = context.Activity.Text;

        // Done
        convoState.topicTitle = null; // Reset conversation flag
        convoState.prompt = null;     // Reset the prompt flag
        
        await context.SendActivity("Thank you. Your profile is complete.");
    }
    else // Somehow our flags got inappropriately set
    {
        await context.SendActivity("Looks like something went wrong, let's start over");
        convoState.userProfile = null;
        convoState.prompt = null;
        convoState.topicTitle = null;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**
```javascript
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        const isMessage = (context.activity.type === 'message');
        // State will store all of your information 
        const convo = conversationState.get(context);

        // Defined flags to manage the conversation flow and prompt states
        // convo.topicStates.topicTitle - Current conversation topic in progress
        // convo.topicStates.prompt - Current prompt state in progress - indicate what question is being asked.
        
        if (isMessage) {
            // Define a topicStates object if it doesn't exist in the convoState.
            if(!convo.topicStates){
                convo.topicStates = {
                    "topicTitle": undefined, // Current conversation topic in progress
                    "prompt": undefined      // Current prompt state in progress - indicate what question is being asked.
                };
            }
            
            // If user profile is not defined then define it.
            if(!convo.userProfile){
                
                await context.sendActivity(`Welcome new user, please fill out your profile information.`);
                convo.topicStates.topicTitle = "profileTopic"; // Start the userProfile topic
                convo.userProfile = { // Define the user's profile object
                    "userName": undefined,
                    "age": undefined,
                    "workPlace": undefined
                }; 
            }

            // Start or continue a conversation if we are in one
            if(convo.topicStates.topicTitle == "profileTopic"){
                if(!convo.userProfile.userName && !convo.topicStates.prompt){
                    convo.topicStates.prompt = "askName";
                    await context.sendActivity("What is your name?");
                }
                else if(convo.topicStates.prompt == "askName"){
                    // Save the user's response
                    convo.userProfile.userName = context.activity.text; 

                    // Ask next question
                    convo.topicStates.prompt = "askAge";
                    await context.sendActivity("How old are you?");
                }
                else if(convo.topicStates.prompt == "askAge"){
                    // Save user's response
                    convo.userProfile.age = context.activity.text;

                    // Ask next question
                    convo.topicStates.prompt = "workPlace";
                    await context.sendActivity("Where do you work?");
                }
                else if(convo.topicStates.prompt == "workPlace"){
                    // Save user's response
                    convo.userProfile.workPlace = context.activity.text;

                    // Done
                    convo.topicStates.topicTitle = undefined; // Reset conversation flag
                    convo.topicStates.prompt = undefined;     // Reset the prompt flag
                    await context.sendActivity("Thank you. Your profile is complete.");
                }
            }

            // Check for valid intents
            else if(context.activity.text && context.activity.text.match(/hi/ig)){
                await context.sendActivity(`Hi ${convo.userProfile.userName}.`);
            }
            else {
                // Default message
                await context.sendActivity("Hi. I'm the Contoso bot.");
            }
        }

    });
});

```

---

В примере кода выше проверяется, определен ли профиль пользователя в памяти. Если профиль не определен и это новый пользователь, устанавливается флаг для автоматического начала темы диалога. Затем показано, как в процессе диалога вы можете задать флагу `prompt` значение текущего вопроса. Если этому флагу задано правильное значение, условное предложение перехватывает каждый ответ пользователя и обрабатывает его соответствующим образом. 

Эти флаги нужно сбросить, когда вы закончите собирать информацию, чтобы бот не выполнял замкнутый цикл. Вы можете расширить эту базовую настройку, чтобы управлять более сложными процессами общения в соответствии с требованиями бота. Для этого нужно просто определить другие флаги или создать ветки диалога на основе введенных пользователем данных.

## <a name="manage-multiple-topics-of-conversations"></a>Управление несколькими темами диалога

Реализовав обработку одной темы диалога, вы можете расширить логику бота, чтобы обрабатывать несколько тем диалога. Так же как и одну тему, несколько тем можно обработать, просто проверяя условия, которые позволяют активировать определенную тему, и выполняя необходимую последовательность действий.

Вы можете выполнить рефакторинг примера выше, чтобы включить другие функции и темы, например резервирование столика или заказ ужина. Вы можете добавлять сведения в флаги профиля пользователя или состояния темы, если нужно отслеживать состояние диалога.

Чтобы было удобнее управлять несколькими темами диалога, можно использовать один метод, который будет предоставлять основное меню. С помощью [предлагаемых действий](bot-builder-howto-add-suggested-actions.md) можно предоставить пользователю выбор темы, которую он может начать, а затем приступить к обсуждению. Возможно, также будет полезно разделить код на независимые функции, чтобы было проще контролировать процесс общения.

## <a name="validate-user-input"></a>Проверка вводимых пользователем данных

В библиотеке **диалогов** предоставляются встроенные методы проверки введенных пользователем данных, но это можно сделать и с помощью собственных запросов. Например, запрашивая возраст пользователя, в качестве ответа мы ожидаем получить число, а не что-то наподобие "Боб".

Анализ чисел или даты и времени — это сложная задача, которая не рассматривается в этой статье. К счастью, вы можете использовать библиотеку. Для правильного анализа этих сведений используется библиотека [распознавателя текста от корпорации Майкрософт](https://github.com/Microsoft/Recognizers-Text). Этот пакет можно получить через NuGet или скачать из репозитория. Он также включен в библиотеку **диалогов**, которая достойна упоминания, но мы не используем ее в рамках этой статьи.

Эта библиотека особенно полезна для анализа стандартного кода или сложных ответов, например на вопросы о дате, времени или номерах телефонов. В этом примере мы только проверяем число гостей, на которых заказан ужин. Но этот же принцип можно доработать, чтобы выполнять более универсальные проверки. Если вы еще не работали с этой библиотекой, просмотрите ее содержимое на сайте, чтобы понять, какие возможности она предлагает.

В примере ниже мы только рассмотрим использование `RecognizeNumber`. Сведения о том, как использовать другие методы распознавателя из библиотеки, можно найти в [документации репозитория](https://github.com/Microsoft/Recognizers-Text/blob/master/README.md).

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы использовать библиотеку распознавателя, добавьте ее в инструкции using.

```csharp
using Microsoft.Recognizers.Text.Number;
using Microsoft.Recognizers.Text;
using System.Linq; // Used to get the first result from the recognizer
```

Затем определите функцию, которая фактически выполняет проверку.

```csharp
private async Task<bool> ValidatePartySize(ITurnContext context, string value)
{
    try
    {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = NumberRecognizer.RecognizeNumber(input, Culture.English);

        // Attempt to convert the Recognizer result to an integer
        Int32.TryParse(result.First().Text, out int partySize);
        
        if (partySize < 6)
        {
            throw new Exception("Party size too small.");
        }
        else if (partySize > 20)
        {
            throw new Exception("Party size too big.");
        }

        // If we got through this, the number is valid
        return true;
    }
    catch (Exception)
    {
        await context.SendActivity("Error with your party size. < br /> Please specify a number between 6 - 20.");
        return false;
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы использовать библиотеку распознавателя, выполните команду require в **app.js**:

```javascript
// Required packages for this bot
var Recognizers = require('@microsoft/recognizers-text-suite');
```

Затем определите функцию, которая фактически выполняет проверку.

```javascript
// Support party size between 6 and 20 only
async function validatePartySize(context, input){
    try {
        // Recognize the input as a number. This works for responses such as
        // "twelve" as well as "12"
        var result = Recognizers.recognizeNumber(input, Recognizers.Culture.English);
        var value = parseInt(results[0].resolution.value);

        if(value < 6) {
            throw new Error(`Party size too small.`);
        }
        else if(value > 20){
            throw new Error(`Party size too big.`);
        }
        return true; // Return the valid value
    }
    catch (err){
        await context.sendActivity(`${err.message} <br/>Please specify a number between 6 - 20.`);
        return false;
    }
}
```

---

В пределах запроса, который нужно проверить, вызовите функцию проверки, прежде чем переходить к следующему запросу. Если проверка завершится ошибкой, задайте вопрос еще раз.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```csharp
if (convoState.prompt == "partySize")
{
    if (await ValidatePartySize(context, context.Activity.Text))
    {
        // Save user's response in our state, ReservationInfo, which 
        // is a new class we've added to our state
        convoState.ReservationInfo.partySize = context.Activity.Text;

        // Ask next question
        convoState.prompt = "reserveName";
        await context.SendActivity("Who's name will this be under?");
    }
    else
    {
        // Ask again
        await context.SendActivity("How many people are in your party?");
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**app.js**

```javascript
// ...
if(convo.topicStates.prompt == "partySize"){
    if(await validatePartySize(context, context.activity.text)){
        // Save user's response
        convo.reservationInfo.partySize = context.activity.text;
        
        // Ask next question
        topicStates.prompt = "reserveName";
        await context.sendActivity("Who's name will this be under?");
    }
    else {
        // Ask again
        await context.sendActivity("How many people are in your party?");
    }
}
// ...
```

---

## <a name="next-step"></a>Дальнейшие действия

Теперь, когда вы знаете, как обрабатывать состояния запросов самостоятельно, узнайте, как с помощью библиотеки **диалогов** запросить у пользователя входные данные.

> [!div class="nextstepaction"]
> [Создание запросов на ввод данных пользователем с помощью библиотеки диалогов](bot-builder-prompts.md)

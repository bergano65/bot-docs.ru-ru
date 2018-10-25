---
title: Перевод введенных пользователем данных для создания бота с поддержкой нескольких языков | Документация Майкрософт
description: Узнайте, как автоматически перевести введенные пользователем данные на язык, используемый ботом по умолчанию, а затем обратно на язык пользователя.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/06/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a6bec55010124892376904070cc9ed88c659efbd
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999861"
---
# <a name="translate-from-the-users-language-to-make-your-bot-multilingual"></a>Бот считается многоязычным после того, как выполнил перевод с языка пользователя

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Чтобы автоматически переводить сообщения на язык, который понимает бот и при необходимости переводить его ответы на язык пользователя, бот может использовать [Microsoft Translator](https://www.microsoft.com/en-us/translator/).
<!-- 
- [Get a Text Services key](#get-a-text-services-key)
- [Installing Packages](#installing-packages)
- [Combine translation with LUIS or QnA Maker](#using-luis)
- [Combine translation with QnA Maker](#using-qna-maker)
-->

## <a name="get-a-text-services-key"></a>Получение ключа Текстовых служб

Для использования службы Microsoft Translator потребуется ключ. Его можно получить на портале Azure, воспользовавшись инструкциями из статьи [Разработка собственного приложения](https://www.microsoft.com/en-us/translator/trial.aspx#get-started).

## <a name="installing-packages"></a>Установка пакетов

Убедитесь, что у вас есть пакеты, необходимые для добавления перевода к вашему боту.

# <a name="ctabcsrefs"></a>[C#](#tab/csrefs)

[Добавьте ссылку](https://docs.microsoft.com/en-us/nuget/tools/package-manager-ui) на предварительную версию следующих пакетов NuGet.

* `Microsoft.Bot.Builder.Integration.AspNet.Core`
* `Microsoft.Bot.Builder.Ai` (требуется для перевода)

Если вы собираетесь объединить перевод с Интеллектуальной службой распознавания речи (LUIS), также необходимо добавить ссылку на

* `Microsoft.Bot.Builder.Luis` (требуется для LUIS)

# <a name="javascripttabjsrefs"></a>[JavaScript](#tab/jsrefs)

Любую из этих служб можно добавить к боту с помощью пакета botbuilder-ai. Этот пакет можно добавить в проект с помощью npm.
* `npm install --save botbuilder@preview`
* `npm install --save botbuilder-ai@preview`

---

## <a name="configure-translation"></a>Настройка перевода

Можно настроить бот на вызов перевода для каждого сообщения, получаемого от пользователя. Для этого просто добавьте его в стек ПО промежуточного слоя бота. ПО промежуточного слоя использует результат перевода, чтобы изменить сообщение пользователя, используя объект контекста.


# <a name="ctabcssetuptranslate"></a>[C#](#tab/cssetuptranslate)

Начните с примера EchoBot в пакете SDK и обновите метод `ConfigureServices` в файле `Startup.cs`, чтобы добавить `TranslationMiddleware` к боту. Данное действие настраивает бот на перевод каждого сообщения, получаемого от пользователя. <!--, by simply adding it to your bot's middleware set. The middleware stores the translation results on the context object. -->

**Startup.cs.**
```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Bot.Builder.Ai;
using Microsoft.Bot.Builder.BotFramework;
using Microsoft.Bot.Builder.Core.Extensions;
using Microsoft.Bot.Builder.Integration.AspNet.Core;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddBot<TranslatorBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;
        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));

    });
}


```

> [!TIP] 
> Пакет SDK для Bot Builder используется в только что отправленных сообщениях для автоматического определения языка пользователя. Чтобы переопределить эту функцию, вы можете указать дополнительные параметры обратного вызова, чтобы добавить собственную логику для определения и изменения языка пользователя.  



Просмотрите код в файле `EchoBot.cs`, где бот отправляет высказывание "You sent", за которым следует сообщение пользователя.

```cs
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using System.Threading.Tasks;

namespace Microsoft.Bot.Samples
{
    public class EchoBot : IBot
    {
        public EchoBot() { }

        public async Task OnTurn(ITurnContext context)
        {
            switch (context.Activity.Type)
            {
                case ActivityTypes.Message:
                // echo back the user's input.
                    await context.SendActivity($"You sent '{context.Activity.Text}'");


                case ActivityTypes.ConversationUpdate:
                    foreach (var newMember in context.Activity.MembersAdded)
                    {
                        if (newMember.Id != context.Activity.Recipient.Id)
                        {
                            await context.SendActivity("Hello and welcome to the echo bot.");
                        }
                    }
                    break;
            }
        }
        
    }
}
```

При добавлении ПО промежуточного слоя для перевода сведения о том, следует ли переводить ответы обратно на язык пользователя, находятся в необязательном параметре. Чтобы перевести сообщения пользователя на язык бота, в `Startup.cs` необходимо установить значение параметра как `false`.

```cs
        // The first parameter is a list of languages the bot recognizes
        // The second parameter is your API key
        // The third parameter specifies whether to translate the bot's replies back to the user's language
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));
```

# <a name="javascripttabjssetuptranslate"></a>[JavaScript](#tab/jssetuptranslate)

Чтобы настроить ПО промежуточного слоя для перевода с помощью бота echo, вставьте следующее в app.js.

```javascript
const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');
const { LanguageTranslator, LocaleConverter } = require('botbuilder-ai');
const restify = require('restify');

// Create server
let server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, function () {
    console.log(`${server.name} listening to ${server.url}`);
});

// Create adapter
const adapter = new BotFrameworkAdapter({ 
    appId: process.env.MICROSOFT_APP_ID, 
    appPassword: process.env.MICROSOFT_APP_PASSWORD 
});

// Add conversation state middleware
const conversationState = new ConversationState(new MemoryStorage());
adapter.use(conversationState);

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['fr', 'de'] 
});
adapter.use(languageTranslator);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, (context) => {
        if (context.activity.type === 'message') {
            const state = conversationState.get(context);
            const count = state.count === undefined ? state.count = 0 : ++state.count;
            return context.sendActivity(`${count}: You said "${context.activity.text}"`);
        } else {
            return context.sendActivity(`[${context.activity.type} event detected]`);
        }
    });
});
```

---

## <a name="run-the-bot-and-see-translated-input"></a>Запуск бота и просмотр переведенных входных данных

Запустите бот и введите несколько сообщений на других языках. Вы увидите, что бот перевел сообщение пользователя и указал перевод в своем ответе.

![Определение ботом языка и перевод на него входных данных](./media/how-to-bot-translate/bot-detects-language-translates-input.png)




## <a name="invoke-logic-in-the-bots-native-language"></a>Вызов логики на языке, который бот использует по умолчанию

Теперь добавьте логику, которая проверяет наличие слов на английском языке. Если пользователь говорит "help" or "cancel" на другом языке, бот переводит это на английский язык и вызывает логику проверки для английского слова "help" or "cancel".

# <a name="ctabcshelp"></a>[C#](#tab/cshelp)
```cs
        case ActivityTypes.Message:
            // check the message text before calling context.SendActivity
            var messagetext = context.Activity.Text.Trim().ToLower();
            if (string.Equals("help", messagetext))
            {
                await context.SendActivity("You asked for help.");
            }
```

# <a name="javascripttabjshelp"></a>[JavaScript](#tab/jshelp)
```javascript
if (context.activity.type === 'message') {
    // check the message text before calling context.sendActivity
    var messagetext = context.activity.text.trim().toLowerCase();
    if(messagetext === "help"){
        await context.sendActivity("you said help");
    }
}
```

---

![Бот определяет слово "help" на французском языке](./media/how-to-bot-translate/bot-detects-help-french.png)



## <a name="translate-replies-back-to-the-users-language"></a>Перевод ответов на язык пользователя

Также можно перевести ответы на язык пользователя, установив значение последнего параметра конструктора как `true`.

# <a name="ctabcstranslateback"></a>[C#](#tab/cstranslateback)
```cs
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "TRANSLATION-SUBSCRIPTION-KEY", true));
```

# <a name="javascripttabjstranslateback"></a>[JavaScript](#tab/jstranslateback)
```javascript
// Use language recognition to detect the user's language from their message, instead of providing helper callbacks.
// Last parameter indicates that we'll translate replies back to the user's language
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);
```

---

## <a name="run-the-bot-to-see-replies-in-the-users-language"></a>Запуск бота для отображения ответов на языке пользователя

Запустите бот и введите несколько сообщений на других языках. Вы увидите, что бот определяет язык пользователя и переводит на него ответ.

![Бот определяет язык и переводит на него ответы](./media/how-to-bot-translate/bot-detects-language-translates-response.png)


## <a name="adding-logic-for-detecting-or-changing-the-user-language"></a>Добавление логики для определения или изменения языка пользователя

Вместо того чтобы позволить пакету SDK для Bot Builder автоматически определять язык пользователя, можно предоставить обратные вызовы, чтобы добавить собственную логику для определения языка пользователя или определения того, когда язык пользователя был изменен.

> [!TIP] 
> Пример бота, который реализует обратные вызовы для обнаружения и изменения языка пользователя, описан в примере перевода пакета SDK. 

В следующем примере обратный вызов `CheckUserChangedLanguage` проверяет сообщение конкретного пользователя на изменение языка. 

# <a name="ctabcschangelanguage"></a>[C#](#tab/cschangelanguage)
```cs
// In Startup.cs
middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", null, TranslatorLocaleHelper.GetActiveLanguage, TranslatorLocaleHelper.CheckUserChangedLanguage));

// Add a file TranslatorLocalHelper.cs with the following:

    class CurrentUserState
    {
        public string Language { get; set; }
    }

    public static void SetLanguage(ITurnContext context, string language) => context.GetConversationState<CurrentUserState>().Language = language;

    public static bool IsSupportedLanguage(string language) => _supportedLanguages.Contains(language);

    public static async Task<bool> CheckUserChangedLanguage(ITurnContext context)
    {
        bool changeLang = false;
        
        // use a specific message from user to change language
        if (context.Activity.Type == ActivityTypes.Message)
        {
            var messageActivity = context.Activity.AsMessageActivity();
            if (messageActivity.Text.ToLower().StartsWith("set my language to"))
            {
                changeLang = true;
            }
            if (changeLang)
            {
                var newLang = messageActivity.Text.ToLower().Replace("set my language to", "").Trim();
                if (!string.IsNullOrWhiteSpace(newLang)
                        && IsSupportedLanguage(newLang))
                {
                    SetLanguage(context, newLang);
                    await context.SendActivity($@"Changing your language to {newLang}");
                }
                else
                {
                    await context.SendActivity($@"{newLang} is not a supported language.");
                }
                // return true to intercept message from further processesing
                return true;
            }
        }

        return false;
    }

    public static string GetActiveLanguage(ITurnContext context)
    {

        if (context.Activity.Type == ActivityTypes.Message
            && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Language != null)
        {
            return context.GetConversationState<CurrentUserState>().Language;
        }

        return "en";
    }

```

# <a name="javascripttabjschangelanguage"></a>[JavaScript](#tab/jschangelanguage)
```javascript
// When the user inputs 'set my language to fr'
// The bot will automatically change all text to French
async function setUserLanguage(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my language to')) {
        state.language = context.activity.text.toLowerCase().replace('set my language to', '').trim();
        await context.sendActivity(`Setting your language to ${state.language}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR MICROSOFT TRANSLATOR API KEY>",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    setUserLanguage: setUserLanguage,
    translateBackToUserLanguage: true
});
adapter.use(languageTranslator);

// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    // Route received activity to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`You said: ${context.activity.text}`)
        }
    });
});

```

---

## <a name="combining-luis-or-qna-with-translation"></a>Совместное использование LUIS или QnA при работе с переводом

При совместном использовании служб в боте, например, LUIS или QnA Maker, добавьте ПО промежуточного слоя перевода для того, чтобы сообщения переводились перед их передачей в другое ПО промежуточного слоя, которое ожидает сообщение на языке, который бот использует по умолчанию.

# <a name="ctabcslanguageluis"></a>[C#](#tab/cslanguageluis)
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(_ => Configuration);
    services.AddBot<AiBot>(options =>
    {
        options.CredentialProvider = new ConfigurationCredentialProvider(Configuration);
        var middleware = options.Middleware;

        // Add translation middleware
        // The first parameter is a list of languages the bot recognizes. Input in these language won't be translated.
        middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR MICROSOFT TRANSLATOR API KEY>", false));

        // Add LUIS middleware after translation middleware to pass translated messages to LUIS
        middleware.Add(
            new LuisRecognizerMiddleware(
                new LuisModel("luisAppId", "subscriptionId", new Uri("luisModelBaseUrl"))));
    });
}
```

# <a name="javascripttabjslanguageluis"></a>[JavaScript](#tab/jslanguageluis)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "xxxxxx",
    noTranslatePatterns: new Set(),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false
});
adapter.use(languageTranslator);

// Add Luis recognizer middleware
const luisRecognizer = new LuisRecognizer({
    appId: "xxxxxx",
    subscriptionKey: "xxxxxxx"
});
adapter.use(luisRecognizer);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            let results = luisRecognizer.get(context);
            for (const intent in results.intents) {
                await context.sendActivity(`intent: ${intent.toString()} : ${results.intents[intent.toString()]}`)
            }
        }
    });
});

```

---

В коде бота результаты LUIS зависят от входных данных, которые уже были переведены на язык, который бот использует по умолчанию. Чтобы проверить результаты, полученные приложением LUIS, попробуйте изменить код бота.

```cs
public async Task OnTurn(ITurnContext context)
{
    switch (context.Activity.Type)
    {
        case ActivityTypes.Message:
            // check LUIS results
            var luisResult = context.Services.Get<RecognizerResult>(LuisRecognizerMiddleware.LuisRecognizerResultKey);
            (string key, double score) topItem = luisResult.GetTopScoringIntent();
            await context.SendActivity($"The top intent was: '{topItem.key}', with score {topItem.score}");

            await context.SendActivity($"You sent '{context.Activity.Text}'");

        case ActivityTypes.ConversationUpdate:
            foreach (var newMember in context.Activity.MembersAdded)
            {
                if (newMember.Id != context.Activity.Recipient.Id)
                {
                    await context.SendActivity("Hello and welcome to the echo bot.");
                }
            }
            break;
        }
    }  
}          
```

## <a name="bypass-translation-for-specified-patterns"></a>Пропуск перевода для указанных шаблонов
Возможно, некоторые слова не нужно переводить, например, собственные имена. Чтобы указать шаблоны, которые не следует переводить, можно предоставить регулярные выражения. Например, если пользователь скажет: "My name is ..." на языке, который бот не использует по умолчанию и при этом нужно избежать перевода имени, то чтобы это указать, можно использовать шаблон.

# <a name="ctabcsbypass"></a>[C#](#tab/csbypass)
```cs
    // Startup.cs

    // Pattern representing input to not translate
    Dictionary<string, List<string>> patterns = new Dictionary<string, List<string>>();
    // single pattern for fr language, to avoid translating what follows "my name is"
    patterns.Add("fr", new List<string> { "mon nom est (.+)" });
    
    middleware.Add(new TranslationMiddleware(new string[] { "en" }, "<YOUR API KEY>", patterns, TranslatorLocaleHelper.GetActiveLanguage, TranslatorLocaleHelper.CheckUserChangedLanguage));

```
<!-- TODO: ADD more explanation (both of these callbacks are run every time), fix image by debugging regex for l'etat -->

# <a name="javascripttabjsbypass"></a>[JavaScript](#tab/jsbypass)
```javascript
// Add language translator middleware
const languageTranslator = new LanguageTranslator({
    translatorKey: "<YOUR API KEY>",
    // single pattern for fr language, to avoid translating what follows "my name is"
    noTranslatePatterns: new Set("fr", "/mon nom est (.+)/"),
    nativeLanguages: ['en'],
    translateBackToUserLanguage: false,
    
});
adapter.use(languageTranslator);

```

---

![Бот не переводит шаблон](./media/how-to-bot-translate/bot-no-translate-name-fr.png)

## <a name="localize-dates"></a>Локализация дат

Для локализации дат можно добавить `LocaleConverterMiddleware`. Например, если вы знаете, что бот ожидает даты в формате `MM/DD/YYYY`, а пользователи с другими языковыми стандартами могут вводить даты в формате `DD/MM/YYYY`, локальный преобразователь ПО промежуточного слоя может автоматически преобразовывать даты в формат, приемлемый для бота.

> [!NOTE]
> Локальный преобразователь ПО промежуточного слоя призван преобразовывать только даты. Он не имеет сведений о результатах перевода ПО промежуточного слоя. Используя ПО промежуточного слоя вместе с локальным преобразователем для перевода, необходимо быть осторожным. ПО промежуточного слоя для перевода переводит некоторые даты, которые были введены вместе с текстом, в то время как даты, введенные без текста, не переводятся.

Например, на следующем рисунке изображен бот, который выводит обратно на экран входные данные пользователя после перевода с английского языка на французский. Он использует только `TranslationMiddleware` (без `LocaleConverterMiddleware`).

![Бот переводит даты без их преобразования](./media/how-to-bot-translate/locale-date-before.png)

Далее приведен тот же бот, в который был добавлен параметр `LocaleConverterMiddleware`.

![Бот переводит даты без их преобразования](./media/how-to-bot-translate/locale-date-after.png)

Языковые преобразователи могут поддерживать языковые стандарты английского, французского, немецкого и китайского языков. <!-- TODO: ADD DETAIL ABOUT SUPPORTED LOCALES -->

# <a name="ctabcstranslation"></a>[C#](#tab/cstranslation)
```cs
// Startup.cs
// Add locale converter middleware
middleware.Add(new LocaleConverterMiddleware(TranslatorLocaleHelper.GetActiveLocale, TranslatorLocaleHelper.CheckUserChangedLocale, "en-us", LocaleConverter.Converter));

// TranslatorLocaleHelper.cs
        public static async Task<bool> CheckUserChangedLocale(ITurnContext context)
        {
            bool changeLocale = false;//logic implemented by developper to make a signal for language changing 
            //use a specific message from user to change language
            if (context.Activity.Type == ActivityTypes.Message)
            {
                var messageActivity = context.Activity.AsMessageActivity();
                if (messageActivity.Text.ToLower().StartsWith("set my locale to"))
                {
                    changeLocale = true;
                }
                if (changeLocale)
                {
                    var newLocale = messageActivity.Text.ToLower().Replace("set my locale to", "").Trim(); //extracted by the user using user state 
                    if (!string.IsNullOrWhiteSpace(newLocale)
                            && IsSupportedLanguage(newLocale))
                    {
                        SetLocale(context, newLocale);
                        await context.SendActivity($@"Changing your language to {newLocale}");
                    }
                    else
                    {
                        await context.SendActivity($@"{newLocale} is not a supported locale.");
                    }
                    //intercepts message
                    return true;
                }
            }

            return false;
        }
        public static string GetActiveLocale(ITurnContext context)
        {
            if (currentLocale != null)
            {
                //the user has specified a different locale so update the bot state
                if (context.GetConversationState<CurrentUserState>() != null
                    && currentLocale != context.GetConversationState<CurrentUserState>().Locale)
                {
                    SetLocale(context, currentLocale);
                }
            }
            if (context.Activity.Type == ActivityTypes.Message
                && context.GetConversationState<CurrentUserState>() != null && context.GetConversationState<CurrentUserState>().Locale != null)
            {
                return context.GetConversationState<CurrentUserState>().Locale;
            }

            return "en-us";
        }
```

# <a name="javascripttabjstranslation"></a>[JavaScript](#tab/jstranslation)

<!-- this snippet only works if the user doesn't actually try to change their locale.  Emailed Mostafa about the issue 
It should change the locale after you type in 'set my locale to....' -->

```javascript
// Delegates for getting and setting user locale
function getUserLocale(context) {
    const state = conversationState.get(context)
    if (state.locale == undefined) {
        return 'en-us';
    } else {
        return state.locale;
    }
}

// Function to change the user's locale.
async function setUserLocale(context) {
    let state = conversationState.get(context)
    if (context.activity.text.toLowerCase().startsWith('set my locale to')) {        
        state.locale = context.activity.text.toLowerCase().replace('set my locale to', '').trim();
        await context.sendActivity(`Setting your locale to ${state.locale}`);
        return Promise.resolve(true);
    } else {
        return Promise.resolve(false);
    }
}

// Add locale converter middleware
// Will convert input to fr-fr
const localeConverter = new LocaleConverter({
    toLocale: 'fr-fr',
    setUserLocale: setUserLocale,
    getUserLocale: getUserLocale
});
adapter.use(localeConverter);


// Listen for incoming requests 
server.post('/api/messages', (req, res) => {
    // Route received request to adapter for processing
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type != 'message') {
            await context.sendActivity(`[${context.activity.type} event detected]`);
        } else {
            await context.sendActivity(`The message that you sent was:  ${context.activity.text}`)
        }
    });
});

```

---

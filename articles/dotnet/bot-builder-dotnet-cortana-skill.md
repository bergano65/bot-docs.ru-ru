---
title: Создание навыка Кортаны с помощью .NET | Документация Майкрософт
description: Основные понятия для создания навыка Кортаны в пакете SDK Bot Builder для .NET.
keywords: Bot Framework, Cortana skill, speech, .NET, Bot Builder, SDK, key concepts, core concepts
author: DeniseMak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: a3d049e349a86437f8c342df1702281600aeddd4
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904396"
---
# <a name="build-a-speech-enabled-bot-with-cortana-skills"></a>Создание бота с поддержкой речи с навыками Кортаны

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-cortana-skill.md)
> - [Node.js](../nodejs/bot-builder-nodejs-cortana-skill.md)


Пакет SDK Bot Builder для .NET позволяет создать бот с поддержкой речи путем его подключения к каналу Кортаны в качестве навыка Кортаны. 


> [!TIP]
> Дополнительные сведения о том, что такое навык и как его можно использовать, см. в статье [Cortana Skills Kit][CortanaGetStarted] (Набор навыков Кортаны).

Создание навыка Кортаны с использованием Bot Framework не требует глубоких знаний о Кортане и предусматривает прежде всего создание бота. Одним из вероятных ключевых отличий от других ботов, которые вы могли создавать в прошлом, является то, что Кортана имеет как визуальный, так и звуковой компонент. Для визуального компонента Кортана предоставляет область холста для отображения содержимого, например карточек. Для звукового компонента необходимо предоставить текст или SSML в сообщениях бота, которые Кортана зачитывает пользователю, предоставляя боту голос. 

> [!NOTE]
> Помощник Кортана доступен на различных устройствах. У одних есть экран, в то время как другие, например автономный динамик, могут и не иметь его. Следует убедиться, что бот способен работать в обоих сценариях. Чтобы узнать, как проверить сведения об устройстве, см. статью [Get Cortana's Channel Data][CortanaSpecificEntities] (Получение данных о канале Кортаны).

## <a name="adding-speech-to-your-bot"></a>Добавление речевых функций для бота

Речевые сообщения бота представлены в виде SSML (Speech Synthesis Markup Language — язык разметки синтеза речи). Пакет SDK Bot Builder позволяет включать SSML в ответы бота, чтобы контролировать его высказывания в дополнение к тому, что он показывает.  Можно также контролировать состояние микрофона Кортаны, указав боту, как реагировать на входные данные пользователя — принимать, ожидать или игнорировать.

Задайте свойство`Speak` объекта `IMessageActivity`, чтобы указать, какое сообщение озвучит Кортана. Если указать обычный текст, Кортана самостоятельно определяет манеру произношения слов. 

```cs
Activity reply = activity.CreateReply("This is the text that Cortana displays."); 
reply.Speak = "This is the text that Cortana will say.";
```

Если вы хотите сами регулировать высоту голоса, тембр и выразительность речи, выберите для свойства `Speak` формат [Speech Synthesis Markup Language (SSML)](http://www.w3.org/TR/speech-synthesis/) (Язык разметки синтеза речи).  

В следующем примере кода указано, что слово "Text" (Текст) должно произноситься с умеренным ударением:
```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "<speak version=\"1.0\" xmlns=\"http://www.w3.org/2001/10/synthesis\" xml:lang=\"en-US\">This is the <emphasis level=\"moderate\">text</emphasis> that will be spoken.</speak>";
```


Свойство **InputHint** позволяет указать помощнику Кортана, ожидает ли бот входные данные. По умолчанию используются значения **ExpectingInput** для запросов и **AcceptingInput** для других типов ответов.


| Значение | ОПИСАНИЕ |
|------|------|
| **AcceptingInput** | Бот пассивно готов к вводу, но не ожидает ответа. Кортана принимает входные данные от пользователя, если пользователь удерживает кнопку микрофона.|
| **ExpectingInput** | Указывает на то, что бот активно ожидает ответа от пользователя. Кортана слушает, что пользователь говорит в микрофон.  |
| **IgnoringInput** | Кортана игнорирует входные данные. Бот может отправить эту подсказку при активной обработке запроса, и будет игнорировать входные данные от пользователей до тех пор, пока запрос не будет выполнен.  |

<!-- TODO: tip about time limit and batching -->

В этом примере показано, как сообщить Кортане, что ожидается ввод данных пользователем. Микрофон останется открытым.
```cs
// Add an InputHint to let Cortana know to expect user input
Activity reply = activity.CreateReply("This is the text that will be displayed."); 
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.ExpectingInput;
```



## <a name="display-cards-in-cortana"></a>Отображение карточек в Кортане

Помимо произношения ответов, Кортана также может отображать вложения карточек. Кортана поддерживает следующие функциональные карточки:

| Тип карточки | ОПИСАНИЕ |
|----|----|
| [HeroCard][heroCard] | Карточка, которая обычно содержит одно большое изображение, одну или несколько кнопок и текст. |
| [ThumbnailCard][thumbnailCard] | Карточка, которая обычно содержит один эскиз, одну или несколько кнопок и текст. |
| [ReceiptCard][receiptCard] | Карточка, с помощью которой бот выдает квитанцию пользователю. Обычно она содержит список элементов, включаемых в квитанцию, налог и общую сумму, а также другую информацию. |
| [SignInCard][signinCard] | Карточка, в которой бот запрашивает вход пользователя. Обычно она содержит текст и одну или несколько кнопок, которые можно нажать, чтобы начать процесс входа. |


Сведения о том, как эти карточки выглядят в Кортане, см. в разделе [Card design best practices][CardDesign] (Рекомендации по проектированию карточек для Кортаны). Пример того, как использовать функциональные карточки в боте, см. в разделе [Add rich card attachments to messages](bot-builder-dotnet-add-rich-card-attachments.md) (Добавление расширенных карточек во вложения сообщений). 

<!--
The following code demonstrates how to add the `Speak` and `InputHint` properties to a message containing a `HeroCard`.
-->


## <a name="sample-rollerskill"></a>Пример: RollerSkill
Код в следующих разделах относится к примеру навыка Кортаны, который предназначен для того, чтобы бросать кости. Скачайте полный код для бота из [репозитория BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples/).

Можно вызвать навык, назвав [имя вызова][InvocationNameGuidelines] помощнику Кортане. После [подключения бота к каналу Кортаны][CortanaChannel] и регистрации его в качестве навыка Кортана можно вызвать навык броска, сказав Кортане: "Ask Roller" (Вызвать навык броска) или "Ask Roller to Roll Dice" (Вызвать навык броска и бросить кости).

### <a name="explore-the-code"></a>Обзор кода



Для вызова соответствующих диалогов обработчики действий, определенные в файле `RootDispatchDialog.cs`, используют регулярные выражения для сопоставления входных данных пользователя. Например, в следующем примере обработчик срабатывает, если пользователь говорит что-то вроде "I'd like to roll some dice" (Хочу бросить кости). Синонимы включены в регулярное выражение, чтобы похожие высказывания запускали диалог.
```cs
        [RegexPattern("(roll|role|throw|shoot).*(dice|die|dye|bones)")]
        [RegexPattern("new game")]
        [ScorableGroup(1)]
        public async Task NewGame(IDialogContext context, IActivity activity)
        {
            context.Call(new CreateGameDialog(), AfterGameCreated);
        }
```

В диалоге `CreateGameDialog` настраивается пользовательская игра для игры с ботом. С помощью `PromptDialog` он спрашивает у пользователя, сколько сторон должны иметь кости и сколько костей нужно бросить. Обратите внимание, что объект `PromptOptions`, используемый для инициализации запроса, содержит свойство `speak` для голосовой версии запроса.

```cs
    [Serializable]
    public class CreateGameDialog : IDialog<GameData>
    {
        public async Task StartAsync(IDialogContext context)
        {
            context.UserData.SetValue<GameData>(Utils.GameDataKey, new GameData());

            var descriptions = new List<string>() { "4 Sides", "6 Sides", "8 Sides", "10 Sides", "12 Sides", "20 Sides" };
            var choices = new Dictionary<string, IReadOnlyList<string>>()
             {
                { "4", new List<string> { "four", "for", "4 sided", "4 sides" } },
                { "6", new List<string> { "six", "sex", "6 sided", "6 sides" } },
                { "8", new List<string> { "eight", "8 sided", "8 sides" } },
                { "10", new List<string> { "ten", "10 sided", "10 sides" } },
                { "12", new List<string> { "twelve", "12 sided", "12 sides" } },
                { "20", new List<string> { "twenty", "20 sided", "20 sides" } }
            };

            var promptOptions = new PromptOptions<string>(
                Resources.ChooseSides,
                choices: choices,
                descriptions: descriptions,
                speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseSidesSSML))); // spoken prompt

            PromptDialog.Choice(context, this.DiceChoiceReceivedAsync, promptOptions);
        }

        private async Task DiceChoiceReceivedAsync(IDialogContext context, IAwaitable<string> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                int sides;
                if (int.TryParse(await result, out sides))
                {
                    game.Sides = sides;
                    context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
                }

                var promptText = string.Format(Resources.ChooseCount, sides);

                var promptOption = new PromptOptions<long>(promptText, choices: null, speak: SSMLHelper.Speak(Utils.RandomPick(Resources.ChooseCountSSML)));

                var prompt = new PromptDialog.PromptInt64(promptOption);
                context.Call<long>(prompt, this.DiceNumberReceivedAsync);
            }
        }

        private async Task DiceNumberReceivedAsync(IDialogContext context, IAwaitable<long> result)
        {
            GameData game;
            if (context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out game))
            {
                game.Count = await result;
                context.UserData.SetValue<GameData>(Utils.GameDataKey, game);
            }

            context.Done(game);
        }
    }
```

Диалог `PlayGameDialog` преобразует результаты, отображая их в карточке `HeroCard` и создавая голосовые сообщения для озвучивания с использованием метода `Speak`.

```cs
   [Serializable]
    public class PlayGameDialog : IDialog<object>
    {
        private const string RollAgainOptionValue = "roll again";

        private const string NewGameOptionValue = "new game";

        private GameData gameData;

        public PlayGameDialog(GameData gameData)
        {
            this.gameData = gameData;
        }

        public async Task StartAsync(IDialogContext context)
        {
            if (this.gameData == null)
            {
                if (!context.UserData.TryGetValue<GameData>(Utils.GameDataKey, out this.gameData))
                {
                    // User started session with "roll again" so let's just send them to
                    // the 'CreateGameDialog'
                    context.Done<object>(null);
                }
            }

            int total = 0;
            var randomGenerator = new Random();
            var rolls = new List<int>();

            // Generate Rolls
            for (int i = 0; i < this.gameData.Count; i++)
            {
                var roll = randomGenerator.Next(1, this.gameData.Sides);
                total += roll;
                rolls.Add(roll);
            }

            // Format rolls results
            var result = string.Join(" . ", rolls.ToArray());
            bool multiLine = rolls.Count > 5;

            var card = new HeroCard()
            {
                Subtitle = string.Format(
                    this.gameData.Count > 1 ? Resources.CardSubtitlePlural : Resources.CardSubtitleSingular,
                    this.gameData.Count,
                    this.gameData.Sides),
                Buttons = new List<CardAction>()
                {
                    new CardAction(ActionTypes.ImBack, "Roll Again", value: RollAgainOptionValue),
                    new CardAction(ActionTypes.ImBack, "New Game", value: NewGameOptionValue)
                }
            };

            if (multiLine)
            {
                card.Text = result;
            }
            else
            {
                card.Title = result;
            }

            var message = context.MakeMessage();
            message.Attachments = new List<Attachment>()
            {
                card.ToAttachment()
            };

            // Determine bots reaction for speech purposes
            string reaction = "normal";

            var min = this.gameData.Count;
            var max = this.gameData.Count * this.gameData.Sides;
            var score = total / max;
            if (score == 1)
            {
                reaction = "Best";
            }
            else if (score == 0)
            {
                reaction = "Worst";
            }
            else if (score <= 0.3)
            {
                reaction = "Bad";
            }
            else if (score >= 0.8)
            {
                reaction = "Good";
            }

            // Check for special craps rolls
            if (this.gameData.Type == "Craps")
            {
                switch (total)
                {
                    case 2:
                    case 3:
                    case 12:
                        reaction = "CrapsLose";
                        break;
                    case 7:
                        reaction = "CrapsSeven";
                        break;
                    case 11:
                        reaction = "CrapsEleven";
                        break;
                    default:
                        reaction = "CrapsRetry";
                        break;
                }
            }

            // Build up spoken response
            var spoken = string.Empty;
            if (this.gameData.Turns == 0)
            {
                spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"Start{this.gameData.Type}GameSSML"));
            }

            spoken += Utils.RandomPick(Resources.ResourceManager.GetString($"{reaction}RollReactionSSML"));

            message.Speak = SSMLHelper.Speak(spoken);

            // Increment number of turns and store game to roll again
            this.gameData.Turns++;
            context.UserData.SetValue<GameData>(Utils.GameDataKey, this.gameData);

            // Send card and bots reaction to user.
            message.InputHint = InputHints.AcceptingInput;
            await context.PostAsync(message);

            context.Done<object>(null);
        }
    }
```
## <a name="next-steps"></a>Дополнительная информация

Если бот запущен на локальном устройстве или развернут в облаке, можно вызвать его с помощью Кортаны. Сведения о действиях, необходимых для проверки навыка Кортаны, см. в разделе [Test a Cortana skill](../bot-service-debug-cortana-skill.md) (Тестирование навыка Кортаны).


## <a name="additional-resources"></a>Дополнительные ресурсы
* [Cortana Skills Kit][CortanaGetStarted] (Набор навыков Кортаны)
* [Добавление речи в сообщения](bot-builder-dotnet-text-to-speech.md)
* [Speech Synthesis Markup Language (SSML) reference][SSMLRef] (Справочник по SSML)
* [Voice design best practices for Cortana][VoiceDesign] (Рекомендации по проектированию речевых функций для Кортаны)
* [Card design best practices for Cortana][CardDesign] (Рекомендации по проектированию карточек для Кортаны)
* [Cortana Dev Center][CortanaDevCenter] (Центр разработки Кортаны)
* [Testing and debugging best practices for Cortana][Cortana-TestBestPractice] (Рекомендации по тестированию и отладке для Кортаны)
* <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочник по пакету SDK Bot Builder для .NET</a>

[CortanaGetStarted]: /cortana/getstarted
[BFPortal]: https://dev.botframework.com/

[SSMLRef]: https://aka.ms/cortana-ssml
[CortanaDevCenter]: https://developer.microsoft.com/en-us/cortana

[CortanaSpecificEntities]: https://aka.ms/lgvcto
[CortanaAuth]: https://aka.ms/vsdqcj

[InvocationNameGuidelines]: https://aka.ms/cortana-invocation-guidelines  
[VoiceDesign]: https://aka.ms/cortana-design-voice
[CardDesign]: https://aka.ms/cortana-design-card
[Cortana-Debug]: https://aka.ms/cortana-enable-debug
[Cortana-Publish]: https://aka.ms/cortana-publish


[CortanaTry]: https://aka.ms/try-cortana-bot
[CortanaChannel]: https://aka.ms/bot-cortana-channel
[Cortana-TestBestPractice]: https://aka.ms/cortana-test-best-practice

[heroCard]: /dotnet/api/microsoft.bot.connector.herocard

[thumbnailCard]: /dotnet/api/microsoft.bot.connector.thumbnailcard 

[receiptCard]: /dotnet/api/microsoft.bot.connector.receiptcard 

[signinCard]: /dotnet/api/microsoft.bot.connector.signincard 



---
title: Сведения о работе ботов — Служба Azure Bot
description: Описание работы с действиями и HTTP в пакете SDK Bot Framework.
keywords: conversation flow, turn, bot conversation, dialogs, prompts, waterfalls, dialog set
author: johnataylor
ms.author: johtaylo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/31/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: d6ea339d008c54c8a60dc20dd9c5252b31e06142
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071862"
---
# <a name="how-bots-work"></a>Принципы работы бота

[!INCLUDE[applies-to](../includes/applies-to.md)]

Бот — это приложение, с которым пользователи взаимодействуют, общаясь с помощью текста, графики (карт или изображений) или речи. Каждое взаимодействие между пользователем и ботом создает некоторое *действие*. Служба Bot Framework, которая входит в состав службы Azure Bot, передает информацию между пользовательскими приложениями (например, Facebook, Skype, Slack и другими *каналами*) и ботом. Каждый канал может передавать в отправляемых событиях дополнительные сведения. Прежде чем создавать ботов, важно хорошо разобраться в использовании объектов действия для общения с пользователями бота. Для начала мы рассмотрим действия, которые передаются при выполнении простого бота проверки связи.

![Схема действий](media/bot-builder-activity.png)

Здесь представлены действия двух типов: *обновление диалога* и *сообщение*.

Служба Bot Framework Service может отправлять события обновления беседы при добавлении к общению новых участников. Например, при начале беседы с эмулятором Bot Framework вы увидите два действия обновления диалога (один для подключаемого пользователя и второй для подключаемого бота). Чтобы различить эти действия обновления диалога, проверьте наличие других участников, кроме самого бота, в свойстве *members added* (добавленные участники).

Действие сообщения передает между сторонами информацию общения. В нашем примере бота проверки связи события сообщений передают простой текст, который будет отображаться в выбранном канале. Кроме того, действие сообщения может содержать голосовые сообщения, предлагаемые действия или карты для отображения.

В нашем примере создается бот и отправляется действие сообщения в ответ на полученное входящее действие сообщения. Но для реальной работы бот может использовать другие методы реагирования на полученные действия сообщения. Кроме того, часто боты отправляют действия сообщения с приветственным текстом в ответ на действие обновления диалога. См. дополнительные сведения о том, [как приветствовать пользователя](bot-builder-welcome-user.md).

### <a name="http-details"></a>Сведения об HTTP

Действия поступают в бот из службы Bot Framework через запросы HTTP POST. В ответ на входящий запрос POST бот возвращает код состояния HTTP 200. Действия, отправляемые из бота в канал, отправляются в отдельном запросе POST к службе Bot Framework. В ответ на него также поступает код состояния HTTP 200.

Протокол не определяет порядок отправки запросов POST подтверждений на них. Но на распространенных платформах служб HTTP такие запросы обычно являются вложенными, то есть исходящий HTTP-запрос выполняется ботом в процессе обработки входящего HTTP-запроса. Этот процесс представлен на схеме выше. Поскольку здесь есть два раздельных HTTP-подключения с откликами на них, модель безопасности должна учитывать оба из них.

### <a name="defining-a-turn"></a>Определение включения

При общении люди обычно говорят по очереди. Как правило, бот реагирует на вводимые пользователем данные. В пакете SDK Bot Framework _ход_ состоит из входящего действия пользователя и всех действий, которые бот немедленно возвращает пользователю в ответ. Ход можно рассматривать как обработку, связанную с поступлением того или иного действия.

Объект *контекста шага* предоставляет сведения о действии. К таким объектам относятся, например, идентификаторы канала, отправителя и получателя, а также другие необходимые данные для обработки действия. Кроме того, можно добавлять в объект контекста шага сведения при обработке шага на разных уровнях бота.

Контекст шага является одной из важнейших абстракций в пакете SDK. Он предоставляет всем компонентам ПО промежуточного слоя и логике приложения не только само входящее действие, но и механизм для отправки исходящих действий.

## <a name="the-activity-processing-stack"></a>Стек обработки действия

Теперь давайте подробнее рассмотрим представленную выше схему, начиная с прибытия действия сообщения.

![Стек обработки действия](media/bot-builder-activity-processing-stack.png)

В приведенном выше примере в ответ на действие сообщения бот отправляет другое действие сообщения, помещая в него текст из первого сообщения. Обработка начинается с поступления на веб-сервер запроса HTTP POST, в котором сведения о действии передаются в формате полезных данных JSON. В C# этот процесс обычно выполняется в проекте ASP.NET, а для проектов JavaScript Node.js часто используются такие платформы, как Express и Restify.

Встроенный компонент пакета SDK, который называется *адаптером*, является ядром среды выполнения SDK. Действие передается в тексте запроса HTTP POST в формате JSON. Этот код JSON десериализуется в объект Activity (Действие), который обрабатывается адаптером с помощью вызова метода *process activity*. При получении действия адаптер создает *turn context* (контекст шага) и вызывает ПО промежуточного слоя.

Как упоминалось выше, контекст шага позволяет ботам отправлять исходящие действия, чаще всего в ответ на входящее действие. Для этого контекст шага предоставляет такие методы реагирования, как _отправка, обновление и удаление действия_. Каждый метод ответа выполняется в асинхронном процессе.

[!INCLUDE [alert-await-send-activity](../includes/alert-await-send-activity.md)]

## <a name="activity-handlers"></a>Обработчики событий

Когда бот получает действие, он передает его в *обработчики действий*. На самом деле есть один базовый обработчик, который называется *обработчиком шагов*. Все действия передаются в него. Затем этот обработчик вызывает обработчика того действия, которое он получил.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Например, если бот получает действие сообщения, обработчик шагов оценивает это входящее действие, а затем отправляет его в обработчик действий `OnMessageActivityAsync`.

У созданного бота логика обработки сообщений и реагирования на них будет связываться с этим обработчиком `OnMessageActivityAsync`. Аналогичным образом, логика обработки участников, добавляемых в диалог, будет связываться с вашим обработчиком `OnMembersAddedAsync`, который вызывается при каждом добавлении участника в диалог.

Чтобы реализовать логику для этих обработчиков, вам нужно переопределить эти методы в боте, как описано в разделе [Логика ботов](#bot-logic) ниже. Базовой реализации для каждого из этих обработчиков нет, поэтому просто добавляйте логику, используемую при переопределении.

В некоторых ситуациях требуется переопределить основной обработчик шагов, например, для [сохранения состояния](bot-builder-concept-state.md) в конце шага. Перед этим обязательно вызовите `await base.OnTurnAsync(turnContext, cancellationToken);`, чтобы убедиться, что базовая реализация `OnTurnAsync` выполняется перед дополнительным кодом. Эта базовая реализация, помимо прочего, отвечает за вызов остальных обработчиков действий, таких как `OnMessageActivityAsync`.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

`ActivityHandler` в JavaScript использует модель передатчика и прослушивателя событий.
Например, метод `onMessage` позволяет зарегистрировать прослушиватель событий для обработки сообщений. Вы можете зарегистрировать несколько прослушивателей. Когда бот получает действие message, обработчик действий получает информацию о нем и отправляет каждому прослушивателю действий `onMessage` в том порядке, в котором они были зарегистрированы.

При создании бота именно в прослушиватели `onMessage` следует помещать логику обработки сообщений и реагирования на них. Аналогичным образом, логика обработки участников, добавляемых в диалог, помещается в прослушиватели `onMembersAdded`, которые вызываются при каждом добавлении участника в диалог.
Чтобы добавить прослушиватели, их следует зарегистрировать в боте, как показано ниже в разделе [Логика бота](#bot-logic). Добавьте логику бота для каждого прослушивателя, а в конце кода **не забудьте вызвать `next()`** . Вызов `next()` позволяет запустить следующего прослушивателя.

Обеспечьте [сохранение состояния](bot-builder-concept-state.md) перед завершением реплики. Для этого переопределите метод обработчика действий `run` и сохраните состояние после завершения родительского метода `run`.

В некоторых ситуациях не требуется переопределять основной обработчик шагов, поэтому делайте это с осторожностью.
Для этого существует специальный обработчик `onDialog`. Обработчик `onDialog` выполняется в конце шага после выполнения остальных обработчиков. Он не привязан к определенным типам действий. Как и для всех упомянутых выше обработчиков, обязательно вызовите `next()`, чтобы закончить завершающую часть процесса.

# <a name="pythontabpython"></a>[Python](#tab/python)

Например, если бот получает действие сообщения, обработчик шагов оценивает это входящее действие, а затем отправляет его в обработчик действий `on_message_activity`.

У созданного бота логика обработки сообщений и реагирования на них будет связываться с этим обработчиком `on_message_activity`. Аналогичным образом, логика обработки участников, добавляемых в диалог, будет связываться с вашим обработчиком `on_members_added`, который вызывается при каждом добавлении участника в диалог.

Чтобы реализовать логику для этих обработчиков, вам нужно переопределить эти методы в боте, как описано в разделе [Логика ботов](#bot-logic) ниже. Базовой реализации для каждого из этих обработчиков нет, поэтому просто добавляйте логику, используемую при переопределении.

В некоторых ситуациях требуется переопределить основной обработчик шагов, например, для [сохранения состояния](bot-builder-concept-state.md) в конце шага. Перед этим обязательно вызовите `await super().on_turn(turnContext);`, чтобы убедиться, что базовая реализация `on_turn` выполняется перед дополнительным кодом. Эта базовая реализация, помимо прочего, отвечает за вызов остальных обработчиков действий, таких как `on_message_activity`.

---

## <a name="middleware"></a>ПО промежуточного слоя

ПО промежуточного слоя действует так же, как любое средство обработки сообщений, и состоит из линейного набора компонентов, каждый из которых выполняется в строгом порядке и получает возможность применить некоторую логику реагирования на событие. Последним этапом конвейера ПО промежуточного слоя является обратный вызов обработчика шагов из класса бота, регистрируемого приложением с использованием метода *действия обработки* адаптера. Как правило, обработчик шагов представляет собой `OnTurnAsync` в C# и `onTurn` в JavaScript.

Обработчик шагов принимает в качестве аргумента контекст шага. Заключенная в обработчике шагов логика приложении обычно обрабатывает содержимое входящего действия и создает в ответ одно или несколько действий, которые затем отправляет с помощью функции *send activity* для контекста шага. Вызов *send activity* для контекста шага приводит к вызову компонентов ПО промежуточного слоя для исходящих действий. Компоненты ПО промежуточного слоя выполняются до и после функции обработчика шагов бота. Такое выполнение является конструктивно вложенным и иногда называется "матрешкой". Дополнительные сведения о ПО промежуточного слоя см. [здесь](~/v4sdk/bot-builder-concept-middleware.md).

## <a name="bot-structure"></a>Структура бота

В следующих разделах мы рассмотрим _основные части_ EchoBot, которые можно легко создать с помощью шаблонов для [**C#** ](../dotnet/bot-builder-dotnet-sdk-quickstart.md) и [**JavaScript**](../javascript/bot-builder-javascript-quickstart.md).

<!--Need to add section calling out the controller in code, and explaining it further-->

Бот — это веб-приложение, и мы предоставляем шаблоны для каждого языка.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Шаблон VSIX создает веб-приложение [ASP.NET MVC Core](https://dotnet.microsoft.com/apps/aspnet/mvc). Основные компоненты для [ASP.NET](https://docs.microsoft.com/aspnet/core/fundamentals/index?view=aspnetcore-2.1&tabs=aspnetcore2x) содержат похожий код в таких файлах, как **Program.cs** и **Startup.cs**. Эти файлы являются обязательными для всех веб-приложений и не зависят от конкретного бота.

### <a name="appsettingsjson-file"></a>Файл appsettings.json

В файле **appsettings.json** указываются сведения о конфигурации для вашего бота, в том числе идентификатор приложения и пароль. Если вы применяете некоторые технологии или используете этот бот в рабочей среде, нужно добавить в эту конфигурацию определенные ключи или URL-адрес. Сейчас для Echo Bot не нужно добавлять такие сведения. Можете не указывать идентификатор приложения и пароль на этом этапе.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

<!-- TODO: Update this aka link to point to samples/javascript_nodejs/02.echobot (instead of samples/javascript_nodejs/02.a.echobot) once work-in-progress is merged into master. -->

Генератор Yeoman создает веб-приложение с типом [restify](http://restify.com/). Краткое руководство по быстрому началу работы в документации по restify содержит приложение, аналогичное созданному файлу **index.js**. Здесь описаны некоторые ключевые файлы, созданные с помощью шаблона. Код некоторых из них мы не приводим, но его можно просмотреть при выполнении бота или в примере [Node.js EchoBot](https://aka.ms/js-echobot-sample).

### <a name="packagejson"></a>package.json

В файле **package.json** указываются зависимости и их связанные версии для бота. Все они настраиваются и системой и согласно шаблону.

### <a name="env-file"></a>Файл ENV

В файле **ENV** указываются сведения о конфигурации для вашего бота, в том числе и номер порта, идентификатор приложения и пароль. Если вы применяете некоторые технологии или используете этот бот в рабочей среде, нужно добавить в эту конфигурацию определенные ключи или URL-адрес. Сейчас для Echo Bot не нужно добавлять такие сведения. Можете не указывать идентификатор приложения и пароль на этом этапе.

Чтобы использовать файл конфигурации **ENV**, нужно добавить в шаблон пакет.  Сначала получите пакет `dotenv` из npm:

`npm install dotenv`

# <a name="pythontabpython"></a>[Python](#tab/python)

### <a name="requirementstxt"></a>requirements.txt

В файле **requirements.txt** указываются зависимости и их связанные версии для бота.  Все это определяется системой и шаблоном.

Зависимости следует устанавливать с помощью `pip install -r requirements.txt`.

### <a name="configpy"></a>config.py

В файле **config.py** указываются сведения о конфигурации для вашего бота, включая номер порта, идентификатор приложения и пароль. Если вы применяете некоторые технологии или используете этот бот в рабочей среде, нужно добавить в эту конфигурацию определенные ключи или URL-адрес. Сейчас для Echo Bot не нужно добавлять такие сведения. Можете не указывать идентификатор приложения и пароль на этом этапе.

---

### <a name="bot-logic"></a>Логика бота

Логика бота обрабатывает входящие действия из одного или нескольких каналов и создает исходящие действия в ответ.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Основная логика бота определяется в коде бота. Здесь это `Bots/EchoBot.cs`. `EchoBot` является производным от `ActivityHandler` (является производным от интерфейса `IBot`). `ActivityHandler` определяет различные обработчики для разных типов действий, например указанных здесь `OnMessageActivityAsync` и `OnMembersAddedAsync`. Эти методы являются защищенными, но могут быть перезаписаны с учетом наследования от `ActivityHandler`.

Ниже описаны обработчики, определенные в `ActivityHandler`:

| Событие | Обработчик | Описание |
| :-- | :-- | :-- |
| Любой тип полученного действия | `OnTurnAsync` | Вызывает один из других обработчиков на основе типа полученного действия. |
| Полученное действие сообщения | `OnMessageActivityAsync` | Переопределите для обработки действия `message`. |
| Полученное действие обновления диалога | `OnConversationUpdateActivityAsync` | В действии `conversationUpdate` вызывает обработчик, если участники не добавлены в бота или покидают беседу. |
| Не добавленные в бота участники, которые присоединяются к беседе | `OnMembersAddedAsync` | Переопределите для обработки участников, присоединяющихся к беседе. |
| Не добавленные в бота участники, которые покидают беседу | `OnMembersRemovedAsync` | Переопределите для обработки участников, покидающих беседу. |
| Полученное действие события | `OnEventActivityAsync` | В действии `event` вызывает обработчик для события определенного типа. |
| Полученное действие события ответа маркера | `OnTokenResponseEventAsync` | Переопределите для обработки событий ответа маркера. |
| Полученное действие события другого ответа | `OnEventAsync` | Переопределите для обработки событий других типов. |
| Получение действия реакции на сообщение | `OnMessageReactionActivityAsync` | При получении действия `messageReaction` вызывает обработчик, если в сообщении были добавлены или удалены одна или несколько реакций. |
| Добавление реакции на сообщение в сообщение | `OnReactionsAddedAsync` | Переопределите, чтобы обрабатывать добавление реакций в сообщение. |
| Удаление реакции на сообщение из сообщения | `OnReactionsRemovedAsync` | Переопределите, чтобы обрабатывать удаление реакций из сообщения. |
| Другой тип полученного действия | `OnUnrecognizedActivityTypeAsync` | Переопределите для обработки действия любого типа, которое иначе не будет обработано. |

Эти разные обработчики используют `turnContext` для получения сведений о входящем действии, которое соответствует входящему HTTP-запросу. Действия могут быть разных типов, поэтому каждый обработчик предоставляет действием со строгой типизацией в параметре контекста шага. В большинстве случаев `OnMessageActivityAsync` является наиболее распространенным и всегда обрабатывается.

Как и в предыдущих версиях 4.x этой платформы, в этой версии также можно реализовать открытый метод `OnTurnAsync`. Сейчас базовая реализация этого метода выполняет проверку ошибок, а затем вызывает каждый из определенных обработчиков (например те два, которые мы определяем в этом примере) в зависимости от типа входящего действия. В большинстве случаев можно не изменять этот метод и использовать отдельные обработчики. Но при необходимости вы также можете работать с пользовательской реализацией `OnTurnAsync`.

> [!IMPORTANT]
> При переопределении метода `OnTurnAsync` вам нужно вызвать `base.OnTurnAsync`, чтобы получить базовую реализацию для вызова всех остальных обработчиков `On<activity>Async`. Но вы также можете вызвать эти обработчики самостоятельно. В противном случае эти обработчики не будут вызваны и код не будет выполняться.

В этом примере мы приветствуем нового пользователя или возвращаем сообщение пользователя, отправленное с помощью вызова `SendActivityAsync`. Исходящее действие соответствует исходящему HTTP-запросу.

```cs
public class MyBot : ActivityHandler
{
    protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
    {
        await turnContext.SendActivityAsync(MessageFactory.Text($"Echo: {turnContext.Activity.Text}"), cancellationToken);
    }

    protected override async Task OnMembersAddedAsync(IList<ChannelAccount> membersAdded, ITurnContext<IConversationUpdateActivity> turnContext, CancellationToken cancellationToken)
    {
        foreach (var member in membersAdded)
        {
            await turnContext.SendActivityAsync(MessageFactory.Text($"welcome {member.Name}"), cancellationToken);
        }
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Основная логика бота определяется в коде бота. Здесь это `bots\echoBot.js`. `EchoBot` является производным от `ActivityHandler`. `ActivityHandler` определяет разные события для разных типов действий. Вы можете изменить поведение бота, зарегистрировав прослушиватели событий, такие как `onMessage` и `onConversationUpdate` в нашем примере.

Используйте следующие методы, чтобы зарегистрировать прослушиватели для каждого типа события.

| Событие | Метод регистрации | Описание |
| :-- | :-- | :-- |
| Любой тип полученного действия | `onTurn` | Регистрация прослушивателя, который вызывается при получении любого действия. |
| Полученное действие сообщения | `onMessage` | Регистрация прослушивателя, который вызывается при получении действия `message`. |
| Полученное действие обновления диалога | `onConversationUpdate` | Регистрация прослушивателя, который вызывается при получении любого действия `conversationUpdate`. |
| Присоединение участников к беседе | `onMembersAdded` | Регистрация прослушивателя, который вызывается при присоединении к беседе любого участника, включая самого бота. |
| Выход участников из беседы | `onMembersRemoved` | Регистрация прослушивателя, который вызывается при выходе из беседы любого участника, включая самого бота. |
| Получение действия реакции на сообщение | `onMessageReaction` | Регистрация прослушивателя, который вызывается при получении любого действия `messageReaction`. |
| Добавление реакции на сообщение в сообщение | `onReactionsAdded` | Регистрация прослушивателя, который вызывается при добавлении реакций к сообщению. |
| Удаление реакции на сообщение из сообщения | `onReactionsRemoved` | Регистрация прослушивателя, который вызывается при удалении реакций из сообщения. |
| Полученное действие события | `onEvent` | Регистрация прослушивателя, который вызывается при получении любого действия `event`. |
| Полученное действие события ответа маркера | `onTokenResponseEvent` | Регистрация прослушивателя, который вызывается при получении события `tokens/response`. |
| Другой тип полученного действия | `onUnrecognizedActivityType` | Регистрация прослушивателя, который вызывается при отсутствии обработчика для определенного типа действия. |
| Выполненные обработчики действий | `onDialog` | Вызывается после завершения всех применимых обработчиков. |

Вызывайте функцию продолжения `next` из каждого обработчика, чтобы продолжить обработку. Если `next` не вызывается, обработка действия завершается.

Например, этот бот регистрирует прослушиватели для сообщений и обновлений беседы. Он возвращает каждое сообщение, полученное от пользователя.

```javascript
const { ActivityHandler } = require('botbuilder');

class MyBot extends ActivityHandler {
    constructor() {
        super();
        // See https://aka.ms/about-bot-activity-message to learn more about the message and other activity types.
        this.onMessage(async (context, next) => {
            await context.sendActivity(`You said '${ context.activity.text }'`);
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
        this.onConversationUpdate(async (context, next) => {
            await context.sendActivity('[conversationUpdate event detected]');
            // By calling next() you ensure that the next BotHandler is run.
            await next();
        });
    }
}

module.exports.MyBot = MyBot;
```

# <a name="pythontabpython"></a>[Python](#tab/python)

Основная логика бота определяется в коде бота. Здесь это `bots/echo_bot.py`. `EchoBot` является производным от `ActivityHandler` (является производным от интерфейса `Bot`). `ActivityHandler` определяет различные обработчики для разных типов действий, например указанных здесь `on_message_activity` и `on_members_added`. Эти методы являются защищенными, но могут быть перезаписаны с учетом наследования от `ActivityHandler`.

Ниже описаны обработчики, определенные в `ActivityHandler`:

| Событие | Обработчик | Описание |
| :-- | :-- | :-- |
| Любой тип полученного действия | `on_turn` | Вызывает один из других обработчиков на основе типа полученного действия. |
| Полученное действие сообщения | `on_message_activity` | Переопределите для обработки действия `message`. |
| Полученное действие обновления диалога | `on_conversation_update_activity` | В действии `conversationUpdate` вызывает обработчик, если участники не добавлены в бота или покидают беседу. |
| Не добавленные в бота участники, которые присоединяются к беседе | `on_members_added_activity` | Переопределите для обработки участников, присоединяющихся к беседе. |
| Не добавленные в бота участники, которые покидают беседу | `on_members_removed_activity` | Переопределите для обработки участников, покидающих беседу. |
| Полученное действие события | `on_event_activity` | В действии `event` вызывает обработчик для события определенного типа. |
| Полученное действие события ответа маркера | `on_token_response_event` | Переопределите для обработки событий ответа маркера. |
| Полученное действие события другого ответа | `on_event_activity` | Переопределите для обработки событий других типов. |
| Получение действия реакции на сообщение | `on_message_reaction_activity` | При получении действия `messageReaction` вызывает обработчик, если в сообщении были добавлены или удалены одна или несколько реакций. |
| Добавление реакции на сообщение в сообщение | `on_reactions_added` | Переопределите, чтобы обрабатывать добавление реакций в сообщение. |
| Удаление реакции на сообщение из сообщения | `on_reactions_removed` | Переопределите, чтобы обрабатывать удаление реакций из сообщения. |
| Другой тип полученного действия | `on_unrecognized_activity_type` | Переопределите для обработки действия любого типа, которое иначе не будет обработано. |

Эти разные обработчики используют `turn_context` для получения сведений о входящем действии, которое соответствует входящему HTTP-запросу. Действия могут быть разных типов, поэтому каждый обработчик предоставляет действием со строгой типизацией в параметре контекста шага. В большинстве случаев `on_message_activity` является наиболее распространенным и всегда обрабатывается.

Как и в предыдущих версиях 4.x этой платформы, в этой версии также можно реализовать открытый метод `on_turn`. Сейчас базовая реализация этого метода выполняет проверку ошибок, а затем вызывает каждый из определенных обработчиков (например те два, которые мы определяем в этом примере) в зависимости от типа входящего действия. В большинстве случаев можно не изменять этот метод и использовать отдельные обработчики. Но при необходимости вы также можете работать с пользовательской реализацией `on_turn`.

> [!IMPORTANT]
> При переопределении метода `on_turn` вам нужно вызвать `super().on_turn`, чтобы получить базовую реализацию для вызова всех остальных обработчиков `on_<activity>`. Но вы также можете вызвать эти обработчики самостоятельно. В противном случае эти обработчики не будут вызваны и код не будет выполняться.

В этом примере мы приветствуем нового пользователя или возвращаем сообщение пользователя, отправленное с помощью вызова `send_activity`. Исходящее действие соответствует исходящему HTTP-запросу.

```py
class MyBot(ActivityHandler):
    async def on_members_added_activity(
        self, members_added: [ChannelAccount], turn_context: TurnContext
    ):
        for member in members_added:
            if member.id != turn_context.activity.recipient.id:
                await turn_context.send_activity("Hello and welcome!")

    async def on_message_activity(self, turn_context: TurnContext):
        return await turn_context.send_activity(
            f"Echo: {turn_context.activity.text}"
        )
```

---

### <a name="access-the-bot-from-your-app"></a>Доступ к боту из приложения

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

#### <a name="set-up-services"></a>Настройка служб

Метод `ConfigureServices` в файле `Startup.cs` загружает подключенные службы, а также их ключи из `appsettings.json` или Azure Key Vault (при наличии), состояние подключения и т. д. Мы добавляем MVC и указываем версию совместимости для служб, а затем делаем адаптер и бота доступными путем внедрения зависимостей в контроллер бота.

<!-- want to explain the singleton vs transient here?-->

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);

    // Create the credential provider to be used with the Bot Framework Adapter.
    services.AddSingleton<ICredentialProvider, ConfigurationCredentialProvider>();

    // Create the Bot Framework Adapter.
    services.AddSingleton<IBotFrameworkHttpAdapter, BotFrameworkHttpAdapter>();

    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
```

Метод `Configure` указывается в конце конфигурации приложения вместе с информацией о том, что приложение использует MVC и несколько других файлов. Для всех ботов, использующих Bot Framework, требуется выполнить вызов конфигурации. Но это поведение уже определено в примерах или шаблоне VSIX при создании бота. `ConfigureServices` и `Configure` вызываются средой выполнения при запуске приложения.

#### <a name="bot-controller"></a>Контроллер бота

В соответствии со стандартной структурой MVC контроллер позволяет настроить маршрутизацию сообщений и HTTP-запросов POST. Для нашего бота мы передаем входящий запрос в метод *асинхронного действия обработки* адаптера, как описано в разделе [Стек обработки действия](#the-activity-processing-stack) выше. В этом вызове мы указываем бота и другие сведения об авторизации, которые могут потребоваться.

Контроллер реализует `ControllerBase`, содержит адаптер и бота, которые мы настроили в `Startup.cs` (доступные путем внедрения зависимостей), и передает в бот необходимые сведения при получении входящего HTTP-запроса POST.

Здесь мы рассмотрим класс, за которым следуют атрибуты маршрута и контроллера. Они помогают платформе соответствующим образом направлять сообщения, определяя, какой контроллер использовать. Если вы измените значение в атрибуте маршрута, также будет изменена конечная точка, которую эмулятор или другие каналы используют для обращения к боту.

```cs
// This ASP Controller is created to handle a request. Dependency Injection will provide the Adapter and IBot
// implementation at runtime. Multiple different IBot implementations running at different endpoints can be
// achieved by specifying a more specific type for the bot constructor argument.
[Route("api/messages")]
[ApiController]
public class BotController : ControllerBase
{
    private readonly IBotFrameworkHttpAdapter Adapter;
    private readonly IBot Bot;

    public BotController(IBotFrameworkHttpAdapter adapter, IBot bot)
    {
        Adapter = adapter;
        Bot = bot;
    }

    [HttpPost]
    public async Task PostAsync()
    {
        // Delegate the processing of the HTTP POST to the adapter.
        // The adapter will invoke the bot.
        await Adapter.ProcessAsync(Request, Response, Bot);
    }
}
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

#### <a name="indexjs"></a>Файл index.js

`index.js` настраивает бота и службы размещения, которые будут пересылать действия в логику бота.

#### <a name="required-libraries"></a>Обязательные библиотеки

В начале файла `index.js` вы найдете набор обязательных модулей или библиотек. Эти модули предоставляют доступ к набору функций, которые, возможно, потребуется добавить в приложение.

```javascript
const dotenv = require('dotenv');
const path = require('path');
const restify = require('restify');

// Import required bot services.
// See https://aka.ms/bot-services to learn more about the different parts of a bot.
const { BotFrameworkAdapter } = require('botbuilder');

// This bot's main dialog.
const { MyBot } = require('./bot');

// Import required bot configuration.
const ENV_FILE = path.join(__dirname, '.env');
dotenv.config({ path: ENV_FILE });
```

#### <a name="set-up-services"></a>Настройка служб

На следующем этапе выполняется настройка сервера и адаптера, которые позволят боту общаться с пользователем и отправлять ответы. Сервер будет ожидать передачи данных на порту, указанном в файле конфигурации, или вернется к порту _3978_, чтобы установить подключение к эмулятору. Адаптер будет выступать в качестве проводника для вашего бота, управлять входящими и исходящими сообщениями, аутентификацией и т. д.

```javascript
// Create HTTP server
const server = restify.createServer();
server.listen(process.env.port || process.env.PORT || 3978, () => {
    console.log(`\n${ server.name } listening to ${ server.url }`);
    console.log(`\nGet Bot Framework Emulator: https://aka.ms/bot-framework-www-portal-emulator`);
    console.log(`\nTo talk to your bot, open the emulator select "Open Bot"`);
});

// Create adapter.
// See https://aka.ms/about-bot-adapter to learn more about how bots work.
const adapter = new BotFrameworkAdapter({
    appId: process.env.MicrosoftAppId,
    appPassword: process.env.MicrosoftAppPassword
});

// Catch-all for errors.
adapter.onTurnError = async (context, error) => {
    // This check writes out errors to console log .vs. app insights.
    console.error(`\n [onTurnError]: ${ error }`);
    // Send a message to the user
    await context.sendActivity(`Oops. Something went wrong!`);
};

// Create the main dialog.
const myBot = new MyBot();
```

#### <a name="forwarding-requests-to-the-bot-logic"></a>Переадресация запросов в логику бота

Метод адаптера `processActivity` отправляет входящие действия в логику бота.
Третий параметр в `processActivity` представляет собой обработчик функций. Он вызывается для выполнения логики бота после того, как адаптер и ПО промежуточного слоя выполнят для полученного [действия](#the-activity-processing-stack) все этапы предварительной обработки. Переменную контекста шага, которая передается обработчику функций в виде аргумента, можно использовать для предоставления сведений о входящем действии, отправителе и получателе, канале, беседе и т.д. Для обработки действие передается в метод `run` бота. `run` определяется в `ActivityHandler`. При этом выполняется проверка ошибок с последующим вызовом обработчиков событий бота на основе типа полученного действия.

```javascript
// Listen for incoming requests.
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        // Route to main dialog.
        await myBot.run(context);
    });
});
```

# <a name="pythontabpython"></a>[Python](#tab/python)

#### <a name="apppy"></a>app.py

`app.py` настраивает бота и службы размещения, которые будут пересылать действия в логику бота.

#### <a name="required-libraries"></a>Обязательные библиотеки

В начале файла `app.py` вы найдете набор обязательных модулей или библиотек. Эти модули предоставляют доступ к набору функций, которые, возможно, потребуется добавить в приложение.

```py
from botbuilder.core import BotFrameworkAdapterSettings, TurnContext, BotFrameworkAdapter
from botbuilder.schema import Activity, ActivityTypes

from bots import MyBot

# Create the loop and Flask app
LOOP = asyncio.get_event_loop()
app = Flask(__name__, instance_relative_config=True)
app.config.from_object("config.DefaultConfig")
```

#### <a name="set-up-services"></a>Настройка служб

На следующем этапе выполняется настройка сервера и адаптера, которые позволят боту общаться с пользователем и отправлять ответы. Сервер будет ожидать передачи данных на порту, указанном в файле конфигурации, или вернется к порту _3978_, чтобы установить подключение к эмулятору. Адаптер будет выступать в качестве проводника для вашего бота, управлять входящими и исходящими сообщениями, аутентификацией и т. д.

```py
# Create adapter.
# See https://aka.ms/about-bot-adapter to learn more about how bots work.
SETTINGS = BotFrameworkAdapterSettings(app.config["APP_ID"], app.config["APP_PASSWORD"])
ADAPTER = BotFrameworkAdapter(SETTINGS)

# Catch-all for errors.
async def on_error(context: TurnContext, error: Exception):
    # This check writes out errors to console log .vs. app insights.
    # NOTE: In production environment, you should consider logging this to Azure
    #       application insights.
    print(f"\n [on_turn_error] unhandled error: {error}", file=sys.stderr)

    # Send a message to the user
    await context.send_activity("The bot encountered an error or bug.")
    await context.send_activity("To continue to run this bot, please fix the bot source code.")
    # Send a trace activity if we're talking to the Bot Framework Emulator
    if context.activity.channel_id == 'emulator':
        # Create a trace activity that contains the error object
        trace_activity = Activity(
            label="TurnError",
            name="on_turn_error Trace",
            timestamp=datetime.utcnow(),
            type=ActivityTypes.trace,
            value=f"{error}",
            value_type="https://www.botframework.com/schemas/error"
        )
        # Send a trace activity, which will be displayed in Bot Framework Emulator
        await context.send_activity(trace_activity)

ADAPTER.on_turn_error = on_error

# Create the Bot
BOT = MyBot()
```

#### <a name="forwarding-requests-to-the-bot-logic"></a>Переадресация запросов в логику бота

Метод адаптера `process_activity` отправляет входящие действия в логику бота.
Третий параметр в `process_activity` представляет собой обработчик функций. Он вызывается для выполнения логики бота после того, как адаптер и ПО промежуточного слоя выполнят для полученного [действия](#the-activity-processing-stack) все этапы предварительной обработки. Переменную контекста шага, которая передается обработчику функций в виде аргумента, можно использовать для предоставления сведений о входящем действии, отправителе и получателе, канале, беседе и т.д. Для обработки действие передается в метод `on_turn` бота. `on_turn` определяется в `ActivityHandler`. При этом выполняется проверка ошибок с последующим вызовом обработчиков событий бота на основе типа полученного действия.

```py
# Listen for incoming requests on /api/messages
@app.route("/api/messages", methods=["POST"])
def messages():
    # Main bot message handler.
    if "application/json" in request.headers["Content-Type"]:
        body = request.json
    else:
        return Response(status=415)

    activity = Activity().deserialize(body)
    auth_header = (
        request.headers["Authorization"] if "Authorization" in request.headers else ""
    )

    try:
        task = LOOP.create_task(
            ADAPTER.process_activity(activity, auth_header, BOT.on_turn)
        )
        LOOP.run_until_complete(task)
        return Response(status=201)
    except Exception as exception:
        raise exception
```

---

## <a name="manage-bot-resources"></a>Управление ресурсами бота

Ресурсами ботов, такими как идентификаторы приложений, пароли, ключи или секреты для подключенных служб, необходимо управлять соответствующим образом. См. подробнее об [управлении ресурсами бота](bot-file-basics.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

- Общие сведения о роли состояния для ботов см. в статье [об управлении состоянием](bot-builder-concept-state.md).

- Основные понятия разработки ботов для Microsoft Teams вам поможет понять статья [How Microsoft Teams bots work](bot-builder-basics-teams.md) (Принцип работы ботов Microsoft Teams).

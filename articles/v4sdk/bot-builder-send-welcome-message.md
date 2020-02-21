---
title: Отправка приветственного сообщения пользователям в службе Bot
description: Сведения о том, как создать приветствие, которое бот отправляет пользователю.
keywords: overview, develop, user experience, welcome, personalized experience, C#, JS, welcome message, bot, greet, greeting
author: DanDev33
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/7/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 92ec5c927baa93095beaf304fe17024adf81666b
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2020
ms.locfileid: "77441674"
---
# <a name="send-welcome-message-to-users"></a>Отправка приветственного сообщения пользователям

[!INCLUDE[applies-to](../includes/applies-to.md)]

Основная цель создания любого бота — ведение осмысленного диалога с пользователем. Лучший способ достичь этой цели — сделать так, чтобы с момента присоединения к диалогу пользователь понимал основное назначение вашего бота, его возможности и причины создания. В этой статье представлены примеры кода, которые помогут создать приветствие, отправляемое ботом пользователю.

## <a name="prerequisites"></a>Предварительные требования

- Понимание [основных принципов работы ботов](bot-builder-basics.md).
- Копия **примера с приветствием пользователя** для [C#](https://aka.ms/welcome-user-mvc), [JavaScript](https://aka.ms/bot-welcome-sample-js) или [Python](https://aka.ms/bot-welcome-python-sample-code). На примере кода в этой статье мы опишем, как отправлять приветственные сообщения.

## <a name="about-this-sample-code"></a>Сведения о примере кода

Этот пример кода демонстрирует, как обнаруживать и приветствовать новых пользователей, которые впервые подключаются к боту. На следующей схеме показан поток логики для такого бота.

### <a name="c"></a>[C#](#tab/csharp)

Два основных события, отслеживаемые ботом:

- `OnMembersAddedAsync` вызывается каждый раз, когда к боту подключается новый пользователь;
- `OnMessageActivityAsync` вызывается каждый раз, когда принимаются данные от нового пользователя.

![Поток логики для приветствия пользователя](media/welcome-user-flow.png)

Каждый раз, когда подключается новый пользователь, бот предоставляет ему `WelcomeMessage`, `InfoMessage` и `PatternMessage`.
При получении данных от нового пользователя бот проверяет WelcomeUserState и выясняет, имеет ли `DidBotWelcomeUser` значение _true_. Если это не так, он возвращает новому пользователю сообщение с приветствием.

### <a name="javascript"></a>[JavaScript](#tab/javascript)

Два основных события, отслеживаемые ботом:

- `onMembersAdded` вызывается каждый раз, когда к боту подключается новый пользователь;
- `onMessage` вызывается каждый раз, когда принимаются данные от нового пользователя.

![Поток логики для приветствия пользователя](media/welcome-user-flow-js.png)

Каждый раз, когда подключается новый пользователь, бот предоставляет ему `welcomeMessage`, `infoMessage` и `patternMessage`.
При получении данных от нового пользователя бот проверяет `welcomedUserProperty` и выясняет, имеет ли `didBotWelcomeUser` значение _TRUE_. Если это не так, он возвращает новому пользователю сообщение с приветствием.
Если `DidBotWelcomeUser` имеет значение _true_, входные данные пользователя оцениваются. В зависимости от содержания этих данных бот выполняет одно из следующих действий:

- возвращает приветствие, полученное от пользователя;
- отображает карту для имиджевого баннера с информацией о ботах;
- повторно отправляет `WelcomeMessage`, где описаны ожидаемые входные данные для этого бота.

### <a name="python"></a>[Python](#tab/python)

Два основных события, отслеживаемые ботом:

- `on_members_added_activity` вызывается каждый раз, когда к боту подключается новый пользователь;
- `on_message_activity` вызывается каждый раз, когда принимаются данные от нового пользователя.

![Поток логики для приветствия пользователя](media/welcome-user-flow-python.png)

При каждом подключении нового пользователя бот выводит *приветственное сообщение*, *информационное сообщение* и *сообщение-шаблон*.
При получении данных от нового пользователя бот проверяет свойство `welcome_user_state.did_welcome_user` и выясняет, имеет ли оно значение *true*. Если это не так, он возвращает новому пользователю сообщение с приветствием. Если свойство имеет значение *true*, в зависимости от содержания этих данных бот выполняет одно из следующих действий:

- возвращает приветствие, полученное от пользователя;
- отображает карту для имиджевого баннера с информацией о ботах;

---

## <a name="create-user-state"></a>Создание состояния пользователя

### <a name="c"></a>[C#](#tab/csharp)

В момент запуска создается объект состояния пользователя, а в конструктор бота добавляются зависимости.

**Startup.cs** [!code-csharp[define state](~/../botBuilder-samples/samples/csharp_dotnetcore/03.welcome-user/Startup.cs?range=30-34)]

**Bots\WelcomeUserBot.cs** [!code-csharp[consume state](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=41-47)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

При запуске состояние пользователя определяется в `index.js` и используется конструктором бота.

**index.js** [!code-javascript[define state](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/index.js?range=51-55)]

**bots/welcomeBot.js** [!code-javascript[consume state](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomeBot.js?range=16-22)]

### <a name="python"></a>[Python](#tab/python)

При запуске состояние пользователя определяется в `app.py` и используется конструктором бота.

**app.py** [!code-python[define state](~/../botbuilder-samples/samples/python/03.welcome-user/app.py?range=61-66)]

**bots/welcome-user-bot.py** [!code-python[consume state](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=23-29)]

---

## <a name="create-property-accessors"></a>Создание методов доступа к свойствам

### <a name="c"></a>[C#](#tab/csharp)

Теперь мы создадим метод доступа к свойству, предоставляющий обработчик `WelcomeUserState` в методе `OnMessageActivityAsync`.
Затем мы вызовем метод `GetAsync`, чтобы получить ключ с правильной областью действия. Мы будем сохранять данные о состоянии пользователя после каждого цикла обработки введенных пользователем данных с помощью метода `SaveChangesAsync`.

**Bots\WelcomeUserBot.cs** [!code-csharp[Get state](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-71)]
[!code-csharp[Save state](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range= 103-105)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

Теперь мы создадим метод доступа к свойству, который будет выполнять роль дескриптора для свойства welcomedUserProperty, хранимого в userState.

**bots/welcomeBot.js** [!code-javascript[Get state](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=24-27)]
[!code-javascript[Save state](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=89-97)]

### <a name="python"></a>[Python](#tab/python)

В этом примере создается метод доступа к свойству состояния диалога (`user_state_accessor`) в конструкторе бота.

**bots/welcome-user-bot.py** [!code-python[constructor](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=31)]

Он использует метод доступа к свойству в обработчике `on_message_activity` и переопределяет обработчик `on_turn`, чтобы сохранить состояние до завершения шага.

[!code-python[Get state](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=80-83)]
[!code-python[save state](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=39-43)]

---

## <a name="detect-and-greet-newly-connected-users"></a>Обнаружение и приветствие новых подключенных пользователей

### <a name="c"></a>[C#](#tab/csharp)

В **WelcomeUserBot** мы проверяем обновление действия с помощью `OnMembersAddedAsync()`, чтобы узнать, добавлен ли новый пользователь в диалог. Затем мы отправляем ему набор из трех приветственных сообщений: `WelcomeMessage`, `InfoMessage` и `PatternMessage`. Ниже приведен полный код для этого взаимодействия.

**Bots\WelcomeUserBot.cs** [!code-csharp[Define messages](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=20-39)]
[!code-csharp[Send messages](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=55-66)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

Этот код JavaScript отправляет начальное приветственное сообщение при добавлении пользователя. Для этого мы проверяем, является ли диалог активным и добавлен ли новый участник в диалог.

**bots/welcomeBot.js** [!code-javascript[onMembersAdded](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=63-86)]

### <a name="python"></a>[Python](#tab/python)

`on_members_added_activity` проверяет, добавлен ли новый пользователь, а затем отправляет три начальных приветственных сообщения: *приветственное сообщение*, *информационное сообщение* и *сообщение-шаблон*.

**bots/welcome-user-bot.py** [!code-python[on_members_added_activity](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=45-74)]

---

## <a name="welcome-new-user-and-discard-initial-input"></a>Приветствие нового пользователя и удаление первого входящего сообщения

### <a name="c"></a>[C#](#tab/csharp)

Важно помнить о том, что введенные пользователем данные могут содержать ценную информацию и что в разных каналах взаимодействие может происходить по-разному. Чтобы облегчить пользователю работу со всеми возможными каналами, нам нужно проверить флаг состояния _didBotWelcomeUser_. Если его значение равно false, мы не обрабатываем начальное сообщение, введенное пользователем. Вместо этого мы предоставляем пользователю начальное приветственное сообщение. Затем мы присваиваем значение true свойству _welcomedUserProperty_, которое хранится в UserState, и с этого момента наш код будет обрабатывать введенные этим пользователем данные от всех дополнительных действий сообщения.

**Bots\WelcomeUserBot.cs** [!code-csharp[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-84)]
[!code-csharp[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=101-105)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

Важно помнить о том, что введенные пользователем данные могут содержать ценную информацию и что в разных каналах взаимодействие может происходить по-разному. Чтобы облегчить пользователю работу со всеми возможными каналами, нам нужно проверить свойство didBotWelcomeUser. Если оно не существует, мы присваиваем ему значение false и не обрабатываем начальное сообщение, введенное пользователем. Вместо этого мы предоставляем пользователю начальное приветственное сообщение. Затем для didBotWelcomeUser устанавливается значение true, и наш код обрабатывает введенные пользователем данные от всех дополнительных действий сообщения.

**bots/welcomeBot.js** [!code-javascript[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=24-39)]
[!code-javascript[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=57-61)]

### <a name="python"></a>[Python](#tab/python)

Важно помнить о том, что введенные пользователем данные могут содержать ценную информацию и что в разных каналах взаимодействие может происходить по-разному. Чтобы обеспечить оптимальную работу пользователя на всех возможных каналах, `on_message_activity` проверяет свойство `did_welcome_user`. В первый раз для свойства задается значение *false* и введенные пользователем данные не обрабатываются. Вместо этого пользователю предоставляется начальное приветственное сообщение. Затем для `did_welcome_user` устанавливается значение *true* и обрабатываются введенные пользователем данные из всех дополнительных действий с сообщениями.

**bots/welcome-user-bot.py** [!code-python[DidBotWelcomeUser](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=85-95)]

---

## <a name="process-additional-input"></a>Обработка дополнительных входных данных

Когда новый пользователь получает приветственное сообщение, введенные им данные оцениваются на каждом шаге диалога, и бот предоставляет ответы на основе контекста этих входных данных. Следующий код демонстрирует логику принятия решений, которая используется для создания ответа.

### <a name="c"></a>[C#](#tab/csharp)

Ввод команд intro или help вызывает функцию `SendIntroCardAsync`, которая предоставляет пользователю информационную карту для имиджевого баннера. Этот код рассматривается в следующем разделе этой статьи.

**Bots\WelcomeUserBot.cs** [!code-csharp[SwitchOnUtterance](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=85-100)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

Ввод команд intro или help обрабатывается с помощью CardFactory для предоставления пользователю вводной адаптивной карты. Этот код рассматривается в следующем разделе этой статьи.

**bots/welcomeBot.js** [!code-javascript[SwitchOnUtterance](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=40-56)]

### <a name="python"></a>[Python](#tab/python)

При введении пользователем команды *intro* или *help* бот вызывает команду `__send_intro_card`, предоставляющую пользователю вводную адаптивную карту.

**bots/welcome-user-bot.py** [!code-python[SwitchOnUtterance](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=101-106&highlight=97-106)]

---

## <a name="using-hero-card-greeting"></a>Использование карты для имиджевого баннера с приветствием

Как упоминалось выше, в ответ на некоторые введенные пользователем данные создается *карта для имиджевого баннера*. См. подробнее в руководстве по [отправке карты с вводными сведениями](./bot-builder-howto-add-media-attachments.md). Ниже представлен код, который предоставляет ответ с картой для имиджевого баннера этого бота.

### <a name="c"></a>[C#](#tab/csharp)

**Bots\WelcomeUserBot.cs** [!code-csharp[SendHeroCardGreeting](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=107-125)]

### <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/welcomeBot.js** [!code-javascript[SendIntroCard](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=99-124)]

### <a name="python"></a>[Python](#tab/python)

**bots/welcome-user-bot.py** [!code-python[SendIntroCard](~/../botbuilder-samples/samples/python/03.welcome-user/bots/welcome_user_bot.py?range=108-143)]

---

## <a name="test-the-bot"></a>Тестирование бота

Скачайте и установите последнюю версию [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

1. Выполните этот пример на локальном компьютере. Если потребуются дополнительные инструкции, в файле README можно найти [пример кода на C#](https://aka.ms/welcome-user-mvc) или [пример кода на JS](https://aka.ms/bot-welcome-sample-js).
1. Примените эмулятор для тестирования бота, как показано ниже.

![Проверка примера бота приветствия](media/welcome-user-emulator-1.png)

Проверьте карту для имиджевого баннера с приветствием.

![Проверка карты для имиджевого баннера бота с приветствием](media/welcome-user-emulator-2.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

См. подробнее о [добавлении мультимедиа в сообщения](./bot-builder-howto-add-media-attachments.md).

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сбор вводимых пользователем данных](bot-builder-prompts.md)

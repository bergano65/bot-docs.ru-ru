---
title: Отправка приветственного сообщения пользователям | Документация Майкрософт
description: Сведения о том, как создать приветствие, которое бот отправляет пользователю.
keywords: overview, develop, user experience, welcome, personalized experience, C#, JS, welcome message, bot, greet, greeting
author: DanDev33
ms.author: v-dashel
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 03/13/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0858cf73c1a744100c2f7106013fd963cff84454
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032514"
---
# <a name="send-welcome-message-to-users"></a>Отправка приветственного сообщения пользователям

[!INCLUDE[applies-to](../includes/applies-to.md)]

Основная цель создания любого бота — ведение осмысленного диалога с пользователем. Лучший способ достичь этой цели — сделать так, чтобы с момента присоединения к диалогу пользователь понимал основное назначение вашего бота, его возможности и причины создания. В этой статье представлены примеры кода, которые помогут создать приветствие, отправляемое ботом пользователю.

## <a name="prerequisites"></a>Предварительные требования
- Понимание [основных принципов работы ботов](bot-builder-basics.md). 
- Копия **примера с приветствием пользователя** для [C#](https://aka.ms/welcome-user-mvc) и [JavaScript](https://aka.ms/bot-welcome-sample-js). На примере кода в этой статье мы опишем, как отправлять приветственные сообщения.

## <a name="about-this-sample-code"></a>Сведения о примере кода
Этот пример кода демонстрирует, как обнаруживать и приветствовать новых пользователей, которые впервые подключаются к боту. На следующей схеме показан поток логики для такого бота. 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
Два основных событиях, отслеживаемых ботом:
- `OnMembersAddedAsync` вызывается каждый раз, когда к боту подключается новый пользователь;
- `OnMessageActivityAsync` вызывается каждый раз, когда принимаются данные от нового пользователя.

![Поток логики для приветствия пользователя](media/welcome-user-flow.png)

Каждый раз, когда подключается новый пользователь, бот предоставляет ему `WelcomeMessage`, `InfoMessage` и `PatternMessage`. При получении данных от нового пользователя бот проверяет WelcomeUserState и выясняет, имеет ли `DidBotWelcomeUser` значение _true_. Если это не так, он возвращает новому пользователю сообщение с приветствием.

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Два основных событиях, отслеживаемых ботом:
- `onMembersAdded` вызывается каждый раз, когда к боту подключается новый пользователь;
- `onMessage` вызывается каждый раз, когда принимаются данные от нового пользователя.

![Поток логики для приветствия пользователя](media/welcome-user-flow-js.png)

Каждый раз, когда подключается новый пользователь, бот предоставляет ему `welcomeMessage`, `infoMessage` и `patternMessage`. При получении данных от нового пользователя бот проверяет `welcomedUserProperty` и выясняет, имеет ли `didBotWelcomeUser` значение _TRUE_. Если это не так, он возвращает новому пользователю сообщение с приветствием.

---

 Если DidBotWelcomeUser имеет значение _true_, оцениваются введенные пользователем данные. В зависимости от содержания этих данных бот выполняет одно из следующих действий:
- возвращает приветствие, полученное от пользователя;
- отображает карту для имиджевого баннера с информацией о ботах;
- повторно отправляет `WelcomeMessage`, где описаны ожидаемые входные данные для этого бота.

## <a name="create-user-object"></a>Создание объекта пользователя
### <a name="ctabcsharp"></a>[C#](#tab/csharp)
В момент запуска создается объект состояния пользователя, а в конструктор бота добавляются зависимости.

**Startup.cs** [!code-csharp[ConfigureServices](~/../botBuilder-samples/samples/csharp_dotnetcore/03.welcome-user/Startup.cs?range=29-33)]

**WelcomeUserBot.cs** [!code-csharp[Class](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=41-47)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Параметры хранилища в памяти и состояния пользователя на момент запуска определены в файле index.js.

**Index.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/Index.js?range=8-10,33-41)]

---

## <a name="create-property-accessors"></a>Создание методов доступа к свойствам
### <a name="ctabcsharp"></a>[C#](#tab/csharp)
Сейчас мы создадим метод доступа к свойству, который будет выполнять роль дескриптора для WelcomeUserState внутри метода OnMessageActivityAsync.
Затем мы вызовем метод GetAsync, чтобы получить ключ с правильной областью действия. Мы будем сохранять данные о состоянии пользователя после каждого цикла обработки введенных пользователем данных с помощью метода `SaveChangesAsync`.

**WelcomeUserBot.cs** [!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-71, 102-105)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Теперь мы создадим метод доступа к свойству, который будет выполнять роль дескриптора для свойства WelcomedUserProperty, хранимого в UserState.

**WelcomeBot.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=7-10,16-22)]

---

## <a name="detect-and-greet-newly-connected-users"></a>Обнаружение и приветствие новых подключенных пользователей

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
В **WelcomeUserBot** мы проверяем обновление действия с помощью `OnMembersAddedAsync()`, чтобы узнать, добавлен ли новый пользователь в диалог. Затем мы отправляем ему набор из трех приветственных сообщений: `WelcomeMessage`, `InfoMessage` и `PatternMessage`. Ниже приведен полный код для этого взаимодействия.

**WelcomeUserBot.cs** [!code-csharp[WelcomeMessages](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=20-40, 55-66)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Этот код JavaScript отправляет начальное приветственное сообщение при добавлении пользователя. Для этого мы проверяем, является ли диалог активным и добавлен ли новый участник в диалог.

**WelcomeBot.js** [!code-javascript[DefineUserState](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=65-87)]

---

## <a name="welcome-new-user-and-discard-initial-input"></a>Приветствие нового пользователя и удаление первого входящего сообщения

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
Важно помнить о том, что введенные пользователем данные могут содержать ценную информацию и что в разных каналах взаимодействие может происходить по-разному. Чтобы облегчить пользователю работу со всеми возможными каналами, нам нужно проверить флаг состояния _didBotWelcomeUser_. Если его значение равно false, мы не обрабатываем начальное сообщение, введенное пользователем. Вместо этого мы предоставляем пользователю начальное приветственное сообщение. Затем мы присваиваем значение true свойству _welcomedUserProperty_, которое хранится в UserState, и с этого момента наш код будет обрабатывать введенные этим пользователем данные от всех дополнительных действий сообщения.

**WelcomeUserBot.cs** [!code-csharp[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=68-82)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Важно помнить о том, что введенные пользователем данные могут содержать ценную информацию и что в разных каналах взаимодействие может происходить по-разному. Чтобы облегчить пользователю работу со всеми возможными каналами, нам нужно проверить свойство didBotWelcomeUser. Если оно не существует, мы присваиваем ему значение false и не обрабатываем начальное сообщение, введенное пользователем. Вместо этого мы предоставляем пользователю начальное приветственное сообщение. Затем для _didBotWelcomeUser_ устанавливается значение true, и наш код обрабатывает введенные пользователем данные от всех дополнительных действий сообщения.

**WelcomeBot.js** [!code-javascript[DidBotWelcomeUser](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=24-38,57-59,63)]

---

## <a name="process-additional-input"></a>Обработка дополнительных входных данных

Когда новый пользователь получает приветственное сообщение, введенные им данные оцениваются на каждом шаге диалога, и бот предоставляет ответы на основе контекста этих входных данных. Следующий код демонстрирует логику принятия решений, которая используется для создания ответа. 

### <a name="ctabcsharp"></a>[C#](#tab/csharp)
Ввод команд intro или help вызывает функцию `SendIntroCardAsync`, которая предоставляет пользователю информационную карту для имиджевого баннера. Этот код рассматривается в следующем разделе этой статьи.

**WelcomeUserBot.cs** [!code-csharp[SwitchOnUtterance](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=85-100)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Ввод команд intro или help обрабатывается с помощью CardFactory для предоставления пользователю вводной адаптивной карты. Этот код рассматривается в следующем разделе этой статьи.

**WelcomeBot.js** [!code-javascript[SwitchOnUtterance](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=40-56)]

---

## <a name="using-hero-card-greeting"></a>Использование карты для имиджевого баннера с приветствием

Как упоминалось выше, в ответ на некоторые введенные пользователем данные создается _карта для имиджевого баннера_. См. подробнее в руководстве по [отправке карты с вводными сведениями](./bot-builder-howto-add-media-attachments.md). Ниже представлен код, который предоставляет ответ с картой для имиджевого баннера этого бота.

### <a name="ctabcsharp"></a>[C#](#tab/csharp)

**WelcomeUserBot.cs** [!code-csharp[SendHeroCardGreeting](~/../BotBuilder-Samples/samples/csharp_dotnetcore/03.welcome-user/bots/WelcomeUserBot.cs?range=107-127)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**WelcomeBot.js** [!code-javascript[SendIntroCard](~/../BotBuilder-Samples/samples/javascript_nodejs/03.welcome-users/bots/welcomebot.js?range=91-116)]

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

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Сбор вводимых пользователем данных](bot-builder-prompts.md)

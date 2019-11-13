---
title: Сохранение данных пользователя и диалога | Документация Майкрософт
description: Узнайте, как сохранять и извлекать данные о состоянии с помощью пакета SDK Bot Framework.
keywords: conversation state, user state, conversation, saving state, managing bot state
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f306fdbe39fd7863faf0ec32f99604c3a99da955
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933740"
---
# <a name="save-user-and-conversation-data"></a>Сохранение данных пользователя и диалога

[!INCLUDE[applies-to](../includes/applies-to.md)]

Бот по своей природе не учитывает состояния. Развернутый бот не обязан выполнять следующий шаг в том же процессе или на том же компьютере, что и предыдущий. Но иногда боту нужно отслеживать контекст беседы, чтобы управлять ее ходом и запоминать ответы на предыдущие вопросы. Функции состояния и хранения, предоставляемые пакетом SDK Bot Framework, позволяют реализовать в боте поддержку состояния. Для администрирования и хранения данных боты используют объекты хранилища и управления состоянием. Диспетчер состояний предоставляет уровень абстракции, позволяющий пол учить доступ к свойствам состояния через методы доступа независимо от типа базового хранилища.

## <a name="prerequisites"></a>Предварительные требования

- Требуется понимание [основных сведений о ботах](bot-builder-basics.md) и [управления состоянием](bot-builder-concept-state.md) для ботов.
- Код в этой статье основан на примере **бота управления состоянием**. Вам потребуется копия этого примера на языке [CSharp](https://aka.ms/statebot-sample-cs) или [JavaScript](https://aka.ms/statebot-sample-js).

## <a name="about-this-sample"></a>Об этом примере
Получив от пользователя входные данные, этот пример проверяет сохраненное состояние беседы и определяет, выводился ли этому пользователю запрос для ввода имени. Если еще нет, он запрашивает имя пользователя и сохраняет ответ в состоянии пользователя. Если да, сохраненное в состоянии пользователя имя используется для взаимодействия с пользователем и обработки входных данных, а затем вместе со временем получения сообщения и идентификатором канала возвращается обратно пользователю. Значения времени и идентификатора канала извлекаются из данных беседы с пользователем, а затем сохраняются в состоянии беседы. На следующей схеме показана связь между ботом, профилем пользователя и классами данных беседы.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

![Пример бота с управлением состоянием](media/StateBotSample-Overview.png)

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![Пример бота с управлением состоянием](media/StateBotSample-JS-Overview.png)

---

## <a name="define-classes"></a>Определение классов

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

При настройке управления состоянием первым делом нужно определить классы, которые будут содержать все нужные сведения для управления состоянием пользователя и беседы. В этом примере мы определили следующее:

- Класс `UserProfile` в файле **UserProfile.cs** для хранения собранных ботом сведений о пользователе. 
- Класс `ConversationData` в файле **ConversationData.cs** для управления состоянием беседы при сборе сведений о пользователе.

Следующий пример кода демонстрирует создание определения для класса UserProfile.

**UserProfile.cs**  
[!code-csharp[UserProfile](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/UserProfile.cs?range=7-11)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

На первом шаге нужно включить службу botbuilder, которая содержит определения для `UserState` и `ConversationState`.

**index.js**  
[!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=7-9)]

---

## <a name="create-conversation-and-user-state-objects"></a>Создание объектов состояния беседы и пользователя

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Теперь мы зарегистрируем метод `MemoryStorage`, который используется для создания объектов `UserState` и `ConversationState`. В `Startup` создаются объекты состояния пользователя и беседы, а в конструктор бота добавляются зависимости. Также для бота регистрируются дополнительные службы: поставщик учетных данных, адаптер и реализация бота.

**Startup.cs.**  
[!code-csharp[ConfigureServices](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Startup.cs?range=26-29)]
[!code-csharp[ConfigureServices](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Startup.cs?range=51-57)]

**Bots/StateManagementBot.cs** [!code-csharp[StateManagement](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Bots/StateManagementBot.cs?range=15-22)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Теперь мы зарегистрируем метод `MemoryStorage` и применим его для создания объектов `UserState` и `ConversationState`.

**index.js**  
[!code-javascript[DefineMemoryStore](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/index.js?range=33-39)]

---

## <a name="add-state-property-accessors"></a>Добавление методов доступа к свойству состояния

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Теперь мы создадим методы доступа к свойствам состояния с помощью метода `CreateProperty`, который предоставляет дескриптор для объекта `BotState`. Каждый метод доступа к свойству состояния позволяет получить или задать значение для соответствующего свойства состояния. Прежде чем использовать свойства состояния, мы вызовем каждый метод доступа, чтобы загрузить свойства из хранилища в кэш состояния и получить их оттуда. Чтобы получить ключ с правильной областью действия, связанный со свойством состояния, мы вызовем метод `GetAsync`.

**Bots/StateManagementBot.cs** [!code-csharp[StateAccessors](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/Bots/StateManagementBot.cs?range=38-46)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Теперь мы создадим методы доступа к свойствам `UserState` и `ConversationState`. Каждый метод доступа к свойству состояния позволяет получить или задать значение для соответствующего свойства состояния. Мы выполняем каждый из методов доступа, чтобы загрузить связанное свойство из хранилища и извлечь из кэша его текущее состояние.

**bots/stateManagementBot.js** [!code-javascript[BotService](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=6-19)]

---

## <a name="access-state-from-your-bot"></a>Доступ к состояниям из кода бота

В предыдущем разделе мы рассматривали шаги, которые на этапе инициализации добавляют в бота методы доступа к свойствам состояния. Теперь мы вызовем эти методы доступа во время выполнения, чтобы считать и (или) записать сведения о состоянии. Следующий пример кода использует представленный здесь поток логики:

- Если свойство userProfile.Name пусто, а conversationData.PromptedUserForName имеет значение _true_, мы получаем предоставленное имя пользователя и сохраняем его в состоянии пользователя.
- Если свойство userProfile.Name пусто, а conversationData.PromptedUserForName имеет значение _false_, мы запрашиваем имя пользователя.
- Если значение userProfile.Name уже было сохранено ранее, мы извлекаем из пользовательского ввода время сообщения и идентификатор канала, затем передаем все эти данные пользователю и сохраняем полученные данные в состоянии беседы.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots/StateManagementBot.cs** [!code-csharp[OnMessageActivityAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=38-85)]

Прежде чем мы выйти из обработчика шагов, мы применим метод _SaveChangesAsync()_ для объектов управления состоянием, чтобы записать все изменения состояния обратно в хранилище.

**Bots/StateManagementBot.cs** [!code-csharp[OnTurnAsync](~/../BotBuilder-Samples/samples/csharp_dotnetcore/45.state-management/bots/StateManagementBot.cs?range=24-31)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/stateManagementBot.js** [!code-javascript[OnMessage](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=21-58)]

Перед выходом из каждого шага диалога мы применяем метод _saveChanges()_ для объектов управления состоянием, чтобы сохранить в хранилище все изменения, внесенные в состояние.

**bots/stateManagementBot.js** [!code-javascript[OnDialog](~/../BotBuilder-Samples/samples/javascript_nodejs/45.state-management/bots/stateManagementBot.js?range=60-67)]

---

## <a name="test-the-bot"></a>Тестирование бота

Скачайте и установите последнюю версию [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

1. Выполните этот пример на локальном компьютере. Если потребуются дополнительные инструкции, в файле README можно найти [пример кода на C#](https://aka.ms/statebot-sample-cs) или [пример кода на JS](https://aka.ms/statebot-sample-js).
1. Примените эмулятор для тестирования бота, как показано ниже.

![Пример тестирования бота с управлением состояния](media/state-bot-testing-emulator.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

**Конфиденциальность.** Если вы собираетесь хранить персональные данные пользователя, обеспечьте соблюдение [Общего регламента по защите данных](https://blog.botframework.com/2018/04/23/general-data-protection-regulation-gdpr).

**Управление состоянием.** Все вызовы методов управления состоянием обрабатываются асинхронно, и по умолчанию применяется только последнее действие, выполняющее запись данных. На практике следует размещать методы get, set и save state как можно ближе друг к другу в коде бота.

**Критически важные бизнес-данные.** Используйте состояние бота для хранения настроек, имени пользователя или сведений о последнем заказе, но не используйте его для хранения критически важных бизнес-данных. Для критически важных данных [создайте собственные компоненты хранилища](bot-builder-custom-storage.md) или записывайте их непосредственно в [хранилище](bot-builder-howto-v4-storage.md).

**Recognizer-Text.** В этом примере используются библиотеки Microsoft/Recognizer-Text для синтаксического анализа и проверки пользовательского ввода. Дополнительные сведения см. на странице [Использование Azure DNS для частных доменов](https://github.com/Microsoft/Recognizers-Text#microsoft-recognizers-text-overview).

## <a name="next-steps"></a>Дополнительная информация

Теперь вы умеете настраивать состояние, которое упрощает чтение и запись данных бота в хранилище. Давайте перейдем к изучению методов, позволяющих задать пользователю ряд вопросов, проверить его ответы и сохранить полученные данные.

> [!div class="nextstepaction"]
> [Создание собственных запросов на сбор данных, вводимых пользователем](bot-builder-primitive-prompts.md).

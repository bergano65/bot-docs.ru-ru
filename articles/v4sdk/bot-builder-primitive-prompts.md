---
title: Создание собственных запросов на сбор данных, вводимых пользователем — Служба Azure Bot
description: Сведения о том, как управлять потоком общения с помощью простых запросов из пакета SDK Bot Framework.
keywords: conversation flow, prompts, conversation state, user state, custom prompts
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8c0572c7c92abfc99ab01c1d734d5ed0dcc463bb
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791303"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>Создание собственных запросов на сбор данных, вводимых пользователем

[!INCLUDE[applies-to](../includes/applies-to.md)]

Общение между ботом и пользователем часто подразумевает запрашивание у пользователя информации, анализ его ответа и выполнение действий с учетом этой информации. Бот должен отслеживать контекст общения, чтобы управлять его ходом и запоминать ответы на предыдущие вопросы. *Состояние* бота — это информация, которую программа отслеживает для правильных ответов на входящие сообщения. 

> [!TIP]
> Библиотека диалогов содержит встроенные запросы, которые предоставляют пользователям дополнительные функциональные возможности. Примеры таких запросов можно найти в статье [о реализации последовательного потока беседы](bot-builder-dialog-manage-conversation-flow.md).

## <a name="prerequisites"></a>предварительные требования

- Код в этой статье основан на примере запроса на ввод данных пользователем. Вам потребуется копия примера для **[C#](https://aka.ms/cs-primitive-prompt-sample), [JavaScript](https://aka.ms/js-primitive-prompt-sample) или [Python](https://aka.ms/python-primitive-prompt-sample)** .
- Понимание принципов [управления состоянием](bot-builder-concept-state.md) и [сохранения данных пользователя и диалога](bot-builder-howto-v4-state.md).

## <a name="about-the-sample-code"></a>Сведения о примере кода

Этот пример бота задает пользователю несколько вопросов, а затем проверяет и сохраняет введенные данные. На следующей схеме показана связь между ботом, профилем пользователя и классами потоков беседы. 

## <a name="ctabcsharp"></a>[C#](#tab/csharp)
![custom-prompts](media/CustomPromptBotSample-Overview.png)

- Класс `UserProfile` для хранения собранных ботом сведений о пользователе.
- Класс `ConversationFlow` для управления состоянием беседы при сборе сведений о пользователе.
- Внутреннее перечисление `ConversationFlow.Question` для отслеживания текущего положения в беседе.

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
![custom-prompts](media/CustomPromptBotSample-JS-Overview.png)

- Класс `userProfile` для хранения собранных ботом сведений о пользователе.
- Класс `conversationFlow` для управления состоянием беседы при сборе сведений о пользователе.
- Внутреннее перечисление `conversationFlow.question` для отслеживания текущего положения в беседе.

## <a name="pythontabpython"></a>[Python](#tab/python)
![custom-prompts](media/CustomPromptBotSample-Python-Overview.png)

- Класс `UserProfile` для хранения собранных ботом сведений о пользователе.
- Класс `ConversationFlow` для управления состоянием беседы при сборе сведений о пользователе.
- Внутреннее перечисление `ConversationFlow.Question` для отслеживания текущего положения в беседе.

---

Состояние пользователя будет отслеживать имя пользователя, возраст и выбранную им дату, а состояние беседы будет отслеживать, что мы уже спрашивали у пользователя.
Мы не планируем развертывать этот бот, поэтому состояние беседы и пользователя будем хранить во _временной памяти_. 

Мы будем управлять потоком беседы и процессом сбора данных, используя обработчик шагов для сообщений бота и свойства состояний пользователя и беседы. В нашем боте мы сохраняем полученные сведения о состоянии свойств на каждой итерации обработчика шагов для сообщений.

## <a name="create-conversation-and-user-objects"></a>Создание объектов беседы и пользователя

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

При запуске создаются объекты состояния пользователя и беседы, а в конструктор бота добавляются зависимости. 

**Startup.cs.**  
[!code-csharp[Startup](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Startup.cs?range=28-35)]

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=21-28)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В **index.js** создайте свойства состояния и бот, затем вызовите метод бота `run` из `processActivity`.

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/index.js?range=33-39)]

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/index.js?range=63-69)]

## <a name="pythontabpython"></a>[Python](#tab/python)

В файле **app.py**создайте свойства состояния и бота.

[!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/app.py?range=66-72)]

[!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/app.py?range=75-76)]

---

## <a name="create-property-accessors"></a>Создание методов доступа к свойствам

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Сначала мы создадим методы доступа к свойствам, которые предоставят нам дескриптор для `BotState` в методе `OnMessageActivityAsync`. Затем мы вызовем метод `GetAsync`, чтобы получить ключ с правильной областью действия:

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-37)]

Наконец, мы сохраним данные с помощью метода `SaveChangesAsync`.

[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=41-44)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

В конструкторе мы создадим методы доступа к свойствам состояния и настроим объекты управления состоянием (созданные выше) для нашей беседы.

**bots/customPromptBot.js**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=23-29)]

Затем мы определим второй обработчик `onDialog`, который будет выполняться после основного обработчика сообщений (как описано в следующем разделе). Этот второй обработчик гарантирует сохранение состояния на всех этапах.

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=41-48)]

## <a name="pythontabpython"></a>[Python](#tab/python)

В конструкторе мы создадим методы доступа к свойствам состояния и настроим объекты управления состоянием (созданные выше) для нашей беседы.

**bots/custom_prompt_bot.py** [!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=40-44)]

Затем сохраните данные с помощью метода `save_changes()`.
[!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=53-55)]

---

## <a name="the-bots-message-turn-handler"></a>Обработчик шагов бота для сообщений

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Чтобы обрабатывать действия сообщений, мы используем вспомогательный метод _FillOutUserProfileAsync()_ перед сохранением состояния с помощью _SaveChangesAsync()_ . Ниже приведен полный код.

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-44)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Для обработки действий сообщений мы зададим данные для беседы и пользователя, а затем вызовем вспомогательный метод `fillOutUserProfile()`. Ниже приведен полный код для обработчика шагов.

**bots/customPromptBot.js**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=31-39)]

## <a name="pythontabpython"></a>[Python](#tab/python)
Для обработки действий сообщений мы зададим данные для беседы и пользователя, а затем вызовем вспомогательный метод `_fill_out_user_profile`. Ниже приведен полный код для обработчика шагов.

**bots/custom_prompt_bot.py** [!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=46-55)]

---

## <a name="filling-out-the-user-profile"></a>Заполнение профиля пользователя

Мы начнем со сбора сведений. Интерфейсы для всех данных будут схожими.

- Возвращаемое значение позволяет узнать, содержат ли входные данные допустимый ответ на этот вопрос.
- Если проверка проходит успешно, она возвращает извлеченное и нормализованное значение для сохранения.
- Если процедура проверки завершается ошибкой, она создает сообщение, которое бот может отправить для повторного запроса той же информации.

 В следующем разделе мы определим вспомогательные методы для анализа и проверки пользовательского ввода.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=46-103)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.js**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=52-116)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**bots/custom_prompt_bot.py** [!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=57-126)]

---

## <a name="parse-and-validate-input"></a>Синтаксический анализ и проверка входных данных

Мы будем использовать для проверки входных данных следующие критерии.

- **Name** (имя) не может быть пустой строкой. Для нормализации выполняется усечение пробелов.
- Значение **age** (возраст) должно находиться в диапазоне от 18 до 120. Для нормализации оно округляется до целого числа.
- **Date** (дата) может быть любой датой или временем в будущем, если разница между указанным значением и текущим временем составляет не менее одного часа.
  Для нормализации возвращается только дата, содержащаяся во входных данных.

> [!NOTE]
> При вводе даты и возраста мы используем библиотеку [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) для начальной обработки.
> Мы предоставляем здесь пример кода, но не описываем работу библиотек средств распознавания текста. Можно использовать и другие варианты синтаксического анализа входных данных.
> Дополнительные сведения об этих библиотеках можно найти в репозитории **README**.

## <a name="ctabcsharp"></a>[C#](#tab/csharp)

Добавьте в бот следующие методы проверки.

**Bots/CustomPromptBot.cs**  
[!code-csharp[custom prompt bot](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=105-203)]

## <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.cs**  
[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=118-189)]

## <a name="pythontabpython"></a>[Python](#tab/python)

**bots/custom_prompt_bot.py** [!code-python[custom prompt bot](~/../botbuilder-python/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=127-189)]
---

## <a name="test-the-bot-locally"></a>Локальная проверка бота
Скачайте и установите [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme) для локального тестирования бота.

1. Выполните этот пример на локальном компьютере. Дополнительные инструкции, включая примеры для [C#](https://aka.ms/cs-primitive-prompt-sample), [JS](https://aka.ms/js-primitive-prompt-sample) и [Python](https://aka.ms/python-primitive-prompt-sample), см. в файле README.
1. Выполните тестирование в эмуляторе, как показано ниже.

![primitive-prompts](media/primitive-prompts.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

[Библиотека Dialogs](bot-builder-concept-dialog.md) предоставляет классы, которые автоматизируют многие аспекты управления беседами. 

## <a name="next-step"></a>Следующий шаг

> [!div class="nextstepaction"]
> [Реализация процесса общения](bot-builder-dialog-manage-conversation-flow.md)

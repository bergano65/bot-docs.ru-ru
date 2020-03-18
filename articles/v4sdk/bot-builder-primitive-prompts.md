---
title: Создание собственных запросов на сбор данных, вводимых пользователем — Служба Azure Bot
description: Сведения о том, как управлять потоком общения с помощью простых запросов из пакета SDK Bot Framework.
keywords: conversation flow, prompts, conversation state, user state, custom prompts
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/7/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: a3742837f15038c0e81d1b1405eeff96ba9efdf1
ms.sourcegitcommit: 2109d9da53fdf65966f33ed1fa628a40ec851d35
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2020
ms.locfileid: "78280175"
---
# <a name="create-your-own-prompts-to-gather-user-input"></a>Создание собственных запросов на сбор данных, вводимых пользователем

[!INCLUDE[applies-to](../includes/applies-to.md)]

Общение между ботом и пользователем часто подразумевает запрашивание у пользователя информации, анализ его ответа и выполнение действий с учетом этой информации. Бот должен отслеживать контекст общения, чтобы управлять его ходом и запоминать ответы на предыдущие вопросы. *Состояние* бота — это информация, которую программа отслеживает для правильных ответов на входящие сообщения.

> [!TIP]
> Библиотека диалогов содержит встроенные запросы, которые предоставляют пользователям дополнительные функциональные возможности. Примеры таких запросов можно найти в статье [о реализации последовательного потока беседы](bot-builder-dialog-manage-conversation-flow.md).

## <a name="prerequisites"></a>Предварительные требования

- Код в этой статье основан на примере запроса на ввод данных пользователем. Вам потребуется копия примера для **[C#](https://aka.ms/cs-primitive-prompt-sample), [JavaScript](https://aka.ms/js-primitive-prompt-sample) или [Python](https://aka.ms/python-primitive-prompt-sample)** .
- Понимание принципов [управления состоянием](bot-builder-concept-state.md) и [сохранения данных пользователя и диалога](bot-builder-howto-v4-state.md).

## <a name="about-the-sample-code"></a>Сведения о примере кода

Этот пример бота задает пользователю несколько вопросов, а затем проверяет и сохраняет введенные данные. На следующей схеме показана связь между ботом, профилем пользователя и классами потоков беседы.

## <a name="c"></a>[C#](#tab/csharp)

![custom-prompts](media/CustomPromptBotSample-Overview.png)

- Класс `UserProfile` для хранения собранных ботом сведений о пользователе.
- Класс `ConversationFlow` для управления состоянием беседы при сборе сведений о пользователе.
- Внутреннее перечисление `ConversationFlow.Question` для отслеживания текущего положения в беседе.

## <a name="javascript"></a>[JavaScript](#tab/javascript)

![custom-prompts](media/CustomPromptBotSample-JS-Overview.png)

- Класс `userProfile` для хранения собранных ботом сведений о пользователе.
- Класс `conversationFlow` для управления состоянием беседы при сборе сведений о пользователе.
- Внутреннее перечисление `conversationFlow.question` для отслеживания текущего положения в беседе.

## <a name="python"></a>[Python](#tab/python)

![custom-prompts](media/CustomPromptBotSample-Python-Overview.png)

- Класс `UserProfile` для хранения собранных ботом сведений о пользователе.
- Класс `ConversationFlow` для управления состоянием беседы при сборе сведений о пользователе.
- Внутреннее перечисление `ConversationFlow.Question` для отслеживания текущего положения в беседе.

---

Состояние пользователя будет отслеживать имя пользователя, возраст и выбранную им дату, а состояние беседы будет отслеживать, что мы уже спрашивали у пользователя.
Мы не планируем развертывать этот бот, поэтому состояние беседы и пользователя будем хранить во _временной памяти_.

Мы будем управлять потоком беседы и процессом сбора данных, используя обработчик шагов для сообщений бота и свойства состояний пользователя и беседы. В нашем боте мы сохраняем полученные сведения о состоянии свойств на каждой итерации обработчика шагов для сообщений.

## <a name="create-conversation-and-user-objects"></a>Создание объектов беседы и пользователя

## <a name="c"></a>[C#](#tab/csharp)

Создание объектов состояния пользователя и сеанса при запуске и их использование с помощью внедрения зависимостей в конструкторе бота.

**Startup.cs.**  
[!code-csharp[Startup.cs](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Startup.cs?range=28-35)]

**Bots/CustomPromptBot.cs**  
[!code-csharp[constructor](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=21-28)]

## <a name="javascript"></a>[JavaScript](#tab/javascript)

Создание объектов состояния пользователя и сеанса **index.js** и их использование в конструкторе бота.

**index.js** [!code-javascript[index.js](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/index.js?range=33-39)]

**bots/customPromptBot.js** [!code-javascript[constructor](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=20-22)]
[!code-javascript[constructor](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=27-29)]

## <a name="python"></a>[Python](#tab/python)

Создание объектов состояния пользователя и сеанса **app.py** и их использование в конструкторе бота.

**app.py**  
[!code-python[app.py](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/app.py?range=67-73)]

**bots/custom_prompt_bot.py**  
[!code-python[constructor](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=29-41)]

---

## <a name="create-property-accessors"></a>Создание методов доступа к свойствам

## <a name="c"></a>[C#](#tab/csharp)

Создание методов доступа к свойствам профиля пользователя и потока диалога и вызов `GetAsync` для получения значения свойства из состояния.

**Bots/CustomPromptBot.cs** [!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-37)]

Перед завершением шага вызовите `SaveChangesAsync`, чтобы записать изменения состояния в хранилище.

[!code-csharp[OnMessageActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=41-44)]

## <a name="javascript"></a>[JavaScript](#tab/javascript)

Создание методов доступа к свойствам профиля пользователя и потока диалога и вызов `get` для получения значения свойства из состояния.

**bots/customPromptBot.js** [!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=31-33)]

Перед завершением шага вызовите `saveChanges`, чтобы записать изменения состояния в хранилище.

[!code-javascript[custom prompt bot](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=42-51)]

## <a name="python"></a>[Python](#tab/python)

В конструкторе мы создадим методы доступа к свойствам состояния и настроим объекты управления состоянием (созданные выше) для нашей беседы.

**bots/custom_prompt_bot.py**  
[!code-python[on_message_activity](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=46-49)]

Перед завершением шага вызовите `SaveChangesAsync`, чтобы записать изменения состояния в хранилище.

[!code-python[on_message_activity](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=53-55)]

---

## <a name="the-bots-message-turn-handler"></a>Обработчик шагов бота для сообщений

При обработке действий с сообщениями обработчик сообщений использует вспомогательный метод для управления диалогом и отправки запроса пользователю. Этот вспомогательный метод описан в следующем разделе.

## <a name="c"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs**  
[!code-csharp[message handler](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=30-44)]

## <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.js**  
[!code-javascript[message handler](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=31-39)]

## <a name="python"></a>[Python](#tab/python)

**bots/custom_prompt_bot.py**  
[!code-python[message handler](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=46-55)]

---

## <a name="filling-out-the-user-profile"></a>Заполнение профиля пользователя

Бот запрашивает у пользователя сведения на основе вопроса, который был задан ботом на предыдущем шаге (если такой вопрос был задан). Входные данные анализируются с помощью метода проверки.

Каждый метод проверки характеризуется следующим образом:

- Возвращаемое значение позволяет узнать, содержат ли входные данные допустимый ответ на этот вопрос.
- Если проверка проходит успешно, она возвращает извлеченное и нормализованное значение для сохранения.
- Если процедура проверки завершается ошибкой, она создает сообщение, которое бот может отправить для повторного запроса той же информации.

Методы проверки описаны в следующем разделе.

## <a name="c"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs**  
[!code-csharp[FillOutUserProfileAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=46-103)]

## <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.js**  
[!code-javascript[fillOutUserProfile](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=53-118)]

## <a name="python"></a>[Python](#tab/python)

**bots/custom_prompt_bot.py**  
[!code-python[_fill_out_user_profile](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=57-125)]

---

## <a name="parse-and-validate-input"></a>Синтаксический анализ и проверка входных данных

Для проверки входных данных бот использует указанные ниже критерии.

- **Name** (имя) не может быть пустой строкой. Для нормализации выполняется усечение пробелов.
- Значение **age** (возраст) должно находиться в диапазоне от 18 до 120. Для нормализации оно округляется до целого числа.
- **Date** (дата) может быть любой датой или временем в будущем, если разница между указанным значением и текущим временем составляет не менее одного часа.
  Для нормализации возвращается только дата, содержащаяся во входных данных.

> [!NOTE]
> При вводе даты и возраста мы используем библиотеку [Microsoft/Recognizers-Text](https://github.com/Microsoft/Recognizers-Text/) для начальной обработки.
> Мы предоставляем здесь пример кода, но не описываем работу библиотек средств распознавания текста. Можно использовать и другие варианты синтаксического анализа входных данных.
> Дополнительные сведения об этих библиотеках можно найти в репозитории **README**.

## <a name="c"></a>[C#](#tab/csharp)

**Bots/CustomPromptBot.cs**  
[!code-csharp[validation methods](~/../botbuilder-samples/samples/csharp_dotnetcore/44.prompt-users-for-input/Bots/CustomPromptBot.cs?range=105-203)]

## <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/customPromptBot.cs**  
[!code-javascript[validation methods](~/../botbuilder-samples/samples/javascript_nodejs/44.prompt-for-user-input/bots/customPromptBot.js?range=120-190)]

## <a name="python"></a>[Python](#tab/python)

**bots/custom_prompt_bot.py**  
[!code-python[validation methods](~/../botbuilder-samples/samples/python/44.prompt-for-user-input/bots/custom_prompt_bot.py?range=127-189)]

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

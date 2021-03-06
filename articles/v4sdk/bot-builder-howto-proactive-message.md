---
title: Отправка упреждающих уведомлений пользователям — Служба Azure Bot
description: Узнайте, как отправлять уведомления
keywords: proactive message, notification message, bot notification,
author: jonathanfingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/24/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 704a37569f5ed9017cd4a09618a4efe75469f4a4
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2020
ms.locfileid: "77441622"
---
# <a name="send-proactive-notifications-to-users"></a>Отправка упреждающих уведомлений пользователям

[!INCLUDE[applies-to](../includes/applies-to.md)]

Как правило, каждое сообщение, которое бот отправляет пользователю, напрямую связано с данными, введенными пользователем ранее.
Иногда боту может потребоваться отправить пользователю сообщение, которое не имеет прямого отношения к текущей теме диалога или последнему сообщению пользователя. Такие сообщения называются _упреждающими_.

Упреждающие сообщения можно использовать в различных сценариях. Например, если пользователь ранее попросил бот отслеживать стоимость продукта, то бот может оповестить пользователя в случае, если стоимость продукта опустилась на 20 %. Если боту требуется какое-то время, чтобы сформировать ответ на вопрос пользователя, он может проинформировать пользователя о задержке и продолжить диалог. Когда бот сформирует ответ на вопрос, он передаст его пользователю.

Если вы реализуете в боте упреждающие сообщения, не отправляйте несколько таких сообщений за короткий промежуток времени. Некоторые каналы ограничивают частоту, с которой бот может отправлять сообщения пользователю, и отключают бот, если он нарушает эти ограничения.

Динамическое упреждающее сообщение — это самый простой тип упреждающих сообщений. Бот просто вставляет сообщение в диалог каждый раз, когда он активируется (вне зависимости от участия пользователя в отдельном разделе диалога с ботом в данный момент), и не пытается изменить диалог каким-либо способом.

Чтобы улучшить обработку уведомлений, рассмотрите другие варианты их интеграции в поток общения, например указав флаг в состоянии диалога или добавив уведомление в очередь.

## <a name="prerequisites"></a>Предварительные требования

- Понимание [основных принципов работы ботов](bot-builder-basics.md).
- Копия примера упреждающих сообщений для [**C#** ](https://aka.ms/proactive-sample-cs), [**JavaScript**](https://aka.ms/proactive-sample-js) и [**Python**](https://aka.ms/bot-proactive-python-sample-code). Этот пример используется в статье в качестве иллюстрации упреждающего обмена сообщениями.

## <a name="about-the-proactive-sample"></a>Сведения о примере с упреждающими сообщениями

Этот пример содержит бот и дополнительный контроллер, который используется для отправки в бот упреждающих сообщений, как показано на следующем рисунке.

![Упреждающий бот](media/proactive-sample-bot.png)

## <a name="retrieve-and-store-conversation-reference"></a>Получение и сохранение ссылки на беседу

Когда эмулятор подключается к боту, этот бот получает два действия обновления беседы. В обработчике бота для действий обновления беседы извлекается ссылка на беседу, которая сохраняется в словаре, как показано ниже.

# <a name="c"></a>[C#](#tab/csharp)

**Bots\ProactiveBot.cs**

[!code-csharp[OnConversationUpdateActivityAsync](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Bots/ProactiveBot.cs?range=26-37&highlight=3-4,9)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**bots/proactiveBot.js**

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=13-17&highlight=2)]

[!code-javascript[onConversationUpdateActivity](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/bots/proactiveBot.js?range=41-44&highlight=2-3)]

# <a name="python"></a>[Python](#tab/python)

**bots/proactive_bot.py** [!code-python[on_conversation_update_activity](~/../botbuilder-samples/samples/python/16.proactive-messages/bots/proactive_bot.py?range=14-16&highlight=2)]

[!code-python[on_conversation_update_activity](~/../botbuilder-samples/samples/python/16.proactive-messages/bots/proactive_bot.py?range=35-45)]

---

Примечание. В реальной системе ссылки на беседы следует хранить в базе данных, а не в объекте в памяти.

Ссылка на беседу имеет свойство _conversation_, в котором указано, к какой беседе относится действие. У беседы есть свойство _user_, которое содержит список пользователей, участвующих в беседе, а также свойство _service URL_, в котором каналы сохраняют URL-адрес для отправки ответов на текущее действие. Для отправки пользователям упреждающих сообщений нужно получить допустимую ссылку на беседу.

## <a name="send-proactive-message"></a>Отправка упреждающих сообщений

Второй контроллер (_notify_) отвечает за отправку в бот упреждающего сообщения. Чтобы создать упреждающее сообщение, выполните следующие действия.

1. Получите ссылку на беседу, в которую отправляется упреждающее сообщение.
1. Вызовите метод _продолжения беседы_ из адаптера, предоставив ссылку на беседу и делегат обработчика шагов, которые он сможет использовать. Метод продолжения беседы создает контекст шага для той беседы, на которую указывает ссылка, а затем вызывает указанный делегат обработчика шагов.
1. В этом делегате контекст шага используется для отправки упреждающего сообщения.

# <a name="c"></a>[C#](#tab/csharp)

**Controllers\NotifyController.cs**

При каждом запросе страницы уведомления бота соответствующий контроллер извлекает из словаря ссылки на беседы.
Этот контроллер выполняет методы `ContinueConversationAsync` и `BotCallback`, чтобы отправить упреждающее сообщение.

[!code-csharp[Notify logic](~/../botbuilder-samples/samples/csharp_dotnetcore/16.proactive-messages/Controllers/NotifyController.cs?range=17-62&highlight=28,40-44)]

Для отправки упреждающего сообщения адаптеру нужен идентификатор приложения бота. В рабочей среде для этого можно использовать реальный идентификатор приложения бота. В локальной тестовой среде подойдет любой произвольный идентификатор. Если боту еще не назначен идентификатор приложения, контроллер уведомления самостоятельно генерирует заполнитель и применяет его в вызове.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

**index.js**

При каждом запросе страницы сервера `/api/notify` этот сервер извлекает из словаря ссылки на беседы.
Затем сервер выполняет метод `continueConversation`, чтобы отправить упреждающее сообщение.
Параметр `continueConversation` является функцией, которая используется как обработчик шага бота для этого шага.

[!code-javascript[Notify logic](~/../botbuilder-samples/samples/javascript_nodejs/16.proactive-messages/index.js?range=68-82&highlight=4-8)]

# <a name="python"></a>[Python](#tab/python)

При каждом запросе страницы уведомлений бота этот сервер извлекает из словаря ссылки на беседы.
Затем сервер использует `_send_proactive_message`, чтобы отправить упреждающее сообщение.

[!code-python[Notify logic](~/../botbuilder-samples/samples/python/16.proactive-messages/app.py?range=97-105&highlight=5-9)]

---

## <a name="test-your-bot"></a>Тестирование бота

1. Установите [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme), если вы этого еще не сделали.
1. Выполните этот пример на локальном компьютере.
1. Запустите эмулятор и подключите его к боту.
1. Загрузите страницу бота api/notify. Это действие создает в эмуляторе упреждающее сообщение.

## <a name="additional-information"></a>Дополнительные сведения

Кроме примера, который используется в этой статье, на [GitHub](https://github.com/Microsoft/BotBuilder-Samples/) есть и другие примеры для C# и JavaScript.

### <a name="avoiding-401-unauthorized-errors"></a>Предотвращение ошибки 401 Unauthorized (не авторизовано)

По умолчанию пакет SDK BotBuilder добавляет `serviceUrl` в список имен доверенных узлов, если входящий запрос прошел проверку BotAuthentication. Эти данные сохраняются в кэше в памяти. При перезапуске бота все пользователи, ожидающие упреждающих сообщений, не смогут их получить, если не отправляли боту новые сообщения после перезапуска.

Чтобы избежать этого, следует вручную добавить `serviceUrl` в список имен доверенных узлов, выполнив следующие действия.

# <a name="c"></a>[C#](#tab/csharp)

```csharp
MicrosoftAppCredentials.TrustServiceUrl(serviceUrl);
```

Для упреждающего обмена сообщениями `serviceUrl` используется как URL-адрес канала, с которым взаимодействует получатель упреждающего сообщения. Его можно найти в `Activity.ServiceUrl`.

Добавьте приведенный выше код прямо перед кодом, который отправляет упреждающее сообщение. В [примере упреждающих сообщений](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/16.proactive-messages) это нужно сделать в `NotifyController.cs` прямо перед `await turnContext.SendActivityAsync("proactive hello");`.

# <a name="javascript"></a>[JavaScript](#tab/javascript)

```js
MicrosoftAppCredentials.trustServiceUrl(serviceUrl);
```

Для упреждающего обмена сообщениями `serviceUrl` используется как URL-адрес канала, с которым взаимодействует получатель упреждающего сообщения. Его можно найти в `activity.serviceUrl`.

Добавьте приведенный выше код прямо перед кодом, который отправляет упреждающее сообщение. В [примере упреждающих сообщений](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/16.proactive-messages) это нужно сделать в `index.js` прямо перед `await turnContext.sendActivity('proactive hello');`.

# <a name="python"></a>[Python](#tab/python)

```python
MicrosoftAppCredentials.trustServiceUrl(serviceUrl)
```

Для упреждающего обмена сообщениями `serviceUrl` используется как URL-адрес канала, с которым взаимодействует получатель упреждающего сообщения. Его можно найти в `activity.serviceUrl`.

Добавьте приведенный выше код прямо перед кодом, который отправляет упреждающее сообщение. В [примере упреждающих сообщений](https://aka.ms/bot-proactive-python-sample-code) добавьте его в `app.py` перед отправкой *упреждающего приветственного сообщения*.

---

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Реализация процесса общения](bot-builder-dialog-manage-conversation-flow.md)

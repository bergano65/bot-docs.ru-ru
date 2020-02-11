---
title: Реализация навыка | Документация Майкрософт
description: Сведения о реализации навыка с помощью пакета SDK Bot Framework.
keywords: навыки
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/27/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 057e4fdf1846d2480d1a50f679bf66c269f0fe26
ms.sourcegitcommit: 36d6f06ffafad891f6efe4ff7ba921de8a306a94
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/31/2020
ms.locfileid: "76895835"
---
# <a name="implement-a-skill"></a>Реализация навыка

[!INCLUDE[applies-to](../includes/applies-to.md)]

Вы можете использовать навыки для расширения функциональности другого бота.
_Навыком_ здесь называется бот, который может выполнять ряд задач для другого бота.

- Интерфейс навыка описывается манифестом. Разработчики, у которых нет доступа к исходному коду навыка, могут использовать информацию из этого манифеста для разработки потребителя навыка.
- Навык может использовать проверку утверждений для управления доступом от других ботов и пользователей.

В этой статье демонстрируется реализация навыка, который выводит на экран вводимые пользователем данные.

<!-- I haven't discussed passing values back-and-forth mid conversation. That could be the basis of another article. -->

## <a name="prerequisites"></a>Предварительные требования

- [Базовые знания о ботах](bot-builder-basics.md) и [навыках](skills-conceptual.md).
- Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начать работу.
- Копия примера **простого навыка для ботов** для языка [**C#** ](https://aka.ms/skills-simple-bot-to-bot-csharp), [**JavaScript**](https://aka.ms/skills-simple-bot-to-bot-js) или [**Python**](https://aka.ms/skills-simple-bot-to-bot-python).

## <a name="about-this-sample"></a>Об этом примере

В пример **простого навыка для ботов** включены проекты двух ботов:

- _бот эхо-навыка_, который реализует этот навык;
- _простой корневой бот_, который реализует бот для использования этого навыка.

Эта статья посвящена навыку, в том числе логике поддержки в боте и адаптере.

### <a name="ctabcs"></a>[C#](#tab/cs)

![Схема классов навыка](./media/skills-simple-skill-cs.png)

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

![Схема классов навыка](./media/skills-simple-skill-js.png)

### <a name="pythontabpython"></a>[Python](#tab/python)

![Схема классов навыка](./media/skills-simple-skill-python.png)

---

Сведения о простом корневом боте см. в статье [о реализации потребителя навыка](skill-implement-consumer.md).

## <a name="resources"></a>Ресурсы

Для проверки подлинности в сценарии взаимодействия ботов требуется предоставить каждому из этих ботов действительный идентификатор приложения и пароль.

Чтобы протестировать навык в качестве бота, доступного для пользователей, зарегистрируйте этот навык в Azure. Для этого можно применить службу "Регистрация каналов бота". Дополнительные сведения см. в статье о [регистрации бота в службе Azure Bot](../bot-service-quickstart-registration.md).

## <a name="application-configuration"></a>Конфигурация приложений

Добавьте в файл конфигурации навыка идентификатор приложения и пароль для этого навыка.

Массив _allowed callers_ позволяет ограничить круг потребителей, имеющих доступ к этому навыку.
Оставьте этот массив пустым, чтобы запросы принимались от любого потребителя навыка.

### <a name="ctabcs"></a>[C#](#tab/cs)

**EchoSkillBot\appsettings.json**

Добавьте в файл appsettings.json идентификатор приложения и пароль для этого навыка.

[!code-csharp[configuration file](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/appsettings.json)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/.env**

Добавьте в ENV-файл идентификатор приложения и пароль для этого навыка.

[!code-javascript[configuration file](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/.env)]

### <a name="pythontabpython"></a>[Python](#tab/python)

Добавьте в файл config.py идентификатор приложения и пароль для этого навыка.

**config.py**

[!code-python[configuration file](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/config.py?range=14-19)]

---

## <a name="activity-handler-logic"></a>Логика обработчика действий

### <a name="to-accept-input-parameters"></a>Прием входных параметров

Потребитель навыка может отправить информацию в этот навык. Один из способов принять такую информацию — через свойство _value_ входящего сообщения. Другой способ — обработка действий события и вызова.

В нашем примере навык не принимает входные параметры.

### <a name="to-continue-or-complete-a-conversation"></a>Продолжение или завершение беседы

Когда навык отправляет действие, потребитель навыка должен перенаправить это действие пользователю.

Но при завершении работы навыка необходимо отправить действие `endOfConversation`, иначе потребитель навыка будет и далее пересылать действия пользователя в навык.
Можно также поместить возвращаемое значение в свойство _value_, а причину завершения навыка — в свойство _code_ обрабатываемого действия.

#### <a name="ctabcs"></a>[C#](#tab/cs)

**EchoSkillBot\Bots\EchoBot.cs**

[!code-csharp[Message handler](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/Bots/EchoBot.cs?range=14-29)]

#### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/bot.js**

[!code-javascript[Message handler](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/bot.js?range=10-26)]

#### <a name="pythontabpython"></a>[Python](#tab/python)

**echo-skill-bot/bots/echo_bot.py**

[!code-python[Message handler](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/bots/echo_bot.py?range=10-27)]

---

### <a name="to-cancel-the-skill"></a>Отмена выполнения навыка

В навыках с несколькими репликами необходимо также принимать от потребителя навыков действия `endOfConversation`, которые позволят потребителю отменить текущую беседу.

Логика навыка в нашем примере не меняется при последовательных репликах. Если реализуемый вами навык выделяет ресурсы для беседы, добавьте в обработчик завершения беседы код для очистки этих ресурсов.

#### <a name="ctabcs"></a>[C#](#tab/cs)

**EchoSkillBot\Bots\EchoBot.cs**

[!code-csharp[End-of-conversation handler](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/Bots/EchoBot.cs?range=31-37)]

#### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/bot.js**

 Чтобы добавить логику завершения беседы, используйте метод `onUnrecognizedActivityType`. В этом обработчике следует проверить, имеет ли нераспознанное действие `type` значение `endOfConversation`.

[!code-javascript[End-of-conversation handler](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/bot.js?range=28-35)]

#### <a name="pythontabpython"></a>[Python](#tab/python)

**echo-skill-bot/bots/echo_bot.py**

[!code-python[End-of-conversation handler](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/bots/echo_bot.py?range=29-33)]

---

## <a name="claims-validator"></a>Средство проверки утверждений

В нашем примере для проверки утверждений используется список разрешенных вызывающих объектов. Этот список определен в файле конфигурации навыка и считывается в объект средства проверки при его создании.

В конфигурацию проверки подлинности можно добавить _средство проверки утверждений_. Утверждения оцениваются после заголовка проверки подлинности. Код средства проверки должен создавать исключение или ошибку, чтобы отклонить запрос. Отклонение запроса, уже прошедшего проверку подлинности, может потребоваться по разным причинам. Пример:

- навык предоставляется как часть платной службы; пользователи, отсутствующие в базе данных, не должны иметь доступ к навыку;
- доступ к навыку ограничен правообладателем; вызвать навык могут только определенные потребители.

<!--TODO Need a link for more information about claims and claims-based validation.-->

### <a name="ctabcs"></a>[C#](#tab/cs)

Создайте класс средства проверки утверждений, производный от класса `ClaimsValidator`. Для отклонения входящего запроса создастся исключение `UnauthorizedAccessException`. Обратите внимание, что метод `IConfiguration.Get` возвращает значение NULL, если значение в файле конфигурации является пустым массивом.

**EchoSkillBot\Authentication\AllowedCallersClaimsValidator.cs**

[!code-csharp[Claims validator](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/Authentication/AllowedCallersClaimsValidator.cs?range=22-52&highlight=24-27)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Определите метод проверки утверждений, который выдает ошибку для отклонения входящего запроса.

**echo-skill-bot/authentication/allowedCallersClaimsValidator.js**

[!code-javascript[Claims validator](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/authentication/allowedCallersClaimsValidator.js?range=6-27&highlight=18-20)]

### <a name="pythontabpython"></a>[Python](#tab/python)

Определите метод проверки утверждений, который выдает ошибку для отклонения входящего запроса.

**echo-skill-bot/authentication/allowed_callers_claims_validator.py**

[!code-python[Claims validator](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/authentication/allowed_callers_claims_validator.py?range=7-28&highlight=17-22)]

---

## <a name="skill-adapter"></a>Адаптер навыка

При возникновении ошибки адаптер навыка должен очистить состояние беседы для этого навыка и отправить потребителю действие `endOfConversation`. Используйте свойство _code_ в этом действии, чтобы сообщить о завершении навыка из-за ошибки.

### <a name="ctabcs"></a>[C#](#tab/cs)

**EchoSkillBot\SkillAdapterWithErrorHandler.cs**

[!code-csharp[Error handler](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/SkillAdapterWithErrorHandler.cs?range=20-59&highlight=19-24)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/index.js**

[!code-javascript[Error handler](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/index.js?range=41-69&highlight=21-28)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**echo-skill-bot/app.py**

[!code-python[Error handler](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/app.py?range=38-69&highlight=27-32)]

---

## <a name="service-registration"></a>Регистрация службы

_Адаптер Bot Framework_ использует объект _конфигурации проверки подлинности_, который настраивается при создании адаптера, для проверки заголовков проверки подлинности во входящих запросах.

В нашем примере в конфигурацию проверки подлинности добавляется проверка утверждений, а также применяется _адаптер навыка с обработчиком ошибок_, который описан в предыдущем разделе.

### <a name="ctabcs"></a>[C#](#tab/cs)

**EchoSkillBot\Startup.cs**

[!code-csharp[Configuration](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/Startup.cs?range=28-32)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/index.js**

[!code-javascript[configuration](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/index.js?range=34-38)]

<!--C# & JS snippets checked 1/14-->
### <a name="pythontabpython"></a>[Python](#tab/python)

**app.py**

[!code-python[configuration](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/app.py?range=22-34)]


---

## <a name="skill-manifest"></a>Манифест навыка

_Манифест навыка_ — это файл в формате JSON с описанием действий, которые может выполнять навык, его входных и выходных параметров, а также конечных точек навыка.
Манифест содержит сведения, которые вам нужны для доступа к навыку из другого бота.

### <a name="ctabcs"></a>[C#](#tab/cs)

**EchoSkillBot\wwwroot\manifest\echoskillbot-manifest-1.0.json**

[!code-json[Manifest](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/EchoSkillBot/wwwroot/manifest/echoskillbot-manifest-1.0.json)]

### <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

**echo-skill-bot/manifest/echoskillbot-manifest-1.0.json**

[!code-json[Manifest](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/echo-skill-bot/manifest/echoskillbot-manifest-1.0.json)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**echo_skill_bot/wwwroot/manifest/echoskillbot-manifest-1.0.json**

[!code-json[Manifest](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/echo-skill-bot/wwwroot/manifest/echoskillbot-manifest-1.0.json)]

---

_Схема манифеста навыка_ представляет собой файл в формате JSON с описанием схемы манифеста навыков. Текущая версия схемы хранится в файле [skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json).

## <a name="test-the-skill"></a>Тестирование навыка

На этом этапе вы можете проверить работу навыка в эмуляторе, как для любого обычного бота. Но прежде, чем протестировать его как навык, необходимо [реализовать потребитель навыка](skill-implement-consumer.md).

Скачайте и установите последнюю версию [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

1. Запустите бот с эхо-навыком на локальном компьютере. Дополнительные инструкции для [C#](https://aka.ms/skills-simple-bot-to-bot-csharp), [JavaScript](https://aka.ms/skills-simple-bot-to-bot-js) и [Python](https://aka.ms/skills-simple-bot-to-bot-python) см. в файле сведений соответствующего примера.
1. Примените эмулятор для тестирования бота, как показано ниже. Обратите внимание, что при отправке навыку сообщения "end" или "stop" он дополняет ответное сообщение действием `endOfConversation`. Навык отправляет действие `endOfConversation`, чтобы обозначить завершение своей работы.

![Тестирование эхо-навыка](media/skills-simple-skill-test.png)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Реализация потребителя навыков](skill-implement-consumer.md)

---
title: Реализация потребителя навыка | Документация Майкрософт
description: Сведения о реализации потребителя навыка с помощью пакета SDK Bot Framework.
keywords: навыки
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/22/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 5dc2dc7327f739c8aee5fecd799aa562a6a98035
ms.sourcegitcommit: 4e1af50bd46debfdf9dcbab9a5d1b1633b541e27
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/25/2020
ms.locfileid: "76753738"
---
# <a name="implement-a-skill-consumer"></a>Реализация потребителя навыка

[!INCLUDE[applies-to](../includes/applies-to.md)]

Вы можете использовать навыки для расширения функциональности другого бота.
_Навыком_ здесь называется бот, который может выполнять ряд задач для другого бота и использует манифест для описания интерфейса.
_Корневым ботом_ называется бот, который взаимодействует с пользователем и может вызывать один или несколько навыков. Корневой бот — типичный вариант _потребителя навыка_.

- Потребитель навыка может использовать проверку утверждений для управления доступом от других навыков и пользователей.
- Потребитель навыка может использовать несколько навыков.
- Разработчики, у которых нет доступа к исходному коду навыка, могут использовать информацию из манифеста этого навыка для разработки потребителей.

В этой статье демонстрируется создание потребителя навыка, который использует эхо-навык для вывода на экран вводимых пользователем данных. Пример манифеста навыка и сведения о реализации эхо-навыка см. в статье [о реализации навыка](skill-implement-skill.md).

## <a name="prerequisites"></a>Предварительные требования

- Понимание [основ работы с ботами](bot-builder-basics.md), [принципа работы ботов с навыками](skills-conceptual.md) и [принципа реализации навыка](skill-implement-skill.md).
- Подписка Azure. Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начать работу.
- Копия примера **простого навыка для ботов** для языка [**C#** ](https://aka.ms/skills-simple-bot-to-bot-csharp), [**JavaScript**](https://aka.ms/skills-simple-bot-to-bot-js) или [**Python**](https://aka.ms/skills-simple-bot-to-bot-python).

## <a name="about-this-sample"></a>Об этом примере

В пример **простого навыка для ботов** включены проекты двух ботов:

- _бот эхо-навыка_, который реализует этот навык;
- _простой корневой бот_, который реализует бот для использования этого навыка.

Эта статья посвящена корневому боту, в том числе логике поддержки в объектах бота и адаптера, а также объектам, которые используются для обмена действиями с помощью навыков. К ним относятся следующие объекты.

- Клиент навыка, который используется для отправки действий в навык.
- Обработчик навыков, который используется для получения действий от навыка.
- Фабрика идентификаторов бесед с навыками, которую клиент и обработчик навыков используют для взаимного преобразования ссылок между беседами пользователя с корневым ботом и корневого бота с навыком.

### <a name="ctabcs"></a>[C#](#tab/cs)

![Диаграмма классов потребителя навыка](./media/skills-simple-root-cs.png)

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![Диаграмма классов потребителя навыка](./media/skills-simple-root-js.png)

### <a name="pythontabpython"></a>[Python](#tab/python)

![Диаграмма классов потребителя навыка](./media/skills-simple-root-python-2.png)

---

Сведения о боте с эхо-навыком см. в статье [о реализации навыка](skill-implement-skill.md).

## <a name="resources"></a>Ресурсы

Для проверки подлинности в сценарии взаимодействия ботов требуется предоставить каждому из этих ботов действительный идентификатор приложения и пароль.

Зарегистрируйте в Azure сам навык и потребитель навыка. Для этого можно применить службу "Регистрация каналов бота". Дополнительные сведения см. в статье о [регистрации бота в службе Azure Bot](../bot-service-quickstart-registration.md).

## <a name="application-configuration"></a>Конфигурация приложений

1. Добавьте идентификатор приложения и пароль для корневого бота.
1. Добавьте URL-адрес конечной точки, по которому навыки будут отправлять ответы на запросы потребителя навыка.
1. Добавьте запись для каждого навыка, который будет использоваться потребителем навыка. Каждая запись содержит следующие сведения:
   - идентификатор, который потребитель навыка использует для идентификации навыка;
   - идентификатор приложения навыка;
   - конечная точка обмена сообщениями для навыка.

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\appsettings.json**

Добавьте в файл appsettings.json идентификатор приложения и пароль для корневого бота. Кроме того, добавьте идентификатор приложения для бота с эхо-навыком в массив `BotFrameworkSkills`.

[!code-csharp[configuration file](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/appsettings.json)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**echo-skill-bot/.env**

Добавьте в ENV-файл идентификатор приложения и пароль для корневого бота. Кроме того, добавьте идентификатор приложения для бота с эхо-навыком.

[!code-javascript[configuration file](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/.env)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple_root_bot/config.py**

Добавьте в ENV-файл идентификатор приложения и пароль для корневого бота. Кроме того, добавьте идентификатор приложения для бота с эхо-навыком.

[!code-python[configuration file](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/config.py?range=16-29)]

---

## <a name="skills-configuration"></a>Настройка навыков

Наш пример бота считывает сведения о каждом навыке из файла конфигурации в коллекцию объектов _skill_.

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\SkillsConfiguration.cs**

[!code-csharp[skills configuration](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/SkillsConfiguration.cs?range=14-38)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/skillsConfiguration.js**

[!code-javascript[skills configuration](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/skillsConfiguration.js?range=7-33)]


### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/config.py**

[!code-python[skills configuration](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/config.py?range=32-36)]

---

## <a name="conversation-id-factory"></a>Фабрика идентификаторов беседы

Этот элемент создает идентификатор беседы, который используется для работы с навыком, и может восстановить исходный идентификатор беседы с пользователем по идентификатору беседы с навыком.

Фабрика идентификаторов бесед для этого примера поддерживает простой сценарий со следующими характеристиками:

- корневой бот предназначен для использования одного конкретного навыка;
- корневой бот поддерживает только одну активную беседу с навыком в конкретный момент времени.

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\SkillConversationIdFactory.cs**

[!code-csharp[Conversation ID factory](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/SkillConversationIdFactory.cs?range=17-40)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/skillConversationIdFactory.js**

[!code-javascript[Conversation ID factory](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/skillConversationIdFactory.js?range=10-29)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/skill_conversation_id_factory.py**

[!code-python[Conversation ID factory](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/skill_conversation_id_factory.py?range=9-47)]

---

В более сложных сценариях фабрика идентификаторов бесед должна поддерживать следующие действия:

- метод _создания идентификатора беседы с навыком_ получает или создает идентификатор беседы с навыком;
- метод _получения ссылки на беседу_ получает доступ к правильной беседе с пользователем.

## <a name="skill-client-and-skill-handler"></a>Клиент навыка и обработчик навыка

Потребитель навыка использует клиент навыка, чтобы пересылать действия в навык.
Для этого клиент навыка использует сведения о конфигурации навыков и фабрику идентификаторов бесед.

Потребитель навыка использует обработчик навыка для получения действий от навыка.
Обработчик использует для этого фабрику идентификаторов бесед, конфигурацию проверки подлинности и поставщик учетных данных, а также имеет зависимости от адаптера корневого бота и обработчика действий.

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\Startup.cs**

[!code-csharp[skill client and handler](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Startup.cs?range=42-43)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/index.js**

[!code-javascript[skill client](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=107-108)]

[!code-javascript[skill handler](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=132)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/app.py**

[!code-python[skill client](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=58)]

[!code-python[skill handler](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=120-122)]

---

Трафик HTTP от навыка поступает в конечную точку по URL-адресу службы, которую потребитель навыка сообщает этому навыку. Для передачи трафика обработчику навыка примените обработчик конечной точки для выбранного языка.

Обработчик навыка по умолчанию выполняет следующее:

- использует объект конфигурации проверки подлинности для аутентификации и проверки утверждений при взаимодействии ботов;
- использует фабрику идентификаторов бесед для преобразования ссылки на беседу потребителя с навыком в ссылку на беседу пользователя с корневым ботом;
- создает упреждающее сообщение, которое позволяет потребителю навыка восстановить контекст реплики в беседе пользователя с корневым ботом и пересылать действия пользователю.

## <a name="activity-handler-logic"></a>Логика обработчика действий

Обратите внимание, что логика потребителя навыка должна обеспечить следующее:

- запоминать наличие активных навыков и правильно передавать в них действия;
- обнаруживать запросы от пользователя, которые нужно передать в навык, и запускать соответствующий навык;
- обнаруживать действие `endOfConversation`, поступающее от любого активного навыка, чтобы зафиксировать его завершение;
- если это уместно, по запросу пользователя или потребителя навыка останавливать навык, который еще не завершил работу;
- сохранять состояние перед вызовом навыка, так как любой ответ может поступить к другому экземпляру потребителя навыка (в том числе балансировку нагрузки и т. п.)

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\Bots\RootBot.cs**

Корневой бот имеет зависимости от состояния беседы, сведений о навыках, клиента навыка и общей конфигурации. В ASP.NET эти объекты реализуются путем внедрения зависимостей.
Также корневой бот определяет метод доступа к свойству состояния беседы, чтобы отслеживать активные навыки.

[!code-csharp[Root bot dependencies](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Bots/RootBot.cs?range=21-55)]

В нашем примере есть вспомогательный метод для перенаправления действий в навык. Он сохраняет состояние беседы перед вызовом навыка и проверяет, успешно ли выполнен HTTP-запрос.

[!code-csharp[Send to skill](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Bots/RootBot.cs?range=125-139)]

Обратите внимание, что в корневой бот включена логика обработки сообщений от пользователя и действий `endOfConversation` от навыка.

[!code-csharp[message/end-of-conversation handlers](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Bots/RootBot.cs?range=57-112)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/rootBot.js**

Корневой бот имеет зависимости от состояния беседы, сведений о навыках и клиента навыка.
Также корневой бот определяет метод доступа к свойству состояния беседы, чтобы отслеживать активные навыки.

[!code-javascript[Root bot dependencies](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/rootBot.js?range=7-30)]

В нашем примере есть вспомогательный метод для перенаправления действий в навык. Он сохраняет состояние беседы перед вызовом навыка и проверяет, успешно ли выполнен HTTP-запрос.

[!code-javascript[Send to skill](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/rootBot.js?range=108-120)]

Обратите внимание, что в корневой бот включена логика обработки сообщений от пользователя и действий `endOfConversation` от навыка.

[!code-javascript[message/end-of-conversation handlers](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/rootBot.js?range=33-85)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/bots/root_bot.py**

Корневой бот имеет зависимости от состояния беседы, сведений о навыках, клиента навыка и общей конфигурации.
Также корневой бот определяет метод доступа к свойству состояния беседы, чтобы отслеживать активные навыки.

[!code-python[Root bot dependencies](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/bots/root_bot.py?range=23-37)]

В нашем примере есть вспомогательный метод для перенаправления действий в навык. Он сохраняет состояние беседы перед вызовом навыка и проверяет, успешно ли выполнен HTTP-запрос.

[!code-python[Send to skill](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/bots/root_bot.py?range=104-117)]

Обратите внимание, что в корневой бот включена логика обработки сообщений от пользователя и действий `endOfConversation` от навыка.

[!code-python[Handled activities](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/bots/root_bot.py?range=39-93)]

---


## <a name="on-turn-error-handler"></a>Глобальный обработчик ошибок с репликами

При возникновении ошибки адаптер очищает состояние беседы, чтобы сбросить параметры беседы с пользователем и избавиться от состояния ошибки.

Мы рекомендуем всегда отправлять сообщение _о завершении беседы_ всем активным навыкам, прежде чем очищать состояние беседы в потребителе навыка. Это позволит навыку освободить все ресурсы, связанные с беседой между потребителем и навыком прежде, чем потребитель очистит эту беседу.

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\AdapterWithErrorHandler.cs**

В нашем примере логика обработки ошибок с репликами распределена между несколькими вспомогательными методами.

[!code-csharp[On turn error](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/AdapterWithErrorHandler.cs?range=40-120)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/index.js**


[!code-javascript[On turn error](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=34-87)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**app.py**

[!code-python[On turn error](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=62-115)]

---

## <a name="skills-endpoint"></a>Конечная точка навыка

Бот определяет конечную точку, которая перенаправляет входящие действия навыка в обработчик навыка корневого бота.


### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\Controllers\SkillController.cs**

[!code-csharp[skill endpoint](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Controllers/SkillController.cs?range=15-23)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/index.js**


[!code-javascript[skill endpoint](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=133-134)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/app.py**

[!code-python[skill endpoint](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=144)]

---

## <a name="service-registration"></a>Регистрация службы

Добавьте объект конфигурации проверки подлинности со всеми необходимыми проверками утверждений, а также любые дополнительные объекты.

### <a name="ctabcs"></a>[C#](#tab/cs)

**SimpleRootBot\Startup.cs**

[!code-csharp[services](~/../botbuilder-samples/samples/csharp_dotnetcore/80.skills-simple-bot-to-bot/SimpleRootBot/Startup.cs?range=22-53)]

### <a name="javascripttabjs"></a>[JavaScript](#tab/js)

**simple-root-bot/index.js**

[!code-javascript[services](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=27-31)]

[!code-javascript[services](~/../botbuilder-samples/samples/javascript_nodejs/80.skills-simple-bot-to-bot/simple-root-bot/index.js?range=95-134)]

### <a name="pythontabpython"></a>[Python](#tab/python)

**simple-root-bot/app.py**

[!code-python[services](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=35-58)]

[!code-python[services](~/../botbuilder-samples/samples/python/80.skills-simple-bot-to-bot/simple-root-bot/app.py?range=118-144)]

---

## <a name="test-the-root-bot"></a>Тестирование корневого бота

Вы можете протестировать потребитель навыка в эмуляторе, как любой обычный бот, но при этом навык и потребитель навыка должны одновременно работать в режиме ботов.
Сведения о настройке навыка см. в статье [о реализации навыка](skill-implement-skill.md).

Скачайте и установите последнюю версию [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

1. Запустите бот эхо-навыка и простой корневой бот на локальном компьютере. Дополнительные инструкции для [C#](https://aka.ms/skills-simple-bot-to-bot-csharp), [JavaScript](https://aka.ms/skills-simple-bot-to-bot-js) и [Python](https://aka.ms/skills-simple-bot-to-bot-python) см. в файле сведений соответствующего примера.
1. Примените эмулятор для тестирования бота, как показано ниже. Обратите внимание, что при отправке навыку сообщения `end` или `stop` он отправляет корневому боту ответное сообщение, дополненное действием `endOfConversation`. Свойство _code_ действия `endOfConversation` указывает, что навык успешно завершен.

![Тестирование потребителя навыка](media/skills-simple-consumer-test.png)

## <a name="additional-information"></a>Дополнительные сведения

Ниже приведены некоторые аспекты, которые нужно учитывать при реализации более сложного корневого бота.

### <a name="to-allow-the-user-to-cancel-a-multi-step-skill"></a>Возможность отменить выполнение многоэтапного навыка

Корневой бот должен проверять сообщение пользователя, прежде чем перенаправить его активному навыку. Если пользователь хочет отменить текущий процесс, корневой бот отправляет в навык действие `endOfConversation`, а не само сообщение пользователя.

### <a name="to-exchange-data-between-the-root-and-skill-bots"></a>Обмен данными между корневым ботом и навыком

Чтобы отправить параметры навыку, потребитель навыка может задать свойство _value_ для сообщений, отправляемых в навык. Чтобы получить возвращаемые значения от навыка, потребитель навыка должен проверять свойство _value_, когда получает действие `endOfConversation` от навыка.

### <a name="to-use-multiple-skills"></a>Использование нескольких навыков

- Если навык уже активен, корневой бот должен определить активный навык и перенаправить сообщение пользователя в нужный навык.
- Если активных навыков нет, корневой бот должен определить навык для запуска (при его наличии), используя сведения о состоянии бота и сообщении пользователя.
- Если вы хотите, чтобы пользователь мог переключаться между несколькими параллельно выполняющимися навыками, корневой бот должен определять, с каким из активных навыков намерен взаимодействовать пользователь, прежде чем перенаправлять сообщение от пользователя.

<!--
## Next steps

TBD: Claims validation? Skill manifest?

> [!div class="nextstepaction"]
> [Add claims validation](skill-add-claims-validation.md)
-->

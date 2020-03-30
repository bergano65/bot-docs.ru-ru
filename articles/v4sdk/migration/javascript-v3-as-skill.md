---
title: Преобразование бота JavaScript версии 3 в навык
description: Сведения о том, как преобразовать существующие боты JavaScript версии 3 в навыки и использовать их в боте JavaScript версии 4.
keywords: JavaScript, миграция бота, навыки, бот версии 3
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/19/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6b29a60d757725e65ae716301e736a66d508332a
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117809"
---
# <a name="convert-a-javascript-v3-bot-to-a-skill"></a>Преобразование бота JavaScript версии 3 в навык

В этой статье описывается, как преобразовать два примера ботов JavaScript версии 3 в навыки и создать потребитель навыков версии 4, который может обращаться к этим навыкам.
Чтобы преобразовать бот .NET версии 3 в навык, воспользуйтесь [этой инструкцией](net-v3-as-skill.md).
Сведения о переносе бота JavaScript из версии 3 в версию 4 см. в [этой статье](conversion-javascript.md).

## <a name="prerequisites"></a>Предварительные требования

- Visual Studio Code.
- Node.js.
- Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
- Чтобы протестировать навыки, вам потребуется Bot Framework Emulator и локальные копии нужных ботов:
  - Навык эхо на JavaScript версии 3: [**Skills/v3-skill-bot**](https://aka.ms/v3-js-echo-skill).
  - Навык заказа версии 3 для JavaScript: [**Skills/v3-booking-bot-skill**](https://aka.ms/v3-js-booking-skill).
  - Пример потребителя навыка на JavaScript версии 4: [**Skills/v4-root-bot**](https://aka.ms/js-simple-root-bot).

## <a name="about-the-bots"></a>Сведения о ботах

В этой статье каждый бот версии 3 предназначен для использования в качестве навыка. Предоставляется также потребитель навыка версии 4, который позволяет протестировать боты, преобразованные в навыки.

- Программа **v3-skill-bot** возвращает все полученные сообщения. Работая как навык, она _завершается_ при получении сообщения end или stop. Бот, который мы будем преобразовывать, основан на минимальном примере бота версии 3.
- С помощью бота **v3-booking-bot-skill** пользователь может забронировать авиабилет или номер в гостинице. Работая как навык, бот после завершения отправляет собранные сведения обратно в родительский объект.

Кроме того, в примере потребителя навыков версии 4 **v4-root-bot** показано, как можно использовать и тестировать навыки.

Чтобы использовать потребитель навыков для тестирования навыков, все 3 бота должны выполняться одновременно. Боты можно тестировать локально с помощью Bot Framework Emulator, где каждый бот использует свой локальный порт.

## <a name="create-azure-resources-for-the-bots"></a>Создание ресурсов Azure для ботов

Для проверки подлинности в сценарии взаимодействия ботов требуется предоставить каждому из этих ботов действительный идентификатор приложения и пароль.

1. Создайте нужное количество регистраций каналов для ботов.
1. Запишите идентификатор приложения и пароль для каждой из них.

## <a name="conversion-process"></a>Процесс преобразования

Чтобы преобразовать существующий бот в бот-навык, нужно выполнить лишь несколько шагов, которые описаны в следующих разделах. Более подробные сведения о навыках см. [здесь](../skills-conceptual.md).

- Обновите файл `.env` бота, включив в него идентификатор приложения и пароль, а также добавив свойство _root bot app ID_.
- Добавьте проверку утверждений. Таким образом вы сможете ограничить доступ к навыку, предоставляя его только пользователям и корневому боту. Дополнительные сведения о стандартных и пользовательских проверках утверждений см. [здесь](#additional-information).
- Измените контроллер сообщений бота, чтобы обрабатывать действия `endOfConversation` от корневого бота.
- Измените код бота, чтобы он возвращал действие `endOfConversation` после завершения работы навыка.
- При каждом завершении работы навыка, если он еще поддерживает беседу или некоторые ресурсы, ему необходимо очистить состояние беседы и освободить ресурсы.
- При необходимости добавьте файл манифеста.
  Так как потребитель навыка не всегда имеет доступ к коду этого навыка, опишите в манифесте все действия, которые ваш навык умеет получать и создавать, все его входные и выходные параметры, а также конечные точки.
  Текущая схема манифеста хранится в файле [skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json).

## <a name="convert-the-echo-bot"></a>Преобразование эхо-бота

Пример [Skills/v3-skill-bot](https://aka.ms/v3-js-echo-skill) содержит эхо-бот версии 3, преобразованный в базовый навык.

1. Создайте простой проект бота для JavaScript версии 3 и импортируйте необходимые модули.

   **v3-skill-bot/app.js**

   [!code-javascript[require statements](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/app.js?range=5-7)]

1. Настройте локальное выполнение бота на порту 3979.

   **v3-skill-bot/app.js**

   [!code-javascript[Setup server and port](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/app.js?range=9-13)]

1. В файл конфигурации добавьте идентификатор приложения и пароль эхо-бота. Также добавьте свойство `ROOT_BOT_APP_ID` со значением идентификатора приложения простого корневого бота.

   **v3-skill-bot/.env**

   [!code[.env file](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/.env)]

1. Создайте для бота соединитель чата. В нашем примере используется стандартная конфигурация проверки подлинности. Присвойте параметру `enableSkills` значение `true`, чтобы этот бот мог использоваться в качестве навыка. `allowedCallers` содержит массив идентификаторов приложений для всех ботов, которым разрешено использовать этот навык. Если первым в этом массиве указано значение "*", использование навыка разрешается любому боту.

   **v3-skill-bot/app.js**

   [!code-javascript[chat connector](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/app.js?range=24-30&highlight=5-6)]

1. Обновите обработчик сообщений, чтобы он отправлял действие `endOfConversation` при завершении работы навыка пользователем.

   **v3-skill-bot/app.js**

   [!code-javascript[universal bot](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/app.js?range=35-46&highlight=6)]

1. Поскольку этот бот не поддерживает состояние беседы и не создает ресурсы для беседы, в нем не требуется обрабатывать действия `endOfConversation`, которые он будет получать от потребителя навыка.

1. Примените этот манифест для эхо-бота. В качестве идентификатора приложения конечной точки укажите идентификатор приложения бота.

   **v3-skill-bot/manifest/v3-skill-bot-manifest.json**

   [!code-json[manifest](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-skill-bot/manifest/v3-skill-bot-manifest.json?highlight=22)]

   См. сведения о [схеме манифеста навыка (skill-manifest-2.0.0.json)](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json).

## <a name="convert-the-booking-bot"></a>Преобразование бота бронирования

Пример [Skills/v3-booking-bot-skill](https://aka.ms/v3-js-booking-skill) содержит бот бронирования версии 3, преобразованный в базовый навык.
До преобразования этот бот был очень похож на пример [core-MultiDialogs](https://aka.ms/v3-js-core-multidialogs) версии 3.

1. Импортируйте необходимые модули.

   **v3-booking-bot-skill/app.js**

   [!code-javascript[require statements](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=2-7)]

1. Настройте локальное выполнение бота на порту 3980.

   **v3-booking-bot-skill/app.js**

   [!code-javascript[Setup server and port](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=9-13)]

1. В файл конфигурации добавьте идентификатор приложения и пароль бота бронирования. Также добавьте свойство `ROOT_BOT_APP_ID` со значением идентификатора приложения простого корневого бота.

   **v3-booking-bot-skill/.env**

   [!code[.env file](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/.env)]

1. Создайте для бота соединитель чата. В нашем примере используется пользовательская конфигурация проверки подлинности. Присвойте параметру `enableSkills` значение `true`, чтобы этот бот мог использоваться в качестве навыка. `authConfiguration` содержит объект пользовательской конфигурации проверки подлинности, который будет использоваться для аутентификации и проверки утверждений.

   **v3-booking-bot-skill/app.js**

   [!code-javascript[chat connector](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=18-24&highlight=5-6)]

   **v3-booking-bot-skill/allowedCallersClaimsValidator.js**

   Здесь реализовано пользовательское средство проверки утверждений, которое создает ошибку при неудачной проверке.

   [!code-javascript[custom claims validation](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/allowedCallersClaimsValidator.js?range=4-47&highlight=22,25,39,41)]

1. Обновите обработчик сообщений, чтобы он отправлял действие `endOfConversation` при завершении навыка. Обратите внимание, что `session.endConversation()` очищает состояние беседы, а не просто отправляет действие `endOfConversation`.

   **v3-booking-bot-skill/app.js**

   Реализуйте вспомогательную функцию, которая будет устанавливать в действии `endOfConversation` значения свойств `code` и `value`, а также очищать состояние беседы.
   Если бот управляет еще какими-то ресурсами для этой беседы, все их следует здесь освободить.

   [!code-javascript[endConversation function](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=122-134&highlight=12)]

   Когда пользователь завершает процесс, вспомогательный метод должен завершить работу навыка и вернуть собранные от пользователя данные.

   [!code-javascript[universal bot](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=39-40)]

   [!code-javascript[universal bot](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=72-77&highlight=4)]

   Вспомогательный метод так само будет вызываться, если пользователь завершит процесс досрочно.

   [!code-javascript[universal bot](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=86-101&highlight=9-10)]

   [!code-javascript[universal bot](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=107-108)]

1. Если навык отменяется потребителем навыка, потребитель отправляет действие `endOfConversation`. Обрабатывайте это действие и освобождайте все ресурсы, связанные с этой беседой.

   **v3-booking-bot-skill/app.js**

   [!code-javascript[endConversation function](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/app.js?range=110-115)]

1. Примените этот манифест для бота бронирования. В качестве идентификатора приложения конечной точки укажите идентификатор приложения бота.

   **v3-booking-bot-skill/manifest/v3-booking-bot-skill-manifest.json**

   [!code-json[manifest](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v3-booking-bot-skill/manifest/v3-booking-bot-skill-manifest.json?highlight=22)]

   См. сведения о [схеме манифеста навыка (skill-manifest-2.0.0.json)](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json).

## <a name="create-the-v4-root-bot"></a>Создание корневого бота версии 4

Этот простой корневой бот выполняет роль потребителя для двух навыков и позволяет убедиться, что шаги беседы проходят в соответствии с планом. Настройте локальное выполнение бота на порту 3978.

1. В файл конфигурации добавьте идентификатор приложения и пароль корневого бота. Добавьте идентификатор приложения для каждого навыка версии 3.

   **v4-root-bot/.env**

   [!code-json[configuration](~/../botbuilder-samples/MigrationV3V4/Node/Skills/v4-root-bot/.env?highlight=2-3,6,10)]

## <a name="test-the-root-bot"></a>Тестирование корневого бота

Скачайте и установите последнюю версию [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

1. Запустите все три бота на локальном компьютере.
1. Запустите эмулятор и подключите его к корневому боту.
1. Протестируйте работу потребителя и навыков.

## <a name="additional-information"></a>Дополнительные сведения

### <a name="bot-to-bot-authentication"></a>Аутентификация взаимодействия между ботами

Корневой бот и навык обмениваются данными по протоколу HTTP. Эта платформа использует токены носителя и идентификаторы приложения бота для идентификации каждого бота. Она использует объект конфигурации проверки подлинности для проверки заголовков проверки подлинности во входящих запросах. В конфигурацию проверки подлинности можно добавить средство проверки утверждений. Утверждения оцениваются после заголовка проверки подлинности. Код средства проверки должен создавать исключение или ошибку, чтобы отклонить запрос.

При создании соединителя чата добавьте в параметр настроек свойство `allowedCallers` или `authConfiguration`, чтобы включить проверку подлинности между ботами.

Свойство `allowedCallers` применяется стандартным средством проверки утверждений для этого соединителя чата. В качестве значения оно должно содержать массив идентификаторов приложений тех ботов, которым разрешено использовать этот навык. Присвойте первому элементу значение "*", чтобы любой бот мог вызвать этот навык.

Чтобы применить пользовательскую функцию поверки, присвойте ее значение полю `authConfiguration`. Эта функция должна принимать массив объектов утверждений и создавать ошибку, если проверка завершается неудачно. Пример средства проверки утверждений вы можете найти на шаге 4 раздела, посвященного [преобразованию бота бронирования](#convert-the-booking-bot).

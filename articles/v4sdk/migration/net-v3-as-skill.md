---
title: Преобразование бота .NET версии 3 в навык
description: Сведения о том, как преобразовать существующие боты .NET версии 3 в навыки и использовать их в боте .NET версии 4.
keywords: .NET, миграция бота, навыки, бот версии 3
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/19/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 2cf0398986203e131cf456344440cef3c2b90955
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117769"
---
# <a name="convert-a-net-v3-bot-to-a-skill"></a>Преобразование бота .NET версии 3 в навык

В этой статье описывается, как преобразовать три примера ботов .NET версии 3 в навыки и создать потребитель навыков версии 4, который может обращаться к этим навыкам.
Чтобы преобразовать бот JavaScript версии 3 в навык, воспользуйтесь [этой инструкцией](javascript-v3-as-skill.md).
Сведения о переносе бота .NET из версии 3 в версию 4 см. в [этой статье](conversion-framework.md).

## <a name="prerequisites"></a>Предварительные требования

- Visual Studio 2019.
- .NET Core 3.1.
- .NET Framework 4.6.1 или более поздней версии.
- Подписка Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.
- Копии примеров ботов для .NET версии 3, которые вы будете преобразовывать: эхо-бот, [**PizzaBot**](https://aka.ms/v3-cs-pizza-bot) и [**SimpleSandwichBot**](https://aka.ms/v3-cs-simple-sandwich-bot).
- Копия примера потребителя навыка для .NET версии 4: [**SimpleRootBot**](https://aka.ms/cs-simple-root-bot).

## <a name="about-the-bots"></a>Сведения о ботах

В этой статье каждый бот версии 3 обновляется для использования в качестве навыка. Предоставляется также потребитель навыка версии 4, который позволяет протестировать боты, преобразованные в навыки.

- Программа **EchoBot** возвращает все полученные сообщения. Работая как навык, она _завершается_ при получении сообщения end или stop.
  Преобразуемый бот основан на шаблоне проекта Bot Builder Echo Bot для версии 3.
- **PizzaBot** проводит пользователя через процесс заказа пиццы. Работая как навык, этот бот после завершения отправляет заказ пользователя обратно в родительский объект.
- **SimpleSandwichBot** проводит пользователя через процесс заказа сандвичей. Работая как навык, этот бот после завершения отправляет заказ пользователя обратно в родительский объект.

Кроме того, в примере потребителя навыков версии 4 **SimpleRootBot** показано, как можно использовать и тестировать навыки.

Чтобы использовать потребитель навыков для тестирования навыков, все 4 бота должны выполняться одновременно. Боты можно тестировать локально с помощью Bot Framework Emulator, где каждый бот использует свой локальный порт.

## <a name="create-azure-resources-for-the-bots"></a>Создание ресурсов Azure для ботов

Для проверки подлинности в сценарии взаимодействия ботов требуется предоставить каждому из этих ботов действительный идентификатор приложения и пароль.

1. Создайте нужное количество регистраций каналов для ботов.
1. Запишите идентификатор приложения и пароль для каждой из них.

## <a name="conversion-process"></a>Процесс преобразования

Чтобы преобразовать существующий бот в бот-навык, нужно выполнить лишь несколько шагов, которые описаны в следующих разделах. Более подробные сведения о навыках см. [здесь](../skills-conceptual.md).

- Обновите файл `web.config` бота, включив в него идентификатор приложения и пароль и добавив свойство _allowed callers_.
- Добавьте проверку утверждений. Таким образом вы сможете ограничить доступ к навыку, предоставляя его только пользователям и корневому боту. Дополнительные сведения о стандартных и пользовательских проверках утверждений см. [здесь](#additional-information).
- Измените контроллер сообщений бота, чтобы обрабатывать действия `endOfConversation` от корневого бота.
- Измените код бота, чтобы он возвращал действие `endOfConversation` после завершения работы навыка.
- При каждом завершении работы навыка, если он еще поддерживает беседу или некоторые ресурсы, ему необходимо очистить состояние беседы и освободить ресурсы.
- При необходимости добавьте файл манифеста.
  Так как потребитель навыка не всегда имеет доступ к коду этого навыка, опишите в манифесте все действия, которые ваш навык умеет получать и создавать, все его входные и выходные параметры, а также конечные точки.
  Текущая схема манифеста хранится в файле [skill-manifest-2.0.0.json](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json).

## <a name="convert-the-echo-bot"></a>Преобразование эхо-бота

1. Создайте новый проект из шаблона проекта **Bot Builder Echo Bot** для версии 3 и настройте его для использования порта 3979.

   1. Создайте проект.
   1. Откройте вкладку свойств этого проекта.
   1. Выберите категорию **Веб** и задайте для параметра **URL-адрес проекта** значение `http://localhost:3979/`.
   1. Сохраните изменения и закройте вкладку свойств.

1. В файл конфигурации добавьте идентификатор приложения и пароль эхо-бота. Там же, в параметрах приложения, добавьте свойство `EchoBotAllowedCallers` и присвойте ему значение идентификатора приложения простого корневого бота.

   **V3EchoBot\\Web.config**

   [!code-xml[app settings](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Web.config?range=11-16)]

1. Добавьте средство проверки утверждений и вспомогательный класс конфигурации проверки подлинности.

   **V3EchoBot\\Authentication\\CustomAllowedCallersClaimsValidator.cs**

   Этот пример реализует пользовательское средство проверки утверждений, а при неудачной проверке создает ошибку `UnauthorizedAccessException`.

   [!code-csharp[claims validator](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Authentication/CustomAllowedCallersClaimsValidator.cs?range=4-72&highlight=45,63,66)]

   **V3EchoBot\\Authentication\\CustomSkillAuthenticationConfiguration.cs**

   Этот пример загружает сведения о допустимых вызывающих из файла конфигурации и применяет `CustomAllowedCallersClaimsValidator` для проверки утверждения.

   [!code-csharp[allowed callers](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Authentication/CustomSkillAuthenticationConfiguration.cs?range=4-20&highlight=12-14)]

1. Обновите класс `MessagesController`.

   **V3EchoBot\\Controllers\\MessagesController.cs**

   Обновите инструкции using.

   [!code-csharp[using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Controllers/MessagesController.cs?range=4-15)]

   Замените атрибут класса `BotAuthentication` на `SkillBotAuthentication`. Используйте необязательный параметр `AuthenticationConfigurationProviderType`, чтобы применить пользовательский поставщик проверки утверждений.

   [!code-csharp[attribute](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Controllers/MessagesController.cs?range=19-21)]

   В методе `HandleSystemMessage` добавьте условие для обработки сообщения `endOfConversation`. Это позволит навыкам очищать состояние и освобождать ресурсы при завершении беседы по сигналу от потребителя навыков.

   [!code-csharp[on end of conversation](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Controllers/MessagesController.cs?range=45-65)]

1. Измените код бота так, чтобы навык мог отмечать завершение беседы при получении от пользователя сообщения end или stop. Также навык должен очищать состояние и освобождать ресурсы, когда он завершает беседу.

   **V3EchoBot\\Dialogs\\RootDialog.cs**

   [!code-csharp[message received](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/Dialogs/RootDialog.cs?range=21-41&highlight=5-13)]

1. Примените этот манифест для эхо-бота. В качестве идентификатора приложения конечной точки укажите идентификатор приложения бота.

   **V3EchoBot\\wwwroot\\echo-bot-manifest.json**

   [!code-json[manifest](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3EchoBot/wwwroot/echo-bot-manifest.json?highlight=22)]

   См. сведения о [схеме манифеста навыка (skill-manifest-2.0.0.json)](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json).

## <a name="convert-the-pizza-bot"></a>Преобразование бота для заказа пиццы

1. Откройте копию проекта PizzaBot и настройте в нем использование порта 3980.

   1. Откройте вкладку свойств этого проекта.
   1. Выберите категорию **Веб** и задайте для параметра **URL-адрес проекта** значение `http://localhost:3980/`.
   1. Сохраните изменения и закройте вкладку свойств.

1. В файл конфигурации добавьте идентификатор приложения и пароль бота для заказа пиццы. Там же, в параметрах приложения, добавьте свойство `AllowedCallers` и присвойте ему значение идентификатора приложения простого корневого бота.

   **V3PizzaBot\\Web.config**

   [!code-xml[app settings](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/Web.config?range=11-16)]

1. Добавьте класс `ConversationHelper` со вспомогательными методами, реализовав следующее:
   - Отправка действия `endOfConversation` по завершении навыка. Позволяет вернуть сведения о заказе в свойстве `Value` действия и задать свойство `Code`, чтобы обозначить причину завершения беседы.
   - Очистка состояния беседы и освобождение всех связанных ресурсов.

   **V3PizzaBot\\ConversationHelper.cs**

   [!code-csharp[conversation helper](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/ConversationHelper.cs?range=4-79)]

1. Обновите класс `MessagesController`.

   **V3PizzaBot\\Controllers\\MessagesController.cs**

   Обновите инструкции using.

   [!code-csharp[using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/Controllers/MessagesController.cs?range=4-16)]

   Замените атрибут класса `BotAuthentication` на `SkillBotAuthentication`. Этот бот использует стандартное средство проверки утверждений.

   [!code-csharp[attribute](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/Controllers/MessagesController.cs?range=20-21)]

   В методе `Post` измените условие действия `message`, чтобы разрешить пользователю отменить процесс заказа из навыка. Также добавьте условие действия `endOfConversation`, которое позволит навыкам очищать состояние и освобождать ресурсы при завершении беседы по сигналу от потребителя навыков.

   [!code-csharp[on end of conversation](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/Controllers/MessagesController.cs?range=69-87)]

1. Изменение кода бота.

   **V3PizzaBot\\PizzaOrderDialog.cs**

   Обновите инструкции using.

   [!code-csharp[using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/PizzaOrderDialog.cs?range=4-14)]

   Добавьте приветственное сообщение. Это поможет пользователю понять, что происходит, когда корневой бот вызывает бот для заказа пиццы в качестве навыка.

   [!code-csharp[start form](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/PizzaOrderDialog.cs?range=29-35)]

   Измените код бота так, чтобы навык мог отмечать завершение диалога, когда пользователь отменяет или завершает заказ. Также навык должен очищать состояние и освобождать ресурсы, когда он завершает беседу.

   [!code-csharp[form complete](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/PizzaOrderDialog.cs?range=76-104)]

1. Примените этот манифест для бота заказа пиццы. В качестве идентификатора приложения конечной точки укажите идентификатор приложения бота.

   **V3PizzaBot\\wwwroot\\pizza-bot-manifest.json**

   [!code-json[manifest](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3PizzaBot/wwwroot/pizza-bot-manifest.json?highlight=22)]

   См. сведения о [схеме манифеста навыка (skill-manifest-2.0.0.json)](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json).

## <a name="convert-the-sandwich-bot"></a>Преобразование бота для заказа сандвичей

1. Откройте копию проекта SimpleSandwichBot и настройте в нем использование порта 3981.

   1. Откройте вкладку свойств этого проекта.
   1. Выберите категорию **Веб** и задайте для параметра **URL-адрес проекта** значение `http://localhost:3981/`.
   1. Сохраните изменения и закройте вкладку свойств.

1. В файл конфигурации добавьте идентификатор приложения и пароль бота для заказа пиццы. Там же, в параметрах приложения, добавьте свойство `AllowedCallers` и присвойте ему значение идентификатора приложения простого корневого бота.

   **V3SimpleSandwichBot\\Web.config**

   [!code-xml[app settings](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Web.config?range=11-16)]

1. Добавьте класс `ConversationHelper` со вспомогательными методами, реализовав следующее:
   - Отправка действия `endOfConversation` по завершении навыка. Позволяет вернуть сведения о заказе в свойстве `Value` действия и задать свойство `Code`, чтобы обозначить причину завершения беседы.
   - Очистка состояния беседы и освобождение всех связанных ресурсов.

   **V3SimpleSandwichBot\\ConversationHelper.cs**

   [!code-csharp[conversation helper](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/ConversationHelper.cs?range=4-79)]

1. Обновите класс `MessagesController`.

   **V3SimpleSandwichBot\\Controllers\\MessagesController.cs**

   Обновите инструкции using.

   [!code-csharp[using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Controllers/MessagesController.cs?range=4-17)]

   Замените атрибут класса `BotAuthentication` на `SkillBotAuthentication`. Этот бот использует стандартное средство проверки утверждений.

   [!code-csharp[attribute](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Controllers/MessagesController.cs?range=21-22)]

   В методе `Post` измените условие действия `message`, чтобы разрешить пользователю отменить процесс заказа из навыка. Также добавьте условие действия `endOfConversation`, которое позволит навыкам очищать состояние и освобождать ресурсы при завершении беседы по сигналу от потребителя навыков.

   [!code-csharp[on end of conversation](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Controllers/MessagesController.cs?range=42-60)]

1. Измените форму заказа сандвичей.

   **V3SimpleSandwichBot\\Sandwich.cs**

   Обновите инструкции using.

   [!code-csharp[using statements](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Sandwich.cs?range=4-8)]

   Измените в форме метод `BuildForm`, чтобы этот навык мог отмечать завершение беседы.

   [!code-csharp[form complete](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/Sandwich.cs?range=47-54&highlight=6)]

1. Примените этот манифест для бота заказа сандвичей. В качестве идентификатора приложения конечной точки укажите идентификатор приложения бота.

   **V3SimpleSandwichBot\\wwwroot\\sandwich-bot-manifest.json**

   [!code-json[manifest](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V3SimpleSandwichBot/wwwroot/sandwich-bot-manifest.json?highlight=22)]

   См. сведения о [схеме манифеста навыка (skill-manifest-2.0.0.json)](https://github.com/microsoft/botframework-sdk/blob/master/schemas/skills/skill-manifest-2.0.0.json).

## <a name="create-the-v4-root-bot"></a>Создание корневого бота версии 4

Этот постой корневой бот выполняет роль потребителя для трех навыков и позволяет убедиться, что шаги беседы проходят в соответствии с планом. Настройте локальное выполнение бота на порту 3978.

1. В файл конфигурации добавьте идентификатор приложения и пароль корневого бота. Добавьте идентификатор приложения для каждого навыка версии 3.

   **V4SimpleRootBot\\appsettings.json**

   [!code-json[configuration](~/../botbuilder-samples/MigrationV3V4/CSharp/Skills/V4SimpleRootBot/appsettings.json?highlight=2-3,8,13,18)]

## <a name="test-the-root-bot"></a>Тестирование корневого бота

Скачайте и установите последнюю версию [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

1. Скомпилируйте и запустите все четыре бота на локальном компьютере.
1. Запустите эмулятор и подключите его к корневому боту.
1. Протестируйте работу потребителя и навыков.

## <a name="additional-information"></a>Дополнительные сведения

### <a name="bot-to-bot-authentication"></a>Аутентификация взаимодействия между ботами

Корневой бот и навык обмениваются данными по протоколу HTTP. Эта платформа использует токены носителя и идентификаторы приложения бота для идентификации каждого бота. Она использует объект конфигурации проверки подлинности для проверки заголовков проверки подлинности во входящих запросах. В конфигурацию проверки подлинности можно добавить средство проверки утверждений. Утверждения оцениваются после заголовка проверки подлинности. Код средства проверки должен создавать исключение или ошибку, чтобы отклонить запрос.

Стандартное средство проверки утверждений считывает параметр приложения `AllowedCallers` из файла конфигурации бота. Этот параметр должен содержать разделенный запятыми список идентификаторов приложений ботов, которым разрешено вызывать этот навык, или значение "*", чтобы разрешить всем ботам вызывать навык.

Чтобы реализовать пользовательское средство проверки утверждений, реализуйте классы, производные от `AuthenticationConfiguration` и `ClaimsValidator`, а затем укажите в атрибуте `SkillBotAuthentication` ссылку на полученную конфигурацию проверки подлинности. Пример классов для средства проверки утверждений вы можете найти на шагах 3 и 4 раздела, посвященного [преобразованию эхо-бота](#convert-the-echo-bot).

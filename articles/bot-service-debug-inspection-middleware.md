---
title: Отладка бота с помощью проверяющего ПО промежуточного слоя | Документация Майкрософт
description: Из этой статьи вы узнаете, как отлаживать бота с помощью проверяющего ПО промежуточного слоя
author: zxyanliu
ms.author: v-liyan
keywords: пакет SDK для Bot Framework, отладка бота, проверяющее промежуточное ПО, эмулятор бота, Регистрация каналов Azure Bot
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
ms.openlocfilehash: 0e59c6d3548e273a8fb164526ddeb6ba66f48e3e
ms.sourcegitcommit: 312a4593177840433dfee405335100ce59aac347
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/12/2019
ms.locfileid: "73933517"
---
# <a name="debug-a-bot-with-inspection-middleware"></a>Отладка бота с помощью проверяющего ПО промежуточного слоя
В этой статье описывается, как отлаживать бота с помощью проверяющего ПО промежуточного слоя. Эта функция позволяет Bot Framework Emulator отлаживать входящий и исходящий трафик для бота, а также просматривать текущее состояние бота. Вы можете использовать сообщение трассировки для отправки данных в эмулятор и проверки состояния бота на любом шаге беседы. 

Чтобы продемонстрировать, как выполнять отладку и проверку состояния сообщений бота, мы используем бот EchoBot, созданный локально с помощью Bot Framework версии 4 ([C#](https://docs.microsoft.com/azure/bot-service/dotnet/bot-builder-dotnet-sdk-quickstart?view=azure-bot-service-4.0) | [JavaScript](https://docs.microsoft.com/azure/bot-service/javascript/bot-builder-javascript-quickstart?view=azure-bot-service-4.0)). Отладку бота также можно выполнять с помощью [IDE](./bot-service-debug-bot.md) или [Bot Framework Emulator](./bot-service-debug-emulator.md), но для отладки состояния в бота нужно добавить проверяющее ПО промежуточного слоя. Соответствующие примеры для бота доступны здесь: [C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection) и [JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection). 

## <a name="prerequisites"></a>Предварительные требования
- Установленное приложение [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).
- Понимание принципов работы [промежуточного слоя](https://docs.microsoft.com/azure/bot-service/bot-builder-concept-middleware?view=azure-bot-service-4.0) бота.
- Понимание принципов [управления состоянием](https://docs.microsoft.com/azure/bot-service/bot-builder-concept-state?view=azure-bot-service-4.0) бота.
- Установленное средство [ngrok](https://ngrok.com/) (если вы хотите отладить бота, настроенного в Azure для использования дополнительных каналов).

## <a name="update-your-emulator-to-the-latest-version"></a>Обновление эмулятора до последней версии 
Прежде чем использовать проверяющее ПО промежуточного слоя для отладки бота, нужно обновить эмулятор до версии не ниже 4.5. Проверьте [последнюю версию](https://github.com/Microsoft/BotFramework-Emulator/releases) на наличие обновлений. 

Чтобы проверить версию эмулятора, выберите в меню пункт **Справка** -> **О программе**. Вы увидите текущую версию эмулятора. 

![Текущая версия](./media/bot-debug-inspection-middleware/bot-debug-check-emulator-version.png) 

## <a name="update-your-bots-code"></a>Обновление кода бота

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
Настройте состояние проверки в **загрузочном файле**. Добавьте проверяющее ПО промежуточного слоя в адаптер. Состояние проверки предоставляется путем внедрения зависимостей. См. обновление кода ниже или пример проверки: [C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection). 

**Startup.cs.**  
[!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/Startup.cs?range=17-37)]

**AdapterWithInspection.cs**  
[!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/AdapterwithInspection.cs?range=11-37)]

**EchoBot.cs**  
[!code-csharp [inspection bot sample](~/../botbuilder-samples/samples/csharp_dotnetcore/47.inspection/Bots/EchoBot.cs?range=14-43)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
Прежде чем обновлять код бота, нужно обновить его пакеты до последних версий, выполнив в терминале следующую команду: 
```cmd
npm install --save botbuilder@latest 
```  
Затем нужно обновить код бота JavaScript следующим образом. См. также раздел об [обновлении кода бота](https://github.com/Microsoft/BotFramework-Emulator/blob/master/content/CHANNELS.md#1-update-your-bots-code) или пример проверки здесь: [JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection). 

**index.js**

Настройте состояние проверки и добавьте проверяющее ПО промежуточного слоя в адаптер в файле **index.js**. 

[!code-javascript [inspection bot sample](~/../botbuilder-samples/samples/javascript_nodejs/47.inspection/index.js?range=10-43)]

**bot.js**

Обновите класс бота в файле **bot.js**. 

[!code-javascript [inspection bot sample](~/../botbuilder-samples/samples/javascript_nodejs/47.inspection/bot.js?range=6-50)]

---

## <a name="test-your-bot-locally"></a>Локальное тестирование бота 
Обновив код, вы можете запустить бота локально и протестировать функцию отладки с помощью двух эмуляторов: для отправки и получения сообщений, а также для проверки их состояния в режиме отладки. Чтобы протестировать бота локально, сделайте следующее: 

1. Перейдите к каталогу бота в терминале и выполните следующую команду, чтобы запустить бота локально: 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

```cmd
dotnet run
```

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

```cmd
npm start 
```

---

2. Откройте эмулятор. Щелкните **Открыть бота**. Вставьте в поле "URL-адрес бота" http://localhost:3978/api/messages , а также значения **MicrosoftAppId** и **MicrosoftAppPassword**. Если вы используете бота на JavaScript, эти значения можно найти в файле **.env**. Если вы используете бота на C#, эти значения можно найти в файле **appsettings.json**. Щелкните **Подключить**. 

3. Теперь откройте другой эмулятор. Второй эмулятор будет работать в качестве отладчика. Следуйте инструкциям, описанным выше. Установите флажок **Открыть в режиме отладки** и щелкните **Подключить**. 

4. На этом этапе вы увидите UUID (`/INSPECT attach <identifier>`) в эмуляторе для отладки. Скопируйте и вставьте UUID в окно чата первого эмулятора. 

> [!NOTE]
> Универсальный уникальный идентификатор (UUID) используется для определения информации. UUID создается при каждом запуске эмулятора в режиме отладки после добавления проверяющего ПО промежуточного слоя в код бота. 

5. Теперь вы можете отправить сообщения в окно чата первого эмулятора и проверить их в эмуляторе для отладки. Для этого щелкните **Состояние бота** в эмуляторе отладки и разверните блок **values** в окне **JSON** справа. Состояние бота будет выглядеть так: ![Состояние бота](./media/bot-debug-inspection-middleware/bot-debug-bot-state.png)

## <a name="inspect-the-state-of-a-bot-configured-in-azure"></a>Проверка состояния бота, настроенного в Azure 
Чтобы проверить состояние бота, настроенного в Azure и подключенного к каналам (например, Teams), необходимо установить и запустить[ngrok](https://ngrok.com/).

### <a name="run-ngrok"></a>Запуск ngrok
На этом этапе вы обновили эмулятор до последней версии и добавили проверяющее ПО промежуточного слоя в код бота. Теперь вы можете запустить ngrok и подготовить локального бота к работе с Регистрацией каналов Azure Bot. Перед запуском ngrok нужно запустить бота локально. 

Чтобы запустить бота локально, сделайте следующее: 
1. Перейдите в папку бота в терминале и настройте для регистрации npm использование [последних сборок](https://botbuilder.myget.org/feed/botbuilder-v4-js-daily/package/npm/botbuilder-azure).

2. Запустите бот на локальном компьютере. Вы увидите, что бот предоставляет номер порта, например 3978. 

3. Откройте другое окно командной строки и перейдите в папку проекта бота. Выполните следующую команду:
```
ngrok http 3978
```
4. Это подключит ngrok к запущенному локально боту. Скопируйте общедоступный IP-адрес. 
![Успешное подключение ngrok](./media/bot-debug-inspection-middleware/bot-debug-ngrok.png)

### <a name="update-channel-registrations-for-your-bot"></a>Обновление Регистраций каналов для бота
Локальный бот подключен к ngrok, и вы можете подготовить его к работе с Регистрацией каналов бота в Azure.

1. Перейдите к Регистрации каналов бота в Azure. В меню слева щелкните **Параметры** и задайте для **конечной точки обмена сообщениями** значение с IP-адресом ngrok. При необходимости добавьте **/api/messages** после IP-адреса. Например, https://e58549b6.ngrok.io/api/messages). Установите флажок **Включить конечную точку потоковой передачи** и щелкните **Сохранить**.
![endpoint](./media/bot-debug-inspection-middleware/bot-debug-channels-setting-ngrok.png)
> [!TIP]
> Если кнопка **Сохранить** неактивна, снимите флажок **Включить конечную точку потоковой передачи** и снова щелкните **Сохранить**. Затем установите флажок **Включить конечную точку потоковой передачи** и снова щелкните **Сохранить**. Убедитесь, что флажок **Включить конечную точку потоковой передачи** установлен, а конфигурация конечной точки сохранена. 

2. Перейдите в группу ресурсов бота, щелкните **Развертывание** и выберите Регистрацию каналов бота (предыдущее успешное развертывание). Щелкните **Входные данные** слева, чтобы получить значения **appId** и **appSecret**. Добавьте в файл **.env** бота (или файл **appsettings.json**в случае использования бота на C#) значения **appId** и **appSecret**. 
![Получение входных данных](./media/bot-debug-inspection-middleware/bot-debug-get-inputs-id-secret.png)

3. Запустите эмулятор, щелкните **Открыть бота** и введите значение http://localhost:3978/api/messages в поле **URL-адрес бота**. Вставьте в поля **Идентификатор приложения Майкрософт** и **Пароль приложения Майкрософт** те же значения **appId** и **appSecret**, которые вы добавили в файл **.env** (**appsettings.json**) бота. Щелкните **Подключить**. 

4. Теперь запущенный бот подключен к Регистрации каналов бота в Azure. Чтобы протестировать веб-чат, щелкните **Тестировать в веб-чате** и отправьте сообщения в поле чата. 
![Тестирование веб-чата](./media/bot-debug-inspection-middleware/bot-debug-test-webchat.png)

5. Теперь давайте активируем режим отладки в эмуляторе. В эмуляторе выберите **Отладка** -> **Начать отладку**. Введите IP-адрес ngrok (не забудьте добавить **/api/messages**) в поле **URL-адрес бота** (например, https://e58549b6.ngrok.io/api/messages) ). Вставьте в поля **Идентификатор приложения Майкрософт** и **Пароль приложения Майкрософт** значения **appId** и **appSecret**. Также убедитесь, что флажок **Открыть в режиме отладки** установлен. Щелкните **Подключить**. 

6. UUID создается в эмуляторе при включении режима отладки. UUID — это уникальный идентификатор, создаваемый при каждом запуске режима отладки в эмуляторе. Скопируйте и вставьте UUID в поле чата **Тестировать в веб-чате** или в поле чата канала. В поле чата вы увидите сообщение Attached to session, all traffic is being replicated for inspection (Подключение к сеансу; весь трафик реплицируется для проверки). 

 Чтобы начать отладку бота, отправьте сообщения в настроенное поле чата канала. Локальный эмулятор будет автоматически включать в сообщения все сведения для отладки. Чтобы проверить состояние сообщений бота, щелкните **Состояние бота** и разверните блок **values** в окне JSON справа. 

 ![Отладка с помощью проверочного ПО промежуточного слоя](./media/bot-debug-inspection-middleware/debug-state-inspection-channel-chat.gif)

## <a name="additional-resources"></a>Дополнительные ресурсы
- Соответствующие примеры для бота доступны здесь: [C#](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/csharp_dotnetcore/47.inspection) и [JavaScript](https://github.com/microsoft/BotBuilder-Samples/tree/master/samples/javascript_nodejs/47.inspection). 
- См. также статьи об [устранении общих проблем](bot-service-troubleshoot-bot-configuration.md) и других неполадок в этом разделе.
- См. также статью об [отладке ботов с помощью эмулятора](bot-service-debug-emulator.md).

---
title: Устранение неполадок проверки подлинности Bot Framework | Документация Майкрософт
description: Сведения об устранении неполадок проверки подлинности с помощью бота.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/17
ms.openlocfilehash: a64edda73832f4d3fff49b08b5eaf6792c021ece
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305830"
---
# <a name="troubleshooting-bot-framework-authentication"></a>Устранение неполадок проверки подлинности Bot Framework

Это руководство поможет при устранении неполадок проверки подлинности с помощью бота, оценивая ряд сценариев, чтобы определить, где существует проблема. 

> [!NOTE]
> Чтобы выполнить все действия, описанные в этом руководстве, необходимо загрузить и использовать [Bot Framework Emulator][Emulator] и иметь доступ к параметрам регистрации бота на <a href="https://dev.botframework.com" target="_blank">портале Bot Framework</a>.

## <a id="PW"></a> Идентификатор приложения и пароль

Безопасность бота настраивается с помощью **идентификатора приложения Microsoft** и  **пароля приложения Microsoft**, которые вы получаете при регистрации бота с помощью Bot Framework. Эти значения обычно указаны в файле конфигурации бота и используются для получения маркеров доступа из службы учетной записи Microsoft. 

Если это не сделано, [зарегистрируйте бот](~/bot-service-quickstart-registration.md) для получения **идентификатора приложения Microsoft** и **пароля приложения Microsoft**, его можно использовать для проверки подлинности. 

> [!NOTE]
> Сведения о том, как найти значения **AppID** и **AppPassword** бота, см. в разделе [MicrosoftAppID и MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

## <a name="step-1-disable-security-and-test-on-localhost"></a>Шаг 1. Отключение системы безопасности и тестирование на локальном компьютере

На этом шаге бот доступен и функционирует на локальном компьютере, когда отключена система безопасности. 

> [!WARNING]
> Отключение системы защиты бота может позволить неизвестным злоумышленникам выдавать себя за пользователей. Если вы работаете в защищенной среде отладки, используйте только следующую процедуру.

### <a id="disable-security-localhost"></a> Отключение системы безопасности

Чтобы отключить систему защиты бота, измените его параметры конфигурации и удалите значения идентификатора и пароля приложения. 

Если используется пакет SDK Bot Builder для .NET, измените эти параметры в файле Web.config.

```xml
<appSettings>
  <add key="MicrosoftAppId" value="" />
  <add key="MicrosoftAppPassword" value="" />
</appSettings>
```

Если используется пакет SDK Bot Builder для Node.js, измените эти значения (или обновите соответствующие переменные среды).

```javascript
var connector = new builder.ChatConnector({
  appId: null,
  appPassword: null
});
```

### <a name="test-your-bot-on-localhost"></a>Тестирование бота на локальном компьютере 

Протестируйте бот на локальном компьютере с помощью Bot Framework Emulator.

1. Запустите бот на локальном компьютере.
2. Установите Bot Framework Emulator.
3. Подключите эмулятор к боту.
    - Введите `http://localhost:port-number/api/messages` в адресную строку эмулятора, где **номер порта** соответствует номеру порта в браузере, в котором запущено приложение. 
    - Убедитесь, что поля **идентификатора приложения Microsoft** и **пароля приложения Microsoft** пусты.
    - Щелкните **Подключить**.
4. Чтобы проверить возможность подключения к боту, введите текст в эмуляторе и нажмите клавишу ВВОД.

Если бот реагирует на входные данные и в окне чата нет ошибок, значит бот доступен и функционирует на локальном узле при выключенной системе безопасности. Перейдите к [Шагу 2](#step-2).

Если в окне чата указаны одна или несколько ошибок, щелкните по ошибке, чтобы узнать подробности. Часто возникающие проблемы.

* Настройки эмулятора указывают неверную конечную точку для бота. Убедитесь, что указан правильный номер порта в URL-адресе и правильный путь в конце URL-адреса (например, `/api/messages`).
* В настройках эмулятора указывается конечная точка бота, начинающаяся с `https`. На локальном узле конечная точка должна начинаться с `http`.
* В настройках эмулятора указано значение для полей **идентификатора приложения Microsoft** и **пароля приложения Microsoft**. Оба поля должны быть пустыми.
* Система безопасности бота не была отключена. [Проверьте](#disable-security-localhost), чтобы не было значений идентификатора или пароля приложения в настройках бота.

## <a id="step-2"></a> Шаг 2: Проверка идентификатора и пароля приложения бота

На этом шаге проверяется допустимость идентификатора и пароля приложения бота, которые используются для проверки подлинности. (Если вы не знаете этих значений, [получите их](#PW) сейчас.) 

> [!WARNING]
> Приведенные ниже инструкции отключают проверку SSL для `login.microsoftonline.com`. Выполняйте эту процедуру только в защищенной сети и после этого измените пароль приложения.

### <a name="issue-an-http-request-to-the-microsoft-login-service"></a>Выдача HTTP-запроса к службе входа в систему Microsoft

Эти инструкции описывают, как использовать [cURL](https://curl.haxx.se/download.html) для HTTP-запроса к службе входа в систему Microsoft. Можно использовать альтернативный инструмент, такой как Postman, просто убедитесь, что запрос соответствует [протоколу проверки подлинности](~/rest-api/bot-framework-rest-connector-authentication.md) Bot Framework.

Чтобы убедиться, что идентификатор и пароль приложения бота действительны, выполните следующий запрос, используя **cURL**, заменив `APP_ID` и `APP_PASSWORD` идентификатором и паролем бота.

```cmd
curl -k -X POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token -d "grant_type=client_credentials&client_id=APP_ID&client_secret=APP_PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default"
```

Этот запрос пытается обменять идентификатор и пароль бота на маркер доступа. Если запрос будет успешным, вы получите полезные данные JSON, которые среди прочих содержат свойство `access_token`. 

```json
{"token_type":"Bearer","expires_in":3599,"ext_expires_in":0,"access_token":"eyJ0eXAJKV1Q..."}
```

Если запрос выполняется успешно, нужно подтвердить, что идентификатор приложения и пароль, указанный в запросе, являются допустимыми. Перейдите к [Шагу 3](#step-3).

Если пришло сообщение об ошибке в ответ на запрос, проверьте его, чтобы определить причину. Если ответ указывает, что идентификатор приложения или пароль недопустимы, [получите правильные значения](#PW) с портала Bot Framework и повторно отправьте запрос с новыми значениями, чтобы подтвердить, что они действительны. 

## Шаг 3. Включение системы безопасности и тестирования на локальном компьютере <a id="step-3"></a>

На этом шаге проверяется доступность и функциональность бота на локальном компьютере, когда система безопасности отключена, и подтверждается, что идентификатор приложения и пароль, которые бот будет использовать для аутентификации, действительны. На этом шаге убедитесь, что бот доступен и работает на локальном компьютере, когда включена система защиты.

### <a id="enable-security-localhost"></a> Включение системы безопасности

Безопасность бота зависит от служб Microsoft, даже если бот работает только на локальном компьютере. Чтобы включить систему защиты для бота, отредактируйте его параметры конфигурации, чтобы заполнить идентификатор и пароль приложения с помощью значений, которые были подтверждены на [Шаге 2](#step-2).

Если используется пакет SDK Bot Builder для .NET, заполните эти параметры в файле Web.config.

```xml
<appSettings>
  <add key="MicrosoftAppId" value="APP_ID" />
  <add key="MicrosoftAppPassword" value="PASSWORD" />
</appSettings>
```

Если используется пакет SDK Bot Builder для Node.js, заполните эти параметры (или обновите соответствующие переменные среды).

```javascript
var connector = new builder.ChatConnector({
  appId: 'APP_ID',
  appPassword: 'PASSWORD'
});
```

> [!NOTE]
> Сведения о том, как найти значения **AppID** и **AppPassword** бота, см. в разделе [MicrosoftAppID и MicrosoftAppPassword](bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

### <a name="test-your-bot-on-localhost"></a>Тестирование бота на локальном компьютере 

Протестируйте бот на локальном компьютере с помощью Bot Framework Emulator.

1. Запустите бот на локальном компьютере.
2. Установите Bot Framework Emulator.
3. Подключите эмулятор к боту.
    - Введите `http://localhost:port-number/api/messages` в адресную строку эмулятора, где **номер порта** соответствует номеру порта в браузере, в котором запущено приложение. 
    - Введите идентификатор приложения бота в поле **Идентификатор приложения Microsoft**.
    - Введите пароль приложения бота в поле **Пароль приложения Microsoft**.
    - Щелкните **Подключить**.
4. Чтобы проверить возможность подключения к боту, введите текст в эмуляторе и нажмите клавишу ВВОД.

Если бот реагирует на входные данные и в окне чата нет ошибок, значит бот доступен и работает на локальном компьютере, когда включена система безопасности.  Перейдите к [Шагу 4](#step-4).

Если в окне чата указаны одна или несколько ошибок, щелкните по ошибке, чтобы узнать подробности. Часто возникающие проблемы.

* Настройки эмулятора указывают неверную конечную точку для бота. Убедитесь, что указан правильный номер порта в URL-адресе и правильный путь в конце URL-адреса (например, `/api/messages`).
* В настройках эмулятора указывается конечная точка бота, начинающаяся с `https`. На локальном узле конечная точка должна начинаться с `http`.
* В настройках эмулятора поля **идентификатор приложения Microsoft** и **пароль приложения Microsoft** не содержат допустимые значения. Оба поля должны быть заполнены и содержать соответствующие значения, которые были проверены на [Шаге 2](#step-2).
* Для бота система безопасности не включена. [Убедитесь](#enable-security-localhost), что настройки конфигурации бота задают значения как для идентификатора приложения, так и для пароля.

## Шаг 4. Тестирование бота в облаке <a id="step-4"></a>

На этом шаге вы подтвердили, что бот доступен и функционирует на локальном узле, когда система безопасности отключена. Функционирование бота на локальном узле подтверждает, что идентификатор и пароль приложения бота являются допустимыми, когда включена система безопасности. На этом шаге разверните бот в облаке и убедитесь, что он доступен и работает там, где включена защита. 

### <a name="deploy-your-bot-to-the-cloud"></a>Развертывание бота в облаке

Bot Framework требует, чтобы боты были доступны из Интернета, поэтому нужно развернуть бот на платформе облачного размещения, такой ​​как Azure. Не забудьте включить систему безопасности для бота до развертывания, как описано на [Шаге 3](#step-3).

> [!NOTE]
> Если у вас еще нет облачного хранилища, можно зарегистрироваться для получения <a href="https://azure.microsoft.com/en-us/free/" target="_blank">бесплатной учетной записи</a>. 

Если разворачивать бот в Azure, протокол SSL будет автоматически настроен для приложения, тем самым позволяя Bot Framework требовать конечную точку **HTTPS**. Если используется другой провайдер облачного размещения, убедитесь, что приложение настроено для протокола SSL, чтобы у бота была конечная точка **HTTPS**.

### <a name="test-your-bot"></a>Тестирование бота 

Чтобы протестировать бот в облаке с включенной системой безопасности, выполните следующие действия.

1. Убедитесь, что бот успешно развернут и запущен. 
2. Войдите на <a href="https://dev.botframework.com" target="_blank">портал Bot Framework</a>.
3. Щелкните **Мои боты**.
4. Выберите бот, который требуется проверить.
5. Нажмите **Тест**, чтобы открыть бот во встроенном элементе управления "Веб-чат".
6. Чтобы проверить подключение к боту, введите текст в элемент управления "Веб-чат" и нажмите ВВОД.

Если в окне "Веб-чат" появится ошибка, используйте сообщение об ошибке, чтобы определить ее причину. Часто возникающие проблемы. 

* Некорректная **Конечная точка обмена сообщениями**, указанная на странице **Параметры** для бота на портале Bot Framework. Убедитесь, что указан правильный путь в конце URL-адреса (например, `/api/messages`).
* **Конечная точка обмена сообщениями**, указанная на странице **Параметры** для бота на портале Bot Framework, не начинается с `https` или не поддерживается платформой Bot Framework. Бот должен иметь допустимую цепочку сертификатов системы безопасности.
* Бот имеет отсутствующие или неверные значения для идентификатора или пароля приложения. [Проверьте](#enable-security-localhost), чтобы настройки конфигурации бота определяли допустимые значения идентификатора и пароля приложения.

Если бот соответствующим образом реагирует на входные данные, нужно подтвердить, что бот доступен и успешно работает в облаке с включенной системой безопасности. На этом шаге бот готов безопасно [подключиться к каналу](~/bot-service-manage-channels.md), такому как Skype, Facebook Messenger, Direct Line и другим.

## <a name="additional-resources"></a>Дополнительные ресурсы

Если проблема не исчезла после выполнения действий, описанных выше, можно сделать следующее.

* [Провести отладку бота в облаке](~/bot-service-debug-emulator.md) с помощью Bot Framework Emulator и <a href="https://ngrok.com/" target="_blank">ngrok</a>.
* Используйте инструменты прокси, например [Fiddler](https://www.telerik.com/fiddler), для проверки трафика HTTPS через бот. *Fiddler не является продуктом корпорации Майкрософт.*
* Чтобы узнать о технологиях аутентификации, которые использует Bot Framework, см. раздел [Authentication][BotConnectorAuthGuide] (Проверка подлинности).
* Запросить помощь от других пользователей, используя [поддержку][Support] ресурсов Bot Framework. 

[BotConnectorAuthGuide]: ~/rest-api/bot-framework-rest-connector-authentication.md
[Support]: bot-service-resources-links-help.md
[Emulator]: bot-service-debug-emulator.md
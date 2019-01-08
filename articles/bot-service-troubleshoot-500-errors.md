---
title: Устранение в ботах неполадок, приводящих к ошибкам HTTP 500 | Документация Майкрософт
description: Устранение неполадок при возникновении ошибок HTTP 500 в развернутом боте.
keywords: troubleshoot, HTTP 500, problems.
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/20/2018
ms.openlocfilehash: 8ab1cd34f2cc239602db423bccd131d9df39222a
ms.sourcegitcommit: f7a8f05fc05ff4a7212a437d540485bf68831604
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/21/2018
ms.locfileid: "53736026"
---
# <a name="troubleshoot-http-500-errors"></a>Устранение неполадок при ошибках HTTP 500

Если вы столкнулись с ошибками HTTP 500, первым шагом для устранения неполадок будет включение Application Insights.

В примерах luis-with-appinsights ([C#](https://aka.ms/cs-luis-with-appinsights-sample) / [JS](https://aka.ms/js-luis-with-appinsights-sample)) и qna-with-appinsights ([C#](https://aka.ms/qna-with-appinsights) / [JS](https://aka.ms/js-qna-with-appinsights-sample)) представлены боты, которые поддерживают Azure Application Insights. См. сведения о добавлении Application Insights к существующему боту в статье [о телеметрии аналитики бесед](https://aka.ms/botPowerBiTemplate).

## <a name="enable-application-insights-on-aspnet"></a>Включение Application Insights для ASP.NET

Базовая поддержка Application Insights описана в статье [Настройка Application Insights для веб-сайта ASP.NET](https://docs.microsoft.com/azure/application-insights/app-insights-asp-net). Платформа Bot Framework (начиная с версии 4.2) предоставляет дополнительный уровень телеметрии для Application Insights, но без него можно обойтись при диагностике ошибок HTTP 500.

## <a name="enable-application-insights-on-nodejs"></a>Включение Application Insights для Node.js

Базовая поддержка Application Insights описана в статье [о мониторинге служб и приложений Node.js с помощью Application Insights](https://docs.microsoft.com/azure/application-insights/app-insights-nodejs). Платформа Bot Framework (начиная с версии 4.2) предоставляет дополнительный уровень телеметрии для Application Insights, но без него можно обойтись при диагностике ошибок HTTP 500.

## <a name="query-for-exceptions"></a>Запрос исключений

Анализ ошибок с кодом состояния HTTP 500 проще всего начать с исключений.

Следующие запросы возвращают последние исключения:

```sql
exceptions
| order by timestamp desc
| project timestamp, operation_Id, appName
```

Выберите несколько идентификаторов операций из результатов первого запроса и получите по ним дополнительные сведения:

```sql
let my_operation_id = "d298f1385197fd438b520e617d58f4fb";
let union_all = () {
    union
    (traces | where operation_Id == my_operation_id),
    (customEvents | where operation_Id == my_operation_id),
    (requests | where operation_Id == my_operation_id),
    (dependencies | where operation_Id  == my_operation_id),
    (exceptions | where operation_Id == my_operation_id)
};

union_all
    | order by timestamp desc
```

Если в списке есть только `exceptions`, проанализируйте подробные сведения и сопоставьте эти данные со строками вашего кода. Если все исключения поступают только от соединителя канала (`Microsoft.Bot.ChannelConnector`), выполните [анализ отсутствия событий Application Insights](#no-application-insights-events) и убедитесь, что служба Application Insights настроена правильно и в коде правильно регистрируются события.

## <a name="no-application-insights-events"></a>Отсутствие событий Application Insights

Если возникают ошибки с кодом 500, но в Application Insights не поступает больше никаких событий от бота, проверьте следующее.

### <a name="ensure-bot-runs-locally"></a>Проверка локального выполнения бота

Прежде всего убедитесь, что бот успешно выполняется в локальном эмуляторе.

### <a name="ensure-configuration-files-are-being-copied-net-only"></a>Проверка копирования файлов конфигурации (только для .NET)

Убедитесь, что файлы конфигурации `.bot` и `appsettings.json` правильно пакуются в процессе развертывания.

#### <a name="application-assemblies"></a>Сборки приложения

Убедитесь, что сборки Application Insights правильно пакуются в процессе развертывания.

- Microsoft.ApplicationInsights
- Microsoft.ApplicationInsights.TraceListener
- Microsoft.AI.Web
- Microsoft.AI.WebServer
- Microsoft.AI.ServeTelemetryChannel
- Microsoft.AI.PerfCounterCollector
- Microsoft.AI.DependencyCollector
- Microsoft.AI.Agent.Intercept

Убедитесь, что файлы конфигурации `.bot` и `appsettings.json` правильно пакуются в процессе развертывания.

#### <a name="appsettingsjson"></a>appsettings.json

Убедитесь, что в файле `appsettings.json` задан ключ инструментирования.

## <a name="aspnet-web-apitabdotnetwebapi"></a>[ASP.NET Web API](#tab/dotnetwebapi)

```json
{
    "Logging": {
        "IncludeScopes": false,
        "LogLevel": {
            "Default": "Debug",
            "System": "Information",
            "Microsoft": "Information"
        },
        "Console": {
            "IncludeScopes": "true"
        }
    }
}
```

## <a name="aspnet-coretabdotnetcore"></a>[ASP.NET Core](#tab/dotnetcore)

```json
{
    "botFilePath": "mybot.bot",
    "botFileSecret": "<my secret>",
    "ApplicationInsights": {
        "InstrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
}
```

---

### <a name="verify-bot-config-file"></a>Проверка файла конфигурации .bot

Убедитесь, что в файл .bot включен ключ Application Insights.

```json
    {
        "type": "appInsights",
        "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "subscriptionId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "resourceGroup": "my resource group",
        "name": "my appinsights name",
        "serviceName": "my service name",
        "instrumentationKey": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "applicationId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
        "apiKeys": {},
        "id": ""
    },
```

### <a name="check-logs"></a>Проверка журналов

ASP.NET и Node создают записи журналов на уровне сервера, которые можно проверить.

#### <a name="set-up-a-browser-to-watch-your-logs"></a>Настройка просмотра журналов в браузере

1. Откройте бот на [портале Azure](http://portal.azure.com/).
1. Откройте страницу **Параметры службы приложений > All App service settings** (Все параметры службы приложений), чтобы проверить все параметры службы.
1. Откройте страницу **мониторинга и журналов диагностики** для службы приложений.
   - Убедитесь, что включен параметр **Вход в приложения (файловая система)**. Не забудьте щелкнуть **Сохранить**, если придется изменить этот параметр.
1. Перейдите на страницу **мониторинга и потока журналов**.
   - Выберите **Журналы веб-сервера** и убедитесь, что появляется сообщение об успешном подключении. Это выглядит примерно так:

     ```bash
     Connecting...
     2018-11-14T17:24:51  Welcome, you are now connected to log-streaming service.
     ```

     Оставьте это окно открытым.

#### <a name="set-up-browser-to-restart-your-bot-service"></a>Настройка перезапуска службы бота в браузере

1. Откройте другое окно браузера и перейдите в нем к странице бота на портале Azure.
1. Откройте страницу **Параметры службы приложений > All App service settings** (Все параметры службы приложений), чтобы проверить все параметры службы.
1. Перейдите на страницу **Обзор** для службы приложений и щелкните **Перезапуск**.
   - Если появится запрос подтверждения, выберите ответ **Да**.
1. Вернитесь в первое окно браузера и проверьте записи в журналах.
1. Убедитесь, что новые записи журналов успешно поступают.
   - Если здесь нет никаких сообщений о действиях, разверните бот повторно.
   - Теперь перейдите на страницу **Журналы приложений** и проверьте наличие ошибок.

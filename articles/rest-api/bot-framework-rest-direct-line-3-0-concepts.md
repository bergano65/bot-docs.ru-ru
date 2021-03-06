---
title: Основные понятия Direct Line API 3.0 в Bot Framework для службы Bot
description: Обзор основных понятий Direct Line API 3.0 в Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/06/2019
ms.openlocfilehash: 5d86cd325c296aceba6f52e81841028d6da79c4a
ms.sourcegitcommit: 308e6df385b9bac9c8d60f8b75eabc813b823c38
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/20/2020
ms.locfileid: "77520013"
---
# <a name="key-concepts-in-direct-line-api-30"></a>Основные понятия Direct Line API 3.0

C помощью API для Direct Line вы можете реализовать обмен данными между ботом и своим клиентским приложением. В этой статье представлены основные понятия Direct Line API 3.0 и сведения о соответствующих ресурсах для разработчиков. Вы можете создать клиент, используя пакет SDK, REST API или Web Chat. 

## <a name="authentication"></a>Аутентификация

Аутентификация запросов Direct Line API 3.0 может осуществляться с использованием **секрета**, который можно получить на странице конфигурации канала Direct Line на [портале Azure](https://portal.azure.com), или с помощью **токена**, который можно получить в среде выполнения. Дополнительные сведения см. в разделе [Authenticate to the Speech API](bot-framework-rest-direct-line-3-0-authentication.md) (Аутентификация в API речи).

## <a name="starting-a-conversation"></a>Начало общения

Общения Direct Line открываются клиентами явным образом и могут выполняться, пока бот и клиент участвуют в них и имеют действительные учетные данные. Дополнительные сведения см. в статье [Начало общения](bot-framework-rest-direct-line-3-0-start-conversation.md).

## <a name="sending-messages"></a>Отправка сообщений

С помощью Direct Line API 3.0 клиент может отправлять боту сообщения, выполняя запросы `HTTP POST`. В каждом запросе клиент может отправить одно сообщение. Дополнительные сведения см. в руководстве по [отправке действия боту](bot-framework-rest-direct-line-3-0-send-activity.md).

## <a name="receiving-messages"></a>Получение сообщений

Клиент может получать сообщения от бота с помощью Direct Line API 3.0 через поток `WebSocket` либо путем отправки запросов `HTTP GET`. Используя любой из этих методов, клиент может получать сразу несколько сообщений от бота как часть `ActivitySet`. Дополнительные сведения см. в статье [Receive activities from the bot](bot-framework-rest-direct-line-3-0-receive-activities.md) (Получение действий от бота).

## <a name="developer-resources"></a>Ресурсы для разработчиков

### <a name="client-libraries"></a>Клиентские библиотеки

Bot Framework предоставляет клиентские библиотеки, которые позволяют легко получить доступ к Direct Line API 3.0 с помощью C# и Node.js. 

- Чтобы использовать клиентскую библиотеку .NET в проекте Visual Studio, установите `Microsoft.Bot.Connector.DirectLine` <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine" target="_blank">пакет NuGet</a>. 

- Чтобы использовать клиентскую библиотеку в проекте Node.js, установите библиотеку `botframework-directlinejs` с помощью <a href="https://www.npmjs.com/package/botframework-directlinejs" target="_blank">NPM</a> (или <a href="https://github.com/Microsoft/BotFramework-DirectLineJS" target="_blank">загрузите</a> источник).

::: moniker range="azure-bot-service-3.0"

### <a name="sample-code"></a>Образец кода

Репозиторий GitHub <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples" target="_blank">BotBuilder-Samples</a> содержит несколько примеров, в которых показано, как с помощью C# и Node.js. использовать Direct Line API 3.0.

| Образец | Язык | Описание |
|----|----|----|
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLine" target="_blank">Пример бота Direct Line</a> | C# | Для взаимодействия примера бота и пользовательского клиента используется API для Direct Line. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/core-DirectLineWebSockets" target="_blank">Пример бота Direct Line (использующего клиент WebSockets)</a> | C# | Для взаимодействия примера бота и пользовательского клиента используется API для Direct Line и WebSockets. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLine" target="_blank">Пример бота Direct Line</a> | JavaScript | Для взаимодействия примера бота и пользовательского клиента используется API для Direct Line. |
| <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/core-DirectLineWebSockets" target="_blank">Пример бота Direct Line (использующего клиент WebSockets)</a> | JavaScript | Для взаимодействия примера бота и пользовательского клиента используется API для Direct Line и WebSockets. |

::: moniker-end

### <a name="web-chat-control"></a>Элемент управления веб-чата 

Bot Framework предоставляет элемент управления, который позволяет внедрить бот, использующий Direct Line, в клиентское приложение. Дополнительные сведения см. в статье о <a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Microsoft Bot Framework WebChat control</a> (Элемент управления WebChat в Microsoft Bot Framework).

---
title: Основные понятия Direct Line API 1.1 в Bot Framework — Служба Azure Bot
description: Обзор основных понятий API 1.1. для Direct Line в Bot Framework.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: d519bd6fa363cf47afa1609aa1930f4635e9e745
ms.sourcegitcommit: 308e6df385b9bac9c8d60f8b75eabc813b823c38
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/20/2020
ms.locfileid: "77520023"
---
# <a name="key-concepts-in-direct-line-api-11"></a>Основные понятия API 1.1 для Direct Line

C помощью API для Direct Line вы можете реализовать обмен данными между ботом и своим клиентским приложением. 

> [!IMPORTANT]
> В этой статье раскрыты основные понятия API 1.1 для Direct Line и приведены сведения о соответствующих ресурсах для разработчиков. При создании подключения между клиентским приложением и ботом используйте [API версии 3.0 для Direct Line](bot-framework-rest-direct-line-3-0-concepts.md).

## <a name="authentication"></a>Аутентификация

Аутентификация запросов Direct Line API 1.1 может осуществляться с использованием **секрета**, который можно получить на странице конфигурации канала Direct Line на [портале Azure](https://portal.azure.com), или с использованием **токена**, который можно получить в среде выполнения.  Дополнительные сведения см. в разделе [Authenticate to the Speech API](bot-framework-rest-direct-line-1-1-authentication.md) (Аутентификация в API речи).

## <a name="starting-a-conversation"></a>Начало общения

Общения Direct Line открываются клиентами явным образом и могут выполняться, пока бот и клиент участвуют в них и имеют действительные учетные данные. Дополнительные сведения см. в статье [Начало общения](bot-framework-rest-direct-line-1-1-start-conversation.md).

## <a name="sending-messages"></a>Отправка сообщений

С помощью API 1.1 для Direct Line клиент может отправлять боту сообщения, выполняя запросы `HTTP POST`. В каждом запросе клиент может отправить одно сообщение. Дополнительные сведения см. в разделе [Отправка сообщения боту](bot-framework-rest-direct-line-1-1-send-message.md).

## <a name="receiving-messages"></a>Получение сообщений

С помощью API 1.1 для Direct Line клиент может получать сообщения, выполняя опросы с запросами `HTTP GET`. В ответе на каждый запрос клиент может получить от бота несколько сообщений в виде части объекта `MessageSet`. Дополнительные сведения см. в статье [Receive messages from the bot](bot-framework-rest-direct-line-1-1-receive-messages.md) (Получение сообщений от бота).

## <a name="developer-resources"></a>Ресурсы для разработчиков

### <a name="client-library"></a>Клиентская библиотека

Bot Framework предоставляет клиентскую библиотеку, которая позволяет легко получить доступ к API для 1.1 Direct Line с помощью C#. Чтобы использовать клиентскую библиотеку в проекте Visual Studio, установите `Microsoft.Bot.Connector.DirectLine` <a href="https://www.nuget.org/packages/Microsoft.Bot.Connector.DirectLine/1.1.1" target="_blank">пакет NuGet версии 1.х</a>. 

Вместо клиентской библиотеки C# вы можете создать собственную клиентскую библиотеку на любом языке, используя <a href="https://docs.botframework.com/restapi/directline/swagger.json" target="_blank">файл Swagger для API 1.1 для Direct Line</a>.

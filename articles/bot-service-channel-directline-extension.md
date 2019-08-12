---
title: Расширение Службы приложений Direct Line
titleSuffix: Bot Service
description: Возможности расширения Службы приложений Direct Line
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/09/2019
ms.openlocfilehash: 27067cf2582de63cf67785cfaaa70b9a33685894
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757766"
---
## <a name="direct-line-app-service-extension"></a>Расширение Службы приложений Direct Line

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

Расширение Службы приложений Direct Line создает набор **постоянных именованных каналов** для подключения к боту через добавляемый пользователем **BotAdapter**.

Оно добавляет расширения потоковой передачи в протокол Bot Framework. Эти расширения заменяют HTTP на другой транспорт обмена сообщениями, который позволяет отправлять двунаправленные запросы **через постоянный канал WebSocket**. Это повышает производительность и уровень изоляции при обмене данными.

До применения расширений потоковой передачи интерфейс API Direct Line предоставлял клиенту один способ отправлять действия в Direct Line и два способа получать действия от Direct Line. Сообщения при этом отправлялись в HTTP-запросе POST и получались HTTP-запросом GET (метод опроса) либо через открытие WebSocket для получения ActivitySets.
Расширения потоковой передачи расширяют использование WebSocket, позволяя **отправлять все сообщения** через этот WebSocket. Расширения потоковой передачи можно также использовать между службами каналов и ботом.


Расширение Службы приложений Direct Line предварительно установлено на всех экземплярах Служб приложений Azure в каждом центре обработки данных по всему миру. Оно поддерживается и управляется корпорацией Майкрософт, и клиентам не нужно выполнять никаких действий для его развертывания.
По умолчанию оно отключен в Службах приложений Azure, но его можно легко включить и настроить для подключения к размещенному боту.

На следующем рисунке показана общая архитектура расширения:

![Архитектура расширения Direct Line](./media/channels/direct-line-extension-architecture.png)

## <a name="see-also"></a>См. также

|ИМЯ|ОПИСАНИЕ|
|---|---|
|[Использование бота .NET с расширением](bot-service-channel-directline-extension-net-bot.md)|Включите в бота поддержку работы с **именованными каналами** и включите расширение службы Direct Line в ресурсе **Службы приложений Azure**, в котором размещен этот бот.  |
|[Использование клиента .NET с расширением](bot-service-channel-directline-extension-net-client.md)|Создайте клиент .NET на языке C#, который подключается к расширению Службы приложений Direct Line.|
|[Использование WebChat с расширением](bot-service-channel-directline-extension-webchat-client.md)|Примените WebChat с расширением Службы приложений Direct Line.|
|[Использование расширения в виртуальной сети](bot-service-channel-directline-extension-vnet.md)|Примените расширение службы приложений Direct Line в виртуальной сети Azure (VNET).|

## <a name="addtional-resources"></a>Дополнительные ресурсы

- [Подключение бота к Direct Line](bot-service-channel-connect-directline.md)
- [Подключение бота к каналу Direct Line Speech](bot-service-channel-connect-directlinespeech.md)

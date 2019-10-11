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
ms.openlocfilehash: c4c54e50450ae81098992c880e23a049229fa09f
ms.sourcegitcommit: 7e901f5f39a0cfb0d37e532321b90a1dcf4baadd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/08/2019
ms.locfileid: "72039755"
---
# <a name="direct-line-app-service-extension"></a>Расширение Службы приложений Direct Line

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

> [!WARNING]
> Предоставляется общедоступная **предварительная версия** **расширения Службы приложений Direct Line**.  

Расширение Службы приложений Direct Line позволяет клиентам подключаться напрямую к узлу, на котором находится бот. Это обеспечивает изоляцию рабочих нагрузок и в некоторых случаях повышает производительность. На следующем рисунке показана общая архитектура расширения:

![Архитектура расширения Direct Line](./media/channels/direct-line-extension-architecture.png)

Расширение Службы приложений Direct Line обогащает протокол потоковой передачи Bot Framework. Предоставляемые расширения заменяют HTTP механизмом обмена сообщениями, который позволяет отправлять двунаправленные запросы **через постоянный канал WebSocket**.

До применения расширений потоковой передачи интерфейс API Direct Line предоставлял клиенту один способ отправлять действия в Direct Line и два способа получать действия от Direct Line. Сообщения при этом можно было отправлять в HTTP-запросе POST и получать в HTTP-запросе GET (метод опроса) или путем открытия WebSocket для получения ActivitySets.
Расширения потоковой передачи расширяют использование WebSocket, позволяя **отправлять все сообщения** через этот WebSocket. Расширения потоковой передачи можно также использовать между службами каналов и ботом.

Расширение Службы приложений Direct Line предварительно установлено во всех экземплярах Служб приложений Azure в каждом центре обработки данных по всему миру. Оно поддерживается и управляется корпорацией Майкрософт, и клиентам не нужно выполнять никаких действий для его развертывания. По умолчанию оно отключен в Службах приложений Azure, но его можно легко включить и настроить для подключения к размещенному боту.


## <a name="see-also"></a>См. также

|ИМЯ|ОПИСАНИЕ|
|---|---|
|[Использование бота .NET с расширением](bot-service-channel-directline-extension-net-bot.md)|Включите в бота поддержку работы с **именованными каналами** и включите расширение службы Direct Line в ресурсе **Службы приложений Azure**, в котором размещен этот бот.  |
|[Создание клиента .NET с использованием расширения](bot-service-channel-directline-extension-net-client.md)|Создание клиента .NET на C#, который подключается к расширению Службы приложений Direct Line|
|[Использование WebChat с расширением](bot-service-channel-directline-extension-webchat-client.md)|Применение WebChat с расширением Службы приложений Direct Line|
|[Использование расширения в виртуальной сети](bot-service-channel-directline-extension-vnet.md)|Использование расширения Службы приложений Direct Line в виртуальной сети Azure (VNET)|

## <a name="addtional-resources"></a>Дополнительные ресурсы

- [Подключение бота к Direct Line](bot-service-channel-connect-directline.md)
- [Подключение бота к каналу Direct Line Speech](bot-service-channel-connect-directlinespeech.md)

---
title: Дополнительные каналы | Документация Майкрософт
description: Узнайте, как настроить дополнительные каналы для вашего бота.
keywords: bot channels, hangouts, Twilio, facebook, azure portal
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 02/08/2019
ms.openlocfilehash: d6cf068f3cd4d57ff9cde2be6d41e084cee6524d
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693744"
---
# <a name="additional-channels"></a>Дополнительные каналы

Кроме каналов, перечисленных в этих документах, есть и дополнительные каналы, доступные как адаптеры. Они предоставляются через наши [платформы](https://botkit.ai/docs/v4/platforms/) (Botkit) или [репозитории сообщества](https://github.com/BotBuilderCommunity/). Предоставляются следующие дополнительные каналы:

- [Webex Teams](https://botkit.ai/docs/v4/platforms/webex.html);
- [WebSocket и Webhooks](https://botkit.ai/docs/v4/platforms/web.html);
- [Google Hangouts и Google Assistant](https://github.com/BotBuilderCommunity/) (доступно через сообщество);
- [Amazon Alexa](https://github.com/BotBuilderCommunity/) (доступно через сообщество).

## <a name="currently-available-adapters"></a>Доступные сейчас адаптеры

См. полный список всех доступных [адаптеров](https://botkit.ai/docs/v4/platforms/). Вы заметите, что некоторые каналы доступны как адаптеры. Именно вы решаете, когда использовать канал, а когда — адаптер.

### <a name="when-to-use-an-adapter"></a>Когда следует использовать адаптер

1. Служба не поддерживает нужный канал.
2. Согласно требованиям развертывания к безопасности и соответствию вы не можете полагаться на внешнюю службу.
3. Глубина функций для определенного канала не поддерживается.

### <a name="when-to-use-a-channel"></a>Когда следует использовать канал

1. Вам требуется совместимость между каналами — бот должен работать более чем с одним из доступных каналов.
2. Встроенная поддержка: корпорация Майкрософт предоставляет обслуживание и исправления для каждого канала при каждом обновлении стороннего продукта.
3. Требуется доступ к дополнительным эксклюзивным каналам Майкрософт, включая Microsoft Teams.
4. Вы хотите использовать графический пользовательский интерфейс, чтобы включить дополнительные каналы для вашего бота.

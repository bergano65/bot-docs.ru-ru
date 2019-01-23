---
redirect_url: /bot-framework/bot-builder-howto-v4-state
ms.openlocfilehash: a0d2b1295be1271e827d617ad09878ee8cfcd356
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54223919"
---
<a name="--"></a><!--
---
заголовок: Состояние и хранение | Документация Майкрософт description: Описывает диспетчер состояния, состояние общения и состояние пользователя в пакете SDK Bot Framework.
keywords: LUIS, состояние беседы, состояние пользователя, хранилище, диспетчер состояния author: DeniseMak ms.author: v-demak manager: kamrani ms.topic: article ms.service: bot-service ms.subservice: sdk ms.date: 02/15/2018 monikerRange: 'azure-bot-service-4.0'
---

# <a name="state-and-storage"></a>Состояние и хранение
[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Важным аспектом качественной разработки бота является отслеживание контекста общения, то есть бот должен запоминать, например, ответы на предыдущие вопросы.
В зависимости от того, для чего используется ваш бот, вам может понадобиться отслеживать информацию о состоянии или хранении данных дольше, чем время существования общения.
*Состояние* бота — это информация, которую программа запоминает, чтобы адекватно отвечать на входящие сообщения. Пакет SDK Bot Framework предоставляет классы, которые сохраняют и извлекают данные о состоянии в виде объекта, связанного с пользователем или беседой.

* **Свойства общения** позволяют боту отслеживать текущее общение с пользователем. Если ваш бот должен выполнить последовательные шаги или переключиться между темами общения, вы можете использовать свойства общения для управления шагами в последовательности или для отслеживания текущей темы. Так как свойства диалога отражают состояние текущего процесса общения, они обычно очищаются в конце диалога, когда бот получает действие _конец общения_.
* **Свойства пользователя** можно использовать для многих целей, например для определения места остановки предыдущего общения пользователя или просто для приветствия вернувшегося пользователя по имени. Если сохранить параметры пользователя, то с помощью этих данных можно настроить общение в следующий раз. Например, можно оповестить пользователя о новостной статье на интересующую его тему или оповестить его о том, что освободилось время для встречи. Эти параметры следует очистить, если бот получает действие _удалить данные пользователя_.

Можно использовать [Хранилище](bot-builder-howto-v4-storage.md) для чтения и записи в постоянное хранилище. Это позволяет боту выполнять такие действия, как обновление общих ресурсов, запись ответов на приглашение или голосов, чтение архивных данных о погоде. Точно так же, как приложение использует хранилище для достижения своих целей, бот может выполнять вышеперечисленные действия во время общения с пользователем.

<!-- 
*Conversation state* pertains to the current conversation that the user is having with your bot. When the conversation ends, your bot deletes this data.

You can also store *user state* that persists after a conversation ends. For example, if you store a user's preferences, you can use that information to customize the conversation the next time you chat. For example, you might alert the user to a news article about a topic that interests her, or alert a user when an appointment becomes available. 
-->

<!-- You should generally avoid saving state using a global variable or function closures.
Doing so will create issues when you want to scale out your bot. Instead, use the conversation state and user state middleware that the BotBuilder SDK provides --> 

<!--
## Types of underlying storage

The SDK provides bot state manager middleware to persist conversation and user state. State can be accessed using the bot's context. This state manager can use Azure Table Storage, file storage, or memory storage as the underlying data storage. You can also create your own storage components for your bot.

Bots built using Azure Table Storage can be designed to be stateless and scalable across multiple compute nodes.

> [!NOTE] 
> File and memory storage won't scale across nodes.

## Writing directly to storage

You can also use the Bot Framework SDK to read and write data directly to storage, without using middleware or without using the bot context. This can be appropriate to data that your bot uses, that comes from a source outside your bot's conversation flow.

For example, let's say your bot allows the user to ask for the weather report, and your bot retrieves the weather report for a specified date, by reading it from an external database. The content of the weather database isn't dependent on user information or the conversation context, so you could just read it directly from storage instead of using the state manager.  See [How to write directly to storage](bot-builder-howto-v4-storage.md) for an example.

## Next steps

Next, lets get into how activities are processed, in depth, and how we respond to them.

> [!div class="nextstepaction"]
> [Activity Processing](bot-builder-concept-activity-processing.md)

## Additional resources

- [How to save state](bot-builder-howto-v4-state.md)
- [How to write directly to storage](bot-builder-howto-v4-storage.md)

-->

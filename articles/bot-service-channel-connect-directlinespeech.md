---
title: Подключение бота к каналу Direct Line Speech (предварительная версия)
titleSuffix: Bot Service
description: Общие сведения и процедура подключения существующих ботов Bot Framework к каналу Direct Line Speech для двустороннего голосового взаимодействия с высокой степенью надежности и низкой задержкой.
services: bot-service
author: trrwilson
manager: nitinme
ms.service: bot-service
ms.subservice: bot-service
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: travisw
ms.custom: ''
ms.openlocfilehash: 8e0d2939078e1e27162c7056373e95790a03eb88
ms.sourcegitcommit: 5042e31bc6b2762d7a6636e98c8f496b90ea33c1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/07/2019
ms.locfileid: "65240430"
---
# <a name="connect-a-bot-to-direct-line-speech-preview"></a>Подключение бота к каналу Direct Line Speech (предварительная версия)

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

Вы можете настроить бот так, чтобы клиентские приложения могли взаимодействовать с ним через канал Direct Line Speech.

Подключение созданного бота к каналу Direct Line Speech предоставит клиентским приложениям подключение с низкой задержкой и высокой надежностью через [пакет SDK службы "Речь"](https://aka.ms/speech/sdk). Эти подключения оптимизированы для двустороннего естественного голосового взаимодействия. Дополнительные сведения о Direct Line Speech и создании клиентских приложений см. в [описании пользовательского виртуального голосового помощника](https://aka.ms/bots/speech/va).  

## <a name="sign-up-for-direct-line-speech-preview"></a>Регистрация для использования Direct Line Speech (предварительная версия)

Сейчас Direct Line Speech предоставляется в предварительной версии и требует несложной регистрации через [портал Azure](https://portal.azure.com). Этот процесс описан ниже. После утверждения вы получите доступ к каналу.

## <a name="add-the-direct-line-speech-channel"></a>Добавление канала Direct Line Speech

1. Чтобы добавить канал Direct Line Speech, откройте нужный бот на [портале Azure](https://portal.azure.com) и щелкните **Каналы** в колонке конфигурации.

    ![Выделенное расположение для выбора каналов для подключения](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-selectchannel.png "Selecting channels")

1. На странице выбора канала найдите и щелкните `Direct Line Speech`, чтобы выбрать канал.

    ![Выбор канала Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-connectspeechchannel.png "Connecting Direct Line Speech")

1. Если вы еще не получили утверждение для доступа, вы увидите страницу для запроса доступа. Введите требуемые данные и щелкните "Запросить". Появится страница подтверждения. Пока запрос ожидает утверждения, вы не сможете попасть на другие страницы.   

1. После утверждения доступа здесь будет отображаться страница конфигурации Direct Line Speech. Ознакомьтесь с условиями использования и щелкните `Save`, чтобы подтвердить выбор канала.

    ![Сохранение конфигурации включенного канала Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-savechannel.png "Save the channel configuration")

## <a name="enable-the-bot-framework-protocol-streaming-extensions"></a>Включение расширений протокола потоковой передачи Bot Framework

Подключив канал Direct Line Speech к боту, следует включить поддержку протокола потоковой передачи Bot Framework, чтобы организовать оптимальное взаимодействие с низкой задержкой.

1. Если вы этого еще не сделали, откройте колонку бота на [портале Azure](https://portal.azure.com). 

1. Щелкните **Параметры** в категории **Bot Management** (Управление ботом) (расположена под элементом **Каналы**). Установите флажок **Enable Streaming Endpoint** (Включить конечную точку потоковой передачи).

    ![Включение протокола потоковой передачи](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablestreamingsupport.png "Enable streaming extension support")

1. В верхней части страницы нажмите кнопку **Сохранить**.

1. В той же колонке в категории **Параметры службы приложений** щелкните **Конфигурация**.

    ![Переход к параметрам службы приложений](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-configureappservice.png "Configure the app service")

1. Щелкните `General settings` и выберите параметр, включающий поддержку `Web socket`.

    ![Включение WebSocket для службы приложений](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablewebsockets.png "Enable websockets")

1. Щелкните `Save` в верхней части страницы конфигурации.

1. Теперь для бота включены расширения протокола потоковой передачи Bot Framework. Теперь переходите к обновлению кода для бота, чтобы [интегрировать поддержку расширения потоковой передачи](https://aka.ms/botframework/addstreamingprotocolsupport) в существующий проект бота.

## <a name="manage-secret-keys"></a>Управление секретными ключами

Клиентским приложениям нужен секрет канала, чтобы подключиться к боту через канал Direct Line Speech. Сохранив выбор канала, вы можете получить эти секретные ключи на странице **Configure Direct Line Speech** (Настройка Direct Line Speech) на портале Azure.

![Получение секретных ключей для канала Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-getspeechsecretkeys.png "Getting secret keys for Direct Line Speech")

## <a name="adding-protocol-support-to-your-bot"></a>Добавление поддержки протокола в бот

Завершив подключение канала Direct Line Speech и настройку поддержки протокола потоковой передачи Bot Framework, остается лишь добавить в бот код для поддержки оптимизированного режима связи. Следуйте инструкциям по [добавлению поддержки расширений потоковой передачи](https://aka.ms/botframework/addstreamingprotocolsupport), чтобы обеспечить полную совместимость с Direct Line Speech.

## <a name="known-issues"></a>Известные проблемы

Обратите внимание на то, что служба находится на этапе предварительной версии и может измениться, что в свою очередь может повлиять на процесс разработки и производительность бота. Ниже приведен список известных проблем. 

1. В настоящее время служба развернута только в [регионе Azure](https://azure.microsoft.com/en-us/global-infrastructure/regions/) "Западная часть США 2". Мы в ближайшее время запустим ее в других регионах, чтобы все пользователи смогли воспользоваться голосовым взаимодействием с ботами с низкой задержкой.

1. Ожидаются незначительные изменения в полях элементов управления, например [serviceUrl](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#service-url).

1. Действия [conversationUpdate](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#conversation-update-activity) и [endOfCoversation](https://github.com/Microsoft/BotBuilder/blob/master/specs/botframework-activity/botframework-activity.md#end-of-conversation-activity), которые обозначают начало и окончание беседы и традиционно используются при формировании приветственных сообщений, будут изменены для согласованности с другими каналами.

1. Этот канал пока не поддерживает [SigninCard](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-connector-add-rich-cards?view=azure-bot-service-4.0). 
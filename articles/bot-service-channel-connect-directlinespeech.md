---
title: Подключение бота к каналу Direct Line Speech
titleSuffix: Bot Service
description: Общие сведения и процедура подключения существующих ботов Bot Framework к каналу Direct Line Speech для двустороннего голосового взаимодействия с высокой степенью надежности и низкой задержкой.
services: bot-service
author: trrwilson
manager: nitinme
ms.service: bot-service
ms.topic: conceptual
ms.date: 11/02/2019
ms.author: travisw
ms.openlocfilehash: d8831930934bef15c06679236efb7f4e7962b1aa
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117501"
---
# <a name="connect-a-bot-to-direct-line-speech"></a>Подключение бота к каналу Direct Line Speech

[!INCLUDE[applies-to-v4](includes/applies-to.md)]

Вы можете настроить бот так, чтобы клиентские приложения могли взаимодействовать с ним через канал Direct Line Speech.

Подключение созданного бота к каналу Direct Line Speech предоставит клиентским приложениям подключение с низкой задержкой и высокой надежностью через [пакет SDK службы "Речь"](https://aka.ms/speech/sdk). Эти подключения оптимизированы для двустороннего естественного голосового взаимодействия. Дополнительные сведения о Direct Line Speech и создании клиентских приложений см. в [описании пользовательского виртуального голосового помощника](https://aka.ms/bots/speech/va).

## <a name="add-the-direct-line-speech-channel"></a>Добавление канала Direct Line Speech

1. Чтобы добавить канал Direct Line Speech, откройте нужный бот на [портале Azure](https://portal.azure.com) и выберите ресурс **Регистрация канала бота** в разделе ресурсов. Щелкните элемент **Каналы** в колонке конфигурации.

    ![Выделенное расположение для выбора каналов для подключения](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-selectchannel.png "Выбор каналов")

1. На странице выбора канала найдите и щелкните `Direct Line Speech`, чтобы выбрать канал.

    ![Выбор канала Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-connectspeechchannel.png "Подключение к Direct Line Speech")

1. Настройте Direct Line Speech, как показано на рисунке ниже.

    ![Выбор канала Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-cognitivesericesaccount-selection.png "Выбор ресурса Cognitive Services")

    Для канала Direct Line Speech требуется ресурс Cognitive Services, в частности ресурс **Речь**. Вы можете выбрать существующий ресурс ресурсов или создать новый. Чтобы создать новый ресурс, сделайте следующее:

    - Перейдите на [портал Azure в раздел создания ресурсов](https://ms.portal.azure.com/#create/hub).
    - Найдите ресурс *Речь* в раскрывающемся списке и выберите его. Отобразится следующее:

        ![Создание ресурса "Речь" Cognitive Services](media/voice-first-virtual-assistants/create-speech-cognitive-resource.PNG "Создание ресурса "Речь" Cognitive Services")

    - Следуйте указаниям мастера.

    См. сведения о [создании ресурса Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account).

1. Ознакомьтесь с условиями использования и щелкните `Save`, чтобы подтвердить выбор канала.

    ![Сохранение конфигурации с включенным каналом Direct Line Speech](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-savechannel.png "Сохранение конфигурации канала")

## <a name="enable-the-bot-framework-protocol-streaming-extensions"></a>Включение расширений протокола потоковой передачи Bot Framework

Подключив канал Direct Line Speech к боту, следует включить поддержку протокола потоковой передачи Bot Framework, чтобы организовать оптимальное взаимодействие с низкой задержкой.

1. Если вы этого еще не сделали, откройте колонку бота на [портале Azure](https://portal.azure.com).

1. Щелкните **Параметры** в категории **Bot Management** (Управление ботом) (расположена под элементом **Каналы**). Установите флажок **Enable Streaming Endpoint** (Включить конечную точку потоковой передачи).

    ![Включение протокола потоковой передачи](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablestreamingsupport.png "Включение поддержки расширения потоковой передачи")

1. В верхней части страницы нажмите кнопку **Сохранить**.

1. В той же колонке в категории **Параметры службы приложений** щелкните **Конфигурация**.

    ![Переход к параметрам Службы приложений](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-configureappservice.png "Настройка Службы приложений")

1. Щелкните `General settings` и выберите параметр, включающий поддержку `Web socket`.

    ![Включение WebSocket для Службы приложений](media/voice-first-virtual-assistants/bot-service-channel-directlinespeech-enablewebsockets.png "Включение WebSocket")

1. Щелкните `Save` в верхней части страницы конфигурации.

1. Теперь для бота включены расширения протокола потоковой передачи Bot Framework. Теперь переходите к обновлению кода для бота, чтобы [интегрировать поддержку расширения потоковой передачи](https://aka.ms/botframework/addstreamingprotocolsupport) в существующий проект бота.

## <a name="adding-protocol-support-to-your-bot"></a>Добавление поддержки протокола в бот

Завершив подключение канала Direct Line Speech и настройку поддержки протокола потоковой передачи Bot Framework, остается лишь добавить в бот код для поддержки оптимизированного режима связи. Следуйте инструкциям по [добавлению поддержки расширений потоковой передачи](https://aka.ms/botframework/addstreamingprotocolsupport), чтобы обеспечить полную совместимость с Direct Line Speech.



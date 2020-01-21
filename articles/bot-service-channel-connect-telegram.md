---
title: Создание бота для Telegram — Служба Azure Bot
description: Сведения о настройке подключения бота к Telegram.
keywords: configure bot, Telegram, bot channel, Telegram bot, access token
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: fd60fbb10e9560e3df6e427f0d5193df463ec473
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791822"
---
# <a name="connect-a-bot-to-telegram"></a>Подключение бота к Telegram

Вы можете настроить взаимодействие бота с другими пользователями с помощью программы для обмена сообщениями Telegram.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="visit-the-bot-father-to-create-a-new-telegram-bot"></a>Получение доступа к BotFather для создания бота Telegram

<a href="https://telegram.me/botfather" target="_blank">Создайте бот Telegram</a> с помощью BotFather.

![Получение доступа к BotFather](~/media/channels/tg-StepVisitBotFather.png)

## <a name="create-a-new-telegram-bot"></a>Создание бота Telegram
Чтобы создать бот Telegram, отправьте команду `/newbot`.

![Создание бота](~/media/channels/tg-StepNewBot.png)

### <a name="specify-a-friendly-name"></a>Указание понятного имени

Присвойте боту Telegram понятное имя.

![Присвоение боту понятного имени](~/media/channels/tg-StepNameBot.png)

### <a name="specify-a-username"></a>Указание имени пользователя

Присвойте боту Telegram уникальное имя пользователя.

![Присвоение боту уникального имени](~/media/channels/tg-StepUsername.png)

### <a name="copy-the-access-token"></a>Копирование маркера доступа

Скопируйте маркер доступа бота Telegram.

![Копирование маркера доступа](~/media/channels/tg-StepBotCreated.png)

## <a name="enter-the-telegram-bots-access-token"></a>Ввод маркера доступа бота Telegram

Перейдите к разделу **Каналы** бота на портале Azure и нажмите кнопку **Telegram**. 

> [!NOTE]
>  Пользовательский интерфейс портала Azure будет выглядеть немного иначе, когда вы подключите бота к Telegram. 

![Выбор Telegram в каналах](~/media/channels/tg-connectBot-Azure.png)

Вставьте ранее скопированный маркер в поле **Маркер доступа** и нажмите кнопку **Сохранить**.

![Маркер доступа Telegram](~/media/channels/tg-accessToken-Azure.png)

Бот настроен для взаимодействия с пользователями в Telegram. 

![Включение бота Telegram](~/media/channels/tg-botEnabled-Azure.png)

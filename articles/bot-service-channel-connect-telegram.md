---
title: Создание бота для Telegram | Документация Майкрософт
description: Сведения о настройке подключения бота к Telegram.
keywords: configure bot, Telegram, bot channel, Telegram bot, access token
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: efe38392117fb871b2b98e3f1d8d798bfaef0c41
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49996511"
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

Вставьте ранее скопированный маркер в поле **Токен доступа** и нажмите кнопку **Отправить**.

## <a name="enable-the-bot"></a>Включение бота
Установите флажок **Enable this bot on Telegram** (Включить этот бот для Telegram). Затем щелкните **I'm done configuring Telegram** (Настройка Telegram завершена).

Как только вы выполните эти действия, ваш бот будет настроен для взаимодействия с пользователями с помощью Telegram.

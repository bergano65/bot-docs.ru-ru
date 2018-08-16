---
title: Настройка бота для работы в одном или нескольких каналах | Документация Майкрософт
description: Сведения о настройке бота для работы в одном или нескольких каналах с помощью портала Bot Framework.
keywords: каналы бота, настройка, Кортана, Facebook Messenger, Kik, Slack, Skype, портал Azure
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: cb682bf77f801c98d00deffa0fc63249962248cd
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352923"
---
# <a name="connect-a-bot-to-channels"></a>Подключение бота к каналам

Канал представляет собой подключение между Bot Framework и коммуникационными приложениями. Подключение бота настраивается для каналов, на которых он будет доступен. Например, бот, подключенный к каналу Skype, можно добавить в список контактов и пользователи Skype смогут с ним взаимодействовать. 

В каналы включено много популярных служб, например, [Кортана](bot-service-channel-connect-cortana.md), [Facebook Messenger](bot-service-channel-connect-facebook.md), [Kik](bot-service-channel-connect-kik.md) и [Slack](bot-service-channel-connect-slack.md), а также много других. [Skype](https://dev.skype.com/bots) и Веб-чат предварительно настроены. 

Используя [портал Azure](https://portal.azure.com), можно легко и быстро подключиться к каналам.

## <a name="get-started"></a>Начало работы

Чтобы запустить бот на большинстве каналов, ему необходимо указать сведения об их конфигурации. Большинство каналов требуют, чтобы бот имел учетную запись на канале, а другие, например, Facebook Messenger, требуют, чтобы бот также имел приложение, зарегистрированное на канале.

Чтобы настроить бот для подключения к каналу, выполните следующие действия.

1. Войдите на <a href="https://portal.azure.com" target="_blank">портал Azure</a>.
1. Выберите бот, который требуется настроить.
3. В колонке "Служба ботов" в разделе **Bot Management** (Управление ботом) щелкните **Каналы**.
4. Щелкните значок канала, который требуется добавить боту.

![Подключение к каналам](~/media/channels/connect-to-channels.png)

После настройки канала пользователи смогут использовать ваш бот.

## <a name="publish-a-bot"></a>Публикация бота

Процесс публикации на каждом канале разный.

[!INCLUDE [publishing](~/includes/snippet-publish-to-channel.md)]


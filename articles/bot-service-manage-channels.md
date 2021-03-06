---
title: Настройка бота для работы в одном или нескольких каналах — Служба Azure Bot
description: Сведения о настройке бота для работы в одном или нескольких каналах с помощью портала Bot Framework.
keywords: каналы бота, настройка, Кортана, Facebook Messenger, Kik, Slack, Skype, портал Azure
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/22/2018
ms.openlocfilehash: 226d31b255d18b39ed1e3817b76d65a9bdd951bc
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75794105"
---
# <a name="connect-a-bot-to-channels"></a>Подключение бота к каналам

Каналом называется соединение между ботом и коммуникационными приложениями. Подключение бота настраивается для каналов, на которых он будет доступен. Служба Bot Framework, настраиваемая через портал Azure, соединяет бот с этими каналами и упрощает взаимодействие между ботом и пользователем. Вы сможете подключить разные популярные службы, такие как [Cortana](bot-service-channel-connect-cortana.md), [Facebook Messenger](bot-service-channel-connect-facebook.md), [Kik](bot-service-channel-connect-kik.md), [Slack](bot-service-channel-connect-slack.md) и многие другие. Канал [веб-чата](bot-service-channel-connect-webchat.md) предварительно настроен. Помимо стандартных каналов, предоставляемых службой Bot Connector, также можно подключить бот к собственному клиентскому приложению, используя [Direct Line](bot-service-channel-connect-directline.md) в качестве канала.

Служба Bot Framework позволяет боту ничего не знать об используемом канале, и берет на себя нормализацию всех отправляемых ботом сообщений. Это включает его преобразование из схемы Bot Framework в схему канала. Однако, если канал не поддерживает все аспекты схемы Bot Framework, служба попытается преобразовать сообщение в формат, поддерживаемый каналом. Например, если бот отправляет сообщение, содержащее карточку с кнопками действий, в канал электронной почты, соединитель может отправить карточку как изображение и включить действия как ссылки в текст сообщения.

Чтобы запустить бот на большинстве каналов, ему необходимо указать сведения об их конфигурации. Большинство каналов требуют, чтобы бот имел учетную запись на канале, а другие, например, Facebook Messenger, требуют, чтобы бот также имел приложение, зарегистрированное на канале.

Чтобы настроить бот для подключения к каналу, выполните следующие действия.

1. Войдите на <a href="https://portal.azure.com" target="_blank">портал Azure</a>.
2. Выберите бот, который требуется настроить.
3. В колонке "Служба ботов" в разделе **Bot Management** (Управление ботом) щелкните **Каналы**.
4. Щелкните значок канала, который требуется добавить боту.

![Подключение к каналам](./media/channels/connect-to-channels.png)

После настройки канала пользователи смогут использовать ваш бот.

## <a name="publish-a-bot"></a>Публикация бота

Процесс публикации на каждом канале разный.

[!INCLUDE [publishing](./includes/snippet-publish-to-channel.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

Этот пакет SDK содержит примеры, которые можно использовать для создания ботов. Посетите [репозиторий примеров в GitHub](https://github.com/Microsoft/BotBuilder-samples), чтобы просмотреть список примеров.

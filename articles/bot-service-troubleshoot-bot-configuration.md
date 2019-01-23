---
title: Устранение неполадок, связанных с конфигурацией бота | Документация Майкрософт
description: Устранение неполадок с конфигурацией для развернутого бота.
keywords: troubleshoot, configuration, web chat, problems.
author: jonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/20/2018
ms.openlocfilehash: 8a3ff4a30e3041937ba831efc237343c9aa27e62
ms.sourcegitcommit: 8161753641368567f239e24a35ad61768acccd8e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/10/2019
ms.locfileid: "54202550"
---
# <a name="troubleshoot-bot-configuration-issues"></a>Устранение неполадок, связанных с конфигурацией бота

При устранении неполадок с ботом прежде всего проверьте его работу в веб-чате. Это позволит понять, с чем связана проблема: с самим ботом (если он не работает ни в одном канале) или только с определенным каналом (если бот успешно работает в некоторых каналах и не работает в других).

## <a name="test-in-web-chat"></a>Тестирование в веб-чате

1. Откройте ресурс бота на [портале Azure](http://portal.azure.com/).
1. Откройте панель **Test in Web Chat** (Тестирование в веб-чате).
1. Отправьте боту сообщение.

![Тестирование в веб-чате](./media/test-in-webchat.png)

Если бот не возвращает ожидаемые выходные данные, перейдите к разделу [Бот не работает в веб-чате](#bot-does-not-work-in-web-chat). В противном случае перейдите к разделу [Бот работает в веб-чате, но не работает в других каналах](#bot-works-in-web-chat-but-not-in-other-channels).

## <a name="bot-does-not-work-in-web-chat"></a>Бот не работает в веб-чате

Неработоспособность бота может быть вызвана разными причинами. Наиболее вероятно, что приложение бота не работает и не может принимать сообщения. Также возможно, что бот получает сообщения но не может ответить.

Чтобы определить, работает ли бот, сделайте следующее.

1. Откройте панель **Обзор**.
1. Скопируйте значение **Messaging endpoint** (Конечная точка обмена сообщениями) и вставьте его в адресную строку браузера.

Если эта конечная точка возвращает ошибку HTTP 405, значит бот доступен и может отвечать на сообщения. В этом случае проверьте, превышается ли для бота [время ожидания](https://github.com/daveta/analytics/blob/master/troubleshooting_timeout.md) и [завершается ли его работа с ошибкой HTTP 5xx](bot-service-troubleshoot-500-errors.md).

Если конечная точка возвращает сообщение об ошибке "This site can't be reached" (Этот сайт недоступен) или "Не удается открыть эту страницу", значит бот не работает и его следует развернуть повторно.

## <a name="bot-works-in-web-chat-but-not-in-other-channels"></a>Бот работает в веб-чате, но не работает в других каналах

Если бот успешно работает веб-чате, но не может работать в каких-либо других каналах, проверьте следующие возможные причины.

- [Проблемы с конфигурацией канала](#channel-configuration-issues)
- [Особое поведение для канала](#channel-specific-behavior)
- [Сбой канала](#channel-outage)

### <a name="channel-configuration-issues"></a>Проблемы с конфигурацией канала

Вполне возможно, что параметры конфигурации канала заданы неправильно или изменились под внешним воздействием. Например, бот настроил канал для связи с определенной страницей на Facebook, которая позднее была удалена. В этом случае проще всего удалить такой канал и заново настроить конфигурацию для него.

Ниже приведены ссылки на инструкции по настройке каналов, поддерживаемых платформой Bot Framework.

- [Cortana](bot-service-channel-connect-cortana.md)
- [DirectLine](bot-service-channel-connect-directline.md)
- [Электронная почта](bot-service-channel-connect-email.md)
- [Facebook](bot-service-channel-connect-facebook.md)
- [GroupMe](bot-service-channel-connect-groupme.md)
- [Kik](bot-service-channel-connect-kik.md)
- [Microsoft Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview)
- [Skype](bot-service-channel-connect-skype.md)
- [Skype для бизнеса](bot-service-channel-connect-skypeforbusiness.md)
- [Slack](bot-service-channel-connect-slack.md)
- [Telegram](bot-service-channel-connect-telegram.md)
- [Twilio](bot-service-channel-connect-twilio.md)

### <a name="channel-specific-behavior"></a>Особое поведение для канала

Возможно, в реализации разных каналов различаются некоторые функции. Например, сейчас не все каналы поддерживают адаптивные карточки. Большинство каналов поддерживают кнопки, но отображают их по-разному. Если вы заметите различия в поведении определенных типов сообщений в разных каналах, используйте [Channel Inspector](https://docs.botframework.com/channel-inspector/channels/Skype).

Ниже приведены некоторые ссылки с дополнительной информацией по отдельным каналам.

- [Add bots to Microsoft Teams apps](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-overview) (Добавление ботов в приложения Microsoft Teams)
- [Facebook: Introduction to the Messenger Platform](https://developers.facebook.com/docs/messenger-platform/introduction) (Facebook: общие сведения о платформе Messenger)
- [Principles of Cortana Skills design](https://docs.microsoft.com/cortana/skills/design-principles) (Принципы проектирования навыков для Кортаны)
- [Сведения о Skype для разработчиков](https://dev.skype.com/bots)
- [Slack: Enabling interactions with bots](https://api.slack.com/bot-users) (Slack: организация взаимодействия с ботами)

### <a name="channel-outage"></a>Сбой канала

В некоторых случаях может прерываться обслуживание отдельных каналов. Обычно такие простои не продолжаются долго. Но если у вас есть основания считать, что произошел сбой канала, проверьте сведения на веб-сайте этого канала или в социальных сетях.

Есть еще один способ быстро проверить наличие сбоев в канале: создайте тестовый бот (например, простейший повторитель сообщений) и добавьте в него проблемный канал. Если тестовый бот успешно работает с частью каналов, но не работает с другими каналами, проблема не в рабочем боте.

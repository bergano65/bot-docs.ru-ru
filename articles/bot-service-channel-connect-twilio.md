---
title: Подключение бота к Twilio | Документация Майкрософт
description: Сведения о настройке подключения бота к Twilio.
keywords: Twilio, bot channels, SMS, App, phone, configure Twilio, cloud communication, text
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 10/9/2018
ms.openlocfilehash: 817623dd04612cd07d8877c8e9a199c05a2fd9e8
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693619"
---
# <a name="connect-a-bot-to-twilio"></a>Подключение бота к Twilio

Вы можете настроить взаимодействие бота с пользователями с помощью облачной платформы обмена данными Twilio.

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>Вход в учетную запись Twilio или ее создание для отправки и получения SMS-сообщений

Если у вас нет учетной записи Twilio, <a href="https://www.twilio.com/try-twilio" target="_blank">создайте ее</a>.

## <a name="create-a-twiml-application"></a>Создание приложения TwiML

<a href="https://support.twilio.com/hc/en-us/articles/223180928-How-Do-I-Create-a-TwiML-App-" target="_blank">Создайте приложения TwiML</a> в соответствии с инструкциями.

![Создание приложения](~/media/channels/twi-StepTwiml.png)

В разделе **Свойства**, введите **FRIENDLY NAME** (Понятное имя). В этом руководстве для примера используется имя "My TwiML app". Поле **REQUEST URL** (URL-адрес запроса) под Voice можно оставить пустым. В разделе **Messaging** (Обмен сообщениями) для параметра **Request URL** (URL-адрес запроса) введите значение `https://sms.botframework.com/api/sms`.

## <a name="select-or-add-a-phone-number"></a>Выбор или добавление номера телефона

Следуйте представленным <a href = "https://support.twilio.com/hc/en-us/articles/223180048-Adding-a-Verified-Phone-Number-or-Caller-ID-with-Twilio" target="_blank">здесь</a> инструкциям, чтобы добавить идентификатор проверенного вызывающего объекта через сайт консоли. После завершения вы увидите проверенный номер в поле **Active Numbers** (Активные номера) раздела **Manage Numbers** (Управление номерами).

![Установка номера телефона](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-voice-and-messaging"></a>Указание приложений для голосовой связи и обмена сообщениями

Щелкните число и перейдите к разделу **Настройка**. Для голосовой связи и обмена сообщениями укажите в поле **CONFIGURE WITH** (Настройка с помощью) значение "TwiML App", а в поле **TWIML APP** (Приложение TwiML) выберите "My TwiML app". Завершив настройку, щелкните **Сохранить**.

![Выбор приложения](~/media/channels/twi-StepPhone2.png)

Вернитесь к разделу **Управление номерами**. Вы увидите, что для голосовой связи и обмена сообщениями теперь в конфигурации указано приложение TwiML.

![Указанный номер](~/media/channels/twi-StepPhone3.png)


## <a name="gather-credentials"></a>Получение учетных данных

Вернитесь к [домашней странице консоли](https://www.twilio.com/console/), где на панели мониторинга проекта вы найдете идентификатор безопасности учетной записи и маркер проверки подлинности, как показано ниже.

![Сбор учетных данных приложения](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>Отправка учетных данных

В отдельном окне браузера откройте сайт Bot Framework, расположенный по адресу https://dev.botframework.com/. 

- Щелкните **My bots** (Мои боты) и выберите бот, который нужно подключить к Twilio. Это действие перенесет вас на портал Azure.
- Выберите **Каналы** в разделе **Управление ботами**. Щелкните значок Twilio (SMS).
- Введите номер телефона, идентификатор безопасности учетной записи и маркер проверки подлинности, которые вы записали ранее. Завершив настройку, щелкните **Сохранить**.

![Отправка учетных данных](~/media/channels/twi-StepSubmit.png)

Как только вы выполните эти действия, ваш бот будет настроен для взаимодействия с пользователями с помощью Twilio.

## <a name="also-available-as-an-adapter"></a>Также предоставляется как адаптер

Этот канал также [предоставляется как адаптер](https://botkit.ai/docs/v4/platforms/twilio-sms.html). См. подробнее о выборе между адаптером и каналом в списке [доступных адаптеров](bot-service-channel-additional-channels.md#currently-available-adapters).
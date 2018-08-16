---
title: Подключение бота к Twilio | Документация Майкрософт
description: Сведения о настройке подключения бота к Twilio.
keywords: Twilio, bot channels, SMS, App, phone, configure Twilio, cloud communication, text
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 7e09126d50cfbebfc0aad0ee7fcb71b4e7551a7d
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301214"
---
# <a name="connect-a-bot-to-twilio"></a>Подключение бота к Twilio

Вы можете настроить взаимодействие бота с пользователями с помощью облачной платформы обмена данными Twilio.

## <a name="log-in-to-or-create-a-twilio-account-for-sending-and-receiving-sms-messages"></a>Вход в учетную запись Twilio или ее создание для отправки и получения SMS-сообщений

Если у вас нет учетной записи Twilio, <a href="https://www.twilio.com/try-twilio" target="_blank">создайте ее</a>.

## <a name="create-a-twiml-application"></a>Создание приложения TwiML

<a href="https://www.twilio.com/user/account/messaging/dev-tools/twiml-apps/add" target="_blank">Создайте приложение TwiML</a>

![Создание приложения](~/media/channels/twi-StepTwiml.png)

 В разделе Messaging (Обмен сообщениями) должен быть указан такой URL-адрес запроса: **https://sms.botframework.com/api/sms**.

## <a name="select-or-add-a-phone-number"></a>Выбор или добавление номера телефона

<a href="https://www.twilio.com/user/account/phone-numbers/incoming" target="_blank">Выберите или добавьте номер телефона</a>. Щелкните номер, чтобы добавить его в созданное приложение TwiML.

![Установка номера телефона](~/media/channels/twi-StepPhone.png)

## <a name="specify-application-to-use-for-messaging"></a>Указание приложения для обмена сообщениями
В разделе **Messaging** (Обмен сообщениями) задайте в поле **TwiML App** (Приложение TwiML) имя созданного приложения TwiML.
Скопируйте значение **номера телефона** для дальнейшего использования.

![Указание приложения](~/media/channels/twi-StepPhone2.png)

## <a name="gather-credentials"></a>Получение учетных данных

<a href="https://www.twilio.com/user/account/settings" target="_blank">Получите учетные данные</a> и щелкните значок глаза для просмотра маркера проверки подлинности.

![Сбор учетных данных приложения](~/media/channels/twi-StepAuth.png)

## <a name="submit-credentials"></a>Отправка учетных данных

Введите номер телефона, SID учетной записи и маркер проверки подлинности, скопированный ранее, и щелкните **Submit Twilio Credentials** (Отправить учетные данные Twilio).

## <a name="enable-the-bot"></a>Включение бота
Установите флажок **Enable this bot on SMS** (Включить этот бот для SMS). Затем щелкните **I'm done configuring SMS** (Настройка SMS завершена).

Как только вы выполните эти действия, ваш бот будет настроен для взаимодействия с пользователями с помощью Twilio.


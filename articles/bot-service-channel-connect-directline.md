---
title: Подключение бота к службе Direct Line | Документация Майкрософт
description: Сведения о настройке подключения бота к службе Direct Line.
keywords: Direct Line, каналы бота, клиент пользователя, подключение к каналам, настройка
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/7/2019
ms.openlocfilehash: edfb61a4f4ca33089bce7d4b44ed242f83cbcc0d
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/09/2019
ms.locfileid: "68866626"
---
# <a name="connect-a-bot-to-direct-line"></a>Подключение бота к службе Direct Line

Установите связь между клиентским приложением и ботом с помощью канала Direct Line. 

## <a name="add-the-direct-line-channel"></a>Добавление канала Direct Line

Чтобы добавить канал Direct Line, откройте бот на [портале Azure](https://portal.azure.com/), щелкните колонку **Каналы**, а затем выберите **Direct Line**.

![Добавление канала Direct Line](media/bot-service-channel-connect-directline/directline-addchannel.png)

## <a name="add-new-site"></a>Добавление нового веб-сайта

Затем добавьте новый веб-сайт, где представлено клиентское приложение, которое необходимо подключить к боту. Щелкните **Добавить новый веб-сайт**, введите имя для веб-сайта и нажмите кнопку **Готово**.

![Добавление веб-сайта Direct Line](media/bot-service-channel-connect-directline/directline-addsite.png)

## <a name="manage-secret-keys"></a>Управление секретными ключами

Если сайт готов, платформа Bot Framework создает секретные ключи, которые клиентское приложение использует для [проверки подлинности](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md) запросов Direct Line API, чтобы обмениваться данными с ботом. Чтобы просмотреть ключ в виде обычного текста, щелкните **Показать** для соответствующего ключа.

![Отображение ключа Direct Line](media/bot-service-channel-connect-directline/directline-showkey.png)

Отобразившийся ключ скопируйте и сохраните в надежном месте. Используете этот ключ для [проверки подлинности](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md) запросов Direct Line API, которые издает клиент для обмена данными с ботом.
Кроме того, используйте Direct Line API для [обмена ключом для токена](~/rest-api/bot-framework-rest-direct-line-3-0-authentication.md#generate-token), который клиент может использовать, чтобы аутентифицировать свои последующие запросы в рамках одного общения.

![Копирование ключа Direct Line](media/bot-service-channel-connect-directline/directline-copykey.png)

## <a name="configure-settings"></a>Настройка параметров

В завершение настройте параметры для веб-сайта.

- Выберите версию протокола Direct Line, которую клиентское приложение будет использовать для обмена данными с ботом.

> [!TIP]
> При создании подключения между клиентским приложением и ботом, используйте Direct Line API 3.0.

Чтобы сохранить конфигурацию веб-сайта, нажмите кнопку **Готово**. Этот процесс можно повторять, начиная с команды [Добавить новый веб-сайт](#add-new-site), для каждого клиентского приложения, которое необходимо подключить к боту.

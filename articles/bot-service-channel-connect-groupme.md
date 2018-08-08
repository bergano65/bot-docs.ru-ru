---
title: Подключение бота к GroupMe | Документы Майкрософт
description: Сведения о настройке подключения бота к GroupMe.
keywords: канал бота, GroupMe, создание GroupMe, учетные данные
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2cc3081f75c755d14f32f02fe9d0d3a3c48bcf14
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305291"
---
# <a name="connect-a-bot-to-groupme"></a>Подключение бота к веб-чату

Вы можете настроить взаимодействие бота с другими пользователями с помощью программы для обмена сообщениями GroupMe.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

## <a name="sign-up-for-a-groupme-account"></a>Регистрация учетной записи GroupMe

Если у вас нет учетной записи GroupMe, [зарегистрируйтесь, чтобы создать учетную запись](https://web.groupme.com/signup).

## <a name="create-a-groupme-application"></a>Создание приложения GroupMe

[Создайте приложение GroupMe](https://dev.groupme.com/applications/new) для вашего бота.

Используйте следующий URL-адрес обратного вызова: `https://groupme.botframework.com/Home/Login`

![Создание приложения](~/media/channels/GM-StepApp.png)

## <a name="gather-credentials"></a>Получение учетных данных

1. В поле **URL-адрес перенаправления** скопируйте значение параметра **client_id =**.
2. Скопируйте значение параметра **Маркер доступа**.

![Копирование идентификатора клиента и маркера доступа](~/media/channels/GM-StepClientId.png)


## <a name="submit-credentials"></a>Отправка учетных данных

1. На странице dev.botframework.com вставьте скопированное значение параметра **client_id** в поле **Идентификатор клиента**.
2. Вставить значение **Маркера доступа** в поле **Маркер доступа**.
2. Выберите команду **Сохранить**.

![Ввод учетных данных](~/media/channels/GM-StepClientIDToken.png)

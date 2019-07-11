---
title: Подключение бота к Teams | Документация Майкрософт
description: Узнайте, как настроить бот для доступа через Teams.
keywords: Teams, канал бота, настройка Teams
author: kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 07/05/2019
ms.openlocfilehash: d2609e4294416691e156ba3dbd09eabc8e0d3423
ms.sourcegitcommit: b498649da0b44f073dc5b23c9011ea2831edb31e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/05/2019
ms.locfileid: "67592233"
---
# <a name="connect-a-bot-to-teams"></a>Подключение бота к Teams

Чтобы добавить канал Microsoft Teams, откройте бот на [портале Azure](https://portal.azure.com), щелкните колонку **Каналы** и выберите **Teams**.

![Добавление канала Teams](media/teams/connect-teams-channel.png)

Затем щелкните **Сохранить**.

![Сохранение канала Teams](media/teams/save-teams-channel.png)

Добавив канал Teams, перейдите на страницу **Каналы** и щелкните **Get bot embed code** (Получить код внедрения бота).

![Получение кода внедрения](media/teams/get-embed-code.png)

- Скопируйте часть кода с _https_ в диалоговом окне **Get bot embed code** (Получение кода внедрения бота). Например, `https://teams.microsoft.com/l/chat/0/0?users=28:b8a22302e-9303-4e54-b348-343232`. 

- В браузере вставьте этот адрес и выберите приложение Microsoft Teams (клиентское или веб-приложение), которое используется для добавления бота в Teams. Вы увидите бота как контакт, с которым вы можете обмениваться сообщениями в Microsoft Teams. 

## <a name="additional-information"></a>Дополнительная информация
См. подробнее о [Microsoft Teams](https://docs.microsoft.com/en-us/microsoftteams/platform/overview). 

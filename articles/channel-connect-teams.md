---
title: Подключение бота к Teams | Документация Майкрософт
description: Узнайте, как настроить бот для доступа через Teams.
keywords: Teams, канал бота, настройка Teams
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/26/2019
ms.openlocfilehash: 1ba0873c880bfb9b1e0f449039e98859e5571028
ms.sourcegitcommit: 0b647dc6716b0c06f04ee22ebdd7b53039c2784a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/28/2019
ms.locfileid: "70076540"
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

> [!IMPORTANT] 
> Добавлять бота по GUID рекомендуется только для тестирования. В противном случае функциональность бота будет существенно ограничена. Боты, используемые в рабочей среде, следует добавлять в Teams как часть приложения. См. сведения о [создании бота](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-create) и [тестировании и отладке бота Microsoft Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-test).


## <a name="additional-information"></a>Дополнительная информация
См. подробнее о [Microsoft Teams](https://docs.microsoft.com/en-us/microsoftteams/platform/overview). 

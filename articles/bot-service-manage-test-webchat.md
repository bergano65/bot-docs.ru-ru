---
title: Тестирование службы Bot в веб-чате | Документация Майкрософт
description: Узнайте, как протестировать службу Bot с помощью элемента управления "Веб-чат" на портале Azure.
keywords: test in web chat, azure portal
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 0c358f4e53f3fd64cce3635f644cc0f2f612e983
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300678"
---
# <a name="test-in-web-chat"></a>Тестирование в веб-чате
Служба Bot включает [элемент управления "Веб-чат"](bot-service-channel-connect-webchat.md), который поможет с легкостью протестировать бот. 

## <a name="test-a-bot-in-the-azure-portal-with-web-chat"></a>Тестирование бота на портале Azure с помощью веб-чата
Войдите на [портал Azure](https://portal.azure.com) и откройте колонку бота. В разделе управления ботами щелкните **Test in Web Chat** (Тестирование в веб-чате). Служба Bot будет загружать элемент управления "Веб-чат" и подключится к боту.

![Пользовательский интерфейс Test in Web Chat (Тестирование в веб-чате)](~/media/test-in-webchat/test-in-webchat.png)

Вы можете ввести текст, чтобы начать чат с ботом. Если ваш бот поддерживает речь, можно нажать кнопку микрофона, чтобы произнести речь для бота. Если ваш бот поддерживает вложения, вы можете отправить вложение, например изображение. Узнайте, как добавлять функции для бота с помощью [пакета SDK для Bot Builder](bot-builder-overview-getstarted.md).

> [!NOTE]
> Если через несколько минут элемент управления "Веб-чат" не загрузился полностью, попробуйте обновить страницу.

## <a name="next-steps"></a>Дополнительная информация
Теперь, когда вы знаете, как протестировать бот в Azure, узнайте, как выполнить более глубокое тестирование и отладку с помощью Bot Framework Emulator.

> [!div class="nextstepaction"]
> [Bot Framework Emulator](bot-service-debug-emulator.md)
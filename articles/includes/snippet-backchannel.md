---
ms.openlocfilehash: b320aadc876074a76fe209ad55a81cb70b1ddcac
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405543"
---
<a href="https://github.com/Microsoft/BotFramework-WebChat" target="_blank">Элемент управления веб-чата с открытым кодом</a> взаимодействует с ботами с помощью [Direct Line API](https://docs.botframework.com/restapi/directline3/#navtitle), что позволяет отправлять `activities` между клиентом и ботом. Наиболее распространенный тип действия — `message`, но существуют также другие типы. Например, тип действия `typing` указывает, что пользователь вводит текст или что бот составляет ответ. 

Механизм обратного канала можно использовать для обмена данными между клиентом и ботом, не показывая их пользователю, задав тип действия `event`. Элемент управления веб-чата будет автоматически игнорировать все действия с `type="event"`.
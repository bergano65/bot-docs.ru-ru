---
title: Пакет SDK Bot Builder для Node.js | Документация Майкрософт
description: Изучите пакет SDK Bot Builder для Node.js — мощную и простую в использовании платформу для создания ботов.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 24da2cc90908a1f22bee9d2cd007607b116ac2cc
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904207"
---
# <a name="bot-builder-sdk-for-nodejs"></a>Пакет SDK Bot Builder для Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Пакет SDK Bot Builder для Node.js — это мощная и удобная в использовании платформа, позволяющая разработчикам для Node.js создавать боты привычным способом.
С помощью этого пакета можно создавать различные разговорные пользовательские интерфейсы — от простых запросов до сеансов общения в свободной форме.

Логика общения бота размещается как веб-служба. Для создания веб-сервера бота в пакете SDK для Bot Builder используется <a href="http://restify.com">Restify</a> — популярная платформа, которая позволяет создавать веб-службы. Этот пакет SDK также совместим с <a href="http://expressjs.com/">Express</a>. Возможно использование и других платформ веб-приложений, но это может потребовать определенной адаптации. 

С помощью этого пакета SDK вы можете использовать преимущества следующих компонентов этого пакета. 

- Мощная система для создания диалогов, позволяющая инкапсулировать логику общения.
- Встроенные запросы для простых использования таких элементов, как элемент выбора "Да" или "Нет", строки, числа и перечисления, а также поддержка сообщений, содержащих изображения и вложения, и форматированных карточек, содержащих кнопки.
- Встроенная поддержка мощных платформ ИИ, таких как <a href="http://luis.ai" target="_blank">LUIS</a>.
- Встроенные распознаватели и обработчики событий, которые посредством диалога с пользователем предоставляют справку, указания по навигации, уточнение и получают подтверждение, если это необходимо.

## <a name="get-started"></a>Начало работы

Если вы новичок в разработке ботов, [создайте свой первый бот с помощью Node.js](bot-builder-nodejs-quickstart.md), следуя пошаговым указаниям. С их помощью вы сможете настроить проект, установить пакет SDK и запустить свой первый бот. 

Если вы не использовали пакет SDK Bot Builder для Node.js, то можете начать с [ключевых понятий](bot-builder-nodejs-concepts.md), которые помогут изучить основные компоненты пакета SDK для Bot Builder.

Чтобы бот подошел для основных сценариев пользователей, ознакомьтесь с [принципами проектирования](../bot-service-design-principles.md) и [изучите шаблоны](../bot-service-design-pattern-task-automation.md), чтобы получить необходимые сведения.

## <a name="get-samples"></a>Получение примеров

В примерах для [пакета SDK Bot Builder для Node.js](bot-builder-nodejs-samples.md) демонстрируются ориентированные на задачи боты, показывающие, как воспользоваться преимуществами функций в пакете SDK Bot Builder для Node.js. С помощью этих примеров можно быстро приступить к созданию отличных ботов с широкими возможностями.

## <a name="next-steps"></a>Дополнительная информация
> [!div class="nextstepaction"]
> [Основные понятия](bot-builder-nodejs-concepts.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

Приведенные ниже практические руководства демонстрируют различные возможности пакета SDK Bot Builder для Node.js.

* [Ответ на сообщения](bot-builder-nodejs-use-default-message-handler.md)
* [Обработка действий пользователя](bot-builder-nodejs-dialog-actions.md)
* [Распознавание намерения пользователя](bot-builder-nodejs-recognize-intent-messages.md)
* [Отправка форматированной карточки](bot-builder-nodejs-send-rich-cards.md)
* [Отправка вложений](bot-builder-nodejs-send-receive-attachments.md)
* [Сохранение данных пользователя](bot-builder-nodejs-save-user-data.md)


Если у вас возникли проблемы или предложения относительно пакета SDK Bot Builder для Node.js, изучите список доступных ресурсов в разделе о [поддержке](../bot-service-resources-links-help.md). 


[DesignGuide]: ../bot-service-design-principles.md 
[DesignPatterns]: ../bot-service-design-pattern-task-automation.md 
[HowTo]: bot-builder-nodejs-use-default-message-handler.md 

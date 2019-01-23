---
title: Пакет SDK Bot Framework для Node.js | Документация Майкрософт
description: Изучите пакет SDK Bot Framework для Node.js, который предоставляет мощную и простую в использовании платформу для создания ботов.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2e25237b616810f5ef10442fec41834568afcb59
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224747"
---
# <a name="bot-framework-sdk-for-nodejs"></a>Пакет SDK Bot Framework для Node.js

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-overview.md)
> - [Node.js](../nodejs/bot-builder-nodejs-overview.md)
> - [REST](../rest-api/bot-framework-rest-overview.md)

Пакет SDK Bot Framework для Node.js — это мощная и удобная в использовании платформа, которая позволяет разработчикам создавать боты на Node.js привычным способом.
С помощью этого пакета можно создавать различные разговорные пользовательские интерфейсы — от простых запросов до сеансов общения в свободной форме.

Логика общения бота размещается как веб-служба. Для создания веб-сервера бота в пакете SDK Bot Framework используется популярная платформа <a href="http://restify.com">Restify</a>, которая позволяет создать веб-сервер для бота. Этот пакет SDK также совместим с <a href="http://expressjs.com/">Express</a>. Возможно использование и других платформ веб-приложений, но это может потребовать определенной адаптации. 

С помощью этого пакета SDK вы можете использовать преимущества следующих компонентов этого пакета. 

- Мощная система для создания диалогов, позволяющая инкапсулировать логику общения.
- Встроенные запросы для простых использования таких элементов, как элемент выбора "Да" или "Нет", строки, числа и перечисления, а также поддержка сообщений, содержащих изображения и вложения, и форматированных карточек, содержащих кнопки.
- Встроенная поддержка мощных платформ ИИ, таких как <a href="http://luis.ai" target="_blank">LUIS</a>.
- Встроенные распознаватели и обработчики событий, которые посредством диалога с пользователем предоставляют справку, указания по навигации, уточнение и получают подтверждение, если это необходимо.

## <a name="get-started"></a>Начало работы

Если вы новичок в разработке ботов, [создайте свой первый бот с помощью Node.js](bot-builder-nodejs-quickstart.md), следуя пошаговым указаниям. С их помощью вы сможете настроить проект, установить пакет SDK и запустить свой первый бот. 

Если вы еще не использовали пакет SDK Bot Framework для Node.js, начните знакомство с [ключевых понятий](bot-builder-nodejs-concepts.md), которые помогут изучить основные компоненты пакета SDK для Bot Framework.

Чтобы бот подошел для основных сценариев пользователей, ознакомьтесь с [принципами проектирования](../bot-service-design-principles.md) и [изучите шаблоны](../bot-service-design-pattern-task-automation.md), чтобы получить необходимые сведения.

## <a name="get-samples"></a>Получение примеров

В [примерах для пакета SDK Bot Framework для Node.js](bot-builder-nodejs-samples.md) демонстрируются ориентированные на задачи боты, которые используют функции пакета SDK Bot Framework для Node.js. С помощью этих примеров можно быстро приступить к созданию отличных ботов с широкими возможностями.

## <a name="next-steps"></a>Дополнительная информация
> [!div class="nextstepaction"]
> [Основные понятия](bot-builder-nodejs-concepts.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

Приведенные ниже практические руководства демонстрируют различные возможности пакета SDK Bot Framework для Node.js.

* [Ответ на сообщения](bot-builder-nodejs-use-default-message-handler.md)
* [Обработка действий пользователя](bot-builder-nodejs-dialog-actions.md)
* [Распознавание намерения пользователя](bot-builder-nodejs-recognize-intent-messages.md)
* [Отправка форматированной карточки](bot-builder-nodejs-send-rich-cards.md)
* [Отправка вложений](bot-builder-nodejs-send-receive-attachments.md)
* [Сохранение данных пользователя](bot-builder-nodejs-save-user-data.md)


Если у вас возникли проблемы или есть предложения относительно пакета SDK Bot Framework для Node.js, см. список доступных ресурсов в статье по [поддержке](../bot-service-resources-links-help.md). 


[DesignGuide]: ../bot-service-design-principles.md 
[DesignPatterns]: ../bot-service-design-pattern-task-automation.md 
[HowTo]: bot-builder-nodejs-use-default-message-handler.md 

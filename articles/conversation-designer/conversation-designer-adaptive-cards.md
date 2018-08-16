---
title: Настройка адаптивных карточек | Документация Майкрософт
description: Узнайте, как настроить адаптивные карточки.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 01f52a6aa6e2c9fc3d2613ff03386a7d87e64a3a
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39306067"
---
# <a name="configure-adaptive-cards"></a>Настройка адаптивных карточек
> [!IMPORTANT]
> Conversation Designer пока доступен не для всех клиентов. Дополнительные сведения о доступности Conversation Designer будут опубликованы позже в этом году.

<a href="http://adaptivecards.io" target="_blank">Адаптивные карточки</a> — это новая схема, определяющая форматированные карточки в пользовательском интерфейсе для использования в нескольких разных конечных точках, включая каналы Microsoft Bot Framework. 

Conversation Designer предоставляет тесно интегрированную среду разработки для создания, предварительного просмотра и использования адаптивных карточек в ботах. 

Адаптивные карточки можно определить в нескольких основных местах.

- Простой ответ на [действие](conversation-designer-tasks.md) для задачи.
- В состоянии отзыва в диалоге.
- В состояниях запросов в диалоге. Обратите внимание на то, что у запросов могут быть отдельные карточки: одна для ответа, а другая для повторного запроса.

Чтобы определить адаптивную карточку, перейдите в соответствующий редактор. Выберите один из имеющихся шаблонов адаптивных карточек или создайте собственный шаблон в редакторе кода JSON. 

<!--TODO: Insert screenshot -->

При создании карточки на портале разработчика отображается ее изображение для предварительного просмотра.

> [!NOTE]
> Разработка функций адаптивных карточек продолжается. В данный момент не все каналы поддерживают все функции адаптивных карточек. Чтобы узнать, какие функции поддерживает каждый канал, ознакомьтесь с разделом [Channel status](/adaptive-cards/get-started/bots#channel-status) (Состояние канала).

## <a name="input-form"></a>Форма ввода

Адаптивные карточки могут содержать формы ввода. В Conversation Designer формы объединяются с сущностями задач. Например, если поле имеет `id` со значением **myName** и выполняется действие формы `Submit`, то будет создана сущность `taskEntity` с именем **myName**, которая будет содержать значение этого поля. 

Во фрагменте кода ниже показано, как сущность **myName** определена в коде.

``javascript
{
   "type": "Input.Text",
   "id": "myName",
   "placeholder": "Last, First"
}
``

Кроме того, если идентификатор поля имеет значение `@task`, то значение поля будет использоваться в качестве имени задачи. При активации этого поля (например, при нажатии кнопки) будет выполняться именованная задача. 

Для примера возьмем этот фрагмент кода.

``javascript
{
  'type': 'Action.Submit',
  'title': 'Search',
  'speak': '<s>Search</s>',
  'data': {
    '@task': 'Hotel Search'
  }
}
``

При нажатии кнопки активируется действие submit и `context.sticky` присваивается значение `Hotel Search`. В результате будет выполнена задача **Hotel Search**. Чтобы использовать эту функцию, убедитесь, что `@task` совпадает с именем задачи, которую вы определили в Conversation Designer.

## <a name="use-entities-and-language-generation-templates"></a>Использование сущностей и шаблонов генерации текста
Адаптивные карточки поддерживают полное разрешение генерации текста.

* `entityName` использует сущности внутри карточки.
* `responseTemplateName` использует шаблоны простых или условных ответов в карточке.

<!--
# Binding form flow input fields to bot entities
TODO: fill this out based on design/ implementation -->

<!-- ## Adaptive Card schema

You can learn more about adaptive cards here  TODO: Insert link to adaptive cards schema documentation -->

## <a name="sample-adaptive-card-payload"></a>Прием полезных данных адаптивной карточки

Приведенный ниже код JSON показывает полезные данные адаптивной карточки.

```json
{
    "$schema": "https://microsoft.github.io/AdaptiveCards/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.0",
    "body": [
        {
            "speak": "<s>Serious Pie is a Pizza restaurant which is rated 9.3 by customers.</s>",
            "type": "ColumnSet",
            "columns": [
                {
                    "type": "Column",
                    "size": "2",
                    "items": [
                        {
                            "type": "TextBlock",
                            "text": "[Greeting], [TimeOfDayTemplate], You can eat in {location}",
                            "weight": "bolder",
                            "size": "extraLarge"
                        },
                        {
                            "type": "TextBlock",
                            "text": "9.3 · $$ · Pizza",
                            "isSubtle": true
                        },
                        {
                            "type": "TextBlock",
                            "text": "[builtin.feedback.display]",
                            "wrap": true
                        }
                    ]
                },
                {
                    "type": "Column",
                    "size": "1",
                    "items": [
                        {
                            "type": "Image",
                            "url": "http://res.cloudinary.com/sagacity/image/upload/c_crop,h_670,w_635,x_0,y_0/c_scale,w_640/v1397425743/Untitled-4_lviznp.jpg",
                            "size":"auto"
                        }
                    ]
                }
            ]
        }
    ],
    "actions": [
        {
            "type": "Action.Http",
            "method": "POST",
            "title": "More Info",
            "url": "http://foo.com"
        },
        {
            "type": "Action.Http",
            "method": "POST",
            "title": "View on Foursquare",
            "url": "http://foo.com"
        }
    ]
}
```

## <a name="next-step"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Подключение к каналам](conversation-designer-deploy.md)

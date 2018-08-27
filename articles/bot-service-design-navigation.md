---
title: Проектирование навигации для бота | Документация Майкрософт
description: Узнайте, как разработать хорошую навигационную структуру для бота и как избежать наиболее распространенных ошибок при проектировании навигации.
keywords: navigation, overview, stubborn bot, clueless bot, mysterious bot, captain obvious bot, bot that can't forget
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 223a822a1a309b8d89f0554eed241a4ae376ea23
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305086"
---
# <a name="design-bot-navigation"></a>Разработка навигации для бота

Пользователи могут перемещаться по веб-сайтам с помощью строки навигации, в приложениях с помощью меню и в веб-браузерах с помощью кнопок, таких как **Вперед** и **Назад**. Однако ни одна из этих широко известных навигационных методик полностью не учитывает требования к навигации в рамках бота. Как уже говорилось [ранее](~/bot-service-design-conversation-flow.md#handle-interruptions), пользователи часто взаимодействуют с ботами нелинейным способом, что затрудняет разработку навигации для бота, которая стабильно обеспечивает удобство работы для пользователя. 

Рассмотрим следующие дилеммы.

- Как гарантировать, что пользователь не потеряется в диалоге с ботом? 
- Может ли пользователь переходить "назад" в диалоге с ботом? 
- Как пользователю переходить к "главному меню" на протяжении беседы с ботом? 
- Как пользователю "отменить" операцию на протяжении беседы с ботом? 

Особенности проектирования навигации для вашего бота будут во многом зависеть от возможностей и функций, которые поддерживает ваш бот. Независимо от типа разрабатываемого бота вы захотите избежать распространенных ошибок плохо разработанных диалоговых интерфейсов. В этой статье описываются эти ошибки с точки зрения пяти персонажей: "упрямого бота", "невежественного бота", "таинственного бота", "бота Капитан Очевидность" и "бота, который не может забыть". 

> [!TIP]
> Ограничение каждого персонажа для вашего бота часто можно выполнить с помощью корректной [обработки пользовательских прерываний](v4sdk/bot-builder-howto-handle-user-interrupt.md).

## <a name="the-stubborn-bot"></a>"Упрямый бот"

"Упрямый бот" настаивает на поддержании текущего курса беседы, даже когда пользователь пытается направить разговор в другое русло. 

Рассмотрим следующие сценарии. 

![бот](~/media/bot-service-design-navigation/stubborn-bot-new.png)

Пользователи часто меняют свое мнение, решают отменить или иногда хотят начать все заново. 

> [!TIP]
> <b>Рекомендовано</b>. Создавать бот, учитывая, что пользователь может изменить ход разговора в любое время. 
>
> <b>Не рекомендовано</b>. Создавать бот, который игнорирует ввод данных пользователем и повторяет тот же вопрос в бесконечном цикле. 

Есть много способов избежать этой ошибки, но, возможно, самый простой способ не дать боту задавать один и тот же вопрос бесконечно, просто указав максимальное количество повторных попыток для каждого вопроса. Если он разработан таким образом, бот не делает ничего "умного", чтобы понять введенные пользователем данные и отвечать соответствующим образом, но, по крайней мере, он не задает один и тот же вопрос бесконечно. 

## <a name="the-clueless-bot"></a>"Невежественный бот"

"Невежественный бот" отвечает бессмысленно, когда не понимает попытки пользователя получить доступ к определенной функции. Пользователь может попытаться использовать ключевые команды, такие как "справка" или "отмена", ожидая, что бот будет реагировать соответствующим образом.

Рассмотрим следующие сценарии. 

![бот](~/media/bot-service-design-navigation/clueless-bot.png)

Хотя у вас может возникнуть соблазн спроектировать каждый диалог внутри вашего бота для прослушивания и соответствующего ответа на определенные ключевые слова, этот подход не рекомендуется. 

> [!TIP]
> <b>Рекомендовано</b>. Реализовать [ПО промежуточного слоя](v4sdk/bot-builder-create-middleware.md), которое анализирует введенные пользователем ключевые слова, например "справка", "отмена" или "начать заново", и реагирует соответствующим образом. 
> 
> <b>Не рекомендовано</b>. Разрабатывать каждый диалог, чтобы проверять введенные пользователем данные на ключевые слова. 

Определив логику в **ПО промежуточного слоя**, вы делаете его доступным для каждого обмена с пользователем. При таком подходе отдельные диалоги и запросы можно спроектировать таким образом, чтобы пропускать ключевые слова, если нужно.

## <a name="the-mysterious-bot"></a>"Таинственный бот"

"Таинственный бот" не может немедленно подтвердить входные пользовательские данные любым способом. 

Рассмотрим следующие сценарии. 

![бот](~/media/bot-service-design-navigation/mysterious-bot.png)

В некоторых случаях эта ситуация может указывать на сбои в работе бота. Однако возможно, что бот просто занят обработкой входящих пользовательских данных и еще не завершил составление своего ответа. 

> [!TIP]
> <b>Рекомендовано</b>. Спроектировать бот, который бы немедленно подтверждал введенные данные, даже в случаях, когда для составления ответа боту нужно немного времени. 
> 
> <b>Не рекомендовано</b>. Проектировать бот, который бы откладывал подтверждение введенных данных до тех пор, пока он не завершит составление ответа.

Немедленно подтверждая введенные данные, вы устраняете любую возможную путаницу в отношении состояния бота. Если составление ответа занимает много времени, рассмотрите возможность отправки сообщения "печатает", чтобы указать, что ваш бот работает, и последующей отправки [упреждающего сообщения](v4sdk/bot-builder-howto-proactive-message.md)

## <a name="the-captain-obvious-bot"></a>"Бот Капитан Очевидность"

"Бот Капитан Очевидность" предоставляет незапрашиваемые сведения, которые совершенно очевидны и поэтому бесполезны для пользователя. 

Рассмотрим следующие сценарии.

![бот](~/media/bot-service-design-navigation/captainobvious-bot.png)

> [!TIP]
> <b>Рекомендовано</b>. Спроектировать бот для предоставления информации, которая будет полезна для пользователя. 
> 
> <b>Не рекомендовано</b>. Спроектировать бот для предоставления совершенно очевидной информации, которая будет бесполезна для пользователя.

Создав бот для получения полезных сведений, вы увеличиваете вероятность взаимодействия пользователя с ботом.

## <a name="the-bot-that-cant-forget"></a>"Бот, который не может забыть"

"Бот, который не может забыть" ошибочно интегрирует информацию из прошлых диалогов с текущей беседой. 

Рассмотрим следующие сценарии.

![бот](~/media/bot-service-design-navigation/rememberall-bot.png)

> [!TIP]
> <b>Рекомендовано</b>. Спроектировать бот, который бы поддерживал текущую тему разговора, пока пользователь не выразит желание вернуться к предыдущей теме. 
> 
> <b>Не рекомендовано</b>. Спроектировать бот, который бы вставлял информацию из прошлых бесед тогда, когда это не относится к текущему разговору.

Поддерживая текущую тему разговора, вы уменьшаете вероятность путаницы и недовольства, а также увеличиваете вероятность того, что пользователь будет продолжать взаимодействовать с вашим ботом.

## <a name="next-steps"></a>Дополнительная информация

При проектировании бота, который избегает этих распространенных ошибок плохо разработанных диалоговых интерфейсов, вы делаете важный шаг на пути к обеспечению отличного пользовательского интерфейса. 

Далее, узнайте больше об [элементах пользовательского интерфейса](~/bot-service-design-user-experience.md), которые боты используют чаще всего для обмена данными с пользователями. 
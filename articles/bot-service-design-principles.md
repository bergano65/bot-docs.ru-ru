---
title: Принципы разработки ботов | Документация Майкрософт
description: Узнайте, что такое эффективный бот для общения и как планировать и разрабатывать ботов в соответствии со своими потребностями и для удовлетворения запросов пользователей.
keywords: best practices, bot design
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: d42df09fe364e04d85704c83b3489d7659cfc98f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301274"
---
# <a name="principles-of-bot-design"></a>Принципы разработки ботов

Bot Framework позволяет разработчикам создавать востребованные боты, которые решают различные бизнес-проблемы. После изучения основных понятий, описанных в этом разделе, вы будете иметь широкие возможности для разработки бота, который соответствует рекомендациям, и приобретенный опыт пригодится вам для продвижения в этой относительно новой области. 

## <a name="designing-a-bot"></a>Разработка бота

Если вы создаете бот, можно с уверенностью предположить, что вы ожидаете его востребованности у пользователей. Можно также предположить, что вы надеетесь на предпочтение вашего бота таким альтернативам, как приложения, веб-сайты, телефонные звонки и другие средства удовлетворения особых потребностей пользователей. Другими словами, бот конкурирует с приложениями и веб-сайтами за время пользователей. Итак, как вы можете повысить шанс того, что бот выполнит свою основную задачу — привлечение и удержание пользователей? Нужно всего лишь расставить приоритеты на правильные факторы при разработки бота.

## <a name="factors-that-do-not-guarantee-a-bots-success"></a>Факторы, которые не гарантируют успешность бота

При разработке бота следует учитывать, что ни один из следующих факторов не гарантирует его успешность: 

- **"Интеллектуальность" бота**. В большинстве случаев маловероятно, что чем интеллектуальнее ваш бот, тем больше вероятность получения популярности у пользователей или их перехода на вашу платформу. На самом деле многие боты не сильно продвинуты в машинном обучении или в возможностях естественного языка. Бот может обладать этими возможностями, если они необходимы для решения проблем, для которых его разрабатывали, однако не следует предполагать, что существует некая корреляция между интеллектуальными возможностями бота и переходом пользователей на взаимодействие с ним.

- **Число поддерживаемых естественных языков бота**. Ваш бот может отлично справляться с диалогами. Он может иметь огромный словарный запас и даже использовать отличные шутки. Но если он не решает проблемы пользователей, то эти возможности мало повлияют на успешность бота. На самом деле у некоторых ботов вообще отсутствует разговорная возможность. И во многих случаях это вполне нормально.

- **Голосовая связь**. Не всегда голосовые функции бота приведут к отличному взаимодействию с пользователями. Если пользователи вынуждены пользоваться голосовой связью, это может привести их к раздражению в процессе взаимодействия. При проектировании бота всегда думайте о том, подходит ли голосовая связь в качестве канала для решения конкретной задачи. Будет ли окружающая среда шумной? Будет ли голосовая связь передавать информацию для совместного использования с пользователем? 

## <a name="factors-that-do-influence-a-bots-success"></a>Факторы, действительно влияющие на успех бота

Наиболее успешные приложения или веб-сайты имеют по крайней мере одну общую вещь — отличное взаимодействие с пользователем. В этом плане боты ничем не отличаются. Таким образом обеспечение отличного взаимодействия с пользователем должно быть приоритетом номер один при разработке бота. Основные рекомендации:

- Решает ли бот с легкостью задачу пользователя при минимальном количестве действий?

- Решает ли бот задачу пользователя лучше, легче или быстрее, чем при взаимодействии с другими альтернативными решениями?

- Запускается ли бот на устройствах и платформах, которые интересуют пользователя?

- Доступен ли бот? Имеет ли бот интуитивно понятное взаимодействие с пользователями?

Ни один из этих вопросов прямо не связан с такими факторами, как интеллектуальность бота, числом поддерживаемых естественных языков, использованием машинного обучения или языком, использованным для написания бота. Пользователи вряд ли заинтересовались бы этими факторами, если бот решает необходимую проблему и обеспечивает отличное взаимодействие. Отличное взаимодействие бота с пользователем не требует от пользователя много печатать, слишком много говорить, повторять сказанное несколько раз или объяснять вещи, которые бот должен знать по умолчанию.

> [!TIP]
> Независимо от типа приложения, которое вы создаете (бот, веб-сайт или приложение), сделайте взаимодействие с пользователем главным приоритетом.

Процесс разработки бота подобен процессу разработки приложения или веб-сайта, поэтому опыт, накопленный десятилетиями создания пользовательского интерфейса и UX для приложений и веб-сайтов, по-прежнему актуален при проектировании ботов. 

Каждый раз, когда вы не уверены в правильности подхода к разработке бота, вернитесь на шаг назад и задайте себе следующий вопрос: "Как следовало бы решить эту проблему в приложении или на веб-сайте?" Скорее всего, тот же ответ применим и для разработки ботов. 

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знакомы с основными принципами разработки ботов, узнайте подробнее о проектировании взаимодействия с пользователем и распространенных шаблонах в оставшейся части этого раздела.
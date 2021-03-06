---
title: Перевод разговора с бота на человека — Служба Azure Bot
description: Узнайте, как поступать, когда пользователь начинает диалог с ботом, а затем его нужно перевести на человека.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 5/2/2019
ms.openlocfilehash: 83cb5b70313855db4fbd759867b905466727499a
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75792152"
---
# <a name="transition-conversations-from-bot-to-human"></a>Перевод разговора с бота на человека

Каким бы искусственным интеллектом ни обладал бот, иногда необходимо перевести разговор на человека. Бот должен понимать, когда это делать, и обеспечивать понятный и удобный переход.

## <a name="scenarios-that-require-human-involvement"></a>Сценарии, требующие участия человека

Участие человека может потребоваться в самых разных сценариях. Например, *сортировка*, *эскалация* и *контроль*. 

### <a name="triage"></a>Рассмотрение

Типичное обращение в службу поддержки начинается с основных вопросов, на которые может ответить бот. Бот первым отвечает на запросы от пользователей, поэтому может узнать имя, адрес, описание проблемы или другую важную информацию о пользователе, а затем передать разговор агенту. Когда бот сортирует запросы пользователей, агенты могут посвятить свое время решению проблемы, а не сбору информации.

### <a name="escalation"></a>Escalation (Эскалация);

Если вы используете бот в службе технической поддержки, он может не только собирать информацию, но и отвечать на основные вопросы и решать простые проблемы, например, помочь пользователю сбросить пароль. Но если в ходе разговора выясняется, что проблема пользователя сложная и требует участия человека, бот должен перевести пользователя на агента. Для реализации такого сценария бот должен отличать проблемы, которые он может решить самостоятельно, от тех, которые нужно передать человеку. Существует много способов, с помощью которых бот может определить, что диалог надо перевести на человека. 

#### <a name="user-driven-menus"></a>Управляемые пользователем меню

Пожалуй, самый простой способ решить этот вопрос — предоставить пользователю меню с различными вариантами. Задачи, которые бот может решать самостоятельно, располагаются в меню над ссылкой "Чат с агентом". Этот тип реализации не требует сложного машинного обучения или понимания естественного языка. Бот просто передает управление агенту-человеку, когда пользователь выбирает вариант "Чат с агентом". 

#### <a name="scenario-driven"></a>На основе сценария

Бот может решить, стоит ли переводить разговор, когда определит, способен ли он разобраться с проблемой самостоятельно. Бот собирает некоторые сведения о запросе пользователя, а затем запрашивает свой внутренний список возможностей, чтобы определить, может ли он ответить на этот запрос. Если бот определяет, что может, он это делает. Но если бот понимает, что запрос находится за пределами его возможностей, он переводит диалог на агента-человека.

#### <a name="natural-language"></a>Естественный язык

Понимание естественного языка и анализ тональности помогают боту определить, когда переводить диалог на человека. Это особенно важно при попытке определить, когда пользователь расстроен или хочет поговорить с агентом-человеком. 
 
Бот анализирует содержимое сообщений пользователя с помощью <a href="https://www.microsoft.com/cognitive-services/text-analytics-api" target="blank">API анализа текста</a> для определения тональности или с помощью <a href="https://www.luis.ai" target="_blank">API распознавания речи</a>. 


> [!TIP]
> Понимание естественного языка не всегда является лучшим способом определения ситуаций, когда диалог надо перевести на человека. Боты, как и люди, иногда ошибаются, и неправильный ответ расстроит пользователя. Если пользователь выбирает вариант из меню, бот всегда реагирует правильно. 

### <a name="supervision"></a>Контроль

В некоторых случаях агент-человек должен просто наблюдать за разговором и не вмешиваться.

Например, пользователь обращается в службу технической поддержки, и бот помогает ему диагностировать проблему с компьютером. Модель машинного обучения помогает боту определить наиболее вероятную причину проблемы, но, прежде чем предложить пользователю решение, бот может незаметно подтвердить диагноз и порядок действий и запросить разрешение у человека. Агент нажмет на кнопку, бот выдаст пользователю инструкции, проблема будет решена. Большую часть работы по-прежнему выполняет бот, но окончательное решение остается за агентом. 

## <a name="transitioning-control-of-the-conversation"></a>Перевод диалога 

Если бот решает перевести диалог на человека, он может сообщить об этом пользователю и поставить диалог в режим ожидания, пока не найдет свободного агента. 

Пока бот ожидает человека, он может автоматически отвечать на все сообщения от пользователя фразой по умолчанию, например: "Ваш запрос находится в очереди". Кроме того, бот может удалить диалог из очереди ожидания, если пользователь отправляет определенные сообщения, например: "Неважно" или "Отмени".

При разработке бота укажите, как будут назначаться агенты для ожидающих пользователей. Например, бот может реализовать простую систему обработки запросов по очередности. При более сложной логике бот будет назначать пользователей агентам по местоположению, языку или другим факторам. Бот также может предоставить агенту пользовательский интерфейс, где он может выбрать пользователя. Когда агент освобождается, он подключается к боту и присоединяется к диалогу.

> [!IMPORTANT]
> Даже после подключения агента бот не отключается и управляет диалогом. Пользователь и агент не общаются напрямую, а передают сообщения через бота. 

## <a name="routing-messages-between-user-and-agent"></a>Пересылка сообщений между пользователем и агентом

Когда агент подключается к боту, бот начинает передавать сообщения между пользователем и агентом. Хотя пользователю и агенту может казаться, что они общаются друг с другом напрямую, на самом деле они обмениваются сообщениями через бота. Бот получает сообщения от пользователя и отправляет их агенту и наоборот. 

> [!NOTE]
> В более сложных сценариях бот может выполнять больше задач, чем просто пересылать сообщения между пользователем и агентом. Например, бот может выбрать подходящий ответ и попросить агента подтвердить выбор.

## <a name="additional-resources"></a>Дополнительные ресурсы

::: moniker range="azure-bot-service-4.0"

- [Диалоги](v4sdk/bot-builder-dialog-manage-conversation-flow.md)
- <a href="https://www.microsoft.com/cognitive-services/text-analytics-api" target="blank">API анализа текста</a>

::: moniker-end

::: moniker range="azure-bot-service-3.0"

- [Управление потоком беседы с помощью диалогов (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [Управление потоком беседы с помощью диалогов (.Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)
- <a href="https://www.microsoft.com/cognitive-services/text-analytics-api" target="blank">API анализа текста</a>


::: moniker-end


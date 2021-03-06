---
title: Справочник по каналам
description: Справочник по каналам Bot Framework
keywords: channels reference, bot builder channels, bot framework channels
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/03/2019
ms.openlocfilehash: d8465a1cfa567790d62fc58ac29b6a452418f582
ms.sourcegitcommit: 772b9278d95e4b6dd4afccf4a9803f11a4b09e42
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2020
ms.locfileid: "80117564"
---
# <a name="categorized-activities-by-channel"></a>Разделенные на категории действия по каналам

В таблице описано, какие события (действия в сети) могут поступать из определенных каналов.

Пояснение к таблицам:

Символ              | Значение
:------------------:|:------------------------------------------------
:white_check_mark:  |Следует ожидать, что бот получит это действие
:x:                 |Следует ожидать, что бот **никогда** не получит это действие
:white_large_square:|Сейчас не определено, может ли бот получать это действие

Действия можно разделить на отдельные категории. Для каждой категории приведена таблица с возможными для нее действиями.

<a name="conversational"></a>Диалоговые
--------------

 \                      | Кортана            | Direct Line        | Direct Line (веб-чат) | Email              | Facebook           | GroupMe;            | Kik                | Команды              | Slack              | Skype   | Skype для бизнеса | Telegram | Twilio  
:---------------------- | :-----:            | :----------------: | :--------------------: |:----:              | :------:           | :-----:            | :-----:            | :---:              | :---:              | :---:   | :------------: | :------: | :----:  
Сообщение                 | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction         | :x:                | :x:                | :x:                    | :x:                | :x:                | :x:                | :x:                | :white_check_mark: | :x:                | :x:      | :x:             | :x:       | :x:      

- Все каналы передают действия сообщений.
- Если используется диалог, действия сообщений, как правило, должны передаваться диалогу.
- К действиям MessageReaction это, вероятно, не относится, хотя они являются важной частью общения.
- Существует два логических типа действий MessageReaction: добавленные и удаленные.


> [!TIP]
> Реакции на сообщения — это, например, знак одобрения в отношении предыдущего комментария. Они могут возникать без определенного порядка и в этом похожи на кнопки. Это действие сейчас отправляется каналом Teams.  


<a name="welcome"></a>Экран приветствия
-------

 \                         | Кортана            | Direct Line        | Direct Line (веб-чат) | Email   | Facebook             | GroupMe; | Kik     | Команды   | Slack   | Skype   | Skype для бизнеса | Telegram | Twilio  
:----------------------    | :-----:            | :---------:        | :--------------------: |:----:   | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
ConversationUpdate         | :white_check_mark: | :white_check_mark: | :white_check_mark:     | :x:     | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                | :x:                | :x:                    | :x:     | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     

- Каналы часто отправляют действия ConversationUpdate.
- Существует два логических типа действий MessageReaction: добавленные и удаленные.
- Идея, что поведение "Приветствие" можно без усилий реализовать с помощью подключения события ConversationUpdate.Added, кажется очень соблазнительной, — и иногда это работает.
- Но это очень поверхностный подход, и чтобы поведение "Приветствие" было достаточно надежным, реализации бота, возможно, также потребуется использовать состояние.


<a name="application-extensibility"></a>Расширяемость приложений
-------------------------

 \                      | Кортана            | Direct Line        | Direct Line (веб-чат) | Email   | Facebook | GroupMe; | Kik     | Команды   | Slack   | Skype   | Skype для бизнеса | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.*                    | :white_large_square: | :white_check_mark: | :white_check_mark:    | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square:  | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  

- Действия событий — это механизм расширяемости в Direct Line (_также называется веб-чатом_).
- Приложение, которому принадлежит клиент и сервер, может выбрать направлять собственные события через службу с помощь этого действия события.


<a name="microsoft-teams"></a>Microsoft Teams
------------------

 \                      | Кортана            | Direct Line        | Direct Line (веб-чат) | Email   | Facebook | GroupMe; | Kik     | Команды   | Slack   | Skype   | Skype для бизнеса | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Invoke.TeamsVerification   | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:      | :x:          | :x: | :x:   | :x:       | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     

- Наряду с некоторыми другими типизированными действиями, Microsoft Teams определяет несколько действий вызова для Teams.
- Действия вызова зависят от приложения и не определяются клиентом.
- Общее понятие об определенных подтипах действий, которые используют вызов, отсутствует.
- Вызов сейчас является единственным действием, которое запускает поведение "запрос — ответ" у бота.

Это очень важно: если для реализации запроса OAuth используются диалоги, действие Invoke.TeamsVerification нужно переадресовать диалогу.


<a name="message-update"></a>Обновление сообщений
--------------

 \                      | Кортана            | Direct Line        | Direct Line (веб-чат) | Email   | Facebook | GroupMe; | Kik     | Команды   | Slack   | Skype   | Skype для бизнеса | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
MessageUpdate | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete | :x:      | :x:          | :x:    | :x: | :x:      | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     

- Обновление сообщений сейчас поддерживается Teams.


<a name="oauth"></a>OAuth
-------

 \                      | Кортана            | Direct Line        | Direct Line (веб-чат) | Email   | Facebook | GroupMe; | Kik     | Команды   | Slack   | Skype   | Skype для бизнеса | Telegram | Twilio  
:---------------------- | :-----:            | :---------:        | :--------------------: |:----:   | :------: | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Event.TokenResponse| :white_large_square:  | :white_check_mark:   | :white_check_mark:    | :x:    | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 

Это очень важно: если для реализации запроса OAuth используются диалоги, действие Event.TokenResponse нужно переадресовать диалогу.


<a name="uncategorized"></a>Без категории 
-------------

 \                      | Кортана  | Direct Line        | Direct Line (веб-чат) | Email | Facebook | GroupMe; | Kik     | Команды | Slack | Skype | Skype для бизнеса | Telegram | Twilio  
:---------------------- | :-----:  | :---------:        | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :------------: | :------: | :----:  
EndOfConversation       | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate      | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Ввод с клавиатуры                  | :x:      | :white_check_mark: | :white_check_mark:     | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                 | :x:      | :x:                | :x:                    | :x:   | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     


<a name="out-of-use-includes-payment-specific-invoke"></a>Не используются (включают вызов для платежа)
---------------------------------------------
- DeleteUserData 
- Invoke.PaymentRequest  
- Invoke.Address
- Проверка связи

---

## <a name="summary-of-activities-supported-per-channel"></a>Сводка действий, поддерживаемых каждым каналом

<a name="cortana"></a>Кортана
-------
- Сообщение
- ConversationUpdate
- _Event.TokenResponse_
- _EndOfConversation (когда закрывается окно?)_

<a name="direct-line"></a>Direct Line
--------
- Сообщение
- ConversationUpdate
- Event.TokenResponse
- Event.*
- _Event.CreateConversation_
- _Event.ContinueConversation_

<a name="email"></a>Email
-----
- Сообщение

<a name="facebook"></a>Facebook
--------
- Сообщение
- _Event.TokenResponse_

<a name="groupme"></a>GroupMe;
-------
- Сообщение
- ConversationUpdate
- _Event.TokenResponse_

<a name="kik"></a>Kik
---
- Сообщение
- ConversationUpdate
- _Event.TokenResponse_

<a name="teams"></a>Команды
-----
- Сообщение
- ConversationUpdate
- MessageReaction
- MessageUpdate
- MessageDelete
- Invoke.TeamsVerification
- Invoke.ComposeResponse


<a name="slack"></a>Slack
-----
- Сообщение
- ConversationUpdate
- _Event.TokenResponse_


<a name="skype"></a>Skype
-----
- Сообщение
- ContactRelationUpdate
- _Event.TokenResponse_


<a name="skype-business"></a>Skype для бизнеса
--------------
- Сообщение
- ContactRelationUpdate 
- _Event.TokenResponse_


<a name="telegram"></a>Telegram
--------
- Сообщение
- ConversationUpdate
- _Event.TokenResponse_


<a name="twilio"></a>Twilio
------
- Сообщение

## <a name="summary-table-all-activities-to-all-channels"></a>Сводная таблица всех действий для всех каналов

 \                         | Кортана              | Direct Line          | Direct Line (веб-чат) | Email                | Facebook             | GroupMe; | Kik     | Команды   | Slack   | Skype   | Skype для бизнеса | Telegram | Twilio  
:----------------------    | :-----:              | :---------:          | :--------------------: |:----:                | :------:             | :-----: | :-----: | :---:   | :---:   | :---:   | :------------: | :------: | :----:  
Сообщение                    | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :white_check_mark:   | :white_check_mark:   | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark:        | :white_check_mark:  | :white_check_mark: 
MessageReaction            | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:      | :x:      | :x:             | :x:       | :x:      
ConversationUpdate         | :white_check_mark:   | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:      | :x:             | :white_check_mark:  | :x:     
ContactRelationUpdate      | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :x:      | :x:      | :white_check_mark: | :white_check_mark:        | :x:       | :x:     
Event.*                    | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.CreateConversation   | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Event.ContinueConversation | :white_large_square: | :white_large_square: | :white_large_square:   | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square: | :white_large_square:  | :white_large_square:  | :white_large_square:  | :white_large_square:           | :white_large_square:     | :white_large_square:  
Invoke.TeamsVerification   | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
Invoke.ComposeResponse     | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:      | :white_check_mark: | :x:    | :x:    | :x:             | :x:       | :x:     
MessageUpdate              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
MessageDelete              | :x:                  | :x:                  | :x:                    | :x:                  | :x:                  | :x:      | :x:  | :white_check_mark: | :white_large_square:  | :x:    | :x:             | :x:       | :x:     
Event.TokenResponse        | :white_large_square: | :white_check_mark:   | :white_check_mark:     | :x:                  | :white_large_square: | :white_large_square: | :white_large_square: | :x:    | :white_large_square: | :white_large_square: | :white_large_square:       | :white_large_square: | :white_large_square: 
EndOfConversation          | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
InstallationUpdate         | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Ввод с клавиатуры                     | :x:                  | :white_check_mark:   | :white_check_mark:     | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     
Handoff                    | :x:                  | :x:                  | :x:                    | :x:                  | :x:      | :x:     | :x:     | :x:   | :x:   | :x:   | :x:            | :x:      | :x:     

## <a name="web-chat"></a>Веб-чат. 
Веб-чат отправит:
- message: с параметрами text и (или) attachments.
- event: с параметрами name and value (в виде JSON/string).
- typing: если пользователь задал параметр, а именно sendTypingIndicator, веб-чат не будет отправлять действие contactRelationUpdate. Веб-чат также не поддерживает действие messageReaction, так как мы не получали явных запросов на реализацию поддержки этой функции.

По умолчанию веб-чат выводит следующее:
- message: отображает в режиме карусели или в столбце в зависимости от параметра действия.
- typing: отображает в течение 5 с и скрывает или отображает до поступления следующего действия.
- conversationUpdate: скрывает.
- event: скрывает.
- Другое: отобразит окно предупреждения (не используется в рабочей среде). Вы можете изменить этот конвейер обработки и добавить, удалить или заменить любую пользовательскую обработку.

Вы можете использовать веб-чат для отправки любого типа действий и полезной нагрузки, но мы не документируем и не рекомендуем использовать эту возможность. Вместо этого используйте действие event.

## <a name="action-support-by-channel"></a>Поддержка действий каждым из каналов

В следующей таблице указано максимальное число рекомендуемых действий и действий карточки, поддерживаемое в каждом канале.  Элемент :x: указывает, что действие совсем не поддерживается в конкретном канале.

| \                 | Кортана | Direct Line | Direct Line (веб-чат) | Email | Facebook | GroupMe; |   Kik   | график;  | Команды | Slack | Skype | Skype для бизнеса | Telegram | Twilio | 
| :---------------- | :-----: | :---------: | :--------------------: |:----: | :------: | :-----: | :-----: | :---: | :---: | :---: | :---: | :------------: | :------: | :----: |
| Рекомендуемые действия |   :x:   |     100     |          100           |  :x:  |    10    |   :x:   |   20    |  13   |  :x:  |  100  |  10   |      :x:       |    100   |   :x:  |  
| Действия карточки      |   100   |     100     |          100           |  :x:  |     3    |   :x:   |   20    |  99   |   3   |  100  |   3   |      :x:       |    :x:   |   :x:  |  

См. сведения о [числах, указанных в приведенной выше таблице](https://aka.ms/channelactions). 

Дополнительные сведения о _рекомендуемых действиях_ см. в статье [Использование кнопки для ввода данных](https://aka.ms/howto-add-buttons).

Дополнительные сведения о _действиях карточки_ см. в разделе [Отправка карточки для имиджевого баннера](https://aka.ms/howto-add-media#send-a-hero-card) в статье _Добавление мультимедиа в сообщения_.

## <a name="card-support-by-channel"></a>Поддержка карт каждым из каналов

| Channel | Адаптивная карта | Анимационная карта | Карта с аудио | Имиджевая карта | Карточка квитанции | Карточка для входа | Карточка с эскизом | Карта с видео |
|:-------:|:-------------:|:--------------:|:----------:|:---------:|:------------:|:-----------:|:--------------:|:----------:|
|Кортана|✔|❌|❌|❌|✔|✔|✔|❌|
|Email|🔶|🌐|🌐|✔|✔|✔|✔|🌐|
|Facebook|⚠🔶|✔|❌|✔|✔|✔|✔|❌|
|GroupMe;|🔶|🌐|🌐|🌐|🌐|🌐|🌐|🌐|
|Kik|🔶|✔|✔|❌|🌐|❌|✔|🌐|
|график;|⚠🔶|✔|🌐|✔|✔|✔|✔|🌐|
|Microsoft Teams|✔|❌|❌|✔|✔|✔|✔|❌|
|Skype|❌|✔|✔|✔|✔|✔|✔|✔|
|Slack|🔶|✔|🌐|🌐|✔|✔|🌐|🌐|
|Telegram|⚠🔶|✔|🌐|✔|✔|✔|✔|✔|
|Twilio|🔶|🌐|❌|🌐|🌐|🌐|🌐|❌|
|Веб-чат.|✔|✔|✔|✔|✔|✔|✔|✔|

*Примечание. Канал Direct Line технически поддерживает все карты, но клиент должен их реализовать.*

* ✔: Поддерживается — карта полностью поддерживается, но некоторые каналы поддерживают только ряд действий CardAction и (или) могут ограничивать количество действий, разрешенных для каждой карты.  Зависит от канала.
* ⚠: Частично поддерживается — карта может не отображаться, если содержит элементы для ввода данных и (или) кнопки. Зависит от канала.
* ❌: Поддержка отсутствует.
* 🔶: Карта преобразована в изображение.
* 🌐: Карта преобразована в неформатированный текст — могут не работать ссылки, не отображаться изображения и (или) не воспроизводиться элементы мультимедиа. Зависит от канала.

Эти категории намеренно описаны кратко и не объясняют подробно, как поддерживается каждая карта в каждом канале, из-за огромного разнообразия возможных сочетаний карт, функций и каналов. Используйте эту таблицу только для справки, но каждую карту в любом случае нужно тестировать во всех нужных каналах.

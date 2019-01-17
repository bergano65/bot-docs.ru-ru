---
ms.openlocfilehash: 4108f02a350d7e02d66db06cd7e65fa9c7a98e7d
ms.sourcegitcommit: 4db09039effc5259feba163128f4e9a3ccdd3d0d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52953629"
---
**Динамическое упреждающее сообщение** — это самый простой тип упреждающих сообщений.
Бот просто вставляет сообщение в диалог каждый раз, когда он активируется (вне зависимости от участия пользователя в отдельном разделе диалога с ботом в данный момент), и не пытается изменить диалог каким-либо способом.

Чтобы улучшить обработку уведомлений, рассмотрите другие варианты их интеграции в поток общения, например указав флаг в состоянии диалога или добавив уведомление в очередь.

<!--Snip
A **dialog-based proactive message** is more complex than an ad hoc proactive message. 
Before it can inject this type of proactive message into the conversation, 
the bot must identify the context of the existing conversation and decide how (or if)
it will resume that conversation after the message interrupts. 

For example, consider a bot that needs to initiate a survey at a given point in time. 
When that time arrives, the bot stops the existing conversation with the user and 
redirects the user to a `SurveyDialog`. 
The `SurveyDialog` is added to the top of the dialog stack and takes control of the conversation. 
When the user finishes all required tasks at the `SurveyDialog`, the `SurveyDialog` closes,
 returning control to the previous dialog, where the user can continue with the prior topic of conversation.

A dialog-based proactive message is more than just simple notification. 
In sending the notification, the bot changes the topic of the existing conversation. 
It then must decide whether to resume that conversation later, or to abandon that conversation altogether by resetting the dialog stack. 
/Snip-->
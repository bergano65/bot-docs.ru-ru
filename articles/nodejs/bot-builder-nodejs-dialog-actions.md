---
title: Обработка действий пользователя | Документация Майкрософт
description: Узнайте, как обрабатывать действия пользователя, позволив боту ожидать и обрабатывать введенные пользователем данные, содержащие определенные ключевые слова, с помощью пакета SDK Bot Framework для Node.js.
author: DucVo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5a0756be0a29660ef63f50a67ce4fa0f27ccc50f
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/26/2019
ms.locfileid: "67405043"
---
# <a name="handle-user-actions"></a>Обработка действий пользователя

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-global-handlers.md)
> - [Node.js](../nodejs/bot-builder-nodejs-dialog-actions.md)

Пользователи часто пытаются получить доступ к определенным функциям бота с помощью ключевых слов, например "help", "cancel" или "start over". Они нередко это делают в середине общения, когда бот ожидает другой ответ. Реализовав **действия**, вы можете спроектировать бот для правильной обработки таких запросов. Обработчики анализируют введенные пользователем ключевые слова, например "help", "cancel" или "start over", и реагируют соответствующим образом. 

![Взаимодействие пользователей](../media/designing-bots/capabilities/trigger-actions.png)

## <a name="action-types"></a>Типы действий

В следующей таблице перечислены типы действий, которые можно подключить к диалогу. С помощью ссылки для каждого имени действия можно перейти к разделу, в котором приводится более подробное описание этого действия.

| Действие | Область | ОПИСАНИЕ |
|------|------| ---- |
| [triggerAction](#bind-a-triggeraction) | Глобальные | Привязывает к диалогу действие, которое очистит стек диалогов и поместит диалог в нижнюю часть стека. Используйте параметр `onSelectAction`, чтобы переопределить это поведение по умолчанию. |
| [customAction](#bind-a-customaction) | Глобальные | Привязывает к боту настраиваемое действие, которое может обрабатывать сведения или выполнять действие, не влияя на стек диалогов. Используйте параметр `onSelectAction`, чтобы настроить функциональные возможности данного действия. |
[beginDialogAction](#bind-a-begindialogaction) | Определение по контексту | Привязывает к диалогу действие, которое при активации запускает другой диалог. Запускаемый диалог будет помещен в стек и удален из него после завершения. |
[reloadAction](#bind-a-reloadaction) | Определение по контексту | Привязывает к диалогу действие, которое при активации вызывает перезагрузку диалога. С помощью `reloadAction` можно обрабатывать такие высказывания пользователя, как "start over". |
[cancelAction](#bind-a-cancelaction) | Определение по контексту | Привязывает к диалогу действие, которое при активации отменяет диалог. С помощью `cancelAction` можно обрабатывать такие высказывания пользователя, как "cancel" или "nevermind". |
[endConversationAction](#bind-an-endconversationaction) | Определение по контексту | Привязывает к диалогу действие, которое при активации завершает общение с пользователем. С помощью `endConversationAction` можно обрабатывать такие высказывания пользователя, как "goodbye". |

## <a name="action-precedence"></a>Приоритет действий 

Получив высказывание пользователя, бот проверяет его на наличие всех зарегистрированных действий, которые находятся в стеке диалогов. Сопоставление начинается с верхней части стека до его нижней части. Если совпадений нет, то бот проверяет высказывание на наличие параметра `matches` всех глобальных действий.

Приоритет действий важен в случаях, когда одна и та же команда используется в разных контекстах. Например, можно использовать команду "Help" для отображения общей справки по боту. Можно также использовать команду "Help" для каждой из задач, но эти команды "Help" будут зависеть от контекста каждой задачи. Рабочий пример, в котором это подробно рассматривается, приведен в разделе [Ответ на пользовательский ввод](bot-builder-nodejs-dialog-manage-conversation-flow.md#respond-to-user-input).

## <a name="bind-actions-to-dialog"></a>Привязка действий к диалогу

Высказывания пользователя или нажатие кнопки может *активировать* действие, связанное с [диалогом](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html).
Если указаны *соответствующие команды*, действие будет ожидать, пока пользователь не произнесет слово или фразу, которая активирует действие.  Параметр `matches` позволяет указать регулярное выражение или имя [распознавателя][RecognizeIntent].
Чтобы привязать действие к нажатию кнопки, используйте [CardAction.dialogAction()][CardAction] для активации этого действия.

Действия можно объединить в *цепочку*, что позволяет привязать к диалогу сколько угодно действий.

### <a name="bind-a-triggeraction"></a>Привязка TriggerAction

Чтобы привязать [triggerAction][triggerAction] к диалогу, выполните следующие действия.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will clear the dialog stack and pushes
// the 'orderDinner' dialog onto the bottom of stack.
.triggerAction({
    matches: /^order dinner$/i
});
```

Привязка `triggerAction` к диалогу регистрирует его в боте. После активации `triggerAction` очистит стек диалогов и поместит активированный диалог в стек. Лучше всего это действие использовать, чтобы переключаться между [темами общения](bot-builder-nodejs-dialog-manage-conversation-flow.md#change-the-topic-of-conversation) или разрешить пользователям запрашивать произвольные изолированные задачи. Если требуется переопределить очистку стека диалогов этим действием, добавьте параметр `onSelectAction` в `triggerAction`.

Во фрагменте кода ниже показано, как предоставить общую справку из глобального контекста, не очищая стек диалогов.

```javascript
bot.dialog('help', function (session, args, next) {
    //Send a help message
    session.endDialog("Global help menu.");
})
// Once triggered, will start a new dialog as specified by
// the 'onSelectAction' option.
.triggerAction({
    matches: /^help$/i,
    onSelectAction: (session, args, next) => {
        // Add the help dialog to the top of the dialog stack 
        // (override the default behavior of replacing the stack)
        session.beginDialog(args.action, args);
    }
});
```

В этом случае `triggerAction` привязывается к самому диалогу `help` (в отличие от диалога `orderDinner`). Параметр `onSelectAction` позволяет запустить этот диалог без очистки стека диалогов. Это позволяет обрабатывать глобальные запросы, например "help", "about", "support" и т. д. Обратите внимание на то, что параметр `onSelectAction` явным образом вызывает метод `session.beginDialog` для запуска активированного диалога. Идентификатор активированного диалога предоставляется с помощью `args.action`. Не следует программировать идентификатор диалога (например: "help") в этом методе вручную, в противном случае могут возникнуть ошибки среды выполнения. Если вы хотите активировать контекстное справочное сообщение для самой задачи `orderDinner`, рассмотрите возможность привязки `beginDialogAction` к диалогу `orderDinner`.

### <a name="bind-a-customaction"></a>Привязка customAction

В отличие от других типов действий, для `customAction` не определено действие по умолчанию. Именно вы можете определить, что делает это действие. Преимущество использования `customAction` состоит в том, что вам предоставляется возможность обрабатывать запросы пользователей, не управляя стеком диалогов. Когда активируется `customAction`, параметр `onSelectAction` позволяет обработать запрос без размещения новых диалогов в стеке. После завершения действия управление возвращается диалогу вверху стека и бот может продолжить работу.

Можно использовать `customAction` для предоставления общих запросов и запросов быстрых действий, например "What is the temperature outside right now?" (Какая сейчас на улице температура?), "What time is it right now in Paris?" (Который сейчас час в Париже?), "Remind me to buy milk at 5pm today" (Напомни мне сегодня купить молока в 5 часов вечера) и т. д. Это общие действия, которые бот может выполнять, не оперируя стеком.

Еще одно важное отличие действия `customAction` заключается в том, что оно привязывается к боту, а не диалогу.

В примере кода ниже демонстрируется привязка `customAction` к боту `bot`, ожидающему передачи запросов для установки напоминания.

```javascript
bot.customAction({
    matches: /remind|reminder/gi,
    onSelectAction: (session, args, next) => {
        // Set reminder...
        session.send("Reminder is set.");
    }
})
```

### <a name="bind-a-begindialogaction"></a>Привязка beginDialogAction

При привязке `beginDialogAction` к диалогу это действие регистрируется в данном диалоге. При активации этот метод запустит другой диалог. Поведение этого действия аналогично вызову метода [beginDialog](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#begindialog). Новый диалог помещается в верх стека диалогов, поэтому текущая задача не завершается автоматически. Выполнение текущей задачи продолжится после завершения нового диалога. 

В следующем фрагменте кода показано, как привязать [beginDialogAction][beginDialogAction] к диалогу.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i
});

// Show dinner items in cart
bot.dialog('showDinnerCart', function(session){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    // End this dialog
    session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
});
```

В случаях, когда необходимо передать дополнительные аргументы в новый диалог, в действие можно добавить параметр [`dialogArgs`](https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs).

Можно изменить приведенный выше пример, чтобы принимать аргументы, переданные в `dialogArgs`.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will start the 'showDinnerCart' dialog.
// Then, the waterfall will resumed from the step that was interrupted.
.beginDialogAction('showCartAction', 'showDinnerCart', {
    matches: /^show cart$/i,
    dialogArgs: {
        showTotal: true;
    }
});

// Show dinner items in cart with the option to show total or not.
bot.dialog('showDinnerCart', function(session, args){
    for(var i = 1; i < session.conversationData.orders.length; i++){
        session.send(`You ordered: ${session.conversationData.orders[i].Description} for a total of $${session.conversationData.orders[i].Price}.`);
    }

    if(args && args.showTotal){
        // End this dialog with total.
        session.endDialog(`Your total is: $${session.conversationData.orders[0].Price}`);
    }
    else{
        session.endDialog(); // Ends without a message.
    }
});
```

### <a name="bind-a-reloadaction"></a>Привязка reloadAction

При привязке `reloadAction` к диалогу это действие регистрируется в данном диалоге. Привязка этого действия к диалогу вызывает перезапуск диалога, когда действие активируется. Активация этого действия аналогична вызову метода [replaceDialog](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#replacedialog). Оно удобно для реализации логики для обработки таких высказываний пользователя, как "start over", или для создания [циклов](bot-builder-nodejs-dialog-replace.md#repeat-an-action).

В следующем фрагменте кода показано, как привязать [reloadAction][reloadAction] к диалогу.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i
});
```

В случаях, когда необходимо передать дополнительные аргументы в перезагруженный диалог, в действие можно добавить параметр [`dialogArgs`](https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.idialogactionoptions#dialogargs). Этот параметр передается в параметр `args`. Если изменить код примера выше, чтобы получать аргумент при действии перезагрузки, он будет выглядеть следующим образом.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    function(session, args, next){
        if(args && args.isReloaded){
            // Reload action was triggered.
        }

        session.send("Lets order some dinner!");
        builder.Prompts.choice(session, "Dinner menu:", dinnerMenu);
    }
    //...other waterfall steps...
])
// Once triggered, will restart the dialog.
.reloadAction('startOver', 'Ok, starting over.', {
    matches: /^start over$/i,
    dialogArgs: {
        isReloaded: true;
    }
});
```

### <a name="bind-a-cancelaction"></a>Привязка cancelAction

При привязке `cancelAction` это действие регистрируется в диалоге. После активации это действие немедленно завершит диалог. После завершения диалога родительский диалог будет возобновлен с использованием возобновленного кода, указывающего, что он был в состоянии `canceled`. Это действие позволяет обрабатывать такие высказывания, как "nevermind" или "cancel". Если же нужно программно отменить диалог, обратитесь к разделу [Отмена диалога](bot-builder-nodejs-dialog-replace.md#cancel-a-dialog). Дополнительные сведения о *возобновленном коде* см. в разделе [Результаты запроса](bot-builder-nodejs-dialog-prompt.md#prompt-results). 

В следующем фрагменте кода показано, как привязать [cancelAction][cancelAction] к диалогу.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
//Once triggered, will end the dialog.
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i
});
```

### <a name="bind-an-endconversationaction"></a>Привязка endConversationAction

При привязке `endConversationAction` это действие регистрируется в диалоге. После активации это действие завершает общение с пользователем. Активация этого действия аналогична вызову метода [endConversation](https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html#endconversation). После завершения беседы пакет SDK Bot Framework для Node.js очищает стек диалогов и сохраняемые данные о состоянии. Дополнительные сведения о данных о сохраняемом состояния см. в статье [Управление данными состояния](bot-builder-nodejs-state.md).

В следующем фрагменте кода показано, как привязать [endConversationAction][endConversationAction] к диалогу.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Once triggered, will end the conversation.
.endConversationAction('endConversationAction', 'Ok, goodbye!', {
    matches: /^goodbye$/i
});
```

## <a name="confirm-interruptions"></a>Подтверждение прерываний

Большинство (если не все) этих действий прерывает обычный поток общения. Многие из них нарушают работу и требуют аккуратного обращения. Например, `triggerAction`, `cancelAction` или `endConversationAction` очистит стека диалогов. Если пользователь по ошибке активирует одно из этих действий, ему придется заново запустить задание. Чтобы убедиться, что пользователь действительно собирался вызвать эти действия, в них можно добавить параметр `confirmPrompt`. В `confirmPrompt` будет отображен запрос на подтверждение отмены или завершения текущей задачи пользователем. Это позволяет пользователю изменить решение и продолжить обработку.

Во фрагменте кода ниже показано действие [cancelAction][cancelAction] с запросом [confirmPrompt](http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions#confirmprompt), позволяющим убедиться, что пользователь действительно хочет отменить обработку заказа.

```javascript
// Order dinner.
bot.dialog('orderDinner', [
    //...waterfall steps...
])
// Confirm before triggering the action.
// Once triggered, will end the dialog. 
.cancelAction('cancelAction', 'Ok, cancel order.', {
    matches: /^nevermind$|^cancel$|^cancel.*order/i,
    confirmPrompt: "Are you sure?"
});
```

После активации этого действия бот спросит у пользователя "Are you sure?" Пользователю будет нужно ответить "Yes", чтобы выполнить действие, или "No", чтобы отменить действие и продолжить текущие операции.

## <a name="next-steps"></a>Дополнительная информация

**Действия** дают возможность ожидать запросы пользователей и корректно обрабатывать их с помощью бота. Многие из этих действий нарушают текущий поток передачи сообщений. Если вы хотите предоставить пользователям возможность переключиться из потока общения и возобновить его, перед переключением необходимо сохранить состояние пользователя. Давайте более подробно рассмотрим способы сохранения состояния пользователя и управление данными о состоянии.

> [!div class="nextstepaction"]
> [Управление данными о состоянии](bot-builder-nodejs-state.md)


[triggerAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[cancelAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#cancelaction

[reloadAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#reloadaction

[beginDialogAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#begindialogaction

[endConversationAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#endconversationaction

[RecognizeIntent]: bot-builder-nodejs-recognize-intent-messages.md

[CardAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.cardaction#dialogaction

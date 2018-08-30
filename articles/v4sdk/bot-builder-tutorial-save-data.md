---
title: Руководство. Сохранение данных состояния пользователя | Документация Майкрософт
description: Узнайте, как сохранить данные состояния пользователя в пакете SDK для Bot Builder.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 86f70fd66f1bc2261339cbe0590061913b51ddbc
ms.sourcegitcommit: 2dc75701b169d822c9499e393439161bc87639d2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42904442"
---
# <a name="save-user-state-data"></a>Сохранение данных состояния пользователя

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Когда бот просит пользователя ввести данные, может возникнуть необходимость сохранить часть информации в некой форме хранилища. Пакет SDK для Bot Builder позволяет хранить данные, вводимые пользователем, с помощью *размещенного в памяти хранилища*, *хранилища файлов* и таких хранилищ баз данных как *CosmosDB* или *SQL*. 

В этом руководстве показано, как определить объект хранилища в промежуточном слое и как сохранить в нем данные, вводимые пользователем.

## <a name="prequisite"></a>Предварительные требования 

Данное руководство основано на статье [Ask the user questions](bot-builder-tutorial-waterfall.md) (Вопросы, задаваемые пользователю).

## <a name="add-storage-to-middleware-layer"></a>Добавление хранилища к ПО промежуточного слоя


## <a name="save-user-input-to-storage"></a>Сохранение вводимых пользователем данных в хранилище

Чтобы управлять общением резервирования столика, необходимо определить **каскадный** диалог, который состоит из четырех шагов. В этом общении вы также будете использовать `DatetimePrompt` и `NumberPrompt` в дополнение к `TextPrompt`.

Диалоговое окно `reserveTable` должно следующим образом.

```javascript
// Reserve a table:
// Help the user to reserve a table

var reservationInfo = {
    dateTime: '',
    partySize: '',
    reserveName: ''
}

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");
        reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        reservationInfo.reserveName = result;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${reservationInfo.dateTime} 
            <br/>Party size: ${reservationInfo.partySize} 
            <br/>Reservation name: ${reservationInfo.reserveName}`;
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);

```

Теперь объект готов для подключения к логике бота.

## <a name="start-the-dialog"></a>Запуск диалогового окна

Метод `processActivity()`, используемый ботом, должен выглядеть следующим образом.

```javascript
// Listen for incoming activity 
server.post('/api/messages', (req, res) => {
    adapter.processActivity(req, res, async (context) => {
        if (context.activity.type === 'message') {
            // State will store all of your information 
            const state = conversationState.get(context);
            const dc = dialogs.createContext(context, state);

            if(context.activity.text.match(/hi/ig)){
                await dc.begin('greeting');
            }
            if(context.activity.text.match(/reserve table/ig)){
                await dc.begin('reserveTable');
            }
            else{
                // Continue executing the "current" dialog, if any.
                await dc.continue();
            }
        }
    });
});
```

Каждый раз, когда пользователь отправляет сообщение, содержащее строку `reserve table`, бот запускает общение `reserveTable`.

## <a name="next-steps"></a>Дополнительная информация

??? 

> [!div class="nextstepaction"]
> [Сохранение данных состояния пользователя](bot-builder-tutorial-save-data.md)

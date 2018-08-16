---
title: Хранение данных пользователя | Документация Майкрософт
description: Узнайте, как сохранить данные состояния пользователя в хранилище в пакете SDK для Bot Builder.
keywords: хранение данных пользователя, хранилище, данные общения
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 4/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 539e9e1cd772495d849ce106ee7d6a157fc1a9c0
ms.sourcegitcommit: 9a38d76afb0e82fdccc1f36f9b1a65042671e538
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/04/2018
ms.locfileid: "39515084"
---
# <a name="persist-user-data"></a>Хранение данных пользователя

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Когда бот попросит пользователя ввести данные, может возникнуть необходимость сохранить часть информации в некой форме в хранилище. С помощью *хранилища в памяти*, *хранилища файлов* или таких баз данных, как *CosmosDB* или *SQL*, пакет SDK для Bot Builder позволяет хранить данные, вводимые пользователем, а локальные типы хранилищ в основном используются для тестирования или создания прототипов, а более поздние типы хранилищ лучше всего подходят для ботов производства.

В этом руководстве показано, как определить объект хранилища и сохранить в нем данные, вводимые пользователем. 

> [!NOTE]
> Независимо от предпочитаемого типа хранилища, процессы подключения и сохранения данных одинаковы для всех типов. В качестве хранилища для хранения данных в этом руководстве используется `FileStorage`.
> Дополнительные сведения о состоянии и других типах хранилища см. в статье [Save state using conversation and user properties](bot-builder-howto-v4-state.md) (Сохранение состояния с помощью свойств общения и пользователя).

## <a name="prequisite"></a>Предварительные требования 

Основой для этого руководства послужила статья [Ask the user questions](bot-builder-tutorial-waterfall.md) (Вопросы, задаваемые пользователю). В этом руководстве будет создан бот, который запрашивает у пользователя сведения (три элемента) о бронировании столика в ресторане. Однако данные, введенные пользователем, не сохраняются. В этом руководстве боту будет добавлено постоянное хранилище данных.

## <a name="add-storage-to-middleware-layer"></a>Добавление хранилища к ПО промежуточного слоя

Пакет SDK Bot Builder версии 4 обрабатывает состояние и хранилище через ПО промежуточного слоя диспетчера состояний. ПО промежуточного слоя обеспечивает абстракцию, позволяющую вам получить доступ к свойствам с помощью простого хранилища ключей, независимо от типа базового хранилища. Диспетчер состояний отвечает за записи данных для хранения и управления параллелизмом, независимо от того, является ли базовым тип хранилища в памяти, хранилище файлов или Хранилище таблиц Azure. В этом руководстве `FileStorage` будет использоваться для сохранения данных, вводимых пользователем.

Поставщик `FileStorage` поставляется вместе с пакетом `bot-builder`. В начальном примере используется поставщик `MemoryStorage`. В этом типе хранилища используется непостоянная *память* бота, которая при его перезапуске очищается. С другой стороны, поведение библиотеки `FileStorage` похоже на поведение базы данных. Это значит, что библиотека записывает информацию о хранилище в файл, который расположен на локальном компьютере. Расположение файла хранилища можно изменить, что позволит вернуться к нему в будущем.

Чтобы использовать `FileStorage`, в боте необходимо найти заявление, в котором объект `conversationState` будет определен и обновлен, что позволит создать `new botbuilder.FileStorage("c:/temp")`. Кроме того, можно определить расположение, куда файл хранилища должен записываться. Таким образом, его можно легко найти и проверить, что было сохранено.

# <a name="ctabcstab"></a>[C#](#tab/cstab)
```cs
var storage = new FileStorage("c:/temp");

// These two classes are simply Dictionaries to store state
options.Middleware.Add(new ConversationState<MyBot.convoState>(storage));
options.Middleware.Add(new UserState<MyBot.userState>(storage));
```

Пакет SDK для Bot Builder с помощью различных областей обеспечивает три состояния объекта, из которых можно выбирать.

| Состояние | Область | ОПИСАНИЕ |
| ---- | ---- | ---- |
| `dc.ActiveDialog.State` | диалог | Состояние доступно для шагов диалога каскада. |
| `ConversationState` | общение | Состояние доступно для текущего общения. |
| `UserState` | user | Состояние доступно в нескольких сеансах общения. |

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

**app.js**
```javascript
// Storage
const storage = new FileStorage("c:/temp");
const convoState = new ConversationState(storage);
const userState  = new UserState(storage);
adapter.use(new BotStateSet(convoState, userState));
```

`BotStateSet` может одновременно управлять как `ConversationState`, так и `UserState`. Выбор можно сделать, когда настает момент сохранения пользовательских данных. Пакет SDK для Bot Builder с помощью различных областей обеспечивает три состояния объекта, из которых можно выбирать.

| Состояние | Область | ОПИСАНИЕ |
| ---- | ---- | ---- |
| `dc.activeDialog.state` | диалог | Состояние доступно для шагов диалога каскада. |
| `ConversationState` | общение | Состояние доступно для текущего общения. |
| `UserState` | user | Состояние доступно в нескольких сеансах общения. |

---
 

## <a name="persist-state"></a>Состояние сохранения

Бот может быть записан в любое из трех состояний расположения, в зависимости от того, что сохраняется и как долго оно должно храниться.  

# <a name="ctabcstab"></a>[C#](#tab/cstab)

*ПО промежуточного слоя диспетчера состояний* управляет состоянием записи в конце каждого шага. Поэтому, все, что нужно сделать в боте — это присвоить данные выбранному объекту состояния. В этом примере `dc.ActiveDialog.State` используется для отслеживания данных, которые были введены при бронировании столика. Таким образом, вместо сохранения пользовательских данных в глобальной переменной их можно хранить в диалоге временной области объекта состояния. Этот объект существует только до тех пор, пока диалог активен и если требуется хранить его дольше, он должен быть перенесен на другой объект состояния. В этом случае на последнем шаге каскада необходимо отметить процесс бронирования столика `msg`, как состояние общения.

```cs
dialogs.Add("reserveTable", new WaterfallStep[]
{
    async (dc, args, next) =>
    {
        // Prompt for the guest's name.
        await dc.Context.SendActivity("Welcome to the reservation service.");

        dc.ActiveDialog.State = new Dictionary<string, object>();

        await dc.Prompt("dateTimePrompt", "Please provide a reservation date and time.");
    },
    async(dc, args, next) =>
    {
        var dateTimeResult = ((DateTimeResult)args).Resolution.First();

        dc.ActiveDialog.State["date"] = Convert.ToDateTime(dateTimeResult.Value);
        
        // Ask for next info
        await dc.Prompt("partySizePrompt", "How many people are in your party?");

    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["partySize"] = (int)args["Value"];

        // Ask for next info
        await dc.Prompt("textPrompt", "Who's name will this be under?");
    },
    async(dc, args, next) =>
    {
        dc.ActiveDialog.State["name"] = args["Text"];
        string msg = "Reservation confirmed. Reservation details - " +
        $"\nDate/Time: {dc.ActiveDialog.State["date"].ToString()} " +
        $"\nParty size: {dc.ActiveDialog.State["partySize"].ToString()} " +
        $"\nReservation name: {dc.ActiveDialog.State["name"]}";

        var convo = ConversationState<convoState>.Get(dc.Context);

        // In production, you may want to store something more helpful
        convo[$"{dc.ActiveDialog.State["name"]} reservation"] = msg;

        await dc.Context.SendActivity(msg);
        await dc.End();
    }
});
```

# <a name="javascripttabjstab"></a>[JavaScript](#tab/jstab)

*ПО промежуточного слоя диспетчера состояний* управляет состоянием записи в файл в конце каждого шага. Поэтому, все, что нужно сделать в боте — это присвоить данные выбранному объекту состояния. В этом примере `dc.activeDialog.state` используется для отслеживания данных, которые были введены пользователем в объект `reservervationInfo`. Таким образом, вместо сохранения пользовательских данных в глобальной переменной их можно хранить в диалоге временной области объекта состояния. Поскольку этот объект существует только до тех пор, пока диалог активен, то если требуется его сохранить, он должен быть перенесен на другой объект состояния. В этом случае на последнем шаге каскада объекту `reservationInfo` требуется присвоить состояние `convo`.

```javascript
// Reserve a table:
// Help the user to reserve a table

dialogs.add('reserveTable', [
    async function(dc, args, next){
        await dc.context.sendActivity("Welcome to the reservation service.");

        dc.activeDialog.state.reservationInfo = {}; // Clears any previous data
        await dc.prompt('dateTimePrompt', "Please provide a reservation date and time.");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.dateTime = result[0].value;

        // Ask for next info
        await dc.prompt('partySizePrompt', "How many people are in your party?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.partySize = result;

        // Ask for next info
        await dc.prompt('textPrompt', "Who's name will this be under?");
    },
    async function(dc, result){
        dc.activeDialog.state.reservationInfo.reserveName = result;
        
        // Persist data
        var convo = conversationState.get(dc.context);; // conversationState.get(dc.context);
        convo.reservationInfo = dc.activeDialog.state.reservationInfo;

        // Confirm reservation
        var msg = `Reservation confirmed. Reservation details: 
            <br/>Date/Time: ${dc.activeDialog.state.reservationInfo.dateTime} 
            <br/>Party size: ${dc.activeDialog.state.reservationInfo.partySize} 
            <br/>Reservation name: ${dc.activeDialog.state.reservationInfo.reserveName}`;
            
        await dc.context.sendActivity(msg);
        await dc.end();
    }
]);
```

---

Теперь объект готов для подключения к логике бота.

## <a name="start-the-dialog"></a>Запуск диалогового окна

Выполнять изменения в коде не требуется. Просто запустите бот и отправьте сообщение, чтобы начать общение `reserveTable`.

## <a name="check-file-storage-content"></a>Проверка содержимого хранилища файлов

После запуска бота и окончания общения `reserveTable`, найдите сведения, сохраненные в файле в указанном расположении (например: "C:/temp"). К имени файла добавлен префикс "conversation!" или "user!" В данном случае может помочь сортировка файлов по дате.

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знаете, как сохранять данные, вводимые пользователем, можно более детально рассмотреть типы ввода, запрашиваемые у пользователя, с помощью библиотеки запросов.

> [!div class="nextstepaction"]
> [Запрос пользователю на ввод данных](~/v4sdk/bot-builder-prompts.md)

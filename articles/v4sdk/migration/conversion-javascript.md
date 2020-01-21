---
title: Перенос существующего бота JavaScript версии 3 в новый проект версии 4 — Служба Azure Bot
description: Узнайте, как перенести существующий бот JavaScript версии 3 в пакет SDK версии 4, используя новый проект.
keywords: JavaScript, bot migration, dialogs, v3 bot
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: f377aacbd809b79ecb0d558384a10da8eca5a772
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791086"
---
# <a name="migrate-a-javascript-v3-bot-to-a-v4-bot"></a>Перенос бота JavaScript версии 3 в бот версии 4

В этой статье описано, как перенести бота [core-MultiDialogs-v3](https://aka.ms/v3-js-core-multidialog-migration-sample) на основе пакета SDK для JavaScript версии 3 в новый бот JavaScript версии 4.
Это преобразование включает следующие этапы:

1. создание нового проекта и добавление зависимостей;
1. обновление точки входа и определение констант;
1. создание и повторная реализация диалогов с использованием пакета SDK версии 4;
1. обновление кода бота для вызова диалогов;
1. перенос вспомогательного файла **store.js**.

Завершив этот процесс, мы получим работающего бота версии 4. Копию преобразованного бота также можно получить из репозитория с примерами [core-MultiDialogs-v4](https://aka.ms/v4-js-core-multidialog-migration-sample).

Пакет SDK Bot Framework версии 4 использует тот же базовый REST API, что и пакет SDK версии 3. Но в пакете SDK версии 4 выполнен рефакторинг кода предыдущей версии пакета SDK для повышения гибкости и улучшения контроля над ботами. Основные изменения в пакете SDK:

- Управление состоянием осуществляется через соответствующие объекты и методы доступа к свойствам.
- Мы изменили способ обработки шагов, а имено то, как бот получает входящие действия из пользовательского канала и реагирует на них.
- В версии 4 вместо объекта `session` используется объект _контекста шага_, который содержит сведения о входящих действиях и позволяет отправлять пользователю ответные действия.
- Новая библиотека диалогов существенно отличается от той, которая была в версии 3. Вам потребуется преобразовать все старые диалоги в новую систему, используя компонентные и каскадные диалоги.

<!-- TODO
For more information about specific changes, see [differences between the v3 and v4 JavaScript SDK](???.md).
-->

> [!NOTE]
> Изменяя процесс миграции, мы также убрали часть кода. Но здесь мы расскажем только о тех изменениях, которые связаны с изменением логики по сравнению с версией 3.

## <a name="prerequisites"></a>предварительные требования

- Node.js
- Visual Studio Code
- [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

## <a name="about-this-bot"></a>Сведения о боте

Бот, который мы переносим, показывает, как использовать несколько диалогов для управления потоком беседы. Этот бот умеет искать сведения о рейсах и отелях.

- Диалог main спрашивает у пользователя, какие сведения ему нужны.
- Диалог отелей предлагает пользователю параметры для поиска и имитирует процесс поиска.
- Диалог рейсов создает сообщение об ошибке, которое бот перехватывает и правильно обрабатывает.

## <a name="create-and-open-a-new-v4-bot-project"></a>Создание и открытие нового проекта для бота версии 4

1. Для переноса кода бота вам потребуется проект версии 4. Чтобы создать такой проект на локальном компьютере, см. руководство по [созданию бота с помощью пакета SDK Bot Framework для JavaScript](../../javascript/bot-builder-javascript-quickstart.md).

    > [!TIP]
    > Вы также можете создать проект в Azure с помощью руководства по [созданию бота с помощью службы Azure Bot](../../bot-service-quickstart.md).
    > Созданные с помощью этих методов вспомогательные файлы немного отличаются. Для этой статьи мы создали локальный проект версии 4.

1. Откройте проект в Visual Studio Code.

## <a name="update-the-packagejson-file"></a>Обновление файла package.json

1. Добавьте зависимость от пакета **botbuilder-dialogs**, введя команду `npm i botbuilder-dialogs` в окне терминала Visual Studio Code.

1. Измените файл **./package.json** и обновите `name`, `version`, `description` и другие свойства.

## <a name="update-the-v4-app-entry-point"></a>Обновление точки входа приложения версии 4

Шаблон версии 4 создает файл **index.js** для точки входа приложения и файл **bot.js** для логики бота. На следующих шагах мы переименуем файл **bot.js** в **bots/reservationBot.js** и добавим в него класс для каждого диалога.

Измените файл **./index.js**, который является точкой входа для приложения бота. В нем сохранится часть кода из файла **app.js** версии 3, где настраивается сервер HTTP.

1. Кроме `BotFrameworkAdapter`, импортируйте `MemoryStorage` и `ConversationState` из пакета **botbuilder**. Также импортируйте модули диалогов bot и main. (Мы создадим их чуть позже, но здесь нам нужно указать ссылку на них.)

    ```javascript
    // Import required bot services.
    // See https://aka.ms/bot-services to learn more about the different parts of a bot.
    const { BotFrameworkAdapter, MemoryStorage, ConversationState } = require('botbuilder');

    // This bot's main dialog.
    const { MainDialog } = require('./dialogs/main')
    const { ReservationBot } = require('./bots/reservationBot');
    ```

1. Определите обработчик `onTurnError` для адаптера.

    ```javascript
    // Catch-all for errors.
    adapter.onTurnError = async (context, error) => {
        const errorMsg = error.message ? error.message : `Oops. Something went wrong!`;
        // This check writes out errors to console log .vs. app insights.
        console.error(`\n [onTurnError]: ${ error }`);
        // Clear out state
        await conversationState.delete(context);
        // Send a message to the user
        await context.sendActivity(errorMsg);
    };
    ```

    В версии 4 _адаптер бота_ перенаправляет в бот входящие действия. Этот адаптер позволяет перехватывать ошибки и реагировать на них до завершения шага. Мы очищаем состояние диалога, когда возникает ошибка, для сброса всех диалогов, чтобы бот не оставался в неправильном состоянии беседы.

1. Замените код шаблона для создаваемого бота следующим кодом.

    ```javascript
    // Define state store for your bot.
    const memoryStorage = new MemoryStorage();

    // Create conversation state with in-memory storage provider.
    const conversationState = new ConversationState(memoryStorage);

    // Create the base dialog and bot
    const dialog = new MainDialog();
    const reservationBot = new ReservationBot(conversationState, dialog);
    ```

    Уровень хранилища в памяти теперь предоставляется классом `MemoryStorage`, и нам нужно явным образом создать объект для управления состоянием беседы.

    Код определения диалога перемещен в класс `MainDialog`, который мы определим чуть ниже. Мы также перенесем код определения бота в класс `ReservationBot`.

1. И наконец, мы обновим обработчик запросов сервера, чтобы он использовал адаптер отправки действий боту.

    ```javascript
    // Listen for incoming requests.
    server.post('/api/messages', (req, res) => {
        adapter.processActivity(req, res, async (context) => {
            // Route incoming activities to the bot.
            await reservationBot.run(context);
        });
    });
    ```

    В версии 4 бот наследуется от `ActivityHandler`, где определен метод `run` для получения действия шага.

## <a name="add-a-constants-file"></a>Добавление файла констант

Создайте файл **./const.js**, в котором будут храниться идентификаторы для бота.

```javascript
module.exports = {
    MAIN_DIALOG: 'mainDialog',
    INITIAL_PROMPT: 'initialPrompt',
    HOTELS_DIALOG: 'hotelsDialog',
    INITIAL_HOTEL_PROMPT: 'initialHotelPrompt',
    CHECKIN_DATETIME_PROMPT: 'checkinTimePrompt',
    HOW_MANY_NIGHTS_PROMPT: 'howManyNightsPrompt',
    FLIGHTS_DIALOG: 'flightsDialog',
};
```

В версии 4 объектам диалогов и запросов присваиваются идентификаторы, по которым они затем вызываются.

## <a name="create-new-dialog-files"></a>Создание файлов диалогов

Создайте описанные ниже файлы.

| Имя файла | Description |
|:---|:---|
| **./dialogs/flights.js** | Здесь будет содержаться логика диалога `hotels`, перенесенная из прежней версии. |
| **./dialogs/hotels.js** | Здесь будет содержаться логика диалога `flights`, перенесенная из прежней версии. |
| **./dialogs/main.js** | Здесь будет содержаться логика бота, перенесенная из прежней версии. Этот файл заменяет старый диалог _root_. |

Мы не переносим диалог поддержки. Пример реализации диалога справки в версии 4 см. в руководстве по [обработке прерываний диалога пользователем](../bot-builder-howto-handle-user-interrupt.md?tabs=javascript).

### <a name="implement-the-main-dialog"></a>Реализация диалога main

В версии 3 все боты создавались поверх системы диалогов. В версии 4 логика бота и логика диалогов наконец разделены. То, что раньше составляло _корневой диалог_ бота версии 3, мы перенесли в класс `MainDialog` для замены.

Измените файл **./dialogs/main.js**.

1. Импортируйте классы и константы, которые потребуются для диалога.

    ```javascript
    const { DialogSet, DialogTurnStatus, ComponentDialog, WaterfallDialog,
        ChoicePrompt } = require('botbuilder-dialogs');
    const { FlightDialog } = require('./flights');
    const { HotelsDialog } = require('./hotels');
    const { MAIN_DIALOG,
        INITIAL_PROMPT,
        HOTELS_DIALOG,
        FLIGHTS_DIALOG
    } = require('../const');
    ```

1. Определите и экспортируйте класс `MainDialog`.

    ```javascript
    const initialId = 'mainWaterfallDialog';

    class MainDialog extends ComponentDialog {
        constructor() {
            super(MAIN_DIALOG);

            // Create a dialog set for the bot. It requires a DialogState accessor, with which
            // to retrieve the dialog state from the turn context.
            this.addDialog(new ChoicePrompt(INITIAL_PROMPT, this.validateNumberOfAttempts.bind(this)));
            this.addDialog(new FlightDialog(FLIGHTS_DIALOG));

            // Define the steps of the base waterfall dialog and add it to the set.
            this.addDialog(new WaterfallDialog(initialId, [
                this.promptForBaseChoice.bind(this),
                this.respondToBaseChoice.bind(this)
            ]));

            // Define the steps of the hotels waterfall dialog and add it to the set.
            this.addDialog(new HotelsDialog(HOTELS_DIALOG));

            this.initialDialogId = initialId;
        }
    }

    module.exports.MainDialog = MainDialog;
    ```

    Здесь объявляются другие диалоги и запросы, на которые напрямую ссылается диалог main.

    - Каскадный диалог main, который содержит шаги для этого диалога. При запуске компонентный диалог выполняет свой _начальный диалог_.
    - Запрос выбора, с помощью которого мы предложим пользователю выбрать нужную задачу. Мы создали запрос выбора с проверяющим элементом управления.
    - Два дочерних диалога для рейсов и отелей.

1. Добавьте в этот класс вспомогательный метод `run`.

    ```javascript
    /**
     * The run method handles the incoming activity (in the form of a TurnContext) and passes it through the dialog system.
     * If no dialog is active, it will start the default dialog.
     * @param {*} turnContext
     * @param {*} accessor
     */
    async run(turnContext, accessor) {
        const dialogSet = new DialogSet(accessor);
        dialogSet.add(this);

        const dialogContext = await dialogSet.createContext(turnContext);
        const results = await dialogContext.continueDialog();
        if (results.status === DialogTurnStatus.empty) {
            await dialogContext.beginDialog(this.id);
        }
    }
    ```

    В версии 4 бот при взаимодействии с системой диалогов сначала создает контекстный диалог, а затем вызывает `continueDialog`. Есть уже есть активный диалог, управление передается ему. В противном случае этот вызов просто завершается. Результат завершения `empty` обозначает, что активного диалога нет, и поэтому мы здесь снова запускаем диалог main.

    Параметр `accessor` передает свойство состояния диалога в метод доступа. В этом свойстве хранится состояние _стека диалогов_. Дополнительные сведения о том, как в версии 4 работают состояния и диалоги, см. в руководствах по [управлению состоянием](../bot-builder-concept-state.md) и [использованию библиотеки диалогов](../bot-builder-concept-dialog.md) соответственно.

1. Добавьте к классу каскадные шаги диалога main и проверяющий элемент для запроса выбора.

    ```javascript
    async promptForBaseChoice(stepContext) {
        return await stepContext.prompt(
            INITIAL_PROMPT, {
                prompt: 'Are you looking for a flight or a hotel?',
                choices: ['Hotel', 'Flight'],
                retryPrompt: 'Not a valid option'
            }
        );
    }

    async respondToBaseChoice(stepContext) {
        // Retrieve the user input.
        const answer = stepContext.result.value;
        if (!answer) {
            // exhausted attempts and no selection, start over
            await stepContext.context.sendActivity('Not a valid option. We\'ll restart the dialog ' +
                'so you can try again!');
            return await stepContext.endDialog();
        }
        if (answer === 'Hotel') {
            return await stepContext.beginDialog(HOTELS_DIALOG);
        }
        if (answer === 'Flight') {
            return await stepContext.beginDialog(FLIGHTS_DIALOG);
        }
        return await stepContext.endDialog();
    }

    async validateNumberOfAttempts(promptContext) {
        if (promptContext.attemptCount > 3) {
            // cancel everything
            await promptContext.context.sendActivity('Oops! Too many attempts :( But don\'t worry, I\'m ' +
                'handling that exception and you can try again!');
            return await promptContext.context.endDialog();
        }

        if (!promptContext.recognized.succeeded) {
            await promptContext.context.sendActivity(promptContext.options.retryPrompt);
            return false;
        }
        return true;
    }
    ```

    Первый шаг каскадного диалога предлагает пользователю сделать выбор. Для этого запускается запрос выбора, который также является диалогом. Второй шаг каскадного диалога обрабатывает результат, полученный от запроса выбора. Он может запустить дочерний диалог (если был сделан выбор) или завершить диалог main (если пользователь не смог сделать выбор).

    Запрос выбора возвращает выбранный пользователем вариант, если он считается допустимым, или повторно предлагает пользователю сделать выбор. Проверяющий элемент отслеживает, сколько раз подряд пользователю предлагается выбор, и после 3 неудачных попыток завершает запрос ошибкой и передает управление каскадному диалогу main.

### <a name="implement-the-flights-dialog"></a>Реализация диалога для рейсов

В боте версии 3 диалог рейсов был реализован как заглушка, которая демонстрирует обработку ботом сообщений об ошибке беседы. Здесь мы сохраним ту же логику.

Отредактируйте файл **./dialogs/flights.js**

```javascript
const { ComponentDialog, WaterfallDialog } = require('botbuilder-dialogs');

const initialId = 'flightsWaterfallDialog';

class FlightDialog extends ComponentDialog {
    constructor(id) {
        super(id);

        // ID of the child dialog that should be started anytime the component is started.
        this.initialDialogId = initialId;

        // Define the conversation flow using a waterfall model.
        this.addDialog(new WaterfallDialog(initialId, [
            async () => {
                throw new Error('Flights Dialog is not implemented and is instead ' +
                    'being used to show Bot error handling');
            }
        ]));
    }
}

exports.FlightDialog = FlightDialog;
```

### <a name="implement-the-hotels-dialog"></a>Реализация диалога отелей

Мы сохраним прежний поток логики для диалога отелей: запрос пункта назначения, запрос даты, запрос количества ночей и предложение списка вариантов, которые соответствуют параметрам поиска.

Отредактируйте файл **./dialogs/hotels.js**.

1. Импортируйте классы и константы, которые потребуются для диалога.

    ```javascript
    const { ComponentDialog, WaterfallDialog, TextPrompt, DateTimePrompt } = require('botbuilder-dialogs');
    const { AttachmentLayoutTypes, CardFactory } = require('botbuilder');
    const store = require('../store');
    const {
        INITIAL_HOTEL_PROMPT,
        CHECKIN_DATETIME_PROMPT,
        HOW_MANY_NIGHTS_PROMPT
    } = require('../const');
    ```

1. Определите и экспортируйте класс `HotelsDialog`.

    ```javascript
    const initialId = 'hotelsWaterfallDialog';

    class HotelsDialog extends ComponentDialog {
        constructor(id) {
            super(id);

            // ID of the child dialog that should be started anytime the component is started.
            this.initialDialogId = initialId;

            // Register dialogs
            this.addDialog(new TextPrompt(INITIAL_HOTEL_PROMPT));
            this.addDialog(new DateTimePrompt(CHECKIN_DATETIME_PROMPT));
            this.addDialog(new TextPrompt(HOW_MANY_NIGHTS_PROMPT));

            // Define the conversation flow using a waterfall model.
            this.addDialog(new WaterfallDialog(initialId, [
                this.destinationPromptStep.bind(this),
                this.destinationSearchStep.bind(this),
                this.checkinPromptStep.bind(this),
                this.checkinTimeSetStep.bind(this),
                this.stayDurationPromptStep.bind(this),
                this.stayDurationSetStep.bind(this),
                this.hotelSearchStep.bind(this)
            ]));
        }
    }

    exports.HotelsDialog = HotelsDialog;
    ```

1. Добавьте к этому классу несколько вспомогательных функций, которые мы применим в шагах диалога.

    ```javascript
    addDays(startDate, days) {
        const date = new Date(startDate);
        date.setDate(date.getDate() + days);
        return date;
    };

    createHotelHeroCard(hotel) {
        return CardFactory.heroCard(
            hotel.name,
            `${hotel.rating} stars. ${hotel.numberOfReviews} reviews. From ${hotel.priceStarting} per night.`,
            CardFactory.images([hotel.image]),
            CardFactory.actions([
                {
                    type: 'openUrl',
                    title: 'More details',
                    value: `https://www.bing.com/search?q=hotels+in+${encodeURIComponent(hotel.location)}`
                }
            ])
        );
    }
    ```

    `createHotelHeroCard` создает карту для имиджевого баннера с информацией об отеле.

1. Добавьте к этому классу каскадные шаги, которые используются в диалоге.

    ```javascript
    async destinationPromptStep(stepContext) {
        await stepContext.context.sendActivity('Welcome to the Hotels finder!');
        return await stepContext.prompt(
            INITIAL_HOTEL_PROMPT, {
                prompt: 'Please enter your destination'
            }
        );
    }

    async destinationSearchStep(stepContext) {
        const destination = stepContext.result;
        stepContext.values.destination = destination;
        await stepContext.context.sendActivity(`Looking for hotels in ${destination}`);
        return stepContext.next();
    }

    async checkinPromptStep(stepContext) {
        return await stepContext.prompt(
            CHECKIN_DATETIME_PROMPT, {
                prompt: 'When do you want to check in?'
            }
        );
    }

    async checkinTimeSetStep(stepContext) {
        const checkinTime = stepContext.result[0].value;
        stepContext.values.checkinTime = checkinTime;
        return stepContext.next();
    }

    async stayDurationPromptStep(stepContext) {
        return await stepContext.prompt(
            HOW_MANY_NIGHTS_PROMPT, {
                prompt: 'How many nights do you want to stay?'
            }
        );
    }

    async stayDurationSetStep(stepContext) {
        const numberOfNights = stepContext.result;
        stepContext.values.numberOfNights = parseInt(numberOfNights);
        return stepContext.next();
    }

    async hotelSearchStep(stepContext) {
        const destination = stepContext.values.destination;
        const checkIn = new Date(stepContext.values.checkinTime);
        const checkOut = this.addDays(checkIn, stepContext.values.numberOfNights);

        await stepContext.context.sendActivity(`Ok. Searching for Hotels in ${destination} from 
            ${checkIn.toDateString()} to ${checkOut.toDateString()}...`);
        const hotels = await store.searchHotels(destination, checkIn, checkOut);
        await stepContext.context.sendActivity(`I found in total ${hotels.length} hotels for your dates:`);

        const hotelHeroCards = hotels.map(this.createHotelHeroCard);

        await stepContext.context.sendActivity({
            attachments: hotelHeroCards,
            attachmentLayout: AttachmentLayoutTypes.Carousel
        });

        return await stepContext.endDialog();
    }
    ```

    Мы перенесли все шаги из диалога отелей версии 3 в диалог отелей версии 4.

## <a name="update-the-bot"></a>Обновление бота

В версии 4 бот может реагировать на действия за пределами системы диалогов. Класс `ActivityHandler` определяет обработчики для распространенных типов действий, чтобы упростить управление кодом.

Переименуйте файл **./bot.js** в **./bots/reservationBot.js** и измените его.

1. Этот файл импортирует `ActivityHandler`, который предоставляет базовую реализацию бота.

    ```javascript
    const { ActivityHandler } = require('botbuilder');
    ```

1. Переименуйте класс в `ReservationBot`.

    ```javascript
    class ReservationBot extends ActivityHandler {
        // ...
    }

    module.exports.ReservationBot = ReservationBot;
    ```

1. Обновите подпись конструктора для принятия получаемых объектов.

    ```javascript
    /**
     *
     * @param {ConversationState} conversationState
     * @param {Dialog} dialog
     * @param {any} logger object for logging events, defaults to console if none is provided
    */
    constructor(conversationState, dialog, logger) {
        super();
        // ...
    }
    ```

1. В конструкторе добавьте проверку параметров со значением NULL и определите свойства для конструктора класса.

    ```javascript
    if (!conversationState) throw new Error('[DialogBot]: Missing parameter. conversationState is required');
    if (!dialog) throw new Error('[DialogBot]: Missing parameter. dialog is required');
    if (!logger) {
        logger = console;
        logger.log('[DialogBot]: logger not passed in, defaulting to console');
    }

    this.conversationState = conversationState;
    this.dialog = dialog;
    this.logger = logger;
    this.dialogState = this.conversationState.createProperty('DialogState');
    ```

    Именно здесь мы создадим метод доступа к свойству состояния диалога, где будет храниться состояние для стека диалогов.

1. Обновите в конструкторе обработчик `onMessage` и добавьте обработчик `onDialog`.

    ```javascript
    this.onMessage(async (context, next) => {
        this.logger.log('Running dialog with Message Activity.');

        // Run the Dialog with the new message Activity.
        await this.dialog.run(context, this.dialogState);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });

    this.onDialog(async (context, next) => {
        // Save any state changes. The load happened during the execution of the Dialog.
        await this.conversationState.saveChanges(context, false);

        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler` перенаправляет действия сообщений в `onMessage`. Этот бот обрабатывает все введенные пользователем в диалоги данные.

    `ActivityHandler` вызывает `onDialog` в конце шага, прежде чем возвращать управление адаптеру. Нам нужно явным образом сохранить состояние перед выходом из шага. В противном случае изменения состояния не будут сохраняться, что нарушит работу диалога.

1. И наконец, обновите обработчик `onMembersAdded` в конструкторе.

    ```javascript
    this.onMembersAdded(async (context, next) => {
        const membersAdded = context.activity.membersAdded;
        for (let cnt = 0; cnt < membersAdded.length; ++cnt) {
            if (membersAdded[cnt].id !== context.activity.recipient.id) {
                await context.sendActivity('Hello and welcome to Contoso help desk bot.');
            }
        }
        // By calling next() you ensure that the next BotHandler is run.
        await next();
    });
    ```

    `ActivityHandler` вызывает `onMembersAdded` при получении действия обновления беседы, информируя о добавлении к беседе любого участника, кроме нашего бота. Мы обновим этот метод, чтобы он отправлял приветственное сообщение, когда пользователь присоединяется к беседе.

## <a name="create-the-store-file"></a>Создание файла для хранилища

Создайте файл **./store.js**, требуемый для диалога отелей. `searchHotels` имитирует работу функции поиска отелей, как и в старом боте версии 3.

```javascript
module.exports = {
    searchHotels: destination => {
        return new Promise(resolve => {

            // Filling the hotels results manually just for demo purposes
            const hotels = [];
            for (let i = 1; i <= 5; i++) {
                hotels.push({
                    name: `${destination} Hotel ${i}`,
                    location: destination,
                    rating: Math.ceil(Math.random() * 5),
                    numberOfReviews: Math.floor(Math.random() * 5000) + 1,
                    priceStarting: Math.floor(Math.random() * 450) + 80,
                    image: `https://placeholdit.imgix.net/~text?txtsize=35&txt=Hotel${i}&w=500&h=260`
                });
            }

            hotels.sort((a, b) => a.priceStarting - b.priceStarting);

            // complete promise with a timer to simulate async response
            setTimeout(() => { resolve(hotels); }, 1000);
        });
    }
};
```

## <a name="test-the-bot-in-the-emulator"></a>Тестирование бота в эмуляторе

На этом этапе мы можем запустить бота локально и подключить его к эмулятору.

1. Выполните этот пример на локальном компьютере.
    Если сеанс отладки запущен в Visual Studio Code, отладочные сведения отправляются в консоль отладки при любых действиях с ботом.
1. Запустите эмулятор и подключите его к боту.
1. Отправьте сообщения, чтобы протестировать все диалоги: диалог main, а также диалоги рейсов и отелей.

## <a name="additional-resources"></a>Дополнительные ресурсы

Тематические статьи по версии 4:

- [Принципы работы бота](../bot-builder-basics.md)
- [Управление состоянием](../bot-builder-concept-state.md)
- [Библиотека диалогов](../bot-builder-concept-dialog.md)

Пошаговые инструкции для версии 4:

- [Отправка и получение текстовых сообщений](../bot-builder-howto-send-messages.md)
- [Сохранение данных пользователя и диалога](../bot-builder-howto-v4-state.md)
- [Реализация процесса общения](../bot-builder-dialog-manage-conversation-flow.md)
- [Отладка с помощью эмулятора](../../bot-service-debug-emulator.md)
- [Добавление данных телеметрии в бот](../bot-builder-telemetry.md)

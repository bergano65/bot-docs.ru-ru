---
title: Добавление возможности распознавания естественного языка в функционал бота | Документация Майкрософт
description: Сведения об использовании LUIS для распознавания естественного языка с пакетом SDK Bot Framework.
keywords: Распознавание речи, LUIS, намерение, распознаватель, сущности, ПО промежуточного слоя
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: cedd6e57ec0490a18b23f22c7895e4ee52dc8509
ms.sourcegitcommit: e9cd857ee11945ef0b98a1ffb4792494dfaeb126
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/01/2019
ms.locfileid: "71694538"
---
# <a name="add-natural-language-understanding-to-your-bot"></a>Добавление возможности распознавания естественного языка в функционал бота

[!INCLUDE[applies-to](../includes/applies-to.md)]
Возможность понимать, что пользователь хочет сказать и какой вкладывает контекст, может быть сложной задачей, но также может способствовать более естественной беседе с ботом. API распознавания речи, так же называемое LUIS, позволяет делать так, чтобы бот мог распознавать намерения пользовательских сообщений, использовать более естественный язык пользователя и лучше направлять поток общения. В этом разделе рассматривается добавление LUIS в приложение для бронирования авиабилетов, чтобы распознавать намерения и сущности в введенных пользователем данных. 

## <a name="prerequisites"></a>Предварительные требования
- Учетная запись [LUIS](https://www.luis.ai).
- Код в этой статье основан на примере **Core Bot**. Вам потребуется копия этого примера на **[C#](https://aka.ms/cs-core-sample) или [JavaScript](https://aka.ms/js-core-sample)** . 
- Понимание [основных принципов работы ботов](bot-builder-basics.md), [обработки естественного языка](https://docs.microsoft.com/azure/cognitive-services/luis/what-is-luis) и [управления ресурсами бота](bot-file-basics.md).

## <a name="about-this-sample"></a>Об этом примере

Этот пример кода для простейшего бота реализует логику приложения для бронирования авиабилетов. С помощью службы LUIS он распознает пользовательский ввод и возвращает наиболее вероятное из обнаруженных LUIS намерений. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)
После каждой обработки введенных пользователем данных `DialogBot` сохраняет текущее состояние `UserState` и `ConversationState`. После сбора всех необходимых сведений в этом примере кода создается демонстрационное резервирование авиабилетов. В этой статье мы будем рассматривать те элементы примера, которые имеют отношение к LUIS. Но в целом поток логических действий в примере выглядит примерно так.

- Когда подключается новый пользователь, вызывается `OnMembersAddedAsync` и отображается приветственная карточка. 
- `OnMessageActivityAsync` вызывается для каждого полученного блока данных, введенных пользователем.

![Поток логических действий в примере LUIS](./media/how-to-luis/luis-logic-flow.png)

Модуль `OnMessageActivityAsync` запускает соответствующий диалог с помощью метода расширения диалога `Run`. Затем главный диалог вызывает вспомогательный метод LUIS для поиска самого вероятного намерения пользователя. Если таким намерением является BookFlight (бронирование авиабилетов), вспомогательный метод сохраняет полученные от LUIS данные пользователя. Затем он запускает `BookingDialog` для сбора требуемых дополнительных сведений от пользователя, например:

- `Origin` — город вылета;
- `TravelDate` — дата для бронирования авиабилетов;
- `Destination` — город прилета;

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)
После каждой обработки введенных пользователем данных `dialogBot` сохраняет текущее состояние `userState` и `conversationState`. После сбора всех необходимых сведений в этом примере кода создается демонстрационное резервирование авиабилетов. В этой статье мы будем рассматривать те элементы примера, которые имеют отношение к LUIS. Но в целом поток логических действий в примере выглядит примерно так.

- Когда подключается новый пользователь, вызывается `onMembersAdded` и отображается приветственная карточка. 
- `OnMessage` вызывается для каждого полученного блока данных, введенных пользователем.

![Поток логических действий в примере LUIS на JavaScript](./media/how-to-luis/luis-logic-flow-js.png)

Модуль `onMessage` выполняет `mainDialog` для сбора введенных пользователем данных.
Затем главный диалог вызывает `FlightBookingRecognizer` вспомогательного метода LUIS для поиска самого вероятного намерения пользователя. Если таким намерением является BookFlight (бронирование авиабилетов), вспомогательный метод сохраняет полученные от LUIS данные пользователя.
Получив ответ, `mainDialog` сохраняет для пользователя информацию, полученную от LUIS, и запускает `bookingDialog`. `bookingDialog` получает от пользователя дополнительные сведения по мере необходимости, например:

- `destination` — город прилета;
- `origin` — город вылета;
- `travelDate` — дата для бронирования авиабилетов.

---

Дополнительные сведения о других аспектах этого примера, в том числе о диалогах и состоянии, можно получить в статьях [Сбор данных, которые вводит пользователь, с помощью диалогового окна](bot-builder-prompts.md) или [Сохранение данных пользователя и диалога](bot-builder-howto-v4-state.md).

## <a name="create-a-luis-app-in-the-luis-portal"></a>Создание приложения LUIS на портале LUIS
Войдите на портал LUIS, чтобы создать собственную версию примера приложения LUIS. Создавать и администрировать приложения можно на странице **Мои приложения**.

1. Выберите **Import new app** (Импортировать новое приложение). 
1. Щелкните **Choose App file (JSON format)…** (Выберите файл приложения в формате JSON) 
1. Выберите файл `FlightBooking.json`, расположенный в папке `CognitiveModels` примера. В поле **Optional Name** (Необязательное имя) введите значение **FlightBooking**. Этот файл содержит три намерения: Book Flight, Cancel и None. Мы будем использовать эти намерения для распознавания желаний пользователя в полученном от него сообщении.
1. [Обучите](https://docs.microsoft.com/azure/cognitive-services/LUIS/luis-how-to-train) приложение.
1. [Опубликуйте](https://docs.microsoft.com/azure/cognitive-services/LUIS/publishapp) приложение в *рабочей* среде.

### <a name="why-use-entities"></a>Для чего нужны сущности
Сущности LUIS позволяют боту лучше понимать некоторые факты или события, которые отличаются от стандартных намерений. Это позволяет получить от пользователя дополнительную информацию, которая позволит боту точнее реагировать на действия пользователя или пропустить некоторые вопросы, в которых от пользователя запрашивается уже полученная информация. В файле FlightBooking.json определены не только три намерения LUIS (Book Flight, Cancel и None), но и набор дополнительных сущностей, таких как From.Airport и To.Airport. Эти сущности позволяют LUIS обнаруживать и возвращать дополнительные сведения из данных, которые пользователь предоставляет при запросе нового бронирования.

См. подробнее об отображении сведений о сущности в результате LUIS в руководстве по [извлечению данных из речевых фрагментов с намерениями и сущностями](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-data-extraction).

## <a name="obtain-values-to-connect-to-your-luis-app"></a>Получение значений для подключения к приложению LUIS
После публикации приложения LUIS ваш бот сможет обратиться к нему. Для доступа к приложению LUIS из кода бота потребуется записать несколько значений. Нужные сведения можно получить с помощью портала LUIS.

### <a name="retrieve-application-information-from-the-luisai-portal"></a>Получение сведений о приложении на портале LUIS.ai
Файл параметров (`appsettings.json` или `.env`) используется для того, чтобы собрать в одном расположении ссылки на все службы. Полученные данные будут добавлены в этот файл в следующем разделе. 
1. Выберите опубликованное приложение LUIS на сайте [luis.ai](https://www.luis.ai).
1. Открыв опубликованное приложение LUIS, выберите в нем вкладку **MANAGE** (Управление). ![Управление приложением LUIS](./media/how-to-luis/manage-luis-app.png)
1. Выберите слева вкладку **Application Information** (Сведения о приложении) и сохраните значение из поля _Application ID_ (Идентификатор приложения) в параметр <YOUR_APP_ID>.
1. Выберите слева вкладку **Keys and Endpoints** (Ключи и конечные точки) и сохраните значение из поля _Authoring Key_ (Ключ разработки) в параметр <YOUR_AUTHORING_KEY>.
1. Прокрутите страницу вниз до конца и перенесите значение из поля _Region_ (Регион) в параметр <YOUR_REGION>.

### <a name="update-the-settings-file"></a>Обновление файла параметров

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Добавьте в файл `appsettings.json` необходимые сведения для доступа к приложению LUIS, включая идентификатор приложения, ключ разработки и регион. Все эти данные вы ранее сохранили из опубликованного приложения LUIS. Обратите внимание, что имя узла API должно иметь формат `<your region>.api.cognitive.microsoft.com`.

**appsetting.json**  
[!code-json[appsettings](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/appsettings.json?range=1-7)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Добавьте в файл `.env` необходимые сведения для доступа к приложению LUIS, включая идентификатор приложения, ключ разработки и регион. Все эти данные вы ранее сохранили из опубликованного приложения LUIS. Обратите внимание, что имя узла API должно иметь формат `<your region>.api.cognitive.microsoft.com`.

Файл с расширением **.env**  
[!code[env](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/.env?range=1-5)]

---

## <a name="configure-your-bot-to-use-your-luis-app"></a>Настройка бота для работы с приложением LUIS

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Убедитесь, что для вашего проекта установлен пакет NuGet **Microsoft.Bot.Builder.AI.Luis**.

Для подключения к службе LUIS бот извлекает сведения, которые вы ранее добавили в файл appsetting.json. Класс `FlightBookingRecognizer` содержит код с параметрами из файла appsetting.json и отправляет запрос к службе LUIS, вызывая метод `RecognizeAsync`.

**FlightBookingRecognizer.cs**  

[!code-csharp[luisHelper](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/FlightBookingRecognizer.cs?range=12-39)]

`FlightBookingEx.cs` содержит логику для извлечения *From*, *To* и *TravelDate*. Это расширение разделяемого класса `FlightBooking.cs`, используемого для хранения результатов LUIS при вызове `FlightBookingRecognizer.RecognizeAsync<FlightBooking>` из `MainDialog.cs`.

**CognitiveModels\FlightBookingEx.cs**  

[!code-csharp[luis helper](~/../BotBuilder-Samples/samples/csharp_dotnetcore/13.core-bot/CognitiveModels/FlightBookingEx.cs?range=8-35)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Чтобы использовать LUIS, в проект следует установить пакет npm **botbuilder-ai**.

Для подключения к службе LUIS бот использует сведения, которые вы ранее добавили в файл `.env`. Класс `flightBookingRecognizer.js` содержит код, который импортирует параметры из файла `.env` и отправляет запрос к службе LUIS, вызывая метод `recognize()`.

**dialogs/flightBookingRecognizer.js**

[!code-javascript[luis helper](~/../BotBuilder-Samples/samples/javascript_nodejs/13.core-bot/dialogs/flightBookingRecognizer.js?range=6-64)]

Логика извлечения From, To и TravelDate реализуется в виде вспомогательных методов в `flightBookingRecognizer.js`. Эти методы используются после вызова `flightBookingRecognizer.executeLuisQuery()` из `mainDialog.js`

---

Теперь служба LUIS для бота полностью настроена и подключена.

## <a name="test-the-bot"></a>Тестирование бота

Скачайте и установите последнюю версию [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme).

1. Выполните этот пример на локальном компьютере. Если потребуются дополнительные инструкции, в файле README можно найти [пример кода на C#](https://aka.ms/cs-core-sample) или [пример кода на JS](https://aka.ms/js-core-sample).

1. Введите в эмуляторе сообщение, например "travel to paris" (лететь в Париж) или "going from paris to berlin" (направляюсь из Парижа в Берлин). Используйте все речевые фрагменты, включенные в файл FlightBooking.json, для обучения намерения Book flight (Бронирование авиабилетов).

![Входные данные LUIS для бронирования](./media/how-to-luis/luis-user-travel-input.png)

Если наиболее вероятное намерение LUIS соответствует элементу Book flight (Бронирование авиабилетов), бот будет задавать дополнительные вопросы, пока не получит достаточно сохраненных сведений для создания бронирования. После этого он возвращает всю собранную информацию о бронировании пользователю. 

![Результат LUIS для бронирования](./media/how-to-luis/luis-travel-result.png)

На этом этапе логика кода обнуляет состояние бота, и вы можете создать новое резервирование. 

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Использование QnA Maker для ответов на вопросы](./bot-builder-howto-qna.md)

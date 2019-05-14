---
title: Использование нескольких моделей LUIS и QnA | Документация Майкрософт
description: Сведения об использовании служб LUIS и QnA Maker в боте.
keywords: Luis, QnA, Dispatch tool, multiple services, route intents
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/15/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: b3e488615f318529935d35dbebbed2dd3b734f62
ms.sourcegitcommit: 3e3c9986b95532197e187b9cc562e6a1452cbd95
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/06/2019
ms.locfileid: "65039744"
---
# <a name="use-multiple-luis-and-qna-models"></a>Использование нескольких моделей LUIS и QnA

[!INCLUDE[applies-to](../includes/applies-to.md)]

Если бот использует несколько моделей LUIS и базы знаний QnA Maker, вы можете применить средство Dispatch, чтобы определить модель LUIS или базу знаний QnA Maker, которые лучше всего соответствуют вводимым пользователем данным. Для этого средство Dispatch создает одно приложение LUIS, чтобы отправить эти данные в соответствующую модель. См. подробнее о средстве Dispatch (включая команды CLI) в файле [README][dispatch-readme].

## <a name="prerequisites"></a>Предварительные требования
- [Базовые знания о ботах](bot-builder-basics.md), [LUIS][howto-luis] и [QnA Maker][howto-qna]. 
- [Средство подготовки к отправке](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
- Пример **NLP with Dispatch** из репозитория для [C#][cs-sample] или [JS][js-sample].
- Учетная запись [luis.ai](https://www.luis.ai/) для публикации приложений LUIS.
- Учетная запись [QnA Maker](https://www.qnamaker.ai/) для публикации базы знаний QnA.

## <a name="about-this-sample"></a>Об этом примере

В этом примере используется готовый набор приложений LUIS и QnA Maker.

## <a name="ctabcs"></a>[C#](#tab/cs)

![Поток логики кода из примера](./media/tutorial-dispatch/dispatch-logic-flow.png)

`OnMessageActivityAsync` вызывается для каждого полученного блока данных, введенных пользователем. Этот модуль обнаруживает намерения пользователя с наивысшими оценками и передает результат в `DispatchToTopIntentAsync`. DispatchToTopIntentAsync, в свою очередь, вызывает соответствующий обработчик приложения.

- `ProcessSampleQnAAsync` — для вопросов и ответов о боте.
- `ProcessWeatherAsync` —для запросов о погоде.
- `ProcessHomeAutomationAsync` — для команд домашнего освещения.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

![Поток логики кода из примера](./media/tutorial-dispatch/dispatch-logic-flow-js.png)

`onMessage` вызывается для каждого полученного блока данных, введенных пользователем. Этот модуль обнаруживает намерения пользователя с наивысшими оценками и передает результат в `dispatchToTopIntentAsync`. DispatchToTopIntentAsync, в свою очередь, вызывает соответствующий обработчик приложения.

- `processSampleQnA` — для вопросов и ответов о боте.
- `processWeather` —для запросов о погоде.
- `processHomeAutomation` — для команд домашнего освещения.

---

Обработчик вызывает службу LUIS или QnA Maker и возвращает полученный результат пользователю.

## <a name="create-luis-apps-and-qna-kb"></a>Создание приложений LUIS и базы данных QnA
Перед созданием модели отправки вам нужно создать и опубликовать приложения LUIS и базу знаний QnA. В этой статье мы опубликуем следующие модели, которые присутствуют в примере _NLP with Dispatch_ в папке `\CognitiveModels`: 

| ИМЯ | ОПИСАНИЕ |
|------|------|
| HomeAutomation | Приложение LUIS распознает намерение обращения к службе автоматизации и данные о сущностях.|
| Weather | Приложение LUIS распознает намерения, связанные с погодой и данными о расположении.|
| QnA Maker  | База данных QnA Maker предоставляет ответы на несколько простых вопросов о боте. |

### <a name="create-luis-apps"></a>Создание приложений LUIS
1. Войдите на [веб-портал LUIS](https://www.luis.ai/). В разделе _Мои приложения_ перейдите на вкладку _Import new app_ (Импорт нового приложения). Откроется следующее диалоговое окно:

![Импорт JSON-файла для LUIS](./media/tutorial-dispatch/import-new-luis-app.png)

2. Нажмите кнопку _Choose app file_ (Выбрать файл приложения), перейдите к папке CognitiveModel с примером кода и выберите файл HomeAutomation.json. Оставьте пустым поле Optional Name (Имя (необязательно)). 

3. Нажмите кнопку _Готово_.

4. Когда служба LUIS откроет выбранное приложение для автоматизации дома, нажмите кнопку _Train_ (Обучение). В результате этого будет выполнено обучение приложения по импортированному ранее набору высказываний с помощью файла home-automation.json.

5. После завершения обучения нажмите кнопку _Publish_ (Публикация). Откроется следующее диалоговое окно:

![Публикация приложения LUIS](./media/tutorial-dispatch/publish-luis-app.png)

6. Выберите среду Production (Рабочая) и нажмите кнопку _Publish_ (Публикация).

7. Когда новое приложение LUIS будет опубликовано, выберите вкладку _MANAGE_ (Управление). На странице Application Information (Сведения о приложении) укажите для `Application ID` и `Display name` значения _app-id-for-app_ и _name-of-app_ соответственно. На странице Key and Endpoints (Ключ и конечные точки) укажите для `Authoring Key` и `Region` значения _your-luis-authoring-key_ и _your-region_ соответственно. Эти значения будут потом использоваться в файле appsetting.json.

8. Завершив эти действия, _обучите_ и _опубликуйте_ приложение LUIS с прогнозом погоды и приложение LUIS для отправки, повторив описанные выше шаги с файлом weather.json.

### <a name="create-qna-maker-kb"></a>Создание базы данных QnA Maker

Первый шаг при настройке базы знаний службы QnA Maker — настройка службы QnA Maker в Azure. Чтобы сделать это, выполните пошаговые инструкции, приведенные на [этой странице](https://aka.ms/create-qna-maker). Теперь войдите на [веб-портал QnA Maker](https://qnamaker.ai). Перейдите к шагу 2.

![Создание QnA, шаг 2](./media/tutorial-dispatch/create-qna-step-2.png)

Выберите здесь следующие значения:
1. Учетную запись Azure AD.
1. Имя подписки Azure.
1. Имя, с которым вы создали службу QnA Maker. (Если нужная служба Azure QnA отсутствует в этом раскрывающемся списке, попробуйте обновить страницу.) 

Перейдите к шагу 3.

![Создание QnA, шаг 3](./media/tutorial-dispatch/create-qna-step-3.png)

Укажите имя для новой базы знаний QnA Maker. Для этого примера мы используем имя sample-qna.

Перейдите к шагу 4.

![Создание QnA, шаг 4](./media/tutorial-dispatch/create-qna-step-4.png)

Нажмите кнопку _+ Add File_ (+ Добавить файл), перейдите к папке CognitiveModel с примером кода и выберите файл QnAMaker.tsv.

Здесь есть дополнительный параметр, позволяющий добавить в базу знаний личность _Chit-chat_ (Беседа), но в нашем примере он не используется.

Перейдите к шагу 5.

Выберите _Create your KB_ (Создать базу знаний).

Когда база знаний будет создана из загруженного файла, выберите действие _Save and train_ (Сохранить и обучить), а когда оно будет выполнено, перейдите на вкладку _PUBLISH_ (Публикация) и опубликуйте приложение.

После публикации приложения QnA Maker откройте вкладку _SETTINGS_ (Параметры) и прокрутите эту страницу вниз до раздела Deployment Details (Сведения о развертывании). Запишите следующие значения из примера HTTP-запроса _Postman_.

```text
POST /knowledgebases/<knowledge-base-id>/generateAnswer
Host: <your-hostname>  // NOTE - this is a URL.
Authorization: EndpointKey <your-endpoint-key>
```

Полная строка URL-адреса для имени узла будет выглядеть так: https://< >.azure.net/qnamaker.

Эти значения будут потом использоваться в файле `appsettings.json` или `.env`.

Запишите имена и идентификаторы приложения LUIS и базы знаний QnA Maker. Также запишите ключ разработки LUIS и ключ подписки службы Cognitive Services. Эти сведения понадобятся вам для завершения процесса.

## <a name="create-the-dispatch-model"></a>Создание модели отправки

Интерфейс CLI для средства Dispatch создает модель для отправки в соответствующую службу.

1. Откройте командную строку или окно терминала и измените каталоги на каталог **CognitiveModels**.
1. Убедитесь, что у вас установлена текущая версия npm и средства Dispatch.

    ```cmd
    npm i -g npm
    npm i -g botdispatch
    ```

1. Используйте `dispatch init` для инициализации файла .dispatch для модели отправки. Присвойте файлу понятное имя.

    ```cmd
    dispatch init -n <filename-to-create> --luisAuthoringKey "<your-luis-authoring-key>" --luisAuthoringRegion <your-region>
    ```

1. Используйте `dispatch add` для добавления приложений LUIS и баз знаний QnA Maker в файл .dispatch.

    ```cmd
    dispatch add -t luis -i "<app-id-for-weather-app>" -n "<name-of-weather-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_Weather
    dispatch add -t luis -i "<app-id-for-home-automation-app>" -n "<name-of-home-automation-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_HomeAutomation
    dispatch add -t qna -i "<knowledge-base-id>" -n "<knowledge-base-name>" -k "<your-cognitive-services-subscription-id>" --intentName q_sample-qna
    ```

1. Используйте `dispatch create` для создания модели отправки из файла .dispatch.

    ```cmd
    dispatch create
    ```

1. Опубликуйте приложения LUIS для отправки, используя JSON-файл модели отправки.

## <a name="use-the-dispatch-model"></a>Использование модели отправки

Созданная модель определяет намерения для каждого приложения и каждой базы знаний, а также намерение _none_ при несоответствии речевого фрагмента.

- `l_HomeAutomation`
- `l_Weather`
- `None`
- `q_sample-qna`

Учтите, что эти службы нужно опубликовать с соответствующими именами, чтобы обеспечить правильную работу бота.

Бот получает доступ к опубликованным службам на основе этих сведений.

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="installing-packages"></a>Установка пакетов

Перед первым запуском приложения убедитесь, что установлены следующие пакеты NuGet:

**Microsoft.Bot.Builder**

**Microsoft.Bot.Builder.AI.Luis**

**Microsoft.Bot.Builder.AI.QnA**

### <a name="manually-update-your-appsettingsjson-file"></a>Обновление файла appsettings.json вручную

После создания всех приложений службы вам нужно добавить сведения о них в файл appsettings.json. Исходный пример кода [C#][cs-sample] содержит пустой файл appsettings.json:

**appsettings.json** [!code-json[AppSettings](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/AppSettings.json?range=8-17)]

Для каждой сущности ниже добавьте значения, которые вы записали ранее при выполнении этих инструкций:

**appsettings.json**
```json
"MicrosoftAppId": "",
"MicrosoftAppPassword": "",
  
"QnAKnowledgebaseId": "<knowledge-base-id>",
"QnAAuthKey": "<your-endpoint-key>",
"QnAEndpointHostName": "<your-hostname>",

"LuisAppId": "<app-id-for-dispatch-app>",
"LuisAPIKey": "<your-luis-authoring-key>",
"LuisAPIHostName": "<your-dispatch-app-region>",
```
Когда все изменения будут внесены, сохраните файл.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="installing-packages"></a>Установка пакетов

Перед первым запуском приложения убедитесь, что установлены следующие пакеты npm:

```powershell
npm install --save botbuilder
npm install --save botbuilder-ai
```
Чтобы использовать файл конфигурации .env, боту требуется дополнительный пакет:

```powershell
npm install --save dotenv
```

### <a name="manually-update-your-env-file"></a>Обновление файла .env вручную

После создания всех приложений службы вам нужно добавить сведения о них в файл .env. Исходный пример кода [JavaScript][js-sample] содержит пустой файл .env: 

**.env** [!code-file[EmptyEnv](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/.env?range=1-10)]

Добавьте сведения о подключении службы, как показано ниже:

Файл с расширением **.env**
```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<your-endpoint-key>"
QnAEndpointHostName="<your-hostname>"

LuisAppId=<app-id-for-dispatch-app>
LuisAPIKey=<your-luis-authoring-key>
LuisAPIHostName=<your-dispatch-app-region>

```
Когда все изменения будут внесены, сохраните файл.

---

### <a name="connect-to-the-services-from-your-bot"></a>Подключение к службам из бота

Чтобы подключиться к средству Dispatch, а также службам LUIS и QnA Maker, бот получает сведения из параметров, которые вы предоставили ранее.

## <a name="ctabcs"></a>[C#](#tab/cs)

В файле **BotServices.cs** сведения из файла конфигурации _appsettings.json_ используются для подключения бота отправки к службам `Dispatch` и `SampleQnA`. Конструкторы используют предоставленные значения для подключения к этим службам.

**BotServices.cs** [!code-csharp[ReadConfigurationInfo](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/BotServices.cs?range=14-30)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В файле **dispatchBot.js** сведения из файла конфигурации _.env_ используются для подключения бота отправки к службам _LuisRecognizer(dispatch)_ и _QnAMaker_. Конструкторы используют предоставленные значения для подключения к этим службам.

**dispatchBot.js** [!code-javascript[ReadConfigurationInfo](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=18-31)]

---

### <a name="call-the-services-from-your-bot"></a>Вызов служб из бота

Логика бота проверяет каждый блок вводимых пользователем данных с использованием объединенной модели отправки, находит первые возвращаемые намерения и использует эти сведения, чтобы вызвать соответствующую службу для входных данных.

## <a name="ctabcs"></a>[C#](#tab/cs)

В файле **DispatchBot.cs** при вызове метода `OnMessageActivityAsync` мы проверяем входящее сообщение пользователя с использованием модели отправки. Затем мы передаем `topIntent` и `recognizerResult` модели отправки в соответствующий метод, чтобы вызвать службу и получить результат.

**DispatchBot.cs** [!code-csharp[OnMessageActivity](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=26-36)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В методе **dispatchBot.js** `onMessage` мы проверяем входящее сообщения пользователя с использованием модели отправки, а затем находим и передаем _topIntent_ путем вызова  _dispatchToTopIntentAsync_.

**dispatchBot.js**

[!code-javascript[OnMessageActivity](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=37-50)]

---

### <a name="work-with-the-recognition-results"></a>Использование результатов распознавания

## <a name="ctabcs"></a>[C#](#tab/cs)

Когда модель возвращает результат, он позволяет понять, какие службы лучше всего подходят для обработки этого высказывания. Код нашего бота направляет запрос в соответствующую службу и обрабатывает полученный от нее ответ. Код использует _намерение_, которое возвращает средство Dispatch, чтобы выполнить перенаправление в соответствующую модель LUIS или службу QnA.

**DispatchBot.cs** [!code-csharp[DispatchToTop](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=51-69)]

Вызываемые методы `ProcessHomeAutomationAsync` или `ProcessWeatherAsync` передают результаты из модели отправки с использованием _luisResult.ConnectedServiceResult_. Затем указанный метод предоставляет отзыв пользователя, отображая первое намерение модели отправки, а также ранжированный список всех намерений и сущностей, которые были обнаружены.

Вызываемый метод `q_sample-qna` использует введенные пользователем данные, содержащиеся в turnContext, для создания ответа из базы знаний и отображения результата пользователю.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Когда модель возвращает результат, он позволяет понять, какие службы лучше всего подходят для обработки этого высказывания. Код в этом примере использует результат в _topIntent_, демонстрируя, как перенаправить запрос в соответствующую службу.

**DispatchBot.cs** [!code-javascript[DispatchToTop](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=67-83)]

Вызываемые методы `processHomeAutomation` или `processWeather` передают результаты из модели отправки с использованием _recognizerResult.luisResult_. Затем указанный метод предоставляет отзыв пользователя, отображая первое намерение модели отправки, а также ранжированный список всех намерений и сущностей, которые были обнаружены.

Вызываемый метод `q_sample-qna` использует введенные пользователем данные, содержащиеся в turnContext, для создания ответа из базы знаний и отображения результата пользователю.

---

> [!NOTE]
> Если используется рабочее приложение, выбранные методы LUIS подключаются к соответствующей службе, передают введенные пользователем данные и обрабатывают возвращаемые LUIS данные о намерениях и сущностях.

## <a name="test-your-bot"></a>Тестирование бота

В среде разработки откройте файл с пустым кодом. Запишите адрес localhost из адресной строки окна браузера, открытого приложением: https://localhost:<Port_Number>. Открыв Bot Framework Emulator, щелкните ссылку `create new bot configuration`, как показано ниже.

![Создание конфигурации](./media/tutorial-dispatch/emulator-create-new-configuration.png)

Введите записанный адрес localhost, добавив в конце /api/messages: https://localhost:<Port_Number>/api/messages.

![Подключение к Emulator](./media/tutorial-dispatch/emulator-create-and-connect.png)

Теперь щелкните `Save and connect`, чтобы получить доступ к запущенному боту. Ниже для справки приведены некоторые вопросы и команды, поддерживаемые в используемых для бота службах.

- QnA Maker
  - `hi`, `good morning`
  - `what are you`, `what do you do`
- LUIS (автоматизация дома)
  - `turn on bedroom light`
  - `turn off bedroom light`
  - `make some coffee`
- LUIS (погода)
  - `whats the weather in redmond washington`
  - `what's the forecast for london`
  - `show me the forecast for nebraska`

## <a name="additional-information"></a>Дополнительная информация

Когда бот будет запущен, вы можете повысить его производительность, удаляя похожие или перекрывающиеся высказывания. Для примера предположим, что в приложении LUIS `Home Automation` для домашней автоматизации запросы типа "turn on the lights" (Включить свет) сопоставляются с намерением TurnOnLights, а запросы типа "Why won't my lights turn on?" (Почему не включается свет?) сопоставляются с намерением None. Во втором случае запрос передается в службу QnA Maker. При объединении приложения LUIS и службы QnA Maker с помощью диспетчеризации необходимо выполнить следующие действия.

- Удалите намерение None из исходного приложения LUIS `Home Automation` и добавьте высказывания из удаленного намерения в намерение None в приложении диспетчеризации.
- Если вы сохраните намерение "None" в исходном приложении LUIS, в бот придется добавить логику для передачи в службу QnA Maker сообщений, которые соответствуют этому намерению.

Любой из этих методов позволяет сократить число сообщений "Couldn't find an answer" (Не удалось найти ответ), которые бот будет возвращать пользователям.

### <a name="to-update-or-create-a-new-luis-model"></a>Обновление или создание модели LUIS

этот пример основан на предварительно настроенной модели LUIS. Сведения, которые вам потребуются для обновления этой модели или создания новой модели LUIS, вы найдете [здесь](https://aka.ms/create-luis-model#updating-your-cognitive-models).

### <a name="to-delete-resources"></a>Удаление ресурсов

С помощью этого примера создается ряд приложений и ресурсов, которые можно удалить с помощью описанных ниже действий. Следите при этом за тем, чтобы не удалить ресурсы, от которых зависят *другие приложения или службы*.

Чтобы удалить ресурсы LUIS, сделайте следующее:

1. Войдите на портал [luis.ai](https://www.luis.ai).
1. Перейдите к странице _My Apps_ (Мои приложения).
1. Выберите приложения, созданные этим примером.
   - `Home Automation`
   - `Weather`
   - `NLP-With-Dispatch-BotDispatch`
1. Щелкните _Delete_ (Удалить), а затем _ОК_ для подтверждения.

Чтобы удалить ресурсы QnA Maker, сделайте следующее:

1. Войдите на портал [qnamaker.ai](https://www.qnamaker.ai/).
1. Перейдите к странице _Мои базы знаний_.
1. Нажмите кнопку "Удалить" для базы знаний `Sample QnA`, затем щелкните _Удалить_ для подтверждения.

### <a name="best-practice"></a>Рекомендации

Чтобы улучшить описанные здесь службы, изучите рекомендации по [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-concept-best-practices) и [QnA Maker](https://docs.microsoft.com/en-us/azure/cognitive-services/qnamaker/concepts/best-practices).


[howto-luis]: bot-builder-howto-v4-luis.md
[howto-qna]: bot-builder-howto-qna.md

[cs-sample]: https://aka.ms/dispatch-sample-cs
[js-sample]: https://aka.ms/dispatch-sample-js

[dispatch-readme]: https://aka.ms/botbuilder-tools-dispatch

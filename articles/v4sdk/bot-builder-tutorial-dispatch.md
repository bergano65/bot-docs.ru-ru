---
title: Использование нескольких моделей LUIS и QnA | Документация Майкрософт
description: Сведения об использовании служб LUIS и QnA Maker в боте.
keywords: Luis, QnA, Dispatch tool, multiple services, route intents
author: diberry
ms.author: diberry
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/22/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: c0e07dfa828854e44b2236aff2e1e17e60d69bfa
ms.sourcegitcommit: a547192effb705e4c7d82efc16f98068c5ba218b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 12/25/2019
ms.locfileid: "75491673"
---
# <a name="use-multiple-luis-and-qna-models"></a>Использование нескольких моделей LUIS и QnA

[!INCLUDE[applies-to](../includes/applies-to.md)]

Если бот использует несколько моделей LUIS и базы знаний QnA Maker, вы можете применить средство Dispatch, чтобы определить модель LUIS или базу знаний QnA Maker, которые лучше всего соответствуют вводимым пользователем данным. Для этого средство Dispatch создает одно приложение LUIS, чтобы отправить эти данные в соответствующую модель. Дополнительные сведения о средстве Dispatch (включая команды CLI) см. в [файле сведений][dispatch-readme].

## <a name="prerequisites"></a>предварительные требования

- [Базовые знания о ботах](bot-builder-basics.md), [LUIS][howto-luis] и [QnA Maker][howto-qna].
- [Средство подготовки к отправке](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch)
- Пример **NLP with Dispatch** из репозитория примеров кода на [C#][cs-sample], [JavaScript][js-sample] или [Python][python-sample].
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

## <a name="pythontabpython"></a>[Python](#tab/python)

![Поток логики кода из примера](./media/tutorial-dispatch/dispatch-logic-flow-python.png)

`on_message_activity` вызывается для каждого полученного блока данных, введенных пользователем. Этот модуль обнаруживает намерения пользователя с наивысшими оценками и передает результат в `_dispatch_to_top_intent`. _dispatch_to_top_intent, в свою очередь, вызывает соответствующий обработчик приложения.

- `_process_sample_qna` — для вопросов и ответов о боте.
- `_process_weather` —для запросов о погоде.
- `_process_home_automation` — для команд домашнего освещения.

---

Обработчик вызывает службу LUIS или QnA Maker и возвращает полученный результат пользователю.

## <a name="create-luis-apps-and-qna-knowledge-base"></a>Создание приложений LUIS и базы знаний QnA

Перед созданием модели отправки вам нужно создать и опубликовать приложения LUIS и базы знаний QnA. В этой статье мы опубликуем следующие модели, которые присутствуют в примере _NLP with Dispatch_ в папке `\CognitiveModels`:

| Имя | Description |
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

8. Завершив эти действия, _обучите_ и _опубликуйте_ приложение LUIS **домашней автоматизации** и приложение LUIS **прогноза погоды**, повторив описанные выше шаги с файлом weather.json.

### <a name="create-qna-maker-knowledge-base"></a>Создание базы знаний QnA Maker

Первым шагом при настройке базы знаний службы QnA Maker является настройка службы QnA Maker в Azure. Чтобы сделать это, см. [пошаговые инструкции](https://aka.ms/create-qna-maker).

Создав в Azure службу QnA Maker, запишите _ключ 1_ Cognitive Services, который предоставлен для вашей службы QnA Maker. Его нужно будет сохранить как \<azure-qna-service-key1> при добавлении приложения QnA Maker в приложение для отправки.

См. дополнительные сведения о [двух типах ключей](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure#types-of-keys-in-qna-maker), используемых с QnA Maker.

Чтобы получить этот ключ, сделайте следующее:

![Выбор Cognitive Services](./media/tutorial-dispatch/select-qna-cognitive-service.png)

1. На портале Azure выберите службу QnA Maker из Cognitive Services.

    ![Выбор ключей в Cognitive Services](./media/tutorial-dispatch/select-cognitive-service-keys.png)

1. Выберите ключи, которые расположены в разделе _управления ресурсами_ в меню слева.

    ![Выбор ключа 1 в Cognitive Services](./media/tutorial-dispatch/select-cognitive-service-key1.png)

1. Скопируйте значение _ключа 1_ в буфер обмена и сохраните его на локальном компьютере. Он будет потом использоваться в качестве значения ключа (-k) \<azure-qna-service-key1> при добавлении приложения QnA Maker в приложение для отправки.

1. Теперь войдите на [веб-портал QnA Maker](https://qnamaker.ai).

1. На шаге 2 выберите следующие значения:

    - Учетную запись Azure AD.
    - Имя подписки Azure.
    - Имя, с которым вы создали службу QnA Maker. (Если нужная служба Azure QnA отсутствует в этом раскрывающемся списке, попробуйте обновить страницу.)

    ![Создание QnA, шаг 2](./media/tutorial-dispatch/create-qna-step-2.png)

1. На шаге 3 укажите имя для новой базы знаний QnA Maker. В этом примере используйте имя sample-qna.

    ![Создание QnA, шаг 3](./media/tutorial-dispatch/create-qna-step-3.png)

1. На шаге 4 нажмите кнопку _+ Add File_ (+ Добавить файл), перейдите к папке CognitiveModel с примером кода и выберите файл QnAMaker.tsv. Здесь есть дополнительный параметр, позволяющий добавить в базу знаний личность _Chit-chat_ (Беседа), но в нашем примере он не используется.

    ![Создание QnA, шаг 4](./media/tutorial-dispatch/create-qna-step-4.png)

1. На шаге 5 выберите _Create your knowledge base_ (Создать базу знаний).

1. Когда база знаний будет создана из переданного файла, выберите действие _Save and train_ (Сохранить и обучить), а когда оно будет выполнено, перейдите на вкладку _PUBLISH_ (Публикация) и опубликуйте приложение.

1. После публикации приложения QnA Maker откройте вкладку _SETTINGS_ (Параметры) и прокрутите эту страницу вниз до раздела Deployment Details (Сведения о развертывании). Запишите следующие значения из примера HTTP-запроса _Postman_.

    ```text
    POST /knowledge bases/<knowledge-base-id>/generateAnswer
    Host: <your-hostname>  // NOTE - this is a URL.
    Authorization: EndpointKey <qna-maker-resource-key>
    ```

    Полная строка URL-адреса для имени узла будет выглядеть так: https://<имя_узла>.azure.net/qnamaker. Эти значения будут потом использоваться в файле `appsettings.json` или `.env`.

## <a name="dispatch-app-needs-read-access-to-existing-apps"></a>Приложению для отправки требуется доступ на чтение к имеющимся приложениям

Средству отправки требуется доступ для чтения имеющихся приложений LUIS и QnA Maker для создания родительского приложения LUIS, которое отправляется приложениям LUIS и QnA Maker. Вместе с этим доступом предоставляются идентификаторы приложений и ключи разработки.

### <a name="service-authoring-keys"></a>Ключи для создания служб

**Ключ разработки** используется только для создания и редактирования моделей. Вам нужен идентификатор и ключ для каждого приложения LUIS и приложения QnA Maker.

|Приложение|Расположение сведений|
|--|--|
|LUIS|**Идентификатор приложения** можно узнать на [портале LUIS](https://www.luis.ai) для каждого приложения через команду "Управление -> Сведения о приложении".<br>**Ключ разработки** можно узнать на портале LUIS. Выберите пользователя в правом верхнем углу и щелкните "Параметры".|
|QnA Maker| [Идентификатор приложения](https://http://qnamaker.ai) можно увидеть на портале **QnA Maker** на странице параметров, которая становится доступна после публикации приложения. Это идентификатор, расположенный в первой части команды POST после элемента knowledgebase. Например, `POST /knowledgebases/<APP-ID>/generateAnswer`.<br>**Ключ разработки** можно узнать на портале Azure для ресурса QnA Maker, в разделе **Ключи**. Вам потребуется только один ключ.|

Ключ разработки не используется для получения оценки прогнозирования или достоверности из опубликованного приложения. Для этого действия нужны ключи конечных точек. Мы найдем и применим **[ключи конечной точки](#service-endpoint-keys)** далее в этом руководстве.

См. дополнительные сведения о [двух типах ключей](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure#types-of-keys-in-qna-maker), используемых с QnA Maker.

## <a name="create-the-dispatch-model"></a>Создание модели отправки

Интерфейс CLI для средства Dispatch создает модель для отправки в соответствующее приложение LUIS или QnA Maker.

1. Откройте командную строку или окно терминала и измените каталоги на каталог **CognitiveModels**.
1. Убедитесь, что у вас установлена текущая версия npm и средства Dispatch.

    ```cmd
    npm i -g npm
    npm i -g botdispatch
    ```

1. Используйте `dispatch init` для инициализации файла `.dispatch` для модели отправки. Присвойте файлу понятное имя.

    ```cmd
    dispatch init -n <filename-to-create> --luisAuthoringKey "<your-luis-authoring-key>" --luisAuthoringRegion <your-region>
    ```

1. Используйте `dispatch add` для добавления приложений LUIS и баз знаний QnA Maker в файл `.dispatch`.

    ```cmd
    dispatch add -t luis -i "<app-id-for-weather-app>" -n "<name-of-weather-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_Weather
    dispatch add -t luis -i "<app-id-for-home-automation-app>" -n "<name-of-home-automation-app>" -v <app-version-number> -k "<your-luis-authoring-key>" --intentName l_HomeAutomation
    dispatch add -t qna -i "<knowledge-base-id>" -n "<knowledge-base-name>" -k "<azure-qna-service-key1>" --intentName q_sample-qna
    ```

1. Используйте `dispatch create` для создания модели отправки из файла `.dispatch`.

    ```cmd
    dispatch create
    ```

1. Опубликуйте созданное приложение LUIS для отправки.

## <a name="use-the-dispatch-luis-app"></a>Использование приложения LUIS для отправки

Созданное приложение LUIS определяет намерения для каждого дочернего приложения и каждой базы знаний, а также намерение _none_ при несоответствии речевого фрагмента.

- `l_HomeAutomation`
- `l_Weather`
- `None`
- `q_sample-qna`

Эти службы нужно опубликовать с соответствующими именами, чтобы обеспечить правильную работу бота.
Бот получает доступ к опубликованным службам на основе этих сведений.

### <a name="service-endpoint-keys"></a>Ключи конечной точки службы

Боту требуются конечные точки прогнозирования запросов для трех приложений LUIS (отправка, погода и домашняя автоматика) и единой базы знаний QnA Maker. Чтобы найти ключи конечной точки, воспользуйтесь сведениями из следующей таблицы:

|Приложение|Расположение ключа конечной точки запроса|
|--|--|
|LUIS|Чтобы найти ключи, связанные с каждым приложением, перейдите на портал LUIS и для каждого приложения LUIS в разделе "Управление" выберите **Keys and Endpoint settings** (Параметры ключей и конечной точки). Если вы в точности выполняете инструкции, приведенные в этом учебнике, ключом конечной точки является ключ `<your-luis-authoring-key>`. Ключ разработки допускает 1000 попаданий конечной точки, а затем срок его действия истекает.|
|QnA Maker|На портале QnA Maker для базы знаний перейдите в раздел "Управление настройками" и используйте значение ключа, отображаемое в параметрах Postman для заголовка **Авторизация** без текста `EndpointKey`.|

Эти значения используются в файле **appsettings.json** для C# и в файле **.env** для javascript.

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="installing-packages"></a>Установка пакетов

Перед первым запуском приложения убедитесь, что установлены следующие пакеты NuGet:

- **Microsoft.Bot.Builder**
- **Microsoft.Bot.Builder.AI.Luis**
- **Microsoft.Bot.Builder.AI.QnA**

### <a name="manually-update-your-appsettingsjson-file"></a>Обновление файла appsettings.json вручную

После создания всех приложений службы вам нужно добавить сведения о них в файл appsettings.json. Исходный пример кода [C#][cs-sample] содержит пустой файл appsettings.json:

**appsettings.json**

[!code-json[AppSettings](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/AppSettings.json?range=8-17)]

Для каждой сущности ниже добавьте значения, которые вы записали ранее при выполнении этих инструкций:

**appsettings.json**

```json
"MicrosoftAppId": "",
"MicrosoftAppPassword": "",
  
"QnAKnowledgebaseId": "<knowledge-base-id>",
"QnAEndpointKey": "<qna-maker-resource-key>",
"QnAEndpointHostName": "<your-hostname>",

"LuisAppId": "<app-id-for-dispatch-app>",
"LuisAPIKey": "<your-luis-endpoint-key>",
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

После создания всех приложений службы вам нужно добавить сведения о них в файл .env. Исходный пример кода [JavaScript][js-sample] содержит пустой ENV-файл. 

Файл с расширением **.env**  
[!code-file[EmptyEnv](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/.env?range=1-10)]

Добавьте сведения о подключении службы, как показано ниже:

Файл с расширением **.env**

```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAEndpointKey="<qna-maker-resource-key>"
QnAEndpointHostName="<your-hostname>"

LuisAppId=<app-id-for-dispatch-app>
LuisAPIKey=<your-luis-endpoint-key>
LuisAPIHostName=<your-dispatch-app-region>
```

Когда все изменения будут внесены, сохраните файл.

## <a name="pythontabpython"></a>[Python](#tab/python)

### <a name="installing-packages"></a>Установка пакетов

Перед первым запуском приложения убедитесь, что установлены следующие пакеты pypi.

```powershell
pip install azure
pip install botbuilder-core
pip install botbuilder-ai
```

### <a name="manually-update-your-configpy-file"></a>Обновление файла config.py вручную
После создания всех приложений службы вам нужно добавить сведения о них в файл config.py. В начальном [примере кода Python][python-sample] содержится пустой файл config.py. 

**config.py**

[!code-python[config.py](~/../botbuilder-python/samples/python/14.nlp-with-dispatch/config.py?range=10-24)]

Для каждой сущности ниже добавьте значения, которые вы записали ранее при выполнении этих инструкций:
```python
APP_ID = os.environ.get("MicrosoftAppId", "")
APP_PASSWORD = os.environ.get("MicrosoftAppPassword", "")

QNA_KNOWLEDGEBASE_ID = os.environ.get("QnAKnowledgebaseId", "<knowledge-base-id>")
QNA_ENDPOINT_KEY = os.environ.get("QnAEndpointKey", "<qna-maker-resource-key>")
QNA_ENDPOINT_HOST = os.environ.get("QnAEndpointHostName", "<your-hostname>")

LUIS_APP_ID = os.environ.get("LuisAppId", "<app-id-for-dispatch-app>")
LUIS_API_KEY = os.environ.get("LuisAPIKey", "<your-luis-endpoint-key>")
# LUIS endpoint host name, ie "westus.api.cognitive.microsoft.com"
LUIS_API_HOST_NAME = os.environ.get("LuisAPIHostName", "<your-dispatch-app-region>")
```
Когда все изменения будут внесены, сохраните файл.

---

### <a name="connect-to-the-services-from-your-bot"></a>Подключение к службам из бота

Чтобы подключиться к средству Dispatch, а также службам LUIS и QnA Maker, бот получает сведения из файла параметров (`appsettings.json` или `.env`).

## <a name="ctabcs"></a>[C#](#tab/cs)

В файле **BotServices.cs** сведения из файла конфигурации _appsettings.json_ используются для подключения бота отправки к службам `Dispatch` и `SampleQnA`. Конструкторы используют предоставленные значения для подключения к этим службам.

**BotServices.cs**

[!code-csharp[ReadConfigurationInfo](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/BotServices.cs?range=16-31)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В файле **dispatchBot.js** сведения из файла конфигурации _.env_ используются для подключения бота отправки к службам _LuisRecognizer(dispatch)_ и _QnAMaker_. Конструкторы используют предоставленные значения для подключения к этим службам.

**bots/dispatchBot.js**

[!code-javascript[ReadConfigurationInfo](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=11-26)]

## <a name="pythontabpython"></a>[Python](#tab/python)
В файле **dispatch_bot.py** сведения из файла конфигурации _config.py_ используются для подключения бота отправки к службам _QnAMaker_ и _LuisRecognizer_. Конструкторы используют предоставленные значения для подключения к этим службам.

**bots/dispatch_bot.py**

[!code-python[ReadConfigurationInfo](~/../botbuilder-python/samples/python/14.nlp-with-dispatch/bots/dispatch_bot.py?range=14-30)]

---

> [!NOTE]
> По умолчанию параметру `includeApiResults` присваивается значение false, означающее, что распознаватель будет возвращать только основные сведения о сущностях и намерениях. Если требуется полный ответ от LUIS (например, `ConnectedServiceResult`, см. далее в этом руководстве), установите для этого параметра значение true. После этого будет добавлен полный ответ от службы LUIS в коллекцию свойств для `RecognizerResult`.

### <a name="call-the-services-from-your-bot"></a>Вызов служб из бота

Логика бота проверяет каждый блок вводимых пользователем данных с использованием объединенной модели отправки, находит первые возвращаемые намерения и использует эти сведения, чтобы вызвать соответствующую службу для входных данных.

## <a name="ctabcs"></a>[C#](#tab/cs)

В файле **DispatchBot.cs** при вызове метода `OnMessageActivityAsync` мы проверяем входящее сообщение пользователя с использованием модели отправки. Затем мы передаем `topIntent` и `recognizerResult` модели отправки в соответствующий метод, чтобы вызвать службу и получить результат.

**bots\DispatchBot.cs**

[!code-csharp[OnMessageActivity](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=26-36)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В методе **dispatchBot.js** `onMessage` мы проверяем входящее сообщение пользователя с использованием модели отправки, а затем находим и передаем _topIntent_ путем вызова _dispatchToTopIntentAsync_.

[!code-javascript[onMessage](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=31-44)]

## <a name="pythontabpython"></a>[Python](#tab/python)

В файле **DispatchBot.cs** при вызове метода `on_message_activity` мы проверяем входящее сообщение пользователя с использованием модели отправки. Затем мы передаем `top_intent` и `recognize_result` модели отправки в соответствующий метод, чтобы вызвать службу и получить результат.

**bots/dispatch_bot.py**

[!code-python[on_message](~/../botbuilder-python/samples/python/14.nlp-with-dispatch/bots/dispatch_bot.py?range=46-54)]

---

### <a name="work-with-the-recognition-results"></a>Использование результатов распознавания

## <a name="ctabcs"></a>[C#](#tab/cs)

Когда модель возвращает результат, он позволяет понять, какие службы лучше всего подходят для обработки этого высказывания. Код нашего бота направляет запрос в соответствующую службу и обрабатывает полученный от нее ответ. Код использует _намерение_, которое возвращает средство Dispatch, чтобы выполнить перенаправление в соответствующую модель LUIS или службу QnA.

**bots\DispatchBot.cs**

[!code-csharp[DispatchToTop](~/../botbuilder-samples/samples/csharp_dotnetcore/14.nlp-with-dispatch/bots/DispatchBot.cs?range=51-69)]

Вызываемые методы `ProcessHomeAutomationAsync` или `ProcessWeatherAsync` передают результаты из модели отправки с использованием _luisResult.ConnectedServiceResult_. Затем указанный метод предоставляет отзыв пользователя, отображая первое намерение модели отправки, а также ранжированный список всех намерений и сущностей, которые были обнаружены.

Вызываемый метод `q_sample-qna` использует введенные пользователем данные, содержащиеся в turnContext, для создания ответа из базы знаний и отображения результата пользователю.

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Когда модель возвращает результат, он позволяет понять, какие службы лучше всего подходят для обработки этого высказывания. Код в этом примере использует результат в _topIntent_, демонстрируя, как перенаправить запрос в соответствующую службу.

**bots/dispatchBot.js**  
[!code-javascript[dispatchToTopIntentAsync](~/../botbuilder-samples/samples/javascript_nodejs/14.nlp-with-dispatch/bots/dispatchBot.js?range=61-77)]

Вызываемые методы `processHomeAutomation` или `processWeather` передают результаты из модели отправки с использованием _recognizerResult.luisResult_. Затем указанный метод предоставляет отзыв пользователя, отображая первое намерение модели отправки, а также ранжированный список всех намерений и сущностей, которые были обнаружены.

Вызываемый метод `q_sample-qna` использует введенные пользователем данные, содержащиеся в turnContext, для создания ответа из базы знаний и отображения результата пользователю.

## <a name="pythontabpython"></a>[Python](#tab/python)

Когда модель возвращает результат, он позволяет понять, какие службы лучше всего подходят для обработки этого высказывания. Код в этом примере использует лучшее распознанное _намерение_, демонстрируя, как перенаправить запрос в соответствующую службу.

**bots\dispatch_bot.py**

[!code-python[dispatch top intent](~/../botbuilder-python/samples/python/14.nlp-with-dispatch/bots/dispatch_bot.py?range=56-70)]

Вызываемые методы `_process_home_automation` или `_process_weather` передают результаты из модели отправки с использованием _recognizer_result.properties["luisResult"]_ . Затем указанный метод предоставляет отзыв пользователя, отображая первое намерение модели отправки, а также ранжированный список всех намерений и сущностей, которые были обнаружены.

Вызываемый метод `q_sample-qna` использует введенные пользователем данные, содержащиеся в turnContext, для создания ответа из базы знаний и отображения результата пользователю.

---

> [!NOTE]
> Если используется рабочее приложение, выбранные методы LUIS подключаются к соответствующей службе, передают введенные пользователем данные и обрабатывают возвращаемые LUIS данные о намерениях и сущностях.

## <a name="test-your-bot"></a>Тестирование бота

1. В среде разработки откройте файл с пустым кодом. Запишите адрес _localhost_ из адресной строки окна браузера, открытого приложением: https://localhost:<Port_Number>. 
1. Откройте Bot Framework Emulator и щелкните `Create a new bot configuration`. Файл `.bot` позволяет использовать _инспектор_ в эмуляторе бота, чтобы просмотреть код JSON, возвращенный из LUIS и QnA Maker.
1. В диалоговом окне **New bot configuration** (Новая конфигурация бота) введите имя своего бота и URL-адрес конечной точки, например `http://localhost:3978/api/messages`. Сохраните файл в корневой папке своего проекта примера кода бота.
1. Откройте файл бота и добавьте разделы для приложений LUIS и QnA Maker. Используйте [этот пример файла](https://github.com/microsoft/botbuilder-tools/blob/master/packages/MSBot/docs/sample-bot-file.json) в качестве шаблона для настроек. Сохраните изменения.
1. Выберите имя бота в списке **My Bots** (Мои боты), чтобы получить доступ к запущенному боту. Ниже для справки приведены некоторые вопросы и команды, поддерживаемые в используемых для бота службах.

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

## <a name="dispatch-for-user-utterance-to-qna-maker"></a>Отправка пользовательских речевых фрагментов в QnA Maker

1. В эмуляторе бота введите текст `hi` и отправьте речевой фрагмент. Бот отправляет этот запрос в приложение LUIS для отправки и возвращает ответ, указывающий дочернее приложение, которое должно получить этот речевой фрагмент для дальнейшей обработки.

1. Выбрав в журнале строку `LUIS Trace`, вы сможете просмотреть ответ LUIS в эмуляторе бота. Результат LUIS, полученный от приложения LUIS для отправки отображается в инспекторе.

    ```json
    {
      "luisResponse": {
        "entities": [],
        "intents": [
          {
            "intent": "q_sample-qna",
            "score": 0.9489713
          },
          {
            "intent": "l_HomeAutomation",
            "score": 0.0612499453
          },
          {
            "intent": "None",
            "score": 0.008567564
          },
          {
            "intent": "l_Weather",
            "score": 0.0025761195
          }
        ],
        "query": "Hi",
        "topScoringIntent": {
          "intent": "q_sample-qna",
          "score": 0.9489713
        }
      }
    }
    ```

    Так как речевой фрагмент `hi` является частью намерения **q_sample-qna** приложения LUIS для отправки и выбрано в качестве `topScoringIntent`, бот выполнит второй запрос с тем же речевым фрагментом, на этот раз к приложению QnA Maker.

1. Выберите строку `QnAMaker Trace` в журнале эмулятора бота. Результат QnA Maker отображается в инспекторе.

```json
{
    "questions": [
        "hi",
        "greetings",
        "good morning",
        "good evening"
    ],
    "answer": "Hello!",
    "score": 1,
    "id": 96,
    "source": "QnAMaker.tsv",
    "metadata": [],
    "context": {
        "isContextOnly": false,
        "prompts": []
    }
}
```

## <a name="resolving-incorrect-top-intent-from-dispatch"></a>Удаление неверного основного намерения из средства Dispatch

Когда бот будет запущен, вы можете повысить его речевые фрагменты, удаляя похожие или перекрывающиеся высказывания в приложениях для отправки.
<!--For example, let's say that in the `Home Automation` LUIS app requests like "turn my lights on" map to a "TurnOnLights" intent, but requests like "Why won't my lights turn on?" map to a "None" intent so that they can be passed on to QnA Maker. These two utterances are too close for the dispatch LUIS app to determine if the correct child app is the LUIS app or the QnA Maker app.

When you combine the LUIS app and the QnA Maker app using dispatch, you need to do _one_ of the following:

- Remove the "None" intent from the child `Home Automation` LUIS app, and instead add the utterances from that intent to the "None" intent in the dispatcher app.
- Add logic in your bot to pass the messages that match the Dispatch LUIS app's "None" intent on to the QnA maker service. Compare the score of the Dispatch LUIS app's score and the score of the QnA Maker app. Use the highest score. This effectively removes QnA Maker from the Dispatch cycle.

Either of the above two actions will reduce the number of times that your bot responds back to your users with the message, 'Couldn't find an answer.'
-->
Чтобы протестировать и оценить модель подготовки к отправке, можно использовать средство командной строки [Dispatch][dispatch-readme].

### <a name="to-update-or-create-a-new-luis-model"></a>Обновление или создание модели LUIS

этот пример основан на предварительно настроенной модели LUIS. Сведения, которые вам потребуются для обновления этой модели или создания новой модели LUIS, вы найдете [здесь](https://aka.ms/create-luis-model#updating-your-cognitive-models).

После обновления базовых моделей (QnA или LUIS) запустите `dispatch refresh`, чтобы обновить приложение LUIS для отправки. По сути `dispatch refresh` отличается от `dispatch create` только тем, что не создает новый идентификатор приложения LUIS. 

Обратите внимание, что добавленные непосредственно в LUIS речевые фрагменты не будут сохранены при выполнении `dispatch refresh`. Чтобы сохранить эти дополнительные речевые фрагменты в приложении отправки, добавьте их в текстовый файл (по одному фрагменту в строке), а затем добавьте этот файл в приложение с помощью следующей команды:

```powershell
dispatch add -t file -f <file path> --intentName <target intent name, ie l_General>
```

Когда вы добавите файл с дополнительными речевыми фрагментами в приложение отправки, фрагменты будут сохраняться нетронутыми после каждого обновления.

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

Чтобы улучшить описанные здесь службы, изучите рекомендации по [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-concept-best-practices) и [QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/concepts/best-practices).

<!-- Foot-note style links -->



[howto-luis]: bot-builder-howto-v4-luis.md
[howto-qna]: bot-builder-howto-qna.md

[cs-sample]: https://aka.ms/dispatch-sample-cs
[js-sample]: https://aka.ms/dispatch-sample-js
[python-sample]: https://aka.ms/dispatch-sample-python

[dispatch-readme]: https://aka.ms/dispatch-command-line-tool
<!--[dispatch-evaluate]: https://aka.ms/dispatch-command-line-tool#evaluating-your-dispatch-model-->

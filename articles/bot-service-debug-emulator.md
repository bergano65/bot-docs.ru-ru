---
title: Тестирование и отладка ботов с помощью эмулятора Bot Framework — Служба Azure Bot
description: Сведения о проверке, тестировании и отладке ботов с помощью классического приложения "Эмулятор Bot Framework".
keywords: расшифровка, инструмент msbot, языковые службы, распознавание речи
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 2/26/2019
ms.openlocfilehash: c7e6dd48df1c80d7a15e06d3fbb874961d3e6aec
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75792868"
---
# <a name="debug-with-the-emulator"></a>Отладка ботов с помощью эмулятора

Эмулятор Bot Framework — это классическое приложение, которое позволяет разработчикам локально или удаленно тестировать и отлаживать боты. С помощью эмулятора вы можете общаться с ботом и проверять сообщения, которые он отправляет и получает. Эмулятор отображает сообщения так, как они будут выглядеть в интерфейсе веб-чата, и регистрирует запросы и ответы JSON при обмене сообщениями с ботом. Перед развертыванием бота в облаке запустите его локально и протестируйте с помощью эмулятора. Вы можете протестировать бот с помощью эмулятора, даже если вы еще не [создали](./bot-service-quickstart.md) его в службе Azure Bot или не настроили для запуска во всех каналах.

## <a name="prerequisites"></a>предварительные требования
- Установите [Bot Framework Emulator](https://aka.ms/Emulator-wiki-getting-started).

## <a name="run-a-bot-locally"></a>Локальный запуск бота
Подключаемого к Bot Framework Emulator бота нужно сперва запустить локально. Это можно сделать с помощью Visual Studio, Visual Studio Code или командной строки. Чтобы запустить бота с помощью командной строки, сделайте следующее:


# <a name="ctabcsharp"></a>[C#](#tab/csharp)

* В командной строке измените каталог на каталог проекта ботов.
* Запустите бота, выполнив следующую команду: 
    ```
    dotnet run
    ```
* Скопируйте номер порта в строке перед текстом *Application started. Press CTRL+C to shut down.*

    ![Номер порта — C#](media/bot-service-debug-emulator/csharp_port_number.png)


# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

* В командной строке измените каталог на каталог проекта ботов.
* Запустите бота, выполнив следующую команду:
    ```
    node index.js
    ```
* Скопируйте номер порта, который прослушивает restify.

    ![Номер порта — JavaScript](media/bot-service-debug-emulator/js_port_number.png)

# <a name="pythontabpython"></a>[Python](#tab/python)
* В командной строке измените каталог на каталог проекта ботов.
* Запустите бота, выполнив следующую команду:
    ```
   python app.py
    ```
* Скопируйте номер порта, который прослушивает restify.

    ![Номер порта — JavaScript](media/bot-service-debug-emulator/js_port_number.png)

---

На этом этапе бот должен запуститься локально. 


## <a name="connect-to-a-bot-running-on-localhost"></a>Подключение к боту, работающему на локальном узле

<!-- auth config steps -->
### <a name="configure-the-emulator-for-authentication"></a>Настройка эмулятора для аутентификации

Если для бота требуется аутентификация и отображается диалоговое окна входа, настройте эмулятор, как показано ниже.

#### <a name="using-sign-in-verification-code"></a>Использование кода проверки входа в систему

1. Запустите эмулятор.
1. В эмуляторе щелкните значок шестеренки в левом нижнем углу или вкладку **параметров эмулятора** в правом верхнем углу.
1. Установите флажок **Use a sign-in verification code for OAuthCards** (Использовать код проверки входа в систему для OAuthCards)
1. Установите флажок **Bypass ngrok for local address** (Обходить ngrok для локального адреса).
1. Нажмите кнопку **Сохранить**.

При нажатии кнопки входа в боте будет создан код проверки.
Чтобы пройти аутентификацию, введите этот код в поле чата бота.
После этого можно выполнять разрешенные операции.

Кроме того, вы можете выполнить описанные ниже действия.

#### <a name="using-authentication-tokens"></a>Использование маркеров аутентификации

1. Запустите эмулятор.
1. В эмуляторе щелкните значок шестеренки в левом нижнем углу или вкладку **параметров эмулятора** в правом верхнем углу.
1. Установите флажок **Use version1.0 authentication tokens** (Использовать маркеры аутентификации версии 1.0).
1. Введите локальный путь к средству **ngrok**. См. дополнительные сведения о [ngrok](https://ngrok.com/).
1. Установите флажок **Run ngrok when the Emulator starts up** (Запустить ngrok при запуске эмулятора).
1. Нажмите кнопку **Сохранить**.

Когда вы нажмете кнопку входа в боте, вам будет предложено ввести свои учетные данные. Будет создан маркер аутентификации. После этого можно выполнять разрешенные операции.


![Интерфейс эмулятора](media/emulator-v4/emulator-welcome.png)

Чтобы подключиться к запущенному локально боту, щелкните **Открыть бота**. Добавьте скопированный ранее номер порта в следующий URL-адрес и вставьте полученный URL-адрес в поле "URL-адрес бота":

*http://localhost:**номер_порта**/api/messages*

![Интерфейс эмулятора](media/bot-service-debug-emulator/open_bot_emulator.png)

Если бот выполняется с [учетными данными учетной записи Майкрософт (MSA)](#use-bot-credentials), введите эти учетные данные.

### <a name="use-bot-credentials"></a>Использование учетных данных бота

Открыв бот, задайте **идентификатор приложения Майкрософт** и **пароль приложения Майкрософт**, если бот выполняется с учетными данными. Если вы создали бот с помощью службы Azure Bot, учетные данные доступны в Службе приложений бота в разделе **Параметры -> Конфигурация**. Если вы не знаете значения, можно удалить их из файла конфигурации локально запущенного бота, а затем запустить бот в эмуляторе. Если бот не работает с этими параметрами, эмулятор тоже не нужно запускать с ними. 

## <a name="view-detailed-message-activity-with-the-inspector"></a>Просмотр действий с сообщениями с помощью инспектора

Отправьте боту сообщение и получите от него ответ. Сообщение можно щелкнуть в окне беседы, чтобы просмотреть необработанные действия JSON с помощью **компонента для проверки** в правой части окна. При выборе сообщение станет желтым, и объект действия JSON будет отображаться слева от окна чата. Данные JSON включают метаданные ключа, в том числе идентификатор канала, тип действия, идентификатор беседы, текстовое сообщение, URL-адрес конечной точки и т. д. Можно проверить действия, отправленные пользователем, а также действия, с помощью которых бот отвечает.

![Действия с сообщениями эмулятора](media/emulator-v4/emulator-view-message-activity-03.png)

> [!TIP]
> Вы можете отлаживать изменения состояния в подключенном к каналу боте, добавив в него [проверяющее ПО промежуточного слоя](bot-service-debug-inspection-middleware.md).

<!--
## Save and load conversations with bot transcripts

Activities in the emulator can be saved as transcripts. From an open live chat window, select **Save Transcript As** to the transcript file. The **Start Over** button can be used any time to clear a conversation and restart a connection to the bot.  

![Emulator save transcripts](media/emulator-v4/emulator-save-transcript.png)

To load transcripts, simply select **File > Open Transcript File** and select the transcript. A new Transcript window will open and render the message activity to the output window. 

![Emulator load transcripts](media/emulator-v4/emulator-load-transcript.png)
--->
<!---
## Add services 

You can easily add a LUIS app, QnA knowledge base, or dispatch model to your bot directly from the emulator. When the bot is loaded, select the services button on the far left of the emulator window. You will see options under the **Services** menu to add LUIS, QnA Maker, and Dispatch. 

To add a service app, simply click on the **+** button and select the service you want to add. You will be prompted to sign in to the Azure portal to add the service to the bot file, and connect the service to your bot application. 

> [!IMPORTANT]
> Adding services only works if you're using a `.bot` configuration file. Services will need to be added independently. For details on that, see [Manage bot resources](v4sdk/bot-file-basics.md) or the individual how to articles for the service you're trying to add.
>
> If you are not using a `.bot` file, the left pane won't have your services listed (even if your bot uses services) and will display *Services not available*.

![LUIS connect](media/emulator-v4/emulator-connect-luis-btn.png)

When either service is connected, you can go back to a live chat window and verify that your services are connected and working. 

![QnA connected](media/emulator-v4/emulator-view-message-activity.png)

--->

## <a name="inspect-services"></a>Проверка служб

С помощью эмулятора версии 4 вы также можете проверить ответы JSON от LUIS и QnA. Для бота с подключенными языковыми службами в окне журнала в нижнем правом углу можно выбрать **трассировку**. Этот новый инструмент также предоставляет компоненты для обновления языковых служб непосредственно из эмулятора. 

![Инспектор LUIS](media/emulator-v4/emulator-luis-inspector.png)

Если подключена служба LUIS, вы заметите, что в ссылке трассировки указана **трассировка Luis**. Выбрав ее, вы увидите необработанный ответ от службы LUIS, включающий намерения, сущности и соответствующие оценки. Также имеется возможность переназначить намерения для фраз пользователей. 

![Инспектор QnA](media/emulator-v4/emulator-qna-inspector.png)

Если подключена служба QnA, журнал отобразит **трассировку QnA**. Выбрав ее, можно выполнить предварительный просмотр пары "вопрос-ответ", связанной с этим действием, наряду с оценкой достоверности. Здесь можно добавить выражения альтернативных вопросов для ответа.

<!--## Configure ngrok

If you are using Windows and you are running the Bot Framework Emulator behind a firewall or other network boundary and want to connect to a bot that is hosted remotely, you must install and configure **ngrok** tunneling software. The Bot Framework Emulator integrates tightly with ngrok tunnelling software (developed by [inconshreveable][inconshreveable]), and can launch it automatically when it is needed.

Open the **Emulator Settings**, enter the path to ngrok, select whether or not to bypass ngrok for local addresses, and click **Save**.

![ngrok path](media/emulator-v4/emulator-ngrok-path.png)
-->

<!---## Login to Azure

You can use Emulator to login in to your Azure account. This is particularly helpful for you to add and manage services your bot depends on. 
See [above](#add-services) to learn more about services you can manage using the Emulator.
-->

### <a name="login-to-azure"></a>Вход в Azure
Для входа в учетную запись Azure можно использовать Эмулятор. Это особенно полезно для добавления служб, от которых зависит ваш бот, и управления ими. Войдите в Azure, сделав следующее:
- Щелкните "Файл" -> "Вход с помощью Azure". ![Вход в Azure](media/emulator-v4/emulator-azure-login.png)
- На экране приветствия нажмите кнопку Sign in with your Azure account (Войти в систему, используя учетную запись Azure). Здесь вы также можете указать, что Эмулятор должен автоматически выполнять вход после перезагрузки приложения в Эмуляторе.
![Вход в Azure](media/emulator-v4/emulator-azure-login-success.png)

## <a name="disabling-data-collection"></a>Отключение сбора данных

Если вы решите, что Эмулятор не должен далее собирать данные об использовании, сбор данных можно отключить следующим образом.

1. Перейдите на страницу параметров Эмулятора, нажав кнопку параметров (с изображением шестеренки) на панели навигации слева.

    ![Отключение сбора данных](media/emulator-v4/emulator-disable-data-1.png)

2. Снимите флажок *Help improve the Emulator by allowing us to collect usage data* (Помогите нам улучшить эмулятор, разрешив собирать данные об использовании) в разделе **Data Collection** (Сбор данных).

    ![Отключение сбора данных](media/emulator-v4/emulator-disable-data-2.png)

3. Нажмите кнопку "Сохранить".

    ![Отключение сбора данных](media/emulator-v4/emulator-disable-data-3.png)
    
Если вы передумаете, его можно снова включить, установив флажок.

## <a name="additional-resources"></a>Дополнительные ресурсы

Эмулятор Bot Framework — это решение с открытым исходным кодом. Вы можете [принять участие][EmulatorGithubContribute] в разработке, [отправляя сведения об ошибках и предложения][EmulatorGithubBugs].

Подробные сведения см. в статье об [устранении общих проблем](bot-service-troubleshoot-bot-configuration.md) и других статья об устранении неполадок в этом разделе.

## <a name="next-steps"></a>Дальнейшие действия

Воспользуйтесь проверяющим ПО промежуточного слоя для отладки подключенного к каналу бота.

> [!div class="nextstepaction"]
> [Отладка бота с помощью файлов записей разговоров](bot-service-debug-inspection-middleware.md)

<!--
Saving a conversation to a transcript file allows you to quickly draft and replay a certain set of interactions for debugging.

> [!div class="nextstepaction"]
> [Debug your bot using transcript files](~/v4sdk/bot-builder-debug-transcript.md)
-->

<!-- Footnote-style URLs -->

[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/

---
title: Тестирование и отладка ботов с помощью эмулятора Bot Framework | Документы Майкрософт
description: Сведения о проверке, тестировании и отладке ботов с помощью классического приложения "Эмулятор Bot Framework".
keywords: расшифровка, инструмент msbot, языковые службы, распознавание речи
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/30/2018
ms.openlocfilehash: bc1da99c7d0f7a6431ad0c2746b8583ef7bfbd5f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305458"
---
# <a name="debug-bots-with-the-bot-framework-emulator"></a>Отладка ботов с помощью эмулятора Bot Framework

Эмулятор Bot Framework — это классическое приложение, которое позволяет разработчикам локально или удаленно тестировать и отлаживать боты. С помощью эмулятора вы можете общаться с ботом и проверять сообщения, которые он посылает и получает. Эмулятор отображает сообщения так, как они будут выглядеть в интерфейсе веб-чата, и регистрирует запросы и ответы JSON при обмене сообщениями с ботом. 

> [!TIP] 
> Перед развертыванием бота в облаке запустите его локально и протестируйте с помощью эмулятора. Вы можете протестировать бот с помощью эмулятора, даже если вы еще не [зарегистрировали](~/bot-service-quickstart-registration.md) его в Bot Framework или не настроили для запуска во всех каналах.

![Интерфейс эмулятора](media/emulator-v4/emulator-welcome.png)

## <a name="download-the-bot-framework-emulator"></a>Загрузка эмулятора Bot Framework

Скачайте пакеты загрузки для Mac, Windows или Linux, которые можно найти на [странице выпусков GitHub](https://github.com/Microsoft/BotFramework-Emulator/releases).

## <a name="connect-to-a-bot-running-on-local-host"></a>Подключение к боту, работающему на локальном узле

![Конечные точки эмулятора](media/emulator-v4/emulator-endpoint.png)

Чтобы подключиться к боту, работающему локально, перейдите на вкладку обозревателя ботов в левом верхнем углу. Щелкните значок **+** рядом с вкладкой **Конечная точка**. Здесь вы можете указать конечную точку для того же порта, где локально выполняется бот, для подключения к нему. Нажмите кнопку **Отправить**, и вы будете перенаправлены в окно чата в реальном времени, где сможете взаимодействовать с ботом.

## <a name="view-detailed-message-activity-with-the-inspector"></a>Просмотр действий с сообщениями с помощью инспектора

![Действия с сообщениями эмулятора](media/emulator-v4/emulator-view-message-activity-02.png)

Можно щелкнуть любое сообщение в окне беседы, чтобы просмотреть необработанные действия JSON с помощью функции **ИНСПЕКТОР** в правой части окна. При выборе сообщение станет желтым, и объект действия JSON будет отображаться слева от окна чата. Данные JSON включают метаданные ключа, в том числе channelID, тип действия, идентификатор беседы, текстовое сообщение, URL-адрес конечной точки и т. д. Можно просмотреть действия проверки, отправленные пользователем, а также действия, с помощью которых бот отвечает. 

Можно также использовать [Инспектор каналов](bot-service-channel-inspector.md) для предварительного просмотра поддерживаемых функций конкретного канала.


## <a name="save-and-load-conversations-with-bot-transcripts"></a>Сохранение и загрузка бесед с расшифровками бота

Действия с сообщениями в эмуляторе можно сохранить в качестве расшифровок. В открытом окне чата в реальном времени можно выбрать действие **Сохранить расшифровку как**, чтобы выбрать расположение и задать имя для выходного файла расшифровки. 

>[!TIP]
> Нажав кнопку **Начать сначала**, можно в любое время очистить беседу и перезапустить подключение к боту.  

![Эмулятор сохраняет расшифровки](media/emulator-v4/emulator-live-chat.png)

Чтобы загрузить расшифровки, выберите **Файл** --> **Открыть файл расшифровки**, а затем выберите расшифровку. Откроется новое окно расшифровки, и в окне вывода отобразятся действия с сообщениями. 

![Эмулятор загружает расшифровки](media/emulator-v4/emulator-load-transcript.png)

## <a name="author-transcripts-with-chatdown"></a>Создание расшифровок с помощью Chatdown

Инструмент [Chatdown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown) — генератор расшифровок, который использует файл [markdown](https://daringfireball.net/projects/markdown/syntax) для создания расшифровок действий. Можно создать собственные расшифровки полностью в формате markdown и сохранить их в виде **CHAT**-файла для создания расшифровок. Это удобно для создания сценариев имитации беседы при разработке ботов.  

### <a name="prerequisites"></a>Предварительные требования

- [Эмулятор Bot Framework](https://github.com/Microsoft/BotFramework-Emulator/releases) версии 4 (или более поздней) 
- [Node.js](https://nodejs.org/en/)
 
Chatdown доступен как модуль npm, который требует Node.js. Чтобы установить Chatdown, глобально установите его на компьютер. 

```
npm install -g chatdown
```
### <a name="create-and-load-transcript-transcript-files"></a>Создание и загрузка файлов расшифровки ###

Ниже приведен пример создания **CHAT**-файла. Формат этих файлов — markdown. Они состоят из 2 частей:
- заголовка, который определяет участников диалога (пользователь, бот);
- беседы с обменом репликами между участниками.

```
user=John Doe
bot=Bot

bot: Hello!
user: hey
bot: [Typing][Delay=3000]
What can I do for you?
user: Actually nevermind, goodbye.
bot: bye!
```
[Щелкните здесь](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown/Examples), чтобы увидеть больше примеров CHAT-файлов. 

Для создания файла расшифровки введите имя CHAT-файла, за которым следует символ ">", и имя выходного файла расшифровки с помощью команды **chatdown** в интерфейс командной строки. 

```
chatdown sample.chat > sample.transcript
```
## <a name="manage-bot-resources-with-the-msbot-tool"></a>Управление ресурсами бота с помощью инструмента MSBot

Новый инструмент [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) позволяет создавать **BOT**-файл, в котором хранятся метаданные о службах, используемых ботом. Этот файл также позволяет боту подключаться к этим службам из интерфейса командной строки. Этот инструмент доступен как модуль npm. Для его установки выполните следующее.

```
npm install -g msbot 
```
![Окно интерфейса командной строки MSBot](media/emulator-v4/msbot-cli-window.png)


Чтобы создать файл бота, в интерфейсе командной строки введите команду **msbot init**, за которой следует имя бота и конечная точка целевого URL-адреса, например:

```shell
msbot init --name az-cli-bot --endpoint http://localhost:3984/api/messages
```
![Файл бота](media/emulator-v4/botfile-generated.png)

>**Примечание**. Бот, используемый в этом руководстве, — простой бот echo, созданный на основе расширения бота интерфейса командной строки Azure. Для просмотра дополнительных сведений о построении ботов с помощью интерфейса командной строки Azure [щелкните здесь](https://github.com/Microsoft/botbuilder-tools/tree/master/AzureCli). 

Теперь с помощью BOT-файла вы можете загрузить бот в эмулятор. BOT-файл также требуется для регистрации разных конечных точек и компонентов языка в боте. 

![Bot-File-Dropdown](media/emulator-v4/bot-file-dropdown.png)

## <a name="add-language-services"></a>Добавление служб языка 

Приложение LUIS или базу знаний QnA можно зарегистрировать в BOT-файле напрямую из эмулятора. После загрузки BOT-файла нажмите кнопку "Службы" в левой части окна эмулятора. Вы увидите параметры для добавления LUIS, QnA Maker, Dispatch, конечных точек и службы Azure Bot под меню **Службы**. 

Чтобы добавить приложение LUIS, нажмите кнопку **+** в меню LUIS, введите свои учетные данные для приложения LUIS и нажмите **Отправить**. Таким образом вы зарегистрируете приложение LUIS в BOT-файле и подключите службу к приложению бота. 

![Подключение к LUIS](media/emulator-v4/emulator-connect-luis-btn.png)

Аналогично, чтобы добавить базу знаний QnA, нажмите кнопку **+** в меню QnA, введите свои учетные данные для базы знаний QnA Maker и нажмите **Отправить**. Теперь ваша база знаний зарегистрирована в BOT-файле и готова к использованию. 

![Подключение к QnA](media/emulator-v4/emulator-connect-qna-btn.png)

Подключив любую из этих служб, можно вернуться в окно чата в реальном времени и убедиться, что службы подключены и работают. 

![QnA подключен](media/emulator-v4/emulator-view-message-activity.png)

## <a name="inspect-language-services"></a>Проверка языковых служб

С помощью эмулятора версии 4 вы также можете проверить ответы JSON от LUIS и QnA. Для бота с подключенными языковыми службами в окне журнала в нижнем правом углу можно выбрать **трассировку**. Этот новый инструмент также предоставляет компоненты для обновления языковых служб непосредственно из эмулятора. 

![Инспектор LUIS](media/emulator-v4/emulator-luis-inspector.png)

Если подключена служба LUIS, вы заметите, что в ссылке трассировки указана **трассировка Luis**. Выбрав ее, вы увидите необработанный ответ от службы LUIS, включающий намерения, сущности и соответствующие оценки. Также имеется возможность переназначить намерения для фраз пользователей. 

![Инспектор QnA](media/emulator-v4/emulator-qna-inspector.png)

Если подключена служба QnA, журнал отобразит **трассировку QnA**. Выбрав ее, можно выполнить предварительный просмотр пары "вопрос-ответ", связанной с этим действием, наряду с оценкой достоверности. Здесь можно добавить выражения альтернативных вопросов для ответа.

[!TIP]
> Эти функции доступны только для ботов пакета SDK версии 4 


## <a name="speech-recognition"></a>Распознавание речи
Эмулятор Bot Framework поддерживает распознавание речи с помощью [API-интерфейсов речи Cognitive Services](/azure/cognitive-services/Speech/home). Благодаря этому вы можете обучать Кортану или бот с поддержкой речевых функций, используя речевые функции в эмуляторе во время разработки. Эмулятор Bot Framework предоставляет бесплатное распознавание речи до трех часов в день на одного бота. 

## <a id="ngrok"></a> Установка и настройка ngrok

Если вы используете Windows и эмулятор Bot Framework за брандмауэром или другой пограничной сетью и хотите подключаться к удаленному боту, необходимо установить и настроить программное обеспечение туннелирования **ngrok**. Эмулятор Bot Framework тесно интегрируется с программным обеспечением туннелирования [ngrok][ngrokDownload] (разработанным [inconshreveable][inconshreveable]) и при необходимости может запускать его автоматически.

Чтобы установить **ngrok** в Windows и настроить эмулятор для его использования, выполните следующие действия. 

1. Скачайте исполняемый файл [ngrok][ngrokDownload] на локальный компьютер.

2. Откройте диалоговое окно настройки приложения эмулятора, введите путь к файлу ngrok, выберите, нужно ли обходить ngrok для локальных адресов, и нажмите кнопку **Сохранить**.

![Путь к файлу ngrok](media/emulator-v4/emulator-ngrok-path.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

Эмулятор Bot Framework — это решение с открытым исходным кодом. Вы можете [принять участие][EmulatorGithubContribute] в разработке, [отправляя ошибки и предложения][EmulatorGithubBugs].


[EmulatorGithub]: https://github.com/Microsoft/BotFramework-Emulator
[EmulatorGithubContribute]: https://github.com/Microsoft/BotFramework-Emulator/wiki/How-to-Contribute
[EmulatorGithubBugs]: https://github.com/Microsoft/BotFramework-Emulator/wiki/Submitting-Bugs-%26-Suggestions

[ngrokDownload]: https://ngrok.com/
[inconshreveable]: https://inconshreveable.com/
[BotFrameworkDevPortal]: https://dev.botframework.com/


[EmulatorConnectPicture]: ~/media/emulator/emulator-connect_localhost_credentials.png
[EmulatorNgrokPath]: ~/media/emulator/emulator-configure_ngrok_path.png
[EmulatorNgrokMonitor]: ~/media/emulator/emulator-testbot-ngrok-monitoring.png
[EmulatorUI]: ~/media/emulator/emulator-ui-new.png

[TroubleshootingGuide]: ~/bot-service-troubleshoot-general-problems.md
[TroubleshootingAuth]: ~/bot-service-troubleshoot-authentication-problems.md
[NodeGetStarted]: ~/nodejs/bot-builder-nodejs-quickstart.md
[CSGetStarted]: ~/dotnet/bot-builder-dotnet-quickstart.md

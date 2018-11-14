---
title: Управление ботами с помощью средств CLI
description: Средства для Bot Builder позволяют управлять ресурсами бота непосредственно из командной строки
keywords: botbuilder templates, ludown, qna, luis, msbot, manage, cli, .bot, bot
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 11/07/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 357f9fdc3da4c703dbcd5c1fa347176002006567
ms.sourcegitcommit: a54a70106b9fdf278fd7270b25dd51c9bd454ab1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/08/2018
ms.locfileid: "51273111"
---
# <a name="manage-bots-using-cli-tools"></a>Управление ботами с помощью средств CLI

Средства Bot Builder позволяют выполнять весь рабочий процесс разработки чат-ботов, включая планирование, сборку, тестирование, публикацию, подключение и оценку. Давайте узнаем, как эти средства помогут вам на каждом из этапов цикла разработки.

## <a name="plan"></a>План

### <a name="create-mock-conversations-using-chatdown"></a>Создание имитации общения с помощью Chatdown

Chatdown — это генератор текстов, который на основе CHAT-файла создает имитацию разговора. Созданные файлы имитации разговора выводятся на стандартный выход stdout.

Хороший бот, как и любое полезное приложение или успешный сайт, начинается с четкого описания поддерживаемых сценариев. Создание имитации диалогов между ботом и пользователем дает следующие возможности:

- разграничение поддерживаемых ботом сценариев;
- составление примеров, предоставляемых ответственным за принятие решений для оценки и отзывов;
- определение "безоблачного пути" (а возможно, и некоторых других) в потоках общения между пользователем и ботом. Формат CHAT-файла поможет вам создать макеты диалогов между пользователем и ботом. Средство командной строки Chatdown преобразует CHAT-файлы в текстовые диалоги (TRANSCRIPT-файлы), которые можно просматривать в [Bot Framework Emulator версии 4](https://github.com/microsoft/botframework-emulator).

Ниже приведен пример файла `.chat`:

```markdown
user=Joe
bot=LulaBot

bot: Hi!
user: yo!
bot: [Typing][Delay=3000]
Greetings!
What would you like to do?
* update - You can update your account
* List - You can list your data
* help - you can get help

user: I need the bot framework logo.

bot:
Here you go.
[Attachment=bot-framework.png]
[Attachment=http://yahoo.com/bot-framework.png]
[AttachmentLayout=carousel]

user: thanks
bot:
Here's a form for you
[Attachment=card.json adaptivecard]

```

### <a name="create-a-transcript-file-from-chat-file"></a>Создание файла расшифровки из CHAT-файла
Команда Chatdown имеет следующий формат:

```bash
chatdown sample.chat > sample.transcript
```

Это действие принимает `sample.chat` и выводит `sample.transcript`. Дополнительные сведения см. в документации по [CLI Chatdown][chatdown].

## <a name="build"></a>Создание
### <a name="create-a-luis-application-with-ludown"></a>Создание приложения LUIS с помощью LUDown
Средство LUDown можно использовать для создания моделей JSON как для LUIS, так и для QnA.  
[Намерения](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents) и [сущности](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities) определяются для приложения LUIS точно так же, как и на портале LUIS.

#\<intent-name\> описывает новый раздел определения намерений. Затем каждая строка перечисляет [высказывания](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances), которые описывают это намерение.

Например, в одном файле LU можно создать множество намерений LUIS, как показано ниже. 

```LUDown
# Greeting
- Hi
- Hello
- Good morning
- Good evening

# Help
- help
- I need help
- please help
```

### <a name="create-qna-pairs-with-ludown"></a>Создание пар QnA в LUDown

С помощью следующей нотации формат файла LU также поддерживает пары QnA: 

```LUDown
> comment
### ? question ?
  ```markdown
    answer
  ```

Средство LUDown автоматически разделит вопросы и ответы и поместит их в файл qnamaker формата JSON, который затем можно будет использовать для создания базы знаний [QnaMaker.ai](http://qnamaker.ai).

```LUDown
### ? How do I change the default message for QnA Maker?
  ```markdown
  You can change the default message if you use the QnAMakerDialog. 
  See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
  ```

Кроме того, к одному ответу можно добавить несколько вопросов, просто добавляя новые строки вариантов вопросов в один ответ.

```LUDown
### ? What is your name?
- What should I call you?
  ```markdown
    I'm the echoBot! Nice to meet you.
  ```

### <a name="generate-json-models-with-ludown"></a>Создание моделей в формате JSON с помощью LUDown

После определения компонентов языка LUIS или QnA в формате LU их можно опубликовать в файле LUIS.json, QnA.json или QnA.tsv. При запуске средство LUDown ищет в рабочем каталоге любые файлы с форматом LU для анализа. Так как средство LUDown может использовать файлы с расширением LU как для LUIS, так и для QnA, просто требуется указать, для какой языковой службы будут создаваться файлы. Для этого можно использовать общую команду **ludown parse<Service> -- in <luFile>**. 

В приведенном примере в каталоге находятся два файла с расширением LU, предназначенные для анализа. Файл "1.lu" используется для создания модели LUIS, а файл "qna1.lu" для создания базы знаний QnA.

#### <a name="generate-luis-json-models"></a>Создание моделей LUIS в формате JSON

Чтобы создать модель LUIS с помощью LUDown, введите следующую команду в текущем рабочем каталоге:

```shell
ludown parse ToLuis --in <luFile>
```

#### <a name="generate-qna-knowledge-base"></a>Создание базы знаний QnA

Аналогичным образом вам достаточно только изменить целевой объект анализа для создания базы знаний QnA.

```shell
ludown parse ToQna --in <luFile> 
```

Полученные файлы JSON могут быть использованы LUIS и QnA либо через соответствующие порталы, либо через новые средства CLI. Дополнительные сведения можно найти в разделе репозитория GitHub для [LUdown CLI][ludown].

### <a name="track-service-references-using-bot-file"></a>Отслеживание ссылок на службы с помощью BOT-файла

Новое средство [MSBot][msbotCli] позволяет создавать файл с расширением **.bot**, в котором хранятся метаданные о службах, используемых ботом. C помощью этого файла бот может использовать интерфейс командной строки для подключения данных служб. Средство доступно в качестве модуля npm. Чтобы его установить, необходимо выполнить следующую команду.

```shell
npm install -g msbot
```

Чтобы создать файл бота, в интерфейсе командной строки введите команду **msbot init**, за которой следует имя бота и конечная точка целевого URL-адреса, например:

```shell
msbot init --name TestBot --endpoint http://localhost:9499/api/messages
```
Чтобы подключить бот к службе, в интерфейс командной строки необходимо ввести команду **msbot connect**, за которой следует имя соответствующей службы:

```shell
msbot connect [Service]
```

Чтобы получить список поддерживаемых служб, изучите файл [readme][msbotCli].

### <a name="create-and-manage-luis-applications-using-luis-cli"></a>Создание приложений LUIS и управление ими с помощью LUIS CLI

Теперь в новый набор средств добавлено [расширение LUIS][luisCli], которое позволяет самостоятельно управлять ресурсами LUIS. Оно доступно в качестве модуля npm, который можно скачать:

```shell
npm install -g luis-apis
```
Основные команды, используемые средством LUIS из интерфейса командной строки, приведены далее.

```shell
luis <action> <resource> <args...>
```
Чтобы подключить бот к LUIS, необходимо создать файл в формате **LUISRC**. Когда приложение выполняет исходящие вызовы, этот файл конфигурации устанавливает идентификатор приложения LUIS и пароль конечной точки службы. Выполнение команды **luis init** позволит создать этот файл, как показано в примере.

```shell
luis init
```
Прежде чем средство создаст файл, в окне терминала пользователю будет предложено ввести ключ разработки LUIS, регион и идентификатор приложения.  

После создания этого файла ваше приложение сможет использовать JSON-файл LUIS (сгенерированный из LUDown), используя следующую команду из интерфейса командной строки.

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```
Дополнительные сведения можно найти в разделе репозитория GitHub для [LUIS CLI][luisCli].

### <a name="create-qna-maker-kb-using-qna-maker-cli"></a>Создание базы знаний QnA Maker с помощью QnA Maker CLI

Теперь в набор средств добавлено [расширение QnA][qnaCli], которое позволяет самостоятельно управлять ресурсами LUIS. Оно доступно в качестве модуля npm, который можно скачать.

```shell
npm install -g qnamaker
```
С помощью средства QnA Maker можно создавать, обновлять, публиковать, удалять и обучать базу знаний. Можно использовать файлы, созданные с помощью команды [ludown parse toqna](#generate-qna-knowledge-base), чтобы создать или заменить базу знаний.

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

Дополнительные сведения можно найти в разделе репозитория GitHub для [QnA Maker CLI][qnaCli].

### <a name="create-dispatch-model-using-dispatch-cli"></a>Создание модели диспетчеризации с помощью интерфейса командной строки в Dispatch

Средство Dispatch позволяет создавать и оценивать модели LUIS, используемая для диспетчеризации намерений на несколько модулей чат-ботов, таких как модели LUIS, базы знаний QnA и т. п. (их можно добавить как тип файла для диспетчеризации).

Модель Dispatch удобно использовать в следующих случаях:

- Бот состоит из нескольких модулей, и вам нужна помощь в передаче высказываний пользователя между модулями и в оценке качества интеграции бота.
- Оцените качество классификации намерений для одной модели LUIS.
- Создайте модель классификации текста на основе текстовых файлов.

Включив в BOT-файл все нужные ему [приложения LUIS][msbotCli-luis] и [базы знаний QnA Maker][msbotCli-qna], вы можете выполнить сборку диспетчеризации следующим образом: 

```shell
dispatch create -b <YOUR-BOT-FILE> | msbot connect dispatch --stdin
```
Подробнее см. документацию по [интерфейсу командной строки в Disptach][dispatchCli].

## <a name="test"></a>Тест

[Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases) — это классическое приложение, которое позволяет разработчикам локально или удаленно (через туннель) тестировать боты и (или) выполнять их отладку.

## <a name="publish"></a>Опубликовать

Вы можете создавать, скачивать и публиковать боты в службе Azure Bot с помощью Azure CLI. Установите расширение бота: 
```shell
az extension add -n botservice
```

### <a name="create-azure-bot-service-bot"></a>Создание бота в службе Azure Bot

Примечание. Необходимо установить последнюю версию `az cli`. Выполните обновление, чтобы использовать Azure CLI со средством MSBot. 

Войдите в учетную запись Azure: 
```shell
az login
```

Успешно выполнив вход, вы сможете создать новый бот в службе Azure Bot следующим образом: 
```shell
az bot create [options]
```

Чтобы создать бот и внести его конфигурацию в BOT-файл, выполните:  
```shell
az bot create [options] --msbot | msbot connect bot --stdin
```

Если уже есть существующий бот:  
```shell
az bot show [options] --msbot | msbot connect bot --stdin
```

| Параметр                            | ОПИСАНИЕ                                   |
|-----------------------------------|-----------------------------------------------|
| --kind -k [обязательный параметр]              | Вид бота.  Допустимые значения: function (функция), registration (регистрация), webapp (веб-приложение).|
| --name -n [обязательный параметр]              | Имя ресурса для бота. |
| --appid                           | Идентификатор учетной записи MSA, которая будет использоваться с ботом.   |
| --location -l                     | Расположение. Расположение по умолчанию можно настроить с помощью `az configure --defaults location=<location>`.  Значение по умолчанию: westus.|
| --msbot                           | Вывод результатов в формате JSON, совместимом с файлом .bot.  Допустимые значения: false, true.|
| --password -p                     | Пароль MSA для бота, полученный на портале разработчика. |
| --resource-group -g               | Имя группы ресурсов. Вы можете настроить расположение по умолчанию с помощью `az configure --defaults group=<name>`.  Значение по умолчанию: build2018. |
| --tags                            | Набор тегов, которые нужно добавить к боту. |


### <a name="configure-channels"></a>Настройка каналов

С помощью Azure CLI вы можете управлять каналами для вашего бота. 
```shell
>az bot -h
Group
   az bot: Manage Bot Services.
    Subgroups:
        directline: Manage Directline Channel on a Bot.
        email     : Manage Email Channel on a Bot.
        facebook  : Manage Facebook Channel on a Bot.
        kik       : Manage Kik Channel on a Bot.
        msteams   : Manage Msteams Channel on a Bot.
        skype     : Manage Skype Channel on a Bot.
        slack     : Manage Slack Channel on a Bot.
        sms       : Manage Sms Channel on a Bot.
        telegram  : Manage Telegram Channel on a Bot.
        webchat   : Manage Webchat Channel on a Bot.

    Commands:
        create    : Create a new Bot Service.
        delete    : Delete an existing Bot Service.
        download  : Download an existing Bot Service.
        publish   : Publish to an existing Bot Service.
        show      : Get an existing Bot Service.
        update    : Update an existing Bot Service.

```

## <a name="additional-information"></a>Дополнительная информация
- [Средства Bot Builder на сайте GitHub][cliTools]

<!-- Footnote links -->

[cliTools]: https://aka.ms/botbuilder-tools-readme
[azureCli]: https://aka.ms/botbuilder-tools-azureCli
[msbotCli]: https://aka.ms/botbuilder-tools-msbot-readme
[msbotCli-luis]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-luis-application
[msbotCli-qna]: https://aka.ms/botbuilder-tools-msbot-readme#connecting-to-qna-maker-knowledge-base
[chatdown]: https://aka.ms/botbuilder-tools-chatdown
[ludown]: https://aka.ms/botbuilder-ludown
[luisCli]: https://aka.ms/botbuilder-luis-cli
[qnaCli]: https://aka.ms/botbuilder-tools-qnaMaker
[dispatchCli]: https://aka.ms/botbuilder-tools-dispatch

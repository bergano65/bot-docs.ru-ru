---
title: Создание ботов с помощью шаблонов Bot Builder
description: Средства Bot Builder позволяют управлять ресурсами бота непосредственно из командной строки
keywords: botbuilder templates, node.js, python, java, .net, ludown, yeoman
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
ms.openlocfilehash: 60cdc3de200336b00173749a553205a47a32457e
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39300702"
---
# <a name="create-bots-with-botbuilder-templates"></a>Создание ботов с помощью шаблонов Bot Builder

Теперь доступны шаблоны для создания ботов на каждой платформе SDK Bot Builder: 

- Node.js
- Python
- Java
- .NET

## <a name="prerequisites"></a>Предварительные требования

Все шаблоны Node.js, Java и Python создают простой бот echo. Доступ ко всем им можно получить через [Yeoman](http://yeoman.io/).

- [Node.js](https://nodejs.org/en/) (версии 8.5 или более новой)
- [Yeoman](http://yeoman.io/)

Если вы еще не установили Yeoman, его необходимо установить глобально.

```shell
npm install -g yo
```

## <a name="nodejs-python-java-templates"></a>Шаблоны Node.js, Python, Java
Из командной строки с помощью команды cd перейдите в новую папку на ваш выбор. Существуют две доступные версии шаблонов Node.js для Bot Builder, **версия 3** и **версия 4**, предназначенные для соответствующих версий пакета SDK. Шаблоны Python и Java доступны только в версии 4. 

Чтобы установить генератор шаблонов **Node.js версии 3 для Bot Builder**, сделайте следующее:

```shell
npm install generator-botbuilder
```

Чтобы установить генератор шаблонов **Node.js версии 4 для Bot Builder**, сделайте следующее:
```shell
npm install generator-botbuilder@preview
```

Шаблоны Python и Java **доступны только в версии 4**. 

Для **Python**:
```shell
npm install generator-botbuilder-python
```

Для **Java**:
```shell
npm install generator-botbuilder-java
```

После установки генератора можно просто выполнить команду **yo** в интерфейсе командной строки для просмотра списка доступных генераторов в Yeoman.

![Интерфейс Yeoman](media/botbuilder-templates/yeoman-generator-botbuilder.png)

Переключитесь на рабочий каталог по своему усмотрению и выберите генератор для использования. Будет предложено настроить разные параметры создания ботов, например имя и описание. Когда все запросы будут завершены, шаблон бота echo создастся в той же рабочей папке.

![Шаблон Node.js в Yeoman](media/botbuilder-templates/new-template-node.png)


## <a name="net"></a>.NET

Доступны два шаблона для .NET, **версия 3** и **версия 4**, предназначенные для соответствующих версий пакета SDK. Оба шаблона доступны как пакеты [VSIX](https://docs.microsoft.com/en-us/visualstudio/extensibility/anatomy-of-a-vsix-package). Чтобы скачать, щелкните одну из следующих ссылок на [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

- [Шаблон Bot Builder версии 3](https://marketplace.visualstudio.com/items?itemName=BotBuilder.BotBuilderV3)
- [Шаблон Bot Builder версии 4](https://aka.ms/Ylcwxk)

#### <a name="prerequisites"></a>Предварительные требования

- [Visual Studio 2015 или более поздней версии](https://www.visualstudio.com/downloads/)
- [Учетная запись Azure](https://azure.microsoft.com/en-us/free/)

### <a name="install-the-templates"></a>Установка шаблонов

Просто откройте пакет VSIX из сохраненного каталога, и шаблон Bot Builder установится в Visual Studio и будет доступным при следующем открытии.

![Пакет VSIX](media/botbuilder-templates/botbuilder-vsix-template.png)

Чтобы создать проект бота, используя шаблон, просто откройте Visual Studio и выберите **Файл** > **новый** > **Проект**, а из Visual C# выберите **Bot Framework** > Simple Echo Bot Application (Простое приложение-бот Echo). При этом будет локально создан проект бота echo, который можно изменить по своему усмотрению. 

![Шаблон бота .NET](media/botbuilder-templates/new-template-dotnet.png)

## <a name="store-your-bot-information-with-msbot"></a>Хранение информации бота с помощью MSBot

Новый инструмент [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) позволяет создавать **BOT**-файл, в котором хранятся метаданные о службах, используемых ботом. C помощью этого файла бот может использовать интерфейс командной строки для подключения данных служб. Средство доступно в качестве модуля npm. Чтобы его установить, необходимо выполнить следующую команду:

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

| Service | ОПИСАНИЕ |
| ------ | ----------- |
| azure  |Подключение бота к регистрации службы Azure Bot|
|localhost| Подключение бота к конечной точке localhost|
|luis     | Подключение бота к приложению LUIS |
| qna     |Подключение бота к базе знаний QnA|
|help [cmd]  |Отображение справки по [cmd]|

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>Подключение бота к ABS с помощью BOT-файла

После установки средства MSBot можно легко подключить бот к имеющейся группе ресурсов службы Azure Bot. Для этого необходимо выполнить команду az bot **show**. 

```azurecli
az bot show -n <botname> -g <resourcegroup> --msbot | msbot connect azure --stdin
```

Для этого из целевой группы ресурсов будут использованы текущая конечная точка, идентификатор приложения MSA и пароль и соответствующим образом обновлена информация о них в BOT-файле. 


## <a name="manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>Обновление или создание служб LUIS и QnA и управление ими с помощью новых средств Bot Builder

[Средства Bot Builder](https://github.com/microsoft/botbuilder-tools) — это новый набор средств для взаимодействия с ресурсами бота и управления ими непосредственно из командной строки. 

>[!TIP]
> Каждое средство Bot Builder содержит команду глобальной справки, доступную в командной строке после ввода **-h** или **--help**. Эта команда доступна в любое время из любого действия, которое обеспечивает отображение полезных вариантов, доступных вместе с их описанием. 

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown) позволяет описывать и создавать мощные компоненты языка для ботов, использующих файлы с расширением **LU**. Новые файлы с форматом LU, который является типом формата разметки, используются средством LUDown для вывода файлов JSON, указанных в целевой службе. В настоящее время файлы с форматом LU можно использовать для создания приложения [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-get-started-create-app) или базы знаний [QnA](https://qnamaker.ai/Documentation/CreateKb) с использованием разных форматов. LUDown доступно в качестве модуля npm и может быть использовано при глобальной установке на компьютер:

```shell
npm install -g ludown
```
Средство LUDown можно использовать для создания моделей JSON как для LUIS, так и для QnA.  


### <a name="creating-a-luis-application-with-ludown"></a>Создание приложения LUIS в LUDown

[Намерения](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents) и [сущности](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities) определяются для приложения LUIS точно так же, как и для портала LUIS. 

#\<intent-name\> описывает новый раздел определения намерений. Затем каждая строка перечисляет [высказывания](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances), которые описывают это намерение.

Например, в одном файле LU можно создать множество намерений LUIS: 

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

### <a name="qna-pairs-with-ludown"></a>Пары QnA в LUDown

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

### <a name="generating-json-models-with-ludown"></a>Создание моделей в формате JSON с помощью LUDown

После определения компонентов языка LUIS или QnA в формате LU их можно опубликовать в файле LUIS.json, QnA.json или QnA.tsv. При запуске средство LUDown ищет в рабочем каталоге любые файлы с форматом LU для анализа. Так как средство LUDown может использовать файлы с расширением LU как для LUIS, так и для QnA, просто требуется указать, для какой языковой службы будут создаваться файлы. Для этого можно использовать общую команду **ludown parse<Service> -- in <luFile>**. 

В приведенном примере в каталоге находятся два файла с расширением LU, предназначенные для анализа. Файл "1.lu" используется для создания модели LUIS, а файл "qna1.lu" для создания базы знаний QnA.

#### <a name="generate-luis-json-models"></a>Создание моделей LUIS в формате JSON

Чтобы создать модель LUIS с помощью LUDown, введите следующую команду в текущем рабочем каталоге:

```shell
ludown parse ToLuis --in <luFile> 
```

#### <a name="generate-qna-knowledge-base-json"></a>Создание базы знаний QnA в формате JSON

Аналогичным образом вам достаточно только изменить целевой объект анализа для создания базы знаний QnA. 

```shell
ludown parse ToQna --in <luFile> 
```

Полученные файлы JSON могут быть использованы LUIS и QnA либо через соответствующие порталы, либо через новые инструменты интерфейса командной строки. 

## <a name="connect-to-luis-an-qna-maker-services-from-the-cli"></a>Подключение к службам QnA Maker и LUIS из интерфейса командной строки

### <a name="connect-to-luis-from-the-cli"></a>Подключение к LUIS из интерфейса командной строки 

Теперь в набор средств добавлено [расширение LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS), которое позволяет самостоятельно управлять ресурсами LUIS. Оно доступно в качестве модуля npm, который можно скачать:

```shell
npm install -g luis-apis
```
Основные команды, используемые средством LUIS из интерфейса командной строки, приведены далее:

```shell
luis <action> <resource> <args...>
```
Чтобы подключить бот к LUIS, необходимо создать файл в формате **LUISRC**. Когда приложение выполняет исходящие вызовы, этот файл конфигурации подготавливает к работе идентификатор приложения LUIS и пароль для конечной точки службы. При выполнении команды **luis init** этот файл будет создан:

```shell
luis init
```
Прежде чем средство создаст файл, в окне терминала пользователю будет предложено ввести ключ разработки LUIS, регион и идентификатор приложения.  

![Команда LUIS init](media/bot-builder-tools/luis-init.png) 


После создания этого файла ваше приложение сможет использовать JSON-файл LUIS (сгенерированный из LUDown), используя следующую команду из интерфейса командной строки.

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>Подключение к QnA из интерфейса командной строки

Теперь в набор средств добавлено [расширение QnA](https://github.com/Microsoft/botbuilder-tools/tree/master/QnAMaker), которое позволяет самостоятельно управлять ресурсами LUIS. Оно доступно в качестве модуля npm, который можно скачать.

```shell
npm install -g qnamaker
```
С помощью средства QnA Maker можно создавать, обновлять, публиковать, удалять и обучать базу знаний. Для начала необходимо создать файл с расширением **QNAMAKERRC**, который требуется для включения конечной точки в службу. Этот файл можно легко создать, если выполнить команду **qnamaker init**, следовать запросам и подготовить идентификатор базы знаний QnA Maker. 

```shell
qnamaker init 
```
![Команда QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

Использовав следующую команду, после создания файла с расширением QNAMAKERRC можно подключиться к базе знаний QnA, что даст возможность использовать файлы базы данных (созданные в LUDown) в формате JSON и TSV.

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="references"></a>Ссылки
- [Bot Builder tools (PREVIEW)](https://github.com/Microsoft/botbuilder-tools) (Средства Bot Builder (предварительная версия))
- [MSBot Command Line tool](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) (Программа командной строки MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown)
- [интерфейс командной строки Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

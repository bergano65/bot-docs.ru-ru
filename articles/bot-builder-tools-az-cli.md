---
title: Создание ботов с помощью Azure CLI
description: Средства Bot Builder позволяют управлять ресурсами бота непосредственно из командной строки
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: tools
ms.date: 08/31/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b614b11852516ec8dd426d210aacc85a0f39c813
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49999421"
---
# <a name="create-bots-with-azure-cli"></a>Создание ботов с помощью Azure CLI

[!INCLUDE [pre-release-label](./includes/pre-release-label-v3.md)]

Из этого руководства вы узнаете, как выполнять следующие задачи: 
- Создать бота с помощью Azure CLI 
- Скачать локальную копию для разработки
- Использовать новое средство MSBot для хранения всей информации о ресурсе бота
- Управлять, создавать или обновлять модели LUIS и QnA с помощью LUDown
- Подключаться к службам QnA Maker и LUIS из интерфейса командной строки
- Развертывать бота из интерфейса командной строки в Azure

## <a name="prerequisites"></a>Предварительные требования

Чтобы использовать эти средства из командной строки, понадобится установить Node.js. 
- [Node.js (версии 8.5 или более поздней)](https://nodejs.org/en/)

## <a name="1-install-tools"></a>1. Средства установки
1. [Установите](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) последнюю версию Azure CLI.
2. [Установите](https://aka.ms/botbuilder-tools-readme) средства для Bot Builder.

Теперь с помощью Azure CLI можно управлять ботами, как и любым другим ресурсом в Azure.

>[!TIP]
> На данный момент расширение службы Azure Bot поддерживает только ботов v3.
  
3. Выполните вход в Azure CLI, выполнив следующую команду.

```azurecli
az login
```
Она откроет окно браузера с интерфейсом для входа. После успешного входа вы увидите следующее сообщение:

![Имя пользователя устройства Майкрософт](media/bot-builder-tools/az-browser-login.png)

А в окне командной строки будет представлена следующая информация:

![Команда для входа в Azure](media/bot-builder-tools/az-login-command.png)

## <a name="2-create-a-new-bot-from-azure-cli"></a>2. Создание бота в Azure CLI

С помощью Azure CLI ботов можно создавать полностью из командной строки. 

```azurecli
az bot [command]
```
|Команды|  |
|----|----|
| create      |добавить ресурс|
| удалить     |клонировать ресурс|
| загрузить   | загрузить исходный код бота|
| Опубликовать   |публиковать в существующей службе бота|
| show |показать существующие ресурсы бота.|
| обновить| Обновление существующей службы бота|

Чтобы создать бот в интерфейсе командной строки, необходимо выбрать существующую [группу ресурсов](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) или создать новую. 

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --version v3 --description "description-of-my-bot"
```
Поддерживаются следующие значения: `function, registration, webapp` для `--kind` и `v3, v4` для `--version`.  Сообщение с подтверждением появится после успешного выполнения запроса.
```
Obtained msa app id and password. Provisioning bot now.
```

> [!TIP]
> Если вы получили сообщение об ошибке, в котором указано, что **группа ресурсов не найдена**, вам может потребоваться установить [подписку](https://docs.microsoft.com/en-us/azure/architecture/cloud-adoption-guide/adoption-intro/subscription-explainer) в Azure CLI. Подписка Azure должна соответствовать той, которая была введена при создании группы ресурсов. Чтобы ее настроить введите следующую команду.
> ```azurecli
> az account set --subscription "your-subscription-name"
> ```
> Чтобы просматривать списки подписок учетной записи, необходимо ввести следующую команду.
> ```azurecli
> az account list
> ```

По умолчанию будет создан бот .NET. Указав язык, с помощью аргумента **-- lang**, также можно указать платформу пакета SDK. В настоящее время пакет расширений ботов поддерживает пакеты SDK Bot Builder для C# и Node.js. Например, чтобы **создать бот Node.js**, используется следующая команда.

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot" --lang Node 
```
Новый эхо-бот будет предоставлен группе ресурсов Azure. Чтобы его проверить, просто выберите **Test в Webchat** (Тестирование в веб-чате) под заголовком управления ботом в представлении бота веб-приложения. 

![Эхо-бот Azure](media/bot-builder-tools/az-echo-bot.png) 

## <a name="3-download-the-bot-locally"></a>3. Локальная загрузка бота

Есть два способа загрузить исходный код:
- на портале Azure;
- с помощью Azure CLI.

Чтобы скачать исходный код бота с [портала Azure](http://portal.azure.com), выберите нужный ресурс бота и щелкните **Cборка** в разделе управления ботом. Существует несколько различных вариантов, доступных для управления или получения исходного кода бота локально.

![Загрузка бота с портала Azure](media/bot-builder-tools/az-portal-manage-code.png)

Чтобы загрузить источник бота с помощью интерфейса командной строки, введите следующую команду. Бот будет скачан в локальную систему.

```azurecli
az bot download --name "my-bot-name" --resource-group "my-resource-group"
```

![Команда загрузки в интерфейсе командной строки](media/bot-builder-tools/cli-bot-download-command.png)

## <a name="4-store-your-bot-information-with-msbot"></a>4. Хранение информации бота с помощью MSBot

Новое средство MSBot позволяет создавать файл с расширением **.bot**, где в едином месте хранятся метаданные о всех используемых ботом службах. C помощью этого файла бот может использовать интерфейс командной строки для подключения данных служб. Средства MSBot поддерживают несколько команд. Подробное описание см в файле [readme](https://aka.ms/botbuilder-tools-msbot-readme). 

Чтобы установить MSBot, выполните следующую команду:

```shell
npm install -g msbot
```

Чтобы создать файл бота, в интерфейсе командной строки введите команду **msbot init**, за которой следует имя бота и конечная точка целевого URL-адреса, например:

```shell
msbot init --name name-of-my-bot --endpoint http://localhost:bot-port-number/api/messages
```

Чтобы подключить бот к службе, в интерфейс командной строки необходимо ввести команду **msbot connect**, за которой следует имя соответствующей службы:

```shell
msbot connect service-type
```

| Service type (Тип службы) | ОПИСАНИЕ |
| ------ | ----------- |
| azure  |Подключение бота к регистрации службы Azure Bot|
|endpoint| подключает бот к конечной точке, например localhost|
|luis     | Подключение бота к приложению LUIS |
| qna     |Подключение бота к базе знаний QnA|
|help [cmd]  |Отображение справки по [cmd]|

Полный список поддерживаемых служб вы найдете в файле [readme](https://aka.ms/botbuilder-tools-msbot-readme).

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>Подключение бота к ABS с помощью BOT-файла

После установки средства MSBot можно легко подключить бот к существующей группе ресурсов службы Azure Bot. Для этого необходимо выполнить команду az bot **show**.

```azurecli
az bot show -n my-bot-name -g my-resource-group --msbot | msbot connect azure --stdin
```

Эта команда возьмет текущую конечную точку, идентификатор приложения MSA и пароль целевой группы ресурсов и обновит информацию о них в файле BOT.


## <a name="5-manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>5. Обновление или создание служб LUIS и QnA и управление ими с помощью новых средств Bot Builder

[Средства Bot Builder](https://aka.ms/botbuilder-tools) — это новый набор средств для взаимодействия с ресурсами бота и управления ими непосредственно из командной строки.

>[!TIP]
> Каждое средство Bot Builder содержит команду глобальной справки, доступ к которой можно получить из командной строки посредством ввода **-h** или **--help**. Эта команда доступна в любое время из любого действия, что обеспечивает отображение доступных вариантов вместе с их описанием.

### <a name="ludown"></a>LUDown

[LUDown](https://aka.ms/botbuilder-ludown) позволяет описывать и создавать мощные компоненты языка для ботов с помощью файлов с расширением **LU**. Файл нового формата LU является типом формата разметки Markdown, который используется средством LUDown для вывода файлов JSON, адаптированных для целевой службы. В настоящее время файлы с форматом LU можно использовать для создания приложения [LUIS](https://docs.microsoft.com/azure/cognitive-services/luis/luis-get-started-create-app) или базы знаний [QnA](https://qnamaker.ai/Documentation/CreateKb) с использованием разных форматов. LUDown доступен в виде модуля npm и может быть использован при глобальной установке на компьютер.

```shell
npm install -g ludown
```

Средство LUDown можно использовать для создания моделей JSON как для LUIS, так и для QnA.  

### <a name="creating-a-luis-application-with-ludown"></a>Создание приложения LUIS с помощью LUDown

[Намерения](https://docs.microsoft.com/azure/cognitive-services/luis/add-intents) и [сущности](https://docs.microsoft.com/azure/cognitive-services/luis/add-entities) определяются для приложения LUIS точно так же, как и на портале LUIS.

`# \<intent-name\>` описывает новый раздел определения намерений. Последующие строки содержат [высказывания](https://docs.microsoft.com/azure/cognitive-services/luis/add-example-utterances), которые описывают это намерение.

Например, в одном файле LU можно создать множество намерений LUIS, как показано ниже. 

```ludown
# Greeting
Hi
Hello
Good morning
Good evening

# Help
help
I need help
please help
```

### <a name="qna-pairs-with-ludown"></a>Пары QnA в LUDown

С помощью следующей нотации формат файла LU также поддерживает пары QnA: 

  ```ludown
  > This is a comment. QnA definitions have the general format:
  ### ? this-is-the-question-string
  - this-is-an-alternate-form-of-the-same-question
  - this-is-another-one
    ```markdown
    this-is-the-answer
    ```
  ```
Средство LUDown автоматически разделит вопросы и ответы и поместит их в файл qnamaker формата JSON, который затем можно будет использовать для создания базы знаний [QnaMaker.ai](http://qnamaker.ai).

  ```ludown
  ### ? How do I change the default message for QnA Maker?
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

Кроме того, к одному ответу можно добавить несколько вопросов, просто добавляя новые строки вариантов вопросов в один ответ. 

  ```ludown
  ### ? What is your name?
  - What should I call you?
    ```markdown
    I'm the echoBot! Nice to meet you.
    ```
  ```

### <a name="generating-json-models-with-ludown"></a>Создание моделей в формате JSON с помощью LUDown

После определения компонентов языка LUIS или QnA в формате LU их можно опубликовать в файле LUIS.json, QnA.json или QnA.tsv. При запуске средство LUDown ищет в рабочей папке любые файлы для анализа с форматом LU. Поскольку средство LUDown может ориентироваться как на LUIS, так и на QnA файлы с расширением LU, просто требуется указать для какой языковой службы он будет создаваться. Для этого можно использовать общую команду **ludown parse<Service>--in<luFile>**. 

В приведенном примере в каталоге находятся два файла LU, предназначенные для синтаксического анализа. Файл "luis-sample.lu" используется для создания модели LUIS, а файл "qna-sample.lu" для создания базы знаний QnA.


#### <a name="generate-luis-json-models"></a>Создание моделей LUIS в формате JSON

**luis-sample.lu** 
```ludown
# Greeting
- Hi
- Hello
- Good morning
- Good evening
```

Чтобы создать модель LUIS с помощью LUDown в текущем рабочем каталоге, введите следующую команду.

```shell
ludown parse ToLuis --in ludown-file-name.lu
```

#### <a name="generate-qna-knowledge-base-json"></a>Создание базы знаний QnA в формате JSON

**qna-sample.lu**
  ```ludown
  > This is a sample ludown file for QnA Maker.

  ### ? How do I change the default message
    ```markdown
    You can change the default message if you use the QnAMakerDialog. 
    See [this link](https://docs.botframework.com/en-us/azure-bot-service/templates/qnamaker/#navtitle) for details. 
    ```
  ```

Аналогичным образом вам достаточно только изменить целевой объект синтаксического анализа для создания базы знаний QnA. 

```shell
ludown parse ToQna --in ludown-file-name.lu
```

Полученные файлы JSON могут быть использованы LUIS и QnA либо через соответствующие порталы, либо через новые средства CLI. 

## <a name="6-connect-to-luis-an-qna-maker-services-from-the-cli"></a>6. Подключение к службам QnA Maker и LUIS из интерфейса командной строки

### <a name="connect-to-luis-from-the-cli"></a>Подключение к LUIS из интерфейса командной строки 

Теперь в набор средств добавлено [расширение LUIS](https://aka.ms/botbuilder-luis-cli), которое позволяет самостоятельно управлять ресурсами LUIS. Оно доступно в качестве модуля npm, который можно скачать:

```shell
npm install -g luis-apis
```
Основные команды, используемые средством LUIS из интерфейса командной строки, приведены далее.

```shell
luis action-name resource-name arguments-list
```
Чтобы подключить бот к LUIS, необходимо создать файл в формате **LUISRC**. Когда приложение выполняет исходящие вызовы, этот файл конфигурации устанавливает идентификатор приложения LUIS и пароль конечной точки службы. Выполнение команды **luis init** позволит создать этот файл, как показано в примере.

```shell
luis init
```
Прежде чем средство создаст файл, в окне терминала пользователю будет предложено ввести ключ разработки LUIS, регион и идентификатор приложения.  

![Команда LUIS init](media/bot-builder-tools/luis-init.png) 


Используя следующую команду в интерфейсе командной строки после создания этого файла, приложение сможет использовать файл LUIS .json (созданный в LUDown). 

```shell
luis import application --in luis-app.json | msbot connect luis --stdin
```

### <a name="connect-to-qna-from-the-cli"></a>Подключение к QnA из интерфейса командной строки

Теперь в набор средств добавлено [расширение QnA](https://aka.ms/botbuilder-tools-qnaMaker), которое позволяет самостоятельно управлять ресурсами LUIS. Оно доступно в качестве модуля npm, который можно скачать.

```shell
npm install -g qnamaker
```
С помощью средства QnA Maker можно создавать, обновлять, публиковать, удалять и обучать базу знаний. Для начала необходимо создать файл с расширением **.qnamakerrc**, который требуется для включения конечной точки в службу. Этот файл можно легко создать если выполнить команду **qnamaker init** и следовать запросам идентификатора базы знаний QnA Maker. 

```shell
qnamaker init 
```
![Команда QnaMaker init](media/bot-builder-tools/qnamaker-init.png)

Использовав следующую команду после создания файла .qnamakerrc, можно подключиться к базе знаний QnA, что даст возможность использовать файлы базы знаний (созданные в LUDown) в формате JSON и TSV.

```shell
qnamaker create --in qnaKB.json --msbot | msbot connect qna --stdin
```

## <a name="7-publish-to-azure-from-the-cli"></a>7. Отправка из интерфейса командной строки в Azure

После внесения изменений в исходный код вашего бота с помощью следующей команды изменения можно легко опубликовать.

```azurecli
az bot publish --name "my-bot-name" --resource-group "my-resource-group"
```

## <a name="references"></a>Ссылки
- [Bot Builder Tools](https://aka.ms/botbuilder-tools-readme) (Средства для Bot Builder)
- [Интерфейс командной строки Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)

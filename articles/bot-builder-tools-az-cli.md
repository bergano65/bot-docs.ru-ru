---
title: Создание ботов с помощью Azure CLI
description: Средства Bot Builder позволяют управлять ресурсами бота непосредственно из командной строки
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 04/25/2018
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 20258949cd8ea403e5cc9bf774d6a3b7c1e86e7e
ms.sourcegitcommit: dcbc8ad992a3e242a11ebcdf0ee99714d919a877
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/30/2018
ms.locfileid: "39352903"
---
# <a name="create-bots-with-azure-cli"></a>Создание ботов с помощью Azure CLI

[Средства Bot Builder](https://github.com/microsoft/botbuilder-tools) — это новый набор средств для управления и взаимодействия с ресурсами бота непосредственно из командной строки. 

Из этого руководства можно узнать, как выполнять следующие действия.

- Включить расширение бота Azure CLI
- Создать бота с помощью Azure CLI 
- Скачать локальную копию для разработки
- Использовать новое средство MSBot для хранения всей информации о ресурсе бота
- Управлять, создавать или обновлять модели LUIS и QnA с помощью LUDown
- Подключаться к службам QnA Maker и LUIS из интерфейса командной строки
- Развертывать бота из интерфейса командной строки в Azure

## <a name="prerequisites"></a>Предварительные требования

Чтобы включить эти средства из командной строки, понадобится установить Node.js. 

- [Node.js (версии 8.5 или более поздней)](https://nodejs.org/en/)

## <a name="1-enable-azure-cli"></a>1. Включение Azure CLI

Теперь с помощью [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/?view=azure-cli-latest) можно управлять ботами, как и любым другим ресурсом в Azure. Чтобы включить Azure CLI, необходимо выполнить следующие действия.

1. [Скачать](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) Azure CLI, если у вас его еще нет. 

2. Ввести следующую команду для загрузки пакета dist расширения службы Azure Bot.

```azurecli
az extension add -n botservice
```

>[!TIP]
> На данный момент расширение службы Azure Bot поддерживает только ботов v3.
  
3. Чтобы [войти](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest)в Azure CLI, выполните следующую команду.

```azurecli
az login
```
Вам будет предложено ввести уникальный временный код. Чтобы войти, необходимо использовать веб-браузер и посетить страницу Майкрософт [Имя пользователя устройства](https://microsoft.com/devicelogin), где для продолжения требуется ввести код, предоставленный интерфейсом командной строки. 

![Имя пользователя устройства Майкрософт](media/bot-builder-tools/ms-device-login.png)

После успешного входа появится экран приветствия Azure CLI, а также список доступных параметров управления учетной записью и ресурсами.

![Интерфейс командной строки бота в Azure](media/bot-builder-tools/az-cli-bot.png)


 Список всех команд Azure CLI можно узнать на [этой странице](https://docs.microsoft.com/en-us/cli/azure/reference-index?view=azure-cli-latest).


## <a name="2-create-a-new-bot-from-azure-cli"></a>2. Создание бота в Azure CLI

С помощью Azure CLI и нового расширение бота создание ботов может целиком проходить в командной строке. 

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

Чтобы создать бот в интерфейсе командной строки, необходимо выбрать существующую [группу ресурсов](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview) или создать новую. 

```azurecli
az bot create --resource-group "my-resource-group" --name "my-bot-name" --kind "my-resource-type" --description "description-of-my-bot"
```
Сообщение с подтверждением появится после успешного выполнения запроса.
```
obtained msa app id and password. Provisioning bot now.
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

Для новых ботов существует два способа загрузки исходного кода.
- Загрузить из портала Azure.
- Загрузить с помощью Azure CLI.

Чтобы загрузить исходный код бота с портала, просто выберите ресурс бота, а затем в управлении ботом выберите **Cборка**. Существует несколько различных вариантов, доступных для управления или получения исходного кода бота локально. 

![Загрузка бота с портала Azure](media/bot-builder-tools/az-portal-manage-code.png)

Чтобы загрузить источник бота с помощью интерфейса командной строки, введите следующую команду. Бот будет загружен в подкаталог. Если подкаталога не существует, он будет создан командой.

```azurecli
az bot download --name "my-bot-name" --resource-group "my-resource-group"
```
Также можно указать каталог для загрузки бота.
Например: 

![Команда загрузки в интерфейсе командной строки](media/bot-builder-tools/cli-bot-download-command.png)

![Загрузка бота через интерфейс командной строки](media/bot-builder-tools/cli-bot-download.png)

Команда, приведенная выше, позволяет загрузить исходный код бота непосредственно в указанное расположение, что позволяет разрабатывать ботов на локальном компьютере.


## <a name="4-store-your-bot-information-with-msbot"></a>4. Хранение информации бота с помощью MSBot

Новое средство [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot) позволяет создавать файл с расширением **.bot**, в котором хранятся метаданные о службах, используемых ботом. C помощью этого файла бот может использовать интерфейс командной строки для подключения данных служб. Средство доступно в качестве модуля npm. Чтобы его установить, необходимо выполнить следующую команду.

```shell
npm install -g msbot 
```

Чтобы создать файл бота, в интерфейсе командной строки необходимо ввести команду **msbot init**, за которой следует ввести имя бота и имя конечной точки целевого URL-адреса, например:

```shell
msbot init --name name-of-my-bot --endpoint http://localhost:bot-port-number/api/messages
```
Чтобы подключить бот к службе, в интерфейсе командной строки необходимо ввести команду **msbot connect**, за которой следует имя соответствующей службы.

```shell
msbot connect service-type
```

| Service type (Тип службы) | ОПИСАНИЕ |
| ------ | ----------- |
| azure  |подключает бот к регистрации службы Azure Bot|
|endpoint| подключает бот к конечной точке, например localhost|
|luis     | подключает бот к приложению LUIS |
| qna     |подключает бот к базе знаний QnA|
|help [cmd]  |отображает справку по [cmd]|

### <a name="connect-your-bot-to-abs-with-the-bot-file"></a>Подключение бота к ABS с помощью файла BOT

После установки средства MSBot можно легко подключить бот к существующей группе ресурсов службы Azure Bot. Для этого необходимо выполнить команду az bot **show**. 

```azurecli
az bot show -n my-bot-name -g my-resource-group --msbot | msbot connect azure --stdin
```

Эта команда возьмет текущую конечную точку, идентификатор приложения MSA и пароль целевой группы ресурсов и обновит информацию о них в файле BOT. 


## <a name="5-manage-update-or-create-luis-and-qna-services-with--new-botbuilder-tools"></a>5. Управление, обновление или создание служб LUIS и QnA с помощью новых средств Bot Builder

[Средства Bot Builder](https://github.com/microsoft/botbuilder-tools) — это новый набор средств для управления и взаимодействия с ресурсами бота непосредственно из командной строки. 

>[!TIP]
> Каждое средство Bot Builder содержит команду глобальной справки, доступ к которой можно получить из командной строки посредством ввода **-h** или **--help**. Эта команда доступна в любое время из любого действия, что обеспечивает отображение доступных вариантов вместе с их описанием.

### <a name="ludown"></a>LUDown
[LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Ludown) позволяет описывать и создавать мощные компоненты языка для ботов с помощью файлов с расширением **LU**. Файл нового формата LU является типом формата разметки Markdown, который используется средством LUDown для вывода файлов JSON, адаптированных для целевой службы. В настоящее время файлы с форматом LU можно использовать для создания приложения [LUIS](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-get-started-create-app) или базы знаний [QnA](https://qnamaker.ai/Documentation/CreateKb), используя различные форматы. LUDown доступен в виде модуля npm и может быть использован при глобальной установке на компьютер.

```shell
npm install -g ludown
```
Средство LUDown можно использовать для создания моделей JSON как для LUIS, так и для QnA.  


### <a name="creating-a-luis-application-with-ludown"></a>Создание приложения LUIS с помощью LUDown

[Намерения](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-intents) и [сущности](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-entities) определяются для приложения LUIS точно так же, как и на портале LUIS. 

`# \<intent-name\>` описывает новый раздел определения намерений. Последующие строки содержат [высказывания](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/add-example-utterances), которые описывают это намерение.

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

С помощью следующего способа записи формат файла LU также поддерживает пары QnA. 

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

Также к одному ответу можно добавить несколько вопросов, просто добавляя новые строки вариантов вопросов в один ответ. 

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

Теперь в набор средств добавлено [расширение LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/LUIS), которое позволяет самостоятельно управлять ресурсами LUIS. Оно доступно в качестве модуля npm, который можно скачать.

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

Теперь в набор средств добавлено [расширение QnA](https://github.com/Microsoft/botbuilder-tools/tree/master/QnAMaker), которое позволяет самостоятельно управлять ресурсами LUIS. Оно доступно в качестве модуля npm, который можно скачать.

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
- [BotBuilder Tools Source Code](https://github.com/Microsoft/botbuilder-tools) (Исходный код средства Bot Builder)
- [MSBot](https://github.com/Microsoft/botbuilder-tools/tree/master/MSBot)
- [ChatDown](https://github.com/Microsoft/botbuilder-tools/tree/master/Chatdown)
- [LUDown](https://github.com/Microsoft/botbuilder-tools/tree/master/ludown)
- [интерфейс командной строки Azure](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)



---
title: Использование QnA Maker для ответов на вопросы | Документация Майкрософт
description: Узнайте, как использовать QnA Maker в боте.
keywords: question and answer, QnA, FAQs, qna maker
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: cognitive-services
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 15581daa570b9e51ff8f7bec93d16deebcd71d45
ms.sourcegitcommit: 93508adfb79523f610a919b361fc34f5c8dd3eff
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/02/2019
ms.locfileid: "67533382"
---
# <a name="use-qna-maker-to-answer-questions"></a>Использование QnA Maker для ответов на вопросы

[!INCLUDE[applies-to](../includes/applies-to.md)]

QnA Maker создает слой вопросов и ответов для диалога на основе ваших данных. Это позволяет боту отправлять вопросы в QnA Maker и получать ответы. Вам при этом не нужно беспокоиться об анализе и интерпретации смысла этих вопросов. 

Одна из основных сложностей при создании собственной службы QnA Maker — заполнить ее начальным набором вопросов и ответов. Во одних случаях вопросы и ответы уже существуют в таком содержимом, как разделы с вопросами и ответами или другая документация. В других случаях вы можете настроить ответы на вопросы в более естественном разговорном стиле. 

## <a name="prerequisites"></a>Предварительные требования

- Код в этой статье основан на примере QnA Maker. Вам потребуется копия примера для **[C#](https://aka.ms/cs-qna) или [JavaScript](https://aka.ms/js-qna-sample)** .
- Учетная запись [QnA Maker](https://www.qnamaker.ai/)
- Знания о [работе ботов](bot-builder-basics.md), [QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/overview/overview) и [управлении ресурсами бота](bot-file-basics.md).

## <a name="about-this-sample"></a>Об этом примере

Чтобы бот мог использовать QnA Maker, необходимо сначала создать базу знаний в [QnA Maker](https://www.qnamaker.ai/). Этот процесс мы рассмотрим в следующем разделе. После этого бот сможет отправить в эту службу запрос пользователя и получить наиболее точный ответ.

## <a name="ctabcs"></a>[C#](#tab/cs)
![Поток логики QnABot — C#](./media/qnabot-logic-flow.png)

`OnMessageActivityAsync` вызывается для каждого полученного блока данных, введенных пользователем. Этот метод обращается к информации `_configuration`, сохраненной в файле `appsetting.json` этого примера кода, и ищет значение для подключения к предварительно настроенной базе знаний QnA Maker. 

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)
![Поток логики QnABot — JavaScript](./media/qnabot-js-logic-flow.png)

`OnMessage` вызывается для каждого полученного блока данных, введенных пользователем. Этот метод обращается к соединителю `qnamaker`, который был предварительно настроен на основе значений, указанных в файле `.env` этого примера кода.  В qnamaker метод `getAnswers` подключает бота к внешней базе знаний QnA Maker.

---
Введенные пользователем данные передаются в эту базу знаний, а наиболее точный полученный ответ отображается пользователю.

## <a name="create-a-qna-maker-service-and-publish-a-knowledge-base"></a>Создание службы QnA Maker и публикация базы знаний
Прежде всего создайте службу QnA Maker. Выполните действия, описанные в [документации по QnA Maker](https://docs.microsoft.com/azure/cognitive-services/qnamaker/how-to/set-up-qnamaker-service-azure), чтобы создать службу в Azure.

После этого создайте базу знаний на основе файла `smartLightFAQ.tsv`, который расположен в папке CognitiveModels этого примера проекта. Действия по созданию, обучению и публикации [базы знаний](https://docs.microsoft.com/azure/cognitive-services/qnamaker/quickstarts/create-publish-knowledge-base) QnA Maker описаны в документации по QnA Maker. При выполнении этих действий присвойте базе знаний имя `qna` и выберите для ее заполнения файл `smartLightFAQ.tsv`.

> Примечание. Инструкции из этой статьи можно применить и для получения доступа к собственной базе знаний QnA Maker, разработанной пользователем.

## <a name="obtain-values-to-connect-your-bot-to-the-knowledge-base"></a>Получение значений для подключения бота к базе знаний
1. На сайте [QnA Maker](https://www.qnamaker.ai/) выберите свою базу знаний.
1. При открытой базе знаний выберите **Параметры**. Сохраните значение из поля _Имя службы_. Это значение поможет вам найти нужную базу знаний в интерфейсе портала QnA Maker. Оно не используется для подключения к этой базе знаний из приложения бота. 
1. Прокрутите вниз, чтобы найти **сведения о развертывании**, и запишите следующие значения из примера HTTP-запроса Postman:
   - POST /knowledgebases/\<идентификатор_базы_знаний>/generateAnswer
   - Узел: \<имя_узла> // полный URL-адрес, заканчивающийся /qnamaker
   - Авторизация: EndpointKey \<ключ_конечной_точки>
   
Полная строка URL-адреса для имени узла будет выглядеть так: https://< >.azure.net/qnamaker. Эти три значения предоставят приложению сведения, необходимые для подключения к базе знаний QnA Maker через службу Azure QnA.  

## <a name="update-the-settings-file"></a>Обновление файла параметров

Во-первых, добавьте в файл параметров сведения, необходимые для доступа к базе знаний: имя узла, ключ конечной точки и идентификатор базы знаний (kbId). Это значения, полученные на вкладке **Settings** (Параметры) базы знаний в QnA Maker. 

Если это развертывание не предназначено для рабочей среды, поля идентификатора приложения и пароля можно оставить пустыми.

> [!NOTE]
> Если вы реализуете доступ к базе знаний QnA Maker в существующем приложении бота, обязательно создайте понятные описательные заголовки для объектов QnA. Параметр name в этом разделе предоставляет ключ для доступа к этой информации из приложения.

## <a name="ctabcs"></a>[C#](#tab/cs)

### <a name="update-your-appsettingsjson-file"></a>Обновление файла appsettings.json

```json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  
  "QnAKnowledgebaseId": "<knowledge-base-id>",
  "QnAAuthKey": "<your-endpoint-key>",
  "QnAEndpointHostName": "<your-hostname>"
}
```

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

### <a name="update-your-env-file"></a>Обновление файла .env

```file
MicrosoftAppId=""
MicrosoftAppPassword=""

QnAKnowledgebaseId="<knowledge-base-id>"
QnAAuthKey="<your-endpoint-key>"
QnAEndpointHostName="<your-hostname>"
```

---

## <a name="set-up-the-qna-maker-instance"></a>Настройка экземпляра QnA Maker

Для начала мы создадим объект для доступа к базе знаний QnA Maker.

## <a name="ctabcs"></a>[C#](#tab/cs)

Убедитесь, что пакет NuGet **Microsoft.Bot.Builder.AI.QnA** установлен для вашего проекта.

В методе `OnMessageActivityAsync` в файле **QnABot.cs** мы создадим экземпляр QnAMaker. Класс `QnABot` извлекает имена для сведений о подключении, сохраненные выше в `appsettings.json`. Если вы выбрали другие имена для сведений о подключении к базе знаний в файле параметров, обязательно обновите имена в этом методе, чтобы они соответствовали выбранным.

**Bots/QnABot.cs**  
[!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=32-37)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

Убедитесь, что для проекта установлен пакет npm **botbuilder-ai**.

В нашем примере код для логики бота находится в файле **QnABot.js**.

В файле **QnABot.js** мы используем сведения о подключении, предоставленные в файле .env, чтобы подключиться к службе QnA Maker: _this.qnaMaker_.

**QnAMaker.js** [!code-javascript[QnAMaker](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=19-23)]


---

## <a name="calling-qna-maker-from-your-bot"></a>Вызов QnA Maker из кода бота

## <a name="ctabcs"></a>[C#](#tab/cs)

Когда боту потребуется ответ от QnAMaker, вызовите из кода бота метод `GetAnswersAsync()`, чтобы получить ответ с учетом текущего контекста. Если вы обращаетесь к собственной базе знаний, измените представленное ниже _сообщение об отсутствии ответов_ на более информативное для пользователей.

**QnABot.cs**  
[!code-csharp[qna connection](~/../botbuilder-samples/samples/csharp_dotnetcore/11.qnamaker/Bots/QnABot.cs?range=43-52)]

## <a name="javascripttabjs"></a>[JavaScript](#tab/js)

В файле **QnABot.js** мы передадим введенные пользователем данные в метод `getAnswers` службы QnA Maker, чтобы получить ответы из базы знаний. Если QnA Maker возвращает ответ, он отображается для пользователя. В противном случае пользователь получает сообщение No QnA Maker answers were found (Ответы в QnA Maker не найдены). 

**QnABot.js** [!code-javascript[OnMessage](~/../botbuilder-samples/samples/javascript_nodejs/11.qnamaker/bots/QnABot.js?range=43-59)]

---

## <a name="test-the-bot"></a>Тестирование бота

Выполните этот пример на локальном компьютере. Установите [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md#download), если вы это еще не сделали. См. подробнее в файле README для [C#](https://aka.ms/cs-qna) и [JavaScript](https://aka.ms/js-qna-sample).

Запустите эмулятор, подключитесь к боту и отправьте сообщение, как показано ниже.

![Тестирование примера QnA](../media/emulator-v4/qna-test-bot.png)

## <a name="next-steps"></a>Дополнительная информация

QnA Maker можно объединять с другими службами Cognitive Services, чтобы сделать бота еще более мощным. Средство подготовки к отправке предоставляет способ объединения QnA с распознаванием речи (LUIS) в боте.

> [!div class="nextstepaction"]
> [Combine LUIS apps and QnA services using the Dispatch tool](./bot-builder-tutorial-dispatch.md) (Объединение приложений LUIS и служб QnA с помощью средства Dispatch)

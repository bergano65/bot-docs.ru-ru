---
title: Создание взаимодействия на основе данных с помощью поиска Azure | Документы Майкрософт
description: Узнайте, как создать взаимодействие на основе данных с помощью поиска Azure и помочь пользователям ориентироваться в большом объеме материалов в боте с помощью пакета SDK Bot Framework для Node.js и Поиска Azure.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 259f709ae460fde13cdf25ce6d7cbf5dd44a333d
ms.sourcegitcommit: cf3786c6e092adec5409d852849927dc1428e8a2
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/01/2019
ms.locfileid: "57224862"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>Создание взаимодействия на основе данных с помощью Поиска Azure 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

Вы можете добавить в бот [поиск Azure][search], чтобы помочь пользователю ориентироваться в большом объеме материалов и создать взаимодействие на основе данных для пользователей вашего бота.

Поиск Azure — это служба Azure, которая обеспечивает поиск по ключевым словам, встроенные лингвистические правила, настраиваемую оценку, фасетную навигацию и многое другое. Поиск Azure может индексировать содержимое из различных источников, включая базы данных SQL Azure, DocumentDB, хранилище BLOB-объектов и хранилище таблиц. Он поддерживает принудительное индексирование для других источников данных и может открывать PDF-файлы, документы Office и другие форматы, содержащие неструктурированные данные. После сбора содержимое переходит в индекс службы поиска Azure, которой бот может отправлять запросы.

## <a name="install-dependencies"></a>Установка зависимостей

Из командной строки перейдите в каталог проекта вашего бота и установите следующие модули с помощью диспетчера пакета Node (NPM):

* [bluebird](https://www.npmjs.com/package/bluebird)
* [lodash](https://www.npmjs.com/package/lodash)
* [request](https://www.npmjs.com/package/request)

## <a name="prerequisites"></a>Предварительные требования

Ниже перечислены **обязательные компоненты**: 
- Подписка Azure и первичный ключ поиска Azure. Его значение можно найти на портале Azure.
- Скопируйте библиотеку [SearchDialogLibrary](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchDialogLibrary) в каталог проекта бота. Эта библиотека содержит общие диалоги для поиска, но ее можно настроить в зависимости от ваших потребностей. 

- Скопируйте библиотеку [SearchProviders](https://github.com/Microsoft/botBuilder-Samples/tree/master/Node/demo-Search/SearchProviders) в каталог проекта бота. Эта библиотека содержит все компоненты, необходимые для создания запроса и его отправки в службу поиска Azure.

## <a name="connect-to-the-azure-service"></a>Подключение к службе Azure 

В главном программном файле вашего бота (например, app.js), создайте пути для ссылок на две библиотеки, установленные ранее. 

```javascript
var SearchLibrary = require('./SearchDialogLibrary');
var AzureSearch = require('./SearchProviders/azure-search');
```

Добавьте следующий пример кода в бот. В объекте `AzureSearch` передайте свои параметры поиска Azure в метод `.create`. Во время выполнения благодаря этому бот будет привязан к службе поиска Azure и ожидать завершения запроса пользователя в виде `Promise`.  

```javascript
// Azure Search
var azureSearchClient = AzureSearch.create('Your-Azure-Search-Service-Name', 'Your-Azure-Search-Primary-Key', 'Your-Azure-Search-Service-Index');
var ResultsMapper = SearchLibrary.defaultResultsMapper(ToSearchHit);
```

 Ссылка `azureSearchClient` создает модель службы поиска Azure, которая передает параметры авторизации службы Azure из параметров бота `.env`. 
 `ResultsMapper` анализирует объект ответа Azure и сопоставляет эти данные в соответствии с определением метода `ToSearchHit`. Пример реализации этого метода см. в разделе [После ответа службы поиска Azure](#after-azure-search-responds).

## <a name="register-the-search-library"></a>Регистрация библиотеки поиска
Вы можете настроить диалоги поиска напрямую в самом модуле `SearchLibrary`. `SearchLibrary` выполняет большую часть тяжелой работы, включая вызов к службе поиска Azure. 

Добавьте следующий код в основной программный файл бота, чтобы зарегистрировать библиотеку диалогов поиска в боте. 

```javascript
bot.library(SearchLibrary.create({
    multipleSelection: true,
    search: function (query) { return azureSearchClient.search(query).then(ResultsMapper); },
    refiners: ['refiner1', 'refiner2', 'refiner3'], // customize your own refiners 
    refineFormatter: function (refiners) {
        return _.zipObject(
            refiners.map(function (r) { return 'By ' + _.capitalize(r); }),
            refiners);
    }
}));
```
`SearchLibrary` не только сохраняет все диалоги, связанные с поиском, но и отправляет запрос пользователя в службу поиска Azure. Вам нужно будет определить свои собственные уточнения в массиве `refiners`, чтобы указать сущности, по которым ваш пользователь может ограничивать или фильтровать результаты поиска.  

## <a name="create-a-search-dialog"></a>Создание диалога поиска

Вы можете структурировать свои диалоги любым способом. Для настройки диалога Поиска Azure нужно лишь вызвать метод `.begin` из объекта `SearchLibrary` и передать ему объект `session`, созданный пакетом SDK Bot Framework. 

```javascript
function (session) {
        // Trigger Azure Search dialogs 
        SearchLibrary.begin(session);
    },
    function (session, args) {
        // Process selected search results
        session.send(
            'Search Completed!',
            args.selection.map(  ); // format your response 
    }
```
Дополнительные сведения о диалогах см. в разделе [Управление беседой с помощью диалогов](bot-builder-nodejs-dialog-manage-conversation.md).

## <a name="after-azure-search-responds"></a>После ответа службы поиска Azure 

После успешного ответа службы поиска Azure вам необходимо сохранить нужные данные из объекта ответа и в понятном виде представить их пользователю.

> [!TIP]
> Для этого можно использовать [модуль служебной программы][NodeUtil]. Он поможет отформатировать и сопоставить ответ от службы поиска Azure.

В главном программном файле вашего бота создайте метод `ToSearchHit`. Этот метод возвращает объект, который форматирует важные данные из ответа Azure. В следующем коде показано, как можно определить собственные параметры в методе `ToSearchHit`. 
 
 ```javascript
 function ToSearchHit(azureResponse) {
     return {
         // define your own parameters 
         key: azureResponse.id,
         title: azureResponse.title,
         description: azureResponse.description,
         imageUrl: azureResponse.thumbnail
     };
 }
```
После этого вам нужно всего лишь показать данные пользователю. 

 В файле **index.js** в проекте **SearchDialogLibrary** метод `searchHitAsCard` анализирует каждый ответ от службы поиска Azure и создает новый объект карты для отображения пользователю. Поля, определенные в методе `ToSearchHit` в главном программном файле бота, необходимо синхронизировать со свойствами в методе `searchHitAsCard`. 

Ниже показано, как и где определенные параметры из метода `ToSearchHit` используются для создания пользовательского интерфейса вложения карты для отображения ответа пользователю. 

```javascript
function searchHitAsCard(showSave, searchHit) {
    var buttons = showSave
        ? [new builder.CardAction().type('imBack').title('Save').value(searchHit.key)]
        : [];

    var card = new builder.HeroCard()
        .title(searchHit.title) 
        .buttons(buttons);

    if (searchHit.description) {
        card.subtitle(searchHit.description);
    }

    if (searchHit.imageUrl) {
        card.images([new builder.CardImage().url(searchHit.imageUrl)]);
    }

    return card;
}
```

## <a name="sample-code"></a>Пример кода

Два полноценных примера ботов с поддержкой Поиска Azure, основанных на пакете SDK Bot Framework для Node.js, вы найдете в [примере бота для недвижимости](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/demo-Search/RealEstateBot) и [примере бота для вакансий](https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/Node/demo-Search/JobListingBot), размещенных в репозитории GitHub. 

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Поиск Azure][search]
* [Служебная программа узла][NodeUtil]
* [Диалоги](bot-builder-nodejs-dialog-manage-conversation.md)

[NodeUtil]: https://nodejs.org/api/util.html
[search]: /azure/search/search-what-is-azure-search

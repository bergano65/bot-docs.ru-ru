---
title: Создание взаимодействия на основе данных с помощью Поиска Azure | Документация Майкрософт
description: Узнайте, как создать взаимодействие на основе данных с помощью Поиска Azure и помочь пользователям ориентироваться в большом объеме материалов в боте с помощью пакета SDK Bot Framework для .NET и Поиска Azure.
author: matthewshim-ms
ms.author: v-shimma
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ef2cb33e8d2ab7d8db291e3c6e051630d6af0394
ms.sourcegitcommit: b15cf37afc4f57d13ca6636d4227433809562f8b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/11/2019
ms.locfileid: "54224549"
---
# <a name="create-data-driven-experiences-with-azure-search"></a>Создание взаимодействия на основе данных с помощью Поиска Azure 

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-search-azure.md)
> - [Node.js](../nodejs/bot-builder-nodejs-search-azure.md)

Вы можете добавить в бот [Поиск Azure](https://azure.microsoft.com/en-us/services/search/), чтобы помочь пользователю ориентироваться в большом объеме содержимого и создать средства взаимодействия на основе данных.

Поиск Azure — это служба Azure, которая обеспечивает поиск по ключевым словам, встроенные лингвистические правила, настраиваемую оценку, фасетную навигацию и многое другое. Поиск Azure может индексировать содержимое из различных источников, включая базы данных SQL Azure, DocumentDB, хранилище BLOB-объектов и хранилище таблиц. Он поддерживает принудительное индексирование для других источников данных и может открывать PDF-файлы, документы Office и другие форматы, содержащие неструктурированные данные. После сбора содержимое переходит в индекс службы поиска Azure, которой бот может отправлять запросы.


## <a name="prerequisites"></a>Предварительные требования

Установите пакет Nuget [Microsoft.Azure.Search](https://www.nuget.org/packages/Microsoft.Azure.Search/4.0.0-preview) в проект бота. 

В решении бота требуются приведенные ниже три проекта C#. Эти проекты обеспечивают дополнительные возможности для ботов и Поиска Azure. Создайте вилку проектов из [GitHub](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search) или напрямую скачайте исходный код.

* [Search.Azure](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search/Search.Azure) определяет вызов службы Azure. 
* [Search.Contracts](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search/Search.Contracts) определяет универсальные интерфейсы и модели данных для обработки данных.
* [Search.Dialogs](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search/Search.Dialogs) включает в себя различные универсальные диалоги Bot Builder, используемые для отправки запросов к Поиску Azure.

## <a name="configure-azure-search-settings"></a>Настройка параметров Поиска Azure 

Настройте параметры Поиска Azure в файле **Web.config** проекта, указав собственные учетные данные Поиска Azure в полях значений. Конструктор в классе `AzureSearchClient` будет использовать эти параметры для регистрации и привязки бота к службе Azure.

```xml
<appSettings>
    <add key="SearchDialogsServiceName" value="Azure-Search-Service-Name" /> <!-- replace value field with Azure Service Name --> 
    <add key="SearchDialogsServiceKey" value="Azure-Search-Service-Primary-Key" /> <!-- replace value field with Azure Service Key --> 
    <add key="SearchDialogsIndexName" value="Azure-Search-Service-Index" /> <!-- replace value field with your Azure Search Index --> 
</appSettings>
```

## <a name="create-a-search-dialog"></a>Создание диалога поиска

В проекте бота создайте класс `AzureSearchDialog` для вызова службы Azure в боте. Этот новый класс должен наследовать класс `SearchDialog` из проекта **Search.Dialogs**, который выполняет основную часть работы. Переопределение `GetTopRefiners()` позволяет пользователям уточнить и отфильтровать результаты поиска без необходимости начинать поиск заново, сохраняя состояние объекта поиска. Вы можете добавить собственные настраиваемые уточнения в массив `TopRefiners`, чтобы пользователи могли отфильтровать или уточнить результаты поиска. 

```cs
[Serializable]
public class AzureSearchDialog : SearchDialog
{
    private static readonly string[] TopRefiners = { "refiner1", "refiner2", "refiner3" }; // define your own custom refiners 

    public AzureSearchDialog(ISearchClient searchClient) : base(searchClient, multipleSelection: true)
    {
    }

    protected override string[] GetTopRefiners()
    {
        return TopRefiners;
    }
}
```

## <a name="define-the-response-data-model"></a>Определение модели данных ответа

Класс **SearchHit.cs** в проекте `Search.Contracts` определяет соответствующие данные из ответа Поиска Azure для анализа. Для бота единственным обязательным свойством является объявление и создание в конструкторе IDictionary `PropertyBag`. Все остальные свойства в этом классе можно определить в соответствии с потребностями бота. 

```cs
[Serializable]
public class SearchHit
{
    public SearchHit()
    {
        this.PropertyBag = new Dictionary<string, object>();
    }

    public IDictionary<string, object> PropertyBag { get; set; }

    // customize the fields below as needed 
    public string Key { get; set; }

    public string Title { get; set; }

    public string PictureUrl { get; set; }

    public string Description { get; set; }
}
```

## <a name="after-azure-search-responds"></a>После ответа службы поиска Azure 

После успешного выполнения запроса к службе Azure полученный результат поиска нужно будет проанализировать, чтобы извлечь соответствующие данные, которые бот покажет пользователю. Для этого необходимо создать класс `SearchResultMapper`. Объект `GenericSearchResult`, созданный в конструкторе, определяет список и словарь для сохранения результатов и аспектов соответственно после анализа каждого результата с применением моделей данных бота. 

Синхронизируйте свойства в методе `ToSearchHit` в соответствии с моделью данных в **SearchHit.cs**. Метод `ToSearchHit` будет выполняться и создавать новый `SearchHit` для каждого результата, найденного в ответе.  

```cs
public class SearchResultMapper : IMapper<DocumentSearchResult, GenericSearchResult>
{
    public GenericSearchResult Map(DocumentSearchResult documentSearchResult)
    {
        var searchResult = new GenericSearchResult();

        searchResult.Results = documentSearchResult.Results.Select(r => ToSearchHit(r)).ToList();
        searchResult.Facets = documentSearchResult.Facets?.ToDictionary(kv => kv.Key, kv => kv.Value.Select(f => ToFacet(f)));

        return searchResult;
    }

    private static GenericFacet ToFacet(FacetResult facetResult)
    {
        return new GenericFacet
        {
            Value = facetResult.Value,
            Count = facetResult.Count.Value
        };
    }

    private static SearchHit ToSearchHit(SearchResult hit)
    {
        return new SearchHit
        {
            // custom properties defined in SearchHit.cs 
            Key = (string)hit.Document["id"],
            Title = (string)hit.Document["title"],
            PictureUrl = (string)hit.Document["thumbnail"],
            Description = (string)hit.Document["description"]
        };
    }
}
```
После анализа и сохранения результатов полученную информацию все еще требуется показать пользователю. Необходимо управлять классом `SearchHitStyler` в соответствии с моделью данных из класса `SearchHit`. Например свойства `Title`, `PictureUrl` и `Description` из класса **SearchHit.cs** используются в примере кода для создания новых вложенных карточек. Приведенный ниже код создает вложенную карточку для каждого объекта `SearchHit` в списке результатов `GenericSearchResult` для отображения пользователю.   

```cs
[Serializable]
public class SearchHitStyler : PromptStyler
{
    public override void Apply<T>(ref IMessageActivity message, string prompt, IReadOnlyList<T> options, IReadOnlyList<string> descriptions = null)
    {
        var hits = options as IList<SearchHit>;
        if (hits != null)
        {
            var cards = hits.Select(h => new ThumbnailCard
            {
                Title = h.Title,
                Images = new[] { new CardImage(h.PictureUrl) },
                Buttons = new[] { new CardAction(ActionTypes.ImBack, "Pick this one", value: h.Key) },
                Text = h.Description
            });

            message.AttachmentLayout = AttachmentLayoutTypes.Carousel;
            message.Attachments = cards.Select(c => c.ToAttachment()).ToList();
            message.Text = prompt;
        }
        else
        {
            base.Apply<T>(ref message, prompt, options, descriptions);
        }
    }
}
```
Вы успешно добавили функции Поиска Azure в бот, и теперь пользователь может просматривать результаты поиска.

## <a name="samples"></a>Примеры

Два полноценных примера ботов с поддержкой Поиска Azure, основанных на пакете SDK Bot Framework для .NET, вы найдете в [примере бота для недвижимости](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search/RealEstateBot) и [примере бота для вакансий](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search/JobListingBot), размещенных в репозитории GitHub. 

## <a name="additional-resources"></a>Дополнительные ресурсы
* [Поиск Azure][search]
* [Диалоги в пакете SDK построителя ботов для .NET](bot-builder-dotnet-dialogs.md)
* [Примеры ботов для Поиска Azure](https://github.com/Microsoft/botBuilder-Samples/tree/master/CSharp/demo-Search)

[search]: /azure/search/search-what-is-azure-search

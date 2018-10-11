---
title: Примеры ботов для пакета SDK Bot Builder для .NET | Документация Майкрософт
description: Изучите примеры ботов, которые помогут повысить эффективность разработки ботов с помощью пакета SDK Bot Builder для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/13/2018
ms.openlocfilehash: 9f80fc551cac2f1994b0d398cd640d2ad3d78e61
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707380"
---
# <a name="bot-builder-sdk-for-net-samples"></a>Примеры для пакета SDK Bot Builder для .NET

::: moniker range="azure-bot-service-3.0"

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

В этих примерах демонстрируются ориентированные на задачи боты, показывающие, как воспользоваться преимуществами функций в пакете SDK Bot Builder для .NET. С помощью этих примеров можно быстро приступить к созданию отличных ботов с широкими возможностями.

## <a name="get-the-samples"></a>Получение примеров
Чтобы получить примеры, клонируйте репозиторий GitHub [BotBuilder-Samples](https://github.com/Microsoft/BotBuilder-Samples) с помощью Git.

```cmd
git clone https://github.com/Microsoft/BotBuilder-Samples.git
cd BotBuilder-Samples
```

Примеры ботов, созданных с помощью пакета SDK Bot Builder для .NET, находятся в каталоге **CSharp**.

Можно также просмотреть примеры на сайте GitHub и развернуть их в Azure.

## <a name="core"></a>Core
В этих примерах показано основные методы создания многофункциональных ботов.

Образец | ОПИСАНИЕ
------------ | ------------- 
[Отправка вложения](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-SendAttachment) | Пример бота, который отправляет простые мультимедийные вложения (изображения) пользователю. 
[Получение вложения](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ReceiveAttachment) | Пример бота, который получает вложения, отправленные пользователем, и скачивает их. 
[Создание общения](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CreateNewConversation)  | Пример бота, который запускает новый сеанс общения с использованием сохраненного ранее адреса пользователя.
[Получение участников общения](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-GetConversationMembers) | Пример бота, который получает список участников общения и обнаруживает его изменение. 
[Direct Line](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLine) | Пример бота и пользовательского клиента, обменивающихся данными между собой с помощью Direct Line API. 
[Direct Line (WebSockets)](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-DirectLineWebSockets) | Пример бота и пользовательского клиента, обменивающихся данными между собой с помощью Direct Line API и WebSockets. 
[Разные диалоги](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-MultiDialogs) | Пример бота, отображающего различные виды диалогов.
[API состояния](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-State) | Пример бота без отслеживания состояния, который отслеживает контекст общения.
[API пользовательского состояния](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-CustomState) | Пример бота без отслеживания состояния, который отслеживает контекст общения с использованием настраиваемого поставщика хранилища.
[channelData](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-ChannelData) | Пример бота, который отправляет собственные метаданные в Facebook с помощью ChannelData.
[AppInsights](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-AppInsights) | Пример бота, который записывает данные телеметрии в экземпляр Application Insights.

## <a name="search"></a>поиска
В этом примере показано, как использовать Поиск Azure в ботах, управляемых данными.

Образец | ОПИСАНИЕ
------------ | -------------
[Поиск Azure;](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-Search) | Два примера ботов, которые позволяют пользователю просмотреть большой объем содержимого.


## <a name="cards"></a>Карточки
В этих примерах показано, как отправлять форматированные карточки в Bot Framework.

Образец | ОПИСАНИЕ
------------ | -------------
[Форматированные карточки](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-RichCards) | Пример бота, который отправляет форматированные карточки разных типов.
[Карусель карточек](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/cards-CarouselCards) | Пример бота, который отправляет несколько форматированных карточек в одном сообщении с использованием макета карусели.

## <a name="intelligence"></a>Аналитика
В этих примерах показано, как добавить в бот возможности искусственного интеллекта, используя интерфейсы Bing и Microsoft Cognitive Services.

Образец | ОПИСАНИЕ
------------ | -------------
[LUIS](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-LUIS) | Пример бота, который использует LuisDialog для интеграции с приложением LUIS.ai.
[Подпись на изображении](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-ImageCaption) | Пример бота, который получает подпись на изображении с помощью API компьютерного зрения в Microsoft Cognitive Services.
[Преобразование речи в текст](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SpeechToText)  | Пример бота, которой получает текст из аудиофайла с помощью API распознавания речи Bing.
[Аналогичные продукты](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-SimilarProducts) | Пример бота, который ищет внешне похожие продукты с помощью API Bing для поиска изображений. 
[Zummer](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/intelligence-Zummer) | Пример бота, который находит статьи Википедии с помощью API Bing для поиска.

## <a name="reference-implementation"></a>Эталонная реализация
Этот пример разработан для демонстрации комплексного сценария. Это отличный источник фрагментов кода, если вы хотите реализовать в боте более сложные функции.


Образец | ОПИСАНИЕ
------------ | -------------
[Contoso Flowers](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/demo-ContosoFlowers) | Пример бота, который использует множество функций Bot Framework.

::: moniker-end

::: moniker range="azure-bot-service-4.0"

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

В примерах для Bot Builder из репозитория демонстрируются ориентированные на задачи боты, показывающие, как воспользоваться преимуществами функций в пакете SDK для .NET. С помощью этих примеров можно быстро приступить к созданию отличных ботов с широкими возможностями. Список примеров и дополнительную информацию см. в файле [сведений](https://github.com/Microsoft/BotBuilder-Samples/blob/master/README.md).

Чтобы получить примеры, клонируйте репозиторий GitHub [botbuilder-samples](https://github.com/Microsoft/botbuilder-samples) с помощью Git.
```cmd
git clone https://github.com/Microsoft/botbuilder-dotnet.git
```

::: moniker-end


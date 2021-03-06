---
title: Cognitive Services — Служба Azure Bot
description: Узнайте, как добавить ботам искусственный интеллект с помощью Microsoft Cognitive Services, чтобы сделать их более полезными и привлекательными.
keywords: распознавание языка, извлечение знаний, распознавание речи, поиск в Интернете
author: RobStand
ms.author: rstand
manager: rstand
ms.topic: article
ms.service: bot-service
ms.date: 12/17/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 6243d74aabd7fe67b1c7b8bfcebc8bd04511d8b5
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75792956"
---
# <a name="cognitive-services"></a>Cognitive Services

Microsoft Cognitive Services позволяет использовать коллекцию мощных алгоритмов ИИ, разработанных экспертами в области компьютерного зрения, речи, обработки естественного языка, извлечения знаний и поиска в Интернете. Эти службы упрощают множество задач на основе ИИ, что способствует быстрому добавлению новейших интеллектуальных технологий для ботов с помощью всего лишь нескольких строк кода. API интегрируются в большинство современных языков и платформ. API также постоянно совершенствуются, учатся и становятся умнее, поэтому опыт всегда обновляется. 

Боты с искусственным интеллектом отвечают так, как будто они могут видеть мир так же, как его видят люди. Они обнаруживают информацию и извлекают знания из разных источников, чтобы дать полезные ответы, и, самое главное, они учатся, приобретая все больше опыта, чтобы постоянно улучшать свои возможности. 

## <a name="language-understanding"></a>распознавание языка;

Взаимодействие между пользователями и ботами в основном происходит в свободной форме, поэтому боты должны понимать язык естественным образом и с учетом контекста. API языка Cognitive Service предоставляют мощные языковые модели, чтобы понять, что хотят пользователи, определить понятия и сущности в заданном предложении и в конечном итоге позволить ботам отвечать соответствующим действием. Пять API поддерживают несколько возможностей анализа текста, такие как проверка орфографии, определение тональности, языковое моделирование и извлечение точных и насыщенных аналитических сведений из текста. 

Cognitive Services предоставляет эти API для распознавания языка:

- <a href="https://www.microsoft.com/cognitive-services/language-understanding-intelligent-service-luis" target="_blank">Интеллектуальная служба распознавания речи (LUIS)</a> способна обрабатывать естественный язык с использованием готовых или специально обученных языковых моделей. Дополнительные сведения можно найти в разделе [Language understanding for bots](v4sdk/bot-builder-concept-luis.md) (Распознавание речи для ботов)

- <a href="https://www.microsoft.com/cognitive-services/text-analytics-api" target="_blank">API анализа текста</a> обнаруживает тональность, ключевые фразы, темы и язык из текста.

- <a href="https://www.microsoft.com/cognitive-services/bing-spell-check-api" target="_blank">API проверки орфографии Bing</a> обеспечивает широкие возможности проверки орфографии и способен распознавать разницу между именами, названиями торговых марок и сленговыми выражениями.

- <a href="https://docs.microsoft.com/azure/machine-learning/studio/text-analytics-module-tutorial" target ="_blank">Модели анализа текста в Студии машинного обучения Azure</a> позволяют создавать и применять модели анализа текста, такие как лемматизация и текстовая предобработка. Эти модели могут помочь в решении проблем, таких как проблемы с классификацией документов или анализом тональности.

Ознакомьтесь с дополнительными сведениями о [распознавании языка][language] с помощью Microsoft Cognitive Services.

## <a name="knowledge-extraction"></a>Извлечение знаний

Cognitive Services предоставляет четыре API знаний, которые позволяют идентифицировать именованные сущности или фразы в неструктурированном тексте, добавлять персонализированные рекомендации, предоставлять предложения автозаполнения, основанные на естественной интерпретации пользовательских запросов, а также искать научные публикации и другие исследования, например персонализированную службу часто задаваемых вопросов.

- <a href="https://www.microsoft.com/cognitive-services/entity-linking-intelligence-service" target="_blank">Аналитическая служба связывания сущностей</a> добавляет заметки в неструктурированный текст с помощью соответствующих сущностей, указанных в тексте. В зависимости от контекста одно и то же слово или фраза могут ссылаться на разные вещи. Эта служба понимает контекст предоставленного текста и будет идентифицировать каждую сущность в тексте.    

- <a href="https://www.microsoft.com/cognitive-services/knowledge-exploration-service" target="_blank">Служба поиска и распознавания данных</a> обеспечивает естественную интерпретацию пользовательских запросов и возвращает добавленные в заметки интерпретации, чтобы обеспечить богатые возможности поиска и автоматического заполнения, которые предугадывают, что печатает пользователь. Предложения по мгновенному завершению запроса и уточнения прогнозируемых запросов основаны на ваших собственных данных и специфических для приложения грамматиках, чтобы позволить пользователям выполнять быстрые запросы.    

- <a href="https://www.microsoft.com/cognitive-services/academic-knowledge-api" target="_blank">Academic Knowledge API</a> возвращает научно-исследовательские статьи, авторов, журналы, конференции, темы и университеты из <a href="https://www.microsoft.com/research/project/microsoft-academic-graph/" target="_blank">Microsoft Academic Graph</a>. Созданная в качестве предметно-ориентированного примера Службы поиска и распознавания данных, Academic Knowledge API предоставляет базу знаний, используя графический диалог с возможностями поиска по сотням миллионов сущностей, связанных с наукой. Выполните поиск по теме, профессору, университету или конференции, и API предоставит соответствующие публикации и связанные сущности. Грамматика также поддерживает естественные запросы, такие как "Papers by Michael Jordan about machine learning after 2010".

- <a href="https://qnamaker.ai" target="_blank">QnA Maker</a> — это простой в использовании REST API и веб-служба, которая обучает ИИ отвечать на вопросы пользователей естественным способом общения. Благодаря оптимизированной логике машинного обучения и способности интегрировать ведущую в отрасли обработку языка QnA Maker извлекает полуструктурированные данные, такие как пары "вопрос — ответ", в отдельные полезные ответы.

Ознакомьтесь с дополнительными сведениями об [извлечении знаний][knowledge] с помощью Microsoft Cognitive Services.

## <a name="speech-recognition-and-conversion"></a>Распознавание и преобразование речи

Используйте API распознавания речи, чтобы добавить боту расширенные навыки речи, которые используют лучшие в отрасли алгоритмы преобразования речи в текст и текста в речь, а также распознавание говорящего. В API распознавания речи используются встроенные языковые и акустические модели, которые с высокой точностью охватывают широкий диапазон сценариев. 

Для приложений, требующих дальнейшей настройки, можно использовать Custom Recognition Intelligent Service (CRIS). Эта служба позволяет настроить языковые и акустические модели распознавателя речи, адаптируя его к словарю приложения или даже к стилю речи пользователей.

В Cognitive Services доступно три API распознавания речи для обработки или синтеза речи:

- <a href="https://www.microsoft.com/cognitive-services/speech-api" target="_blank">API распознавания речи Bing</a> поддерживает возможности преобразования речи в текст и текста в речь.
- <a href="https://www.microsoft.com/cognitive-services/custom-recognition-intelligent-service-cris" target="_blank">Custom Recognition Intelligent Service (CRIS)</a> позволяет создавать пользовательские модели распознавания речи для адаптации преобразования речи в текст к словарю приложения или к стилю речи пользователя.
- <a href="https://www.microsoft.com/cognitive-services/speaker-recognition-api" target="_blank">API распознавания говорящего</a> позволяет идентифицировать и проверять говорящего по голосу.

Следующие ресурсы предоставляют дополнительную информацию о добавлении возможности распознавания речи боту.

* [Видеообзор приложений для общения с ботом](https://channel9.msdn.com/events/Build/2017/P4114)
* [Библиотека Microsoft.Bot.Client для приложений UWP или Xamarin](https://aka.ms/bot-client)
* [Пример клиентской библиотеки бота](https://aka.ms/trivia-bot-speech-sample)
* [Клиент WebChat с поддержкой речи](https://aka.ms/BotFramework-WebChat)

Ознакомьтесь с дополнительными сведениями о [распознавании и преобразовании речи][speech] с помощью Microsoft Cognitive Services.

## <a name="web-search"></a>Поиск в Интернете

API-интерфейсы поиска Bing позволяют добавлять ботам интеллектуальные возможности поиска в Интернете. С помощью нескольких строк кода можно получить доступ к миллиардам веб-страниц, изображений, видео, новостей и других типов результатов. Можно настроить API для возврата результатов по географическому расположению, рынку или языку для большей релевантности. Можно настроить поиск, используя поддерживаемые параметры поиска, такие как Safesearch, чтобы отфильтровать содержимое для взрослых, и Freshness, чтобы возвращать результаты в соответствии с конкретной датой.

В Cognitive Services доступно пять API-интерфейсов поиска Bing.

- <a href="https://www.microsoft.com/cognitive-services/bing-web-search-api" target="_blank">API Bing для поиска в Интернете</a> предоставляет веб-страницы, изображения, видео, новости и похожие результаты поиска с помощью одного вызова API.

- <a href="https://www.microsoft.com/cognitive-services/bing-image-search-api" target="_blank">API Bing для поиска изображений</a> возвращает результаты изображения с расширенными метаданными (доминирующий цвет, вид изображения и т. д.). Также он поддерживает несколько фильтров изображений для настройки результатов.

- <a href="https://www.microsoft.com/cognitive-services/bing-video-search-api" target="_blank">API Bing для поиска видео</a> извлекает видеорезультаты с богатыми метаданными (размер видео, качество, цена и т. д.) и предварительным просмотром видео. Также поддерживает несколько фильтров видео для настройки результатов.

- <a href="https://www.microsoft.com/cognitive-services/bing-news-search-api" target="_blank">API Bing для поиска новостей</a> находит новостные статьи по всему миру, которые соответствуют вашему поисковому запросу или в настоящее время набирают популярность в Интернете.

- <a href="https://www.microsoft.com/cognitive-services/bing-autosuggest-api" target="_blank">API автозаполнения Bing</a> предлагает мгновенные предложения по завершению запроса, чтобы быстрее завершить поисковой запрос и уменьшить количество ввода. 

Ознакомьтесь с дополнительными сведениями о [поиске в Интернете][search] с помощью Microsoft Cognitive Services.

## <a name="image-and-video-understanding"></a>Распознавание изображения и видео

API компьютерного зрения обеспечивают улучшенные навыки ботов для распознавания изображений и видео. Новейшие алгоритмы позволяют обрабатывать изображения или видео и возвращать информацию, которую можно преобразовать в действия. Например, их можно использовать для распознавания объектов, лиц людей, возраста, пола или даже чувств. 

API компьютерного зрения поддерживают различные функции распознавания изображений. Они могут определять содержимое для взрослых или откровенный контент, оценивать и акцентировать цвета, классифицировать содержимое изображений, выполнять оптическое распознавание символов и описывать изображение полными английскими предложениями. API компьютерного зрения также поддерживает несколько возможностей обработки изображений и видео, таких как интеллектуальное создание эскизов изображений или видео или стабилизация выхода видео.

Cognitive Services предоставляет четыре API, которые можно использовать для обработки изображений или видео:

- <a href="https://www.microsoft.com/cognitive-services/computer-vision-api" target="_blank">API компьютерного зрения</a> извлекает подробные сведения об изображениях (таких как объекты или люди), определяет, содержит ли изображение материалы для взрослых или откровенный контент, и обрабатывает текст (используя OCR) на изображениях.

- <a href="https://www.microsoft.com/cognitive-services/emotion-api" target="_blank">API распознавания эмоций</a> анализирует лица людей и распознает их эмоции в восьми возможных категориях эмоций людей.

- <a href="https://www.microsoft.com/cognitive-services/face-api" target="_blank">API распознавания лиц</a> определяет лица людей, сравнивает их с похожими лицами и может даже упорядочивать людей по группам в соответствии с визуальным сходством.

- <a href="https://www.microsoft.com/cognitive-services/video-api" target="_blank">API для видео</a> анализирует и обрабатывает видео для стабилизации его выхода, обнаружения движения, отслеживания лиц и может генерировать анимированный эскиз видео.

Ознакомьтесь с дополнительными сведениями о [распознавании изображений и видео][vision] с помощью Microsoft Cognitive Services.

## <a name="additional-resources"></a>Дополнительные ресурсы

Вы можете найти исчерпывающую документацию каждого продукта и соответствующие им ссылки на API в <a href="https://docs.microsoft.com/azure/cognitive-services" target="_blank">документации по службам Cognitive Services </a>.

[language]: https://docs.microsoft.com/azure/cognitive-services/luis/home
[search]: https://docs.microsoft.com/azure/cognitive-services/bing-web-search/search-the-web
[vision]: https://docs.microsoft.com/azure/cognitive-services/computer-vision/home
[knowledge]: https://docs.microsoft.com/azure/cognitive-services/kes/overview
[speech]: https://docs.microsoft.com/azure/cognitive-services/speech/home
[location]: https://docs.microsoft.com/azure/cognitive-services/

---
title: Обработка действий | Документация Майкрософт
description: Сведения об обработке действий в пакете SDK для ботов.
keywords: адаптер ботов, пользовательское ПО промежуточного слоя, укорачивание, резервирование, обработчики событий
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/22/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8e08ac4721ee78bbd5d13dac09d9e505d3a1b134
ms.sourcegitcommit: f95702d27abbd242c902eeb218d55a72df56ce56
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/19/2018
ms.locfileid: "39306491"
---
# <a name="activity-processing"></a>Обработка действий

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Боты и пользователи обмениваются данными с помощью действий. Каждое действие, получаемое приложением бота, передается адаптеру бота, который передает сведения о действиях в логику бота и в конечном счете отправляет ответы пользователю.

[!INCLUDE [Define a turn](~/includes/snippet-definition-turn.md)]

> [!IMPORTANT]
> Действия, особенно те, которые [формируются](#generating-responses) во время работы бота, обрабатываются асинхронно. Это необходимая часть построения бота. Если нужно разобраться, как это работает, изучите в зависимости от выбора языка [средства асинхронного программирования для .NET](https://docs.microsoft.com/en-us/dotnet/csharp/async) или [средства асинхронного программирования для JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function).

## <a name="the-bot-adapter"></a>Адаптер бота

Ботом управляет **адаптер**, который является его проводником. Адаптер отвечает за проверку подлинности, маршрутизацию входящих и исходящих сообщений и т. д. Адаптер отличается в зависимости от среды его работы (внутренний адаптер по-разному работает локально и в Azure), но в каждом случае он достигает той же цели. В большинстве случаев работа с ним происходит не напрямую, например, когда создается бот на основе шаблона, но знать о нем и о его назначении полезно.

Адаптер ботов инкапсулирует процессы проверки подлинности, а также отправляет и получает действия от службы Bot Connector. Когда бот получает действие, адаптер скрывает полные сведения о нем, создает [объект контекста](#turn-context), передает логику приложения бота и отправляет ответы, созданные ботом, обратно в почтовый ящик канала.

## <a name="authentication"></a>Authentication

Адаптер проверяет подлинность каждой входящей активности, получаемой приложением, используя информацию из действия и заголовок `Authentication` из запроса REST.

Адаптер использует объект соединителя и учетные данные приложения для проверки подлинности исходящих действий для пользователя.

Проверка подлинности службы Bot Connector использует маркеры JWT (JSON Web Token) `Bearer`, **идентификатор приложения Microsoft** и **пароль приложения Microsoft**, которые Azure создает вместе со службой ботов или при регистрации бота. Приложение должно учитывать эти учетные данные во время инициализации, чтобы разрешить адаптеру проверить подлинность трафика.

> [!NOTE]
> Если управление или тестирование бота происходит локально, например с помощью Bot Framework Emulator, это можно сделать без настройки адаптера для проверки подлинности трафика к боту и от бота.

## <a name="turn-context"></a>Включение контекста

Когда адаптер получает действие, он создает объект с функцией **Включить контекст**, которая предоставляет информацию о поступающей активности, отправителе и получателе, канале, общении и другие данные, необходимые для обработки активности. Затем адаптер передает боту этот объект с контекстом. Объект контекста сохраняется на протяжении включения и предоставляет сведения о следующем.

* Общение. Идентификация общения, включая информацию о боте и пользователе, участвующем в общении.
* Действие. Запросы и ответы в общении имеют все типы действий. Этот контекст предоставляет сведения о входящих действиях, включая сведения о маршрутизации, канале, общении, отправителе и получателе.
* Состояние. Свойства используются для отслеживания [состояния](~/v4sdk/bot-builder-storage-concept.md), например состояние общения с пользователем, сведения о пользователе или другие сведения бизнес-логики.
* Пользовательские сведения. Если расширить бот либо путем реализации промежуточного слоя, либо в логике ботов, можно предоставить дополнительные сведения о каждом этапе.

Объект контекста также может использоваться для отправки ответа пользователю и получения ссылки на адаптер для создания нового общения или продолжения существующего.

> [!NOTE]
> Приложение и адаптер будут обрабатывать запросы асинхронно. Однако для вашей бизнес-логики не требуется управление типа "запрос — ответ".

## <a name="middleware"></a>ПО промежуточного слоя

К адаптеру можно добавить ПО промежуточного слоя. ПО промежуточного слоя — это слои подключаемых модулей, добавленные к боту и его основной логике. Дополнительные сведения о ПО промежуточного слоя см. [здесь](~/v4sdk/bot-builder-concept-middleware.md).

ПО промежуточного слоя и логика бота используют объект контекста для получения информации о действиях и работают соответственно. ПО промежуточного слоя и бот могут также обновить или добавить сведения в объект контекста, например, для отслеживания состояния включения, общения и других областей. Пакет SDK предоставляет некоторое _состояние ПО промежуточного слоя_, которое можно использовать для добавления сохранения состояния к боту.

## <a name="generating-responses"></a>Создание ответов

Объект контекста предоставляет методы реагирования на действия, позволяющие коду реагировать на действие.

* Методы _отправка действия_ и _отправка действий_ отправляют одно или несколько действий в общение.
* Если канал поддерживает метод действия _обновить действие_, он обновляет действие в общении.
* Если канал поддерживает метод действия _удалить действие_, он убирает действие из общения.

Каждый метод ответа выполняется в асинхронном процессе. При вызове метода ответа действия перед вызовом обработчиков он клонирует связанный список обработчиков событий, что значит, что он будет содержать каждый обработчик, добавленный до этой точки, но не будет содержать ничего добавленного после запуска процесса.

Это также означает, что порядок ответов не гарантируется, особенно когда одна задача сложнее другой. Если бот может создать несколько ответов на входящее действие, убедитесь, что пользователь получил их в правильном порядке.

> [!IMPORTANT]
> Поток, обрабатывающий первое включение бота, связан с удалением объекта контекста после окончания работы. Если ответ (включая его обработчиков) занимает какое-то значительное время и пытается действовать на объект контекста, может произойти ошибка `Context was disposed`. **Обязательно ожидайте параметр `await` любых вызовов активности**, чтобы первичный поток дождался сгенерированного действия до завершения его работы и утилизации контекста включения.

## <a name="response-event-handlers"></a>Обработчики событий ответа

В дополнение к логике бота и ПО промежуточного слоя в контекстный объект могут быть добавлены обработчики ответов (иногда называемые обработчиками событий или обработчиками событий активности). Эти обработчики вызываются, когда связанный ответ происходит в текущем объекте контекста перед выполнением фактического ответа. Эти обработчики полезны, когда вы знаете, что хотите сделать до или после фактического события по каждому действию этого типа для остальной части текущего ответа.

> [!WARNING]
> Будьте осторожны, чтобы не вызвать метод ответа активности из своего соответствующего обработчика событий ответа, например, вызвав из обработчика метод активности _отправка действия_. Таким образом можно создать бесконечный цикл.

Каждое новое действие получает новый поток для выполнения. Когда создается поток для обработки действия, список обработчиков для этого действия копируется в этот новый поток. Если нет добавленных обработчиков, то после этой точки будет выполняться определенное действие.

Адаптер управляет обработчиками, зарегистрированными на объекте контекста, точно так же, как он управляет [конвейером ПО промежуточного слоя](~/v4sdk/bot-builder-concept-middleware.md#the-bot-middleware-pipeline).

А именно, обработчики вызываются в том порядке, в котором добавлены, и при команде _Далее_ делегат передает управление следующему зарегистрированному обработчику событий. Если обработчик не вызывает следующий делегат, адаптер не дает команду для обработчика событий, происходит [укорачивание](~/v4sdk/bot-builder-concept-middleware.md#short-circuiting) и ответ не попадает в канал.

## <a name="next-steps"></a>Дополнительная информация

Теперь, когда вы знакомы с некоторыми ключевыми понятиями бота, давайте обсудим все аспекты того, как бот может отправлять упреждающие сообщения.

> [!div class="nextstepaction"]
> [ПО промежуточного слоя](~/v4sdk/bot-builder-concept-middleware.md)
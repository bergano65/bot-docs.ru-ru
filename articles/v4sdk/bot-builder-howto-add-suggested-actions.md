---
title: Использование кнопки для ввода данных — Служба Azure Bot
description: Сведения о добавлении предлагаемых действий в сообщения с помощью пакета SDK Bot Framework для JavaScript.
keywords: Предлагаемые действия. Кнопки. Дополнительный ввод
author: Kaiqb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/10/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7a8ec7dbe7d8079967d66d9461b67fb3d2152d94
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75798415"
---
# <a name="use-button-for-input"></a>Использование кнопки для ввода данных

[!INCLUDE[applies-to](../includes/applies-to.md)]

Вы можете настроить бот для предоставления кнопок, с помощью которых пользователь может вводить данные. Кнопки удобны тем, что пользователи могут отвечать на вопросы или выбирать нужные варианты простым касанием, а не вводить ответ с помощью клавиатуры. В отличие от кнопок, которые появляются в функциональных карточках (и которые остаются видимыми и доступными для пользователя даже после касания), кнопки, отображаемые в области предлагаемых действий, исчезнут, как только будет сделан выбор. Это происходит для того, чтобы пользователь не касался устаревших кнопок в диалоге. Кроме того, это упрощает разработку ботов (так как для этого сценария учетная запись не требуется). 

## <a name="suggest-action-using-button"></a>Предложение действий с использованием кнопки

Функция *предлагаемых действий* позволяет боту представлять кнопки. Можно создать список предлагаемых действий ("быстрые ответы"), которые будут показаны пользователю для общения. 

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Представленный здесь исходный код основан на примере [предложенных действий](https://aka.ms/SuggestedActionsCSharp).

[!code-csharp[suggested actions](~/../botbuilder-samples/samples/csharp_dotnetcore/08.suggested-actions/Bots/SuggestedActionsBot.cs?range=87-101)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Представленный здесь исходный код основан на примере [предложенных действий](https://aka.ms/SuggestActionsJS).

[!code-javascript[suggested actions](~/../botbuilder-samples/samples/javascript_nodejs/08.suggested-actions/bots/suggestedActionsBot.js?range=61-64)]


# <a name="pythontabpython"></a>[Python](#tab/python)

Представленный здесь исходный код основан на примере [предложенных действий](https://aka.ms/SuggestActionsPython).

[!code-python[suggested actions](~/../botbuilder-python/samples/python/08.suggested-actions/bots/suggested_actions_bot.py?range=63-81)]


---

## <a name="additional-resources"></a>Дополнительные ресурсы

Вы можете получить полный исходный код:
- [Пример на языке C#](https://aka.ms/SuggestedActionsCSharp)
- [Пример на языке JavaScript](https://aka.ms/SuggestActionsJS)
- [Пример для Python](https://aka.ms/SuggestActionsPython)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сохранение данных пользователя и диалога](./bot-builder-howto-v4-state.md)

---
title: Распознавание намерения из содержимого сообщений | Документация Майкрософт
description: Узнайте, как распознавать намерение пользователя с помощью регулярных выражений или на основе содержимого сообщения.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: e308445a43507db94fe54735432790dabdb88731
ms.sourcegitcommit: a295a90eac461f8b96770dd902ba44919acf33fc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/26/2019
ms.locfileid: "67404851"
---
# <a name="recognize-user-intent-from-message-content"></a>Распознавание намерений пользователя из содержимого сообщения

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

Когда бот получает сообщение от пользователя, он использует **распознаватель**, чтобы проверить сообщение и определить намерение. Намерение обеспечивает сопоставление сообщений для вызова диалоговых окон. В этой статье объясняется, как распознавать намерение с помощью регулярных выражений или проверяя содержимое сообщения. Например, бот может использовать регулярные выражения для проверки того, содержит ли сообщение слово "help", и вызывать диалоговое окно справки. Бот также может проверить свойства пользователя сообщения, например, если пользователь отправляет изображение вместо текста и вызывает диалоговое окно обработки изображения. 

> [!NOTE]
> Сведения о распознавании намерений с помощью LUIS см. в разделе [Распознавание намерений и сущностей с помощью LUIS](bot-builder-nodejs-recognize-intent-luis.md) 


## <a name="use-the-built-in-regular-expression-recognizer"></a>Использование встроенного распознавателя регулярных выражений
Используйте [RegExpRecognizer][RegExpRecognizer], чтобы обнаруживать намерения пользователя с помощью регулярного выражения. Несколько выражений можно передать в распознаватель для поддержки нескольких языков. 

> [!TIP]
> Дополнительные сведения о локализации бота см. в разделе [Support localization](bot-builder-nodejs-localization.md) (Поддержка локализации).

Следующий код создает распознаватель регулярного выражения с именем `CancelIntent` и добавляет его боту. В этом примере распознаватель предоставляет регулярные выражения для обоих языковых стандартов `en_us` и `ja_jp`. 

[!code-js[Add a regular expression recognizer (JavaScript)](../includes/code/node-regex-recognizer.js#addRegexRecognizer)]

После добавления распознавателя боту присоедините [triggerAction][triggerAction] to the dialog that you want the bot to invoke when the recognizer detects the intent. Use the [matches][matches] к диалоговому окну, чтобы бот активизировался тогда, когда распознаватель обнаружит намерение. Используйте параметр matches, чтобы указать имя намерений, как показано в следующем коде.

[!code-js[Map the CancelIntent recognizer to a cancel dialog (JavaScript)](../includes/code/node-regex-recognizer.js#bindCancelDialogToRegexRecognizer)]

Распознаватели намерений являются глобальными. Поэтому распознаватель будет выполнять действие по каждому сообщению, полученному от пользователя. Если распознаватель обнаружит намерение, которое привязано к диалоговому окну с помощью действия `triggerAction`, он может активировать прерывание активного диалогового окна. Использование и обработка прерываний — гибкий механизм, учитывающий реальные действия пользователей.

> [!TIP] 
> Чтобы узнать, как работает `triggerAction` с диалогами, см. раздел об [управлении потоком общения с помощью диалогов](bot-builder-nodejs-manage-conversation-flow.md). Дополнительные сведения о различных действиях, которые можно связать с распознаваемым намерением, см. в разделе [Handle user actions](bot-builder-nodejs-dialog-actions.md) (Обработка действий пользователя).

## <a name="register-a-custom-intent-recognizer"></a>Регистрация настраиваемого распознавателя намерений
Кроме того, можно реализовать настраиваемый распознаватель. В этом примере добавляется простой распознаватель, который говорит пользователю "help" или "goodbye". Можно легко добавить распознавателя, который выполняет более сложную обработку, например распознает, когда пользователь отправляет изображение. 


[!code-js[Add a custom recognizer (JavaScript)](../includes/code/node-howto-recognize-intent.js#addCustomRecognizer)]

После регистрации распознавателя его можно связать с действием, используя предложение `matches`.

[!code-js[Bind intents to actions (JavaScript)](../includes/code/node-howto-recognize-intent.js#bindIntentsToActions)]

## <a name="disambiguate-between-multiple-intents"></a>Неоднозначность между несколькими намерениями

Бот может зарегистрировать более одного распознавателя. Обратите внимание на то, что пример пользовательского распознавателя включает в себя присвоенную числовую оценку для каждого намерения. Это связано с тем, что бот может иметь более одного распознавателя, а пакет SDK Bot Framework предоставляет встроенную логику для устранения неоднозначности между намерениями, возвращенными несколькими распознавателями. Обычно намерению присваивается оценка в диапазоне от 0,0 до 1,0, но настраиваемый распознаватель может определить для намерения значение выше 1,1, чтобы логика устранения неоднозначности пакета SDK Bot Framework всегда выбирала именно этот вариант. 

По умолчанию распознаватели действуют одновременно, но можно задать команду recognizeOrder в [IIntentRecognizerSetOptions][IntentRecognizerSetOptions], чтобы процесс завершился, как только бот найдет оценку, равную 1,0.

Пакет SDK для Bot Builder включает [пример][DisambiguationSample] that demonstrates how to provide custom disambiguation logic in your bot by implementing [IDisambiguateRouteHandler][IDisambiguateRouteHandler], который показывает, как предоставить пользовательскую логику устранения неоднозначности боту путем реализации IDisambiguateRouteHandler.

## <a name="next-steps"></a>Дополнительная информация
Логика использования регулярных выражений и проверки содержимого сообщения может усложниться, особенно в том случае, если поток общения бота является открытым. Чтобы помочь боту обрабатывать более широкий диапазон речи и текстовых входных данных от пользователей и добавить ему понимание естественного языка, можно использовать службу распознавания намерений, например [LUIS][LUIS].

> [!div class="nextstepaction"]
> [Распознавание намерений и сущностей с помощью LUIS](bot-builder-nodejs-recognize-intent-luis.md)


[LUIS]: https://www.luis.ai/

[triggerAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction

[matches]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.itriggeractionoptions.html#matches

[node-js-bot-how-to]: bot-builder-nodejs-recognize-intent-luis.md

[LUISAzureDocs]: /azure/cognitive-services/LUIS/Home

[IMessage]: http://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage

[IntentRecognizerSetOptions]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iintentrecognizersetoptions.html

[LuisRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer

[LUISSample]: https://aka.ms/v3-js-luisSample

[LUISConcepts]: https://docs.botframework.com/node/builder/guides/understanding-natural-language/

[DisambiguationSample]: https://aka.ms/v3-js-onDisambiguateRoute

[IDisambiguateRouteHandler]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.idisambiguateroutehandler.html

[RegExpRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.regexprecognizer.html

[AlarmBot]: https://aka.ms/v3-js-luisSample

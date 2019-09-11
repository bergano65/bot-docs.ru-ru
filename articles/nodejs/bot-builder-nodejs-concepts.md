---
title: Основные понятия, используемые при работе с пакетом SDK Bot Framework для Node.js | Документация Майкрософт
description: Обзор основных понятий, используемых при работе с пакетом SDK Bot Framework для Node.js, и предоставляемых в нем инструментов для создания и развертывания чат-ботов.
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2456d60357a218f1d790602134ab40767a3ce60f
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299914"
---
# <a name="key-concepts-in-the-bot-framework-sdk-for-nodejs"></a>Основные концепции, которые используются в пакете SDK Bot Framework для Node.js.

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.js](../nodejs/bot-builder-nodejs-concepts.md)

В этой статье раскрыты основные понятия, используемые при работе с пакетом SDK Bot Framework для Node.js. Общие сведения о Bot Framework см. в разделе [Обзор Bot Framework](../overview-introduction-bot-framework.md).

## <a name="connector"></a>Соединитель
Соединитель Bot Framework Connector — это служба, которая подключает бот к нескольким *каналам*, включая такие клиенты, как [Teams](https://docs.microsoft.com/microsoftteams/platform/concepts/bots/bots-create), Skype, Facebook, Slack и SMS. 

Соединитель обеспечивает обмен данными между ботом и пользователем, ретранслируя сообщения из бота в канал и из канала в бот. Логика бота размещается в качестве веб-службы, которая получает сообщения от пользователей через службу соединителя. Ответы бота отправляются в соединитель с помощью метода HTTPS POST. 

Пакет SDK Bot Framework для Node.js предоставляет классы [UniversalBot][UniversalBot] и [ChatConnector][ChatConnector], чтобы настроить бот отправлять и получать сообщения через соединитель Bot Framework. Класс `UniversalBot` — это центральный компонент логики бота. Он отвечает за управление всеми беседами бота с пользователем. Класс `ChatConnector` подключает бота к службе Bot Framework Connector.
Пример, демонстрирующий использование этих классов, см. в разделе [Создание бота с помощью пакета SDK Bot Framework для Node.js](bot-builder-nodejs-quickstart.md).

Соединитель также нормализует сообщения, отправляемые ботом в каналы, позволяя разрабатывать боты, независимые от платформы. Нормализация сообщения включает его преобразование из схемы Bot Framework в схему канала. Если канал не поддерживает все аспекты схемы платформы, соединитель будет пытаться преобразовать сообщение в формат, поддерживаемый каналом. Например, если бот отправляет сообщение, содержащее карточку с кнопками действий, в канал SMS, соединитель может обрабатывать карточку как изображение и включать действия как ссылки в тексте сообщения. [Channel Inspector][ChannelInspector] — это веб-средство, которое показывает вам, как соединитель будет обрабатывать сообщения для различных каналов.

Для `ChatConnector` требуется настроить конечную точку API в боте. С помощью пакета SDK для Node.js это можно сделать путем установки модуля Node.js `restify`. Боты также можно создавать для консоли с помощью [ConsoleConnector][ConsoleConnector], который не требует конечную точку API.

## <a name="messages"></a>Сообщения

Сообщения могут состоять из отображаемого текста, произносимого текста, вложений, форматированных карточек и предлагаемых действий. Метод [session.send][SessionSend] используется для отправки сообщений в ответ на сообщение от пользователя. Бот может вызывать `send` столько раз, сколько нужно в ответ на сообщение от пользователя. Соответствующий пример см. в разделе [Ответ на сообщения пользователя][RespondMessages].

Пример, демонстрирующий способ отправки форматированной графической карточки, содержащей интерактивные кнопки, которые пользователь нажимает для запуска действий, см. в разделе [Добавление форматированных карточек в сообщения](bot-builder-nodejs-send-rich-cards.md). Пример, в котором показано, как отправлять и получать вложения, см. в разделе [Отправка вложений](bot-builder-nodejs-send-receive-attachments.md). Пример, в котором показано, как отправить сообщение с текстом, который должен произнести бот, в канал с поддержкой речевых функций см. в разделе [Добавление речи в сообщения](bot-builder-nodejs-text-to-speech.md). Пример, демонстрирующий отправку предлагаемых действий, см. в разделе [Отправка предлагаемых действий](bot-builder-nodejs-send-suggested-actions.md).

## <a name="dialogs"></a>Диалоги
Диалоги помогают организовать естественную логику в боте и играют решающую роль в [конструировании потока беседы](../bot-service-design-conversation-flow.md). Общие сведения о диалогах см. в разделе [Управление беседой с помощью диалогов](bot-builder-nodejs-dialog-manage-conversation.md).

## <a name="actions"></a>Действия
Вам наверняка пригодится возможность обработки ботом прерываний, таких как запросы отмены или справки, в любой момент в потоке беседы. Пакет SDK Bot Framework для Node.js предоставляет глобальные обработчики сообщений, которые инициируют действия отмены или вызова других диалогов. Примеры использования обработчиков [triggerAction][triggerAction] см. в статье [Обработка действий пользователя](bot-builder-nodejs-dialog-actions.md).
<!--[Handling cancel](bot-builder-nodejs-manage-conversation-flow.md#handling-cancel), [Confirming interruptions](bot-builder-nodejs-manage-conversation-flow.md#confirming-interruptions) and-->


## <a name="recognizers"></a>Распознаватели
Когда пользователи что-то запрашивают в боте, например "справка" или "найти новости", бот должен понимать, что требуется пользователю, и предпринимать соответствующие действия. Для бота можно настроить распознавание намерений на основе ввода данных пользователем и связать намерение с действиями. 

Вы можете использовать встроенный распознаватель регулярных выражений, который предоставляется пакетом SDK Bot Framework, вызвать внешнюю службу, например API LUIS, или реализовать настраиваемый распознаватель для определения намерений пользователя. Примеры, демонстрирующие добавление распознавателей для бота и их использование для запуска действий, см. в разделе [Распознавание намерения пользователя](bot-builder-nodejs-recognize-intent-messages.md).


## <a name="saving-state"></a>Сохранение состояния

Очень важным аспектом является отслеживание контекста беседы, то есть бот должен запоминать такие вещи, как последний вопрос, заданный пользователем. Боты, созданные с помощью пакета SDK Bot Framework, не отслеживают состояние, что позволяет масштабировать систему, выполняя их на нескольких вычислительных узлах. Bot Framework предоставляет систему хранения, в которой хранятся данные ботов, что обеспечивает масштабирование веб-службы ботов. По этой причине рекомендуется избегать сохранения состояния с помощью глобальной переменной или закрытия функции, так как это приведет к проблемам при масштабировании бота. Вместо этого используйте следующие свойства объекта [сеанса][Session] бота для сохранения данных, связанных с пользователем или диалогом:

* **userData** глобально сохраняет данные для пользователя во всех беседах.
* **conversationData** глобально сохраняет данные для одной беседы. Эти данные видны всем участникам беседы, поэтому при сохранении данных в этом свойстве следует соблюдать осторожность. Оно включено по умолчанию, и его можно отключить с помощью параметра [persistConversationData][PersistConversationData] бота.
* **privateConversationData** глобально сохраняет данные для одной беседы, однако это закрытые данные, относящиеся к текущему пользователю. Эти данные охватывают все диалоги, что делает это свойство удобным для хранения временного состояния, которое требуется очистить после завершения беседы.
* **dialogData** сохраняет данные для одного экземпляра диалога. Это важно для хранения временных данных между шагами [каскада](bot-builder-nodejs-dialog-waterfall.md) в диалоге.

Примеры, демонстрирующие использование этих свойств для хранения и извлечения данных, см. в разделе [Управление данными состояния](bot-builder-nodejs-state.md).

## <a name="natural-language-understanding"></a>распознавание естественного языка;

Построитель ботов позволяет использовать для бота функции распознавания естественного языка LUIS с помощью класса [LuisRecognizer][LuisRecognizer]. Вы можете добавить экземпляр **LuisRecognizer**, который ссылается на опубликованную языковую модель, а затем — обработчики для выполнения действий в ответ на фразы пользователя. Руководство (10 минут), демонстрирующее работу службы LUIS:

* [Руководство по Microsoft LUIS][LUISVideo] (видео)

## <a name="next-steps"></a>Дополнительная информация
> [!div class="nextstepaction"]
> [Диалоги в пакете SDK построителя ботов для .NET](bot-builder-nodejs-dialog-overview.md)



[PersistConversationData]: https://docs.botframework.com/node/builder/chat-reference/interfaces/_botbuilder_d_.iuniversalbotsettings.html#persistconversationdata
[UniversalBot]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.universalbot.html
[ChatConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.chatconnector.html
[ConsoleConnector]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.consoleconnector.html

[ChannelInspector]: ../bot-service-channel-inspector.md

[Session]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session.html
[SessionSend]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.session#send

[triggerAction]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.dialog.html#triggeraction
[waterfall]: bot-builder-nodejs-prompts.md

[RespondMessages]:bot-builder-nodejs-use-default-message-handler.md

[LUISRecognizer]: https://docs.botframework.com/node/builder/chat-reference/classes/_botbuilder_d_.luisrecognizer
[LUISVideo]: https://vimeo.com/145499419

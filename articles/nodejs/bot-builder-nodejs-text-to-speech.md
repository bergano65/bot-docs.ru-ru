---
title: Добавление речи в сообщения | Документы Майкрософт
description: Сведения о добавлении речи в сообщения с помощью пакета SDK Bot Builder для Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 04d1f92687668267ca8226257ee83f993b5d09df
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305554"
---
# <a name="add-speech-to-messages"></a>Добавление речи в сообщения
> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

При создании бота для канала с поддержкой речевых функций, такого как Cortana, можно сформировать сообщения, в которых указывается произносимый ботом текст. Можно также попытаться повлиять на состояние микрофона клиента, задав [подсказку для ввода](bot-builder-nodejs-send-input-hints.md), чтобы указать, что бот принимает, ожидает или игнорирует ввод данных пользователем.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>Указание текста, произносимого ботом

С помощью пакета SDK Bot Builder для Node.js можно несколькими способами указать текст, произносимый ботом по каналу с поддержкой речевых функций. Можно задать свойство `IMessage.speak` и отправить сообщение с помощью метода `session.send()`, отправить сообщение с помощью метода `session.say()` (передав параметры, которые задают отображаемый текст, произносимый текст, и параметры) или отправить сообщение с помощью встроенного запроса (указав параметры `speak` и `retrySpeak`).

### <a id="message-speak"></a> IMessage.speak 

При создании сообщения, которое будет отправлено с помощью метода `session.send()`, задайте свойство `speak`, чтобы указать текст, произносимый ботом. В следующем примере кода показано создание сообщения, которое задает текст для произнесения и указывает, что бот [принимает входные данные пользователя](bot-builder-nodejs-send-input-hints.md).

[!code-javascript[IMessage.speak](../includes/code/node-text-to-speech.js#IMessageSpeak)]

### <a id="session-say"></a> session.say()

В качестве альтернативы `session.send()` можно вызвать метод `session.say()` для создания и отправки сообщения, которое, помимо текста для отображения и других параметров, указывает текст для произнесения. Метод определяется следующим образом.

`session.say(displayText: string, speechText: string, options?: object)`

| Параметр | ОПИСАНИЕ |
|----|----|
| `displayText` | Отображаемый текст. |
| `speechText` | Произносимый текст (в виде обычного текста или в формате <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">SSML</a>). |
| `options` | Объект [IMessage][IMessage], который может содержать вложение или [подсказку для ввода](bot-builder-nodejs-send-input-hints.md). |

В следующем примере кода показана отправка сообщения, которое задает текст для отображения и текст для произнесения и указывает, что бот [игнорирует входные данные пользователя](bot-builder-nodejs-send-input-hints.md).

[!code-javascript[Session.say()](../includes/code/node-text-to-speech.js#SessionSay)]

### <a id="prompt-options"></a> Параметры запроса

С помощью любого встроенного запроса можно задать параметры `speak` и `retrySpeak`, чтобы указать текст, произносимый ботом. В следующем примере кода создается запрос, указывающий отображаемый текст, изначально произносимый текст и текст, произносимый спустя некоторое время после ожидания входных данных пользователя. Он указывает, что бот [ожидает входные данные пользователя](bot-builder-nodejs-send-input-hints.md) и использует форматирование [SSML](#ssml), чтобы указать, что слово "sure" требуется произносить с умеренным выделением.

[!code-javascript[Prompt](../includes/code/node-text-to-speech.js#Prompt)]

## <a id="ssml"></a> Язык разметки синтеза речи (Speech Synthesis Markup Language, SSML)

Чтобы задать текст, произносимый ботом, используйте строку обычного текста или строку в формате SSML — языка разметки на основе XML, который позволяет контролировать различные характеристики речи бота, например голос, скорость, громкость, произношение, высоту звука и многое другое. Дополнительные сведения о SSML см. в <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">справочнике по языку разметки синтеза речи</a>.

> [!TIP]
> Для создания SSML в правильном формате следует использовать <a href="https://www.npmjs.com/search?q=ssml" target="_blank">библиотеку SSML</a>.

## <a name="input-hints"></a>Подсказки для ввода

При отправке сообщения по каналу с поддержкой речевых функций можно попытаться оказать влияние на состояние микрофона клиента, включив подсказку для ввода, указывающую, как бот реагирует на ввод данных пользователем — принимает, ожидает или игнорирует. Дополнительные сведения см. в статье [Добавление подсказок для ввода в сообщения](bot-builder-nodejs-send-input-hints.md).

## <a name="sample-code"></a>Пример кода 

Полный пример, в котором показано создание бота с поддержкой речевых функций с помощью пакета SDK Bot Builder для .NET, см. в <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">примере игрока</a> на сайте GitHub.

## <a name="additional-resources"></a>Дополнительные ресурсы

- <a href="https://msdn.microsoft.com/en-us/library/hh378377(v=office.14).aspx" target="_blank">Язык разметки синтеза речи (Speech Synthesis Markup Language, SSML)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/demo-RollerSkill" target="_blank">Пример игрока (GitHub)</a>
- [Справочник по пакету SDK построителя ботов для Node.js][SDKReference]

[SDKReference]: https://docs.botframework.com/en-us/node/builder/chat-reference/modules/_botbuilder_d_.html

[Message]: https://docs.botframework.com/en-us/node/builder/chat-reference/classes/_botbuilder_d_.message

[IMessage]: http://docs.botframework.com/en-us/node/builder/chat-reference/interfaces/_botbuilder_d_.imessage
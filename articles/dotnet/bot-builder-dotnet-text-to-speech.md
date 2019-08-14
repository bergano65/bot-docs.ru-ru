---
title: Добавление речи в сообщения | Документы Майкрософт
description: Сведения о добавлении речи в сообщения с помощью пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 94c0e7dd55e36c88066662ec7c9f3be1ce2dfd06
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/12/2019
ms.locfileid: "67405603"
---
# <a name="add-speech-to-messages"></a>Добавление речи в сообщения

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-text-to-speech.md)
> - [Node.js](../nodejs/bot-builder-nodejs-text-to-speech.md)
> - [REST](../rest-api/bot-framework-rest-connector-text-to-speech.md)

При создании бота для канала с поддержкой речевых функций, такого как Cortana, можно сформировать сообщения, в которых указывается произносимый ботом текст. Можно также попытаться повлиять на состояние микрофона клиента, задав [подсказку для ввода](bot-builder-dotnet-add-input-hints.md), чтобы указать, что бот принимает, ожидает или игнорирует ввод данных пользователем.

## <a name="specify-text-to-be-spoken-by-your-bot"></a>Указание текста, произносимого ботом

С помощью пакета SDK Bot Framework для .NET можно несколькими способами указать текст, произносимый ботом по каналу с поддержкой речевых функций. Можно задать свойство `Speak` [сообщения][IMessageActivity], вызвать метод `IDialogContext.SayAsync()` или указать параметры запроса `speak` и `retrySpeak` при отправке сообщения с помощью встроенного запроса.

### <a id="message-speak"></a> IMessageActivity.Speak

При создании [сообщения][IMessageActivity] и задании его отдельных свойств можно задать свойство сообщения `Speak`, чтобы указать текст, произносимый ботом. В следующем примере кода показано создание сообщения, которое задает текст для отображения и текст для произнесения и указывает, что бот [принимает входные данные пользователя](bot-builder-dotnet-add-input-hints.md).

[!code-csharp[Set speak property](../includes/code/dotnet-text-to-speech.cs#Speak1)]

### <a id="say-async"></a> IDialogContext.SayAsync()

Если вы используете [диалоги](bot-builder-dotnet-dialogs.md), можно вызвать метод `SayAsync()` для создания и отправки сообщения, которое, помимо текста для отображения и других параметров, указывает текст для произнесения. В следующий пример создает сообщение, которое задает текст для отображения и текст для произнесения.

[!code-csharp[Call SayAsync()](../includes/code/dotnet-text-to-speech.cs#Speak2)]

### <a id="prompt-options"></a> Параметры запроса

С помощью любого встроенного запроса можно задать параметры `speak` и `retrySpeak`, чтобы указать текст, произносимый ботом. В следующем примере кода создается запрос, указывающий отображаемый текст, изначально произносимый текст и текст, произносимый спустя некоторое время после ожидания входных данных пользователя. Он использует формат [SSML](#ssml), чтобы указать, что слово "sure" должно быть произнесено с умеренным количеством акцентов.

[!code-csharp[Set Prompt options](../includes/code/dotnet-text-to-speech.cs#Speak3)]

## <a id="ssml"></a> Язык разметки синтеза речи (Speech Synthesis Markup Language, SSML)

Чтобы указать боту текст для голосового воспроизведения, ему можно присвоить строку в формате (SSML). SSML — это язык разметки на базе XML (следовательно, текст в этом формате должен соответствовать требованиям XML). Он позволяет контролировать различные характеристики речи бота, такие как голос, скорость, тон и др. Дополнительные сведения о SSML см. в <a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">справочнике по языку разметки синтеза речи</a>.

После предоставления форматированной строки SSML внешний элемент программы-оболочки SSML можно проигнорировать.

## <a name="input-hints"></a>Подсказки для ввода

При отправке сообщения по каналу с поддержкой речевых функций можно попытаться оказать влияние на состояние микрофона клиента, включив подсказку для ввода, указывающую, как бот реагирует на ввод данных пользователем — принимает, ожидает или игнорирует. Дополнительные сведения см. в статье [Добавление подсказок для ввода в сообщения](bot-builder-dotnet-add-input-hints.md).

## <a name="sample-code"></a>Пример кода 

Полный пример, в котором показано создание бота с поддержкой речевых функций с помощью пакета SDK Bot Framework для .NET, см. в <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp" target="_blank">примере Roller Skill</a> на сайте GitHub.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Создание сообщений](bot-builder-dotnet-create-messages.md)
- [Добавление подсказок для ввода в сообщения](bot-builder-dotnet-add-input-hints.md)
- <a href="https://msdn.microsoft.com/library/hh378377(v=office.14).aspx" target="_blank">Язык разметки синтеза речи (Speech Synthesis Markup Language, SSML)</a>
- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/v3-sdk-samples/CSharp/demo-RollerSkill" target="_blank">Пример Roller Skill (GitHub)</a>
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
- <a href="/dotnet/api/microsoft.bot.connector.imessageactivity" target="_blank">Интерфейс IMessageActivity</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.dialogcontext" target="_blank">Класс DialogContext</a>
- <a href="/dotnet/api/microsoft.bot.builder.dialogs.internals.prompt-2" target="_blank">Класс Prompt</a>

[IMessageActivity]: /dotnet/api/microsoft.bot.connector.imessageactivity


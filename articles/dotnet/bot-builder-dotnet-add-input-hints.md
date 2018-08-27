---
title: Добавление подсказок для ввода в сообщения | Документы Майкрософт
description: Сведения о добавлении подсказок для ввода в сообщения с помощью пакета SDK построителя ботов для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 87fc068c831dba752fa52a6430327232719a74a9
ms.sourcegitcommit: 67445b42796d90661afc643c6bb6533e9a662cbc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/06/2018
ms.locfileid: "39574830"
---
# <a name="add-input-hints-to-messages"></a>Добавление подсказок для ввода в сообщения

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-input-hints.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-input-hints.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-input-hints.md)

Задав подсказку для ввода, можно указать, какое действие бот выполнит после того, как сообщение будет доставлено клиенту: примет, будет ожидать или пропустит ввод данных пользователем. Для многих каналов это позволяет клиентам соответствующим образом задавать состояние элементов управления ввода данных пользователем. Например, если подсказка для ввода указывает, что бот пропускает ввод данных пользователем, клиент может закрыть микрофон и отключить поле ввода, чтобы предотвратить ввод данных пользователем.

## <a name="accepting-input"></a>Принятие ввода данных

Чтобы указать, что ваш бот пассивно готов к вводу, но не ожидает ответа от пользователя, присвойте подсказке для ввода значение `InputHints.AcceptingInput`. В результате во многих каналах поле ввода будет включено, а микрофон закрыт, но доступен пользователю. Например, Кортана откроет микрофон, чтобы принять входные данные пользователя, если пользователь будет удерживать кнопку микрофона. Следующий пример кода создает сообщение, которое указывает, что бот принимает ввод данных пользователем.

```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.AcceptingInput;
await connector.Conversations.ReplyToActivityAsync(reply);
```

## <a name="expecting-input"></a>Ожидание ввода данных

Чтобы указать, что ваш бот ожидает ответа от пользователя, присвойте подсказке для ввода в сообщение значение `InputHints.ExpectingInput`. В результате во многих каналах поле ввода будет включено, а микрофон открыт. Следующий пример кода создает сообщение, которое указывает, что бот ожидает ввод данных пользователем.

```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.ExpectingInput;
await connector.Conversations.ReplyToActivityAsync(reply);
```

## <a name="ignoring-input"></a>Пропуск ввода данных

Чтобы указать, что ваш бот не готов к получению входных данных от пользователя, присвойте подсказке для ввода значение `InputHints.IgnorningInput`. В результате во многих каналах поле ввода будет отключено, а микрофон закрыт. Следующий пример кода создает сообщение, которое указывает, что бот игнорирует ввод данных пользователем.

```cs
Activity reply = activity.CreateReply("This is the text that will be displayed.");
reply.Speak = "This is the text that will be spoken.";
reply.InputHint = InputHints.IgnoringInput;
await connector.Conversations.ReplyToActivityAsync(reply);
```

## <a name="default-values-for-input-hint"></a>Значения по умолчанию для подсказки для ввода

Если подсказка для ввода в сообщении не задана, пакет SDK для Bot Builder автоматически настроит ее соответственно следующей логике.

- Если бот отправляет запрос, подсказка для ввода в сообщение укажет, что бот **ожидает ввода данных**.</li>
- Если бот отправляет одно сообщение, подсказка для ввода в сообщение укажет, что бот **принимает ввод данных**.</li>
- Если бот отправляет ряд последовательных сообщений, подсказка для ввода во всех сообщениях, кроме последнего в серии, укажет, что бот **пропускает ввод данных**, а подсказка для ввода в последнем сообщении в серии укажет, что бот **принимает ввод данных**.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Создание сообщений](bot-builder-dotnet-create-messages.md)
- [Добавление речи в сообщения](bot-builder-dotnet-text-to-speech.md)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
- <a href="/dotnet/api/microsoft.bot.connector.inputhints" target="_blank">Класс InputHints</a>
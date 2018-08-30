---
title: Добавление подсказок ввода к сообщениям | Документация Майкрософт
description: Сведения о добавлении подсказок ввода к сообщениям с помощью пакета SDK для Bot Builder.
keywords: Подсказки для ввода, принятие ввода данных, ожидание ввода данных, игнорирование ввода, речь
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/24/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 02c6cf56a0c1161fa0393880810c1481c5eb2461
ms.sourcegitcommit: f89ed979eb6321232fb21100ef376d9b0d5113c9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/24/2018
ms.locfileid: "42914634"
---
# <a name="add-input-hints-to-messages"></a>Добавление подсказок ввода к сообщениям

[!INCLUDE [pre-release-label](~/includes/pre-release-label.md)]

Задав подсказку для ввода, можно указать, какое действие бот выполнит после того, как сообщение будет доставлено клиенту: примет, будет ожидать или пропустит ввод данных пользователем. Для многих каналов это позволяет клиентам соответствующим образом задавать состояние элементов управления ввода данных пользователем. Например, если подсказка для ввода указывает, что бот пропускает ввод данных пользователя, клиент может закрыть микрофон и отключить поле ввода, чтобы предотвратить ввод данных пользователем.

Убедитесь, что необходимые библиотеки включены для ввода указаний.

# <a name="ctabcs"></a>[C#](#tab/cs)

```cs
using Microsoft.Bot.Schema;
```

<!--TODO: Remove the following remark after the next release of the NuGet packages.-->

Класс **MessageFactory**, используемый в этих примерах, определяется в следующем пространстве имен.

```cs
using Microsoft.Bot.Builder.Core.Extensions;
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const {InputHints, MessageFactory} = require('botbuilder');
```

---

## <a name="accepting-input"></a>Принятие ввода данных

Чтобы указать, что бот пассивно готов к вводу, но не ожидает ответа от пользователя, установите подсказку ввода на _Принятие ввода данных_. В результате во многих каналах поле ввода будет включено, а микрофон закрыт, но доступен пользователю. Например, Кортана откроет микрофон, чтобы принять входные данные пользователя, если пользователь будет удерживать кнопку микрофона. Следующий код создает сообщение, которое указывает, что бот принимает ввод данных пользователем.

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.AcceptingInput);
await context.SendActivity(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.AcceptingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="expecting-input"></a>Ожидание ввода данных

Чтобы указать, что бот ожидает ответа от пользователя, установите подсказку ввода сообщения _Ожидание ввода данных_. В результате во многих каналах поле ввода будет включено, а микрофон открыт. Следующий пример кода создает сообщение, которое указывает, что бот ожидает ввод данных пользователем.

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.ExpectingInput);
await context.SendActivity(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.ExpectingInput);
await context.sendActivity(basicMessage);
```

---

## <a name="ignoring-input"></a>Пропуск ввода данных

Чтобы указать, что бот не готов к получению входных данных от пользователя, установите подсказку ввода на _Игнорирование ввода данных_. В результате во многих каналах поле ввода будет отключено, а микрофон закрыт. Следующий пример кода создает сообщение, которое указывает, что бот игнорирует ввод данных пользователем.

# <a name="ctabcs"></a>[C#](#tab/cs)

```csharp
var reply = MessageFactory.Text(
    "This is the text that will be displayed.",
    "This is the text that will be spoken.",
    InputHints.IgnoringInput);
await context.SendActivity(reply).ConfigureAwait(false);
```

# <a name="javascripttabjs"></a>[JavaScript](#tab/js)

```javascript
const basicMessage = MessageFactory.text('This is the text that will be displayed.', 'This is the text that will be spoken.', InputHints.IgnoringInput);
await context.sendActivity(basicMessage);
```

---

## <a name="default-values-for-input-hint"></a>Значения по умолчанию для подсказки для ввода

Если подсказка для ввода в сообщении не задана, пакет SDK для Bot Builder автоматически настроит ее соответственно следующей логике.

- Если бот отправляет запрос, подсказка для ввода в сообщении укажет, что бот **ожидает ввода данных**.</li>
- Если бот отправляет одно сообщение, подсказка для ввода в сообщении укажет, что бот **принимает ввод данных**.</li>
- Если бот отправляет ряд последовательных сообщений, подсказка для ввода во всех сообщениях, кроме последнего в серии, укажет, что бот **игнорирует ввод данных**, а подсказка для ввода в последнем сообщении в серии укажет, что бот **принимает ввод данных**.

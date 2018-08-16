---
title: Определение функции Script в качестве действия Do | Документация Майкрософт
description: Узнайте, как настроить функцию действия Script в качестве действия Do.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 674781c2cb5a8700941feb59b2ce7ab902979d4f
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39306206"
---
# <a name="define-a-script-function-as-a-do-action"></a>Определение функции Script в качестве действия Do

[Функции действия Script](conversation-designer-context-object.md#script-callback-functions) выполняют пользовательский сценарий для завершения задачи. Например, звонят в службу, чтобы установить термостат на уровне 74 градуса, когда пользователь говорит "Set my thermostat to 74 degrees". 

Чтобы использовать пользовательский сценарий в качестве действия, выберите **действие Script** как тип действия **DO** в редакторе задачи. Затем введите имя функции, которая реализует действие. Щелкните **Изменить**, чтобы открыть редактор **сценариев** и написать собственную реализацию для функции. 

## <a name="script-action-function-parameter"></a>Параметр функции действия Script

Функция обратного вызова указанного действия всегда будет вызываться с помощью объекта [`context`](conversation-designer-context-object.md).

Объект `context` включает в себя `taskEntities` и `contextEntities`. Сущности задачи — это сущности, которые определяются или создаются для этой задачи, а сущности контекста — это контейнеры свойств, в которых сущности могут сохраняться во время общения с пользователем.

## <a name="return-value"></a>Возвращаемое значение
Функции **действие Script** предполагают возвращение логического значения.

## <a name="sample-script-action-function"></a>Пример функции действия Script
Приведенный ниже пример функции совершает HTTP-вызов для получения ответа перед возвратом соответствующего триггера.

```javascript
module.exports.NewTask_do_onRun = function(context) {
    var options =  {
        host: 'HOST',
        path: 'PATH',
        method: 'post',
        headers: {
            "HEADER1" : "VALUE"
        }, 
        body: {
            "BODY": VALUE
        }
    };

    return http.request(options).then(function(response) {
      // parse response payload
      // Send a message to user
      context.responses.push({text: "Done", type: "message"});
    });
} 
```

## <a name="next-step"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Действие Dialogue](conversation-designer-dialogues.md)

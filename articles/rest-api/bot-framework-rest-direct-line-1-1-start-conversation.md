---
title: Сведения о том, как начать общение с помощью Direct Line API 1.1 — Служба Azure Bot
description: Сведения о том, как начать диалог с помощью API Direct Line версии 1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 6aa107dc54f6a960aabc4f03b5e9acb49a9c6cb0
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789595"
---
# <a name="start-a-conversation"></a>Начало общения

> [!IMPORTANT]
> В этой статье объясняется, как начать диалог с помощью API Direct Line версии 1.1. При создании подключения между клиентским приложением и ботом используйте [API версии 3.0 для Direct Line](bot-framework-rest-direct-line-3-0-start-conversation.md).

Общения Direct Line открываются клиентами явным образом и могут выполняться, пока бот и клиент участвуют в них и имеют действительные учетные данные. Бот и клиент могут отправлять сообщения, пока общение остается открытым. К определенному диалогу могут подключаться несколько клиентов, и каждый из них может участвовать в нем от имени нескольких пользователей.

## <a name="open-a-new-conversation"></a>Открытие нового общения

Чтобы с помощью бота открыть новое общение, необходимо выполнить следующий запрос.

```http
POST https://directline.botframework.com/api/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

Ниже приведены примеры фрагментов кода, предназначенные для запроса "начало общения" и соответствующего ответа.

### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/api/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Ответ

Если запрос выполнен, ответ будет содержать идентификатор для диалога, маркер и значение, указывающее количество секунд до истечения срока действия маркера.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800
}
```

## <a name="start-conversation-versus-generate-token"></a>Сравнение операций "Начало общения" и "Создать маркер"

Операция начала диалога (`POST /api/conversations`) аналогична операции [создания маркера](bot-framework-rest-direct-line-1-1-authentication.md#generate-token) (`POST /api/tokens/conversation`) в том смысле, что обе операции возвращают маркер `token`, который можно использовать для доступа к одному диалогу. Однако операция начала диалога также начинает диалог и контактирует с ботом, тогда как операция создания маркера не поддерживает ни одно из этих действий. 

Если требуется немедленно начать общение, используйте операцию "Начало общения". Если требуется распространить маркер среди клиентов, а также заставить их начать общение, используйте операцию [создание маркера Direct Line](bot-framework-rest-direct-line-1-1-authentication.md#generate-token). 

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-1-1-concepts.md)
- [Аутентификация](bot-framework-rest-direct-line-1-1-authentication.md)
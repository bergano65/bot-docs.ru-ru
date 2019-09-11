---
title: Начало диалога | Документация Майкрософт
description: Сведения о том, как начать диалог с помощью API Direct Line версии 1.1.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 16ad7b817ccde2139b703e858e57bee2abf36fb8
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299604"
---
# <a name="start-a-conversation"></a>Начало диалога

> [!IMPORTANT]
> В этой статье объясняется, как начать диалог с помощью API Direct Line версии 1.1. При создании подключения между клиентским приложением и ботом используйте [API версии 3.0 для Direct Line](bot-framework-rest-direct-line-3-0-start-conversation.md).

Диалоги Direct Line открываются клиентами явным образом и могут выполняться, пока бот и клиент участвуют в них и имеют действительные учетные данные. Бот и клиент могут отправлять сообщения, пока общение остается открытым. К определенному диалогу могут подключаться несколько клиентов, и каждый из них может участвовать в нем от имени нескольких пользователей.

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

## <a name="start-conversation-versus-generate-token"></a>Сравнение операций начала диалога и создания маркера

Операция начала диалога (`POST /api/conversations`) аналогична операции [создания маркера](bot-framework-rest-direct-line-1-1-authentication.md#generate-token) (`POST /api/tokens/conversation`) в том смысле, что обе операции возвращают маркер `token`, который можно использовать для доступа к одному диалогу. Однако операция начала диалога также начинает диалог и контактирует с ботом, тогда как операция создания маркера не поддерживает ни одно из этих действий. 

Если вы хотите начать диалог немедленно, используйте операцию начала диалога. Если требуется распространить маркер среди клиентов, а также заставить их начать общение, используйте операцию [создание маркера Direct Line](bot-framework-rest-direct-line-1-1-authentication.md#generate-token). 

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-1-1-concepts.md)
- [Аутентификация](bot-framework-rest-direct-line-1-1-authentication.md)
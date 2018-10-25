---
title: Начало общения | Документация Майкрософт
description: Сведения о том, как начать общение с помощью Direct Line API 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
ms.openlocfilehash: 6754935070fefb890c3d5b7bd90f1c1a4c5d401d
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "50000301"
---
# <a name="start-a-conversation"></a>Начало общения

Общения Direct Line открываются клиентами явным образом и могут выполняться, пока бот и клиент участвуют в них и имеют действительные учетные данные. Бот и клиент могут отправлять сообщения, пока общение остается открытым. К определенному диалогу могут подключаться несколько клиентов, и каждый из них может участвовать в нем от имени нескольких пользователей.

## <a name="open-a-new-conversation"></a>Открытие нового общения

Чтобы с помощью бота открыть новое общение, необходимо выполнить следующий запрос.

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer SECRET_OR_TOKEN
```

Ниже приведены примеры фрагментов кода, предназначенные для запроса "начало общения" и соответствующего ответа.

### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/v3/directline/conversations
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Ответ

Если запрос выполнен, ответ будет содержать идентификатор общения, маркер и значение, указывающее количество секунд до истечения срока действия маркера, и URL-адрес потока, который может быть использован пользователем для [получения действий через потоковую передачу WebSocket](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket).

```http
HTTP/1.1 201 Created
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "expires_in": 1800,
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?t=RCurR_XV9ZA.cwA..."
}
```

Как правило, запрос "Начало общения" используется для открытия нового общения и, если оно успешно запущено, возвращается код состояния **HTTP 201**. Однако если клиент отправляет запрос "Начало общения" с маркером Direct Line в заголовок `Authorization`, к которому ранее была применена операция "Начало общения", в качестве ответа будет получен код статуса **HTTP 200**, который указывает, что запрос был обработан, но общение создано не было (так как оно уже существует).

> [!TIP]
> На подключение к URL-адресу потока через протокол WebSocket выделяется 60 секунд. Чтобы создать новый URL-адрес потока, если в течение этого времени не удается установить подключение, можно [повторно подключиться к общению](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md).

## <a name="start-conversation-versus-generate-token"></a>Сравнение операций "Начало общения" и "Создать маркер"

Операция "создать общение" (`POST /v3/directline/conversations`) аналогична операции [создать маркер Direct Line](bot-framework-rest-direct-line-3-0-authentication.md#generate-token) (`POST /v3/directline/tokens/generate`) в том, что обе операции возвращают `token`, который можно использовать для доступа к одному общению. Тем не менее операция "Начало общения" также начинает общение, контактирует с ботом и создает URL-адрес потока протокола WebSocket, тогда как операция "создание маркера" не поддерживает ни одно из вышеупомянутых действий. 

Если требуется немедленно начать общение, используйте операцию "Начало общения". Если требуется распространить маркер среди клиентов, а также заставить их начать общение, используйте операцию [создание маркера Direct Line](bot-framework-rest-direct-line-3-0-authentication.md#generate-token). 

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-3-0-concepts.md)
- [Аутентификация](bot-framework-rest-direct-line-3-0-authentication.md)
- [Receive activities via WebSocket stream](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) (Получение действий через поток по протоколу WebSocket)
- [Reconnect to a conversation](bot-framework-rest-direct-line-3-0-reconnect-to-conversation.md) (Повторное подключение к общению)
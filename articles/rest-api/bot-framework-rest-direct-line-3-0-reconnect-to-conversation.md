---
title: Повторное подключение к диалогу | Документация Майкрософт
description: Сведения о повторном подключении к диалогу с помощью API Direct Line версии 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2c6b3a7e9f0fdc7d5227fc8112cb6f3e330a2bcc
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305026"
---
# <a name="reconnect-to-a-conversation"></a>Повторное подключение к диалогу

Если клиент использует [интерфейс WebSocket](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) для получения сообщений, но теряет подключение, может возникнуть необходимость повторного подключения. В этом сценарии клиент должен создать новый URL-адрес потока по протоколу WebSocket, который можно использовать для повторного подключения к диалогу.

## <a name="generate-a-new-websocket-stream-url"></a>Создание нового URL-адреса потока по протоколу WebSocket

Чтобы создать новый URL-адрес потока по протоколу WebSocket, который можно использовать для повторного подключения к существующему диалогу, выполните этот запрос: 

```http
GET https://directline.botframework.com/v3/directline/conversations/{conversationId}?watermark={watermark_value}
Authorization: Bearer SECRET_OR_TOKEN
```

В этом URI запроса замените **{conversationId}** идентификатором диалога, а **{watermark_value}** — значением водяного знака (если указан параметр `watermark`). Параметр `watermark` не обязателен. Если в URI запроса указан параметр `watermark`, диалог воспроизводится начиная с водяного знака, за счет чего ни одно сообщение не будет потеряно. Если в URI запроса параметр `watermark` пропущен, будут воспроизведены только те сообщения, которые были получены после выполнения запроса о повторном подключении.

Ниже приведены примеры фрагментов кода для запроса о повторном подключении и ответа на него.

### <a name="request"></a>Запрос

```http
GET https://directline.botframework.com/v3/directline/conversations/abc123?watermark=0000a-42
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Ответ

Если запрос выполнен, ответ будет содержать идентификатор диалога, маркер и новый URL-адрес потока по протоколу WebSocket.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn",
  "streamUrl": "https://directline.botframework.com/v3/directline/conversations/abc123/stream?watermark=000a-4&amp;t=RCurR_XV9ZA.cwA..."
}
```

## <a name="reconnect-to-the-conversation"></a>Повторное подключение к диалогу

Клиент должен использовать новый URL-адрес потока по протоколу WebSocket, чтобы [повторно подключиться к диалогу](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) в течение 60 секунд. Если в течение этого времени не удается установить подключение, клиент должен выполнить новый запрос на повторное подключение, чтобы создать новый URL-адрес потока.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-3-0-concepts.md)
- [Аутентификация](bot-framework-rest-direct-line-3-0-authentication.md)
- [Receive activities via WebSocket stream](bot-framework-rest-direct-line-3-0-receive-activities.md#connect-via-websocket) (Получение действий через поток по протоколу WebSocket)
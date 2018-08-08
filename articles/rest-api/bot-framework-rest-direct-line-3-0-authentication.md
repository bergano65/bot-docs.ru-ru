---
title: Проверка подлинности | Документы Майкрософт
description: Сведения о проверке подлинности для запросов к API Direct Line версии 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 18a5815a96e2052a54c48f6af211d8b28e20d983
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305262"
---
# <a name="authentication"></a>Authentication

Клиент может проверить подлинность запросов к API Direct Line версии 3.0 с помощью **секрета**, который [можно получить на странице конфигурации канала Direct Line](../bot-service-channel-connect-directline.md) на портале Bot Framework, или с помощью **маркера**, который можно получить во время выполнения. Секрет или маркер необходимо указать в заголовке `Authorization` каждого запроса, используя следующий формат: 

```http
Authorization: Bearer SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>Секреты и маркеры

**Секрет** Direct Line — это главный ключ, который может использоваться для доступа ко всем диалогам, связанным с соответствующим ботом. **Секрет** также может быть использован для получения **маркера**. Срок действия секрета не ограничен. 

**Маркер** Direct Line — это ключ, который может использоваться для доступа к одному диалогу. Срок действия маркера ограничен, но его можно продлить. 

При создании приложения между службами проще всего указать **секрет** в заголовке `Authorization` запросов API Direct Line. Если вы создаете приложение, в котором клиент запускается в веб-браузере или в мобильном приложении, вы можете обменять секрет на маркер (который будет действительным только для одного диалога и перестанет действовать, если не будет продлен) и указать **маркер** в заголовке `Authorization` запросов API Direct Line. Выберите модель безопасности, которая подходит вам лучше всего.

> [!NOTE]
> Учетные данные клиента Direct Line отличаются от учетных данных вашего бота. Это позволяет проверить ключи независимо друг от друга и предоставить маркеры клиента, не раскрывая пароль бота. 

## <a name="get-a-direct-line-secret"></a>Получение секрета Direct Line

Вы можете [получить секрет Direct Line](../bot-service-channel-connect-directline.md) на странице настройки канала Direct Line для вашего бота на <a href="https://dev.botframework.com/" target="_blank">портале Bot Framework</a>:

![Конфигурация Direct Line](../media/direct-line-configure.png)

## <a id="generate-token"></a> Создание маркера Direct Line

Для создания маркера Direct Line, который может использоваться для доступа к одному диалогу, необходимо сначала получить секрет Direct Line на странице настройки канала Direct Line на <a href="https://dev.botframework.com/" target="_blank">портале Bot Framework</a>. Затем выполните следующий запрос, чтобы обменять секрет Direct Line на маркер Direct Line:

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer SECRET
```

В заголовке `Authorization` этого запроса замените **SECRET** на значение секрета Direct Line.

Ниже приведены примеры фрагментов кода для запроса на создание маркера и соответствующего ответа.

### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Ответ

Если запрос выполнен успешно, ответ содержит `token` для одного диалога, а значение `expires_in` указывает количество секунд до истечения срока действия маркера. Чтобы продолжить использование маркера, необходимо [продлить маркер](#refresh-token) до того, как его срок действия истечет.

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

### <a name="generate-token-versus-start-conversation"></a>Сравнение операций создания маркера и начала диалога

Операция создания маркера (`POST /v3/directline/tokens/generate`) аналогична операции [начала диалога](bot-framework-rest-direct-line-3-0-start-conversation.md) (`POST /v3/directline/conversations`) в том, что обе операции возвращают `token`, который можно использовать для доступа к одному диалогу. Тем не менее, в отличие от операции начала диалога, операция создания маркера не начинает диалог, не связывается с ботом и не создает URL-адрес потоковой передачи WebSocket. 

Если вы хотите передать маркер клиентам, а также желаете, чтобы они начали диалог, используйте операцию создания маркера. Если вы хотите начать диалог немедленно, используйте операцию [начала диалога](bot-framework-rest-direct-line-3-0-start-conversation.md).

## <a id="refresh-token"></a> Продление маркера Direct Line

Маркер Direct Line можно продлевать неограниченное количество раз, до тех пор, пока срок его действия не истек. Продлить маркер с истекшим сроком действия невозможно. Чтобы продлить маркер Direct Line, выполните следующий запрос: 

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

В заголовке `Authorization` этого запроса замените **TOKEN_TO_BE_REFRESHED** на маркер Direct Line, который вы хотите обновить.

Ниже приведены примеры фрагментов кода для запроса на обновление маркера и соответствующего ответа.

### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/v3/directline/tokens/refresh
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Ответ

Если запрос выполнен успешно, ответ содержит новый `token`, который действителен для того же диалога, что и предыдущий маркер, а значение `expires_in` указывает количество секунд до истечения срока нового действия маркера. Чтобы продолжить использование нового маркера, необходимо [продлить маркер](#refresh-token) до того, как его срок действия истечет.

```http
HTTP/1.1 200 OK
[other headers]
```

```json
{
  "conversationId": "abc123",
  "token": "RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0",
  "expires_in": 1800
}
```

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-3-0-concepts.md)
- [Подключение бота к Direct Line](../bot-service-channel-connect-directline.md)
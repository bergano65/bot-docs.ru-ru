---
title: Аутентификация | Документация Майкрософт
description: Сведения об аутентификации для запросов к API версии 1.1 для Direct Line.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 555cb3298114c3eb8ba8a4e1c41b5515e929fd91
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70299643"
---
# <a name="authentication"></a>Аутентификация

> [!IMPORTANT]
> В этой статье описана аутентификация в API 1.1 для Direct Line. При создании подключения между клиентским приложением и ботом используйте [API версии 3.0 для Direct Line](bot-framework-rest-direct-line-3-0-authentication.md).

Клиент может выполнить аутентификацию запросов к API Direct Line версии 1.1 с помощью **секрета**, который [доступен на странице конфигурации канала Direct Line](../bot-service-channel-connect-directline.md) на портале Bot Framework, или с помощью **маркера**, который можно получить во время выполнения.

Секрет или маркер нужно указать в заголовке `Authorization` каждого запроса с помощью схемы Bearer или BotConnector. 

**Схема Bearer**:
```http
Authorization: Bearer SECRET_OR_TOKEN
```

**Схема BotConnector**:
```http
Authorization: BotConnector SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>Секреты и маркеры

**Секрет** Direct Line — это главный ключ, который можно использовать для доступа ко всем диалогам, связанным с соответствующим ботом. **Секрет** также можно использовать для получения **маркера**. Срок действия секрета не ограничен. 

**Маркер** Direct Line — это ключ, который можно использовать для доступа к одному диалогу. Срок действия маркера ограничен, но его можно продлить. 

При создании приложения между службами проще всего указать **секрет** в заголовке `Authorization` запросов API Direct Line. Если вы создаете приложение, в котором клиент запускается в веб-браузере или в мобильном приложении, вы можете обменять секрет на маркер (который будет действительным только для одного диалога и перестанет действовать, если не будет продлен) и указать **маркер** в заголовке `Authorization` запросов API Direct Line. Выберите наиболее подходящую вам модель безопасности.

> [!NOTE]
> Учетные данные клиента Direct Line отличаются от учетных данных вашего бота. Это позволяет проверить ключи независимо друг от друга и предоставить маркеры клиента, не раскрывая пароль бота. 

## <a name="get-a-direct-line-secret"></a>Получение секрета Direct Line

Вы можете [получить секрет Direct Line](../bot-service-channel-connect-directline.md) на странице настройки канала Direct Line для вашего бота на <a href="https://dev.botframework.com/" target="_blank">портале Bot Framework</a>:

![Конфигурация Direct Line](../media/direct-line-configure.png)

## <a id="generate-token"></a> Создание маркера Direct Line

Чтобы создать маркер Direct Line, который можно использовать для доступа к одному диалогу, необходимо сначала получить секрет Direct Line на странице настройки канала Direct Line на <a href="https://dev.botframework.com/" target="_blank">портале Bot Framework</a>. Затем выполните следующий запрос, чтобы обменять секрет Direct Line на маркер Direct Line:

```http
POST https://directline.botframework.com/api/tokens/conversation
Authorization: Bearer SECRET
```

В заголовке `Authorization` этого запроса замените **SECRET** значением секрета Direct Line.

Ниже приведены примеры фрагментов кода для запроса на создание маркера и соответствующего ответа.

### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/api/tokens/conversation
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

### <a name="response"></a>Ответ

Если запрос выполнен успешно, ответ будет содержать маркер, который действителен для одного диалога. Срок действия маркера истечет через 30 минут. Чтобы продолжить использование маркера, необходимо [продлить маркер](#refresh-token) до того, как его срок действия истечет.

```http
HTTP/1.1 200 OK
[other headers]

"RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn"
```

### <a name="generate-token-versus-start-conversation"></a>Сравнение операций создания маркера и начала диалога

Операция создания маркера (`POST /api/tokens/conversation`) аналогична операции [начала диалога](bot-framework-rest-direct-line-1-1-start-conversation.md) (`POST /api/conversations`) в том, что обе операции возвращают `token`, который можно использовать для доступа к одному диалогу. Тем не менее, в отличие от операции начала диалога, операция создания маркера не начинает диалог и не связывается с ботом. 

Если вы хотите передать маркер клиентам, а также желаете, чтобы они начали диалог, используйте операцию создания маркера. Если вы хотите начать диалог немедленно, используйте операцию [начала диалога](bot-framework-rest-direct-line-1-1-start-conversation.md).

## <a id="refresh-token"></a> Продление маркера Direct Line

Маркер Direct Line действует в течение 30 минут от момента создания. Его можно продлевать неограниченное количество раз, до тех пор, пока срок его действия не истек. Продлить маркер с истекшим сроком действия невозможно. Чтобы продлить маркер Direct Line, выполните следующий запрос:

```http
POST https://directline.botframework.com/api/tokens/{conversationId}/renew
Authorization: Bearer TOKEN_TO_BE_REFRESHED
```

В URI запроса замените **{conversationId}** идентификатором диалога, для которого действует маркер, а в заголовке `Authorization` этого запроса, замените **TOKEN_TO_BE_REFRESHED** маркером Direct Line, который нужно продлить.

Ниже приведены примеры фрагментов кода для запроса на обновление маркера и соответствующего ответа.

### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/api/tokens/abc123/renew
Authorization: Bearer CurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xn
```

### <a name="response"></a>Ответ

Если запрос выполнен успешно, ответ содержит новый маркер, который действителен для того же диалога, что и предыдущий маркер. Срок действия нового маркера истечет через 30 минут. Чтобы продолжить использование нового маркера, необходимо [продлить маркер](#refresh-token) до того, как его срок действия истечет.

```http
HTTP/1.1 200 OK
[other headers]

"RCurR_XV9ZA.cwA.BKA.y8qbOF5xPGfiCpg4Fv0y8qqbOF5x8qbOF5xniaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0"
```

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-1-1-concepts.md)
- [Подключение бота к Direct Line](../bot-service-channel-connect-directline.md)
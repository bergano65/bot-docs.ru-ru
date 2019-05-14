---
title: Проверка подлинности | Документы Майкрософт
description: Сведения о проверке подлинности для запросов к API Direct Line версии 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/10/2019
ms.openlocfilehash: 717a95d580bad218ade9a884522724f1c6b96ad7
ms.sourcegitcommit: f84b56beecd41debe6baf056e98332f20b646bda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/03/2019
ms.locfileid: "65032637"
---
# <a name="authentication"></a>Authentication

Клиент может проверить подлинность запросов к API Direct Line версии 3.0 с помощью **секрета**, который [можно получить на странице конфигурации канала Direct Line](../bot-service-channel-connect-directline.md) на портале Bot Framework, или с помощью **маркера**, который можно получить во время выполнения. Секрет или маркер необходимо указать в заголовке `Authorization` каждого запроса, используя следующий формат: 

```http
Authorization: Bearer SECRET_OR_TOKEN
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
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer SECRET
```

В заголовке `Authorization` этого запроса замените **SECRET** значением секрета Direct Line.

Ниже приведены примеры фрагментов кода для запроса на создание маркера и соответствующего ответа.

### <a name="request"></a>Запрос

```http
POST https://directline.botframework.com/v3/directline/tokens/generate
Authorization: Bearer RCurR_XV9ZA.cwA.BKA.iaJrC8xpy8qbOF5xnR2vtCX7CZj0LdjAPGfiCpg4Fv0
```

Полезные данные запроса, куда входят параметры маркера, необязательны, но рекомендуются. При создании маркера, который можно отправить обратно в службу Direct Line, предоставьте следующие полезные данные для более безопасного подключения. Благодаря этим значениям Direct Line сможет выполнять дополнительную проверку идентификатора и имени пользователя, препятствуя изменению этих значений злоумышленниками. Также эти значения упрощают для Direct Line отправку действия _обновления диалога_, что позволяет создать обновление беседы немедленно при присоединении пользователя. Если эта информация не предоставляется, Direct Line сможет обновить беседу только после того, как пользователь отправит в нее какое-либо содержимое.

```json
{
  "user": {
    "id": "string",
    "name": "string"
  },
  "trustedOrigins": [
    "string"
  ]
}
```

| Параметр | type | ОПИСАНИЕ |
| :--- | :--- | :--- |
| `user.id` | строка | Необязательный элемент. Идентификатор пользователя в конкретном канале, который кодируется в маркере. Для пользователя Direct Line это значение начинается с `dl_`. Вы можете создать уникальный идентификатор пользователя для каждой беседы, а для повышения безопасности следует исключить возможность угадать этот идентификатор. |
| `user.name` | строка | Необязательный элемент. Понятное отображаемое имя пользователя, которое кодируется в маркере. |
| `trustedOrigins` | массив строк | Необязательный элемент. Список доверенных доменов для внедрения в маркер. Здесь перечисляются домены, которые могут размещать клиент веб-чата для бота. Этот список должен совпадать с тем, который указан на странице настройки Direct Line для бота. |

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
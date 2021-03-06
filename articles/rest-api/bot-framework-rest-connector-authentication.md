---
title: Запросы проверки подлинности — Служба Azure Bot
description: Сведения о проверке подлинности запросов API в API службы Bot Connector и API службы "Состояние бота".
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 60a246d60f3b74b037f793a306d58df1a5398141
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790792"
---
# <a name="authentication"></a>Аутентификация

Бот взаимодействует со службой Bot Connector по протоколу HTTP через защищенный канал (SSL/TLS). Запрос, отправляемый ботом в службу Bot Connector, должен содержать сведения, с помощью которых служба будет проверять удостоверение бота. Точно так же, когда служба Bot Connector отправляет запрос боту, она должна включить в него сведения, с помощью которых бот проверит удостоверение службы. В этой статье описываются технологии проверки подлинности и требования к проверке подлинности уровня службы, выполняемой между ботом и службой Bot Connector. При написании собственного кода для проверки подлинности необходимо реализовать описанные в этой статье процедуры безопасности, чтобы бот мог обмениваться сообщениями со службой Bot Connector.

> [!IMPORTANT]
> Очень важно сделать это правильно. После выполнения всех действий в этой статье снижается вероятность прочтения злоумышленником сообщений, отправляемых боту, уменьшается риск отправки сообщений, олицетворяющих бот, и кражи секретных ключей. 

Если вы используете [пакет SDK Bot Framework для .NET](../dotnet/bot-builder-dotnet-overview.md) или [пакет SDK Bot Framework для Node.js](../nodejs/index.md), выполнять описанные в этой статье процедуры безопасности не требуется, так как эту задачу автоматически реализует пакет SDK. Просто настройте проект с использованием идентификатора приложения и пароля, полученных для бота во время [регистрации](../bot-service-quickstart-registration.md), а все остальное сделает пакет SDK.

> [!WARNING]
> В декабре 2016 г. в протоколе безопасности Bot Framework версии 3.1 были представлены изменения для нескольких значений, которые используются во время создания и проверки маркеров. В конце осени 2017 г. был представлен протокол безопасности Bot Framework версии 3.2 с внесенными изменениями для значений, которые используются во время создания и проверки маркеров.
> Дополнительные сведения см. в разделе об [изменениях в протоколе безопасности](#security-protocol-changes).

## <a name="authentication-technologies"></a>Технологии проверки подлинности

Для установления доверительных отношений между ботом и службой Bot Connector используются следующие четыре технологии проверки подлинности.

| Технология | Description |
|----|----|
| **SSL/TLS** | SSL/TLS используется для всех подключений между службами. Сертификаты `X.509v3` используются для идентификации всех служб HTTPS. **Клиенты всегда должны проверять сертификаты службы, чтобы убедиться в их надежности и допустимости.** (В рамках этой схемы сертификаты клиента не используются.) |
| **OAuth 2.0** | OAuth 2.0 создает маркер безопасности с использованием службы входа в учетную запись Azure Active Directory (Azure AD) версии 2 для. С помощью этого маркера которого бот может отправлять сообщения. Этот маркер работает между службами. Вход пользователя не требуется. |
| **JSON Web Token (JWT)** | JSON Web Token используется для кодирования маркеров, отправляемых ботам и из них. **Клиенты должны полностью проверять все получаемые маркеры JWT** в соответствии с требованиями, описанными в этой статье. |
| **Метаданные OpenID** | Служба Bot Connector публикует список допустимых маркеров, используемых для подписи собственных маркеров JWT в метаданных OpenID в хорошо известной статической конечной точке. |

В этой статье описывается, как использовать эти технологии с помощью стандартных форматов HTTPS и JSON. Специальные пакеты SDK не требуются, однако могут быть полезны вспомогательные функции для OpenID и т. д.

## <a id="bot-to-connector"></a> Проверка подлинности запросов от бота к службе Bot Connector

Для взаимодействия со службой Bot Connector необходимо указать маркер доступа в заголовке `Authorization` каждого запроса API, используя следующий формат: 

```http
Authorization: Bearer ACCESS_TOKEN
```

На этой схеме показаны действия для проверки подлинности бота в службе:

![Проверка подлинности в службе входа MSA и затем в боте](../media/connector/auth_bot_to_bot_connector.png)

### <a name="step-1-request-an-access-token-from-the-azure-ad-v2-account-login-service"></a>Шаг 1. Запрос маркера доступа из службы входа в учетную запись Azure AD версии 2

> [!IMPORTANT]
> Если это еще не сделано, [зарегистрируйте бот](../bot-service-quickstart-registration.md) в Bot Framework, чтобы получить идентификатор приложения и пароль. Идентификатор приложения и пароль бота потребуются для запроса маркера доступа.

Чтобы запросить маркер доступа из службы входа, выполните приведенный ниже запрос, заменив **MICROSOFT-APP-ID** и **MICROSOFT-APP-PASSWORD** идентификатором приложения и паролем бота, которые вы получили при [регистрации](../bot-service-quickstart-registration.md) бота в Bot Framework.

```http
POST https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials&client_id=MICROSOFT-APP-ID&client_secret=MICROSOFT-APP-PASSWORD&scope=https%3A%2F%2Fapi.botframework.com%2F.default
```

### <a name="step-2-obtain-the-jwt-token-from-the-the-azure-ad-v2-account-login-service-response"></a>Шаг 2. Получение маркера JWT из ответа службы входа в учетную запись Azure AD версии 2

Если приложение авторизовано службой входа, в тексте ответа JSON будет указан маркер доступа, его тип и срок действия (в секундах).

При добавлении маркера в заголовок `Authorization` запроса необходимо использовать точное значение, которое указано в этом ответе (т. е. не экранировать и не кодировать значение маркера). Маркер доступа действителен до окончания срока его действия. Чтобы предотвратить влияние окончания срока действия маркера на производительность бота, маркер можно кэшировать и заранее обновить.

В этом примере показан ответ службы входа в учетную запись Azure AD версии 2.

```http
HTTP/1.1 200 OK
... (other headers)
```

```json
{
    "token_type":"Bearer",
    "expires_in":3600,
    "ext_expires_in":3600,
    "access_token":"eyJhbGciOiJIUzI1Ni..."
}
```

### <a name="step-3-specify-the-jwt-token-in-the-authorization-header-of-requests"></a>Шаг 3. Указание маркера JWT в заголовке авторизации запросов

При отправке запроса API в службу Bot Connector укажите маркер доступа в заголовке `Authorization` каждого запроса API, используя следующий формат:

```http
Authorization: Bearer ACCESS_TOKEN
```

Все запросы, отправляемые в службу Bot Connector, должны содержать маркер доступа в заголовке `Authorization`.
Если маркер имеет правильный формат, является действующим и был создан службой входа в учетную запись Azure AD версии 2, служба Bot Connector авторизует запрос. Чтобы убедиться, что маркер принадлежит боту, который отправил запрос, выполняются дополнительные проверки.

В следующем примере показано, как указать маркер доступа в заголовке `Authorization` запроса. 

```http
POST https://smba.trafficmanager.net/apis/v3/conversations/12345/activities 
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
    
(JSON-serialized Activity message goes here)
```

> [!IMPORTANT]
> Маркер JWT следует задавать только в заголовке `Authorization` запросов, отправляемых в службу Bot Connector.
> Не отправляйте маркер через незащищенные каналы и не включайте его в HTTP-запросы, отправляемые в другие службы.
> Маркер JWT, полученный из службы входа в учетную запись Azure AD версии 2, аналогичен паролю, и при работе с ним нужно соблюдать меры безопасности. Пользователь, обладающий маркером, может использовать его для выполнения операций от имени вашего бота.

#### <a name="bot-to-connector-example-jwt-components"></a>Запрос от бота к службе Bot Connector: пример компонентов JWT

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "https://api.botframework.com",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  appid: "<YOUR MICROSOFT APP ID>",
  ... other fields follow
}
```

> [!NOTE]
> На практике могут использоваться другие поля. Создайте и проверьте все маркеры JWT, как указано выше.

## <a id="connector-to-bot"></a> Проверка подлинности запросов от службы Bot Connector к боту

Когда служба Bot Connector отправляет запрос боту, она указывает подписанный маркер JWT в заголовке `Authorization` запроса. Бот может проверить подлинность вызовов от службы Bot Connector, проверив подлинность подписанного маркера JWT. 

На этой схеме показаны действия для проверки подлинности службы в боте:

![Проверка подлинности вызовов из службы Bot Connector к боту](../media/connector/auth_bot_connector_to_bot.png)

### <a id="openid-metadata-document"></a> Шаг 2. Получение документа метаданных OpenID

Документ метаданных OpenID указывает расположение второго документа, в котором приведены допустимые ключи подписи службы Bot Connector. Чтобы получить документ метаданных OpenID, выполните следующий запрос по протоколу HTTPS:

```http
GET https://login.botframework.com/v1/.well-known/openidconfiguration
```

> [!TIP]
> Это статический URL-адрес, который можно жестко задать в приложении. 

В следующем примере показан документ метаданных OpenID, возвращаемый в ответ на запрос `GET`. Свойство `jwks_uri` указывает расположение документа, который содержит допустимые ключи подписи службы Bot Connector.

```json
{
    "issuer": "https://api.botframework.com",
    "authorization_endpoint": "https://invalid.botframework.com",
    "jwks_uri": "https://login.botframework.com/v1/.well-known/keys",
    "id_token_signing_alg_values_supported": [
      "RS256"
    ],
    "token_endpoint_auth_methods_supported": [
      "private_key_jwt"
    ]
}
```

### <a id="connector-to-bot-step-3"></a> Шаг 3. Получение списка допустимых ключей подписи

Чтобы получить список допустимых ключей подписи, отправьте запрос `GET` по протоколу HTTPS на URL-адрес, указанный в свойстве `jwks_uri` в документе метаданных OpenID. Пример:

```http
GET https://login.botframework.com/v1/.well-known/keys
```

В тексте ответа указан документ в [формате JWK](https://tools.ietf.org/html/rfc7517) и также содержатся дополнительные свойства для каждого ключа: `endorsements`. Список ключей относительно стабилен и может кэшироваться в течение длительного времени (по умолчанию — 5 дней в пакете SDK Bot Framework).

Свойство `endorsements` в каждом ключе содержит одну или несколько строк подтверждения, которые можно использовать для аутентификации идентификатора канала, указанного в свойстве `channelId` в объекте [Действие][] входящего запроса. Список идентификаторов каналов, требующих подтверждения, настраивается в каждом боте. По умолчанию в список будут входить все опубликованные идентификаторы каналов, однако разработчики ботов могут переопределить выбранные значения идентификаторов. 

### <a name="step-4-verify-the-jwt-token"></a>Шаг 4. Проверка маркера JWT

Чтобы проверить подлинность маркера, отправленного службой Bot Connector, необходимо извлечь его из заголовка `Authorization` запроса, проанализировать, проверить его содержимое и подпись. 

Библиотеки анализа JWT доступны для целого ряда платформ, и большинство из них предоставляет безопасные и надежные возможности анализа маркеров JWT, несмотря на то, что обычно эти библиотеки требуется настраивать для указания правильных значений в характеристиках маркера (например, издатель, аудитория и проч.). При анализе маркера необходимо настроить библиотеку анализа или написать собственную процедуру проверки маркера, чтобы маркер соответствовал приведенным ниже требованиям.

1. Маркер был отправлен в заголовке HTTP `Authorization` со схемой носителя.
2. Маркер имеет допустимый формат JSON, соответствующий [стандарту JWT](http://openid.net/specs/draft-jones-json-web-token-07.html).
3. Маркер содержит утверждение "issuer" со значением `https://api.botframework.com`.
4. Маркер содержит утверждение "audience" со значением, соответствующим идентификатору приложения Майкрософт для бота.
5. Маркер является действующим. Отраслевая разница в показаниях часов составляет 5 минут.
6. Маркер имеет допустимую криптографическую подпись с ключом, приведенным в документе с ключами OpenID, который был получен на [шаге 3](#connector-to-bot-step-3), и алгоритмом подписи, который указан в свойстве `id_token_signing_alg_values_supported`документа метаданных OpenID, который был получен на [шаге 2](#openid-metadata-document).
7. Маркер содержит утверждение serviceUrl со значением, которое соответствует свойству `servieUrl` в корне объекта [Действие][] входящего запроса. 

Если требуется подтверждение для идентификатора канала:

- любой объект `Activity`, отправляемый в бот с этим идентификатором канала, должен сопровождаться маркером JWT, который подписан с использованием подтверждения для этого канала; 
- если подтверждение отсутствует, бот должен отклонить запрос, возвратив код состояния **HTTP 403 (Запрещено)** .

> [!IMPORTANT]
> Все эти требования являются важными, особенно требования 4 и 6. При невозможности выполнить все эти требования бот будет открыт для атак, что может привести к раскрытию его маркера JWT.

Разработчики должны следить за тем, чтобы проверка маркера JWT, отправляемого в бот, всегда была включена.

#### <a name="connector-to-bot-example-jwt-components"></a>Запрос от службы Bot Connector к боту: пример компонентов JWT

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOU MICROSOFT APP ID>",
  iss: "https://api.botframework.com",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> На практике могут использоваться другие поля. Создайте и проверьте все маркеры JWT, как указано выше.

## <a id="emulator-to-bot"></a> Проверка подлинности запросов от Bot Framework Emulator к боту

> [!WARNING]
> В конце осени 2017 г. был представлен протокол безопасности Bot Framework версии 3.2. Новая версия содержит новое значение "issuer" в маркерах, которыми обмениваются Bot Framework Eumaltor и бот. Чтобы подготовиться к этому изменению, ознакомьтесь с приведенными ниже действиями по проверке значений "issuer" в версиях 3.2 и 3.1. 

[Bot Framework Emulator](../bot-service-debug-emulator.md) — это инструмент для настольных систем, используемый для тестирования работоспособности бота. Несмотря на то, что в Bot Framework Emulator применяются те же [технологии проверки подлинности](#authentication-technologies), которые были описаны выше, это средство не может олицетворять реальную службу Bot Connector. Оно использует идентификатор приложения Майкрософт и пароль приложения Майкрософт, указываемые при подключении эмулятора к боту для создания маркеров, аналогичных создаваемых ботом. Когда эмулятор отправляет запрос боту, он указывает маркер JWT в заголовке `Authorization` запроса и, по сути, использует учетные данные бота для проверки подлинности запроса. 

Если вы реализуете библиотеку проверки подлинности и хотите принимать запросы от Bot Framework Emulator, необходимо добавить этот путь дополнительной проверки. По своей структуре путь подобен пути проверки [От службы Bot Connector к боту](#connector-to-bot), но вместо документа OpenID службы Bot Connector в нем используется документ OpenID MSA.

На этой схеме показаны действия для проверки подлинности эмулятора в боте:

![Проверка подлинности вызовов из Bot Framework Emulator к боту](../media/connector/auth_bot_framework_emulator_to_bot.png)

---
### <a name="step-2-get-the-msa-openid-metadata-document"></a>Шаг 2. Получение документа метаданных OpenID MSA

Документ метаданных OpenID указывает расположение второго документа, в котором перечислены допустимые ключи подписи. Чтобы получить документ метаданных OpenID MSA, отправьте следующий запрос по протоколу HTTPS:

```http
GET https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration
```

В следующем примере показан документ метаданных OpenID, возвращаемый в ответ на запрос `GET`. Свойство `jwks_uri` указывает расположение документа, который содержит допустимые ключи подписи.

```json
{
    "authorization_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/authorize",
    "token_endpoint":"https://login.microsoftonline.com/common/oauth2/v2.0/token",
    "token_endpoint_auth_methods_supported":["client_secret_post","private_key_jwt"],
    "jwks_uri":"https://login.microsoftonline.com/common/discovery/v2.0/keys",
    ...
}
```

### <a id="emulator-to-bot-step-3"></a> Шаг 3. Получение списка допустимых ключей подписи

Чтобы получить список допустимых ключей подписи, отправьте запрос `GET` по протоколу HTTPS на URL-адрес, указанный в свойстве `jwks_uri` в документе метаданных OpenID. Пример:

```http
GET https://login.microsoftonline.com/common/discovery/v2.0/keys 
Host: login.microsoftonline.com
```

В тексте ответа документ указывается в [формате JWK](https://tools.ietf.org/html/rfc7517). 

### <a name="step-4-verify-the-jwt-token"></a>Шаг 4. Проверка маркера JWT

Чтобы проверить подлинность маркера, отправленного эмулятором, необходимо извлечь его из заголовка `Authorization` запроса, проанализировать, проверить его содержимое и подпись. 

Библиотеки анализа JWT доступны для целого ряда платформ, и большинство из них предоставляет безопасные и надежные возможности анализа маркеров JWT, несмотря на то, что обычно эти библиотеки требуется настраивать для указания правильных значений в характеристиках маркера (например, издатель, аудитория и проч.). При анализе маркера необходимо настроить библиотеку анализа или написать собственную процедуру проверки маркера, чтобы маркер соответствовал приведенным ниже требованиям.

1. Маркер был отправлен в заголовке HTTP `Authorization` со схемой носителя.
2. Маркер имеет допустимый формат JSON, соответствующий [стандарту JWT](http://openid.net/specs/draft-jones-json-web-token-07.html).
3. Маркер содержит утверждение "issuer" со значением `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` или `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/`. (Проверка обоих значений "issuer" гарантирует, что эти значения проверяются в протоколе безопасности версий 3.1 и 3.2.)
4. Маркер содержит утверждение "audience" со значением, соответствующим идентификатору приложения Майкрософт для бота.
5. Маркер содержит утверждение "appid" со значением, соответствующим идентификатору приложения Майкрософт для бота.
6. Маркер является действующим. Отраслевая разница в показаниях часов составляет 5 минут.
7. Маркер имеет допустимый криптографическую подпись с ключом, приведенным в документе с ключами OpenID, который был получен на [шаге 3](#emulator-to-bot-step-3).

> [!NOTE]
> Требование 5 применяется к пути проверки эмулятора. 

Если маркер не соответствует всем требованиям, бот должен завершить запрос, возвратив код состояния **HTTP 403 (Запрещено)** .

> [!IMPORTANT]
> Все эти требования являются важными, особенно требования 4 и 7. При невозможности выполнить все эти требования бот будет открыт для атак, что может привести к раскрытию его маркера JWT.

#### <a name="emulator-to-bot-example-jwt-components"></a>Запрос от эмулятора к боту: пример компонентов JWT

```json
header:
{
  typ: "JWT",
  alg: "RS256",
  x5t: "<SIGNING KEY ID>",
  kid: "<SIGNING KEY ID>"
},
payload:
{
  aud: "<YOUR MICROSOFT APP ID>",
  iss: "https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/",
  nbf: 1481049243,
  exp: 1481053143,
  ... other fields follow
}
```

> [!NOTE]
> На практике могут использоваться другие поля. Создайте и проверьте все маркеры JWT, как указано выше.

## <a name="security-protocol-changes"></a>Изменения протокола безопасности

> [!WARNING]
> Поддержка протокола безопасности версии 3.0 была прекращена **31 июля 2017 г**. Если вы написали собственный код проверки подлинности (т. е. не использовали для создания бота пакет SDK Bot Framework), следует обновить протокол безопасности до версии 3.1 путем обновления приложения для использования приведенных ниже значений версии 3.1. 

### <a name="bot-to-connector-authenticationbot-to-connector"></a>[Проверка подлинности бота в службе Bot Connector](#bot-to-connector)

#### <a name="oauth-login-url"></a>URL-адрес входа OAuth

| Версия протокола | Допустимое значение |
|----|----|
| 3\.1 и 3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>Область OAuth

| Версия протокола | Допустимое значение |
|----|----|
| 3\.1 и 3.2 |  `https://api.botframework.com/.default` |

### <a name="connector-to-bot-authenticationconnector-to-bot"></a>[Проверка подлинности службы Bot Connector в боте](#connector-to-bot)

#### <a name="openid-metadata-document"></a>Документ метаданных OpenID

| Версия протокола | Допустимое значение |
|----|----|
| 3\.1 и 3.2 | `https://login.botframework.com/v1/.well-known/openidconfiguration` |

#### <a name="jwt-issuer"></a>Издатель JWT

| Версия протокола | Допустимое значение |
|----|----|
| 3\.1 и 3.2 | `https://api.botframework.com` |

### <a name="emulator-to-bot-authenticationemulator-to-bot"></a>[Проверка подлинности эмулятора в боте](#emulator-to-bot)

#### <a name="oauth-login-url"></a>URL-адрес входа OAuth

| Версия протокола | Допустимое значение |
|----|----|
| 3\.1 и 3.2 | `https://login.microsoftonline.com/botframework.com/oauth2/v2.0/token` |

#### <a name="oauth-scope"></a>Область OAuth

| Версия протокола | Допустимое значение |
|----|----|
| 3\.1 и 3.2 |  Идентификатор приложения Майкрософт для вашего бота и `/.default` |

#### <a name="jwt-audience"></a>Аудитория JWT

| Версия протокола | Допустимое значение |
|----|----|
| 3\.1 и 3.2 | Идентификатор приложения Майкрософт для вашего бота |

#### <a name="jwt-issuer"></a>Издатель JWT

| Версия протокола | Допустимое значение |
|----|----|
| 3\.1 | `https://sts.windows.net/d6d49420-f39b-4df7-a1dc-d59a935871db/` |
| 3\.2 | `https://sts.windows.net/f8cdef31-a31e-4b4a-93e4-5f571e91255a/` |

#### <a name="openid-metadata-document"></a>Документ метаданных OpenID

| Версия протокола | Допустимое значение |
|----|----|
| 3\.1 и 3.2 | `https://login.microsoftonline.com/botframework.com/v2.0/.well-known/openid-configuration` |

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Устранение неполадок проверки подлинности Bot Framework](../bot-service-troubleshoot-authentication-problems.md)
- [Принципы использования действий в Bot Framework](https://aka.ms/botSpecs-activitySchema)
- [JSON Web Token (JWT) draft-jones-json-web-token-07](http://openid.net/specs/draft-jones-json-web-token-07.html)
- [JSON Web Signature (JWS) draft-jones-json-web-signature-04](https://tools.ietf.org/html/draft-jones-json-web-signature-04)
- [Документ RFC 7517 для JSON Web Key (JWK)](https://tools.ietf.org/html/rfc7517)

[Действие]: bot-framework-rest-connector-api-reference.md#activity-object

---
title: Проверка подлинности — Служба Azure Bot
description: Сведения о проверке подлинности для запросов к API Direct Line версии 3.0.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 08/22/2019
ms.openlocfilehash: 59a97acffe26b0bb896ec75dd5ca03fa43eafd67
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789524"
---
# <a name="authentication"></a>Аутентификация

Клиент может проверить подлинность запросов к API Direct Line версии 3.0 с помощью **секрета**, который [можно получить на странице конфигурации канала Direct Line](../bot-service-channel-connect-directline.md) на портале Bot Framework, или с помощью **маркера**, который можно получить во время выполнения. Секрет или маркер необходимо указать в заголовке `Authorization` каждого запроса, используя следующий формат: 

```http
Authorization: Bearer SECRET_OR_TOKEN
```

## <a name="secrets-and-tokens"></a>Секреты и маркеры

**Секрет** Direct Line — это главный ключ, который можно использовать для доступа ко всем диалогам, связанным с соответствующим ботом. **Секрет** также можно использовать для получения **маркера**. Срок действия секрета не ограничен. 

**Маркер** Direct Line — это ключ, который можно использовать для доступа к одному диалогу. Срок действия маркера ограничен, но его можно продлить.

При принятии решения о том, следует ли использовать **секретный ключ** или **токен** и когда это нужно делать, нужно учитывать вопросы безопасности.
Секретный ключ должен предоставляться с какой-то целью (намеренно) и с осторожностью. Собственно, это поведение по умолчанию, благодаря чему Direct Line может определять, является ли клиент допустимым.
Но в целом, если вы пытаетесь сохранить данные пользователя, обеспечение безопасности является критически важным.
См. сведения о [вопросах безопасности](#security-considerations).

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

| Параметр | Тип | Description |
| :--- | :--- | :--- |
| `user.id` | строка | Необязательный параметр. Идентификатор пользователя в конкретном канале, который кодируется в маркере. Для пользователя Direct Line это значение начинается с `dl_`. Вы можете создать уникальный идентификатор пользователя для каждой беседы, а для повышения безопасности следует исключить возможность угадать этот идентификатор. |
| `user.name` | строка | Необязательный параметр. Понятное отображаемое имя пользователя, которое кодируется в маркере. |
| `trustedOrigins` | массив строк | Необязательный параметр. Список доверенных доменов для внедрения в маркер. Здесь перечисляются домены, которые могут размещать клиент веб-чата для бота. Этот список должен совпадать с тем, который указан на странице настройки Direct Line для бота. |

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

## <a name="azure-bot-service-authentication"></a>Проверка подлинности службы Azure Bot

Сведения, представленные в этом разделе, основаны на инструкциях по [добавлению проверки подлинности к боту с помощью службы Azure Bot](../v4sdk/bot-builder-authentication.md).

**Функция проверки подлинности службы Azure Bot** позволяет проверять подлинность пользователей и **получать маркеры доступа** от различных поставщиков удостоверений, например *Azure Active Directory*, *GitHub*, *Uber* и т. д. Кроме того, настроить проверку подлинности можно для пользовательского поставщика удостоверений **OAuth2**. Все это позволяет использовать только **один фрагмент кода проверки подлинности**, который будет работать для всех поддерживаемых поставщиков удостоверений и всех каналов. Чтобы использовать эти возможности, выполните следующие действия:

1. Статически настройте `settings` в боте, который содержит сведения о регистрации приложения у поставщика удостоверений.
2. Используйте `OAuthCard`, указав сведения о приложении, которые используются для входа пользователя и были указаны на предыдущем шаге.
3. Получите маркеры доступа с помощью **API службы Azure Bot**.

### <a name="security-considerations"></a>Вопросы безопасности

<!-- Summarized from: https://blog.botframework.com/2018/09/25/enhanced-direct-line-authentication-features/ -->

При использовании *проверки подлинности службы Azure Bot* для [Web Chat](../bot-service-channel-connect-webchat.md) важно учитывать несколько моментов, связанных с безопасностью.

1. **Олицетворение**. Здесь олицетворение означает действие злоумышленника, в результате которого бот считает его другим пользователем. В Web Chat злоумышленник может олицетворять кого-то, изменив **идентификатор пользователя** в активном экземпляре канала. Чтобы избежать этого, разработчики должны сделать так, чтобы **идентификатор пользователя нельзя было угадать**. Если включить **расширенную проверку подлинности**, служба Azure Bot сможет обнаруживать и отклонять любые изменения идентификатора пользователя. Это означает, что идентификатор пользователя (`Activity.From.Id`) в сообщениях, отправляемых боту из Direct Line, всегда будет тем же, с которым вы инициализировали экземпляр Web Chat. Обратите внимание, что для использования этой функции идентификатор пользователя должен начинаться с `dl_`.
1. **Удостоверения пользователей**. Необходимо учитывать, что при работе используется два удостоверения пользователей:

    1. Удостоверение пользователя в канале.
    1. Удостоверение пользователя в поставщике удостоверений для бота.
  
    Когда бот запрашивает у пользователя А в канале выполнить вход в поставщик удостоверений П, процесс входа должен гарантировать, что именно пользователь A выполняет вход в П. Если другому пользователю Б разрешено входить в систему, пользователь A сможет получить доступ к ресурсу пользователя Б через бота. Web Chat предоставляет два механизма, обеспечивающих надлежащий процесс входа, как описано далее.

    1. Раньше по завершении входа пользователю предоставлялся сформированный случайным образом код из шести цифр (магический код). Пользователь должен был ввести этот код в диалоге, который инициировал завершение процесса входа. Как правило, такой механизм неудобно использовать. Кроме того, он обычно подвергается фишинговым атакам. Злоумышленник может обманным путем вынудить другого пользователя выполнить вход, чтобы получить магический код путем фишинга.

    2. Из-за проблем с предыдущим подходом мы решили не использовать в службе Azure Bot магический код. Служба Azure Bot гарантирует, что процесс входа может выполняться только **в том же сеансе браузера**, в котором запущен экземпляр Web Chat. 
    Чтобы включить такую защиту, разработчик бота должен запустить Web Chat с **маркером Direct Line**, содержащим **список доверенных доменов, которым разрешено размещать клиент Web Chat для бота**. Ранее этот токен можно получить, только передав незарегистрированный необязательный параметр в API Direct Line. Но теперь благодаря расширенным параметрам проверки подлинности можно настроить статический список доверенных доменов (источников) на странице настройки Direct Line.

    См. сведения о [включении проверки подлинности для бота с помощью службы Azure Bot](../v4sdk/bot-builder-authentication.md).

### <a name="code-examples"></a>Примеры кода

Следующий контроллер .NET использует включенные расширенные параметры проверки подлинности и возвращает маркер Direct Line и идентификатор пользователя.

```csharp
public class HomeController : Controller
{
    public async Task<ActionResult> Index()
    {
        var secret = GetSecret();

        HttpClient client = new HttpClient();

        HttpRequestMessage request = new HttpRequestMessage(
            HttpMethod.Post,
            $"https://directline.botframework.com/v3/directline/tokens/generate");

        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", secret);

        var userId = $"dl_{Guid.NewGuid()}";

        request.Content = new StringContent(
            JsonConvert.SerializeObject(
                new { User = new { Id = userId } }),
                Encoding.UTF8,
                "application/json");

        var response = await client.SendAsync(request);
        string token = String.Empty;

        if (response.IsSuccessStatusCode)
        {
            var body = await response.Content.ReadAsStringAsync();
            token = JsonConvert.DeserializeObject<DirectLineToken>(body).token;
        }

        var config = new ChatConfig()
        {
            Token = token,
            UserId = userId  
        };

        return View(config);
    }
}

public class DirectLineToken
{
    public string conversationId { get; set; }
    public string token { get; set; }
    public int expires_in { get; set; }
}
public class ChatConfig
{
    public string Token { get; set; }
    public string UserId { get; set; }
}

```

Следующий контроллер JavaScript использует включенные расширенные параметры проверки подлинности и возвращает маркер Direct Line и идентификатор пользователя.

```javascript
var router = express.Router(); // get an instance of the express Router

// Get a directline configuration (accessed at GET /api/config)
const userId = "dl_" + createUniqueId();

router.get('/config', function(req, res) {
    const options = {
        method: 'POST',
        uri: 'https://directline.botframework.com/v3/directline/tokens/generate',
        headers: {
            'Authorization': 'Bearer ' + secret
        },
        json: {
            User: { Id: userId }
        }
    };

    request.post(options, (error, response, body) => {
        if (!error && response.statusCode < 300) {
            res.json({ 
                    token: body.token,
                    userId: userId
                });
        }
        else {
            res.status(500).send('Call to retrieve token from Direct Line failed');
        } 
    });
});

```

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Основные понятия](bot-framework-rest-direct-line-3-0-concepts.md)
- [Подключение бота к Direct Line](../bot-service-channel-connect-directline.md)
- [Добавление аутентификации для бота с помощью службы Azure Bot](../bot-builder-tutorial-authentication.md)

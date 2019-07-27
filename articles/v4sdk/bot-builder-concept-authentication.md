---
title: Проверка подлинности пользователей в службе Azure Bot | Документация Майкрософт
description: Узнайте о функциях проверки подлинности пользователей в службе Azure Bot.
keywords: azure bot service, authentication, bot framework token service
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: abs
ms.date: 05/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6eea954f58096d89cd3278058146b93fa04f435f
ms.sourcegitcommit: e276008fb5dd7a37554e202ba5c37948954301f1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 06/05/2019
ms.locfileid: "66693764"
---
# <a name="user-authentication-within-a-conversation"></a>Проверка подлинности пользователей в диалоге

Для выполнения определенных операций от имени пользователя, включая проверку электронной почты, создание ссылки на календарь, проверку состояния рейсов или размещение заказа, боту нужно вызвать внешнюю службу, например Microsoft Graph, GitHub или корпоративные службы REST.
Каждая из этих внешних служб определенным образом защищает эти вызовы. Чаще всего для этого запросы выдаются с использованием _токена пользователя_, который однозначно определяет пользователя во внешней службе (JWT).

Чтобы защитить вызов к внешней службе, бот должен попросить пользователя выполнить вход, чтобы он получил токен пользователя для этой службы.
Многие службы поддерживают извлечение токенов по протоколу OAuth или OAuth2.
Служба Azure Bot предоставляет специализированные карты для входа и службы, которые работают с протоколом OAuth и управляют жизненным циклом токенов. Бот может использовать эти функции, чтобы получить токен пользователя.

- Как часть конфигурации бота в ресурсе службы Azure Bot в Azure регистрируется _параметр подключения OAuth_.

    Каждый параметр подключения содержит сведения об используемых внешней службе или поставщике удостоверений, секрет и допустимый идентификатор клиента OAuth, области OAuth для включения, а также другие метаданные подключения, требуемые внешней службой или поставщиком удостоверений.

- В коде бота параметр подключения OAuth используется для входа пользователей и получения токена пользователя.

Эти службы используются в рабочем процессе входа.

![Общие сведения об аутентификации](./media/bot-builder-concept-authentication.png)

## <a name="about-the-bot-framework-token-service"></a>О службе токенов Bot Framework

Служба токенов Bot Framework отвечает за следующие задачи:

- упрощение использования протокола OAuth с разными внешними службами;
- безопасное хранение токенов для определенного бота, канала, диалога и пользователя;
- управление жизненным циклом токенов, включая попытки их обновления.

Например, боту, который может проверять последние сообщения электронной почты пользователя с помощью Microsoft Graph API, нужен токен пользователя Azure Active Directory. Во время проектирования разработчик бота регистрирует приложение Azure Active Directory с использованием службы токенов Bot Framework (через портал Azure), а затем настраивает параметр подключения OAuth (с именем `GraphConnection`) для бота. Когда пользователь взаимодействует с ботом, рабочий процесс будет следующим:

1. Пользователь просит бота проверить электронную почту.
1. Действие с этим сообщением отправляется от пользователя в службу канала Bot Framework. Служба канала гарантирует, что поле `userid` настроено для действия, а сообщение отправлено боту.

    Идентификаторы пользователя зависят от канала, включая идентификатор Facebook или номер телефона, с которого отправляются SMS.

1. Бот получает действие сообщения и определяет, что намерение заключается в проверке электронной почты пользователя. Бот отправляет запрос службе токенов Bot Framework, пытаясь узнать, если ли у нее токен для UserId для параметра подключения OAuth (`GraphConnection`).
1. Так как пользователь взаимодействует с ботом впервые, у службы токенов Bot Framework еще нет токена для этого пользователя, поэтому она возвращает боту ответ _NotFound_.
1. Бот создает OAuthCard с именем подключения `GraphConnection` и обращается к пользователю, чтобы тот выполнил вход с использованием этой карты.
1. Действие передается в службу канала Bot Framework, которая создает вызов в службу токенов Bot Framework для создания допустимого URL-адреса для входа (OAuth) для этого запроса. Этот URL-адрес для входа добавляется в OAuthCard, а сама карта затем возвращается пользователю.
1. Пользователь получает приглашение выполнить вход с помощью соответствующей кнопки OAuthCard.
1. Когда пользователь нажимает кнопку входа, служба канала открывает веб-браузер и вызывает внешнюю службу для загрузки страницы входа.
1. Пользователь выполняет вход на этой странице для внешней службы. После этого внешняя служба завершает обмен протоколами OAuth со службой токенов Bot Framework, а затем она отправляет токен пользователя в службу токенов Azure Bot Framework. Служба токенов Bot Framework безопасно сохраняет этот токен и отправляет действие боту с использованием этого токена.
1. Бот получает действие с токеном и может использовать его для создания вызовов к API Graph.

## <a name="securing-the-sign-in-url"></a>Защита URL-адреса для входа

Настраивая упрощенную процедуру входа для пользователя с помощью службы Bot Framework, важно обеспечить защиту URL-адреса для входа. Пользователю предоставляется URL-адрес для входа, который связан с определенными идентификаторами диалога и пользователя для этого бота. Этот URL-адрес не следует предоставлять в общий доступ, так как это вызовет проблемы со входом в определенный диалог с ботом. Чтобы предотвратить атаки на систему безопасности, нацеленные на общее использование URL-адреса для входа, убедитесь, что пользователь, который переходит по этому URL-адресу на соответствующем компьютере, — это _владелец_ окна диалога.

Некоторые каналы, например Cortana, Teams, Direct Line и WebChat, могут сделать это без участия пользователя. Например, WebChat использует файлы cookie сеанса, чтобы убедиться, что поток входа выполнен в том же браузере, в котором ведется диалог WebChat. Другие каналы часто определяют пользователя по шестизначному _коду_. Это похоже на встроенную многофакторную проверку подлинности, так как служба токенов Bot Framework не выдаст токен боту, пока пользователь не пройдет окончательную проверку подлинности, подтверждающую, что у выполнившего вход пользователя есть доступ к возможностям чата с использованием шестизначного кода.

## <a name="azure-activity-directory-application-registration"></a>Регистрация приложения Azure Active Directory

Каждый бот, зарегистрированный как служба Azure Bot, использует идентификатор приложения Azure Active Directory (AD). Очень важно **не** использовать повторно этот идентификатор приложения и пароль для выполнения входа пользователей. Идентификатор приложения Azure AD службы Azure Bot позволяет защитить обмен данными между ботом и службами канала Bot Framework. Чтобы пользователи могли войти в Azure AD, вам нужно создать отдельную регистрацию приложения Azure AD с соответствующими разрешениями и областями.

## <a name="configure-an-oauth-connection-setting"></a>Настройка параметра подключения OAuth

См. подробнее о том, как зарегистрировать и использовать параметр подключения OAuth в руководстве по [добавлению проверки подлинности для бота](bot-builder-authentication.md).
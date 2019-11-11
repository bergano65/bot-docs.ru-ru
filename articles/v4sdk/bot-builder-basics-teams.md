---
title: Принцип работы ботов для Microsoft Teams
description: Продолжение статьи о том, как работают боты на примере ботов для Microsoft Teams
author: WashingtonKayaker
ms.author: kamrani
manager: kamrani
ms.topic: overview
ms.service: bot-service
ms.date: 10/31/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3a7e60e1fa72402b5f783676f5281579a2be5371
ms.sourcegitcommit: 490810d278d1c8207330b132f28a5eaf2b37bd07
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/05/2019
ms.locfileid: "73592208"
---
# <a name="how-microsoft-teams-bots-work"></a>Принцип работы ботов Microsoft Teams

Эта вводная статья основана на сведениях из статьи [Принципы работы бота](https://docs.microsoft.com/azure/bot-service/bot-builder-basics), с которой необходимо ознакомиться до того, как приступать к этой.

Основные различия в работе ботов, которые разрабатываются для Microsoft Teams, заключаются в способе обработки действий. Обработчик действий Microsoft Teams является производным от обработчика действий Bot Framework, и он сначала обрабатывает все действия команд и лишь затем разрешает обработку любых действий, не относящихся к группам.

## <a name="teams-activity-handlers"></a>Обработчики событий команд

Разработанный для Microsoft Teams бот, как и любой другой, передает каждое полученное действие *обработчикам действий*. На системном уровне существует лишь один базовый обработчик, который называется *обработчиком шагов*, и именно он получает все действия. Этот *обработчик шагов* вызывает соответствующий обработчик для того типа действия, которое он получил. Но бот для Microsoft Teams отличается от других тем, что является производным от класса `Activity Handler` в _Teams_, который в свою очередь наследуется от класса `Activity Handler` в Bot Framework.  Класс `Activity Handler` в _Teams_ содержит несколько специальных обработчиков действий Microsoft Teams, которые будут рассмотрены в этой статье.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

Как и многие другие боты, которые используют Microsoft Bot Framework, в этом боте обработчик шагов оценивает каждое входящее действие сообщения, а затем отправляет его в обработчик действий `OnMessageActivityAsync`. Эта функциональная возможность остается неизменной, но если бот получает действие обновления диалога, обработчик шагов обратит внимание на это входящее действие и отправит его в обработчик действий `OnConversationUpdateActivityAsync` _Teams_, который сначала проверит наличие команд событий для Teams, а при их отсутствии передаст действие в обработчик действия Bot Framework.

В классе обработчика действий Teams существует два основных обработчика действий: `OnConversationUpdateActivityAsync` обрабатывает все действия по обновлению диалога, а `OnInvokeActivityAsync` обрабатывает все команды, которые вызывают действия Teams.  Все обработчики действий, которые описаны в разделе [Логика бота](https://aka.ms/how-bots-work#bot-logic) статьи `How bots work`, будут работать с ботами для Teams так же, как и с другими ботами, за одним исключением: при обработке действий добавления и удаления участников в контексте команд участники будут добавляться в команду, а не в поток сообщений.  Дополнительные сведения см. в таблице _Действия по обновлению беседы Teams_ в разделе [Логика бота](#bot-logic).

Чтобы реализовать логику для обработчиков действий Teams, вам нужно переопределить эти методы в боте, как описано ниже в разделе [Логика бота](#bot-logic). Базовой реализации для каждого из этих обработчиков нет, поэтому просто добавляйте логику, используемую при переопределении.

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

Как и многие другие боты, которые используют Microsoft Bot Framework, в этом боте обработчик шагов оценивает каждое входящее действие сообщения, а затем отправляет его в обработчик действий `onMessage`. Эта функциональная возможность остается неизменной, но если бот получает действие обновления диалога, обработчик шагов обратит внимание на это входящее действие и отправит его в обработчик действий `dispatchConversationUpdateActivity` _Teams_, который сначала проверит наличие команд событий для Teams, а при их отсутствии передаст действие в обработчик действия Bot Framework.

В классе обработчика действий Teams существует два основных обработчика действий: `dispatchConversationUpdateActivity` обрабатывает все действия по обновлению диалога, а `onInvokeActivity` обрабатывает все команды, которые вызывают действия Teams.  Все обработчики действий, которые описаны в разделе [Логика бота](https://aka.ms/how-bots-work#bot-logic) статьи `How bots work`, будут работать с ботами для Teams так же, как и с другими ботами, за одним исключением: при обработке действий добавления и удаления участников в контексте команд участники будут добавляться в команду, а не в поток сообщений.  Дополнительные сведения см. в таблице _Действия по обновлению беседы Teams_ в разделе [Логика бота](#bot-logic).

Как и с любым другим обработчиком ботов, чтобы реализовать логику для обработчиков Teams, вам нужно переопределить эти методы в боте, как описано в разделе [Логика ботов](#bot-logic) ниже. Для каждого из этих обработчиков определите логику бота, а затем **вызовите `next()` в конце шага**. Вызывая `next()`, вы обеспечиваете выполнение следующего обработчика.

---

### <a name="bot-logic"></a>Логика бота

Логика бота обрабатывает входящие действия из одного или нескольких каналов бота и создает исходящие действия в ответ.  Этот механизм сохраняется и для бота, производного от класса обработчика действий Teams, который первым делом проверяет действия для Teams, а затем передает все остальные действия обработчику действий Bot Framework.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

#### <a name="teams-conversation-update-activities"></a>Действие обновления диалога Teams

Ниже приведен список всех обработчиков действий Teams, которые вызываются из обработчика действий `OnConversationUpdateActivityAsync` _Teams_. В статье [о событиях обновления диалога](https://aka.ms/azure-bot-subscribe-to-conversation-events) описывается, как использовать каждое из этих событий в боте.

| Событие | Обработчик | ОПИСАНИЕ |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedAsync` | Переопределите, чтобы обрабатывать создание канала Teams. Дополнительные сведения см. в разделе [о создании канала](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-created). |
| channelDeleted | `OnTeamsChannelDeletedAsync` | Переопределите, чтобы обрабатывать удаление канала Teams. Дополнительные сведения см. в разделе [об удалении канала](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-deleted). |
| channelRenamed | `OnTeamsChannelRenamedAsync` | Переопределите, чтобы обрабатывать переименование канала Teams. Дополнительные сведения см. в разделе [о переименовании канала](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-renamed). |
| teamRenamed | `OnTeamsTeamRenamedAsync` | `return Task.CompletedTask;` Переопределите, чтобы обрабатывать переименование команды Teams. Дополнительные сведения см. в разделе [о переименовании команд](https://aka.ms/azure-bot-subscribe-to-conversation-events#team-renamed). |
| MembersAdded | `OnTeamsMembersAddedAsync` | Вызывает метод `OnMembersAddedAsync` из `ActivityHandler`. Переопределите, чтобы обрабатывать добавление участников в команду. Дополнительную информацию см. в разделе [о добавлении участника команды](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Added).|
| MembersRemoved | `OnTeamsMembersRemovedAsync` | Вызывает метод `OnMembersRemovedAsync` из `ActivityHandler`. Переопределите, чтобы обрабатывать удаление участников из команды. Дополнительные сведения см. в разделе [об удалении участника команды](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Removed).|


<!--
| Event | Handler | Description |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedAsync` | Override this to handle a Teams channel being created. |
| channelDeleted | `OnTeamsChannelDeletedAsync` | Override this to handle a Teams channel being deleted. |
| channelRenamed | `OnTeamsChannelRenamedAsync` | Override this to handle a Teams channel being renamed. |
| teamRenamed | `OnTeamsTeamRenamedAsync` | `return Task.CompletedTask;` Override this to handle a Teams Team being Renamed. |
| MembersAdded | `OnTeamsMembersAddedAsync` | Calls the `OnMembersAddedAsync` method in `ActivityHandler`. Override this to handle members joining a team. |
| MembersRemoved | `OnTeamsMembersRemovedAsync` | Calls the `OnMembersRemovedAsync` method in `ActivityHandler`. Override this to handle members leaving a team. |
-->

#### <a name="teams-invoke--activities"></a>Команды, вызывающие Teams

Здесь приведен список всех обработчиков действий Teams, которые вызываются из обработчика действий `OnInvokeActivityAsync` _Teams_.

| Вызываемые типы                    | Обработчик                              | ОПИСАНИЕ                                                  |
| :-----------------------------  | :----------------------------------- | :----------------------------------------------------------- |
| CardAction.Invoke               | `OnTeamsCardActionInvokeAsync`       | Teams Card Action Invoke. |
| fileConsent/invoke              | `OnTeamsFileConsentAcceptAsync`      | Teams File Consent Accept. |
| fileConsent/invoke              | `OnTeamsFileConsentAsync`            | Teams File Consent. |
| fileConsent/invoke              | `OnTeamsFileConsentDeclineAsync`     | Teams File Consent. |
| actionableMessage/executeAction | `OnTeamsO365ConnectorCardActionAsync` | Teams O365 Connector Card Action. |
| signin/verifyState              | `OnTeamsSigninVerifyStateAsync`      | Teams Sign in Verify State. |
| task/fetch                      | `OnTeamsTaskModuleFetchAsync`        | Teams Task Module Fetch. |
| task/submit                     | `OnTeamsTaskModuleSubmitAsync`       | Teams Task Module Submit. |

Приведенные выше действия вызова предназначены для ботов общения в Teams. Пакет SDK для Bot Framework также поддерживает вызовы, относящиеся к расширениям для обмена сообщениями. Дополнительные сведения об этих расширениях см. [в этой статье](https://aka.ms/azure-bot-what-are-messaging-extensions).

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

#### <a name="teams-conversation-update-activities"></a>Действие обновления диалога Teams

Ниже приведен список всех обработчиков действий Teams, которые вызываются из обработчика действий `dispatchConversationUpdateActivity` _Teams_. В статье [о событиях обновления диалога](https://aka.ms/azure-bot-subscribe-to-conversation-events) описывается, как использовать каждое из этих событий в боте.

| Событие | Обработчик | ОПИСАНИЕ |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedEvent` | Переопределите, чтобы обрабатывать создание канала Teams. Дополнительные сведения см. в разделе [о создании канала](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-created). |
| channelDeleted | `OnTeamsChannelDeletedEvent` | Переопределите, чтобы обрабатывать удаление канала Teams. Дополнительные сведения см. в разделе [об удалении канала](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-deleted).|
| channelRenamed | `OnTeamsChannelRenamedEvent` | Переопределите, чтобы обрабатывать переименование канала Teams. Дополнительные сведения см. в разделе [о переименовании канала](https://aka.ms/azure-bot-subscribe-to-conversation-events#channel-renamed). |
| teamRenamed | `OnTeamsTeamRenamedEvent` | `return Task.CompletedTask;` Переопределите, чтобы обрабатывать переименование команды Teams. Дополнительные сведения см. в разделе [о переименовании команд](https://aka.ms/azure-bot-subscribe-to-conversation-events#team-renamed). |
| MembersAdded | `OnTeamsMembersAddedEvent` | Вызывает метод `OnMembersAddedEvent` из `ActivityHandler`. Переопределите, чтобы обрабатывать добавление участников в команду. Дополнительную информацию см. в разделе [о добавлении участника команды](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Added). |
| MembersRemoved | `OnTeamsMembersRemovedEvent` | Вызывает метод `OnMembersRemovedEvent` из `ActivityHandler`. Переопределите, чтобы обрабатывать удаление участников из команды. Дополнительные сведения см. в разделе [об удалении участника команды](https://aka.ms/azure-bot-subscribe-to-conversation-events#Team-Member-Removed). |

<!--
| Event | Handler | Description |
| :-- | :-- | :-- |
| channelCreated | `OnTeamsChannelCreatedEvent` | Override this to handle a Teams channel being created. |
| channelDeleted | `OnTeamsChannelDeletedEvent` | Override this to handle a Teams channel being deleted. |
| channelRenamed | `OnTeamsChannelRenamedEvent` | Override this to handle a Teams channel being renamed. |
| teamRenamed | `OnTeamsTeamRenamedEvent` | `return Task.CompletedTask;` Override this to handle a Teams Team being Renamed. |
| MembersAdded | `OnTeamsMembersAddedEvent` | Calls the `OnMembersAddedEvent` method in `ActivityHandler`. Override this to handle members joining a team. |
| MembersRemoved | `OnTeamsMembersRemovedEvent` | Calls the `OnMembersRemovedEvent` method in `ActivityHandler`. Override this to handle members leaving a team. |
-->

#### <a name="teams-invoke--activities"></a>Команды, вызывающие Teams

Здесь приведен список всех обработчиков действий Teams, которые вызываются из обработчика действий `onInvokeActivity` _Teams_.

| Вызываемые типы                    | Обработчик                              | ОПИСАНИЕ                                                  |
| :-----------------------------  | :----------------------------------- | :----------------------------------------------------------- |
| CardAction.Invoke               | `handleTeamsCardActionInvoke`       | Teams Card Action Invoke. |
| fileConsent/invoke              | `handleTeamsFileConsentAccept`      | Teams File Consent Accept. |
| fileConsent/invoke              | `handleTeamsFileConsent`            | Teams File Consent. |
| fileConsent/invoke              | `handleTeamsFileConsentDecline`     | Teams File Consent. |
| actionableMessage/executeAction | `handleTeamsO365ConnectorCardAction` | Teams O365 Connector Card Action. |
| signin/verifyState              | `handleTeamsSigninVerifyState`      | Teams Sign in Verify State. |
| task/fetch                      | `handleTeamsTaskModuleFetch`        | Teams Task Module Fetch. |
| task/submit                     | `handleTeamsTaskModuleSubmit`       | Teams Task Module Submit. |

Приведенные выше действия вызова предназначены для ботов общения в Teams. Пакет SDK для Bot Framework также поддерживает вызовы, относящиеся к расширениям для обмена сообщениями. Дополнительные сведения об этих расширениях см. [в этой статье](https://aka.ms/azure-bot-what-are-messaging-extensions).

---

## <a name="next-steps"></a>Дополнительная информация
Сведения о создании ботов для Teams см. в [документации](https://aka.ms/teams-docs) для разработчика Microsoft Teams.
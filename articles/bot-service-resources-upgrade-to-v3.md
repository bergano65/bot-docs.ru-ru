---
title: Обновление бота до API Bot Framework версии 3 — Служба Azure Bot
description: Сведения об обновлении бота до API Bot Framework версии 3.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 6f08da5acbf8ac050d5dbf6a1a0290c3e6288013
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790996"
---
# <a name="upgrade-your-bot-to-bot-framework-api-v3"></a>Обновление бота до API Bot Framework версии 3.

На конференции Build 2016 корпорация Майкрософт объявила о выпуске Microsoft Bot Framework и первой итерации API соединителя ботов, наряду с пакетами SDK построителя ботов и соединителя ботов. С этого момента мы собираем ваши отзывы и предложения и активно работаем над улучшением REST API и пакетов SDK.

В июле 2016 г. был выпущен API Bot Framework версии 3, а API Bot Framework версии 1 признан нерекомендуемым. Боты, использующие API версии 1, перестали работать в Skype в декабре 2016 г. и во всех остальных каналах — 23 февраля 2017 г. Если вы создали бот, используя API версии 1, обновите его до API версии 3, следуя инструкциям в этой статье, чтобы он мог снова функционировать. Чтобы получить полное представление о процессе обновления, прочитайте эту статью перед началом обновления. 

## <a name="step-1-get-your-app-id-and-password-from-the-bot-framework-portal"></a>Шаг 1. Получение идентификатора приложения и пароля на портале Bot Framework

Войдите на [портал Bot Framework](https://dev.botframework.com/), нажмите кнопку **Мои боты**, затем выберите нужный бот, чтобы открыть его панель мониторинга. Затем перейдите по ссылке **Параметры** в левой части страницы в разделе **Управление ботами**. 

В разделе **Конфигурация** на странице параметров просмотрите содержимое поля **Идентификатор приложения Майкрософт**, а затем переходите к дальнейшим действиям.

<!-- TODO: Remove this 
### Case 1: App ID field is already populated

If the **App ID** field is already populated, complete these steps:
-->

1. Щелкните **Управление идентификатором и паролем приложения Майкрософт**.  
![Конфигурация](./media/upgrade/manage-app-id.png)

2. Щелкните **Создать новый пароль**.  
![Создание нового пароля](./media/upgrade/generate-new-password.png)

3. Скопируйте и сохраните новый пароль, а также идентификатор приложения MSA. Эти значения потребуются в будущем.  
![Новый пароль.](./media/upgrade/new-password-generated.png)

Еще один способ получить **идентификатор приложения Майкрософт и пароль** описывается в этих [инструкциях](https://blog.botframework.com/2018/07/03/find-your-azure-bots-appid-and-appsecret/).

<!-- TODO: These steps are no longer valid. AppID will always be generated, confirmed with Support Engineers
### Case 2: App ID field is empty

If the **App ID** field is empty, complete these steps:

1. Click **Create Microsoft App ID and password**.  
   ![Create App ID and password](~/media/upgrade/generate-appid-and-password.png)
   > [!IMPORTANT]
   > Do not select the **Version 3.0** radio button yet. You will do this later, after you have [updated your bot code](#update-code).</div>

2. Click **Generate a password to continue**.  
   ![Generate app password](~/media/upgrade/generate-a-password-to-continue.png)

3. Copy and save the new password along with the MSA App Id; you will need these values in the future.  
   ![New password](~/media/upgrade/new-password-generated.png)

4. Click **Finish and go back to Bot Framework**.  
   ![Finish and go back to Portal](~/media/upgrade/finish-and-go-back-to-bot-framework.png)

5. Back on the bot settings page in the Bot Framework Portal, scroll to the bottom of the page and click **Save changes**.  
   ![Save changes](~/media/upgrade/save-changes.png)
-->

## <a id="update-code"></a> Шаг 2. Обновление кода бота до версии 4.0

Боты версии 1 больше не поддерживаются. Чтобы обновить бота, необходимо создать нового бота версии 3. Если вы хотите сохранить какой либо старый код, его потребуется перенести вручную.

Самое простое решение — заново создать бота с помощью нового [пакета SDK](https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0) и развернуть его. 

Если вы хотите сохранить старый код, выполните указанные ниже действия.

1. Создайте приложение бота.
2. Скопируйте старый код в новое приложение.
3. Обновите пакет SDK до последней версии с помощью диспетчера пакетов Nuget.
4. Исправьте все обнаруженные ошибки, ссылаясь на новый [пакет SDK](https://docs.microsoft.com/azure/bot-service/?view=azure-bot-service-4.0).
5. Разверните бота в Azure, следуя этим [инструкциям](https://docs.microsoft.com/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0).

<!-- TODO: Remove outdated code 
To update your bot code to version 3.0, complete these steps:

1. Update to the latest version of the [Bot Framework SDK](https://github.com/Microsoft/BotBuilder) for your bot's language.
2. Update your code to apply the necessary changes, according the guidance below.
3. Use the [Bot Framework Emulator](~/bot-service-debug-emulator.md) to test your bot locally and then in the cloud.

The following sections describe the key differences between API v1 and API v3. After you have updated your code to API v3, you can finish the upgrade process by [updating your bot settings](#step-3) in the Bot Framework Portal.
-->

### <a name="botbuilder-and-connector-are-now-one-sdk"></a>Построитель ботов и соединитель теперь входят в один пакет SDK

Вместо того, чтобы устанавливать отдельные пакеты SDK для построителя и соединителя в виде нескольких пакетов NuGet (или модулей NPM), теперь можно получить обе библиотеки в одном пакете SDK Bot Framework:

- Пакет SDK Bot Framework для .NET: пакет NuGet `Microsoft.Bot.Builder`;
- Пакет SDK Bot Framework для Node.js: модуль NPM `botbuilder`.

Автономной пакет SDK `Microsoft.Bot.Connector` теперь является устаревшим и больше не поддерживается.

### <a name="message-is-now-activity"></a>Объект Message заменен объектом Activity

Объект `Message` заменен объектом `Activity` в API версии 3. Наиболее распространенным типом действия является **сообщение**, однако существуют другие типы действий, которые можно использовать для передачи данных различных типов в бот или канал. Дополнительные сведения о сообщениях см. в разделе [Создание сообщений](~/dotnet/bot-builder-dotnet-create-messages.md) и [Отправка и получение действий](~/dotnet/bot-builder-dotnet-connector.md).

### <a name="activity-types--events"></a>Типы действий и события

Некоторые события были переименованы в API версии 3, или к ним был применен рефакторинг. Кроме того, в соединитель добавлено новое перечисление `ActivityTypes`, что позволяет отказаться от необходимости запоминать конкретные типы действий.

- Тип действия `conversationUpdate` заменяет действия добавления и удаления пользователя или бота из беседы одним методом.
- Новый тип действия `typing` позволяет боту указывать, что он компилирует ответ, и узнавать о том, что пользователь вводит ответ.
- Новый тип действия `contactRelationUpdate` позволяет боту узнавать, что он добавлен в список контактов пользователя или удален из него.

При получении ботом действия `conversationUpdate` свойства `MembersRemoved` и `MembersAdded` будут указывать, кто был добавлен в беседу или удален из нее. При получении ботом действия `contactRelationUpdate` свойство `Action` укажет, добавил ли пользователь бота в список контактов или удалил из него. Дополнительные сведения о типах действий см. в разделе [Общие сведения о действиях](~/dotnet/bot-builder-dotnet-activities.md).

### <a name="addressing-messages"></a>Адресация сообщений

В API версии 3 немного изменилось место указания сведений об отправителе, получателе и канале в сообщении.

|Поле версии API 1 | Поле версии API 3|
|--------|--------|
| Объект `From`. | Объект `From`. |
| Объект `To`. | Объект `Recipient`. |
| Свойство `ChannelConversationID` | Объект `Conversation`.|
| Свойство `ChannelId` | Свойство `ChannelId` |

Дополнительные сведения об адресации сообщений см. в разделе [Отправка и получение действий](~/dotnet/bot-builder-dotnet-connector.md).

### <a name="sending-replies"></a>Отправка ответов

В API Bot Framework версии 3 все ответы для пользователя будут передаваться асинхронно через отдельно инициируемый HTTP-запрос, а не в запросе HTTP POST для входящего сообщения для бота. Поскольку сообщения не будут возвращаться как встроенные пользователю через соединитель, типом возвращаемого значения метода POST бота будет `HttpResponseMessage`. Это означает, что бот не "возвращает" синхронно строку, которую вы хотите отправить пользователю, а передает ответное сообщение в любой точке кода, вместо того чтобы отправлять сообщение как ответ на входящий запрос POST. Оба эти метода отправляют сообщение в беседу:

- `SendToConversation`
- `ReplyToConversation`

Метод `SendToConversation` добавит указанное сообщение в конец беседы, тогда как метод `ReplyToConversation` (для бесед, которые его поддерживают) добавит указанное сообщение как прямой ответ на предыдущее сообщение в беседе. Дополнительные сведения об этих методах см. в разделе [Отправка и получение действий](~/dotnet/bot-builder-dotnet-connector.md).

### <a name="starting-conversations"></a>Начало беседы

В API Bot Framework версии 3 можно начать беседу с помощью нового метода `CreateDirectConversation` (запуск частной беседы с одним пользователем) или с помощью нового метода `CreateConversation` (запуск групповой беседы с несколькими пользователями). Дополнительные сведения о начале беседы см. в разделе [Отправка и получение действий](~/dotnet/bot-builder-dotnet-connector.md#start-a-conversation).

### <a name="attachments-and-options"></a>Вложения и параметры

В API Bot Framework версии 3 представлена более надежная реализация вложений и карточек. Тип `Options` больше не поддерживается в API версии 3. Он заменен карточками. Дополнительные сведения о добавлении вложений в сообщения с помощью .NET см. в разделе [Добавление вложений мультимедиа в сообщения](~/dotnet/bot-builder-dotnet-add-media-attachments.md) и [Добавление форматированных карточек в сообщения](~/dotnet/bot-builder-dotnet-add-rich-card-attachments.md).

### <a name="bot-data-storage-bot-state"></a>Хранилище данных бота (состояния бота)

В API Bot Framework версии 1 API для управления данными состояния бота был свернут в API обмена сообщениями. В API Bot Framework версии 3 эти API-интерфейсы разделены. Теперь для получения данных о состоянии необходимо использовать службу состояния бота (не полагаясь на то, что они включены в объект `Message`) и сохранять данные о состоянии (вместо того чтобы передавать их как часть объекта `Message`). Сведения об управлении данными состояния бота с помощью службы состояния бота см. в разделе [Управление данными о состоянии](~/dotnet/bot-builder-dotnet-state.md).

> [!IMPORTANT]
> API службы состояния Bot Framework не рекомендуется использовать для рабочей среды. Он может считаться устаревшим в будущем выпуске. Рекомендуется обновить код бота для использования хранилища в памяти в целях тестирования или использовать одно из **расширений Azure** для ботов в рабочей среде. Дополнительные сведения см. в разделе **Управление данными состояния** для реализации [.NET](~/dotnet/bot-builder-dotnet-state.md) или [Node.js](~/nodejs/bot-builder-nodejs-state.md).

### <a name="webconfig-changes"></a>Изменения в Web.config

В API Bot Framework версии 1 свойства проверки подлинности сохранялись со следующими ключами в файле **Web.Config**:

- `AppID`
- `AppSecret`

В API Bot Framework версии 3 свойства проверки подлинности сохраняются со следующими ключами в файле **Web.Config**:

- `MicrosoftAppID`
- `MicrosoftAppPassword`

## <a id="step-3"></a> Шаг 3. Развертывание обновленного бота в Azure.

Обновив код бота до API версии 3, просто разверните бот в Azure, следуя этим [инструкциям](https://docs.microsoft.com/azure/bot-service/bot-builder-howto-deploy-azure?view=azure-bot-service-4.0). Так как версия 1 больше не поддерживается, все боты будут автоматически использовать API версии 3 при развертывании в службах Azure.

<!-- TODO: Documentation set for removal 
1. Sign in to the [Bot Framework Portal](https://dev.botframework.com/).

2. Click **My bots** and select your bot to open its dashboard. 

3. Click the **SETTINGS** link that is located near the top-right corner of the page. 

4. Under **Version 3.0** within the **Configuration** section, paste your bot's endpoint into the **Messaging endpoint** field.  
![Version 3 configuration](~/media/upgrade/paste-new-v3-enpoint-url.png)

5. Select the **Version 3.0** radio button.  
![Select version 3.0](~/media/upgrade/switch-to-v3-endpoint.png)

6. Scroll to the bottom of the page and click **Save changes**.  
![Save changes](~/media/upgrade/save-changes.png)
-->
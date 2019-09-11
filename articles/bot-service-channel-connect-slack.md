---
title: Подключение бота к Slack | Документация Майкрософт
description: Сведения о настройке подключения бота к Slack.
keywords: connect a bot, bot channel, Slack bot, Slack messaging app
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 01/09/2019
ms.openlocfilehash: 9648d1aa9e2da6243f42a5e044451ae4d8fb9d71
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298301"
---
# <a name="connect-a-bot-to-slack"></a>Подключение бота к Slack

Вы можете настроить взаимодействие бота с другими пользователями с помощью программы для обмена сообщениями Slack.

## <a name="create-a-slack-application-for-your-bot"></a>Создание приложения Slack для бота

Войдите в [Slack](https://slack.com/signin) и перейдите на канал для [создания приложения Slack](https://api.slack.com/apps).

![Настройка бота](~/media/channels/slack-NewApp.png)

## <a name="create-an-app-and-assign-a-development-slack-team"></a>Создание приложения и назначение группы разработки Slack

Введите имя приложения и выберите группу разработчиков Slack. Если вы еще не является членом команды разработчиков Slack, [создайте ее или присоединитесь к ней](https://slack.com/).

![Создание приложения](~/media/channels/slack-CreateApp.png)

Нажмите кнопку **Create App** (Создать приложение). Slack создаст приложение, а также идентификатор и секрет клиента.

## <a name="add-a-new-redirect-url"></a>Добавление нового URL-адреса перенаправления

Далее добавьте новый URL-адрес перенаправления.

1. Выберите вкладку **OAuth & Permissions** (OAuth и разрешения).
2. Щелкните **Add a new Redirect URL** (Добавить новый URL-адрес перенаправления).
3. Укажите https://slack.botframework.com.
4. Щелкните **Добавить**.
5. Нажмите кнопку **Save URLs** (Сохранить URL-адреса).

![Добавление URL-адреса перенаправления](~/media/channels/slack-RedirectURL.png)

## <a name="create-a-slack-bot-user"></a>Создание пользователя бота Slack

Добавив пользователя бота, можно назначить боту имя и выбрать, будет ли он всегда находиться в сети.

1. Выберите вкладку **Bot Users** (Пользователи бота).
2. Щелкните **Add a Bot User** (Добавить пользователя бота).

![Создание бота](~/media/channels/slack-CreateBot.png)

Щелкните **Add a Bot User** (Добавить пользователя бота) для проверки параметров, выберите для параметра **Always Show My Bot as Online** (Бот постоянно в сети) значение **Вкл.** и нажмите кнопку **Сохранить изменения**.

![Создание бота](~/media/channels/slack-CreateApp-AddBotUser.png)

## <a name="subscribe-to-bot-events"></a>Подписка на события бота

Выполните следующие действия для подписки на шесть определенных событий бота. При этом приложение будет получать уведомления о действиях пользователя по указанному вами URL-адресу.

> [!TIP]
> Дескриптор бота — это его имя. Для получения дескриптора бота перейдите по адресу [https://dev.botframework.com/bots](https://dev.botframework.com/bots), выберите бот и запишите имя этого бота.

1. Выберите вкладку **Подписки на события**.
2. Выберите для параметра **Enable Events** (Включить события) значение **Вкл**.
3. В поле **URL-адрес запроса** введите значение `https://slack.botframework.com/api/Events/{YourBotHandle}`, где `{YourBotHandle}` обозначает дескриптор бота без фигурных скобок. Для этого примера используется дескриптор **ContosoBot**.

   ![Подписка на события (верх)](~/media/channels/slack-SubscribeEvents-a.png)

4. В разделе **Subscribe to Bot Events** (Подписаться на события ботов) щелкните **Add Bot User Event** (Добавить пользовательское событие бота).
5. В списке событий выберите следующие шесть типов событий:
    * `member_joined_channel`
    * `member_left_channel`
    * `message.channels`
    * `message.groups`
    * `message.im`
    * `message.mpim`

   ![Подписка на события (середина)](~/media/channels/slack-SubscribeEvents-b.png)

6. Нажмите кнопку **Сохранить изменения**.

   ![Подписка на события (низ)](~/media/channels/slack-SubscribeEvents-c.png)

## <a name="add-and-configure-interactive-messages-optional"></a>Добавление и настройка интерактивных сообщений (необязательно)

Если ваш бот будет использовать функции Slack, например кнопки, выполните следующие действия.

1. Выберите вкладку **Interactive Components** (Интерактивные компоненты) и щелкните **Enable Interactive Components** (Включить интерактивные компоненты).
2. Введите `https://slack.botframework.com/api/Actions` в качестве **URL-адреса запроса**.
3. Нажмите кнопку **Save changes** (Сохранить изменения).

![Включение сообщений](~/media/channels/slack-MessageURL.png)

## <a name="gather-credentials"></a>Получение учетных данных

Выберите вкладку **Основные сведения** и перейдите к разделу **Учетные данные приложения**.
Вы увидите идентификатор клиента, секрет клиента и токен проверки, требуемые для настройки бота Slack.

![Получение учетных данных](~/media/channels/slack-AppCredentials.png)

## <a name="submit-credentials"></a>Отправка учетных данных

В отдельном окне браузера вернитесь на сайт Bot Framework `https://dev.botframework.com/`.

1. Выберите **My bots** (Мои боты) и выберите бот, который нужно подключить к Slack.
2. В разделе **Channels** (Каналы) щелкните значок Slack.
3. В разделе **Enter your Slack credentials** (Ввод учетных данных Slack) вставьте учетные данные приложения с веб-сайта Slack в соответствующие поля.
4. **URL-адрес целевой страницы** является необязательным. Его можно опустить или изменить.
5. Выберите команду **Сохранить**.

![Отправка учетных данных](~/media/channels/slack-SubmitCredentials.png)

Следуйте инструкциям, чтобы авторизовать доступ приложения Slack к команде разработчиков Slack.

## <a name="enable-the-bot"></a>Включение бота

На странице Configure Slack (Настройка Slack) убедитесь, что ползунок возле кнопки "Сохранить" установлен в положение **Включено**.
Бот настроен для взаимодействия с пользователями в Slack.

## <a name="create-an-add-to-slack-button"></a>Создание кнопки Add to Slack (Добавить в Slack)

Slack предоставляет HTML-код, с помощью которого можно упростить поиск вашего бота в разделе *Add the Slack button* (Создание кнопки "Добавить в Slack") на [этой странице](https://api.slack.com/docs/slack-button).
Чтобы использовать этот HTML-код с ботом, замените значение href (начинается с `https://`) URL-адресом, найденным в параметрах канала Slack вашего бота.
Выполните следующие действия, чтобы получить URL-адрес на замену.

1. На странице [https://dev.botframework.com/bots](https://dev.botframework.com/bots) щелкните свой бот.
2. Выберите **Channels** (Каналы), щелкните правой кнопкой мыши запись с именем **Slack** и выберите пункт **Copy link** (Копировать ссылку). Этот URL-адрес теперь находится в буфере обмена.
3. Вставьте его из буфера обмена в HTML-код, предоставленный для кнопки Slack. Этот URL-адрес заменяет значение href, предоставленное Slack для этого бота.

Авторизованные пользователи могут нажать кнопку **Add to Slack** (Добавить в Slack), предоставленную этим измененным HTML, для доступа к боту в Slack.

## <a name="also-available-as-an-adapter"></a>Также предоставляется как адаптер

Этот канал также [предоставляется как адаптер](https://botkit.ai/docs/v4/platforms/slack.html). См. подробнее о выборе между адаптером и каналом в списке [доступных адаптеров](bot-service-channel-additional-channels.md#currently-available-adapters).

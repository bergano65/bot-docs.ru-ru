---
title: Подключение бота к Facebook Messenger | Документация Майкрософт
description: Узнайте, как настроить подключение бота к Facebook Messenger.
keywords: Facebook Messenger, bot channel, Facebook App, App ID, App Secret, Facebook bot, credentials
manager: kamrani
ms.topic: article
ms.author: kamrani
ms.service: bot-service
ms.date: 08/03/2019
ms.openlocfilehash: 4e5dc332b463e9490c7aa265a08e8f126d59d3f9
ms.sourcegitcommit: 6a83b2c8ab2902121e8ee9531a7aa2d85b827396
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/09/2019
ms.locfileid: "68866546"
---
# <a name="connect-a-bot-to-facebook"></a>Подключение бота к Facebook

Вы можете подключить бот к Facebook Messenger и Facebook Workplace, чтобы он мог взаимодействовать с пользователями на обеих платформах. В этом руководстве описано, как подключить бота к этим двум каналам.

> [!NOTE]
> Пользовательский интерфейс Facebook может выглядеть несколько иначе, в зависимости от используемой версии.

## <a name="connect-a-bot-to-facebook-messenger"></a>Подключение бота к Facebook Messenger

Дополнительные сведения о разработке приложений для Facebook Messenger см. в [документации по платформе Messenger](https://developers.facebook.com/docs/messenger-platform). Вы можете просмотреть [инструкции перед запуском](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public), [краткое руководство](https://developers.facebook.com/docs/messenger-platform/guides/quick-start) и [руководство по настройке](https://developers.facebook.com/docs/messenger-platform/guides/setup) Facebook.

Чтобы настроить бот для обмена данными с Facebook Messenger, включите Facebook Messenger на странице Facebook и подключите бота.

### <a name="copy-the-page-id"></a>Копирование идентификатора страницы

Доступ к боту осуществляется на странице Facebook.

1. [Создайте страницу Facebook](https://www.facebook.com/bookmarks/pages) или перейдите на имеющуюся.

1. Перейдите к странице **About** (О программе) на странице Facebook, а затем скопируйте и сохраните **идентификатор страницы**.

### <a name="create-a-facebook-app"></a>Создание приложения Facebook

1. В браузере перейдите на страницу [создания приложения Facebook](https://developers.facebook.com/quickstarts/?platform=web).
1. Укажите имя приложения и щелкните **Create New Facebook App ID** (Создать идентификатор нового приложения Facebook).

    ![Создание приложения](media/channels/fb-create-messenger-bot-app.png)

1. В появившемся диалоговом окне введите адрес электронной почты и щелкните **Create App ID** (Создать идентификатор приложения).

    ![Создание идентификатора приложения](media/channels/fb-create-messenger-bot-app-id.png)

1. Выполните указания мастера.

1. Введите необходимые сведения для проверки и щелкните **Skip Quick Start** (Пропустить быстрый запуск) вверху справа.

1. В следующем окне слева откройте раздел *Settings* (Параметры) и щелкните **Basic** (Общие).

1. В области справа скопируйте и сохраните **идентификатор приложения** и **секрет приложения**.

    ![Копирование идентификатора приложения и секрета приложения](media/channels/fb-messenger-bot-get-appid-secret.png)

1. В области слева в разделе *Settings* (Параметры) щелкните **Advanced** (Дополнительно).

1. В области справа установите ползунок **Allow API Access to App Settings** (Разрешить доступ API к параметрам приложения) в положение **Yes** (Да).

    ![Копирование идентификатора приложения и секрета приложения](media/channels/fb-messenger-bot-api-settings.png)

1. Внизу справа щелкните **Save Changes** (Сохранить изменения).

### <a name="enable-messenger"></a>Включение Messenger

1. В области слева щелкните **Dashboard** (Панель мониторинга).
1. В области справа прокрутите вниз и в поле **Messenger** щелкните **Set Up** (Настроить). Запись Messenger отобразится в разделе *PRODUCTS* (Продукты) в области слева.  

    ![Включение Messenger](media/channels/fb-messenger-bot-enable-messenger.png)

### <a name="generate-a-page-access-token"></a>Создание маркера доступа к странице

1. В области слева под записью Messenger щелкните **Settings** (Параметры).
1. В области справа прокрутите вниз и в разделе **Token Generation** (Создание токена) выберите целевую страницу.

    ![Включение Messenger](media/channels/fb-messenger-bot-select-messenger-page.png)

1. Щелкните **Edit Permissions** (Изменить разрешения), чтобы использовать pages_messaging приложения для создания маркера доступа.
1. Следуйте указаниям мастера. Наконец, примите параметры по умолчанию и щелкните **Done** (Готово). Будет создан **маркер доступа к странице**.

    ![Разрешения Messenger](media/channels/fb-messenger-bot-permissions.png)

1. Скопируйте и сохраните **маркер доступа к странице**.

### <a name="enable-webhooks"></a>Включение веб-перехватчиков

Чтобы отправлять сообщения и другие события из бота в Facebook Messenger, необходимо включить интеграцию веб-перехватчиков. На этом этапе можно оставить параметры Facebook как есть. Мы еще вернемся к ним.

1. В браузере откройте новое окно и перейдите на [портал Azure.](https://portal.azure.com/) 

1. В списке ресурсов щелкните регистрацию ресурса бота и в соответствующей колонке щелкните **Каналы**.

1. Справа щелкните значок **Facebook**.

1. В мастере введите данные Facebook, сохраненные на предыдущих шагах. Если информация правильная, в мастере внизу вы увидите **URL-адрес обратного вызова** и **токен проверки**. Скопируйте и сохраните эти значения.  

    ![Конфигурация канала Facebook Messenger](media/channels/fb-messenger-bot-config-channel.PNG)

1. Нажмите кнопку **Сохранить** .


1. Вернемся к параметрам Facebook. Справа прокрутите вниз и в разделе **Webhooks** (Веб-перехватчики) щелкните **Subscribe To Events** (Подписаться на события). Так вы настроите переадресацию событий обмена сообщениями из Facebook Messenger в бота.

    ![Включение веб-перехватчиков](media/channels/fb-messenger-bot-webhooks.PNG)

1. В появившемся диалоговом окне введите сохраненные ранее значения **URL-адреса обратного вызова** и **токена проверки**. В разделе **Subscription Fields** (Поля подписки) выберите *message\_deliveries*, *messages*, *messaging\_options* и *messaging\_postbacks*.

    ![Настройка веб-перехватчика](media/channels/fb-messenger-bot-config-webhooks.png)

1. Щелкните **Verify and Save** (Проверить и сохранить).
1. Подпишите веб-перехватчик на нужную страницу Facebook. Щелкните **Subscribe** (Подписать).

    ![Страница настройки веб-перехватчика](media/channels/fb-messenger-bot-config-webhooks-page.PNG)

### <a name="submit-for-review"></a>Отправка на проверку

На странице базовых параметров приложения Facebook требуется указать URL-адрес политики конфиденциальности и условий использования. На странице [Code of Conduct](https://investor.fb.com/corporate-governance/code-of-conduct/default.aspx) (Правила поведения) содержатся ссылки на сторонние ресурсы для создания политики конфиденциальности. На странице [Terms of Use](https://www.facebook.com/terms.php) (Условия использования) содержится пример условий, помогающий создать соответствующий документ с условиями использования.

После завершения работы с ботом в Facebook будет реализован собственный механизм [рассмотрения](https://developers.facebook.com/docs/messenger-platform/app-review) для приложений, которые публикуются в Messenger. Бот будет проверен на соответствие [политикам платформы](https://developers.facebook.com/docs/messenger-platform/policy-overview) Facebook.

### <a name="make-the-app-public-and-publish-the-page"></a>Предоставления общего доступа к приложению и публикация страницы

> [!NOTE]
> До публикации приложение находится в [режиме разработки](https://developers.facebook.com/docs/apps/managing-development-cycle). Функции подключаемого модуля и API будут работать только для администраторов, разработчиков и тест-инженеров.

После успешной проверки на панели мониторинга приложения в разделе рассмотрения приложения выберите значение Public (общедоступное).
Убедитесь, что страница Facebook, связанная с этим ботом, опубликована. Состояние появляется в параметрах страницы.

## <a name="connect-a-bot-to-facebook-workplace"></a>Подключение бота к Facebook Workplace

Откройте [центр справки Workplace](https://workplace.facebook.com/help/work/), чтобы изучить сведения о Facebook Workplace и [документацию для разработчиков Workplace](https://developers.facebook.com/docs/workplace), с подробными рекомендациями.

Чтобы настроить в боте связь через Facebook Workplace, создайте пользовательскую интеграцию и подключите к ней бота.


1. Создайте учетную запись Facebook Workplace Premium Выполните представленные [здесь](https://www.facebook.com/workplace) инструкции для создания учетной записи Facebook Workplace Premium и присвоения себе прав системного администратора. Как вы помните, пользовательские интеграции может создавать только системный администратор Workplace.

1. Создайте [пользовательскую интеграцию](https://developers.facebook.com/docs/workplace/custom-integrations-new) для Workplace, как описано ниже. При создании пользовательской интеграции автоматически создаются приложение с нужными разрешениями и страница типа "Бот", доступная только в пределах вашего сообщества Workplace.

1. На **панели администратора** откройте вкладку **Интеграции**.
1. Нажмите кнопку **Create your own custom App** (Создать пользовательское приложение).

    ![Интеграция Workplace](media/channels/fb-integration.png)

1. Выберите для приложения отображаемое имя и изображение профиля. Эта информация будет использоваться совместно со страницей типа "Бот".
1. Установите для параметра **Allow API Access to App Settings** (Разрешить доступ API к параметры приложения) значение Yes (Да).
1. Скопируйте и безопасно сохраните код приложения, секрет приложения и маркер приложения, которые вы здесь видите.

    ![Ключи Workplace](media/channels/fb-keys.png)

1. Итак, вы завершили создание пользовательской интеграции. Страницу типа "Бот" можно найти своем сообществе Workplace, как показано ниже.

    ![Страница Workplace](media/channels/fb-page.png)

### <a name="provide-facebook-credentials"></a>Указание учетных данных Facebook

На портале Azure вставьте значения **идентификатора приложения Facebook**, **секрета приложения Facebook** и **маркера доступа к странице**, скопированные ранее из Facebook Messenger. Вместо традиционного идентификатора страницы используйте числа, указанные после имени интегрируемого приложения на соответствующей **странице** со сведениями. Как и при подключении бота к Facebook Messenger, веб-перехватчики можно подключить с учетными данными, которые отображаются в Azure.

### <a name="submit-for-review"></a>Отправка на проверку
Подробные сведения см. в разделе **о подключении бота к Facebook Messenger** и в [документации для разработчика Workplace](https://developers.facebook.com/docs/workplace).

### <a name="make-the-app-public-and-publish-the-page"></a>Предоставления общего доступа к приложению и публикация страницы
Подробные сведения см. в разделе **о подключении бота к Facebook Messenger**.

## <a name="setting-the-api-version"></a>Указание версии API

Если вы получите уведомление от Facebook о том, что определенная версия API Graph теперь считается нерекомендуемой, перейдите на [страницу для разработчиков Facebook](https://developers.facebook.com). Перейдите к **параметрам приложения** своего бота и выберите **Параметры > Дополнительные параметры > Обновить версию API**, а затем укажите для параметра **Обновить все вызовы** версию 3.0.

![Обновление версии API](media/channels/fb-version-upgrade.png)

## <a name="see-also"></a>См. также

- **Пример кода**. См. пример бота <a href="https://aka.ms/facebook-events" target="_blank">Facebook-events</a>, чтобы узнать об обмене данными между ботом и Facebook Messenger.

- **Предоставляется как адаптер.** Этот канал также [предоставляется как адаптер](https://botkit.ai/docs/v4/platforms/facebook.html). См. подробнее о выборе между адаптером и каналом в списке [доступных адаптеров](bot-service-channel-additional-channels.md#currently-available-adapters).

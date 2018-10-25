---
title: Подключение бота к Facebook Messenger | Документация Майкрософт
description: Узнайте, как настроить подключение бота к Facebook Messenger.
keywords: Facebook Messenger, bot channel, Facebook App, App ID, App Secret, Facebook bot, credentials
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 10/12/2018
ms.openlocfilehash: 1424a1929afa7ab9bec48e038fc93a2f8262241a
ms.sourcegitcommit: 54ed5000c67a5b59e23b667547565dd96c7302f9
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/13/2018
ms.locfileid: "49315200"
---
# <a name="connect-a-bot-to-facebook"></a>Подключение бота к Facebook

Вы можете подключить бот к Facebook Messenger и Facebook Workplace, чтобы он мог взаимодействовать с пользователями на обеих платформах. В этом руководстве представлена пошаговая процедура подключения бота к этим двум каналам.

> [!NOTE]
> Пользовательский интерфейс Facebook может выглядеть несколько иначе, в зависимости от используемой версии.

## <a name="connect-a-bot-to-facebook-messenger"></a>Подключение бота к Facebook Messenger

Дополнительные сведения о разработке приложений для Facebook Messenger см. в [документации по платформе Messenger](https://developers.facebook.com/docs/messenger-platform). Вы можете просмотреть [инструкции перед запуском](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public), [краткое руководство](https://developers.facebook.com/docs/messenger-platform/guides/quick-start) и [руководство по настройке](https://developers.facebook.com/docs/messenger-platform/guides/setup) Facebook.

Чтобы настроить бот для связи с использованием Facebook Messenger, включите Facebook Messenger на странице Facebook и подключите бот к приложению.

### <a name="copy-the-page-id"></a>Копирование идентификатора страницы

Доступ к боту осуществляется на странице Facebook. [Создайте страницу Facebook](https://www.facebook.com/bookmarks/pages) или перейдите на имеющуюся.

* Перейдите к странице **About** (О программе) на странице Facebook, а затем скопируйте и сохраните **идентификатор страницы**.

### <a name="create-a-facebook-app"></a>Создание приложения Facebook

[Создайте приложение Facebook](https://developers.facebook.com/quickstarts/?platform=web) на странице и создайте для него идентификатор и секрет.

![Создание идентификатора приложения](~/media/channels/FB-CreateAppId.png)

* Скопируйте и сохраните **идентификатор приложения** и **секрет приложения**.

![Сохранение идентификатора и секрета приложения](~/media/channels/FB-get-appid.png)

Установите ползунок Allow API Access to App Settings (Разрешить доступ API к параметры приложения) в положение Yes (Да).

![Параметры приложения](~/media/bot-service-channel-connect-facebook/api_settings.png)

### <a name="enable-messenger"></a>Включение Messenger

Включите Facebook Messenger в новом приложении Facebook.

![Включение Messenger](~/media/channels/FB-AddMessaging1.png)

### <a name="generate-a-page-access-token"></a>Создание маркера доступа к странице

На панели **Token Generation** (Создание токена) раздела Messenger выберите целевую страницу. Будет создан маркер доступа к странице.

* Скопируйте и сохраните **маркер доступа к странице**.

![Создание токена](~/media/channels/FB-generateToken.png)

### <a name="enable-webhooks"></a>Включение веб-перехватчиков

Нажмите кнопку **Set up Webhooks** (Настроить веб-перехватчики) для переадресации событий обмена сообщениями из Facebook Messenger в бот.

![Включение веб-перехватчика](~/media/channels/FB-webhook.png)

### <a name="provide-webhook-callback-url-and-verify-token"></a>Указание URL-адреса обратного вызова веб-перехватчика и проверка токена

На [портале Azure](https://portal.azure.com/) откройте бот, щелкните вкладку **Каналы**, а затем выберите **Facebook Messenger**.

* Скопируйте значения **URL-адреса обратного вызова** и **проверки токена** с портала.

![Копирование значений](~/media/channels/fb-callbackVerify.png)

1. Вернитесь к Facebook Messenger и вставьте значения **URL-адреса обратного вызова** и **проверки токена**.

2. В разделе **Subscription Fields** (Поля подписки) выберите *message\_deliveries*, *messages*, *messaging\_options* и *messaging\_postbacks*.

3. Нажмите кнопку **Verify and Save** (Проверить и сохранить).

![Настройка веб-перехватчика](~/media/channels/FB-webhookConfig.png)

4. Подпишите веб-перехватчик на страницу Facebook.

![Подписка веб-перехватчика](~/media/bot-service-channel-connect-facebook/subscribe-webhook.png)


### <a name="provide-facebook-credentials"></a>Указание учетных данных Facebook

На портале Azure вставьте значения **идентификатора приложения Facebook**, **секрета приложения Facebook**, **идентификатора страницы** и **маркера доступа к ней**, ранее скопированные из Facebook Messenger. Можно использовать один и тот же бот на нескольких страницах Facebook, добавив идентификаторы и маркеры доступа этих страниц.

![Ввод учетных данных](~/media/channels/fb-credentials2.png)

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

### <a name="create-a-facebook-workplace-premium-account"></a>Создайте учетную запись Facebook Workplace Premium

Выполните представленные [здесь](https://www.facebook.com/workplace) инструкции для создания учетной записи Facebook Workplace Premium и присвоения себе прав системного администратора. Как вы помните, пользовательские интеграции может создавать только системный администратор Workplace.

### <a name="create-a-custom-integration"></a>Создание пользовательской интеграции

При создании пользовательской интеграции автоматически создаются приложение с нужными разрешениями и страница типа "Бот", доступная только в пределах вашего сообщества Workplace.

Создайте [пользовательскую интеграцию](https://developers.facebook.com/docs/workplace/custom-integrations-new) для вашего Workplace, выполнив следующее.

- На **панели администратора** откройте вкладку **Интеграции**.
- Нажмите кнопку **Create your own custom App** (Создать пользовательское приложение).

![Интеграция Workplace](~/media/channels/fb-integration.png)

- Выберите для приложения отображаемое имя и изображение профиля. Эта информация будет использоваться совместно со страницей типа "Бот".
- Установите для параметра **Allow API Access to App Settings** (Разрешить доступ API к параметры приложения) значение Yes (Да).
- Скопируйте и безопасно сохраните код приложения, секрет приложения и маркер приложения, которые вы здесь видите.

![Ключи Workplace](~/media/channels/fb-keys.png)

Итак, вы завершили создание пользовательской интеграции. теперь найдите страницу типа "Бот" в своем сообществе Workplace, как показано ниже.

![Страница Workplace](~/media/channels/fb-page.png)

### <a name="provide-facebook-credentials"></a>Указание учетных данных Facebook

На портале Azure вставьте значения **идентификатора приложения Facebook**, **секрета приложения Facebook** и **маркера доступа к странице**, скопированные ранее из Facebook Messenger. Вместо традиционного идентификатора страницы используйте числа, указанные после имени интегрируемого приложения на соответствующей **странице** со сведениями. Как и при подключении бота к Facebook Messenger, веб-перехватчики можно подключить с учетными данными, которые отображаются в Azure.

### <a name="submit-for-review"></a>Отправка на проверку
Подробные сведения см. в разделе **о подключении бота к Facebook Messenger** и в [документации для разработчика Workplace](https://developers.facebook.com/docs/workplace).

### <a name="make-the-app-public-and-publish-the-page"></a>Предоставления общего доступа к приложению и публикация страницы
Подробные сведения см. в разделе **о подключении бота к Facebook Messenger**.

## <a name="sample-code"></a>Пример кода

Дополнительные сведения можно получить, изучив в примере бота для <a href="https://aka.ms/facebook-events" target="_blank">обработки событий Facebook</a> код взаимодействия с Facebook Messenger.

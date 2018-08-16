---
title: Подключение бота к Facebook Messenger | Документация Майкрософт
description: Узнайте, как настроить подключение бота к Facebook Messenger.
keywords: Facebook Messenger, bot channel, Facebook App, App ID, App Secret, Facebook bot, credentials
author: RobStand
ms.author: RobStand
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 0a9ad7d51234b417d5d0f27dbcffe4ce839ba94a
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301204"
---
# <a name="connect-a-bot-to-facebook-messenger"></a>Подключение бота к Facebook Messenger

Дополнительные сведения о разработке приложений для Facebook Messenger см. в [документации по платформе Messenger](https://developers.facebook.com/docs/messenger-platform). Вы можете просмотреть [инструкции перед запуском](https://developers.facebook.com/docs/messenger-platform/product-overview/launch#app_public), [краткое руководство](https://developers.facebook.com/docs/messenger-platform/guides/quick-start) и [руководство по настройке](https://developers.facebook.com/docs/messenger-platform/guides/setup) Facebook.

Чтобы настроить бот для связи с использованием Facebook Messenger, включите Facebook Messenger на странице Facebook и подключите бот к приложению.

[!INCLUDE [Channel Inspector intro](~/includes/snippet-channel-inspector.md)]

> [!NOTE]
> Пользовательский интерфейс Facebook может выглядеть несколько иначе, в зависимости от используемой версии.

## <a name="copy-the-page-id"></a>Копирование идентификатора страницы

Доступ к боту осуществляется на странице Facebook. [Создайте страницу Facebook](https://www.facebook.com/bookmarks/pages) или перейдите на имеющуюся.

* Перейдите к странице **About** (О программе) на странице Facebook, а затем скопируйте и сохраните **идентификатор страницы**.

## <a name="create-a-facebook-app"></a>Создание приложения Facebook

[Создайте приложение Facebook](https://developers.facebook.com/quickstarts/?platform=web) на странице и создайте для него идентификатор и секрет.

![Создание идентификатора приложения](~/media/channels/FB-CreateAppId.png)

* Скопируйте и сохраните **идентификатор приложения** и **секрет приложения**.

![Сохранение идентификатора и секрета приложения](~/media/channels/FB-get-appid.png)

Установите ползунок Allow API Access to App Settings (Разрешить доступ API к параметры приложения) в положение Yes (Да).

![Параметры приложения](~/media/bot-service-channel-connect-facebook/api_settings.png)

## <a name="enable-messenger"></a>Включение Messenger


Включите Facebook Messenger в новом приложении Facebook.

* На странице приложения **Product Setup** (Настройка продукта) щелкните **Get Started** (Приступить к работе) и щелкните **Get Started** (Приступить к работе) еще раз.


![Включение Messenger](~/media/channels/FB-AddMessaging1.png)

## <a name="generate-a-page-access-token"></a>Создание маркера доступа к странице

На панели **Token Generation** (Создание токена) раздела Messenger выберите целевую страницу. Будет создан маркер доступа к странице.

* Скопируйте и сохраните **маркер доступа к странице**.

![Создание токена](~/media/channels/FB-generateToken.png)

## <a name="enable-webhooks"></a>Включение веб-перехватчиков

Нажмите кнопку **Set up Webhooks** (Настроить веб-перехватчики) для переадресации событий обмена сообщениями из Facebook Messenger в бот.

![Включение веб-перехватчика](~/media/channels/FB-webhook.png)

## <a name="provide-webhook-callback-url-and-verify-token"></a>Указание URL-адреса обратного вызова веб-перехватчика и проверка токена

Вернитесь на [портал Bot Framework](https://dev.botframework.com/). Откройте бот, щелкните **Каналы**, а затем щелкните **Facebook Messenger**.

* Скопируйте значения **URL-адреса обратного вызова** и **проверки токена** с портала.

![Копирование значений](~/media/channels/fb-callbackVerify.png)

1. Вернитесь к Facebook Messenger и вставьте значения **URL-адреса обратного вызова** и **проверки токена**.

2. В разделе **Subscription Fields** (Поля подписки) выберите *message\_deliveries*, *messages*, *messaging\_options* и *messaging\_postbacks*.

3. Нажмите кнопку **Verify and Save** (Проверить и сохранить).

![Настройка веб-перехватчика](~/media/channels/FB-webhookConfig.png)

4. Подпишите веб-перехватчик на страницу Facebook.

![Подписка веб-перехватчика](~/media/bot-service-channel-connect-facebook/subscribe-webhook.png)


## <a name="provide-facebook-credentials"></a>Указание учетных данных Facebook

На портале Bot Framework вставьте значения **идентификатора страницы**, **идентификатора приложения**, **секрета приложения** и **маркера доступа к странице**, ранее скопированные из Facebook Messenger.

![Ввод учетных данных](~/media/channels/fb-credentials2.png)

## <a name="submit-for-review"></a>Отправка на проверку

На странице базовых параметров приложения Facebook требуется указать URL-адрес политики конфиденциальности и условий использования. На странице [Code of Conduct](https://aka.ms/bf-conduct) (Правила поведения) содержатся ссылки на сторонние ресурсы для создания политики конфиденциальности. На странице [Terms of Use](https://aka.ms/bf-terms) (Условия использования) содержится пример условий, помогающий создать соответствующий документ с условиями использования.

После завершения работы с ботом в Facebook будет реализован собственный механизм [рассмотрения](https://developers.facebook.com/docs/messenger-platform/app-review) для приложений, которые публикуются в Messenger. Бот будет проверен на соответствие [политикам платформы](https://developers.facebook.com/docs/messenger-platform/policy-overview) Facebook.

## <a name="make-the-app-public-and-publish-the-page"></a>Предоставления общего доступа к приложению и публикация страницы

> [!NOTE]
> До публикации приложение находится в [режиме разработки](https://developers.facebook.com/docs/apps/managing-development-cycle). Функции подключаемого модуля и API будут работать только для администраторов, разработчиков и тест-инженеров.

После успешной проверки на панели мониторинга приложения в разделе рассмотрения приложения выберите значение Public (общедоступное).
Убедитесь, что страница Facebook, связанная с этим ботом, опубликована. Состояние появляется в параметрах страницы.

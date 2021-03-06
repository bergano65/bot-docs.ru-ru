---
title: Запрос платежа (JS версии 3) — Служба Azure Bot
description: Сведения об отправке запроса платежей с помощью пакета SDK Bot Framework для Node.js.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b4b8d6763c79cea6fc06666145dc851283bd8a63
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75790635"
---
# <a name="request-payment"></a>Запрос платежей

> [!NOTE]
> Служба платежей, упомянутая в этой статье, станет нерекомендуемой 1-го декабря 2019 г.


> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-request-payment.md)
> - [Node.js](../nodejs/bot-builder-nodejs-request-payment.md)

Если бот позволяет пользователям приобретать элементы, он делает запрос платежей, включая специальный тип кнопки в [форматированной карточке](bot-builder-nodejs-send-rich-cards.md). Эта статья описывает отправку запроса платежей с помощью пакета SDK Bot Framework для Node.js.

## <a name="prerequisites"></a>предварительные требования

Чтобы отправлять запросы платежей с помощью пакета SDK Bot Framework для Node.js, необходимо выполнить обязательные предварительные задачи.

### <a name="register-and-configure-your-bot"></a>Регистрация и настройка бота

Обновите переменные среды бота для `MicrosoftAppId` и `MicrosoftAppPassword` до значений идентификатора приложения и пароля, которые были созданы для вашего бота в процессе [регистрации](~/bot-service-quickstart-registration.md). 

> [!NOTE]
> Сведения о том, как найти значения **AppID** и **AppPassword** для бота, см. в разделе [MicrosoftAppID и MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

### <a name="create-and-configure-merchant-account"></a>Создание и настройка учетной записи продавца

1. <a href="https://dashboard.stripe.com/register" target="_blank">Создайте и активируйте учетную запись Stripe, если у вас ее еще нет.</a>

2. <a href="https://seller.microsoft.com/dashboard/registration/seller/?accountprogram=botframework" target="_blank">Войдите в Seller Center с учетной записью Microsoft.</a>

3. Подключите свою учетную запись в Seller Center с помощью Stripe.

4. Перейдите в Seller Center к панели мониторинга и скопируйте значение **MerchantID**.

5. Обновите переменную среды `PAYMENTS_MERCHANT_ID` до значения, которое вы скопировали из панели мониторинга Центра продавца. 

[!INCLUDE [Payment process overview](../includes/snippet-payment-process-overview.md)]

## <a name="payment-bot-sample"></a>Пример бота для оплаты

Пример <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">бота для оплаты</a> предоставляет пример бота, который отправляет запрос платежей с помощью Node.js. Чтобы увидеть этот пример бота в действии, вы можете <a href="https://webchat.botframework.com/embed/paymentsample?s=d39Bk7JOMzQ.cwA.Rig.dumLki9bs3uqfWFMjXPn5PFnQVmT2VAVR1Zl1iPi07k" target="_blank">испытать его в веб-чате</a>, <a href="https://join.skype.com/bot/9fbc0f17-43eb-40fe-bf3b-af151e6ce45e" target="_blank">добавить его как контакт в Skype</a> или загрузить пример бота для оплаты и запустить его локально с помощью эмулятора Bot Framework. 

> [!NOTE]
> Чтобы завершить процесс оплаты, используя пример **бота для оплаты** в веб-чате или Skype, необходимо указать действительную кредитную или дебетовую карту в своей учетной записи Майкрософт (т. е. действительную карту эмитента карт в США). Сумма с карты не будет снята, и код безопасности карты не будет проверен, потому что пример **бота для оплаты** запускается в тестовом режиме (т. е. `PAYMENTS_LIVEMODE` установлен в `false` в **ENV**).

В следующих двух разделах этой статьи описываются три этапа процесса оплаты в контексте примера **бота для оплаты**.

## <a id="request-payment"></a> Запрос платежей

Бот может запросить платеж у пользователя, отправив сообщение, содержащее [форматированную карточку](bot-builder-nodejs-send-rich-cards.md) с кнопкой, которая указывает `type` "payment". Этот фрагмент кода из примера **бота для оплаты** создает сообщение, содержащее имиджевую карточку с кнопкой **Купить**, которую пользователь может щелкнуть (или нажать), чтобы инициировать процесс оплаты. 

[!code-javascript[Request payment](../includes/code/node-request-payment.js#requestPayment)]

В этом примере тип кнопки указан как `payments.PaymentActionType`, что приложение определяет как "payment". Значение кнопки заполняется функцией `createPaymentRequest`, которая возвращает объект `PaymentRequest`, содержащий информацию о поддерживаемых способах оплаты, сведениях и опциях. Дополнительные сведения о деталях реализации см. в разделе **app.js** в пример <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">бота для оплаты</a>.

На этом снимке экрана показана имиджевая карточка (с кнопкой **Купить**), созданная в приведенном выше фрагменте кода. 
 
![Пример бота для оплаты](../media/payments-bot-buy.png) 

> [!IMPORTANT]
> Любой пользователь, имеющий доступ к кнопке **Купить**, может использовать ее для запуска процесса оплаты. В контексте группового общения невозможно назначить кнопку для использования только конкретному пользователю. 

## <a id="user-experience"></a> Взаимодействие с пользователем

Когда пользователь нажимает кнопку **Купить**, он или она направляется на веб-интерфейс платежей, чтобы предоставить все необходимые данные для оплаты и доставки и контактную информацию через свою учетную запись Майкрософт. 

![Платеж Майкрософт](../media/microsoft-payment.png)

### <a name="http-callbacks"></a>Обратные вызовы HTTP

Обратные вызовы HTTP будут отправляться в бот, чтобы указать, что он должен выполнять определенные операции. Каждый обратный вызов будет событием, которое содержит такие значения свойств. 

| Свойство | Значение |
|----|----|
| `type` | invoke | 
| `name` | Указывает тип операции, которую должен выполнять бот (например, обновление адреса доставки, обновление способа доставки, завершение оплаты). | 
| `value` | Запрошенные полезные данные в формате JSON. | 
| `relatesTo` |  Описывает канал и пользователя, которые связаны с запросом платежа. | 

> [!NOTE]
> `invoke` – это специальный тип события, зарезервированный для использования в Microsoft Bot Framework. Отправитель события `invoke` ожидает, что бот подтвердит обратный вызов, отправив ответ HTTP.

## <a id="process-callbacks"></a> Обработка обратных вызовов

[!INCLUDE [Process callbacks overview](../includes/snippet-payment-process-callbacks-overview.md)]

### <a name="shipping-address-update-and-shipping-option-update-callbacks"></a>Обратные вызовы обновления адреса доставки и обновления способа доставки

При получении обратного вызова обновления адреса доставки или обновления варианта доставки боту будет предоставлено текущее состояние деталей платежа от клиента в свойстве `value` события.
Продавцам необходимо рассматривать эти обратные вызовы как статические. Получив входные данные платежа, вы будете вычислять некоторые выходные данные платежа и терпеть неудачу, если входное состояние, предоставленное клиентом, по какой-либо причине является недопустимым. 
Если бот определяет, что данная информация действительна как есть, отправьте код состояния HTTP `200 OK` вместе с неизменными реквизитами платежа. Кроме того, бот может отправлять код состояния HTTP `200 OK` вместе с обновленными реквизитами платежа, которые должны применяться до того, как заказ будет обработан. В некоторых случаях бот может определить, что обновленная информация недействительна и заказ не может быть обработан. Например, адрес доставки может указывать страну, в которую поставщик не отправляет товар. В этом случае бот может отправить код состояния HTTP `200 OK` и сообщение, заполняющее свойство ошибки объекта сведений оплаты. Отправка любого кода состояния HTTP в диапазоне `400` или `500` приведет к общей ошибке для клиента.

### <a name="payment-complete-callbacks"></a>Обратные вызовы для завершения оплаты

При получении обратного вызова для завершения оплаты боту будет предоставлена ​​копия первоначального, неизмененного запроса для оплаты, а также объекты ответа для оплаты в свойстве `value` события. Объект ответа оплаты будет содержать окончательный выбор клиента вместе с токеном для оплаты. Бот должен воспользоваться возможностью пересчета окончательного запроса для оплаты на основе первоначального запроса и окончательного выбора клиента. Предполагая, что выбор клиента будет действительным, бот должен проверить сумму и валюту в заголовке токена для оплаты, чтобы убедиться, что они соответствуют окончательному запросу для оплаты.  Если бот решает взимать плату с клиента, он должен взимать только плату, указанную в заголовке токена для оплаты, так как это цена, которую подтвердил клиент. Если существует несоответствие между значениями, которые бот ожидает, и значениями, которые он получает в обратном вызове для завершения оплаты, он может завершить запрос для оплаты ошибкой, отправляя код состояния HTTP `200 OK` наряду с определением поля как `failure`.   

Помимо проверки платежных реквизитов бот также должен удостовериться в выполнении заказа, прежде чем начнет обработку платежа. Например, он может проверить наличие приобретаемых товаров на складе. Если значения правильные и процессор для оплаты успешно зарядил токен для оплаты, бот должен ответить кодом состояния HTTP `200 OK` вместе с установкой поля результата на `success`, чтобы веб-интерфейс платежей отображал подтверждение оплаты. Токен для оплаты, который получает бот, может использоваться продавцом, который его запросил, только один раз и должен отправляться в Stripe (единственный процессор для оплаты, который в настоящее время поддерживает Bot Framework). Отправка любого кода состояния HTTP в диапазоне `400` или `500` приведет к общей ошибке для клиента.

Этот фрагмент кода из примера **бота для оплаты** обрабатывает обратные вызовы, которые получает бот. 

[!code-javascript[Request payment](../includes/code/node-request-payment.js#processCallback)]

В этом примере бот исследует свойство `name` входящего события для определения типа операции, которую он должен выполнить, а затем вызывает соответствующие методы для обработки обратного вызова. Дополнительные сведения о деталях реализации см. в разделе **app.js** в пример <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">бота для оплаты</a>.

## <a name="testing-a-payment-bot"></a>Тестирование бота для оплаты

[!INCLUDE [Test a payment bot](../includes/snippet-payment-test-bot.md)]

В примере <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">бота для оплаты</a> переменная среды `PAYMENTS_LIVEMODE` в **ENV** определяет, содержатся ли в обратных вызовах для завершения оплаты эмулированные токены для оплаты или токены для реальной оплаты. Если `PAYMENTS_LIVEMODE` присваивается `false`, заголовок добавляется в запрос исходящего бота и указывает, что бот находится в тестовом режиме, а обратный вызов завершения оплаты будет содержать эмулированный токен для оплаты, который не может быть оплачен. Если `PAYMENTS_LIVEMODE` присваивается `true`, то заголовок, который указывает, что бот находится в тестовом режиме, не указывается в запросе исходящего платежа бота, а обратный вызов завершения оплаты будет содержать реальный токен для оплаты, который бот будет отправлять в Stripe для обработки оплаты. Это будет реальная транзакция осуществления платежей в указанном платежном средстве. 

## <a name="additional-resources"></a>Дополнительные ресурсы

- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/sample-payments" target="_blank">Пример бота для оплаты</a>
- [Добавление вложений в виде форматированных карточек в сообщения](bot-builder-nodejs-send-rich-cards.md)
- <a href="http://www.w3.org/Payments/" target="_blank">Web Payments at W3C</a> (Веб-платежи в W3C) 

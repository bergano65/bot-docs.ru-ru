---
title: Запрос платежей | Документация Майкрософт
description: Сведения об отправке запроса платежей с помощью пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 23dd1c86d2605c50fafc572adcf9ca4369850131
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297848"
---
# <a name="request-payment"></a>Запрос платежей

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-request-payment.md)
> - [Node.js](../nodejs/bot-builder-nodejs-request-payment.md)

Если бот позволяет пользователям приобретать элементы, он делает запрос платежей, включая специальный тип кнопки в [форматированной карточке](bot-builder-dotnet-add-rich-card-attachments.md). Эта статья описывает отправку запроса платежей с помощью пакета SDK Bot Framework для .NET.

## <a name="prerequisites"></a>Предварительные требования

Чтобы отправлять запросы платежей с помощью пакета SDK Bot Framework для .NET, необходимо выполнить обязательные предварительные задачи.

### <a name="update-webconfig"></a>Обновление файла Web.config

Обновите файл **Web.config** бота, чтобы установить значения `MicrosoftAppId` и `MicrosoftAppPassword` для идентификатора приложения и пароля, которые были созданы для бота в процессе [регистрации](~/bot-service-quickstart-registration.md). 

> [!NOTE]
> Сведения о том, как найти значения **AppID** и **AppPassword** бота, см. в разделе [MicrosoftAppID и MicrosoftAppPassword](~/bot-service-manage-overview.md#microsoftappid-and-microsoftapppassword).

### <a name="create-and-configure-merchant-account"></a>Создание и настройка учетной записи продавца

1. <a href="https://dashboard.stripe.com/register" target="_blank">Создайте и активируйте учетную запись Stripe, если у вас ее еще нет.</a>

2. <a href="https://seller.microsoft.com/dashboard/registration/seller/?accountprogram=botframework" target="_blank">Войдите в Seller Center с учетной записью Microsoft.</a>

3. Подключите свою учетную запись в Seller Center с помощью Stripe.

4. Перейдите в Seller Center к панели мониторинга и скопируйте значение **MerchantID**.

5. Обновите файл **Web.config** бота, чтобы установить `MerchantId` значение, скопированное из панели мониторинга Seller Center. 

[!INCLUDE [Payment process overview](../includes/snippet-payment-process-overview.md)]

## <a name="payment-bot-sample"></a>Пример бота для оплаты

Пример <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">бота для оплаты</a> предоставляет пример бота, который отправляет запрос платежей с помощью .NET. Чтобы увидеть этот пример бота в действии, можно <a href="https://webchat.botframework.com/embed/paymentsample?s=d39Bk7JOMzQ.cwA.Rig.dumLki9bs3uqfWFMjXPn5PFnQVmT2VAVR1Zl1iPi07k" target="_blank">опробовать его в веб-чате</a>, <a href="https://join.skype.com/bot/9fbc0f17-43eb-40fe-bf3b-af151e6ce45e" target="_blank">добавить его как контакт в Skype</a> или загрузить пример бота для оплаты и запустить его локально с помощью Bot Framework Emulator. 

> [!NOTE]
> Чтобы завершить процесс оплаты, используя пример **бота для оплаты** в веб-чате или Skype, необходимо указать действительную кредитную или дебетовую карту в своей учетной записи Майкрософт (т. е. действительную карту эмитента карт в США). Сумма с карты не будет снята и код безопасности карты не будет проверен, потому что пример **бота для оплаты** запускается в тестовом режиме (т. е. `LiveMode` присваивается `false` в **Web.config**).

В следующих двух разделах этой статьи описываются три этапа процесса оплаты в контексте примера **бота для оплаты**.

## <a id="request-payment"></a> Запрос платежей

Бот может запросить платеж у пользователя, отправив сообщение, содержащее [вложенную форматированную карточку](bot-builder-dotnet-add-rich-card-attachments.md) с кнопкой, которая указывает `CardAction.Type` "payment". Этот фрагмент кода из примера **бота для оплаты** создает сообщение, содержащее имиджевую карточку с кнопкой **Купить**, которую пользователь может щелкнуть (или нажать), чтобы инициировать процесс оплаты. 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#requestPayment)]

В этом примере тип кнопки указан как `PaymentRequest.PaymentActionType`, который библиотека Bot Builder определяет как "payment". Значение кнопки заполняется методом `BuildPaymentRequest`, который возвращает объект `PaymentRequest`, содержащий информацию о поддерживаемых способах оплаты, сведениях и параметрах. Дополнительные сведения о деталях реализации см. в разделе **Dialogs/RootDialog.cs** в примере <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">бота для оплаты</a>.

На этом снимке экрана показана имиджевая карточка (с кнопкой **Купить**), созданная в приведенном выше фрагменте кода. 
 
![Пример бота для оплаты](../media/payments-bot-buy.png) 

> [!IMPORTANT]
> Любой пользователь, имеющий доступ к кнопке **Купить**, может использовать ее для запуска процесса оплаты. В контексте группового общения невозможно назначить кнопку для использования только конкретному пользователю. 

## <a id="user-experience"></a> Взаимодействие с пользователем

Когда пользователь нажимает кнопку **Купить**, он или она направляется на веб-интерфейс платежей, чтобы предоставить все необходимые данные для оплаты и доставки и контактную информацию через свою учетную запись Майкрософт. 

![Платеж Майкрософт](../media/microsoft-payment.png)

### <a name="http-callbacks"></a>Обратные вызовы HTTP

Обратные вызовы HTTP будут отправляться в бот, чтобы указать, что он должен выполнять определенные операции. Каждый обратный вызов будет [действием](bot-builder-dotnet-activities.md), которое содержит такие значения свойств. 

| Свойство | Значение |
|----|----|
| `Activity.Type` | invoke | 
| `Activity.Name` | Указывает тип операции, которую должен выполнять бот (например, обновление адреса доставки, обновление способа доставки, завершение оплаты). | 
| `Activity.Value` | Запрошенные полезные данные в формате JSON. | 
| `Activity.RelatesTo` |  Описывает канал и пользователя, которые связаны с запросом платежа. | 

> [!NOTE]
> `invoke` — это специальный тип события, зарезервированный для использования в Microsoft Bot Framework. Отправитель действия `invoke` ожидает, что бот подтвердит обратный вызов, отправив ответ HTTP.

## <a id="process-callbacks"></a> Обработка обратных вызовов

[!INCLUDE [Process callbacks overview](../includes/snippet-payment-process-callbacks-overview.md)]

### <a name="shipping-address-update-and-shipping-option-update-callbacks"></a>Обратные вызовы обновления адреса доставки и обновления способа доставки

[!INCLUDE [Process shipping address and shipping option callbacks](../includes/snippet-payment-process-callbacks-1.md)]

### <a name="payment-complete-callbacks"></a>Обратные вызовы для завершения оплаты

При получении обратного вызова для завершения оплаты боту будет предоставлена ​​копия первоначального, неизмененного запроса оплаты, а также объекты ответа оплаты в свойстве `Activity.Value`. Объект ответа оплаты будет содержать окончательный выбор клиента вместе с токеном для оплаты. Бот должен воспользоваться возможностью пересчета окончательного запроса для оплаты на основе первоначального запроса и окончательного выбора клиента. Предполагая, что выбор клиента будет действительным, бот должен проверить сумму и валюту в заголовке токена для оплаты, чтобы убедиться, что они соответствуют окончательному запросу для оплаты.  Если бот решает взимать плату с клиента, он должен взимать только плату, указанную в заголовке токена для оплаты, так как это цена, которую подтвердил клиент. Если существует несоответствие между значениями, которые бот ожидает, и значениями, которые он получает в обратном вызове для завершения оплаты, он может завершить запрос для оплаты ошибкой, отправляя код состояния HTTP `200 OK` наряду с определением поля как `failure`.   

Помимо проверки платежных реквизитов бот также должен удостовериться в выполнении заказа, прежде чем начнет обработку платежа. Например, он может проверить наличие приобретаемых товаров на складе. Если значения правильные и процессор для оплаты успешно зарядил токен для оплаты, бот должен ответить кодом состояния HTTP `200 OK` вместе с установкой поля результата на `success`, чтобы веб-интерфейс платежей отображал подтверждение оплаты. Токен для оплаты, который получает бот, может использоваться продавцом, который его запросил, только один раз и должен отправляться в Stripe (единственный процессор для оплаты, который в настоящее время поддерживает Bot Framework). Отправка любого кода состояния HTTP в диапазоне `400` или `500` приведет к общей ошибке для клиента.

Метод `OnInvoke` из примера **бота для оплаты** обрабатывает обратные вызовы, которые получает бот. 

[!code-csharp[Request payment](../includes/code/dotnet-request-payment.cs#processCallback)]

В этом примере бот исследует свойство `Name` входящего действия для определения типа операции, которую он должен выполнить, а затем вызывает соответствующий метод для обработки обратного вызова. Дополнительные сведения о деталях реализации см. в разделе **Controllers/MessagesControllers.cs** в примере <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">бота для оплаты</a>.

## <a name="testing-a-payment-bot"></a>Тестирование бота для оплаты

[!INCLUDE [Test a payment bot](../includes/snippet-payment-test-bot.md)]

В примере <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">бота для оплаты</a> параметр конфигурации `LiveMode` в **Web.config** определяет, содержатся ли в обратных вызовах для завершения оплаты эмулированные или реальные токены для оплаты. Если `LiveMode` присваивается `false`, заголовок добавляется в запрос исходящего бота и указывает, что бот находится в тестовом режиме, а обратный вызов завершения оплаты будет содержать эмулированный токен для оплаты, который не может быть оплачен. Если `LiveMode` присваивается `true`, то заголовок, который указывает, что бот находится в тестовом режиме, не указывается в запросе исходящего платежа бота, а обратный вызов завершения оплаты будет содержать реальный токен для оплаты, который бот будет отправлять в Stripe для обработки оплаты. Это будет реальная транзакция осуществления платежей в указанном платежном средстве. 

## <a name="additional-resources"></a>Дополнительные ресурсы

- <a href="https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/sample-payments" target="_blank">Пример бота для оплаты</a>
- [Activities overview](bot-builder-dotnet-activities.md) (Общие сведения о действиях)
- [Добавление вложений в виде форматированных карточек в сообщения](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="http://www.w3.org/Payments/" target="_blank">Web Payments at W3C</a> (Веб-платежи в W3C) 
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочная информация по пакету SDK Bot Framework для .NET</a>

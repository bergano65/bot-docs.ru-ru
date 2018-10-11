---
title: Ключи Application Insights | Документация Майкрософт
description: Узнайте, как получить ключи Application Insights, чтобы добавить телеметрию в бот.
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 07fb6e9630996a61932da99b0575d43f4604141e
ms.sourcegitcommit: 3cb288cf2f09eaede317e1bc8d6255becf1aec61
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/27/2018
ms.locfileid: "47389433"
---
# <a name="application-insights-keys"></a>Ключи Application Insights

В Azure Application Insights данные о приложении отображаются в **ресурсе** Microsoft Azure. Чтобы добавить данные телеметрии для бота, требуется подписка Azure и ресурс Application Insights, созданный для вашего бота. Из этого ресурса можно получить три ключа для настройки бота:

1. Ключ инструментирования
2. Идентификатор приложения
3. Ключ API

В этом разделе показано, как создать ключи Application Insights.

> [!NOTE]
> Во время создания или регистрации бота вы могли *включить* или *выключить* **Application Insights**. Если вы *включили* эту службу, у бота уже есть все необходимые ключи Application Insights. Если же вы *отключили* ее, выполните инструкции в этом разделе для создания этих ключей вручную.

## <a name="instrumentation-key"></a>Ключ инструментирования

Чтобы получить ключ инструментирования, сделайте следующее:
1. На [портале Azure](http://portal.azure.com) в разделе "Монитор" создайте ресурс **Application Insights** (или используйте существующий).
![Снимок экрана портала со списком Application Insights](~/media/portal-app-insights-add-new.png)

2. В списке ресурсов Application Insights щелкните только что созданный ресурс Application Insight.

3. Нажмите **Обзор**.

4. Разверните блок **Основные компоненты** и найдите **ключ инструментирования**. 
![Снимок экрана портала с ключом инструментирования](~/media/portal-app-insights-instrumentation-key.png)

5. Скопируйте **ключ инструментирования** и вставьте его в поле **Application Insights Instrumentation Key** (Ключ инструментирования Application Insights) в параметрах своего бота.

## <a name="application-id"></a>Идентификатор приложения

Чтобы получить идентификатор приложения, сделайте следующее:
1. В ресурсе Application Insights щелкните **Доступ через API**.

2. Скопируйте **идентификатор приложения** и вставьте его в поле **Application Insights Application ID** (Идентификатор приложения Application Insights) в параметрах своего бота. 
![Снимок экрана портала с идентификатором приложения](~/media/portal-app-insights-appid.png)

## <a name="api-key"></a>Ключ API

Чтобы получить ключ API, сделайте следующее:
1. Из ресурса Application Insights щелкните **Доступ через API**.

2. Щелкните **Создание ключа API**.

3. Введите краткое описание, установите флажок **Read telementry** (Чтение телеметрии) и нажмите кнопку **Создать ключ**.
![Снимок экрана портала с идентификатором приложения и ключом API](~/media/portal-app-insights-appid-apikey.png)

   > [!WARNING]
   > Скопируйте этот **ключ API** и сохраните его, так как вы больше не сможете его просмотреть. Если ключ будет утерян, понадобится создать новый.

4. Скопируйте ключ API в поле **Ключ API Application Insights** в параметрах своего бота.

## <a name="additional-resources"></a>Дополнительные ресурсы
Дополнительные сведения о подключении этих полей в параметрах бота см. в разделе [Включение аналитики](~/bot-service-manage-analytics.md#enable-analytics).
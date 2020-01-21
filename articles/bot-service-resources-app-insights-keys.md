---
title: Ключи Application Insights — Служба Azure Bot
description: Узнайте, как получить ключи Application Insights, чтобы добавить телеметрию в бот.
author: v-ducvo
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/19/2019
ms.openlocfilehash: bec8ff08f7aa591af8379018ebbbb5e6f1d48a29
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75788756"
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
1. На [портале Azure](https://portal.azure.com) в разделе "Монитор" создайте ресурс **Application Insights** (или используйте существующий).
![Снимок экрана портала со списком Application Insights](~/media/portal-app-insights-add-new.png)

2. В списке ресурсов Application Insights щелкните только что созданный ресурс Application Insight.

3. Нажмите **Обзор**.

4. Разверните блок **Основные компоненты** и найдите **ключ инструментирования**. 
![Снимок экрана со страницей обзора на портале](~/media/portal-app-insights-instrumentation-key-dropdown.png)
![Снимок экрана с ключом инструментирования на портале](~/media/portal-app-insights-instrumentation-key.png)

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

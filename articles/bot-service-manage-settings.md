---
title: Настройка параметров бота — Служба Azure Bot
description: Узнайте, как настроить различные параметры своего бота с помощью портала Azure.
keywords: configure bot settings, Display Name, Icon, Application Insights, Settings blade
author: v-royhar
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: a3f822c7af635d7c7cac7d9f56038bd5d20a1722
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75794993"
---
# <a name="configure-bot-settings"></a>Настройка параметров бота

Параметры бота, такие как отображаемое имя, значок и Application Insights, можно просмотреть и изменить в колонке **Параметры**.

![Колонка параметров бота](~/media/bot-service-portal-configure-settings/bot-settings-blade.png)

Ниже приведен список полей на вкладке **Параметры**:

| Поле | Description |
| :---  | :---        |
| Значок | Пользовательский значок для визуального обозначения бота на каналах и значок для Skype, Кортаны и других служб. Этот значок должен быть в формате PNG и иметь размер не более 30 КБ. Это значение можно изменить в любое время. |
| Отображаемое имя | Имя бота на каналах и в каталогах. Это значение можно изменить в любое время. Допустимое число символов: 35. |
| Bot handle (Дескриптор бота) | Уникальный идентификатор для бота. Это значение нельзя изменить после создания бота с помощью службы Bot. |
| Messaging endpoint (Конечная точка обмена сообщениями) | Конечная точка для обмена данными с ботом. |
| Microsoft App ID (Идентификатор приложения Майкрософт) | Уникальный идентификатор для бота. Это значение невозможно изменить. Вы можете создать пароль, щелкнув ссылку **Управление**. |
| Application Insights Instrumentation key (Ключ инструментирования Application Insights) | Уникальный ключ для телеметрии бота. Скопируйте ключ Azure Application Insights в это поле, если вы хотите получать данные телеметрии для этого бота. Это значение является необязательным. Для ботов, созданных на портале Azure, этот ключ создается автоматически. Дополнительные сведения об этом поле см. в статье [Ключи Application Insights](~/bot-service-resources-app-insights-keys.md). |
| Ключ API Application Insights | Уникальный ключ для аналитики бота. Скопируйте ключ API Azure Application Insights в это поле, если вы хотите просматривать аналитику бота на панели мониторинга. Это значение является необязательным. Дополнительные сведения об этом поле см. в статье [Ключи Application Insights](~/bot-service-resources-app-insights-keys.md). |
| Application Insights Application ID (Идентификатор приложения Application Insights) | Уникальный ключ для аналитики бота. Скопируйте ключ идентификатора Azure Insights в это поле, если вы хотите просматривать аналитику бота на панели мониторинга. Это значение является необязательным. Для ботов, созданных на портале Azure, этот ключ создается автоматически. Дополнительные сведения об этом поле см. в статье [Ключи Application Insights](~/bot-service-resources-app-insights-keys.md). |

> [!NOTE]
> После изменения параметров для бота нажмите кнопку **Сохранить** в верхней части колонки, чтобы сохранить параметры нового бота.

## <a name="next-steps"></a>Дальнейшие действия
Теперь, когда вы узнали, как настроить параметры для службы бота, узнайте о способах настройки подготовки речи.
> [!div class="nextstepaction"]
> [Подготовка речи](bot-service-manage-speech-priming.md)
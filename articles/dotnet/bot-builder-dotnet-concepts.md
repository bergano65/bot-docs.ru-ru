---
title: Основные концепции, которые используются в пакете SDK Bot Framework для .NET — Служба Azure Bot
description: Обзор основных понятий, используемых при работе с пакетом SDK Bot Framework для .NET, и предоставляемых в нем инструментов для создания и развертывания чат-ботов.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 027006f2ad3d6aff1094142a41d791ba0e5d8922
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789825"
---
# <a name="key-concepts-in-the-bot-framework-sdk-for-net"></a>Основные концепции, которые используются в пакете SDK Bot Framework для .NET.

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-concepts.md)
> - [Node.js](../nodejs/bot-builder-nodejs-concepts.md)

В этой статье раскрыты основные понятия, используемые при работе с пакетом SDK Bot Framework для .NET.

## <a name="connector"></a>Соединитель

[Bot Framework Connector](bot-builder-dotnet-connector.md) предоставляет единый интерфейс REST API, позволяющий боту взаимодействовать через различные каналы связи, такие как Skype, электронная почта, Slack и т. д. Эта служба обеспечивает обмен данными между ботом и пользователем, ретранслируя сообщения из бота в канал и из канала в бот. 

В пакете SDK Bot Framework для .NET библиотека [Connector][connectorLibrary] обеспечивает доступ к соединителю. 

## <a name="activity"></a>Действие

[!INCLUDE [Activity concept overview](../includes/snippet-dotnet-concept-activity.md)]

Дополнительные сведения о действиях в пакете SDK Bot Framework для .NET, см. в [этой статье](bot-builder-dotnet-activities.md).

## <a name="dialog"></a>Диалог

Если вы создаете бот с помощью пакета SDK Bot Framework для .NET, для моделирования беседы и управления [ее потоком](../bot-service-design-conversation-flow.md#dialog-stack) можно использовать [диалоги](bot-builder-dotnet-dialogs.md). Диалог можно сочетать с другими диалогами для наиболее эффективного повторного использования. Кроме того, в контексте диалога сохраняется [стек диалогов](../bot-service-design-conversation-flow.md), которые активны в беседе в любой момент времени. Беседу, содержащую диалоги, можно переносить между компьютерами, что обеспечивает масштабирование реализуемого бота. 

В состав пакета SDK Bot Framework для .NET входит библиотека [Builder][builderLibrary], которая позволяет управлять диалогами.

## <a name="formflow"></a>FormFlow

Чтобы упростить создание бота, который собирает сведения от пользователя, вы можете использовать [FormFlow](bot-builder-dotnet-formflow.md) в пакете SDK Bot Framework для .NET. Например бот, который принимает заказы на сэндвичи должен собирать несколько фрагментов данных от пользователя, такие как тип хлеба, выбор начинки, размер и т. д. Учитывая основные рекомендации, FormFlow может автоматически создавать диалоги, необходимые для управления такой интерактивной беседой.

## <a name="state"></a>Штат

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

Дополнительные сведения об управлении данными о состоянии с помощью пакета SDK Bot Framework для .NET см. в [этой статье](bot-builder-dotnet-state.md).

## <a name="naming-conventions"></a>Соглашения об именовании.

В библиотеке пакета SDK Bot Framework для .NET используется строго типизированное соглашения об именовании в стиле Pascal. Но в сообщениях JSON, которые передаются по сети, применяется соглашения об именовании, согласно которому используется нижний регистр. Например, в сообщении JSON, передаваемом по сети, свойство **ReplyToId** в C# сериализуется как **replyToId**.

## <a name="security"></a>безопасность

Убедитесь, что конечную точку бота может вызывать только служба Bot Framework Connector. Дополнительные сведения об этом см. в статье [о защите бота](bot-builder-dotnet-security.md).

## <a name="next-steps"></a>Дальнейшие действия

Вы ознакомились с принципами работы каждого бота. Теперь вы можете быстро [создать бот в Visual Studio](bot-builder-dotnet-quickstart.md) с помощью шаблона. Далее переходите к изучению отдельных основных понятий, начиная с диалогов.

> [!div class="nextstepaction"]
> [Диалоги в пакете SDK Bot Framework для .NET.](bot-builder-dotnet-dialogs.md)

[connectorLibrary]: /dotnet/api/microsoft.bot.connector

[builderLibrary]: /dotnet/api/microsoft.bot.builder.dialogs

---
title: Создание бота с помощью пакета SDK Bot Builder для JavaScript | Документация Майкрософт
description: Быстрое создание бота с помощью пакета SDK Bot Builder для JavaScript.
keywords: Краткое руководство, пакет SDK Bot Builder, приступая к работе
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/23/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 473ae0ce9daff8d93ef91268fffff4fbf45e7752
ms.sourcegitcommit: abde9e0468b722892f94caf2029fae165f96092f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/09/2018
ms.locfileid: "48875701"
---
```
{
    "name": "BasicBot",
    "description": "Demo",
    "services": [
        {
            "type": "endpoint",
            "name": "development",
            "id": "http://localhost:3978/api/messages",
            "appId": "",
            "appPassword": "",
            "endpoint": "http://localhost:3978/api/messages"
        },
        {
            "type": "luis",
            "name": "basic-bot-LUIS",
            "appId": "<your app id>",
            "version": "0.1",
            "authoringKey": "<your authoring key>",
            "subscriptionKey": "<your subscription key>",
            "region": "westus",
            "id": "206"
        }
    ],
    "secretKey": "",
    "version": "2.0"
}
```

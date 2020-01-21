---
title: Руководство по созданию и развертыванию простого бота — Служба Azure Bot
description: Узнайте, как создать простой бот и развернуть его в Azure.
keywords: echo bot, deploy, azure, tutorial
author: Ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: fadc7410925d337a518129736c9374035fe2114d
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791216"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>Руководство. Создание и развертывание простого бота

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

В этом руководстве описано, как создать простой бот на основе пакета SDK Bot Framework и развернуть его в Azure. Если вы уже создали бот и запустили его локально, сразу переходите к разделу [Развертывание бота](#deploy-your-bot).

В этом руководстве описано следующее.

> [!div class="checklist"]
> * Создание простого бота Echo
> * Запуск бота в локальной среде и взаимодействие с ним
> * Публикация бота

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

# <a name="pythontabpython"></a>[Python](#tab/python)

[!INCLUDE [python quickstart](~/includes/quickstart-python.md)]

---

## <a name="deploy-your-bot"></a>Развертывание бота

### <a name="prerequisites"></a>предварительные требования
[!INCLUDE [deploy prerequisite](~/includes/deploy/snippet-prerequisite.md)]

### <a name="prepare-for-deployment"></a>Подготовка к развертыванию
[!INCLUDE [deploy prepare intro](~/includes/deploy/snippet-prepare-deploy-intro.md)]

#### <a name="1-login-to-azure"></a>1. Вход в Azure
[!INCLUDE [deploy az login](~/includes/deploy/snippet-az-login.md)]

#### <a name="2-set-the-subscription"></a>2. Настройка подписки
[!INCLUDE [deploy az subscription](~/includes/deploy/snippet-az-set-subscription.md)]

#### <a name="3-create-an-app-registration"></a>3. Регистрация приложения
[!INCLUDE [deploy create app registration](~/includes/deploy/snippet-create-app-registration.md)]

#### <a name="4-deploy-via-arm-template"></a>4. Развертывание с использованием шаблона ARM
Вы можете развернуть бот в новой группе ресурсов или использовать имеющуюся. Выберите любой вариант, который вам подходит.

> [!NOTE]
> Боты Python нельзя развертывать в группе ресурсов, содержащей службы или боты Windows.  Хотя в одной группе ресурсов можно развернуть несколько ботов Python, другие службы (LUIS, QnA и пр.) следует создавать в другой группе ресурсов.
>

##### <a name="deploy-via-arm-template-with-new-resource-group"></a>**Развертывание с помощью шаблона ARM в новой группе ресурсов**
[!INCLUDE [ARM with new resourece group](~/includes/deploy/snippet-ARM-new-resource-group.md)]

##### <a name="deploy-via-arm-template-with-existing-resource-group"></a>**Развертывание с помощью шаблона ARM в существующей группе ресурсов**
[!INCLUDE [ARM with existing resourece group](~/includes/deploy/snippet-ARM-existing-resource-group.md)]

#### <a name="5-prepare-your-code-for-deployment"></a>5. Подготовка кода к развертыванию
##### <a name="retrieve-or-create-necessary-iiskudu-files"></a>**Получение или создание файлов, необходимых для IIS или Kudu**
[!INCLUDE [retrieve or create IIS/Kudu files](~/includes/deploy/snippet-IIS-Kudu-files.md)]

##### <a name="zip-up-the-code-directory-manually"></a>**Архивация каталога кода вручную**
[!INCLUDE [zip up code](~/includes/deploy/snippet-zip-code.md)]

### <a name="deploy-code-to-azure"></a>Развертывание кода в Azure
[!INCLUDE [deploy code to Azure](~/includes/deploy/snippet-deploy-code-to-az.md)]

### <a name="test-in-web-chat"></a>Тестирование в веб-чате
[!INCLUDE [test in web chat](~/includes/deploy/snippet-test-in-web-chat.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Использование QnA Maker в боте для ответов на вопросы.](bot-builder-tutorial-add-qna.md)
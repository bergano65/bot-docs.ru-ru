---
title: Руководство по созданию и развертыванию простого бота — Служба Azure Bot
description: Узнайте, как создать простой бот и развернуть его в Azure.
keywords: echo bot, deploy, azure, tutorial
author: Ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 03/23/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 3b4e3652f7799fb09219b8536bd98a78e0952223
ms.sourcegitcommit: 126c4f8f8c7a3581e7521dc3af9a937493e6b1df
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2020
ms.locfileid: "80499910"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>Руководство по Создание и развертывание простого бота

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

В этом руководстве описано, как создать простой бот на основе пакета SDK Bot Framework и развернуть его в Azure. Если вы уже создали бот и запустили его локально, сразу переходите к разделу [Развертывание бота](#deploy-your-bot).

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Создание простого бота Echo
> * Запуск бота в локальной среде и взаимодействие с ним
> * Публикация бота

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

# <a name="c"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

# <a name="python"></a>[Python](#tab/python)

[!INCLUDE [python quickstart](~/includes/quickstart-python.md)]

---

## <a name="deploy-your-bot"></a>Развертывание бота

### <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [deploy prerequisite](~/includes/deploy/snippet-prerequisite.md)]

### <a name="prepare-for-deployment"></a>Подготовка к развертыванию

> [!TIP]
> В этой процедуре для развертывания бота используется ZIP-файл. В C# такая процедура может завершиться сбоем, если на этапе сборки в качестве конфигурации решения указано значение **Debug**.
> В Visual Studio для конфигурации решения укажите значение **Release**. Затем для решения выполните повторную сборку с очисткой, прежде чем продолжить.

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

[!INCLUDE [Work around for .NET Core 3.1 SDK](~/includes/deploy/samples-workaround-3-1.md)]

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
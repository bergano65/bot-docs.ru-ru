---
title: Развертывание бота — Служба Azure Bot
description: Развертывание бота в облаке Azure
keywords: deploy bot, azure deploy bot, publish bot
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: conceptual
ms.service: bot-service
ms.date: 03/23/2020
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 9e4c67f644e9797f3e210546a91d09f5161aa100
ms.sourcegitcommit: 126c4f8f8c7a3581e7521dc3af9a937493e6b1df
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2020
ms.locfileid: "80499829"
---
# <a name="deploy-your-bot"></a>Развертывание бота

[!INCLUDE [applies-to](./includes/applies-to.md)]

В этой статье описано, как развернуть простого бота в Azure. Мы объясним, как подготовить бота к развертыванию, развернуть его в Azure и протестировать в Web Chat. Эту статью полезно изучить до выполнения описанных действий, чтобы ознакомиться со всеми процессами, связанными с развертыванием бота.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [deploy prerequisite](~/includes/deploy/snippet-prerequisite.md)]

## <a name="prepare-for-deployment"></a>Подготовка к развертыванию

[!INCLUDE [deploy prepare intro](~/includes/deploy/snippet-prepare-deploy-intro.md)]

### <a name="1-login-to-azure"></a>1. Вход в Azure

[!INCLUDE [deploy az login](~/includes/deploy/snippet-az-login.md)]

### <a name="2-set-the-subscription"></a>2. Настройка подписки

[!INCLUDE [deploy az subscription](~/includes/deploy/snippet-az-set-subscription.md)]

### <a name="3-create-the-application-registration"></a>3. Создание регистрации приложения

[!INCLUDE [deploy create app registration](~/includes/deploy/snippet-create-app-registration.md)]

### <a name="4-create-the-bot-application-service"></a>4. Создание службы приложения бота

При создании службы приложения бота можно развернуть бота в новой или существующей группе ресурсов. Выберите любой вариант, который вам подходит.

> [!IMPORTANT]
> Боты Python нельзя развертывать в группе ресурсов, содержащей службы или боты Windows.  Хотя в одной группе ресурсов можно развернуть несколько ботов Python, другие службы (LUIS, QnA и пр.) следует создавать в другой группе ресурсов.

Убедитесь, что у вас есть правильный путь к каталогу шаблонов развертывания ARM проекта `DeploymentTemplates`. Он нужен для присваивания значения `template-file`.

#### <a name="deploy-via-arm-template-with-new-resource-group"></a>**Развертывание с помощью шаблона ARM (в **новой** группе ресурсов)**

<!-- ##### Create Azure resources -->
[!INCLUDE [ARM with new resource group](~/includes/deploy/snippet-ARM-new-resource-group.md)]


#### <a name="deploy-via-arm-template-with-existing--resource-group"></a>**Развертывание с помощью шаблона ARM (в **существующей** группе ресурсов)**

[!INCLUDE [ARM with existing resource group](~/includes/deploy/snippet-ARM-existing-resource-group.md)]

---

### <a name="5-prepare-your-code-for-deployment"></a>5. Подготовка кода к развертыванию

[!INCLUDE [Work around for .NET Core 3.1 SDK](~/includes/deploy/samples-workaround-3-1.md)]

#### <a name="51-retrieve-or-create-necessary-iiskudu-files"></a>5.1 Получение или создание файлов, необходимых для IIS либо Kudu

[!INCLUDE [retrieve or create IIS/Kudu files](~/includes/deploy/snippet-IIS-Kudu-files.md)]

#### <a name="52-zip-up-the-code-directory-manually"></a>5.2 Архивация каталога кода вручную

[!INCLUDE [zip up code](~/includes/deploy/snippet-zip-code.md)]

## <a name="deploy-code-to-azure"></a>Развертывание кода в Azure

[!INCLUDE [deploy code to Azure](~/includes/deploy/snippet-deploy-code-to-az.md)]

## <a name="test-in-web-chat"></a>Тестирование в веб-чате

[!INCLUDE [test in web chat](~/includes/deploy/snippet-test-in-web-chat.md)]

## <a name="additional-information"></a>Дополнительные сведения

Развертывание бота в Azure подразумевает оплату за используемые службы. Руководство по [управлению счетами и расходами](https://docs.microsoft.com/azure/billing/) поможет вам понять, как расшифровывать счета Azure, отслеживать использование и расходы, а также управлять учетными записями и подписками.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Настройка непрерывного развертывания](bot-service-build-continuous-deployment.md)

<!-- ## Appendix

[!INCLUDE [deploy csharp bot to Azure](~/includes/deploy/snippet-deploy-simple-csharp-echo-bot.md)] -->

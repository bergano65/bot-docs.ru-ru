---
title: Руководство по созданию и развертыванию простого бота | Документация Майкрософт
description: Узнайте, как создать простой бот и развернуть его в Azure.
keywords: echo bot, deploy, azure, tutorial
author: Ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 04/18/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: bf3c8ffb00034a39700f10cd326af764b8deeb6c
ms.sourcegitcommit: aea57820b8a137047d59491b45320cf268043861
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/22/2019
ms.locfileid: "59905127"
---
# <a name="tutorial-create-and-deploy-a-basic-bot"></a>Руководство по Создание и развертывание простого бота

[!INCLUDE [applies-to-v4](../includes/applies-to.md)]

В этом руководстве описано, как создать простой бот на основе пакета SDK Bot Framework и развернуть его в Azure. Если вы уже создали бот и запустили его локально, сразу переходите к разделу [Развертывание бота](#deploy-your-bot).

Из этого руководства вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * Создание простого бота Echo
> * Запуск бота в локальной среде и взаимодействие с ним
> * Публикация бота

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

# <a name="ctabcsharp"></a>[C#](#tab/csharp)

[!INCLUDE [dotnet quickstart](~/includes/quickstart-dotnet.md)]

# <a name="javascripttabjavascript"></a>[JavaScript](#tab/javascript)

[!INCLUDE [javascript quickstart](~/includes/quickstart-javascript.md)]

---

## <a name="deploy-your-bot"></a>Развертывание бота

Разверните созданный бот в Azure.

### <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]

### <a name="login-to-azure-cli-and-set-your-subscription"></a>Вход в Azure CLI и настройка подписки.

Итак, вы уже создали бот и протестировали его локально, а теперь хотите развернуть его в Azure.

[!INCLUDE [az login snippet](~/includes/deploy/snippet-az-login.md)]

### <a name="create-a-web-app-bot"></a>Создание бота веб-приложения

Если у вас еще нет группы ресурсов для публикации бота, создайте ее.

[!INCLUDE [az create group snippet](~/includes/deploy/snippet-az-create-group.md)]

[!INCLUDE [az create web app snippet](~/includes/deploy/snippet-create-web-app.md)]

Прежде чем продолжить работу, ознакомьтесь с описанными ниже инструкциями, выбрав нужный раздел в зависимости от типа учетной записи электронной почты, с которой вы входите в Azure.

#### <a name="msa-email-account"></a>Учетная запись электронной почты MSA

Если вы используете учетную запись электронной почты [MSA](https://en.wikipedia.org/wiki/Microsoft_account), вам потребуется создать идентификатор и пароль приложения на портале регистрации приложений для использования в команде `az bot create`.

[!INCLUDE [create bot msa snippet](~/includes/deploy/snippet-create-bot-msa.md)]

#### <a name="business-or-school-account"></a>Рабочая или учебная учетная запись

[!INCLUDE [create bot snippet](~/includes/deploy/snippet-create-bot.md)]

### <a name="download-the-bot-from-azure"></a>Скачивание бота из Azure

Теперь скачайте бот, который вы только что создали. 
[!INCLUDE [download bot snippet](~/includes/deploy/snippet-download-bot.md)]

[!INCLUDE [download keys snippet](~/includes/snippet-abs-key-download.md)]

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>Расшифровка скачанного файла .bot и применение его в проекте

Конфиденциальные сведения в файле .bot шифруются, и для удобства работы их следует расшифровать. 

Для начала перейдите в каталог со скачанным ботом.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="test-your-bot-locally"></a>Локальное тестирование бота

На этом этапе бот будет работать точно так же, как с использованием старого файла `.bot`. Убедитесь, что он работает правильно с новым файлом `.bot`.

Теперь вы увидите в эмуляторе конечную точку *Production*. Если ее там нет, скорее всего, вы используете старый файл `.bot`.

### <a name="publish-your-bot-to-azure"></a>Публикация бота в Azure

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

Теперь вы можете протестировать бот в веб-чате.

## <a name="additional-resources"></a>Дополнительные ресурсы

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>Дополнительная информация
> [!div class="nextstepaction"]
> Чтобы продолжить, [добавьте в бот дополнительные службы](bot-builder-tutorial-add-qna.md).


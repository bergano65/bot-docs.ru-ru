---
title: Развертывание бота с помощью Azure CLI | Документация Майкрософт
description: Развертывание бота в облаке Azure.
keywords: deploy bot, azure deploy, publish bot, az deploy bot, visual studio deploy bot, msbot publish, msbot clone
author: ivorb
ms.author: v-ivorb
manager: kamrani
ms.topic: get-started-article
ms.service: bot-service
ms.subservice: abs
ms.date: 01/07/2019
ms.openlocfilehash: 3ebc13cf9e2d111d716d081c36f125d28a441811
ms.sourcegitcommit: bdb981c0b11ee99d128e30ae0462705b2dae8572
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/17/2019
ms.locfileid: "54360744"
---
# <a name="deploy-your-bot-using-azure-cli"></a>Развертывание бота с помощью Azure CLI

[!INCLUDE [pre-release-label](./includes/pre-release-label.md)]

Завершив создание и локальную проверку бота, его можно развернуть в Azure, чтобы сделать доступным из любого расположения. Развертывание бота в Azure подразумевает оплату за используемые службы. Руководство по [управлению счетами и расходами](https://docs.microsoft.com/en-us/azure/billing/) поможет вам понять, как расшифровывать счета Azure, отслеживать использование и расходы, а также управлять учетными записями и подписками.

В этой статье объясняется, как развернуть в Azure боты, написанные на C# или JavaScript, с помощью средств командной строки `az` и `msbot`. Эту статью полезно изучить до выполнения описанных действий, чтобы ознакомиться со всеми процессами, связанными с развертыванием бота.

## <a name="prerequisites"></a>Предварительные требования

[!INCLUDE [prerequisite snippet](~/includes/deploy/snippet-prerequisite.md)]


## <a name="deploy-javascript-and-c-bots-using-az-cli"></a>Развертывание ботов, написанных на JavaScript и C#, с помощью az cli

Итак, вы уже создали бот и протестировали его локально, а теперь хотите развернуть его в Azure. В этих шагах предполагается, что вы уже создали необходимые ресурсы Azure.

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

### <a name="decrypt-the-downloaded-bot-file-and-use-in-your-project"></a>Расшифровка скачанного файла .bot и применение его в проекте

Все конфиденциальные сведения в файле .bot шифруются.

[!INCLUDE [decrypt bot snippet](~/includes/deploy/snippet-decrypt-bot.md)]

### <a name="update-the-bot-file"></a>Обновление файла .bot

Если ваш бот использует службы LUIS, QnA Maker или службу диспетчеризации, следует добавить ссылки на них в файл .bot. В противном случае этот шаг можно пропустить.

1. Откройте бот в эмуляторе BotFramework, используя новый файл .bot. Бот не нужно запускать локально.
1. На панели **обозревателя ботов** разверните раздел **СЛУЖБЫ**.
1. Чтобы добавить ссылки на приложения LUIS, щелкните значок плюс (+) справа от раздела **СЛУЖБЫ**.
   1. Выберите **Add Language Understanding (LUIS)** (Добавить "Распознавание речи" (LUIS)).
   1. Если появится запрос на вход в учетную запись Azure, выполните его.
   1. Здесь вы увидите список приложений LUIS, к которым у вас есть доступ. Выберите те, которые нужны для вашего бота.
1. Чтобы добавить ссылки на базу знаний QnA Maker, щелкните значок плюс (+) справа от раздела **СЛУЖБЫ**.
   1. Выберите **Add QnA Maker** (Добавить QnA Maker).
   1. Если появится запрос на вход в учетную запись Azure, выполните его.
   1. Здесь вы увидите список баз знаний, к которым у вас есть доступ. Выберите те, которые нужны для вашего бота.
1. Чтобы добавить ссылки на модели диспетчеризации, щелкните значок плюс (+) справа от раздела **СЛУЖБЫ**.
   1. Выберите **Add Dispatch** (Добавить диспетчеризацию).
   1. Если появится запрос на вход в учетную запись Azure, выполните его.
   1. Здесь вы увидите список моделей диспетчеризации, к которым у вас есть доступ. Выберите те, которые нужны для вашего бота.

### <a name="test-your-bot-locally"></a>Локальное тестирование бота

На этом этапе бот должен работать точно так же, как со старым файлом .bot. Убедитесь, что с новым файлом .bot он работает правильно.

### <a name="publish-your-bot-to-azure"></a>Публикация бота в Azure

<!-- TODO: re-encrypt your .bot file? -->

[!INCLUDE [publish snippet](~/includes/deploy/snippet-publish.md)]

<!-- TODO: If we tell them to re-encrypt, this step is not necessary. -->

[!INCLUDE [clear encryption snippet](~/includes/deploy/snippet-clear-encryption.md)]

## <a name="additional-resources"></a>Дополнительные ресурсы

[!INCLUDE [additional resources snippet](~/includes/deploy/snippet-additional-resources.md)]

## <a name="next-steps"></a>Дополнительная информация
> [!div class="nextstepaction"]
> [Настройка непрерывного развертывания](bot-service-build-continuous-deployment.md)

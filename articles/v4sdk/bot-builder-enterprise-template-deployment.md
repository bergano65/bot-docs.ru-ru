---
title: Развертывание шаблона Enterprise Bot | Документация Майкрософт
description: Узнайте, как развернуть все вспомогательные ресурсы Azure для бота Enterprise Bot.
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 0f4c5e0db9dae86f81414ccd9bbb1e5de4dce624
ms.sourcegitcommit: b8bd66fa955217cc00b6650f5d591b2b73c3254b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/15/2018
ms.locfileid: "49326401"
---
# <a name="enterprise-bot-template---deploying-your-bot"></a>Развертывание Enterprise Bot с помощью шаблона

> [!NOTE]
> Приведенные здесь сведения относятся к пакету SDK версии 4. 

## <a name="prerequisites"></a>Предварительные требования

- Убедитесь, что установлен [диспетчер пакетов Node](https://nodejs.org/en/).

- Установите средства командной строки (CLI) Службы Azure Bot. Даже если вы ранее использовали эти средства, обязательно выполните эти шаги, чтобы убедиться, что у вас установлена их последняя версия.

```shell
npm install -g ludown luis-apis qnamaker botdispatch msbot luisgen chatdown
```

- [Установите](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-windows?view=azure-cli-latest) средства командной строки (CLI) Службы Azure Bot.

- Установите расширение AZ для Службы Bot.
```shell
az extension add -n botservice
```

## <a name="configuration"></a>Параметр Configuration

- Получите ключ разработки LUIS:
   - Ознакомьтесь [с этой статьей](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions), чтобы выбрать портал LUIS, соответствующий региону, в котором планируется развертывание. 
   - Выполнив вход, щелкните свое имя в правом верхнем углу.
   - Выберите пункт "Параметры" и запишите ключ разработки, который понадобится на следующем этапе.

## <a name="deployment"></a>Развертывание

>Если у вас несколько подписок Azure и нужно убедиться, что для развертывания выбрана правильная подписка, выполните следующие команды, прежде чем продолжить.

 Войдите в учетную запись Azure в браузере.
```shell
az login
az account list
az account set --subscription "YOUR_SUBSCRIPTION_NAME"
```

Для выполнения сквозных операций ботам на основе шаблона Enterprise Bot необходимы такие зависимости:
- Веб-приложение Azure
- учетная запись хранения Azure (расшифровки);
- Azure Application Insights (телеметрия);
- Azure Cosmos DB (состояние);
- Azure Cognitive Services — Интеллектуальная служба распознавания речи;
- Azure Cognitive Services — QnA Maker (включая Поиск Azure и Веб-приложение Azure);
- Azure Cognitive Services — Content Moderator (необязательный этап, выполняемый вручную).

Новый проект бота предусматривает возможность развертывания с помощью команды `msbot clone services`, которая автоматически развертывает все перечисленные выше службы в подписке Azure и обновляет BOT-файл проекта, добавляя в него все службы и ключи, необходимые для бесперебойного функционирования бота.

> После развертывания проверьте ценовые категории созданных служб и измените их в соответствии со своим сценарием.

В файле README.md созданного проекта содержится пример команды msbot clone services. В этой команде нужно передать имя создаваемого бота и общую версию, как показано ниже. Укажите ключ разработки LUIS, полученный на предыдущем этапе, и выберите нужное расположение центра обработки данных Azure (например, westus или westeurope).

> Убедитесь, что ключ разработки LUIS, полученный на предыдущем этапе, соответствует региону, который вы укажете далее.

```shell
msbot clone services --name "YOUR_BOT_NAME" --luisAuthoringKey "YOUR_AUTHORING_KEY" --folder "DeploymentScripts\msbotClone" --location "westus"
```

Средство msbot отобразит план развертывания, включая расположение и номер SKU. Проверьте эти данные, прежде чем продолжить.

![Подтверждение развертывания](./media/enterprise-template/EnterpriseBot-ConfirmDeployment.png)

>По завершении развертывания **обязательно** запишите полученный секрет BOT-файла. Он понадобится вам на следующих этапах.

- Обновите файл `appsettings.json`, добавив в него имя и секрет созданного BOT-файла.
- Выполните приведенную ниже команду, получите ключ инструментирования (InstrumentationKey) для экземпляра Application Insights и укажите этот ключ в файле `appsettings.json`.

`msbot list --bot YOURBOTFILE.bot --secret YOUR_BOT_SECRET`

        {
          "botFilePath": ".\\YOURBOTFILE.bot",
          "botFileSecret": "YOUR_BOT_SECRET",
          "ApplicationInsights": {
            "InstrumentationKey": "YOUR_INSTRUMENTATION_KEY"
          }
        }

## <a name="testing"></a>Тестирование

После завершения развертывания запустите проект бота в среде разработки и откройте эмулятор Bot Framework Emulator. В Emulator в меню File (Файл) щелкните пункт Open Bot (Открыть бот) и выберите BOT-файл в своем каталоге.

Затем введите ```hi``` и убедитесь, что все работает.

## <a name="deploy-to-azure"></a>Развернуть в Azure

Тестирование можно провести комплексно и локально. Когда вы будете готовы развернуть бот в Azure для дополнительного тестирования, опубликует исходный код с помощью приведенной ниже команды. Ее можно выполнять каждый раз, когда нужно отправить обновления исходного кода.

```shell
az bot publish -g YOUR_BOT_NAME -n YOUR_BOT_NAME --proj-file YOUR_BOT_NAME.csproj --sdk-version v4
```

## <a name="enabling-more-scenarios"></a>Активация дополнительных возможностей

Проект бота имеет дополнительные функциональные возможности, которые можно активировать.

### <a name="authentication"></a>Authentication

Чтобы включить аутентификацию, сначала на портале Azure укажите в настройках бота имя подключения для аутентификации, а затем выполните приведенные ниже действия. Дополнительные сведения см. в [документации](https://docs.microsoft.com/en-us/azure/bot-service/bot-builder-tutorial-authentication?view=azure-bot-service-3.0).

Зарегистрируйте `SignInDialog` в конструкторе MainDialog:
    
`AddDialog(new SignInDialog(_services.AuthConnectionName));`

Добавьте приведенную ниже строку в любое удобное место в коде, чтобы проверить простую последовательность входа.
    
`var signInResult = await dc.BeginAsync(SignInDialog.Name);`

### <a name="content-moderation"></a>Модерация содержимого

Функция модерации содержимого предназначена для обнаружения персональных данных и материалов для взрослых в отправляемых боту сообщениях. Чтобы активировать эту функцию, на портале Azure создайте новую службу Content Moderator. Запишите ключ подписки и регион, чтобы добавить эти сведения в BOT-файл. 

> Этот шаг будет автоматизирован в будущем.

Добавьте в нижней части метода service.AddBot<>() в классе Startup приведенный ниже код, чтобы включить модерацию содержимого в каждом отправляемом сообщении. Результаты модерации содержимого можно получить с помощью службы "Состояние бота". 
    
```
    // Content Moderation Middleware (analyzes incoming messages for inappropriate content including PII, profanity, etc.)
    var moderatorService = botConfig.Services.Where(s => s.Name == ContentModeratorMiddleware.ServiceName).FirstOrDefault();
    if (moderatorService != null)
    {
        var moderator = moderatorService as GenericService;
        var moderatorKey = moderator.Configuration["subscriptionKey"];
        var moderatorRegion = moderator.Configuration["region"];
        var moderatorMiddleware = new ContentModeratorMiddleware(moderatorKey, moderatorRegion);
        options.Middleware.Add(moderatorMiddleware);
    }
```
Чтобы просмотреть результат ПО промежуточного слоя, вызовите в стеке диалогов следующее:
```     
    var cm = dc.Context.TurnState.Get<Microsoft.CognitiveServices.ContentModerator.Models.Screen>(ContentModeratorMiddleware.TextModeratorResultKey);
```

## <a name="customize-your-bot"></a>Настройка бота

Убедившись, что развернутый бот готов к работе, настройте его в соответствии со своими потребностями. Сведения о настройке бота см. [здесь](bot-builder-enterprise-template-customize.md).

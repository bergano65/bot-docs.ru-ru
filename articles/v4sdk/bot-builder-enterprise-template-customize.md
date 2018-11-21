---
title: Настройка шаблона Enterprise Bot | Документация Майкрософт
description: Сведения о том, как настроить бот, созданный на основе шаблона Enterprise Bot
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 09/18/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: ea507bbdf916ff1955aea0db17b765791432f430
ms.sourcegitcommit: 8b7bdbcbb01054f6aeb80d4a65b29177b30e1c20
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 11/14/2018
ms.locfileid: "51645584"
---
# <a name="enterprise-bot-template---customize-your-bot"></a>Шаблон Enterprise Bot — настройка бота

> [!NOTE]
> Приведенные здесь сведения относятся к пакету SDK версии 4. 

## <a name="net"></a>.NET
[Развернув и протестировав шаблон Enterprise Bot](bot-builder-enterprise-template-deployment.md) и убедившись, что все его компоненты работают правильно, вы можете легко настроить бот в соответствии с собственным сценарием и потребностями. Задача шаблона — предоставить надежную базу, на основе которой можно создать собственную систему общения.

## <a name="project-structure"></a>Структура проекта

Структура папок бота показана ниже. Эта структура отражает наши рекомендации по структурированию проекта бота и обработке входящих сообщений.

    | - YourBot.bot         // The .bot file containing all of your Bot configuration including dependencies
    | - README.md           // README file containing links to documentation
    | - Program.cs          // Default Program.cs file
    | - Startup.cs          // Core Bot Initialisation including Bot Configuration LUIS, Dispatcher, etc. 
    | - <BOTNAME>State.cs   // The Root State class for your Bot
    | - appsettings.json    // References above .bot file for Configuration information. App Insights key
    | - CognitiveModels     
        | - LUIS            // .LU file containing base conversational intents (Greeting, Help, Cancel)
        | - QnA             // .LU file containing example QnA items
    | - DeploymentScripts   // msbot clone recipe for deployment
    | - Dialogs             // All Bot dialogs sit under this folder
        | - Main            // Root Dialog for all messages
            | - MainDialog.cs       // Dialog Logic
            | - MainResponses.cs    // Dialog responses
            | - Resources           // Adaptive Card JSON, Resource File
        | - Onboarding
            | - OnboardingDialog.cs       // Onboarding dialog Logic
            | - OnboardingResponses.cs    // Onboarding dialog responses
            | - OnboardingState.cs        // Localised dialog state
            | - Resources                 Resource File
        | - Cancel
        | - Escalate
        | - Signin
    | - Middleware          // Telemetry, Content Moderator
    | - ServiceClients      // SDK libraries, example GraphClient provided for Auth example
   
## <a name="update-introduction-message"></a>Изменение приветственного сообщения

Для приветственного сообщения используется [адаптивная карточка](https://www.adaptivecards.io). Чтобы настроить приветственное сообщение бота, найдите в папке Dialogs/Main/Resources файл JSON с именем ```Intro.json```. Для изменения адаптивной карточки в соответствии с требованиями к боту используйте [визуализатор адаптивных карточек](http://adaptivecards.io/visualizer).

## <a name="update-bot-responses"></a>Изменение ответов бота

Каждый диалог в проекте содержит набор ответов, которые хранятся во вспомогательных файлах ресурсов (файлы с расширением .resx). Эти файлы можно найти в папках Resources каждого диалога.

Ответы бота можно настроить, воспользовавшись редактором ресурсов Visual Studio, как показано ниже.

![Настройка ответов бота](media/enterprise-template/EnterpriseBot-CustomisingResponses.png)

Такой подход поддерживает ответы на нескольких языках с использованием стандартного метода для локализации файлов ресурсов. См. [дополнительные сведения](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/localization?view=aspnetcore-2.1).

## <a name="updating-your-cognitive-models"></a>Обновление когнитивных моделей

По умолчанию шаблон Enterprise Bot содержит две когнитивные модели: пример базы знаний QnAMaker с часто задаваемыми вопросами и модель LUIS для общих намерений (приветствие, справка, отмена и т. п.). Эти модели можно настроить в соответствии с вашими потребностями. Вы также можете добавить новые модели LUIS и базы знаний QnAMaker, чтобы расширить возможности бота.

### <a name="updating-an-existing-luis-model"></a>Изменение существующей модели LUIS
Чтобы обновить существующую модель LUIS в шаблоне Enterprise Bot, выполните следующие шаги:
1. Измените модель LUIS на [портале LUIS](http://luis.ai) или воспользуйтесь инструментами командной строки [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) и [LUIS](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/LUIS). 
2. Выполните следующую команду, чтобы обновить модель Dispatch в соответствии с внесенными изменениями (для правильной маршрутизации сообщений):
```shell
    dispatch refresh --bot "YOURBOT.bot" --secret YOURSECRET
```
3. Выполните следующую команду в корневой папке проекта для каждой обновленной модели, чтобы обновить соответствующие классы LuisGen: 
```shell
    luis export version --appId [LUIS_APP_ID] --versionId [LUIS_APP_VERSION] --authoringKey [YOUR_LUIS_AUTHORING_KEY] | luisgen --cs [CS_FILE_NAME] -o "\Dialogs\Shared\Resources"
```

### <a name="updating-an-existing-qnamaker-knowledge-base"></a>Обновление существующей базы знаний QnAMaker
Чтобы обновить существующую базу знаний QnAMaker, выполните следующие шаги:
1. Внесите изменения в базу знаний QnAMaker с помощью инструментов командной строки [LuDown](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Ludown) и [QnAMaker](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/QnAMaker) или [портала QnAMaker](https://qnamaker.ai).
2. Выполните следующую команду, чтобы обновить модель Dispatch в соответствии с внесенными изменениями (для правильной маршрутизации сообщений):
```shell
    dispatch refresh --bot "YOURBOT.bot" --secret YOURSECRET
```

### <a name="adding-a-new-luis-model"></a>Добавление новой модели LUIS

Если используемый сценарий требует добавления в проект новой модели LUIS, вам необходимо обновить конфигурацию бота и компонент Dispatch, чтобы добавить сведения о дополнительной модели. 
1. Создайте новую модель LUIS с помощью инструментов командной строки LuDown/LUIS или портала LUIS.
2. Выполните следующую команду, чтобы подключить новое приложение LUIS к файлу с расширением .bot:
```shell
    msbot connect luis --appId [LUIS_APP_ID] --authoringKey [LUIS_AUTHORING_KEY] --subscriptionKey [LUIS_SUBSCRIPTION_KEY] 
```
3. Добавьте новую модель LUIS в компонент Dispatch, выполнив следующую команду:
```shell
    dispatch add -t luis -id YOUR_LUIS_APPID -bot "YOURBOT.bot" -secret YOURSECRET
```
4. Обновите модель диспетчеризации, чтобы применить изменения модели LUIS, выполнив следующую команду:
```shell
    dispatch refresh -bot "YOURBOT.bot" -secret YOURSECRET
```

### <a name="adding-an-additional-qnamaker-knowledgebase"></a>Добавление дополнительной базы знаний QnAMaker

Возможно, вам потребуется добавить в бот дополнительную базу знаний QnAMaker. Ниже описано, как это сделать.

1. Создайте базу знаний QnAMaker из JSON-файла с помощью приведенной ниже команды в каталоге помощника.
```shell
qnamaker create kb --in <KB.json> --msbot | msbot connect qna --stdin --bot "YOURBOT.bot" --secret YOURSECRET
```
2. Выполните приведенную ниже команду, чтобы обновить модель Dispatch в соответствии с внесенными изменениями.
```shell
dispatch refresh --bot "YOURBOT.bot" --secret YOURSECRET
```
3. Обновите строго типизированный класс Dispatch таким образом, чтобы он отражал новый источник QnA.
```shell
msbot get dispatch --bot "YOURBOT.bot" | luis export version --stdin > dispatch.json
luisgen dispatch.json -cs Dispatch -o Dialogs\Shared
```
4.  Обновите файл `Dialogs\Main\MainDialog.cs` таким образом, чтобы он включал соответствующее намерение Dispatch для нового источника QnA, как в приведенном ниже примере.

Теперь вы можете использовать несколько источников QnA в своем боте.

## <a name="adding-a-new-dialog"></a>Добавление нового диалога

Чтобы добавить в бот новый диалог, сначала необходимо создать подпапку в папке Dialogs и убедиться, что этот класс наследует от `EnterpriseDialog`. Затем нужно настроить инфраструктуру диалога. Диалог Onboarding демонстрирует простой пример, который можно использовать для справки. Ниже приведен фрагмент кода и перечень шагов.

- Добавьте каскадный диалог в конструктор.
- Определите шаги каскада.
- Создайте шаги каскада.
- Вызовите AddDialog, чтобы добавить каскад.
- Вызовите AddDialog, чтобы добавить все запросы, которые используются в каскаде.
- В качестве значения InitialDialogId укажите первый диалог, который должен запустить компонент.

```
InitialDialogId = nameof(OnboardingDialog);

var onboarding = new WaterfallStep[]
{
    AskForName,
    AskForEmail,
    AskForLocation,
    FinishOnboardingDialog,
};

AddDialog(new WaterfallDialog(InitialDialogId, onboarding));
AddDialog(new TextPrompt(NamePrompt));
AddDialog(new TextPrompt(EmailPrompt));
AddDialog(new TextPrompt(LocationPrompt));
```

Затем нужно создать экземпляр TemplateManager для обработки ответов. Создайте класс, наследующий от TemplateManager. Пример можно найти в файле OnboardingResponses.cs, и ниже приведен фрагмент кода.

```
public const string _namePrompt = "namePrompt";
public const string _haveName = "haveName";
public const string _emailPrompt = "emailPrompt";
      
private static LanguageTemplateDictionary _responseTemplates = new LanguageTemplateDictionary
{
    ["default"] = new TemplateIdMap
    {
        {
            _namePrompt,
            (context, data) => OnboardingStrings.NAME_PROMPT
        },
        {
            _haveName,
            (context, data) => string.Format(OnboardingStrings.HAVE_NAME, data.name)
        },
        {
            _emailPrompt,
            (context, data) => OnboardingStrings.EMAIL_PROMPT
        },
```

Чтобы отображать ответы, можно использовать экземпляр TemplateManager для доступа к этим ответам с помощью методов `ReplyWith` или `RenderTemplate` экземпляров Prompt. Примеры приведены ниже.

```
Prompt = await _responder.RenderTemplate(sc.Context, "en", OnboardingResponses._namePrompt),
await _responder.ReplyWith(sc.Context, OnboardingResponses._haveName, new { name });
```

Последним элементом инфраструктуры диалога является создание класса State, область видимости которого ограничена диалогом. Создайте класс, наследующий от `DialogState`.

Созданный диалог нужно добавить его в компонент `MainDialog` с помощью метода `AddDialog`. Для использования нового диалога вызовите метод `dc.BeginDialogAsync()` из метода `RouteAsync`, активировав его с помощью соответствующего намерения LUIS.

## <a name="conversational-insights-using-powerbi-dashboard-and-application-insights"></a>Аналитические сведения об общении с использованием панели мониторинга PowerBI и Application Insights
- Чтобы начать получать аналитические сведения об общении, ознакомьтесь со [статьей о настройке аналитики общения с помощью панели мониторинга PowerBI](bot-builder-enterprise-template-powerbi.md).


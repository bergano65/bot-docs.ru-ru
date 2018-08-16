---
title: Настройка непрерывного развертывания для службы Bot | Документация Майкрософт
description: Узнайте, как настроить непрерывное развертывание из системы управления версиями для службы Bot.
keywords: continuous deployment, publish, deploy, azure portal
author: v-ducvo
ms.author: v-ducvo
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 03/08/2018
ms.openlocfilehash: 596d264c4df72959c71ab353e5038175fc2bed31
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305095"
---
# <a name="set-up-continuous-deployment"></a>Непрерывное развертывание с использованием GIT в службе приложений Azure

Непрерывное развертывание позволяет разрабатывать бот локально. Непрерывное развертывание удобно, если ваш бот записывается после изменения в систему управления версиями, например **GitHub** или **Visual Studio Team Services**. После записи изменений в исходном репозитории они будут автоматически развернуты в Azure.

> [!NOTE]
> После настройки непрерывного развертывания [интерактивный редактор кода](bot-service-build-online-code-editor.md) становится *доступным только для чтения*.

В этом разделе показано, как настроить непрерывное развертывание для **GitHub** и **Visual Studio Team Services**.

## <a name="continuous-deployment-using-github"></a>Непрерывное развертывание с помощью GitHub

Чтобы настроить непрерывное развертывание с помощью GitHub, сделайте следующее:

1. [Создайте вилку](https://help.github.com/articles/fork-a-repo/) в репозитории GitHub, содержащем код, который вы хотите развернуть в Azure.
2. На портале Azure перейдите в колонку **Build** (Сборка) своего бота и щелкните **Configure continuous deployment** (Настройка непрерывного развертывания). 
3. Щелкните **Настройка**.
   
   ![Настройка непрерывного развертывания](~/media/azure-bot-build/continuous-deployment-setup.png)

4. Нажмите кнопку **Выбор источника** и выберите **GitHub**.

   ![Выбор GitHub](~/media/azure-bot-build/continuous-deployment-setup-github.png)

5. Щелкните **Авторизация**, нажмите кнопку **Авторизовать** и следуйте инструкциям, чтобы предоставить Azure право доступа к своей учетной записи GitHub.

Настройка непрерывного развертывания с помощью GitHub завершена. При фиксации изменений они будут автоматически развернуты в Azure.

## <a name="continuous-deployment-using-visual-studio"></a>Непрерывное развертывание с помощью Visual Studio

1. В колонке **Build** (Сборка) своего бота щелкните **Configure continuous deployment** (Настройки непрерывного развертывания). 
2. Щелкните **Настройка**.
   
   ![Настройка непрерывного развертывания](~/media/azure-bot-build/continuous-deployment-setup.png)

3. Щелкните **Выбор источника** и выберите **Visual Studio Team Services**.

   ![Выбор Visual Studio Team Services](~/media/azure-bot-build/continuous-deployment-setup-vs.png)

4. Щелкните **Выбор учетной записи** и выберите учетную запись.

> [!NOTE]
> Если вы не видите свою учетную запись, убедитесь, что она связана с вашей подпиской Azure.
> Дополнительные сведения см. в разделе [Связывание учетной записи VSTS с подпиской Azure](https://github.com/projectkudu/kudu/wiki/Setting-up-a-VSTS-account-so-it-can-deploy-to-a-Web-App#linking-your-vsts-account-to-your-azure-subscription).

5. Щелкните **Выбор проекта** и выберите проект.

> [!NOTE]
> Поддерживаются только проекты VSTS Git.

6. Щелкните **Выбор ветви** и выберите ветвь.
7. Нажмите кнопку **ОК**, чтобы завершить настройку.

   ![Настройка Visual Studio](~/media/azure-bot-build/continuous-deployment-setup-vs-configuration.png)

Настройка непрерывного развертывания с помощью Visual Studio Team Services завершена. При фиксации изменений они будут автоматически развернуты в Azure.

## <a name="disable-continuous-deployment"></a>Отключение непрерывного развертывания

Хотя ваш бот и настроен для непрерывного развертывания, не используйте интерактивный редактор кода для внесения изменений в бот. Если вы хотите использовать интерактивный редактор кода, можно временно отключить непрерывное развертывание.

Чтобы отключить непрерывное развертывание, сделайте следующее:

1. В колонке **Build** (Сборка) своего бота щелкните **Configure continuous deployment** (Настройки непрерывного развертывания). 
2. Нажмите кнопку **Отключить** для отключения непрерывного развертывания. Чтобы снова включить непрерывное развертывание, повторите действия, описанные в соответствующем разделе выше.

## <a name="next-steps"></a>Дополнительная информация
Теперь, когда ваш бот настроен для непрерывного развертывания, протестируйте свой код, используя интерактивный веб-чат.

> [!div class="nextstepaction"]
> [Тестирование в веб-чате](bot-service-manage-test-webchat.md)

---
title: Реализация приветственного диалога | Документация Майкрософт
description: Используйте диалог для приветствия пользователя, когда он присоединяется к беседе.
keywords: greeting, dialogs, conversation flow, dialog set
author: JonathanFingold
ms.author: v-jofing
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 82e6273b8d6dc984e29bef891f3e8f67b1c53eed
ms.sourcegitcommit: ea64a56acfabc6a9c1576ebf9f17ac81e7e2a6b7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/24/2019
ms.locfileid: "66215417"
---
# <a name="implement-a-greeting-dialog"></a>Реализация приветственного диалога

[!INCLUDE [pre-release-label](../includes/pre-release-label.md)]

Вы можете создать диалог для приветствия пользователя, который присоединяется к беседе.

См. подробнее об [отправке приветственных сообщений пользователям][send-welcome].

## <a name="prerequisites"></a>Предварительные требования

- Опыт работы с [управлением состоянием][concept-state], [библиотеками диалогов][concept-dialogs], [управлением диалогами][simple-flow] и [сбором введенных пользователем данных с использованием приглашения к диалогу][prompting].
- Копия примера ??? на языке [**CSharp**][cs-sample] или [**JavaScript**][js-sample].

## <a name="task-as-in-to-do-x-do-these-things"></a>\<Задача> [чтобы сделать X, выполните следующее]

<!--The key lines of code for this task.
    here are the cool lines that do that.
    just the few lines of implementation without setup.
-->

## <a name="about-the-sample-code"></a>Сведения о примере кода

<!--setup & implementation & discussion of the sample code-->

- Дополнительные необходимые пакеты (AI.Luis, Dialog и т. д.).

<!--Any other key elements to get the code to work.
    Include setup for only the bits critical to the task at hand.
    don't go over all the code in the sample.
-->

## <a name="to-test-the-bot"></a>Тестирование бота

1. Установите [Bot Framework Emulator](https://aka.ms/bot-framework-emulator-readme), если вы этого еще не сделали.
1. Выполните этот пример на локальном компьютере.
1. Запустите эмулятор, подключитесь к боту и отправьте несколько сообщений, как показано ниже.

TODO: сделать новый снимок экрана.

<!--![test dialog prompt sample](~/media/emulator-v4/test-dialog-prompt.png)-->

## <a name="discussion-optional"></a>Обсуждение [необязательно]

<!--Might be short and descriptive or include additional code for scenarios not covered in the samples repo
-->

## <a name="addition-information"></a>Дополнительная информация

<!--include cross-linking other articles about the same sample.-->

- Ссылка на целевую страницу со сведениями о боте и общим примером (для статей о базовом боте).

## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Обработка прерываний диалога пользователем](bot-builder-howto-handle-user-interrupt.md)

<!-- Footnote-style links -->

[concept-basics]: bot-builder-basics.md
[concept-state]: bot-builder-concept-state.md
[concept-dialogs]: bot-builder-concept-dialog.md

[send-welcome]: bot-builder-send-welcome-message.md

[simple-flow]: bot-builder-dialog-manage-conversation-flow.md
[prompting]: bot-builder-prompts.md
[component-dialogs]: bot-builder-compositcontrol.md

[cs-sample]: ???
[js-sample]: ???

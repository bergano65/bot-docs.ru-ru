---
title: Сохранение бота Conversation Designer | Документация Майкрософт
description: Сведения о сохранении и обучении модели распознавания речи и первоклассное распознавание речи в боте Conversation Designer.
author: vkannan
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ROBOTS: NoIndex, NoFollow
ms.openlocfilehash: 3a911158c379f879c0be604fb5e8ba30ab22ee44
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39306222"
---
# <a name="saving-your-conversation-designer-bot"></a>Сохранение бота Conversation Designer
> [!IMPORTANT]
> Conversation Designer пока недоступен для всех клиентов. Дополнительные сведения о доступности Conversation Designer будут опубликованы позже в этом году.

При работе в Conversation Designer вся работа кэшируется в памяти браузера. Чтобы зафиксировать изменения, нажмите кнопку **Сохранить**, расположенную в верхнем левом углу меню навигации. Чтобы избежать потери работы, рекомендуется часто сохранять свою работу. Помимо нажатия кнопки **Сохранить** можно также сохранить работу с помощью короткой комбинации клавиш **CTRL+S**.

## <a name="trains-luis-and-primes-speech-recognition"></a>Обучение LUIS и первоклассное распознавание речи

Нажатие кнопки **Сохранить** сохранит изменения в боте и выполнит несколько дополнительных задач. В отличие от короткой комбинации клавиш, нажатие кнопки **Сохранить** также укажет Conversation Designer на необходимость выполнить следующие задачи.

- Обучение любым новым намерениям и сущностям LUIS для бота и публикация модели LUIS локально в службе Azure Bot (при необходимости). Эти намерения могут быть добавлены в Conversation Designer или в соответствующее приложение [LUIS](https://luis.ai) бота.
- Обновление времени выполнения общения для использования новой модели LUIS.
- Подготовка распознавания речи путем подготовки и отправки примерных высказываний в Microsoft Cognitive Services, что значительно улучшает точность распознавания речи для этого бота.

## <a name="next-step"></a>Дальнейшие действия
> [!div class="nextstepaction"]
> [Test bot](conversation-designer-debug-bot.md) (Тестирование бота)

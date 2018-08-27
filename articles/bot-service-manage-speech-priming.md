---
title: Настройка подготовки речи | Документация Майкрософт
description: Сведения о настройке подготовки речи для службы ботов с помощью портала Azure.
keywords: speech priming, speech recognition, LUIS
author: v-royhar
ms.author: v-royhar
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 2e0b85af834bc92a9da8c9f9be2794da88c2b3bc
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39301193"
---
# <a name="configure-speech-priming"></a>Настройка подготовки речи

Подготовка речи улучшает возможность распознавания произнесенных слов и фраз, стандартно используемых в боте. Для ботов с поддержкой речевых функций, использующих каналы "Веб-чат" и Кортана, подготовка речи использует примеры, указанные в приложениях "Распознавание речи" (LUIS), для повышения точности распознавания речи при применении важных слов.

В бот уже может быть интегрировано приложение LUIS. Вы также можете создать приложение LUIS для сопоставления с ботом для подготовки речи. Приложение LUIS содержит примеры ожидаемой речи, произносимой пользователями для бота. Важные слова, которые бот должен распознавать, следует пометить как сущности. Например, в боте для игры в шахматы требуется убедиться, что когда пользователь произносит фразу Move knight, она не интерпретируется как Move night. Приложение LUIS должно включать примеры, в которых слово knight отмечено как сущность.

> [!NOTE]
> Чтобы использовать подготовку речи с помощью канала "Веб-чат", необходимо использовать службу "Распознавание речи Bing". Пояснения по поводу использования этой службы см. в статье [Как включить речь в канале "Веб-чат"](~/bot-service-channel-connect-webchat-speech.md).

> [!IMPORTANT]
> Подготовка речи применяется только к ботам, настроенным для канала Кортаны или "Веб-чат".

## <a name="change-the-list-of-luis-apps-your-bot-uses"></a>Изменение списка приложений LUIS, используемых в боте

Чтобы изменить список приложений LUIS, используемых службой "Распознавание речи Bing" с помощью бота, сделайте следующее:

1. В колонке службы ботов щелкните **Speech priming** (Подготовка речи). Появится список доступных приложений LUIS.
2. Выберите приложения LUIS, которые должна использовать служба "Распознавание речи Bing".
 
    a. Чтобы выбрать приложение LUIS в списке, наведите указатель мыши на модель LUIS. Появится флажок. Установите его.
     
    b. Чтобы выбрать приложение LUIS, которого нет в списке, прокрутите список вниз и введите GUID приложения LUIS в текстовое поле.
     
3. Нажмите кнопку **Сохранить**, чтобы сохранить список приложений LUIS, связанных со службой "Распознавание речи Bing", для своего бота.

![Панель подготовки речи](~/media/bot-service-manage-speech-priming/speech-priming.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

- Дополнительные сведения о включении распознавания речи в веб-чате см. в [этой статье](~/bot-service-channel-connect-webchat-speech.md).
- Дополнительные сведения о подготовке речи см. в статье [Speech Support in Bot Framework — Webchat to Directline, to Cortana](https://blog.botframework.com/2017/06/26/Speech-To-Text/) (Поддержка речи в Bot Framework: компоненты "Веб-чат", Directline и Кортана).
- Дополнительные сведения о приложениях LUIS см. в статье [Language Understanding (LUIS)](https://www.luis.ai) (Распознавание речи (LUIS)).
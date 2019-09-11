---
title: Включение распознавания речи в веб-чате | Документация Майкрософт
description: Узнайте, как включить распознавание речи в элементе управления "Веб-чат" для бота, подключенного к каналу "Веб-чат".
keywords: speech, web chat, voice, microphone, audio
author: DeniseMak
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: b83dff7969c58451e5752938f74b682b2163c49d
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70298203"
---
# <a name="enable-speech-in-web-chat"></a>Включение распознавания речи в веб-чате
Вы можете включить голосовой интерфейс в элементе управления "Веб-чат". Пользователи взаимодействуют с голосовым интерфейсом, используя микрофон в элементе управления "Веб-чат".

![Пример распознавания речи в веб-чате](~/media/bot-service-channel-webchat/webchat-sample-speech.png)

Если пользователь печатает, вместо того чтобы говорить, веб-чат отключает функции распознавания речи и бот предоставляет только текстовый ответ, вместо того чтобы говорить вслух. Чтобы снова включить устный ответ, пользователь может при следующем ответе боту использовать микрофон. Если микрофон принимает входные данные, его значок отображается как темный или заполненный. Если он выделен серым, значит пользователь щелкнул, чтобы отключить его.

## <a name="prerequisites"></a>Предварительные требования

  Перед запуском образца необходимо иметь секрет Direct Line или токен для бота, который вы хотите запустить с помощью элемента управления "Веб-чат". 
  * Сведения о том, как получить секрет Direct Line, связанный с вашим ботом, см. в статье [Connect a bot to Direct Line](bot-service-channel-connect-directline.md) (Подключение бота к Direct Line).
  * Сведения об обмене секрета на токен см. в статье [Authentication](rest-api/bot-framework-rest-direct-line-3-0-authentication.md) (Проверка подлинности).

## <a name="customizing-web-chat-for-speech"></a>Настройка веб-чата для распознавания речи
Чтобы включить функцию распознавания речи в веб-чате, необходимо настроить код JavaScript, вызывающий элемент управления "Веб-чат". Вы можете проверить голосовое управление веб-чатом локально, выполнив следующие действия.

1. Скачайте [пример index.html](https://aka.ms/web-chat-speech-sample). <!-- this aka.ms link needs to be updated if the sample location changes -->
2. Отредактируйте код `index.html` в соответствии с типом поддержки речи, который вы хотите добавить. Типы речевых реализаций описаны в разделе [Enable speech services](#enable-speech-services) (Включение служб речи). 
3. Запустите веб-сервер. Один из способов сделать это — использовать `npm http-server` в командной строке Node.js.

   * Чтобы установить `http-server` глобально для запуска из командной строки, выполните следующую команду:

     ```
     npm install http-server -g
     ```

   * Чтобы запустить веб-сервер с помощью порта 8000, из каталога, содержащего `index.html`, выполните следующую команду:

     ```
     http-server -p 8000
     ```
4. В браузере перейдите к странице `http://localhost:8000/samples?parameters`. Например, `http://localhost:8000/samples?s=YOURDIRECTLINESECRET` вызывает бот с помощью секрета Direct Line. В следующей таблице описаны параметры, которые можно задать в строке запроса:

   | Параметр | ОПИСАНИЕ |
   |-----------|-------------|
   | s | Секрет Direct Line. Сведения о том, как получить секрет Direct Line, см. в статье [Connect a bot to Direct Line](bot-service-channel-connect-directline.md) (Подключение бота к Direct Line). |
   | t | Токен Direct Line. Сведения о создании этого токена см. в статье [Authentication](rest-api/bot-framework-rest-direct-line-3-0-authentication.md) (Проверка подлинности). |
   | Домен | Необязательный элемент. URL-адрес альтернативной конечной точки Direct Line.  |
   | webSocket | Необязательный элемент. Установите значение true, чтобы использовать WebSocket для получения сообщений. Значение по умолчанию — `false`. |
   | userid | Необязательный элемент. Идентификатор пользователя бота.  |
   | Имя пользователя | Необязательный элемент. Имя пользователя бота.  |
   | botid | Необязательный элемент. Идентификатор бота. |
   | botname | Необязательный элемент. Имя бота. |


## <a name="enable-speech-services"></a>Включение служб речи
Настройка позволяет добавить функцию распознавания речи любым из следующих способов.

* **Распознавание речи, предоставленное браузером**. Используется функция распознавания речи, встроенная в браузер. В настоящее время эта функция доступна только в браузере Chrome.
<!--* **Use Bing Speech service** - You can use the Bing Speech service to provide speech recognition and synthesis. This way of access speech functionality is supported by a variety of browsers. In this case, the processing is done on a server instead of on the browser.-->
* **Создание пользовательской службы распознавания речи**. Вы можете создать собственные компоненты для распознавания речи и ее синтеза.

### <a name="browser-provided-speech"></a>Распознавание речи, предоставленное браузером

В следующем экземпляре кода реализованы компоненты распознавания речи и ее синтеза, которые входят в состав браузера. Такой способ добавления распознавания речи поддерживается не всеми браузерами. 

> [!NOTE] 
> Google Chrome поддерживает распознаватель речи браузера. Тем не менее Chrome может блокировать микрофон в следующих случаях.
> * Если URL-адрес страницы, содержащей веб-чат, начинается с `http://` вместо `https://`.
> * Если URL-адрес является локальным файлом, который использует протокол `file://` вместо `http://localhost:8000`.

[!code-js[Specify speech options to use in-browser speech (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BrowserSpeech)]

<!--### Bing Speech service

The following code instantiates speech recognizer and speech synthesis components that use the Bing Speech service. The recognition and generation of speech is performed on the server. This mechanism is supported in multiple browsers. 

> [!TIP]
> You can use speech recognition priming to improve your bot's speech recognition accuracy if you use the Bing Speech service. For more information, check out the [Speech Support in Bot Framework](https://blog.botframework.com/2017/06/26/Speech-To-Text) blog post.

[!code-js[Specify speech options to use the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#BingSpeech)]

#### Use the Bing Speech service with a token

You also have the option to enable Cognitive Services speech recognition using a token. The token is generated in a secure back end using your API key.

The following example code shows how the token fetch is done from a secure back end to avoid exposing the API key.

[!code-js[Fetch a token to use with the Bing Speech API (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#FetchToken)]
-->
### <a name="custom-speech-service"></a>Пользовательская служба распознавания речи

Вы также можете предоставить собственные компоненты распознавания речи, которые реализуют ISpeechRecognizer, или синтеза речи, который реализует ISpeechSynthesis. 

[!code-js[Fetch a token to use with a custom speech service (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#CustomSpeechService)]

### <a name="pass-the-speech-options-to-web-chat"></a>Передача параметров речи в веб-чат

Приведенный ниже код передает параметры речи в элемент управления "Веб-чат":

[!code-js[Pass speech options to Web Chat (JavaScript)](./includes/code/bot-service-channel-connect-webchat-speech.js#PassSpeechOptionsToWebChat)]

## <a name="next-steps"></a>Дополнительная информация
Теперь, когда вы можете включить голосовое взаимодействие с веб-чатом, узнайте, как ваш бот создает голосовые сообщения и изменяет состояние микрофона.
* [Добавление речи в сообщение (C#)](dotnet/bot-builder-dotnet-text-to-speech.md)
* [Добавление речи в сообщение (Node.js)](nodejs/bot-builder-nodejs-text-to-speech.md)

## <a name="additional-resources"></a>Дополнительные ресурсы

* Вы можете [загрузить исходный код](https://github.com/Microsoft/BotFramework-WebChat) для элемента управления "Веб-чат" с сайта GitHub.
* Дополнительную информацию об API распознавания речи Bing см. в [этой статье](https://docs.microsoft.com/azure/cognitive-services/speech/home).


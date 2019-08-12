---
title: Применение WebChat с расширением Службы приложений Direct Line
titleSuffix: Bot Service
description: Применение WebChat с расширением Службы приложений Direct Line
services: bot-service
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.author: kamrani
ms.date: 07/25/2019
ms.openlocfilehash: 5e74627530e77f4ae5f1f8ec1ae36dc5b07959a6
ms.sourcegitcommit: a1eaa44f182a7210197bd793250907df00e9edab
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 08/03/2019
ms.locfileid: "68757746"
---
## <a name="use-webchat-with-the-direct-line-app-service-extension"></a>Применение WebChat с расширением Службы приложений Direct Line

В этой статье описано, как применить WebChat с расширением Службы приложений Direct Line.

### <a name="get-your-direct-line-secret"></a>Получение секрета Direct Line

Прежде всего вам нужно узнать секрет Direct Line. Для этого выполните инструкции, приведенные в статье [о подключении бота к Direct Line](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-directline?view=azure-bot-service-4.0).

### <a name="get-the-preview-version-of-directlinejs"></a>Получение предварительной версии DirectLineJS
Предварительную версию DirectLineJS можно найти здесь: https://github.com/Jeffders/DirectLineAppServiceExtensionPreview/tree/master/libraries

### <a name="integrate-webchat-client"></a>Интеграция клиента WebChat

В целом сохраняется тот же подход, что описан выше. Единственное отличие заключается в том, что была создана новая версия **WebChat**, которая поддерживает двусторонний трафик **WebSocket** и подключается не к https://directline.botframework.com/, а напрямую к размещенному боту.
URL-адрес Direct Line для бота будет иметь вид `https://<your_app_service>.azurewebsites.net/.bot/`, где расширение `/.bot/` является **конечной точкой** Direct Line в Службе приложений.
Даже если вы настроите собственное доменное имя, к нему необходимо добавить путь `/.bot/` для обращения к интерфейсам REST API Direct Line.

1. Обменяйте секрет на маркер, выполнив инструкции из [статьи об аутентификации](https://docs.microsoft.com/en-us/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-authentication?view=azure-bot-service-4.0). Но вместо расположения `https://directline.botframework.com/v3/directline/tokens/generate` вы будете использовать для получения маркера непосредственно расширение Службы приложений Direct Line из расположения `https://<your_app_service>.azurewebsites.net/.bot/v3/directline/tokens/generate`.  

1. Получив маркер, вы можете внести следующие изменения в веб-страницу, на которой используется WebChat:

```html
<!DOCTYPE html>
<html lang="en-US">
<head>
    <title>Direct Line Streaming Sample</title>
    <script src="~/directLine.js"></script>
    <script src="https://cdn.botframework.com/botframework-webchat/master/webchat.js"></script>
    <style>
        html, body {
            height: 100%
        }

        body {
            margin: 0
        }

        #webchat,

        #webchat > * {
            height: 100%;
            width: 100%;
        }
    </style>
</head>

<body>
    <div id="webchat" role="main"></div>
    <script>
        const activityMiddleware = () => next => card => {
            if (card.activity.type === 'trace') {
                // Return false means, don't render the trace activities
                return () => false;
            } else {
                return children => next(card)(children);
            }
        };


        var dl = new DirectLine.DirectLine({
            secret: '<your token>',
            domain: 'https://<your_site>.azurewebsites.net/.bot/v3/directline',
            webSocket: true,
            conversationId: '<your conversation id>'
        });
        window.WebChat.renderWebChat({
            activityMiddleware,
            directLine: dl,
            userID: '<your generated user id>'
        }, document.getElementById('webchat'));
    </script>
</body>
</html>

```

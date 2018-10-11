---
title: Создание бота с помощью пакета SDK Bot Builder для Python | Документация Майкрософт
description: Быстрое создание бота с помощью пакета SDK Bot Builder для Python.
keywords: Bot Builder SDK, create a bot, quickstart, python, getting started
author: jonathanfingold
ms.author: jonathanfingold
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 08/30/2018
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6b63fe2780c51e57ee16c5e3dba5a83f46566157
ms.sourcegitcommit: 3bf3dbb1a440b3d83e58499c6a2ac116fe04b2f6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/23/2018
ms.locfileid: "46707286"
---
# <a name="create-a-bot-with-the-bot-builder-sdk-for-python"></a>Создание бота с помощью пакета SDK Bot Builder для Python

>[!NOTE] 
> Пакет SDK для Python доступен в режиме **предварительной версии**. Чтобы получить дополнительные сведения, перейдите в [репозиторий для Python](https://github.com/Microsoft/botbuilder-python) на сайте GitHub. 

В этом кратком руководстве описывается процесс создания бота и его тестирование с помощью эмулятора Bot Framework. 

## <a name="pre-requisite"></a>Предварительные требования
- [Python 3.6.4](https://www.python.org/downloads/); 
- [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases).

## <a name="create-a-bot"></a>Создание бота
В файле main.py импортируйте следующие стандартные модули:

```python
import http.server
import json
import asyncio
```

Также импортируйте следующие модули SDK:
```python
from botbuilder.schema import (Activity, ActivityTypes)
from botframework.connector import ConnectorClient
from botframework.connector.auth import (MicrosoftAppCredentials,
                                         JwtTokenValidation, SimpleCredentialProvider)
```
Затем добавьте следующий код для создания бота с помощью ConnectorClient:
```python
APP_ID = ''
APP_PASSWORD = ''


class BotRequestHandler(http.server.BaseHTTPRequestHandler):

    @staticmethod
    def __create_reply_activity(request_activity, text):
        return Activity(
            type=ActivityTypes.message,
            channel_id=request_activity.channel_id,
            conversation=request_activity.conversation,
            recipient=request_activity.from_property,
            from_property=request_activity.recipient,
            text=text,
            service_url=request_activity.service_url)

    def __handle_conversation_update_activity(self, activity):
        self.send_response(202)
        self.end_headers()
        if activity.members_added[0].id != activity.recipient.id:
            credentials = MicrosoftAppCredentials(APP_ID, APP_PASSWORD)
            reply = BotRequestHandler.__create_reply_activity(activity, 'Hello and welcome to the echo bot!')
            connector = ConnectorClient(credentials, base_url=reply.service_url)
            connector.conversations.send_to_conversation(reply.conversation.id, reply)

    def __handle_message_activity(self, activity):
        self.send_response(200)
        self.end_headers()
        credentials = MicrosoftAppCredentials(APP_ID, APP_PASSWORD)
        connector = ConnectorClient(credentials, base_url=activity.service_url)
        reply = BotRequestHandler.__create_reply_activity(activity, 'You said: %s' % activity.text)
        connector.conversations.send_to_conversation(reply.conversation.id, reply)

    def __handle_authentication(self, activity):
        credential_provider = SimpleCredentialProvider(APP_ID, APP_PASSWORD)
        loop = asyncio.new_event_loop()
        try:
            loop.run_until_complete(JwtTokenValidation.authenticate_request(
                activity, self.headers.get("Authorization"), credential_provider))
            return True
        except Exception as ex:
            self.send_response(401, ex)
            self.end_headers()
            return False
        finally:
            loop.close()

    def __unhandled_activity(self):
        self.send_response(404)
        self.end_headers()

    def do_POST(self):
        body = self.rfile.read(int(self.headers['Content-Length']))
        data = json.loads(str(body, 'utf-8'))
        activity = Activity.deserialize(data)

        if not self.__handle_authentication(activity):
            return

        if activity.type == ActivityTypes.conversation_update.value:
            self.__handle_conversation_update_activity(activity)
        elif activity.type == ActivityTypes.message.value:
            self.__handle_message_activity(activity)
        else:
            self.__unhandled_activity()


try:
    SERVER = http.server.HTTPServer(('localhost', 9000), BotRequestHandler)
    print('Started http server on localhost:9000')
    SERVER.serve_forever()
except KeyboardInterrupt:
    print('^C received, shutting down server')
    SERVER.socket.close()
```


Сохраните файл main.py. Чтобы запустить пример в Windows, в окне командной строки введите следующую команду:
```
python main.py
```
В окне терминала на локальном компьютере должно отобразиться сообщение "Started http server on localhost:9000".

## <a name="start-the-emulator-and-connect-your-bot"></a>Запуск эмулятора и подключение бота

После этого запустите эмулятор и подключитесь к боту в эмуляторе.

1. Щелкните ссылку **Open Bot** (Открыть бот) на вкладке приветствия в эмуляторе. 
2. Выберите файл с расширением .bot, расположенный в каталоге созданного проекта.

## <a name="interact-with-your-bot"></a>Взаимодействие с ботом

Отправьте сообщение боту и получите от него сообщение в ответ.
![Работающий эмулятор](../media/emulator-v4/emulator-running.png)


## <a name="next-steps"></a>Дополнительная информация

> [!div class="nextstepaction"]
> [Основные понятия о ботах](../v4sdk/bot-builder-basics.md)

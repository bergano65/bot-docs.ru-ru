---
title: Общие сведения об интерфейсе командной строки (CLI) Azure Bot Framework — Служба Azure Bot
description: Сведения об интерфейсе командной строки (CLI) в Bot Framework.
keywords: Интерфейс командной строки Bot Framework, Bot Framework CLI
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 11/01/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 8b885b19ed22c4d91163b59abe4e253018531b59
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791431"
---
<!--TODO:
- [?] Add to TOC: Reference/Bot Framework CLI/Reference
- [?] Add other topics to the same node for each of the command groups
-->
# <a name="bot-framework-cli-overview"></a>Общие сведения о Bot Framework CLI

[!INCLUDE[applies-to](../includes/applies-to.md)]

Интерфейс командной строки (CLI) Bot Framework представляет собой кроссплатформенное средство, с помощью которого можно управлять ботами и связанными с ними службами. Он заменяет коллекцию старых автономных средств CLI, объединяя их в единый инструмент. 

## <a name="prerequisites"></a>предварительные требования

* [Node.js](https://nodejs.org/) версии 10.14.1 или более поздней.

## <a name="installation"></a>Установка

Интерфейс командной строки Bot Framework устанавливается из командной строки.

~~~cmd
npm i -g @microsoft/botframework-cli
~~~

## <a name="available-commands"></a>Доступные команды

В настоящее время поддерживаются следующие команды.

| Ранее использовавшийся инструмент | Набор команд Bot Framework | Description |
| :--- | :--- | :--- |
| ChatDown | [`bf chatdown`](bf-cli-reference.md#bf-chatdown) | Команды для работы с файлами диалога чата ( **.chat**). |
| Нет | [`bf config`](bf-cli-reference.md#bf-config) | Настраивает разные параметры в интерфейсе командной строки. |
| LuDown, LuisGen | [`bf luis`](bf-cli-reference.md#bf-luis) | Команды для работы с файлами ресурсов LUIS и управления моделями LUIS. |
| QnA Maker | [`bf qnamaker`](bf-cli-reference.md#bf-qnamaker) | Команды для работы с файлами ресурсов QnA Maker и управления базами знаний. |

В предстоящих выпусках будут реализованы следующие средства:
- LUIS (API)
- Dispatch

Сведения о сопоставлении между прежними и новыми инструментами см. в [схеме переноса](https://github.com/microsoft/botframework-cli/blob/master/PortingMap.md).

_Примечание. Прежние средства командной строки будут объявлены устаревшими в предстоящих выпусках, после чего их поддержка будет прекращена. Все дополнения, исправления ошибок и новые функции отныне будут вноситься только в интерфейс командной строки Bot Framework._

## <a name="overview"></a>Обзор

Bot Framework CLI управляет ботами и связанными службами. Он является частью комплексной платформы Microsoft Bot Framework, которая предназначена для создания решений ИИ корпоративного уровня. Помимо управления ресурсами ботов, Bot Framework CLI может применяться в конвейерах непрерывной интеграции и непрерывного развертывания (CI/CD). При создании собственного бота вам может потребоваться интеграция с такими службами, как LUIS для понимания языка, QnAMaker для ответов на простые вопросы в формате "Вопросы и ответы" и другими. Чтобы интегрировать в бот службу ИИ, можно применить следующие средства.

* Команда [`bf luis`](bf-cli-reference.md#bf-luis) для работы с файлами ресурсов LUIS ( **.lu**) и управления моделями LUIS. Она также может создать соответствующий исходный код (на C# или JavaScript).
* [LUIS API Tool](https://github.com/microsoft/botbuilder-tools/tree/master/packages/LUIS/readme.md) для развертывания локальных файлов, обучения, тестирования и публикации их в службе LUIS в виде моделей распознавания речи.
* Команда [`bf qnamaker`](bf-cli-reference.md#bf-qnamaker) для работы с базами знаний QnA Maker. Она может создавать ресурсы QnA Maker и управлять ими как локально, так и в службе QnA Maker.

* Сведения о работе с форматами файлов **.lu** и **.qna** см. в [документации по библиотеке lu](https://github.com/microsoft/botframework-cli/tree/master/packages/lu/README.md).

По мере того, как растет сложность бота, вы можете использовать средство командной строки [Dispatch](https://github.com/Microsoft/botbuilder-tools/tree/master/packages/Dispatch), чтобы создавать, оценивать и распределять намерения по нескольким моделям LUIS и базам знаний QnA Maker.

Для тестирования и тонкой настройки бота можно использовать новое средство [Bot Framework Emulator](https://github.com/Microsoft/BotFramework-Emulator/releases). Этот эмулятор позволяет тестировать и отлаживать боты на локальном компьютере или в облаке.

На ранних стадиях разработки вам могут пригодиться имитации диалогов между пользователем и ботом для определенных сценариев, которые будет поддерживать ваш бот. Используйте команду [`bf chatdown`](bf-cli-reference.md#bf-chatdown) для создания имитаций диалогов в виде файлов **.chat**, преобразования их в обогащенные расшифровки и просмотра диалогов в эмуляторе.

Наконец, с помощью [Azure CLI](https://github.com/microsoft/botframework-cli/blob/master/AzureCli.md) (команда `az bot`) вы можете создавать, скачивать, публиковать и настраивать каналы в [службе Azure Bot](https://azure.microsoft.com/services/bot-service/). Этот подключаемый модуль расширяет функциональные возможности [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) для управления ресурсами службы Azure Bot.

## <a name="privacy-and-instrumentation"></a>Конфиденциальность и инструментирование
Bot Framework CLI содержит средства инструментирования, которые помогают нам улучшать средство на основе **анонимных** данных об использовании. __По умолчанию эта функция отключена. При ее включении требуется получить согласие пользователя__. Если вы решите предоставить такое согласие, Майкрософт будет собирать некоторые данные об использовании:

* вызовы группы команд;
* установленные флаги **без** некоторых значений. Например, для параметра `--folder:name` собираются только сведения об использовании `--folder`, а имена папок не передаются.

Чтобы изменить сбор данных, используйте команду [`bf config`](bf-cli-reference.md#bf-config).

Дополнительные сведения см. в [заявлении о конфиденциальности Майкрософт](https://privacy.microsoft.com/privacystatement).  

## <a name="issues-and-feature-requests"></a>Сообщения о проблемах и запросы о реализации функций
- Сообщить о проблеме или направить запрос о реализации функции можно [здесь](https://github.com/microsoft/botframework-cli/issues).
- Известные проблемы описаны [здесь](https://github.com/microsoft/botframework-cli/labels/known-issues).

## <a name="next-steps"></a>Дальнейшие действия
- [Справочник по интерфейсу командной строки Bot Framework](bf-cli-reference.md)

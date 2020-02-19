---
title: Сведения о канале Direct Line
titleSuffix: Bot Service
description: Свойства канала Direct Line
services: bot-service
author: ivorb
manager: kamrani
ms.service: bot-service
ms.topic: conceptual
ms.date: 11/01/2019
ms.author: kamrani
ms.openlocfilehash: b5a6296e65ab05cd8a5af24872d31e5ef356dcf7
ms.sourcegitcommit: e5bf9a7fa7d82802e40df94267bffbac7db48af7
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2020
ms.locfileid: "77441509"
---
# <a name="about-direct-line"></a>Сведения о Direct Line

Канал Direct Line в Bot Framework обеспечивает простой способ интеграции бота в мобильное приложение, веб-страницу или другое приложение.
Direct Line доступен в трех видах:
- глобальная и надежна служба Direct Line, которая подойдет большинству разработчиков;
- расширения Службы приложений Direct Line — это выделенные возможности обеспечения безопасности и повышения производительности Direct Line (общедоступная предварительная версия);
- Direct Line Speech — оптимизированный канал для высокопроизводительных функций обработки речи (общедоступная версия).

Вы можете выбрать для себя оптимальный вариант Direct Line, сравнивая возможности каждого предложения с потребностями своего решения. Со временем эти предложения будут упрощены.

|                            | Direct Line | Расширение Службы приложений Direct Line | Канал Direct Line Speech |
|----------------------------|-------------|-----------------------------------|--------------------|
| Доступность и лицензирование    | Общедоступная версия | Общедоступная предварительная версия без соглашения об уровне обслуживания  | GA |
| Производительность распознавания речи и преобразования текста в речь | Standard | Standard | Высокопроизводительные |
| Поддержка устаревших веб-браузеров | Да | В общедоступной версии | Да |
| Поддержка пакета SDK Bot Framework | Все версии 3 и 4 | Требуется более поздняя версия, чем 4.63 | Требуется более поздняя версия, чем 4.63 |
| Поддержка клиентских пакетов SDK    | JS, C# | JS, C# | C++, C#, Unity, JS|
| Работа с Web Chat  | Да | Да | нет|
| Виртуальная сеть | нет | Preview (Предварительный просмотр) | нет |


## <a name="additional-resources"></a>Дополнительные ресурсы
- [Подключение бота к Direct Line](bot-service-channel-connect-directline.md)
- [Подключение бота к каналу Direct Line Speech](bot-service-channel-connect-directlinespeech.md)

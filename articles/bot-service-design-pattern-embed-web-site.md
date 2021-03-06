---
title: Внедрение бота на веб-сайт — Служба Azure Bot
description: Узнайте о том, как разработать бот, внедренный на веб-сайте.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 48b483ad16a5af837869879c5f87d0e249f0788d
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75792196"
---
# <a name="embed-a-bot-in-a-website"></a>Внедрение бота на веб-сайт

Несмотря на то что боты часто находятся за пределами веб-сайтов, их также можно внедрить на веб-сайтах. Например, можно внедрить [бота набора знаний](~/bot-service-design-pattern-knowledge-base.md) на веб-сайте, чтобы пользователи могли быстро найти сведения, которые в противном случае было бы трудно найти в сложной структуре веб-сайта. Кроме того, бот можно внедрить на веб-сайте службы поддержки, который будет выступать в качестве первого отвечающего устройства на входящие запросы пользователей. Бот может независимо устранять простые проблемы и [передавать](~/bot-service-design-pattern-handoff-human.md) более сложные агенту-человеку. 

В этой статье рассматриваются интеграции ботов с веб-сайтами и использование механизма *обратного канала* для упрощения частной связи между веб-страницей и ботом. 

Корпорация Майкрософт предоставляет два различных способа интеграции бота с веб-сайтом: веб-элемент управления Skype и веб-элемента управления с открытым кодом.

## <a name="skype-web-control"></a>Веб-элемент управления Skype

[Веб-элемент управления Skype](https://aka.ms/bot-skype-web-control) — это по сути клиент Skype в веб-элементе управления. Встроенная проверка подлинности Skype позволяет боту проводить проверку подлинности и распознавание пользователей, не требуя от разработчика написания пользовательского кода. Служба Skype автоматически распознает учетные записи Майкрософт, используемых в ее веб-клиенте. 

Так как веб-элемент управления Skype выступает в качестве внешнего интерфейса для Skype, клиент Skype пользователя автоматически имеет доступ ко всему контексту любого диалога, осуществляемого с использованием веб-элемента управления. Даже закрыв веб-браузер, пользователь может продолжать взаимодействие с ботом с помощью клиента Skype. 

## <a name="open-source-web-control"></a>Веб-элемент управления с открытым кодом

<a href="https://aka.ms/BotFramework-WebChat" target="_blank">Элемент управления веб-чатом с открытым кодом</a> создан на базе ReactJS и использует [Direct Line API][directLineAPI] для взаимодействия с Bot Framework. Элемент управления веб-чатом предоставляет пустой холст для реализации веб-чата, его возможности, а также дает полный контроль над его поведением. 

Механизм *обратного канала* обеспечивает прямое взаимодействие веб-страницы, на которой размещается элемент управления, с ботом способом, полностью невидимым для пользователя. Эта возможность позволяет использовать ряд полезных сценариев: 

- Веб-страница может отправлять боту соответствующие данные (например, GPS-расположение).
- Веб-страница может сообщать боту о действиях пользователя (например, "пользователь только что выбрал вариант А из раскрывающегося списка").
- Веб-страница может отправить боту маркер проверки подлинности для пользователя, вошедшего в систему.
- Бот может отправлять соответствующие данные на веб-страницу (например, текущее значение портфеля пользователя).
- Бот может отправлять "команды" веб-странице (например, изменение цвета фона).

## <a name="using-the-backchannel-mechanism"></a>Использование механизма обратного канала

[!INCLUDE [Introduction to backchannel mechanism](~/includes/snippet-backchannel.md)]

## <a name="sample-code"></a>Образец кода

<a href="https://aka.ms/BotFramework-WebChat" target="_blank">Элемент управления веб-чатом с открытым кодом</a> доступен на сайте GitHub. Дополнительные сведения о том, как можно реализовать механизм обратного канала с помощью элемента управления веб-чатом с открытым кодом и пакета SDK Bot Framework для Node.js, см. в статье [Use the backchannel mechanism](~/nodejs/bot-builder-nodejs-backchannel.md) (Использование механизма обратного канала).

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Direct Line API][directLineAPI]
- [Элемент управления "Веб-чат" с открытым кодом](https://github.com/Microsoft/BotFramework-WebChat)
- [Использование механизма обратного канала](~/nodejs/bot-builder-nodejs-backchannel.md)

[directLineAPI]: https://docs.botframework.com/restapi/directline3/#navtitle

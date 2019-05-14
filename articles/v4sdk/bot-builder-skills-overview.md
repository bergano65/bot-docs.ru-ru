---
title: Общие сведения о навыках Bot Framework | Документация Майкрософт
description: Сведения о навыках Bot Framework
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/06/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 7ce7556b58c047e20f1b32597c896baa802968aa
ms.sourcegitcommit: b94c4286f6f64955fd51ccf4a68109c43db0e47d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 05/06/2019
ms.locfileid: "65083771"
---
# <a name="virtual-assistant---skills-overview"></a>Обзор навыков (виртуальный помощник)

> [!NOTE]
> Приведенные здесь сведения относятся к пакету SDK версии 4. 

## <a name="overview"></a>Обзор

Разработчики могут создавать решения для общения, комбинируя доступные для повторного использования возможности ведения беседы, называемые навыками.

Например, компания может создать одного родительского бота, который объединяет несколько вложенных ботов от разных подразделений, или расширить использование предоставляемых другими разработчиками возможностей. В этой предварительной версии навыков разработчики могут создать нового бота (чаще всего на основе шаблона виртуального помощника), а затем добавлять и удалять навыки с помощью одной операции командной строки, объединяющей все изменения диспетчеризации и конфигурации.     

Навыки представляют собой удаленно вызываемые боты. Для упрощения создания навыков предоставляется шаблон разработчика навыков (.NET, TS).

Навыки создавались с целью обеспечить согласованность протокола действий, чтобы процесс разработки минимально отличался от обычной работы с пакетом SDK версии 4 для ботов. 

![Сценарии навыков](./media/enterprise-template/skills-scenarios.png)

## <a name="bot-framework-skills"></a>Навыки Bot Framework

В настоящее время мы предлагаем следующие навыки Bot Framework, которые основаны на платформе Microsoft Graph и предоставляются на нескольких языках.

![Сценарии навыков](./media/enterprise-template/skills-at-build.png)

| ИМЯ | ОПИСАНИЕ |
| ---- | ----------- |
|[Навык взаимодействия с календарем](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-calendar.md)|Добавление возможностей календаря в виртуальный помощник. Основан на платформах Microsoft Graph и Google.|
|[Навык взаимодействия с электронной почтой](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-email.md)|Добавление возможностей электронной почты в виртуальный помощник. Основан на платформах Microsoft Graph и Google.|
|[Навык взаимодействия со списком дел](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-todo.md)|Добавление возможностей управления заданиями в виртуальный помощник. Основан на платформе Microsoft Graph.|
|[Навык взаимодействия с точками интереса](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/productivity-pointofinterest.md)|Поиск достопримечательностей и маршрутов. Основан на платформе Azure Maps и FourSquare.|
|[Навык взаимодействия с автомобилем](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/automotive.md)|Отраслевой навык для демонстрации возможностей управления функциями автомобиля.|
|[Экспериментальные навыки](https://github.com/Microsoft/AI/blob/master/docs/reference/skills/experimental.md)|Новости, погода и заказ столиков в ресторанах.|

## <a name="getting-started"></a>Приступая к работе

Воспользуйтесь статьей [о начале работы](https://github.com/Microsoft/AI/tree/master/docs#tutorials), чтобы научиться использовать предоставленные навыки и самостоятельно создавать новые.

---
title: Управление данными состояния — Служба Azure Bot
description: Узнайте, как сохранять и извлекать данные состояния с помощью службы "Состояние бота".
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 02/20/2020
ms.openlocfilehash: 1dc6fa1a9223e61daaa52cf475095fc08e0356d7
ms.sourcegitcommit: 308e6df385b9bac9c8d60f8b75eabc813b823c38
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/20/2020
ms.locfileid: "77519963"
---
# <a name="manage-state-data"></a>Управление данными состояния

Боты обычно используют хранилище для отслеживания того, в каком месте диалога находится пользователь, а также для сбора сведений о взаимоотношениях пользователя с ботом. Пакет SDK для Bot Framework управляет состоянием пользователя и диалога автоматически для разработчиков ботов. 

Изначально в Bot Framework включалась служба состояний для хранения этих данных. Но большинство современных ботов (и все последние выпуски пакета SDK для Bot Framework) используют хранилище, управляемое непосредственно разработчиком бота, а не централизованно управляемую службу. 

Поддержка центральной службы состояний Bot Framework прекращена 30 марта 2018 г. См. сведения о том, что [вам нужно знать в связи с прекращением поддержки службы состояний Bot Framework](https://blog.botframework.com/2018/04/02/reminder-the-bot-framework-state-service-has-been-retired-what-you-need-to-know/).

---
title: Управление данными состояния — Служба Azure Bot
description: Узнайте, как сохранять и извлекать данные состояния с помощью службы "Состояние бота".
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 1453ad74725e0084f0b5f45fa7d20475dbee2fa6
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75789216"
---
# <a name="manage-state-data"></a>Управление данными состояния

Служба состояний Microsoft Bot Framework устарела с 30 марта 2018 г. Ранее боты, созданные на основе службы Azure Bot или пакета SDK для Bot Builder, по умолчанию использовали подключение к этой службе, размещенной корпорацией Майкрософт, для хранения данных о состоянии бота. Теперь эти боты нужно обновить, чтобы они использовали собственное хранилище состояний.

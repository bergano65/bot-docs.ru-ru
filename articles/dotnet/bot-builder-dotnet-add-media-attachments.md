---
title: Добавление мультимедийных вложений в сообщения (C# версии 3) — Служба Azure Bot
description: Сведения о добавлении мультимедийных вложений в сообщения с помощью пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: ad6a1228b8fc54f8f626c07c7ed43375c249c456
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75796022"
---
# <a name="add-media-attachments-to-messages"></a>Добавление мультимедийных вложений в сообщения

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

Обмен сообщениями между пользователем и ботом может включать мультимедийные вложения (например, изображения, видео, аудио и файлы). 

Свойство `Attachments` объекта <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> содержит массив объектов <a href="https://docs.microsoft.com/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment</a>, представляющих мультимедийные вложения и функциональные карточки, прилагаемые к сообщениям. 

> [!NOTE]
> [См. сведения о добавлении функциональных карточек в сообщения](bot-builder-dotnet-add-rich-card-attachments.md).

## <a name="add-a-media-attachment"></a>Добавление мультимедийного вложения  

Чтобы добавить мультимедийное вложение в сообщение, создайте объект `Attachment` для действия `message` и задайте свойства `ContentType`, `ContentUrl` и `Name`. 

[!code-csharp[Add media attachment](../includes/code/dotnet-add-attachments.cs#addMediaAttachment)]

Если вложение представляет собой изображение, аудиофайл или видео, служба соединителя будет передавать данные вложения каналу так, чтобы позволить [каналу](bot-builder-dotnet-channeldata.md) обрабатывать это вложение в диалоге. Если вложение представляет собой файл, URL-адрес файла будет отображаться в диалоге как гиперссылка.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Справочник по каналам][inspector]
- [Общие сведения о действиях](bot-builder-dotnet-activities.md)
- [Создание сообщений](bot-builder-dotnet-create-messages.md)
- [Добавление вложений в виде форматированных карточек в сообщения](bot-builder-dotnet-add-rich-card-attachments.md)
- <a href="https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
- <a href="https://docs.microsoft.com/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment class</a> (Класс Attachment)

[inspector]: ../bot-service-channels-reference.md


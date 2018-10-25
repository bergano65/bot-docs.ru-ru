---
title: Добавление мультимедийных вложений в сообщения | Документация Майкрософт
description: Сведения о добавлении мультимедийных вложений в сообщения с помощью пакета SDK Bot Builder для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 8fd9b181676eee24b1e9c64c79663d0d0ac8abfa
ms.sourcegitcommit: b78fe3d8dd604c4f7233740658a229e85b8535dd
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 10/24/2018
ms.locfileid: "49997321"
---
# <a name="add-media-attachments-to-messages"></a>Добавление мультимедийных вложений в сообщения

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-add-media-attachments.md)
> - [Node.js](../nodejs/bot-builder-nodejs-send-receive-attachments.md)
> - [REST](../rest-api/bot-framework-rest-connector-add-media-attachments.md)

Обмен сообщениями между пользователем и ботом может включать мультимедийные вложения (например, изображения, видео, аудио и файлы). Свойство `Attachments` объекта <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Activity</a> содержит массив объектов <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment</a>, представляющих мультимедийные вложения и функциональные карточки, прилагаемые к сообщениям. 

> [!NOTE]
> [См. сведения о добавлении функциональных карточек в сообщения](bot-builder-dotnet-add-rich-card-attachments.md).

## <a name="add-a-media-attachment"></a>Добавление мультимедийного вложения  

Чтобы добавить мультимедийное вложение в сообщение, создайте объект `Attachment` для действия `message` и задайте свойства `ContentType`, `ContentUrl` и `Name`. 

[!code-csharp[Add media attachment](../includes/code/dotnet-add-attachments.cs#addMediaAttachment)]

Если вложение представляет собой изображение, аудиофайл или видео, служба соединителя будет передавать данные вложения каналу так, чтобы позволить [каналу](bot-builder-dotnet-channeldata.md) обрабатывать это вложение в диалоге. Если вложение представляет собой файл, URL-адрес файла будет отображаться в диалоге как гиперссылка.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Предварительный просмотр компонентов с помощью Channel Inspector][inspector]
- [Общие сведения о действиях](bot-builder-dotnet-activities.md)
- [Создание сообщений](bot-builder-dotnet-create-messages.md)
- [Add Rich Cards to Messages](bot-builder-dotnet-add-rich-card-attachments.md) (Добавление функциональных карточек в сообщения)
- <a href="https://docs.botframework.com/en-us/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html" target="_blank">Класс Activity</a>
- <a href="https://docs.microsoft.com/en-us/dotnet/api/microsoft.bot.connector.attachments?view=botconnector-3.12.2.4" target="_blank">Attachment class</a> (Класс Attachment)

[inspector]: ../bot-service-channel-inspector.md



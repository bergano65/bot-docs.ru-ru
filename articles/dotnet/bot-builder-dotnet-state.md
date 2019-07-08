---
title: Управление данными о состоянии | Документация Майкрософт
description: Узнайте, как сохранять и извлекать данные о состоянии с помощью пакета SDK Bot Framework для .NET.
author: RobStand
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.subservice: sdk
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: fc4fe669c57c53b8b53a98bf495ed858094ef4b6
ms.sourcegitcommit: 697a577d72aaf91a0834d4b4c2ef5aa11291f28f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/01/2019
ms.locfileid: "67496634"
---
# <a name="manage-state-data"></a>Управление данными состояния

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

> [!div class="op_single_selector"]
> - [.NET](../dotnet/bot-builder-dotnet-state.md)
> - [Node.js](../nodejs/bot-builder-nodejs-state.md)

[!INCLUDE [State concept overview](../includes/snippet-dotnet-concept-state.md)]

## <a name="in-memory-data-storage"></a>Хранилище данных в памяти

Хранилище данных в памяти предназначено только для тестирования. Это временное и нестабильное хранилище. Данные очищаются при каждом перезапуске бота. Чтобы использовать хранилище в памяти для тестирования, сделайте следующее: 

Установите следующие пакеты NuGet: 
- Microsoft.Bot.Builder.Azure
- Autofac.WebApi2

В методе **Application_Start** создайте экземпляр хранилища в памяти и зарегистрируйте это хранилище данных:

```cs
// Global.asax file

var store = new InMemoryDataStore();

Conversation.UpdateContainer(
           builder =>
           {
               builder.Register(c => store)
                         .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                         .AsSelf()
                         .SingleInstance();

               builder.Register(c => new CachingBotDataStore(store,
                          CachingBotDataStoreConsistencyPolicy
                          .ETagBasedConsistency))
                          .As<IBotDataStore<BotData>>()
                          .AsSelf()
                          .InstancePerLifetimeScope();


           });
GlobalConfiguration.Configure(WebApiConfig.Register);

```

Этот метод можно использовать для настройки пользовательского хранилища данных или использования любого из *расширений Azure*.

## <a name="manage-custom-data-storage"></a>Управление пользовательским хранилищем данных

Для повышения производительности и (или) безопасности в рабочей среде можно реализовать собственное хранилище данных или выбрать любой из следующих вариантов для хранения данных:

1. [управление данными о состоянии с помощью Cosmos DB](bot-builder-dotnet-state-azure-cosmosdb.md);

2. [управление данными о состоянии с помощью хранилища таблиц](bot-builder-dotnet-state-azure-table-storage.md).

При выборе любого из этих [расширений Azure](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/) механизм установки и сохранения данных с помощью пакета SDK Bot Framework для .NET остается тем же, что для хранилища данных в памяти.

## <a name="bot-state-methods"></a>Методы состояний для ботов

В таблице ниже перечислены методы для управления данными о состоянии.

| Метод | Область действия | Назначение |                                                
|----|----|----|
| `GetUserData` | Пользователь | Получение данных о состоянии, ранее сохраненных для пользователя в указанном канале |
| `GetConversationData` | Беседа | Получение данных о состоянии, ранее сохраненных для беседы в указанном канале |
| `GetPrivateConversationData` | Пользователь и беседа | Получение данных о состоянии, ранее сохраненных для пользователя в определенной беседе в указанном канале |
| `SetUserData` | Пользователь | Сохранение данных о состоянии для пользователя в указанном канале |
| `SetConversationData` | Беседа | Сохранение данных о состоянии для беседы в указанном канале <br/><br/>**Примечание**. Так как метод `DeleteStateForUser` не удаляет данных, сохраненные ранее с помощью метода `SetConversationData`, НЕ ИСПОЛЬЗУЙТЕ этот метод для хранения персональных данных пользователя. |
| `SetPrivateConversationData` | Пользователь и беседа | Сохранение данных о состоянии для пользователя в определенной беседе в указанном канале |
| `DeleteStateForUser` | Пользователь | Удаление данных о состоянии, ранее сохраненных для пользователя с помощью методов `SetUserData` и (или) `SetPrivateConversationData`. <br/><br/>**Примечание**. Бот должен выполнять этот метод каждый раз, когда он получает действие типа [deleteUserData](bot-builder-dotnet-activities.md#deleteuserdata) или [contactRelationUpdate](bot-builder-dotnet-activities.md#contactrelationupdate), свидетельствующее об удалении бота из списка контактов пользователя. |

Если бот сохранит данные о состоянии с помощью любого из методов **Set...Data**, эти данные будут включены во все будущие сообщения в том же контексте, и бот сможет получить их с помощью соответствующего метода **Get...Data**.

## <a name="useful-properties-for-managing-state-data"></a>Полезные свойства для управление данными о состоянии

Каждый объект [Activity][Activity] содержит свойства, которые вы можете использовать для управления данными о состоянии.

| Свойство | ОПИСАНИЕ | Вариант использования |
|----|----|----|
| `From` | Уникальный идентификатор пользователя в канале | Сохранение и извлечение данных о состоянии, связанных с определенным пользователем |
| `Conversation` | Уникальный идентификатор беседы | Сохранение и извлечение данных о состояния, связанных с определенной беседой |
| `From` и `Conversation` | Уникальный идентификатор сочетания пользователя и беседы | Сохранение и извлечение данных о состоянии, связанных с определенным пользователем в контексте определенной беседы |

> [!NOTE]
> Вы можете использовать эти значения свойств в качестве ключей, даже если данные о состоянии сохраняются в пользовательской базе данных, а не хранилище данных о состоянии Bot Framework.

## <a name="handle-concurrency-issues"></a>Устранение проблем с параллелизмом

Бот может получить сообщение об ошибке с кодом состояния HTTP **412 (Необходимое условие не выполнено)** при попытке сохранить данные о состоянии, если другой экземпляр бота уже изменил те же данные. При разработке бота вы можете учесть этот сценарий, как показано в следующем примере кода.

[!code-csharp[Handle exception saving state](../includes/code/dotnet-state.cs#handleException)]

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Руководство по устранению неполадок с Bot Framework](../bot-service-troubleshoot-general-problems.md)
- <a href="/dotnet/api/?view=botbuilder-3.11.0" target="_blank">Справочная информация по пакету SDK Bot Framework для .NET</a>

[Activity]: https://docs.botframework.com/csharp/builder/sdkreference/dc/d2f/class_microsoft_1_1_bot_1_1_connector_1_1_activity.html

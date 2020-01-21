---
title: Управление данными пользовательских состояний с помощью Хранилища таблиц Azure (C# версии 3) — Служба Azure Bot
description: Узнайте, как сохранять и извлекать данные о состоянии в хранилище таблиц Azure с помощью пакета SDK Bot Framework для .NET.
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 5fcce457e5365cd54be77812f0b9fd70382a813b
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75797935"
---
# <a name="manage-custom-state-data-with-azure-table-storage-for-net"></a>Управление данными пользовательских состояний с помощью Хранилища таблиц Azure для .NET

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

В этой статье вы реализуете Хранилище таблиц Azure для хранения данных о состоянии вашего бота и управления ими. Служба Connector State Service, используемая ботами по умолчанию, не предназначена для рабочей среды. Следует или использовать [расширения Azure](https://github.com/Microsoft/BotBuilder-Azure) с веб-сайта GitHub, или реализовывать клиент состояния пользователя с помощью любой платформы хранилища данных. Ниже приведены причины, по которым следует использовать хранилище состояния пользователя:
 - более высокая пропускная способность API состояния (больше контроля над производительностью);
 - меньше задержка для географического распределения;
 - управление местом хранения данных;
 - доступ к данным о фактическом состоянии;
 - возможность хранения более 32 КБ данных.

## <a name="prerequisites"></a>предварительные требования
Что вам понадобится:
 - [Учетная запись Microsoft Azure](https://azure.microsoft.com/free/)
 - [Visual Studio 2015 или более поздней версии](https://www.visualstudio.com/).
 - [Пакет Azure NuGet для Bot Builder](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/).
 - [Пакет NuGet Autofac.WebApi2](https://www.nuget.org/packages/Autofac.WebApi2/).
 - [Bot Framework Emulator](https://emulator.botframework.com/).
 - [Обозреватель службы хранилища Azure](http://storageexplorer.com/)
 
## <a name="create-azure-account"></a>Создание учетной записи Azure
Если у вас нет учетной записи Azure, нажмите [здесь](https://azure.microsoft.com/free/), чтобы зарегистрировать бесплатную учетную запись.

## <a name="set-up-the-azure-table-storage-service"></a>Настройка службы хранилища таблиц Azure
1. После входа на портал Azure нажмите **Создать**, чтобы создать новую службу хранилища таблиц Azure. 
2. Найдите **учетную запись хранения**, реализующую таблицу Azure. 
3. Заполните поля и нажмите кнопку **Создать** в нижней части экрана, чтобы развернуть новую службу хранилища. После развертывания новой службы хранилища в ней отобразятся доступные вам функции и параметры.
4. Выберите слева вкладку **Ключи доступа** и скопируйте строку подключения для последующего использования. Ваш бот будет использовать ее для вызова службы хранилища и сохранения данных состояния.

## <a name="install-nuget-packages"></a>Установка пакетов Nuget
1. Откройте существующий проект бота C# или создайте его с помощью шаблона бота C# в Visual Studio. 
2. Установите следующие пакеты NuGet:
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>Добавление строки подключения 
Добавьте следующую запись в файл Web.config. 
```XML
  <connectionStrings>
    <add name="StorageConnectionString"
    connectionString="YourConnectionString"/>
  </connectionStrings>
```
Замените YourConnectionString строкой подключения для Хранилища таблиц Azure, которую вы записали ранее. Сохраните файл Web.config.

## <a name="modify-your-bot-code"></a>Изменение кода бота
Добавьте в файл Global.asax.cs следующие операторы `using`.
```cs
using Autofac;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;
```
В методе `Application_Start()` создайте экземпляр класса `TableBotDataStore`. Класс `TableBotDataStore` реализует интерфейс `IBotDataStore<BotData>`. Интерфейс `IBotDataStore` позволяет переопределить подключение к службе состояния соединителя по умолчанию.
 ```cs
 var store = new TableBotDataStore(ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);
 ```
Зарегистрируйте эту службу, как показано ниже.
 ```cs
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
 ```
Сохраните файл Global.asax.cs.

## <a name="run-your-bot-app"></a>Запуск приложения бота
Запустите бот в Visual Studio. Добавленный код создаст пользовательскую таблицу **botdata** в Azure.

## <a name="connect-your-bot-to-the-emulator"></a>Подключение бота к эмулятору
На этом этапе бот выполняется локально. После этого запустите эмулятор и подключитесь к боту в эмуляторе.
1. Введите http://localhost:port-number/api/messages в адресной строке, где значение port-number должно соответствовать номеру порта, указанному в браузере, где выполняется приложение. Поля <strong>Идентификатор приложения Microsoft</strong> и <strong>Пароль приложения Microsoft</strong> пока можно не заполнять. Вы получите эти сведения позже, когда выполните [регистрацию бота](~/bot-service-quickstart-registration.md).
2. Нажмите кнопку **Соединить**. 
3. Протестируйте бот, введя несколько сообщений в эмуляторе. 

## <a name="view-data-in-azure-table-storage"></a>Просмотр данных в Хранилище таблиц Azure
Чтобы просмотреть данные о состоянии, откройте **Обозреватель службы хранилища** и подключитесь к Azure с помощью своих учетных данных для портала Azure или подключитесь напрямую к таблице с помощью имени хранилища и ключа к хранилищу, затем перейдите к нужной таблице.  

## <a name="next-steps"></a>Дальнейшие действия
В этой статье было реализовано Хранилище таблиц Azure для сохранения данных бота и управления ими. Далее вы можете узнать, как моделировать поток общения с помощью диалогов.

> [!div class="nextstepaction"]
> [Управление потоком общения с помощью диалогов](bot-builder-dotnet-manage-conversation-flow.md)


## <a name="additional-resources"></a>Дополнительные ресурсы

Если вы не знакомы с контейнерами инверсии управления и шаблоном внедрения зависимостей, используемыми в приведенном выше коде, перейдите на сайт [Autofac](http://autofac.readthedocs.io/en/latest/), чтобы узнать больше. 

Вы также можете скачать полный пример для [Хранилища таблиц Azure](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/AzureTable) с сайта GitHub.

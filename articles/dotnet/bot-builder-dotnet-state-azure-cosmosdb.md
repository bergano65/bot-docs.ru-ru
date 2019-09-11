---
title: Управление данными о состоянии пользователя с помощью Azure Cosmos DB | Документация Майкрософт
description: Узнайте, как сохранять и извлекать данные о состоянии в Azure Cosmos DB с помощью пакета SDK Bot Framework для .NET.
author: kamrani
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
monikerRange: azure-bot-service-3.0
ms.openlocfilehash: 2495e0506072609e47f4014f1027ed5033fb4bf7
ms.sourcegitcommit: a6d02ec4738e7fc90b7108934740e9077667f3c5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 09/04/2019
ms.locfileid: "70297775"
---
# <a name="manage-custom-state-data-with-azure-cosmos-db-for-net"></a>Управление данными о пользовательских состояниях с помощью Azure Cosmos DB

[!INCLUDE [pre-release-label](../includes/pre-release-label-v3.md)]

В этой статье вы реализуете Azure Cosmos DB для хранения данных о состоянии вашего бота и управления ими. Служба состояния соединителя, используемая ботами по умолчанию, не предназначена для рабочей среды. Следует или использовать [расширения Azure](https://github.com/Microsoft/BotBuilder-Azure) с веб-сайта GitHub, или реализовывать клиент состояния пользователя с помощью любой платформы хранилища данных. Ниже приведены причины, по которым следует использовать хранилище пользовательских состояний:
 - более высокая пропускная способность API состояния (больше контроля над производительностью);
 - меньше задержка для географического распределения;
 - управление местом хранения данных;
 - доступ к данным о фактическом состоянии;
 - возможность хранения более 32 КБ данных.
 
## <a name="prerequisites"></a>Предварительные требования
Что вам понадобится:
 - [Учетная запись Microsoft Azure](https://azure.microsoft.com/free/).
 - [Visual Studio 2015 или более поздней версии](https://www.visualstudio.com/).
 - [Пакет Azure NuGet для Bot Builder](https://www.nuget.org/packages/Microsoft.Bot.Builder.Azure/).
 - [Пакет NuGet Autofac.WebApi2](https://www.nuget.org/packages/Autofac.WebApi2/).
 - [Bot Framework Emulator](~/bot-service-debug-emulator.md).
 
## <a name="create-azure-account"></a>Создание учетной записи Azure
Если у вас нет учетной записи Azure, нажмите [здесь](https://azure.microsoft.com/free/), чтобы зарегистрировать бесплатную учетную запись.

## <a name="set-up-the-azure-cosmos-db-database"></a>Настройка базы данных Azure Cosmos DB
1. После входа на портал Azure щелкните **Создать**, чтобы создать базу данных *Azure Cosmos DB*. 
2. Щелкните **Базы данных**. 
3. Найдите **Azure Cosmos DB** и щелкните **Создать**.
4. Заполните поля. Для поля **API** выберите значение **SQL (DocumentDB)** . Заполните все поля и нажмите кнопку **Создать** в нижней части экрана, чтобы развернуть новую базу данных. 
5. После развертывания новой базы данных перейдите к ней. Щелкните **Ключи доступа**, чтобы найти ключи и строки подключения. Ваш бот будет использовать эту информацию для вызова службы хранилища и сохранения данных о состоянии.

## <a name="install-nuget-packages"></a>Установка пакетов Nuget
1. Откройте существующий проект бота C# или создайте его с помощью шаблона бота в Visual Studio. 
2. Установите следующие пакеты NuGet:
   - Microsoft.Bot.Builder.Azure
   - Autofac.WebApi2

## <a name="add-connection-string"></a>Добавление строки подключения 
Добавьте следующие записи в файл Web.config.
```XML
<add key="DocumentDbUrl" value="Your DocumentDB URI"/>
<add key="DocumentDbKey" value="Your DocumentDB Key"/>
```
Вы замените это значение универсальным кодом ресурса (URI) и первичным ключом из Azure Cosmos DB. Сохраните файл Web.config.

## <a name="modify-your-bot-code"></a>Изменение кода бота
Чтобы использовать хранилище **Azure Cosmos DB**, добавьте следующие строки кода в метод **Application_Start()** в файле **Global.asax.cs** своего бота.

```cs
using System;
using Autofac;
using System.Web.Http;
using System.Configuration;
using Microsoft.Bot.Connector;
using Microsoft.Bot.Builder.Azure;
using Microsoft.Bot.Builder.Dialogs;
using Microsoft.Bot.Builder.Dialogs.Internals;

namespace SampleApp
{
    public class WebApiApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
            GlobalConfiguration.Configure(WebApiConfig.Register);
            var uri = new Uri(ConfigurationManager.AppSettings["DocumentDbUrl"]);
            var key = ConfigurationManager.AppSettings["DocumentDbKey"];
            var store = new DocumentDbBotDataStore(uri, key);

            Conversation.UpdateContainer(
                        builder =>
                        {
                            builder.Register(c => store)
                                .Keyed<IBotDataStore<BotData>>(AzureModule.Key_DataStore)
                                .AsSelf()
                                .SingleInstance();

                            builder.Register(c => new CachingBotDataStore(store, CachingBotDataStoreConsistencyPolicy.ETagBasedConsistency))
                                .As<IBotDataStore<BotData>>()
                                .AsSelf()
                                .InstancePerLifetimeScope();

                        });

        }
    }
}
```

Сохраните файл global.asax.cs. Теперь все готово к тестированию бота в эмуляторе.

## <a name="run-your-bot-app"></a>Запуск приложения бота
Запустите бот в Visual Studio. Добавленный код создаст пользовательскую таблицу **botdata** в Azure.

## <a name="connect-your-bot-to-the-emulator"></a>Подключение бота к эмулятору
На этом этапе бот выполняется локально. После этого запустите эмулятор и подключитесь к боту в эмуляторе.
1. Введите http://localhost:port-number/api/messages в адресной строке, где значение port-number должно соответствовать номеру порта, указанному в браузере, где выполняется приложение. Поля <strong>Идентификатор приложения Microsoft</strong> и <strong>Пароль приложения Microsoft</strong> пока можно не заполнять. Вы получите эти сведения позже, когда выполните [регистрацию бота](~/bot-service-quickstart-registration.md).
2. Щелкните **Подключить**. 
3. Протестируйте бот, введя несколько сообщений в эмуляторе. 

## <a name="view-state-data-on-azure-portal"></a>Просмотр данных о состоянии на портале Azure.
Чтобы просмотреть данные о состоянии, войдите на портал Azure и перейдите к базе данных. Щелкните **Обозреватель данных (предварительная версия)** , чтобы убедиться, что сведения о состоянии из бота сохраняются. 

## <a name="next-steps"></a>Дополнительная информация
В этой статье вы использовали Cosmos DB для сохранения данных бота и управления ими. Далее вы можете узнать, как моделировать поток общения с помощью диалогов.

> [!div class="nextstepaction"]
> [Управление потоком общения с помощью диалогов](bot-builder-dotnet-manage-conversation-flow.md)

## <a name="additional-resources"></a>Дополнительные ресурсы
Если вы не знакомы с контейнерами инверсии управления и шаблоном внедрения зависимостей, используемыми в приведенном выше коде, посетите сайт [Autofac](http://autofac.readthedocs.io/en/latest/), чтобы узнать больше. 

Вы также можете скачать [пример](https://github.com/Microsoft/BotBuilder-Azure/tree/master/CSharp/Samples/DocumentDb) с сайта GitHub, чтобы узнать больше об использовании Cosmos DB для управления состоянием. 

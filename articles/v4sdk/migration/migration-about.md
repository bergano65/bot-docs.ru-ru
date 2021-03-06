---
title: Различия между версиями 3 и 4 пакета SDK — Служба Azure Bot
description: Описывает различия между версиями 3 и 4 пакета SDK.
keywords: bot migration, formflow, dialogs, state
author: JonathanFingold
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 6f11ae22ca92ec2177fdc66bccba9ba74d5ae8d3
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791036"
---
# <a name="differences-between-the-v3-and-v4-net-sdk"></a>Различия между версиями 3 и 4 пакета SDK для .NET

Версия 4 пакета SDK Bot Framework поддерживает ту же базовую службу Bot Framework, что и версия 3. Но в версии 4 выполнен рефакторинг кода предыдущей версии пакета SDK для повышения гибкости и улучшения контроля над ботами. Основные изменения в пакете SDK:

- Общие сведения об адаптере бота. Адаптер — это компонент стека обработки действий.
  - Адаптер выполняет аутентификацию Bot Framework.
  - Адаптер управляет входящим и исходящим трафиком между каналом и обработчиком шагов вашего бота, инкапсулируя вызовы в Bot Framework Connector.
  - Адаптер инициализирует контекст для каждого шага.
  - Дополнительные сведения см. в статье [Принципы работы бота][about-bots].
- Выполнен рефакторинг управления состоянием.
  - Данные о состоянии автоматически больше не предоставляются в боте.
  - Управление состоянием теперь осуществляется через соответствующие объекты и методы доступа к свойствам.
  - Дополнительные сведения см. в статье [Управление состоянием][about-state].
- Новая библиотека диалогов.
  - Диалоги версии 3 потребовалось переписать для новой библиотеки диалогов.
  - Диалоги с возможностью оценки Scoreable больше не существуют. Вы можете проверить наличие глобальных команд, прежде чем передавать средства управления в диалоги. В зависимости от архитектуры бота версии 4 это можно сделать в обработчике сообщений или в родительском диалоге. Пример см. в статье [Обработка прерываний со стороны пользователя][interruptions].
  - Дополнительные сведения см. в статье [Библиотека диалогов][about-dialogs].
- Поддержка ASP.NET Core.
  - Шаблоны создания ботов C# предназначены для платформы ASP.NET Core.
  - Вы все еще можете использовать ASP.NET для своих ботов, но в версии 4 мы хотим уделить основное внимание поддержке платформы ASP.NET Core.
  - Дополнительные сведения об этой платформе см. в статье [Введение в ASP.NET Core](https://docs.microsoft.com/aspnet/core/).

## <a name="activity-processing"></a>Обработка действий

При создании адаптера для бота вы также указываете делегата обработчика сообщений, который будет получать входящие действия от каналов и пользователей. Адаптер создает объект контекста шага для каждого полученного действия. Он передает объект контекста шага обработчику шагов в боте, а затем удаляет объект после завершения шага.

Обработчик шагов может получать действия разных типов. Как правило, диалогам в боте необходимо перенаправлять только действия _сообщений_. Если вы наследуете бота из `ActivityHandler`, обработчик шагов в нем будет пересылать все сообщения о действиях в `OnMessageActivityAsync`. Переопределите этот метод, чтобы добавить логику обработки сообщений. Дополнительные сведения о типах действий см. в статье о [Bot Framework — Activity][].

### <a name="handling-turns"></a>Шаги обработки

При обработке сообщений используйте контекст шага, чтобы получить данные о входящем действии и отправить действия пользователю:

| | |
|-|-|
| Получение входящего действия | Получите свойство `Activity` контекста шага. |
| Создание действия и отправка его пользователю | Вызовите метод `SendActivityAsync` контекста шага.<br/>Дополнительные сведения см. в статьях [Отправка и получение текстовых сообщений][send-messages] и [Добавление мультимедиа в сообщения][send-media]. |

Класс `MessageFactory` предоставляет некоторые вспомогательные методы для создания и форматирования действий.

### <a name="scorables-is-gone"></a>Элементы с возможностью оценки больше не используются

Обрабатывайте такие элементы в цикле сообщений бота. Описание такого процесса для диалогов в версии 4 см. в статье [Обработка прерываний со стороны пользователя][interruptions].

Составные деревья отправки с возможностью оценки и составные последовательные диалоги, например _исключение по умолчанию_, также больше не используются. Чтобы включить эти возможности, вы можете реализовать их в обработчике шагов бота.

## <a name="state-management"></a>Управление данными о состоянии

В версии 3 вы могли хранить данные о беседе в службе "Состояние бота", которая входит в обширный набор служб для Bot Framework. Но после 31 марта 2018 г. эта служба более недоступна. Начиная с версии 4, применяются те же рекомендации по управлению состоянием, что и для любого веб-приложения, и его можно реализовать несколькими способами. Обычно проще всего кэшировать состояние в памяти в том же процессе, но в производственных приложениях состояние следует сохранять в более устойчивом хранилище, например в базе данных SQL или NoSQL или в больших двоичных объектах.

В версии 4 для управления состоянием не используются свойства `UserData`, `ConversationData` и `PrivateConversationData`, а также контейнеры данных.
Управление состоянием теперь осуществляется через соответствующие объекты и методы доступа к свойствам, как описано в статье [Управление состоянием][about-state].

Версия 4 определяет классы `UserState`, `ConversationState` и `PrivateConversationState`, которые управляют данными состояния для бота. Вам нужно создать метод доступа к свойству состояния для каждого свойства, которое нужно сохранить (а не просто считывать и записывать его в предопределенный контейнер данных).

### <a name="setting-up-state"></a>Настройка состояния

Если это возможно, состояние необходимо настроить с использованием одноэлементных экземпляров (в **Startup.cs** для .NET Core или в **Global.asax.cs** для .NET Framework).

1. Инициализируйте один или несколько объектов `IStorage`. Вы получите резервное хранилище для данных вашего бота.
    Пакет SDK версии 4 предоставляет несколько [уровней хранения](../bot-builder-concept-state.md#storage-layer).
    Вы также можете реализовать собственные уровни, чтобы подключаться к хранилищам различных типов.
1. Затем при необходимости создайте и зарегистрируйте объекты [управления состоянием](../bot-builder-concept-state.md#state-management).
    Вам доступны те же области, что и в версии 3, и при необходимости вы можете создать другие.
1. Затем создайте и зарегистрируйте [методы доступа к свойствам состояния](../bot-builder-concept-state.md#state-property-accessors) для нужных боту свойств.
    В объекте управления состоянием каждый метод доступа к свойству должен иметь уникальное имя.

### <a name="using-state"></a>Использование состояния

Вы можете использовать внедрение зависимостей для получения к ним доступа при каждом создании бота.
(В ASP.NET новый экземпляр бота или контроллера сообщений создается для каждого шага.) Используйте методы доступа к свойствам состояния, чтобы получить и обновить свои свойства, а объекты управления состоянием, чтобы записать какие-либо изменения в хранилище. Так как вам необходимо учитывать особенности параллелизма, ниже описано, как выполнять некоторые стандартные задачи.

| | |
|-|-|
| Создание метода доступа к свойству состояния | Вызовите процедуру `BotState.CreateProperty<T>`.<br/>`BotState` является абстрактным базовым классом для беседы, закрытой беседы и состояния пользователя. |
| Получение текущего значения свойства | Вызовите процедуру `IStatePropertyAccessor<T>.GetAsync`.<br/>Если значение не задано ранее, для его создания будет использоваться фабричный параметр по умолчанию. |
| Обновление текущего кэшированного значения свойства | Вызовите процедуру `IStatePropertyAccessor<T>.SetAsync`.<br/>Эта операция обновляет только кэш, но не уровень резервного хранилища. |
| Сохранение изменений состояний в хранилище | Вызовите `BotState.SaveChangesAsync` для любого объекта управления состоянием, в котором состояние было изменено до выхода из обработчика шагов. |

### <a name="managing-concurrency"></a>Управление параллелизмом

Возможно, вашему боту потребуется управлять параллелизмом состояния. Дополнительные сведения см. в разделе [Сохранение состояния](../bot-builder-concept-state.md#saving-state) статьи **Управление состоянием** и в разделе [Управление параллелизмом с помощью тегов eTag](../bot-builder-howto-v4-storage.md#manage-concurrency-using-etags) статьи **Запись данных напрямую в хранилище**.

## <a name="dialogs-library"></a>Библиотека диалогов

Ниже приведены некоторые из основных изменений в диалогах:

- Библиотека диалогов теперь имеет вид отдельного пакета NuGet: **Microsoft.Bot.Builder.Dialogs**.
- Классы диалогов теперь не нужно сериализовать. Управление состоянием диалога осуществляется через метод доступа к свойству состояния `DialogState`.
  - Свойство состояния диалога теперь сохраняется между шагами в отличие от самого объекта диалога.
- Интерфейс `IDialogContext` заменен классом `DialogContext`. В ходе выполнения шага вы создаете контекст диалога для _набора диалогов_.
  - Этот контекст диалога инкапсулирует стек диалога (старый кадр стека). Эта информация сохраняется в свойстве состояния диалога.
- Интерфейс `IDialog` заменен абстрактным классом `Dialog`.

### <a name="defining-dialogs"></a>Определение диалогов

Версия 3 предоставляла гибкую возможность реализовать диалоги через интерфейс `IDialog`, но для этого требовалось создать собственный код для таких функций, как проверка. В версии 4 появились классы запроса, которые автоматически проверяют пользовательский ввод, применяют к нему ограничение по типу (например целое число) и повторно запрашивают данные у пользователя, пока они не будут соответствовать этим ограничениям. В большинстве случаев это позволяет разработчику писать меньше кода.

Теперь вам доступно несколько способов определения диалогов:

| | |
|:--|:--|
| Компонентный диалог, наследуемый от класса `ComponentDialog`. | Позволяет инкапсулировать код диалога без конфликтов именования с другими контекстами. См. статью [Повторное использование диалогов][reuse-dialogs]. |
| Каскадный диалог, экземпляр класса `WaterfallDialog`. | Разработан для оптимальной работы с диалогами запроса, которые предлагают пользователю ввести данные и проверяют их. Каскадный диалог автоматизирует большинство процессов, но требует применения определенной формы для кода диалога (см. статью [Реализация процесса общения][sequential-flow]). |
| Настраиваемый диалог, полученный из абстрактного класса `Dialog`. | Дает максимальную гибкость в аспекте поведения диалогов, но также требует более глубоких знаний о реализации стека диалогов. |

В версии 3, вы использовали `FormFlow` для выполнения фиксированного количества шагов для задачи. В версии 4 FormFlow заменен каскадным диалогом. Создавая каскадный диалог, вы определяете шаги диалога в конструкторе. Порядок выполняемых действий строго соответствует тому, в котором вы их объявили, и переход между ними происходит автоматически.

Можно также создать сложные потоки управления с помощью нескольких диалогов, как описано в статье [Создание сложного потока беседы с использованием ветвления и циклов][complex-flow].

Чтобы получить доступ к диалогу, вам необходимо разместить его экземпляр в _наборе диалогов_, а затем сгенерировать _контекст диалога_ для такого набора. При создании набора диалогов вам нужно указать метод доступа к свойству состояния диалога. Это позволит платформе сохранять состояние диалога при переходе между шагами. Управление состоянием в версии 4 описано в статье [Управление состоянием][about-state].

### <a name="using-dialogs"></a>Использование диалогов

Ниже приведен список стандартных операций в версии 3, а также описывается их выполнение в каскадном диалоге. Обратите внимание, что каждый шаг каскадного диалога должен возвращать значение `DialogTurnResult`. В противном случае каскадный диалог может преждевременно завершиться.

| Операция | Версия 3 | версия 4 |
|:---|:---|:---|
| Обработка запуска диалога | Реализуйте `IDialog.StartAsync` | Сделайте этот шаг первым в каскадном диалоге. |
| Отправка действия | Вызовите процедуру `IDialogContext.PostAsync`. | Вызовите процедуру `ITurnContext.SendActivityAsync`.<br/>Используйте свойство `Context` контекста шага, чтобы получить контекст шага.  |
| Ожидание ответа пользователя | Используйте параметр `IAwaitable<IMessageActivity>` и вызовите `IDialogContext.Wait`. | Верните ожидание `ITurnContext.PromptAsync`, чтобы начать диалог с запросом. Затем получите результаты на следующем шаге каскадного диалога. |
| Обработка продолжения диалога | Вызовите процедуру `IDialogContext.Wait`. | Добавьте в каскадный диалог дополнительные шаги или реализуйте `Dialog.ContinueDialogAsync` |
| Обозначение завершения обработки до следующего сообщения пользователя | Вызовите процедуру `IDialogContext.Wait`. | Возвращается значение `Dialog.EndOfTurn`. |
| Запуск дочернего диалога | Вызовите процедуру `IDialogContext.Call`. | Верните ожидание для метода `BeginDialogAsync` контекста шага.<br/>Если дочерний диалог вернет значение, оно будет доступно на следующем шаге каскадного диалога с помощью свойства `Result` контекста шага. |
| Замена текущего диалога новым | Вызовите процедуру `IDialogContext.Forward`. | Верните ожидание для `ITurnContext.ReplaceDialogAsync`. |
| Информирование о завершении текущего диалога | Вызовите процедуру `IDialogContext.Done`. | Верните ожидание для метода `EndDialogAsync` контекста шага. |
| Выход из диалога | Вызовите процедуру `IDialogContext.Fail`. | Сгенерируйте исключение, которое будет зарегистрировано на другом уровне бота, завершите шаг с состоянием `Cancelled` либо вызовите шаг или `CancelAllDialogsAsync` контекста диалога.<br/>Обратите внимание, что в версии 4 исключения в диалоге распространяются по стеку C#, а не по стеку диалога. |

Другие примечания о коде версии 4:

- Различные производные классы `Prompt` в версии 4 реализуют запросы пользователей в виде отдельных диалогов из двух шагов. Изучите, как правильно [реализовать последовательный поток беседы][sequential-flow].
- Используйте `DialogSet.CreateContextAsync`, чтобы создать контекст диалога для текущего шага.
- Используйте свойство `DialogContext.Context` из диалога, чтобы получить контекст текущего шага.
- Каскадные шаги имеют параметр `WaterfallStepContext`, который является производным от `DialogContext`.
- Все конкретные классы диалогов и приглашений наследуют от абстрактного класса `Dialog`.
- Идентификатор присваивается при создании компонентного диалога. Каждый диалог в наборе диалогов должен иметь уникальный в пределах этого набора идентификатор.

### <a name="passing-state-between-and-within-dialogs"></a>Передача состояния в диалогах и между ними

В разделах о [состоянии диалога](../bot-builder-concept-dialog.md#dialog-state), [свойствах контекста для каскадного шага](../bot-builder-concept-dialog.md#waterfall-step-context-properties) и [использовании диалогов](../bot-builder-concept-dialog.md#using-dialogs) статьи **Библиотека диалогов** описано, как управлять состоянием диалогов в версии 4.

### <a name="iawaitable-is-gone"></a>IAwaitable больше не используется

Чтобы получить действие пользователя на шаге, получите его из контекста шага.

Чтобы отправить запрос пользователю и получить результат,:

- Добавьте соответствующий экземпляр запроса к вашему набору диалогов.
- Вызовите запрос из шага в каскадном диалоге.
- Извлеките результат из свойства `Result` контекста шага на следующем шаге.

### <a name="formflow"></a>FormFlow

В версии 3 FormFlow входит в состав пакета SDK для C#, но не в состав пакета SDK для JavaScript. В пакете SDK версии 4 FormFlow отсутствует, но доступна версия для сообщества для C#.

| Имя пакета NuGet | Репозиторий сообщества GitHub |
|-|-|
| Bot.Builder.Community.Dialogs.Formflow | [BotBuilderCommunity/botbuilder-community-dotnet/libraries/Bot.Builder.Community.Dialogs.FormFlow](https://github.com/BotBuilderCommunity/botbuilder-community-dotnet/tree/master/libraries/Bot.Builder.Community.Dialogs.FormFlow) |

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Перенос бота с версии 3 в версию 4 пакета SDK для .NET](conversion-framework.md)

<!-- -->

[about-bots]: ../bot-builder-basics.md
[about-state]: ../bot-builder-concept-state.md
[about-dialogs]: ../bot-builder-concept-dialog.md

[send-messages]: ../bot-builder-howto-send-messages.md
[send-media]: ../bot-builder-howto-add-media-attachments.md

[sequential-flow]: ../bot-builder-dialog-manage-conversation-flow.md
[complex-flow]: ../bot-builder-dialog-manage-complex-conversation-flow.md
[reuse-dialogs]: ../bot-builder-compositcontrol.md
[interruptions]: ../bot-builder-howto-handle-user-interrupt.md

[Bot Framework — Activity]: https://aka.ms/botSpecs-activitySchema (Схема действия Bot Framework)
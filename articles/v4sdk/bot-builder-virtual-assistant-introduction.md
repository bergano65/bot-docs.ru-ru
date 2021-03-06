---
title: Общие сведения о виртуальном помощнике — Служба Azure Bot
description: Сведения о том, как создать собственного виртуального помощника
author: darrenj
ms.author: darrenj
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 05/23/2019
monikerRange: azure-bot-service-4.0
ms.openlocfilehash: 4f817ff4911229d8bc36d2c50348bf0320d09e47
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75791862"
---
# <a name="virtual-assistant-overview"></a>Общие сведения о виртуальном помощнике

## <a name="overview"></a>Обзор

Многие клиенты и партнеры хотели бы иметь помощника для ведения диалога, который бы представлял конкретную торговую марку, учитывал особенности целевой аудитории и был доступным для широкого ряда устройств и холстов.

В рамках подхода, примененного при разработке пакета SDK для Bot Framework с открытым кодом от Майкрософт, решение виртуального помощника с открытым исходным кодом предоставляет набор важнейших средств и полный контроль над взаимодействием с пользователем.

Этот шаблон включает созданный ранее корпоративный шаблон и учитывает все рекомендации и вспомогательные компоненты, выявленные при создании функций общения, в том числе: основные намерения беседы, интеграция со средством Dispatch, QnA Maker, Application Insights и автоматизированное развертывание. Он значительно упрощает создание проектов ботов.

Мы уверены, что наши клиенты должны иметь все возможности для оптимизации работы и получения необходимых сведений. Виртуальный помощник предоставляет клиентам и партнерам возможность управления взаимодействием с пользователями через открытый код на GitHub. Имя, голос и личность помощника можно изменить в соответствии с потребностями организации. Используя наше решение, вы можете без труда создать собственного виртуального помощника за считаные минуты, а затем расширить его возможности с помощью наших комплексных средств разработки.

Виртуальный помощник — это многофункциональное средство, предоставляющее пользователям широкий ряд возможностей. Чтобы ускорить процесс разработки и обеспечить активную экосистему многократно используемых решений для ведения диалога, мы предоставляем разработчикам начальные примеры многократно используемых навыков ведения диалога. Эти навыки можно добавить в приложение для общения, чтобы оптимизировать некоторые возможности, включая поиск точки интереса, а также взаимодействие с календарем, задачами, электронной почтой и многие другие сценарии. Навыки можно настраивать. Они включают языковые модели для нескольких языков, диалоги и код.

![Схема виртуального помощника](./media/enterprise-template/customassistantdiagram.jpg)

## <a name="getting-started"></a>Приступая к работе

Изучите [документацию по виртуальному помощнику и навыкам](https://aka.ms/bf-solutions-docs), чтобы получить дополнительные сведения.

## <a name="whats-in-the-box"></a>Что включает в себя помощник 

Шаблон виртуального помощника объединяет лучшие методики, определенные нами при создании решений для ведения диалога, и автоматизирует интеграцию компонентов, которые мы считаем очень полезными для разработчиков, работающих с платформой Bot Framework. В этом разделе содержатся общие сведения о ключевых решениях, которые помогут понять, почему шаблон работает именно так, а не иначе.

Шаблон виртуального помощника включает все возможности созданного ранее корпоративного шаблона, включая базовый набор намерений для реализации общения на нескольких языках, средство Dispatch, QnA и анализ беседы. Ниже перечислены доступные возможности помощника. Мы планируем пополнять этот список, тесно сотрудничая с клиентами и партнерами в вопросах информирования о стратегии развития.

Компонент | Description |
------------ | -------------
Переход | Пример процесса начала общения, который позволяет помощнику отправить приветственное сообщение пользователю и собирать начальные сведения.
Архитектура службы обработки событий | В контексте виртуального помощника события обеспечивают его размещение в клиентском приложении (в веб-браузере или на устройстве, например в системе автомобиля или устройстве громкой связи). Это позволяет обмениваться информацией о событиях пользователя или устройства и получать события для выполнения операций на устройстве.
Связанные учетные записи | В сценарии общения пользователю нецелесообразно указывать свое имя и пароль для вспомогательных систем с помощью голосовых команд. Поэтому предоставляются отдельные вспомогательные функции, с помощью которых пользователь может выполнять вход и предоставлять виртуальному помощнику разрешения на получение маркеров для последующего использования.
Включение навыков | Сейчас доступно множество базовых функций, благодаря чему разработчики могут создавать собственные решения. Наше решение виртуального помощника включает новую функцию "Навык", которая позволяет подключать к личному помощнику дополнительные возможности путем изменения конфигурации. Также решение предоставляет механизм аутентификации для навыков, чтобы запрашивать маркеры для последующих действий.
Навык для поиска точки интереса | В предварительной версии этого навыка предоставляется комплексная языковая модель для поиска точек интереса (POI) и запроса направлений. Сейчас для этого навыка поддерживается интеграция со службой Azure Maps.
Навык взаимодействия с календарем | В предварительной версии этого навыка доступна комплексная языковая модель для стандартных действий с календарем. Сейчас для этого навыка поддерживается интеграция с Microsoft Graph (Office 365 и Outlook.com). В будущем мы реализуем поддержку API Google.
Навык взаимодействия с электронной почтой | В предварительной версии этого навыка предоставляется комплексная языковая модель для стандартных действий с электронной почтой. Сейчас для этого навыка поддерживается интеграция с Microsoft Graph (Office 365 и Outlook.com). В будущем мы реализуем поддержку API-интерфейсов Google.
Навык взаимодействия со списком дел | В предварительной версии этого навыка доступна комплексная языковая модель для стандартных действий с задачами. Сейчас для этого навыка поддерживается интеграция с OneNote. В будущем мы реализуем возможность интеграции с Microsoft Graph (outlookTask).
Интеграция с устройствами | Наши пакеты SDK службы Azure Bot (DirectLine) и службы "Речь", а также адаптивные карточки обеспечивают простую кроссплатформенную интеграцию с устройствами. Мы планируем предоставить примеры интеграции, а также поддержку таких платформ, как Edge.
Окружения теста | Кроме Bot Framework Emulator предоставляется окружение теста на основе веб-чата, что позволяет тестировать более сложные сценарии аутентификации. В простом окружении теста на основе консоли применяется подход к обмену сообщениями, демонстрирующий простоту интеграции с устройством.
Автоматизированное развертывание | Все ресурсы Azure, необходимые для работы помощника, развертываются автоматически: регистрация бота, Служба приложений Azure, LUIS, QnAMaker, Content Moderator, CosmosDB, служба хранилища Azure и Application Insights. Кроме того, создаются, обучаются и публикуются модели LUIS для навыков, QnAMaker и модели Dispatch, чтобы обеспечить оперативное тестирование.
Языковая модель для автомобильной промышленности | В ближайшее время появится языковая модель для автомобильной промышленности, которая охватывает такие области, как телефонная связь, а также система навигации и управления функциями в автомобиле.

## <a name="example-scenarios"></a>Примеры сценариев

Виртуальный помощник поддерживает сценарии для множества отраслей. Ниже для справки приведены некоторые примеры сценариев.

- **Автомобильная промышленность.** Личный помощник с поддержкой голосовых функций, интегрированный в систему автомобиля, позволяет пользователям выполнять в автомобиле обычные действия, например использовать навигацию или слушать радио. Наряду с этим они могут выполнять операции, связанные с личной эффективностью: переносить встречи в случае опоздания, добавлять элементы в список задач и предпринимать упреждающие меры, при которых автомобиль может предложить выполнить задачи, связанные с определенными событиями, например запуском мотора, поездкой домой или включением системы круиз-контроля. Адаптивные карточки отображаются на головном устройстве, а интеграция речи выполняется при нажатии клавиши или с использованием слова для активации.

- **Гостиничный бизнес.** Личный помощник с поддержкой голосовых функций интегрирован в устройство в номере отеля и предназначен для выполнения широкого ряда сценариев, характерных для гостиничного бизнеса (например, продление проживания в отеле, запрос позднего выселения из номера, обслуживание номеров), в том числе функций консьержа, а также поиска местных ресторанов и других достопримечательностей. При желании помощник можно связать с учетными записями для оптимизации производительности. Это позволит применять дополнительные настраиваемые функции, например сигналы будильника, предупреждения о погоде и возможность изучения необходимой информации во время пребывания в отелях. Развитие технологий телевещания позволяет персонализировать средства взаимодействия в каждом номере.

- **Корпоративные решения**. Фирменный помощник сотрудника с поддержкой голосовых и текстовых функций на корпоративных устройствах и в существующих холстах диалога (например, Teams, WebChat, Slack). Сотрудники могут управлять своими календарями, находить доступные помещения для проведения встреч, находить людей с определенными навыками и выполнять операции, связанные с кадровыми вопросами.

## <a name="virtual-assistant-principles"></a>Принципы работы виртуального помощника

### <a name="your-data-your-brand-and-your-experience"></a>Ваши данные, торговая марка и возможности
Все аспекты взаимодействия с пользователем контролируются вами, включая фирменную символику, имя, голос, личность помощника, его ответы и аватар. Исходный код виртуального помощника и вспомогательные навыки предоставляются в полном объеме, чтобы вы могли настраивать их по своему усмотрению.

Ваш виртуальный помощник будет развернут в подписке Azure. Это значит, что все создаваемые помощником данные (задаваемые вопросы и ответы, сведения о поведении пользователей и т. д.) также содержатся в подписке Azure. См. о [надежном облаке Azure служб Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) и [центре управления безопасностью Azure](https://www.microsoft.com/TrustCenter/CloudServices/Azure).

### <a name="write-it-once-embed-it-anywhere"></a>Напишите код один раз и внедряйте его в любую программу
Виртуальный помощник использует платформу искусственного интеллекта Майкрософт для общения, следовательно, его можно предоставить через любой [канал](https://docs.microsoft.com/azure/bot-service/bot-service-manage-channels?view=azure-bot-service-4.0) Bot Framework.

Кроме того, с помощью канала [Direct Line](https://docs.microsoft.com/azure/bot-service/rest-api/bot-framework-rest-direct-line-3-0-concepts?view=azure-bot-service-4.0) можно внедрить средства взаимодействия в классические и мобильные приложения, а также такие устройства, как автомобили, динамики, будильники и т. д.

### <a name="enterprise-grade-solutions"></a>Решения корпоративного уровня
В основе виртуального помощника лежат такие службы, как Azure Bot, "Распознавание речи" (Cognitive Service), единая служба "Речь", а также множество вспомогательных компонентов Azure. Это означает, что вы можете использовать преимущества [глобальной инфраструктуры Azure](https://azure.microsoft.com/global-infrastructure/), включая сертификацию ISO 27018, HIPPA, PCI DSS и SOC 1, 2, 3.

Кроме того, поддержка функций распознавания речи обеспечивается службой LUIS Cognitive Service, которая поддерживает [ряд языков](https://docs.microsoft.com/azure/cognitive-services/luis/luis-supported-languages). [Translator Cognitive Service](https://azure.microsoft.com/services/cognitive-services/translator-text-api/) предоставляет дополнительные функции машинного перевода, еще больше расширяя возможности виртуального помощника.

### <a name="integrated-and-context-aware"></a>Интеграция с учетом контекста
Ваш виртуальный помощник можно интегрировать с устройством и экосистемой, создав действительно целостное и интеллектуальное решение. Эта поддержка контекста позволяет расширить возможности интеллектуального взаимодействия за счет персонализации.

### <a name="3rd-party-assistant-integration"></a>Интеграция с помощником от стороннего производителя
Виртуальный помощник позволяет выстраивать уникальное взаимодействие по вашему усмотрению. Но для некоторых типов вопросов пользователям также предоставляется выбранный цифровой помощник.

### <a name="flexible-integration"></a>Гибкая интеграция
Наш виртуальный помощник имеет гибкую архитектуру: его можно интегрировать с используемыми на устройствах функциями обработки речи или естественного языка, а также существующими серверными системами и API.

### <a name="adaptive-cards"></a>Адаптивные карточки
Благодаря [адаптивным карточкам](https://adaptivecards.io/) виртуальный помощник может возвращать не только ответы на основе текста, но и элементы пользовательского интерфейса (например, карточки, изображения, кнопки). Эти адаптивные карточки отображаются на разных устройствах и платформах с экраном или холстом диалога. Это обеспечивает необходимую поддержку пользовательского интерфейса. См. примеры [адаптивных карточек](https://adaptivecards.io/samples/) и сведения о [параметрах отображения](https://docs.microsoft.com/adaptive-cards/rendering-cards/getting-started).

### <a name="skills"></a>Навыки
Кроме помощника разработчик должен самостоятельно создавать целый ряд базовых функций. Все это напрямую связано с производительностью. Организации нужно предусмотреть все аспекты, включая языковые модели (LUIS), диалоги (код), интеграцию (код) и генерирование текста (ответы), чтобы обеспечить взаимодействие с календарем, электронной почтой или задачами.

Но это не все, ведь еще необходимо реализовать поддержку нескольких языков. Все это приводит к тому, что организация, занимающаяся созданием своего помощника, сталкивается с большим объемом работы.

Наш виртуальный помощник включает новую функцию "Навык", которая позволяет подключать к личному помощнику новые дополнительные возможности путем изменения конфигурации. 

Все аспекты каждого навыка (языковая модель, диалоги, интеграция, код и генерирование текстов) полностью настраиваются разработчиками, так как полный исходный код доступен в GitHub вместе с виртуальным помощником.

## <a name="getting-started"></a>Приступая к работе

См. о [создании и развертывании виртуального помощника](https://aka.ms/bfs-tutorials).


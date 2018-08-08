---
title: Устранение неполадок с ботами | Документы Майкрософт
description: Устранение общих проблем в процессе разработки ботов с помощью ответов на часто задаваемые технические вопросы.
author: DeniseMak
ms.author: v-demak
manager: kamrani
ms.topic: article
ms.prod: bot-framework
ms.date: 12/13/2017
ms.openlocfilehash: 648a2e3be901bfa82d84423358fa7df32d403391
ms.sourcegitcommit: f576981342fb3361216675815714e24281e20ddf
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 07/18/2018
ms.locfileid: "39305567"
---
# <a name="troubleshooting-general-problems"></a>Устранение общих проблем
Ответы на эти часто задаваемые вопросы помогут вам устранить распространенные проблемы, связанные с разработкой или эксплуатацией ботов.

## <a name="how-can-i-troubleshoot-issues-with-my-bot"></a>Как можно устранить неполадки с ботом?

1. Выполните отладку исходного кода бота в [Visual Studio Code](debug-bots-locally-vscode.md) или Visual Studio.
2. Перед развертыванием в облаке протестируйте бот с помощью [эмулятора](bot-service-debug-emulator.md).
3. Разверните бот на облачной платформе размещения, например Azure, а затем проверьте возможность подключения к боту с помощью встроенного элемента управления веб-чата на панели мониторинга бота на <a href="https://dev.botframework.com" target="_blank">портале Bot Framework</a>. При возникновении проблем с ботом после его развертывания в Azure рекомендуется обратиться к руководству [Устранение неполадок веб-приложения в службе приложений Azure с помощью Visual Studio](https://azure.microsoft.com/en-us/documentation/articles/web-sites-dotnet-bot-service-troubleshoot-visual-studio/).
4. Исключите [проверку подлинности][TroubleshootingAuth] как возможную проблему.
5. Проверьте бот в Skype. Это поможет проверить полное взаимодействие с пользователем.
6. Протестируйте бот в каналах с требованиями дополнительной проверки подлинности, таких как Direct Line или веб-чат.

## <a name="how-can-i-troubleshoot-authentication-issues"></a>Как устранить неполадки, связанные с проверкой подлинности?

Дополнительные сведения об устранении неполадок бота, связанных с проверкой подлинности, см. в статье [Устранение неполадок проверки подлинности Bot Framework][TroubleshootingAuth].

## <a name="im-using-the-bot-builder-sdk-for-net-how-can-i-troubleshoot-issues-with-my-bot"></a>Я использую пакет SDK Bot Builder для .NET. Как можно устранить неполадки с ботом?

**Просмотрите исключения.**  
В Visual Studio 2017 последовательно выберите **Отладка** > **Windows** > **Параметры исключений**. В окне **Параметры исключений** установите флажок **Break When Thrown** (Прерывать при возникновении) рядом с полем **Исключения среды CLR**. При наличии возникших или необработанных исключений в окне выходных данных могут отображаться результаты диагностики.

**Просмотрите стек вызовов.**  
В Visual Studio можно выбрать вариант отладки [Только мой код](https://msdn.microsoft.com/en-us/library/dn457346.aspx). После изучения всего стека вызовов вы сможете глубже понять любую проблему.

**Убедитесь, что все методы диалога заканчиваются планом по обработке следующего сообщения.**  
Все методы `IDialog` должны завершаться `IDialogStack.Call`, `IDialogStack.Wait` или `IDialogStack.Done`. Эти методы `IDialogStack` предоставляются через `IDialogContext`, передаваемый в каждый метод `IDialog`. При вызове `IDialogStack.Forward` и использовании системных запросов с помощью статических методов `PromptDialog` будет вызван один из этих методов в их реализации.

**Убедитесь, что все диалоги сериализуемые.**  
Для этого можно просто использовать атрибут `[Serializable]` в реализациях `IDialog`. Однако имейте в виду, что замыкания анонимного метода не являются сериализуемыми, если для записи переменных они ссылаются на свою внешнюю среду. Для сериализации типов, которые не помечены как сериализуемые, Bot Framework поддерживает суррогат сериализации на основе отражения.

## <a name="why-doesnt-the-typing-activity-do-anything"></a>Почему действие ввода с клавиатуры не дает никаких результатов?
Некоторые каналы не поддерживают временные обновления ввода с клавиатуры.

## <a name="what-is-the-difference-between-the-connector-library-and-builder-library-in-the-sdk"></a>Чем отличается библиотека соединителя от библиотеки построителя в пакете SDK?

Библиотека соединителя является представлением REST API. Библиотека построителя добавляет модель программирования диалогов и другие функции и компоненты, такие как запросы, каскадные модели, цепочки и управляемое заполнение форм. Библиотека построителя также предоставляет доступ к Cognitive Services, например LUIS.

## <a name="what-causes-an-error-with-http-status-code-429-too-many-requests"></a>Почему возникает ошибка с кодом состояния HTTP 429 "Слишком много запросов"?

Сообщение об ошибке с кодом состояния HTTP 429 указывает, что в течение заданного промежутка времени было отправлено слишком много запросов. Текст ответа должен содержать описание проблемы. В нем также может указываться минимальный обязательный интервал между запросами. Одной из возможных причин этой ошибки является [ngrok](https://ngrok.com/). Если вы используете бесплатный план и достигли пределов ngrok, перейдите на веб-страницу с ценами и ограничениями, чтобы изучить дополнительные [варианты](https://ngrok.com/product#pricing). 
 

## <a name="how-can-i-run-background-tasks-in-aspnet"></a>Как выполнять фоновые задачи в ASP.NET? 

В некоторых случаях может потребоваться запустить асинхронную задачу, которая в течение нескольких секунд находится в режиме ожидания, а затем выполняет код для очистки профиля пользователя или сброса состояния диалога или беседы. Дополнительные сведения о том, как это сделать, см. в статье о [выполнении фоновых задач в ASP.NET](https://www.hanselman.com/blog/HowToRunBackgroundTasksInASPNET.aspx). В частности, рассмотрите возможность использования [HostingEnvironment.QueueBackgroundWorkItem](https://msdn.microsoft.com/en-us/library/dn636893(v=vs.110).aspx). 


## <a name="how-do-user-messages-relate-to-https-method-calls"></a>Каким образом сообщения пользователя связаны с вызовами методов HTTPS?

Когда пользователь отправляет сообщение по каналу, веб-служба Bot Framework отправляет запрос HTTPS POST в конечную точку веб-службы бота. По этому каналу бот может отправлять пользователю одно или много сообщений либо не отправлять ни одного, выполняя отдельный запрос HTTPS POST в Bot Framework для каждого отправляемого сообщения.

## <a name="my-bot-is-slow-to-respond-to-the-first-message-it-receives-how-can-i-make-it-faster"></a>Мой бот медленно реагирует на первое получаемое сообщение. Как повысить его быстродействие?

Боты представляют собой веб-службы, и некоторые платформы размещения, включая Azure, автоматически вводят службу в спящий режим, если она не получает трафик в течение определенного периода времени. Если это происходит с вашим ботом, его необходимо перезапустить с нуля при очередном получении сообщения, значительно замедляющего его ответ по сравнению с тем, как если бы он был уже запущен.

Некоторые платформы для размещения позволяют настроить службу таким образом, чтобы ее было невозможно перевести в спящий режим. Чтобы сделать это в Azure, перейдите к службе бота на [портале Azure](https://portal.azure.com), выберите **Параметры приложения**, а затем — **Always on** (Всегда включено). Этот параметр доступен в большинстве, но не во всех планах обслуживания.

## <a name="how-can-i-guarantee-message-delivery-order"></a>Как гарантировать определенный порядок доставки сообщений?

По возможности Bot Framework будет поддерживать порядок доставки сообщений. Например, если вы отправляете сообщение А и ждете завершения этой операции HTTP прежде чем начать другую операцию HTTP по отправке сообщения Б, Bot Framework автоматически распознает, что сообщение А должно предшествовать сообщению Б. Однако в целом порядок доставки сообщений не гарантируется, так как канал несет полную ответственность за доставку сообщений и может изменить их порядок. Чтобы снизить риск доставки сообщений в неверном порядке, можно реализовать временную задержку между сообщениями.

## <a name="how-can-i-intercept-all-messages-between-the-user-and-my-bot"></a>Каким образом можно перехватывать все сообщения между пользователем и моим ботом?

С помощью пакета SDK Bot Builder для .NET можно предоставлять реализации интерфейсов `IPostToBot` и `IBotToUser` для контейнера внедрения зависимостей `Autofac`. Для этой же цели можно использовать ПО промежуточного слоя из пакета SDK Bot Builder для Node.js. Репозиторий [BotBuilder-Azure](https://github.com/Microsoft/BotBuilder-Azure) содержит библиотеки C# и Node.js, которые будут регистрировать эти данные в таблице Azure.

## <a name="why-are-parts-of-my-message-text-being-dropped"></a>Почему отбрасываются части текста сообщения?

Bot Framework и многие каналы интерпретируют текст так, как если бы он был отформатирован с помощью [Markdown](https://en.wikipedia.org/wiki/Markdown). Проверьте, содержит ли текст символы, которые могут быть интерпретированы как синтаксис Markdown.

## <a name="how-can-i-support-multiple-bots-at-the-same-bot-service-endpoint"></a>Каким образом можно поддерживать несколько ботов в одной конечной точке службы ботов? 

В этом [примере](https://github.com/Microsoft/BotBuilder/issues/2258#issuecomment-280506334) показана настройка `Conversation.Container` с правильным классом `MicrosoftAppCredentials` и использование простого `MultiCredentialProvider` для проверки подлинности нескольких идентификаторов приложений и паролей.

## <a name="identifiers"></a>"Identifiers" (Идентификаторы)

## <a name="how-do-identifiers-work-in-the-bot-framework"></a>Как работают идентификаторы в Bot Framework?

Дополнительные сведения об идентификаторах в Bot Framework см. в [руководстве по идентификаторам Bot Framework][BotFrameworkIDGuide].

## <a name="how-can-i-get-access-to-the-user-id"></a>Как получить доступ к идентификатору пользователя?

SMS-сообщения и сообщения электронной почты предоставляют неформатированный идентификатор пользователя в свойстве `from.Id`. В сообщениях Skype свойство `from.Id` будет содержать уникальный идентификатор пользователя, который отличается от идентификатора пользователя Skype. Чтобы подключиться к существующей учетной записи, можно использовать карточку для входа и реализовать собственный поток OAuth для подключения идентификатора пользователя к своему ИД пользователя службы.

## <a name="why-are-my-facebook-user-names-not-showing-anymore"></a>Почему имена пользователей Facebook больше не отображаются?

Вы меняли пароль для Facebook? Если да, маркер доступа будет недействителен, и вам потребуется обновить параметры конфигурации бота для канала Facebook Messenger на <a href="https://dev.botframework.com" target="_blank">портале Bot Framework</a>.

## <a name="why-is-my-kik-bot-replying-im-sorry-i-cant-talk-right-now"></a>Почему бот Kik отвечает: "К сожалению, я не могу сейчас говорить"?

Боты для Kik поддерживаются для 50 подписчиков. После того как с ботом пообщалось 50 уникальных пользователей, любой новый пользователь, который пытается завести разговор с ботом, получит сообщение "К сожалению, я не могу сейчас говорить". Дополнительные сведения см. в [документации по Kik](https://botsupport.kik.com/hc/en-us/articles/225764648-How-can-I-share-my-bot-with-Kik-users-while-in-development-).

## <a name="how-can-i-use-authenticated-services-from-my-bot"></a>Как использовать прошедшие проверку подлинности службы из бота?

Для проверки подлинности Azure Active Directory рекомендуется использовать библиотеку [BotAuth NuGet](https://www.nuget.org/packages/BotAuth). Примеры проверки подлинности в Facebook см. на странице [примеров для пакета SDK Bot Builder для .NET](https://github.com/Microsoft/BotBuilder/tree/master/CSharp/Samples) на сайте GitHub. 

> [!NOTE] 
> При добавлении в бот функций проверки подлинности и безопасности следует убедиться, что шаблоны, реализуемые в коде, соответствуют стандартам безопасности, которые подходят для вашего приложения.

## <a name="how-can-i-limit-access-to-my-bot-to-a-pre-determined-list-of-users"></a>Как можно предоставить доступ к боту только предопределенному списку пользователей?

Некоторые каналы, такие как SMS и электронная почта, предоставляют неограниченный диапазон адресов. В этих случаях сообщения от пользователя будут содержать неформатированный идентификатор пользователя в свойстве `from.Id`.

Другие каналы, например Skype, Facebook и Slack, предоставляют адреса с областью действия или клиентские адреса таким образом, чтобы не позволить боту заранее прогнозировать идентификатор пользователя. В этих случаях будет необходимо проверить подлинность пользователя через ссылку для входа или общий секрет, чтобы определить, авторизованы ли они для использования бота.

## <a name="why-does-my-direct-line-11-conversation-start-over-after-every-message"></a>Почему диалог Direct Line 1.1 начинается заново после каждого сообщения?

Если диалог Direct Line начинается заново после каждого сообщения, скорее всего, свойство `from` отсутствует или имеет значение `null` в сообщениях, которые клиент Direct Line отправил боту. Когда клиент Direct Line отправляет сообщение с отсутствующим или имеющим значение `null` свойством `from`, служба Direct Line автоматически выделяет идентификатор, поэтому каждое отправляемое клиентом сообщение будет считаться сообщением от нового пользователя.

Чтобы устранить эту проблему, задайте свойству `from` в каждом сообщении, отправленном клиентом Direct Line, постоянное значение, однозначно представляющее пользователя, который отправляет сообщение. Например, если пользователь уже выполнил вход на веб-страницу или в приложение, этот существующий идентификатор можно использовать в качестве значения свойства `from` в сообщениях, отправляемых пользователем. Кроме того, можно создать произвольный идентификатор пользователя при загрузке страницы или загрузке приложения, сохранить этот идентификатор в файле cookie или состоянии устройства и использовать его в качестве значения свойства `from` в сообщениях, отправляемых пользователем.

## <a name="what-causes-the-direct-line-30-service-to-respond-with-http-status-code-502-bad-gateway"></a>По какой причине служба Direct Line 3.0 отвечает с кодом состояния HTTP 502 "Недопустимый шлюз"?
Direct Line 3.0 возвращает код состояния HTTP 502 при попытке связаться с ботом, но запрос завершается ошибкой. Эта ошибка означает, что либо бот вернул ошибку, либо истекло время ожидания запроса. Чтобы получить дополнительные сведения об ошибках, возникающих в боте, перейдите к панели мониторинга бота на <a href="https://dev.botframework.com" target="_blank">портале Bot Framework</a> и щелкните ссылку "Проблемы" для затронутых каналов. Если для бота настроена служба Application Insights, в ней вы также сможете найти подробные сведения об ошибке. 

## <a name="what-is-the-idialogstackforward-method-in-the-bot-builder-sdk-for-net"></a>Что собой представляет метод IDialogStack.Forward в пакете SDK Bot Builder для .NET?

Основная цель метода `IDialogStack.Forward` заключается в повторном использовании существующего дочернего диалога, который часто является реактивным, — дочерний диалог (в `IDialog.StartAsync`) ожидает объект `T` с обработчиком `ResumeAfter`. В частности, при наличии дочернего диалога, который ожидает `IMessageActivity` `T`, можно перенаправить входящий интерфейс `IMessageActivity` (уже полученный определенным родительским диалогом) с помощью метода `IDialogStack.Forward`. Например, чтобы перенаправлять входящий интерфейс `IMessageActivity` в `LuisDialog`, вызовите `IDialogStack.Forward` для отправки `LuisDialog` в стек диалога, выполните код в `LuisDialog.StartAsync` до его планирования ожидания следующего сообщения и затем сразу же удовлетворите это ожидание с помощью перенаправленного интерфейса `IMessageActivity`.

Как правило, объект `T` является интерфейсом `IMessageActivity`, так как для ожидания этого типа действия обычно создается `IDialog.StartAsync`. Можно использовать `IDialogStack.Forward` в `LuisDialog` в качестве механизма для перехвата сообщений от пользователя для обработки перед пересылкой сообщения в существующий `LuisDialog`. Кроме того, для этой цели можно также использовать `DispatchDialog` с `ContinueToNextGroup`.

Обычно перенаправленный элемент находится в первом обработчике `ResumeAfter` (например, `LuisDialog.MessageReceived`), запланированном `StartAsync`.

## <a name="what-is-the-difference-between-proactive-and-reactive"></a>Какова разница между понятиями "упреждающий" и "реагирующий"?

С точки зрения бота "реагирующий" означает, что пользователь начинает диалог, отправляя сообщения боту, а бот реагирует, отвечая на это сообщение. Напротив, "упреждающий" означает, что бот начинает диалог, отправляя первое сообщение пользователю. Например, бот может отправить упреждающее сообщение, чтобы уведомить пользователя об окончании срока действия или возникновении события.

## <a name="how-can-i-send-proactive-messages-to-the-user"></a>Как отправить упреждающее сообщение пользователю?

Примеры отправки упреждающих сообщений см. на странице [примеров на C#](https://github.com/Microsoft/BotBuilder-Samples/tree/master/CSharp/core-proactiveMessages) и [примеров Node.js](https://github.com/Microsoft/BotBuilder-Samples/tree/master/Node/core-proactiveMessages) в репозитории BotBuilder-Samples на сайте GitHub.

## <a name="how-can-i-reference-non-serializable-services-from-my-c-dialogs"></a>Каким образом можно ссылаться на несериализуемые службы из диалогов C#?

Есть несколько вариантов.

* Разрешить зависимость посредством `Autofac` и `FiberModule.Key_DoNotSerialize`. Это самое четкое решение.
* Использовать атрибуты [NonSerialized](https://msdn.microsoft.com/en-us/library/system.nonserializedattribute(v=vs.110).aspx) и [OnDeserialized](https://msdn.microsoft.com/en-us/library/system.runtime.serialization.ondeserializedattribute(v=vs.110).aspx) для восстановления зависимости во время десериализации. Это самое простое решение.
* Не сохранять эту зависимость, чтобы исключить ее сериализуемость. Хотя это решение технически выполнимо, применять его не рекомендуется.
* Использование суррогата сериализации на основе отражения. В некоторых случаях это решение может быть нецелесообразным. Кроме того, слишком высоки риски сериализации.

## <a name="where-is-conversation-state-stored"></a>Где хранится состояние диалога?

Данные в наборах свойств пользователя, диалога и частного диалога хранятся с использованием интерфейса `IBotState` соединителя. Каждый набор свойств ограничен идентификатором бота. Набор свойств пользователя помечается ключом идентификатора пользователя, набор свойств диалога помечается ключом идентификатора диалога и набор свойств частного диалога помечается ключами идентификатора пользователя и идентификатора диалога. 

Если для создания бота используется пакет SDK Bot Builder для .NET или пакет SDK Bot Builder для Node.js, стек диалога и данные диалога автоматически сохраняются как записи в наборе свойств частного диалога. В реализации C# применяется двоичная сериализация, а в реализации Node.js — сериализация JSON.

Интерфейс REST `IBotState` реализуется двумя службами.

* Bot Framework Connector предоставляет облачную службу, которая реализует этот интерфейс и сохраняет данные в Azure.  Эти данные шифруются при хранении и намеренно не имеют окончания срока действия.
* Bot Framework Emulator предоставляет реализацию этого интерфейса в памяти для отладки бота. Срок действия данных истекает при завершении процесса эмулятора.

Чтобы хранить эти данные в центрах обработки данных, предоставьте пользовательскую реализацию службы состояния. Это можно сделать по меньшей мере двумя способами.

* Использовать уровень REST для предоставления настраиваемой службы `IBotState`.
* Использовать интерфейсы построителя на уровне языка (Node.js или C#).

> [!IMPORTANT]
> API службы состояния Bot Framework не рекомендуется использовать для рабочей среды. Он может считаться устаревшим в будущем выпуске. Рекомендуется обновить код бота для использования хранилища в памяти в целях тестирования или использовать одно из **расширений Azure** для ботов в рабочей среде. Дополнительные сведения см. в разделе **Управление данными состояния** в документации по реализации [.NET](~/dotnet/bot-builder-dotnet-state.md) или [Node](~/nodejs/bot-builder-nodejs-state.md).

## <a name="what-is-an-etag--how-does-it-relate-to-bot-data-bag-storage"></a>Что такое ETag?  Как он связан с хранилищем набора данных бота?

[ETag](https://en.wikipedia.org/wiki/HTTP_ETag) — это механизм для [управления оптимистичным параллелизмом](https://en.wikipedia.org/wiki/Optimistic_concurrency_control). Хранилище набора данных бота использует теги ETag для предотвращения применения конфликтующих обновлений к данным. Ошибка ETag с кодом состояния HTTP 412 "Необходимое условие не выполнено" означает, что для этого набора данных бота одновременно выполнялось несколько последовательностей чтения, изменения и записи.

Состояние и стек диалога хранятся в наборах данных бота. Например, ошибка ETag "Необходимое условие не выполнено" может возникнуть, если при получении нового сообщения для этого диалога бот все еще обрабатывает предыдущее сообщение.

## <a name="what-causes-an-error-with-http-status-code-412-precondition-failed-or-http-status-code-409-conflict"></a>Почему возникает ошибка с кодом состояния HTTP 412 "Необходимое условие не выполнено" или с кодом состояния HTTP 409 "Конфликт"?

Служба `IBotState` соединителя используется для хранения наборов данных бота (т. е. наборов данных пользователя, диалога и частных данных бота, где набор частных данных бота содержит состояние "поток управления" стека диалога). Управление параллелизмом в службе `IBotState` осуществляет механизм оптимистичного параллелизма посредством тегов ETag. Если во время последовательности чтения, изменения и записи возникает конфликт обновления (из-за одновременного обновления набора данных одного бота), происходит следующее:

* если теги ETag сохраняются, в службе `IBotState` возникает ошибка с кодом состояния HTTP 412 "Необходимое условие не выполнено". Это поведение по умолчанию в пакете SDK Bot Builder для .NET.
* если теги ETag не сохраняются (т. е. ETag имеет значение `\*`), вступает в силу политика приоритета последней записи, которая исключает возникновение ошибки "Необходимое условие не выполнено", но приводит к риску потери данных. Это поведение по умолчанию в пакете SDK Bot Builder для Node.js.

## <a name="how-can-i-fix-precondition-failed-412-or-conflict-409-errors"></a>Как можно устранить ошибки "Необходимое условие не выполнено" (412) или "Конфликт" (409)?

Эти ошибки указывают, что бот за один раз обрабатывал несколько сообщений для одного диалога. Если бот подключен к службам, которым требуются точно упорядоченные сообщения, рекомендуется заблокировать состояние диалога, чтобы исключить параллельную обработку сообщений. В пакете SDK Bot Builder для .NET доступен механизм (класс `LocalMutualExclusion`, реализующий `IScope`) для пессимистической сериализации обработки отдельных диалогов с помощью семафора в памяти. Эту реализацию можно расширить за счет использования аренды Redis, ограниченной адресом диалога.

Если бот не подключен к внешним службам или если допускается параллельная обработка сообщений из одного диалога, можно добавить этот код, чтобы игнорировать конфликты, возникающие в API службы состояния бота. В этом случае последний ответ будет задавать состояние диалога.

```cs
var builder = new ContainerBuilder();
builder
    .Register(c => new CachingBotDataStore(c.Resolve<ConnectorStore>(), CachingBotDataStoreConsistencyPolicy.LastWriteWins))
    .As<IBotDataStore<BotData>>()
    .AsSelf()
    .InstancePerLifetimeScope();
builder.Update(Conversation.Container);
```

## <a name="is-there-a-limit-on-the-amount-of-data-i-can-store-using-the-state-api"></a>Есть ли ограничение на объем данных, который можно сохранять с помощью API состояния?

Да, каждое хранилище состояний (т. е. набор данных пользователя, диалога и частных данных бота) может содержать до 64 КБ данных. Дополнительные сведения см. в разделе [Управление данными состояния][StateAPI].

## <a name="how-do-i-version-the-bot-data-stored-through-the-state-api"></a>Как назначить версию данным бота, сохраненным с помощью API состояния?

> [!IMPORTANT]
> API службы состояния Bot Framework не рекомендуется использовать для рабочей среды. Он может считаться устаревшим в будущем выпуске. Рекомендуется обновить код бота для использования хранилища в памяти в целях тестирования или использовать одно из **расширений Azure** для ботов в рабочей среде. Дополнительные сведения см. в разделе **Управление данными состояния** в документации по реализации [.NET](~/dotnet/bot-builder-dotnet-state.md) или [Node](~/nodejs/bot-builder-nodejs-state.md).

Служба состояния позволяет сохранять ход беседы с помощью диалогов, чтобы пользователь мог вернуться к разговору с ботом без потери места, в котором была сделана остановка. В этом случае наборы свойств данных бота, сохраненные с помощью API состояния, не удаляются автоматически при изменении кода бота. Учитывая совместимость измененного кода с прежними версиями данных, нужно принять решение о необходимости удаления данных бота. 

* Чтобы вручную сбросить стек и состояние диалога в беседе во время разработки бота, используйте команду ` /deleteprofile` для удаления данных о состоянии. Обязательно включите в команду начальный пробел, чтобы запретить каналу ее интерпретацию.
* После развертывания бота в рабочей среде можно назначить версию его данным, чтобы в случае изменения версии все связанные данные о состоянии удалялись. При работе с пакетом SDK Bot Builder для Node.js это можно сделать с помощью ПО промежуточного слоя, а при работе с пакетом SDK Bot Builder для .NET — с помощью реализации `IPostToBot`.

> [!NOTE]
> Если стек диалога не удается правильно десериализовать из-за изменений формата сериализации или слишком серьезного изменения кода, состояние беседы будет сброшено.

## <a name="what-are-the-possible-machine-readable-resolutions-of-the-luis-built-in-date-time-duration-and-set-entities"></a>Какие возможные обрабатываемые компьютером решения существуют для встроенных сущностей даты, времени, длительности и установки в службе LUIS?

Список примеров см. в [разделе о предварительно созданных сущностях][LUISPreBuiltEntities] документации по LUIS.

## <a name="how-can-i-use-more-than-the-maximum-number-of-luis-intents"></a>Что можно сделать, если требуемое количество намерений LUIS превышает максимально допустимое значение?

Возможно, потребуется разделить модель и вызывать службу LUIS циклами или параллельно.

## <a name="how-can-i-use-more-than-one-luis-model"></a>Каким образом можно использовать несколько моделей LUIS?

Пакет SDK Bot Builder для Node.js и пакетом SDK Bot Builder для .NET поддерживают возможность вызова нескольких моделей LUIS из одного диалога намерений LUIS. Учитывайте следующие моменты.

* При использовании нескольких моделей LUIS предполагается, что модели LUIS имеют неперекрывающийся набор намерений.
* При использовании нескольких моделей LUIS предполагается, что оценки из различных моделей являются сопоставимыми, и это необходимо учитывать при выборе максимально подходящего намерения в нескольких моделях.
* Использование нескольких моделей LUIS означает, что если намерение соответствует одной модели, оно будет также строго соответствовать намерению "None" других моделей. Можно избежать выбора намерения "None" в этой ситуации — пакет SDK Bot Builder для Node.js будет автоматически снижать оценку для намерений "None".

## <a name="where-can-i-get-more-help-on-luis"></a>Где можно получить дополнительную помощь по LUIS?

* [Общие сведения о службе "Распознавание речи" (LUIS) — Microsoft Cognitive Services](https://www.youtube.com/watch?v=jWeLajon9M8) (видео)
* [Расширенный сеанс обучения по службе "Распознавание речи" (LUIS) ](https://www.youtube.com/watch?v=39L0Gv2EcSk) (видео)
* [Документация по LUIS](/azure/cognitive-services/LUIS/Home)
* [Форум по службе "Распознавание речи"](https://social.msdn.microsoft.com/forums/azure/en-US/home?forum=LUIS) 


## <a name="what-are-some-community-authored-dialogs"></a>Какие диалоги входят в число созданных сообществом?

* [BotAuth](https://www.nuget.org/packages/BotAuth) — проверка подлинности Azure Active Directory
* [BestMatchDialog](http://www.garypretty.co.uk/2016/08/01/bestmatchdialog-for-microsoft-bot-framework-now-available-via-nuget/) — регулярная основанная на выражениях отправка текста пользователя в методы диалогов

## <a name="what-are-some-community-authored-templates"></a>Какие шаблоны входят в число созданных сообществом?

* [ES6 BotBuilder](https://github.com/brene/botbuilder-es6-template) — шаблон ES6 Bot Builder

## <a name="why-do-i-get-an-authorizationrequestdenied-exception-when-creating-a-bot"></a>Почему при создании бота возникает исключение Authorization_RequestDenied?

Управление разрешениями на создание ботов службы ботов Azure осуществляется на портале Azure Active Directory (AAD). Если на [портале AAD](http://aad.portal.azure.com) неправильно настроены разрешения, при попытке создать службу бота пользователи будут получать исключение **Authorization_RequestDenied**.

Сначала проверьте, являетесь ли вы гостевым пользователем каталога.

1. Войдите на [портал Azure](http://portal.azure.com).
2. Выберите **Все службы** и выполните поиск по слову *активный*.
3. Выберите **Azure Active Directory**.
4. Выберите раздел **Пользователи**.
5. В списке найдите пользователя и убедитесь, что в колонке **Тип пользователя** не указано значение **Гость**.

![Тип пользователя Azure Active Directory](~/media/azure-active-directory/user_type.png)

После проверки, подтверждающей, что вы не являетесь **гостевым пользователем**, нужно убедиться, что пользователи в Active Directory могут создавать службу ботов. Для этого администратору каталога необходимо настроить следующие параметры.

1. Войдите на [портал AAD](http://aad.portal.azure.com). В колонке **Пользователи и группы** выберите **Параметры пользователей**.
2. В разделе **Регистрация приложения** задайте параметру **Пользователи могут регистрировать приложения** значение **Да**. После этого пользователи в вашем каталоге смогут создавать службу ботов.
3. В разделе **Внешние пользователи** задайте параметру **Разрешения для гостей ограничены** значение **Нет**. После этого гостевые пользователи в вашем каталоге смогут создавать службу ботов.

![Центр администрирования Azure Active Directory](~/media/azure-active-directory/admin_center.png)

## <a name="why-cant-i-migrate-my-bot"></a>Почему не удается перенести бот?

Проблемы с миграцией бота могут быть связаны с тем, что бот входит в каталог, отличный от каталога по умолчанию. Попробуйте выполнить следующие действия.

1. Из целевого каталога добавьте нового пользователя (с помощью адреса электронной почты), который не входит в каталог по умолчанию, назначьте пользователю роль участника для подписок, являющихся целью миграции.

2. На [портале разработчика](https://dev.botframework.com) добавьте адрес электронной почты пользователя в качестве совладельца бота, который следует перенести. Затем выполните выход.

3. Войдите на [портал разработчика](https://dev.botframework.com) в качестве нового пользователя и начните процесс миграции бота.

## <a name="where-can-i-get-more-help"></a>Где можно получить дополнительную помощь?

* Используйте сведения из ранее данных ответов на вопросы на сайте [Stack Overflow](https://stackoverflow.com/questions/tagged/botframework) или разместите свои вопросы с помощью тега `botframework`. Обратите внимание, что на Stack Overflow существуют правила для воспроизведения проблемы, например обязательное описательное название, полная и краткая формулировка проблемы и достаточное количество сведений. В данном документе не рассматриваются запросы функций или слишком общие вопросы. Для получения дополнительных сведений новым пользователям необходимо посетить [центр справки Stack Overflow](https://stackoverflow.com/help/how-to-ask).
* Чтобы найти информацию по известным проблемам с пакетом SDK Bot Builder или сообщить о новой проблеме, перейдите на страницу с [проблемами BotBuilder](https://github.com/Microsoft/BotBuilder/issues) на сайте GitHub.
* Используйте сведения в обсуждениях сообщества BotBuilder на сайте [Gitter](https://gitter.im/Microsoft/BotBuilder).




[LUISPreBuiltEntities]: /azure/cognitive-services/luis/pre-builtentities
[BotFrameworkIDGuide]: bot-service-resources-identifiers-guide.md
[StateAPI]: ~/rest-api/bot-framework-rest-state.md
[TroubleshootingAuth]: bot-service-troubleshoot-authentication-problems.md

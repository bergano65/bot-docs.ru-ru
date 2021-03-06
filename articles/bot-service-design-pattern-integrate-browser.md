---
title: Интеграция бота с веб-браузером — Служба Azure Bot
description: Узнайте, как проектировать удобный переход пользователя от бота к веб-браузеру и обратно.
author: matvelloso
ms.author: mateusv
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 12/13/2017
ms.openlocfilehash: 8570873a63a539b4f7c96053aec2ab1a1f615eb9
ms.sourcegitcommit: f8b5cc509a6351d3aae89bc146eaabead973de97
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/09/2020
ms.locfileid: "75792132"
---
# <a name="integrate-your-bot-with-a-web-browser"></a>Интеграция бота с веб-браузером

В некоторых сценариях требуется больше, чем просто бот для выполнения требования. Боту может понадобиться отправить пользователя в веб-браузер для выполнения задачи и возобновить диалог с пользователем после ее завершения. 

## <a name="authentication-and-authorization"></a>Аутентификация и авторизация
Если бот хочет эффективно читать календарь пользователя в Office 365 или возможно даже создавать встречи от имени этого пользователя, тогда пользователь должен сначала пройти проверку подлинности с помощью Microsoft Azure Active Directory и авторизовать бот для доступа к данным календаря. Бот перенаправляет пользователя в веб-браузер для выполнения проверки подлинности и авторизации и впоследствии возобновит диалог c ним. 

## <a name="security-and-compliance"></a>Безопасность и соответствие требованиям
Требования к безопасности и соответствию часто ограничены типом информации, которой могут обмениваться бот с пользователем. В некоторых случаях пользователю может потребоваться отправить или получить данные за рамками текущего диалога. Например, если пользователь хочет выполнить оплату с помощью стороннего поставщика обработчика платежей, то номер кредитной карты не указывается в контексте диалога. Вместо этого бот направит пользователя в веб-браузер для выполнения процесса оплаты и впоследствии возобновит диалог c пользователем.

В этой статье рассматривается процесс упрощения перехода пользователя от бота к веб-браузеру и обратно. 

> [!NOTE]
> Переход от чата к веб-браузеру и обратно — неидеальное решение, как и переключении между приложениями, что может легко запутать пользователя. Чтобы упростить работу, многие каналы предоставляют встроенные окна HTML, которые бот может использовать для представления приложений, без этого они бы отображались в веб-браузере. Этот метод позволяет пользователю оставаться в диалоге при доступе к внешним ресурсам. Этот подход аналогичен мобильным приложениям, управляющим потоками авторизации с помощью OAuth в рамках внедренных веб-представлений.

## <a name="bot-to-web-browser-and-back-again"></a>От бота к веб-браузеру и обратно

Ниже показана общая схема потока для интеграции между ботом и веб-браузером. 

![Взаимодействие бота с Интернетом](~/media/bot-service-design-pattern-integrate-browser/bot-to-web1.png)

Рассмотрим каждое действие потока:

1. <a id="generate-hyperlink"></a>Бот создает и отображает гиперссылку, которая будет перенаправлять пользователя на веб-сайт. 
   Гиперссылка обычно содержит данные с параметрами строки запроса на целевой URL-адрес, указывающий сведения о контексте текущего диалога, например ИД беседы, идентификатор канала и идентификатор пользователя в канале. 

2. Пользователь щелкает гиперссылку и перенаправляется на целевой URL-адрес в веб-браузере. 

3. Бот переходит в состояние ожидания коммуникационного взаимодействия от веб-сайта для указания, что поток веб-сайта выполнен.  
   > [!TIP]
   > Проектируйте этот поток таким образом, чтобы бот не оставался постоянно в состоянии "ожидания", если пользователь никогда не завершит поток веб-сайта. Другими словами, если пользователь отменяет веб-браузер и начинает взаимодействовать с ботом опять, то бот должен подтвердить, а не [игнорировать](~/bot-service-design-navigation.md#the-mysterious-bot) эти входные данные.

4. Пользователь выполняет необходимые задачи в веб-браузере. 
   Это может быть поток OAuth или любая последовательность событий, необходимых в имеющемся сценарии. 

5. <a id="generate-magic-number"></a>Когда пользователь завершает поток веб-сайта, то веб-сайт создает "[магическое число](#verify-identity)" и указывает пользователю, чтобы тот скопировал значение и вставил его в диалог с ботом. 

6. <a id="signal-to-bot"></a>Веб-сайт [сигнализирует боту](#website-signal-to-bot), что пользователь прошел поток веб-сайта. 
   И через "магическое число" взаимодействует с ботом и предоставляет все необходимые релевантные данные.
   Например, в случае потока OAuth, веб-сайт предоставит маркер доступа боту.

7. Пользователь возвращается к боту и в чате вставляет "магическое число". 
   Бот проверяет, что "магическое число", указанное пользователем соответствует ожидаемому значению. Это проверка того, что текущий пользователь является тем же пользователем, который ранее щелкнул гиперссылку, чтобы инициировать поток веб-сайта. 

### <a id="verify-identity"></a> Проверка удостоверения пользователя с помощью "магического числа"

Создание "магического числа" во время потока от бота к веб-сайту ([шаг 5](#generate-magic-number) выше) позволяет боту впоследствии убедиться, что пользователь, инициировавший поток веб-сайта, действительно является тем же пользователем, для которого он был предназначен. Например, если бот проводит групповой чат с несколькими пользователями, любой из них может щелкнуть гиперссылку, чтобы инициировать поток веб-сайта. Без процесса проверки "магического числа" y бота нет способа узнать, какой пользователь завершил поток. Один пользователь может пройти проверку подлинности и внедрить маркеры доступа в сеансе другого пользователя. 

> [!WARNING] 
> Это не только риск в рамках группового чата. Без процесса проверки "магического числа" любой, получивший гиперссылку, чтобы запустить поток веб-сайта, может подделать удостоверение пользователя. 

Магическое число должно быть случайным числом, созданным с помощью библиотеки устойчивого шифрования. Пример процесса создания в C# см. в <a href="https://github.com/MicrosoftDX/botauth/tree/master/CSharp" target="_blank">этом коде</a> в библиотеке <a href="https://www.nuget.org/packages/BotAuth" target="_blank">BotAuth</a>. BotAuth включает ботов, которые создаются на платформе Microsoft Bot Framework для реализации потока от бота к веб-сайту для проверки подлинности пользователя на веб-сайте, чтобы впоследствии использовать маркер доступа, созданный из процесса проверки подлинности. Так как BotAuth не делает никаких предположений о возможностях канала, такие потоки должны хорошо работать с большинством каналов. 

> [!NOTE]
> При создании каналами собственных внедренных веб-представлений не рекомендуется выполнять процесс проверки "магического числа".

### <a id="website-signal-to-bot"></a> Как веб-сайт "сигнализирует" боту?

Когда бот [создает гиперссылку](#generate-hyperlink), которую пользователь выбирает для инициации потока веб-сайта, она включает информацию в параметрах строки запроса в целевом URL-адресе о контексте текущего диалога, например ИД беседы, идентификатор канала и идентификатор пользователя в канале. Эти сведения впоследствии могут использоваться веб-сайтом для чтения и записи переменных состояния этого пользователя или диалога с помощью пакета SDK Bot Framework или REST API. Пример того, как веб-сайт "сигнализирует" боту о завершении потока веб-сайта, смотрите на [шаге 6](#signal-to-bot) выше.

## <a name="sample-code"></a>Образец кода

Как описано в этой статье, библиотека <a href="https://github.com/MicrosoftDX/botauth" target="_blank">BotAuth</a> позволяет потокам OAuth быть привязанными к ботам, которые создаются с помощью .NET и Node в Microsoft Bot Framework.

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Диалоги](~/dotnet/bot-builder-dotnet-dialogs.md)
- [Управление потоком беседы с помощью диалогов (.NET)](~/dotnet/bot-builder-dotnet-manage-conversation-flow.md)
- [Управление потоком беседы с помощью диалогов (.Node.js)](~/nodejs/bot-builder-nodejs-manage-conversation-flow.md)

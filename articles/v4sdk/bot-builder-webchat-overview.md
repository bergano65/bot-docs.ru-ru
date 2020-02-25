---
title: Общие сведения о веб-чате — Служба Azure Bot
description: Сведения о настройке веб-чата Bot Framework.
keywords: bot framework, webchat, chat, samples, react, reference
author: ivorb
ms.author: kamrani
manager: kamrani
ms.topic: article
ms.service: bot-service
ms.date: 06/07/2019
ms.openlocfilehash: a85f1ddcb9a1440d8b157b3e28a7cbefb834955a
ms.sourcegitcommit: d24fe2178832261ac83477219e42606f839dc64d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/07/2020
ms.locfileid: "77071792"
---
# <a name="web-chat-overview"></a>Общие сведения о веб-чате

В этой статье содержатся сведения о компоненте [веб-чата Bot Framework](https://github.com/microsoft/BotFramework-WebChat). Компонент веб-чата Bot Framework представляет собой веб-клиент с широкими возможностями настройки для пакета SDK для Bot Framework версии 4. Пакет SDK для Bot Framework версии 4 позволяет разработчикам моделировать беседы и создавать сложные приложения-боты.

Если вы хотите перейти с использования веб-чата версии 3 на использование версии 4, перейдите к [этому разделу](#migrating-from-web-chat-v3-to-v4).

## <a name="how-to-use"></a>Использование

> [!NOTE]
> Сведения о предыдущих версиях веб-чата (версия 3) см. [на сайте GitHub](https://github.com/Microsoft/BotFramework-WebChat/tree/v3).

Сначала создайте бот с помощью [службы Azure Bot](https://azure.microsoft.com/services/bot-service/).
После этого вам необходимо [получить секрет веб-чата бота](../bot-service-channel-connect-webchat.md#get-your-bot-secret-key) на портале Azure. Затем с помощью секрета [создайте маркер](../rest-api/bot-framework-rest-direct-line-3-0-authentication.md) и передайте его в свой веб-чат.

Добавить элемент управления веб-чата на веб-сайт можно следующим образом:

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID',
               username: 'Web Chat User',
               locale: 'en-US',
               botAvatarInitials: 'WC',
               userAvatarInitials: 'WW'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

> Передавать параметры `userID`, `username`, `locale`, `botAvatarInitials` и `userAvatarInitials` в метод `renderWebChat` необязательно. Дополнительные сведения о свойствах веб-чата см. в разделе [Справочник по API веб-чата](#web-chat-api-reference) в этой статье.
> ![Снимок экрана веб-чата ](https://raw.githubusercontent.com/Microsoft/BotFramework-WebChat/master/media/weatherquery.png.jpg)

### <a name="integrate-with-javascript"></a>Интеграция с помощью JavaScript

Веб-чат разработан для интеграции с имеющимся веб-сайтом с помощью JavaScript или React. Интеграция с JavaScript предоставляет поддержку стилей и настроек. Дополнительные сведения см. в статье [об интеграции WebChat с веб-сайтом.](https://aka.ms/integrate-webchat-into-site)

Вы можете воспользоваться полным стандартным пакетом веб-чата, содержащим часто используемые компоненты.

```html
<!DOCTYPE html>
<html>
   <body>
      <div id="webchat" role="main"></div>
      <script src="https://cdn.botframework.com/botframework-webchat/latest/webchat.js"></script>
      <script>
         window.WebChat.renderWebChat(
            {
               directLine: window.WebChat.createDirectLine({
                  token: 'YOUR_DIRECT_LINE_TOKEN'
               }),
               userID: 'YOUR_USER_ID'
            },
            document.getElementById('webchat')
         );
      </script>
   </body>
</html>
```

Рабочий пример полного пакета веб-чата можно найти [на сайте GitHub](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/01.a.getting-started-full-bundle).

### <a name="integrate-with-react"></a>Интеграция с помощью React

Для полноценной настройки вы можете использовать [React](https://reactjs.org/), чтобы реконструировать компоненты веб-чата.

Чтобы установить производственную сборку из npm, выполните команду `npm install botframework-webchat`.

```jsx
import { DirectLine } from 'botframework-directlinejs';
import React from 'react';
import ReactWebChat from 'botframework-webchat';

export default class extends React.Component {
  constructor(props) {
    super(props);

    this.directLine = new DirectLine({ token: 'YOUR_DIRECT_LINE_TOKEN' });
  }

  render() {
    return (
      <ReactWebChat directLine={ this.directLine } userID='YOUR_USER_ID' />
      element
    );
  }
}
```

> Кроме того, с помощью команды `npm install botframework-webchat@master` можно установить сборку разработки, синхронизированную с веткой `master` веб-чата на сайте GitHub.

Ознакомьтесь с рабочим примером [веб-чата, созданного с помощью React, на сайте GitHub](https://github.com/Microsoft/BotFramework-WebChat/tree/master/samples/03.a.host-with-react/).

> [!TIP]
> Если вы не знакомы с React и JSX, воспользуйтесь обучающими материалами на странице [о начале работы с React](https://reactjs.org/docs/getting-started.html).


## <a name="customize-web-chat-ui"></a>Настройка пользовательского интерфейса веб-чата

Разработка веб-чата предполагает настройку без разветвления исходного кода. В приведенной ниже таблице указано, какие настройки можно выполнять, импортируя веб-чат различными способами. Этот список не является исчерпывающим.

|                               | Пакет CDN         | React              |
| ----------------------------- | ------------------ | ------------------ |
| Изменение цвета                 | :heavy_check_mark: | :heavy_check_mark: |
| Изменение размеров                  | :heavy_check_mark: | :heavy_check_mark: |
| Обновление и замена стилей CSS     | :heavy_check_mark: | :heavy_check_mark: |
| Прослушивание событий              | :heavy_check_mark: | :heavy_check_mark: |
| Взаимодействие с веб-страницей размещения | :heavy_check_mark: | :heavy_check_mark: |
| Настраиваемые действия для преобразования для просмотра      |                    | :heavy_check_mark: |
| Настраиваемые вложения для преобразования для просмотра     |                    | :heavy_check_mark: |
| Добавление компонентов пользовательского интерфейса         |                    | :heavy_check_mark: |
| Реконструирование всего пользовательского интерфейса        |                    | :heavy_check_mark: |

Ознакомьтесь с дополнительными сведениями [о настройке веб-чата на сайте GitHub](https://github.com/Microsoft/BotFramework-WebChat/blob/master/SAMPLES.md).

> [!NOTE]
> Сведения о сетях доставки содержимого (CDN) см. в [этой статье](https://aka.ms/CDN-best-practices).

## <a name="migrating-from-web-chat-v3-to-v4"></a>Переход с использования версии 3 веб-чата на использование версии 4

Существует три возможных способа перехода с использования версии 3 на использование версии 4. Сначала сравните ваш начальный сценарий, как описано выше.

Список связанных примеров см. в статье [Web Chat hosted samples](https://aka.ms/botframework-webchat-samples) (Размещенные примеры для Web Chat).

### <a name="my-current-website-integrates-web-chat-using-an-iframe-element-obtained-from-azure-bot-services-i-want-to-upgrade-to-v4"></a>Мой текущий веб-сайт интегрирует веб-чат с помощью элемента `<iframe>`, полученного из службы Azure Bot. Я хочу перейти на использование версии 4.

Начиная с мая 2019 года мы развертываем версию 4 на веб-сайты, которые интегрируют веб-чат с помощью элемента `<iframe>`. Сведения об интеграции веб-чата с помощью `<iframe>` см. в [этой статье](https://github.com/Microsoft/BotFramework-WebChat/tree/master/packages/embed).

### <a name="my-website-is-integrated-with-web-chat-v3-and-uses-customization-options-provided-by-web-chat-no-customization-at-all-or-very-little-of-my-own-customization-that-was-not-available-with-web-chat"></a>Мой веб-сайт интегрирован с веб-чатом версии 3 и использует предоставляемые веб-чатом параметры настройки, не использует настройки вообще или же использует малое количество пользовательских настроек, недоступных в веб-чате.

Выполните реализацию примера [`00.migration/a.v3-to-v4`](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/00.migration/a.v3-to-v4), чтобы преобразовать веб-страницу для использования версии 4 веб-чата вместо версии 3.

### <a name="my-website-is-integrated-with-a-fork-of-web-chat-v3-i-have-implemented-a-lot-of-customization-in-my-version-of-web-chat-and-i-am-concerned-v4-is-not-compatible-with-my-needs"></a>Мой сайт интегрирован с разветвлением веб-чата версии 3. Я реализовал множество настроек в своей версии веб-чата. Будет ли версия 4 соответствовать моим требованиям?

Преимуществом версии 4 является возможность добавлять настройки **без необходимости в разветвлении веб-чата**. Хотя это приведет к дополнительным расходам для пользователей версии 3, которые ранее выполняли разветвление веб-чата, мы постараемся обеспечить поддержку таким клиентам. Попробуйте использовать рекомендации, приведенные ниже.

-  Ознакомьтесь со сведениями о реализации примера [`00.migration/a.v3-to-v4`](https://github.com/microsoft/BotFramework-WebChat/tree/master/samples/00.migration/a.v3-to-v4). Это отличная отправная точка для начала работы с веб-чатом.
-  Затем просмотрите [список примеров](https://aka.ms/botframework-webchat-samples), чтобы сравнить свои требования к настройке с поддерживаемыми в веб-чате. Эти примеры состоят из часто запрашиваемых компонентов для веб-чата.
-  Если необходимые вам компоненты отсутствуют в примерах, просмотрите страницы с [рассматриваемыми и уже решенными проблемами](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+), [меткой примеров](https://github.com/Microsoft/BotFramework-WebChat/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3ASample) и [меткой поддержки перехода](https://github.com/Microsoft/BotFramework-WebChat/issues?q=is%3Aissue+migrate+label%3A%22Migration+Support%22), чтобы найти примеры запросов или поддержку настройки для необходимых компонентов. Комментируя рассматриваемые вопросы, вы поможете команде определить приоритеты распространенных запросов. Мы рекомендуем принимать участие в обсуждениях в нашем сообществе.
-  Если вы не нашли необходимый компонент в списке открытых запросов, [отправьте собственный запрос](https://github.com/Microsoft/BotFramework-WebChat/issues/new). Как упоминалось ранее, другие клиенты, добавляющие комментарии к вашей нерешенной проблеме, помогут нам определить, какие компоненты нужны пользователям веб-чата чаще всего.
-  Наконец, если компонент нужен вам как можно скорее, вы можете отправить [запрос на вытягивание](https://github.com/Microsoft/BotFramework-WebChat/compare) в веб-чат. Если вам известен код для реализации этого компонента, мы будем благодарны за дополнительную поддержку! Самостоятельное создание этого компонента сделает его доступным не только для вас, но и для других клиентов, ищущих такой же или похожий компонент.
-  Обязательно ознакомьтесь с остальной частью этого руководства `README`, чтобы узнать больше о версии 4.


## <a name="web-chat-api-reference"></a>Справочник по API веб-чата

Существует несколько свойств, которые можно передать в компонент React веб-чата (`<ReactWebChat>`) или метод `renderWebChat()`. Изучите исходный код, начав с [`packages/component/src/Composer.js`](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Composer.js#L378). Ниже приведено краткое описание доступных свойств.

| Свойство                   | Описание                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `activityMiddleware`       | Цепочка ПО промежуточного слоя, смоделированная на основе [ПО промежуточного слоя Redux](https://medium.com/@jacobp100/you-arent-using-redux-middleware-enough-94ffe991e6), позволяет разработчику добавлять компоненты новой модели DOM в имеющуюся модель DOM действий. Подпись ПО промежуточного слоя выглядит следующим образом: `options => next => card => children => next(card)(children)`.                                                                                                                                                                                                                                           |
| `activityRenderer`         | "Плоская" версия `activityMiddleware`, аналогичная концепции [средства улучшения хранилища](https://github.com/reduxjs/redux/blob/master/docs/Glossary.md#store-enhancer) в Redux.                                                                                                                                                                                                                                                                                                                                                                                                                |
| `adaptiveCardHostConfig`   | Передайте пользовательскую конфигурацию узла адаптивных карточек. Обязательно проверьте конфигурацию узла с используемой версией адаптивных карточек. Дополнительные сведения о конфигурации узла см. [на этой странице](https://github.com/microsoft/BotFramework-WebChat/issues/2034#issuecomment-501818238).                                                                                                                                                                                                                                                                                                                                    |
| `attachmentMiddleware`     | Цепочка ПО промежуточного слоя, которая позволяет разработчику добавлять собственные HTML-элементы во вложения. Подпись выглядит следующим образом: `options => next => card => next(card)`.                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `attachmentRenderer`       | "Плоская" версия `attachmentMiddleware`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `cardActionMiddleware`     | Цепочка ПО промежуточного слоя, которая позволяет разработчику изменять действия карточек, например действия адаптивных карточек или предлагаемые действия. Подпись ПО промежуточного слоя выглядит следующим образом: `cardActionMiddleware: () => next => ({ cardAction, getSignInUrl }) => next(cardAction)`.                                                                                                                                                                                                                                                                                                                                           |
| `createDirectLine`         | Фабричный метод для создания экземпляра объекта Direct Line. Пользователи Azure для государственных организаций должны использовать `createDirectLine({ domain: 'https://directline.botframework.azure.us/v3/directline', token });` для изменения конечной точки. Полный список параметров: `conversationId`, `domain`, `fetch`, `pollingInterval`, `secret`, `streamUrl`, `token`, `watermark` `webSocket`.                                                                                                                                                                                                                         |
| `createStore`              | Цепочка ПО промежуточного слоя, которая позволяет разработчику изменять действия хранилища. Подпись ПО промежуточного слоя выглядит следующим образом: `createStore: ({}, ({ dispatch }) => next => action => next(cardAction)`.                                                                                                                                                                                                                                                                                                                                                                                                |
| `directLine`               | Укажите объект DirectLine с помощью маркера DirectLine.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `disabled`                 | Отключите пользовательский интерфейс (например, для режима презентации) веб-чата.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `grammars`                 | Укажите список правил грамматики для службы "Речь" (Распознавание речи Bing или службы распознавания речи Cognitive Services).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| `groupTimeStamp`           | Измените настройки по умолчанию для группирования меток времени.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| `locale`                   | Укажите язык по умолчанию для веб-чата. Настоятельно рекомендуем использовать коды, состоящие из четырех букв (например, `en-US`).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `renderMarkdown`           | Измените объект отрисовщика Markdown по умолчанию.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `sendTypingIndicator`      | Отобразите сигнал ввода от пользователя к боту, чтобы указать, что пользователь не находится в состоянии простоя.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `store`                    | Укажите пользовательское хранилище, например для добавления программной активности в бот.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| `styleOptions`             | Объект, хранящий значения настройки для задания стиля веб-чата. Полный список (часто обновляемых) параметров стиля по умолчанию см. в файле [defaultStyleOptions.js](https://github.com/Microsoft/BotFramework-WebChat/blob/master/packages/component/src/Styles/defaultStyleOptions.js).                                                                                                                                                                                                                                                                              |
| `styleSet`                 | Не рекомендуемый способ переопределения стилей.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `userID`                   | Укажите идентификатор пользователя. Вы можете указать `userID` двумя методами: в свойствах или в маркере при создании вызова маркера (`createDirectLine()`). При использовании обоих методов будет использовано свойство userID маркера, и во время выполнения отобразится `console.warn`. Если `userID` предоставляется через свойства, но содержит префикс `'dl'`, например `'dl_1234'`, значение будет удалено. Затем будет создан идентификатор `ID`. Если идентификатор `userID` не указан, по умолчанию будет использоваться случайный идентификатор пользователя. Использование одного идентификатора пользователя несколькими пользователями не рекомендуется; их пользовательское состояние будет использоваться совместно. |
| `username`                 | Укажите имя пользователя.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `webSpeechPonyFillFactory` | Укажите объект Web Speech для преобразования текста в речь и речи в текст.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
## <a name="browser-compatibility"></a>Совместимость с браузерами
Веб-чат поддерживает две последние версии современных браузеров, таких как Chrome, Microsoft Edge и FireFox.
Если вам требуется веб-чат в Internet Explorer 11, ознакомьтесь с руководством о [демонстрации пакета ES5](https://microsoft.github.io/BotFramework-WebChat/01.b.getting-started-es5-bundle).

Но обратите внимание на следующее:
- Веб-чат не поддерживает Internet Explorer старше версии 11.
- Настройка, описанная в примерах, не относящихся к ES5, не поддерживается для Internet Explorer. Так как IE11 — устаревший браузер, он не поддерживает ES6, и многие примеры, использующие функции стрелок и современные возможности, необходимо будет вручную преобразовать в ES5.  Если предполагается полноценная настройка приложения, настоятельно рекомендуем разработать приложение для современного браузера, такого как Google Chrome или Edge.
- Реализация поддержки примеров IE11 (ES5) для веб-чата не планируется.
   - Для клиентов, которые хотят вручную перезаписать другие примеры для работы в IE11, рекомендуем рассмотреть возможность преобразования кода из ES6+ в ES5 с использованием полизаполнений и компиляторов, таких как [`babel`](https://babeljs.io/docs/en/next/babel-standalone.html).

## <a name="how-to-test-with-web-chats-latest-bits"></a>Выполнение тестирования с помощью последних фрагментов кода веб-чата

*В настоящее время тестирование невыпущенных компонентов доступно только с помощью упаковки MyGet.*

Если вы хотите протестировать компонент или исправление ошибки, которые еще не были выпущены, укажите свой пакет веб-чата в веб-канале чата с ежедневным обновлением, а не в официальном веб-канале npmjs.

В настоящее время вы можете получить доступ к веб-каналу чата с ежедневным обновлением, подписавшись на наш веб-канал MyGet. Для этого вам нужно будет обновить реестр в своем проекте. **Это изменение обратимо. В наших инструкциях описан процесс возвращения к подписке на официальный выпуск**.

### <a name="subscribe-to-latest-bits-on-mygetorg"></a>Подписка на получение сведений о новых фрагментах кода на сайте `myget.org`

Для этого вы можете добавить свои пакеты, а затем изменить реестр проекта.

1. Добавьте зависимости своего проекта, кроме веб-чата.
1. В корневом каталоге проекта создайте файл `.npmrc`.
1. Добавьте в файл следующую строку: `registry=https://botbuilder.myget.org/F/botframework-webchat/npm/`.
1. Добавьте веб-чат в зависимости проекта `npm i botframework-webchat --save`.
1. Обратите внимание, что теперь в `package-lock.json` реестры указывают на MyGet. В проекте веб-чата включен вышестоящий прокси-сервер, который перенаправляет пакеты, отличные от MyGet, в `npmjs.com`.

### <a name="re-subscribe-to-official-release-on-npmjscom"></a>Повторная подписка на официальный выпуск на сайте `npmjs.com`
Повторная подписка требует сброса реестра.

1. Удалите `.npmrc file`.
1. Удалите корневой элемент `package-lock.json`.
1. Удалите каталог `node_modules`.
1. Переустановите пакеты с помощью `npm i`.
1. Обратите внимание, что в `package-lock.json` реестры снова указывают на https://npmjs.com/.


## <a name="contributing"></a>Участие

Подробные сведения о создании проекта и рекомендации репозитория по запросам на вытягивание см. на [этой странице](https://github.com/Microsoft/BotFramework-WebChat/tree/master/.github/CONTRIBUTING.md).

В рамках этого проекта действуют [правила поведения в отношении продуктов с открытым исходным кодом Майкрософт](https://opensource.microsoft.com/codeofconduct/).
Дополнительные сведения:[вопросы и ответы по правилам поведения](https://opensource.microsoft.com/codeofconduct/faq/). С любыми другими вопросами или комментариями обращайтесь по адресу[opencode@microsoft.com](mailto:opencode@microsoft.com).

## <a name="reporting-security-issues"></a>Сообщение о проблемах с безопасностью

О проблемах с безопасностью и ошибках следует сообщать в частном порядке по электронной почте в Центр реагирования на вопросы безопасности (MSRC) по адресу [secure@microsoft.com](mailto:secure@microsoft.com). Вы должны получить ответ в течение 24 часов. Если по какой-либо причине вы не получили ответ, свяжитесь с нами по электронной почте, чтобы убедиться, что мы получили исходное сообщение. Дополнительные сведения, включая ключ [MSRC PGP](https://technet.microsoft.com/security/dn606155), можно найти в [техническом центре безопасности](https://technet.microsoft.com/security/default).

(c) Корпорация Майкрософт (Microsoft Corporation). Все права защищены Все права защищены.
